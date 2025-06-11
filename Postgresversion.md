### 🔐 **Yes — Just Like MongoDB, Redis, and Elasticsearch, PostgreSQL Can Be Configured to Enforce TLS 1.2+ and Restrict Cipher Suites**

You asked:

> **Can PostgreSQL be configured to enforce strong TLS versions and cipher suites like TLS 1.2+, and fail if the client doesn’t support them?**

✅ **Short Answer: Yes**, PostgreSQL supports:
- Enforcing **TLS 1.2+**
- Restricting allowed **cipher suites**
- Requiring **client certificates (mTLS)**
- Failing connections that don't meet these requirements

Let’s walk through how to configure this in PostgreSQL and ensure it aligns with your security goals.

---

## ✅ PostgreSQL TLS Support Overview

| PostgreSQL Version | Native TLS Support | Notes |
|--------------------|---------------------|-------|
| ≤ 9.2               | ❌ Very limited      | Basic SSL support |
| 9.3 – 9.6          | ⚠️ Basic             | TLS 1.0–1.2 support |
| 10 – 13            | ✅ Good              | Supports TLS 1.2+ |
| 14+                | ✅ Strong            | Better defaults, better control over ciphers |

➡️ For full enforcement of **TLS 1.2+** and **cipher suite restrictions**, use **PostgreSQL 10 or newer**.

---

## 🔒 To Meet Your Requirements, You Must Configure PostgreSQL to:

| Requirement | Action Required |
|-------------|------------------|
| Enable TLS | ✅ Yes |
| Disable TLS 1.0/1.1 | ✅ Yes |
| Restrict cipher suites | ✅ Yes |
| Require client certificate (optional mTLS) | ✅ Optional |
| Fail handshake if no match | ✅ By default if no overlap |

---

## ✅ Example: PostgreSQL Secure TLS Configuration (v14+)

Here's a sample `postgresql.conf` configuration enforcing:

- TLS 1.2+
- Strong cipher suites
- Mutual TLS (optional)

```conf
# Enable SSL/TLS
ssl = on
ssl_cert_file = '/etc/postgresql/certs/server.crt'
ssl_key_file = '/etc/postgresql/certs/server.key'
ssl_ca_file = '/etc/postgresql/certs/root.crt'

# Enforce modern TLS versions
ssl_min_protocol_version = 'TLSv1.2'

# Restrict cipher suites
ssl_ciphers = 'HIGH:!aNULL:!MD5:!RC4:!DH:!kRSA'
```

Also update `pg_hba.conf` to require SSL for remote access:

```conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             0.0.0.0/0               md5
hostssl all             all             0.0.0.0/0               md5
hostnossl all          all             all                     reject
```

This ensures:
- Only connections using TLS are accepted
- Plaintext (non-TLS) connections are rejected

---

## 🧱 Full Configuration Breakdown

| Setting | Description |
|--------|-------------|
| `ssl = on` | Enables TLS support |
| `ssl_cert_file`, `ssl_key_file` | Paths to server certificate and private key |
| `ssl_ca_file` | CA cert for validating client certs (if using mTLS) |
| `ssl_min_protocol_version` | Minimum TLS version allowed |
| `ssl_ciphers` | Colon-separated list of allowed cipher suites |
| `hostssl` in `pg_hba.conf` | Forces TLS-only connections |
| `hostnossl` in `pg_hba.conf` | Rejects non-TLS connections |

---

## 📦 How to Generate Allowed Cipher Suite String

PostgreSQL uses OpenSSL-style cipher suite strings. Here's an example that enforces only secure ciphers:

```conf
ssl_ciphers = 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES256-GCM-SHA384'
```

To see what cipher suites your system supports:

```bash
openssl ciphers -v 'HIGH:!aNULL:!MD5:!RC4:!DH:!kRSA'
```

---

## 🧪 Test TLS Settings from Client

From a client machine:

```bash
psql "host=my-pg-server dbname=mydb user=myuser sslmode=require"
```

If the client does not support the required cipher suite or TLS version → connection fails with:

```
FATAL:  no pg_hba.conf entry for host "...", user "...", database "...", SSL off
```

Or:

```
SSL error: tlsv1 alert protocol version
```

---

## 🛡️ Optional: Enforce Mutual TLS (mTLS)

To require client certificates:

```conf
ssl = on
ssl_cert_file = '/etc/postgresql/certs/server.crt'
ssl_key_file = '/etc/postgresql/certs/server.key'
ssl_ca_file = '/etc/postgresql/certs/root.crt'
ssl_ciphers = 'HIGH:!aNULL'
ssl_min_protocol_version = 'TLSv1.2'
```

Update `pg_hba.conf`:

```conf
hostssl all             all             0.0.0.0/0               cert
```

Clients must then connect with their certificate:

```bash
psql "host=my-pg-server dbname=mydb user=myuser sslmode=verify-full sslcert=/path/to/client.crt sslkey=/path/to/client.key"
```

---

## 📊 Summary Table

| Feature | Supported in PostgreSQL? |
|--------|---------------------------|
| TLS 1.2+ | ✅ Yes (v10+) |
| Cipher suite restriction | ✅ Yes |
| Mutual TLS (mTLS) | ✅ Yes |
| Reject connection if no cipher match | ✅ Yes |
| Reject TLS < 1.2 | ✅ Yes |
| Syslog-compatible logs for failures | ✅ Yes (logs handshake failures) |

---

## ✅ Best Practice Recommendations

| Task | Description |
|------|-------------|
| Use PostgreSQL ≥ 10 | For better TLS support |
| Always enable TLS for remote access | Especially in production |
| Set `ssl_min_protocol_version` to TLSv1.2 | Prevent downgrade attacks |
| Set `ssl_ciphers` to allow only modern ones | ECDHE, CHACHA20, AES-GCM |
| Enable mTLS (optional) | For internal services |
| Monitor logs for failed handshakes | Detect misconfigurations early |

---

## 🛠 Need Help Generating a Full PostgreSQL TLS Config?

I can help you generate:

- A full PostgreSQL TLS config enforcing TLS 1.2+, modern cipher suites
- Kubernetes/OpenShift manifests for secure deployment
- Scripts to test TLS connectivity
- Helm values for PostgreSQL Operator deployments

Would you like a working example tailored to your deployment model (e.g., standalone, OCP, Crunchy Data)? Let me know!
