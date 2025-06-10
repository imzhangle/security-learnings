# security-learnings


Hardening the **Ingress Controller** in OpenShift is critical for securing your cluster's external traffic. The Ingress Controller (based on HAProxy by default in OpenShift) acts as the entry point for external traffic into the cluster, so it must be configured securely to protect against attacks and misconfigurations.

Below is a comprehensive guide to **OpenShift Ingress hardening**, covering best practices, configurations, and security considerations:

---

## 🔐 1. **Use TLS Everywhere**
Ensure all routes use **HTTPS/TLS** encryption.

### ✅ Steps:
- Use **Red Hat OpenShift Service Mesh** or **cert-manager** to automate certificate management.
- Enforce **re-encryption** if using internal services with certificates.
- Disable plain HTTP unless absolutely necessary.

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: secure-route
spec:
  host: secure.example.com
  tls:
    termination: edge
    key: <PEM-encoded key>
    certificate: <PEM-encoded certificate>
    caCertificate: <CA cert if needed>
    destinationCACertificate: <internal CA cert if re-encrypting>
  to:
    kind: Service
    name: my-service
```

---

## 🔒 2. **Disable Unsecured Routes**
By default, OpenShift allows unencrypted HTTP traffic. You should disable this.

### ✅ Steps:
Edit the IngressController to disable HTTP:

```bash
oc edit ingresscontroller -n openshift-ingress-operator <ingress-name>
```

Add:

```yaml
spec:
  routeAdmission:
    disableHTTP: true
```

This blocks cleartext HTTP access to all routes served by this controller.

---

## 🧱 3. **Restrict Access Using IP Whitelisting**
Limit access to certain routes based on client IP addresses.

### ✅ Steps:
Update the IngressController:

```yaml
spec:
  endpointPublishingStrategy:
    type: HostNetwork
  httpHeaders:
    XForwardedHeaders: Append
  routeAdmission:
    allowedInsecureEdgeTerminationPolicyTypes:
      - None
    sourceCIDRs:
      - 192.168.100.0/24
```

This restricts access to only clients coming from the specified CIDR range.

> ⚠️ Ensure you are not locking yourself out.

---

## 🛡️ 4. **Harden TLS Configuration**
Ensure strong ciphers and protocols are used.

### ✅ Steps:
Create a custom `IngressController` with specific TLS settings:

```yaml
spec:
  tlsSecurityProfile:
    type: Custom
    cipherSuites:
      - ECDHE-ECDSA-AES256-GCM-SHA384
      - ECDHE-RSA-AES256-GCM-SHA384
      - ECDHE-ECDSA-CHACHA20-POLY1305
      - ECDHE-RSA-CHACHA20-POLY1305
      - ECDHE-ECDSA-AES128-GCM-SHA256
      - ECDHE-RSA-AES128-GCM-SHA256
    minTLSVersion: VersionTLS12
```

---

## 🧼 5. **Sanitize Headers and Prevent Information Disclosure**
Avoid leaking unnecessary headers like `Server`, `Via`, etc.

### ✅ Steps:
Patch HAProxy via ConfigMap or modify IngressController config.

Example patch (advanced):
- Modify the HAProxy template used by the router pod to remove unwanted headers.

---

## 🔒 6. **Rate Limiting and DDoS Protection**
OpenShift does not provide native rate limiting, but you can integrate:

### ✅ Options:
- Use **OpenShift Service Mesh (Istio)** with Envoy’s rate-limiting features.
- Deploy a **WAF (Web Application Firewall)** like **ModSecurity** alongside HAProxy.
- Use cloud provider tools (e.g., AWS WAF, Azure Front Door, Cloudflare).

---

## 🔐 7. **Separate Sensitive Applications**
Use multiple Ingress Controllers for different classes of applications.

### ✅ Steps:
Deploy additional IngressControllers:

```bash
oc create namespace sensitive-ingress
oc apply -f - <<EOF
apiVersion: operator.openshift.io/v1
kind: IngressController
metadata:
  name: sensitive-router
  namespace: openshift-ingress-operator
spec:
  namespaceSelector:
    matchLabels:
      app: sensitive
  replicas: 2
  endpointPublishingStrategy:
    type: HostNetwork
EOF
```

Apply strict policies to this dedicated Ingress Controller.

---

## 📊 8. **Monitor Logs and Metrics**
Enable logging and monitoring for suspicious activity.

### ✅ Steps:
- Enable access logs for the IngressController:

```yaml
spec:
  logging:
    access:
      enabled: true
```

- Integrate with **OpenShift Logging** or external SIEM (e.g., Splunk, ELK).
- Use Prometheus and Grafana for real-time metrics.

---

## 🔍 9. **Regular Security Audits**
Perform regular checks and updates.

### ✅ Steps:
- Use `oc adm` commands to audit routes and IngressController configs.
- Keep OpenShift updated to latest versions (especially for CVE fixes).
- Run penetration tests on exposed endpoints.

---

## 🧩 10. **Integrate with External Authentication**
Offload authentication to an identity provider.

### ✅ Steps:
- Configure OAuth with external providers (LDAP, GitHub, OIDC, etc.).
- Use JWT validation in Service Mesh for per-route auth.
- Protect routes with OpenShift’s built-in OAuth proxy sidecar if needed.

---

## 🧪 Bonus: Tools for Hardening & Testing

| Tool | Purpose |
|------|---------|
| `curl -v https://myroute.example.com` | Check TLS handshake and headers |
| `nuclei` | Scan for common web vulnerabilities |
| `testssl.sh` | Test SSL/TLS configuration |
| `openshift-install` | Inspect IPI-generated IngressController config |
| `oc get ingresses.config.openshift.io` | View global Ingress settings |

---

## ✅ Summary Checklist

| Hardening Step | Status |
|----------------|--------|
| Enforce HTTPS/TLS everywhere | ✅ |
| Disable HTTP access | ✅ |
| Use strong TLS versions/ciphers | ✅ |
| Restrict IPs via CIDR | ✅ |
| Separate ingress for sensitive apps | ✅ |
| Monitor and log traffic | ✅ |
| Implement rate limiting/WAF | ✅ |
| Use external identity providers | ✅ |

---

Would you like a sample hardened IngressController YAML manifest or help implementing any of these steps?
