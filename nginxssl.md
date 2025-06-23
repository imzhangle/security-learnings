To configure **NGINX** to meet the following **security requirements**:

| Requirement | Desired Setting |
|------------|------------------|
| Minimum TLS version | TLS 1.2 |
| Cipher suites | AES-256-GCM and forward secrecy (ECDHE) |
| Forward Secrecy | ✅ Enforced via ECDHE key exchange |
| Modern, secure defaults | ✅ Yes |

You'll need to:
1. Use a **strong SSL/TLS configuration**
2. Define a set of **secure cipher suites**
3. Enable only **TLS 1.2 and optionally TLS 1.3**
4. Ensure **Perfect Forward Secrecy (PFS)** is used

---

## ✅ NGINX Configuration Example

Here’s a **secure NGINX config snippet** that enforces:

- TLS 1.2 minimum
- Strong cipher suites including `AES-256-GCM`
- Forward secrecy via ECDHE
- Secure protocols and headers

```nginx
server {
    listen 443 ssl http2;
    server_name yourdomain.com;

    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # TLS Protocols
    ssl_protocols TLSv1.2 TLSv1.3;

    # Cipher Suites with Forward Secrecy (ECDHE-first)
    ssl_ciphers HIGH:!aNULL:!MD5:ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:!3DES:!DSS;

    # Prefer server ciphers
    ssl_prefer_server_ciphers on;

    # Session cache & timeout
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 valid=300s;
    resolver_timeout 5s;

    # HSTS (HTTP Strict Transport Security)
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;

    # Disable insecure renegotiation
    ssl_insecure_redirect off;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 🔐 Explanation of Key Directives

### `ssl_protocols TLSv1.2 TLSv1.3;`
- Enforces minimum TLS 1.2
- Includes optional support for faster, more secure TLS 1.3

### `ssl_ciphers HIGH:!aNULL:!MD5:ECDH+AESGCM:DH+AESGCM:ECDH+AES256...`
- Prioritizes ECDHE-based cipher suites for **forward secrecy**
- Uses strong encryption like **AES-256-GCM**
- Disables weak or outdated algorithms (`!MD5`, `!aNULL`, `!3DES`)

### `ssl_prefer_server_ciphers on;`
- Ensures NGINX chooses the strongest cipher suite available

### `add_header Strict-Transport-Security ...`
- Tells browsers to always use HTTPS for this domain

### `ssl_stapling on; ssl_stapling_verify on;`
- Enables OCSP stapling for better performance and revocation checks

---

## 🧪 Optional: Test Your TLS Configuration

Use tools like:
- [SSL Labs - SSL Test](https://www.ssllabs.com/ssltest/)
- `openssl s_client -connect yourdomain.com:443`

Look for:
- Protocol: TLS 1.2 or TLS 1.3
- Cipher Suite: ECDHE-RSA-AES256-GCM-SHA384 or similar
- Forward Secrecy: YES

---

## 📦 Certificates Management Tips

- Use **Let's Encrypt**, **AWS ACM**, or another trusted CA
- Store keys securely — consider using:
  - AWS Secrets Manager
  - HashiCorp Vault
  - Kubernetes secrets (if in K8s)

Example command to generate a strong DH group (for DHE cipher suites):

```bash
openssl dhparam -out /etc/nginx/dhparams.pem 4096
```

Then reference it in NGINX:

```nginx
ssl_dhparam /etc/nginx/dhparams.pem;
```

> Note: ECDHE is preferred over DHE due to better performance.

---

## 🛡️ Want Mutual TLS (mTLS)?

If you want to enforce **client certificate authentication**, add:

```nginx
ssl_verify_client on;
ssl_client_certificate /etc/nginx/ssl/ca.crt;
```

This requires clients to present a valid certificate signed by the specified CA.

---

## 🧱 Deployment Tips

### Dockerfile Snippet
If deploying NGINX in a container:

```Dockerfile
FROM nginx:latest
COPY ./ssl:/etc/nginx/ssl
COPY ./nginx.conf:/etc/nginx/conf.d/default.conf
```

Make sure to mount certs securely in production.

---

## ✅ Summary Table

| Feature | NGINX Config |
|--------|--------------|
| Minimum TLS Version | `ssl_protocols TLSv1.2 TLSv1.3;` |
| AES-256-GCM Ciphers | Included in `ssl_ciphers` list |
| Forward Secrecy | ✅ Via ECDHE key exchange |
| Fully customizable | ✅ Yes |
| Managed alternative | ❌ No – must be manually maintained |
| Compliance-ready | ✅ Yes, with correct settings |

---

Would you like:
- A **complete Docker image with TLS configured**?
- An **Ansible playbook** to deploy this config?
- Help integrating with **ACM or Let's Encrypt**?

Let me know and I’ll provide a full working setup tailored to your environment!
