Thanks for the clarification. Since you have a **remote OpenTelemetry Collector** acting as the middle layer, the typical secure flow for **Temenos T24** telemetry is:

**T24 (instrumented with OpenTelemetry or via DataEventStreaming / JMX exporter) → Remote OTel Collector (over OTLP with TLS) → Prometheus (via remote_write with TLS or Prometheus exporter)**

Here’s how to enable **TLS** on the relevant parts, focusing on the connections involving the remote collector.

### 1. Configure T24 Side to Send Telemetry Securely to the Remote Collector
T24 (especially TAFJ-based or newer Transact releases) uses **OpenTelemetry instrumentation** or the **DataEventStreaming** façade to emit metrics/traces/logs in OTLP format.

- In your T24 configuration (usually in properties files, `TAFJ` JVM args, or DataEventStreaming config):
  - Point the OTLP exporter to your remote collector’s **HTTPS/gRPC** endpoint.
  - Example JVM system properties or OTel environment variables for T24/TAFJ:

    ```properties
    -Dotel.exporter.otlp.endpoint=https://your-remote-collector.example.com:4317   # gRPC (preferred for performance)
    # or for HTTP: https://your-remote-collector.example.com:4318

    -Dotel.exporter.otlp.protocol=grpc   # or http/protobuf
    -Dotel.exporter.otlp.insecure=false
    -Dotel.exporter.otlp.headers=Authorization=Bearer your-token-if-any   # optional

    # TLS client certificates (if mTLS required)
    -Djavax.net.ssl.keyStore=/path/to/t24-client-keystore.jks
    -Djavax.net.ssl.keyStorePassword=yourpassword
    -Djavax.net.ssl.trustStore=/path/to/ca-truststore.jks   # CA that signed the collector's cert
    -Djavax.net.ssl.trustStorePassword=yourpassword

    # Force strong TLS
    -Dhttps.protocols=TLSv1.2,TLSv1.3
    ```

- If using **DataEventStreaming** or Temenos-specific telemetry config, look for parameters like:
  - `OTEL_EXPORTER_OTLP_ENDPOINT`
  - `OTEL_EXPORTER_OTLP_CERTIFICATE` or similar TLS-related keys in the streaming façade configuration.

Restart the T24 component (TCServer, TAFJ nodes, etc.) after changes.

**Tip**: Start with `insecure=false` and proper CA trust. Use mTLS only if your security policy requires client certificate authentication from T24.

### 2. Configure the Remote OpenTelemetry Collector for TLS
You need to enable TLS on two sides in the collector’s `config.yaml`:

#### A. Receiver Side (Incoming from T24) – Server TLS
Secure the OTLP receiver so T24 can send data over encrypted channels:

```yaml
receivers:
  otlp:
    protocols:
      grpc:                  # Recommended for T24 → Collector
        endpoint: 0.0.0.0:4317
        tls:
          cert_file: /path/to/collector-server.crt     # Server certificate
          key_file: /path/to/collector-server.key      # Private key
          # For mTLS (stronger auth):
          # client_ca_file: /path/to/client-ca.crt     # CA to validate T24's client cert
          min_version: "1.2"

      http:                  # Alternative if T24 uses HTTP/protobuf
        endpoint: 0.0.0.0:4318
        tls:
          cert_file: /path/to/collector-server.crt
          key_file: /path/to/collector-server.key
```

#### B. Exporter Side (Outgoing to Prometheus) – Client TLS
Choose one of the common options:

**Option 1: prometheusremotewrite exporter (push model – recommended for remote Prometheus)**

```yaml
exporters:
  prometheusremotewrite:
    endpoint: https://your-remote-prometheus.example.com/api/v1/write
    tls:
      insecure: false
      ca_file: /path/to/prometheus-ca.crt          # CA that signed Prometheus cert
      # If Prometheus requires client cert (mTLS):
      # cert_file: /path/to/collector-client.crt
      # key_file: /path/to/collector-client.key
      min_version: "1.2"
```

**Option 2: If you expose a /metrics endpoint on the collector for Prometheus to scrape** (use prometheus exporter)

```yaml
exporters:
  prometheus:
    endpoint: 0.0.0.0:8889
    tls:
      cert_file: /path/to/collector-metrics.crt
      key_file: /path/to/collector-metrics.key
      # min_version etc.
```

### 3. Full Example Collector Pipeline (Metrics Only)

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
        tls:
          cert_file: /etc/certs/collector-server.crt
          key_file: /etc/certs/collector-server.key

processors:
  batch:
  memory_limiter:

exporters:
  prometheusremotewrite:
    endpoint: https://prometheus.example.com/api/v1/write
    tls:
      insecure: false
      ca_file: /etc/certs/prom-ca.crt

service:
  pipelines:
    metrics:
      receivers: [otlp]
      processors: [batch, memory_limiter]
      exporters: [prometheusremotewrite]
```

### 4. Prometheus Side Configuration (to Accept from Collector)
In `prometheus.yml`:

```yaml
remote_write:
  - url: https://your-prometheus.example.com/api/v1/write   # if collector pushes
    tls_config:
      ca_file: /path/to/collector-ca.crt
      # cert_file / key_file only if mTLS

# OR if scraping the collector's /metrics:
scrape_configs:
  - job_name: 'otel-collector'
    scheme: https
    tls_config:
      ca_file: /path/to/collector-ca.crt
      insecure_skip_verify: false
    static_configs:
      - targets: ['your-remote-collector.example.com:8889']
    metrics_path: /metrics
```

### 5. Best Practices & Troubleshooting
- **Certificate Management**: Use a proper internal CA. Place certs with restrictive permissions (`600` for keys).
- **Testing**:
  - From T24 server: `curl -v --cacert ca.pem https://collector:4318` (or use `grpcurl` for gRPC).
  - Validate collector config: `otelcol --config=config.yaml --dry-run`
- **Common Issues**:
  - "x509: certificate signed by unknown authority" → Add correct CA to truststore on T24 and collector.
  - mTLS handshake failures → Ensure client cert is trusted by the receiver’s `client_ca_file`.
  - Performance: Prefer gRPC + TLS over HTTP for high-volume T24 telemetry.
- **Security**: Avoid `insecure: true` or `insecure_skip_verify: true` in production. Enable `min_version: "1.2"` or `"1.3"`.

If your T24 version uses a specific telemetry module (e.g., exact name of the DataEventStreaming property or JMX agent setup), or if you’re hitting any error messages during testing, share them and I can refine the steps further.

Also, confirm:
- Is T24 pushing **OTLP metrics** directly, or are you using a different mechanism (JMX → collector, etc.)?
- Do you need **mTLS** or just one-way TLS?

This setup should give you end-to-end encrypted telemetry from T24 through the remote collector to Prometheus.
