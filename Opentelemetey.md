Yes, there are known vulnerabilities associated with the **OpenTelemetry Collector in gateway mode**, particularly when the OTLP gRPC receiver is exposed over the network (as required in your setup with external T24 sending telemetry across different networks). However, **most are mitigable** with proper configuration, and the Red Hat build used in OpenShift includes security patches and hardening.

### Primary Known Vulnerability: CVE-2024-36129 (Unsafe Decompression DoS)
- **Description**: An unauthenticated attacker can send specially crafted compressed gRPC (or HTTP) requests (e.g., "zip bomb" or decompression bomb using gzip, zstd, etc.) to the OTLP receiver, causing excessive memory consumption and crashing the Collector (Out-of-Memory kill).
- **Impact on gateway mode**: High relevance — gateway mode often exposes the OTLP gRPC receiver (port 4317) to the network for external senders like your T24 server. Without protections, this is exploitable remotely.
- **Affected versions**: OpenTelemetry Collector before v0.102.1 (fixed in confighttp/configgrpc modules too).
- **Status in 2026**: This is the most cited issue for OTLP gRPC receivers. Newer Collector versions (including Red Hat builds) include the fix. Red Hat has released security updates (e.g., RHSA-2026:4267 for opentelemetry-collector).

Other related issues include older gRPC-related DoS (e.g., CVE-2023-47108 in otelgrpc for unbound cardinality) and general collector concerns like high-cardinality metrics leading to resource exhaustion.

No major new CVEs specific to "gateway mode" itself were found — gateway mode is just a deployment pattern (centralized Deployment/StatefulSet). Vulnerabilities stem from the exposed receivers and configuration, not the mode per se.

Additional notes:
- A 2026 PATH hijacking issue (CVE-2026-24051) affects certain OpenTelemetry components (mainly Go SDK on macOS/Darwin, or contrib in some workflows) but is less directly relevant to a standard Collector gateway on OpenShift (Linux containers).
- Prometheus exporter side can inherit general Prometheus exposure risks (e.g., DoS via pprof or high cardinality), but that's separate from the gateway's OTLP receiver.

### Why Gateway Mode Increases Exposure in Your Case
Since T24 is external (different network), the OTLP gRPC receiver must be reachable via Route, LoadBalancer, or Ingress. This makes the "public port" scenario from CVE-2024-36129 more likely if not hardened.

### How to Resolve / Mitigate These Vulnerabilities
1. **Update Everything (Most Important Fix)**  
   - Use the latest **Red Hat build of OpenTelemetry** via the Operator (check for RHSA advisories like 2026:4267).  
   - Target Collector version ≥ 0.102.1 (ideally much newer, e.g., 0.110+ or current 2026 releases).  
   - Rebuild/redeploy your `OpenTelemetryCollector` CR after updating the Operator.

2. **Enable Strong Authentication + mTLS on OTLP gRPC Receiver**  
   - This prevents unauthenticated attacks like CVE-2024-36129.  
   - In your Collector CR (under `receivers.otlp.protocols.grpc`):  
     ```yaml
     tls:
       cert_file: /certs/server.crt
       key_file: /certs/server.key
       client_ca_file: /certs/ca.crt   # mTLS enforcement
     auth:   # Add bearer token or OIDC extension
       authenticator: bearer
     ```
   - On T24: Configure client cert + CA + optional authorization header.

3. **Add Resource Protection Processors**  
   ```yaml
   processors:
     memory_limiter:
       check_interval: 1s
       limit_percentage: 70
       spike_limit_percentage: 30
     batch:
   ```
   - This caps memory usage and prevents OOM from decompression bombs or high volume.

4. **Network and Exposure Controls (OpenShift-Specific)**  
   - Use **Route with Passthrough termination** for gRPC (end-to-end TLS).  
   - Apply strict **NetworkPolicy** allowing ingress only from T24's IP/CIDR.  
   - Prefer internal ClusterIP + LoadBalancer only if network allows; avoid exposing unnecessarily.  
   - Set resource requests/limits on the Collector Deployment to contain any DoS impact.

5. **Additional Hardening**  
   - Disable unnecessary receivers/exporters.  
   - Use attributes processor to drop sensitive or high-cardinality data.  
   - Monitor Collector self-metrics (`/metrics`) for anomalies (dropped spans, high memory).  
   - Regularly scan images with Trivy/Grype and apply OpenShift security updates.  
   - Follow official best practices: Store config/secrets securely, minimize privileges, use `localhost` defaults where possible (though not for your external gateway).

6. **For the Prometheus Side**  
   - Secure the `/metrics` endpoint with basic auth or OpenShift RBAC.  
   - Disable admin/pprof endpoints if not needed.

With these steps (especially updating + mTLS + memory_limiter), the gateway mode becomes quite secure for cross-network use. The decompression vulnerability is largely neutralized once authentication is enforced and the fixed version is running.

If your security scanner reports specific CVEs (share the IDs or output), or if you have the current Collector version/OpenShift version, I can give more precise mitigation steps or updated CR examples. Also check the official OpenTelemetry CVE list for the latest status.
