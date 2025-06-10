### 🔐 Excellent Follow-up!

You're asking:

> **Which version of MongoDB enables strong TLS versions (like TLS 1.2+) and secure cipher suites by default**, so that you don’t have to configure them manually?

Let’s dive into how MongoDB handles TLS settings across different versions, what's enabled by default, and what you need to do for compliance with modern security requirements like:

- **TLS 1.2+**
- **Strong cipher suites** (e.g., `ECDHE`-based)
- **No weak or deprecated protocols/ciphers**

---

## ✅ TL;DR: MongoDB Default TLS Behavior

| MongoDB Version | Default TLS Version | Secure Cipher Suites? | Requires Manual Config? |
|------------------|----------------------|------------------------|--------------------------|
| MongoDB ≤ 3.0    | TLS 1.0              | ❌ No                  | ✅ Yes                   |
| MongoDB 3.2 – 4.0| TLS 1.0/1.1/1.2      | ⚠️ Some                | ✅ Yes                   |
| MongoDB 4.2      | TLS 1.2 (if supported) | ⚠️ Some               | ✅ Yes                   |
| MongoDB 4.4+     | TLS 1.2+             | ✅ Yes (default ciphers are modern) | ⚠️ Still recommend custom config |
| MongoDB 5.0+     | TLS 1.2+             | ✅ Strong defaults     | ✅ Recommended to enforce |
| MongoDB 6.0+     | TLS 1.2+, optional TLS 1.3 | ✅ Strong defaults | ✅ Enforce if needed |

---

## 🧱 What MongoDB Enables By Default (as of v4.4+)

Starting from **MongoDB 4.4**, the server will negotiate:

- **TLS 1.2+** if supported by the client and system libraries
- A set of **modern cipher suites**, including:
  - `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`
  - `TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384`
  - `TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305`
  - `TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305`

However, **MongoDB does not disable older TLS versions (like TLS 1.0 or 1.1) by default**, and **does not restrict cipher suites strictly** unless explicitly configured.

---

## 🔒 To Be Fully Compliant with Modern Standards, You Must:

| Requirement | Action Required |
|-------------|-----------------|
| Disable TLS 1.0/1.1 | ✅ Yes — set `disabledProtocols` |
| Restrict cipher suites | ✅ Yes — set `sslCipherSuite` |
| Enforce mutual TLS (mTLS) | ✅ Optional but recommended |
| Use FIPS mode (if required) | ✅ Only in Enterprise or via OS-level FIPS |
| Require client certificates | ✅ Optional (`verifyClientCertificate`) |

---

## ✅ Example: MongoDB 5.0+ Secure Defaults

In MongoDB **5.0+**, here’s what’s enabled by default (assuming your system supports TLS 1.2+):

```yaml
sslCipherSuite: "DEFAULT@SECLEVEL=2"
```

This means:
- Weak ciphers like `DES`, `RC4`, `MD5` are disabled
- Prefer `ECDHE`-based ciphers
- TLS 1.2+ is used when possible

But again, it's not fully locked down unless you enforce it.

---

## 🔐 Sample Secure MongoDB Configuration (v5.0+)

```yaml
net:
  ssl:
    mode: requireSSL
    PEMKeyFile: /etc/mongodb/tls/server.pem
    CAFile: /etc/mongodb/tls/ca.pem
    disabledProtocols: TLSv1,TLSv1.1
    sslCipherSuite: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384:TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
    verifyClientCertificate: false  # Set to true for mTLS
```

Or via command line:

```bash
mongod \
  --sslMode requireSSL \
  --sslPEMKeyFile /etc/mongodb/tls/server.pem \
  --sslCAFile /etc/mongodb/tls/ca.pem \
  --sslDisabledProtocols TLSv1,TLSv1.1 \
  --sslCipherSuite "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384:TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384" \
  --sslVerifyClientCertificate
```

---

## 🧪 How to Check Enabled TLS Versions & Cipher Suites

From a client machine:

```bash
openssl ciphers -v 'DEFAULT@SECLEVEL=2'
```

To test against your MongoDB instance:

```bash
openssl s_client -connect mongodb-host:27017 -tls1_2 \
  -cipher TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
```

If the handshake fails, the cipher suite or TLS version isn't supported.

---

## 📊 Summary Table

| MongoDB Version | Supports TLS 1.2+ Out-of-the-box? | Secure Cipher Suites by Default? | Needs Manual Hardening? |
|------------------|-----------------------------------|----------------------------------|--------------------------|
| ≤ 3.6            | ❌ No                             | ❌ No                            | ✅ Yes                   |
| 4.0 – 4.2        | ⚠️ Conditional                    | ⚠️ Partial                        | ✅ Yes                   |
| 4.4              | ✅ Yes                            | ✅ Better                         | ✅ Yes                   |
| 5.0+             | ✅ Yes                            | ✅ Good                           | ✅ Recommended           |
| 6.0+             | ✅ Yes (TLS 1.3 support)          | ✅ Strong                          | ✅ Enforce strict policy |

---

## ✅ Best Practice Recommendation

| Task | Description |
|------|-------------|
| Use MongoDB ≥ 4.4 | For better TLS defaults |
| Always set `sslDisabledProtocols` | Disable TLS 1.0/1.1 |
| Set `sslCipherSuite` | Allow only secure ciphers |
| Enable mTLS (optional) | For internal services |
| Monitor logs for failed handshakes | Detect misconfigurations early |

---

## 🛠 Need Help Generating a Full Config?

I can help you generate:

- A full MongoDB TLS configuration enforcing TLS 1.2+ and modern cipher suites
- Kubernetes/OpenShift manifests for secure deployment
- Scripts to test connectivity and cipher suite enforcement
- Helm values for MongoDB Operator deployments

Would you like a working example tailored to your deployment model (e.g., standalone, replica set, OCP operator)? Let me know!
