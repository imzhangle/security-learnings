Perfect question 🔑 — this is where people get tripped up, because **T24 itself doesn’t implement SSL/TLS** at the application layer.
It relies on the **middleware layer** it runs on (e.g., **TAFJ = Java stack, TAFC = C stack**) and the **JMS/HTTP server** or **JVM** to enforce TLS protocols and cipher suites.

Let me break it down:

---

## 🔹 1. Which layer controls TLS in T24?

* **OFS over HTTP (via TCServer / JBoss / WebSphere / Tomcat)**
  → TLS is handled by the **application server (Java container)**.
  You configure **keystore, truststore, protocol, cipher suites** in the server (`server.xml`, `standalone.xml`, `security.xml`, depending on the container).
  T24 just inherits it.

* **OFS over JMS (via IBM MQ, ActiveMQ, WebLogic JMS, etc.)**
  → TLS is handled by the **JMS provider (broker)**.
  You configure the **cipher suite and TLS version on the MQ channel** (server side). On the T24 side, you configure the **MQ client/JNDI properties** to use SSL/TLS.

* **OFS over raw sockets (legacy)**
  → If secured, usually wrapped in **stunnel** or at the **network layer** — not native in T24.

---

## 🔹 2. How to enforce Cipher Suites / TLS versions

### For **Java-based TAFJ (JMS or HTTP APIs)**

* T24 runs in a JVM, so you control security through JVM options.

Example (`setenv.sh` or JVM args in `standalone.conf`):

```bash
-Dhttps.protocols=TLSv1.2,TLSv1.3
-Djdk.tls.client.protocols=TLSv1.2
-Dhttps.cipherSuites=TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
```

If using **ActiveMQ** or **IBM MQ JMS**:

```properties
javax.net.ssl.keyStore=/opt/t24/ssl/client.jks
javax.net.ssl.keyStorePassword=changeit
javax.net.ssl.trustStore=/opt/t24/ssl/truststore.jks
javax.net.ssl.trustStorePassword=changeit
```

And if using IBM MQ:

```properties
com.ibm.mq.cfg.useIBMCipherMappings=false
com.ibm.mq.cfg.SSLCipherSuite=TLS_RSA_WITH_AES_256_CBC_SHA256
com.ibm.mq.cfg.SSLFipsRequired=true
```

### For **TAFC (C-based stack)**

* TLS isn’t native. You’d usually put it behind:

  * **stunnel** (wraps T24’s TCP in TLS), or
  * **API Gateway / Load Balancer** that enforces TLS policies.

---

## 🔹 3. Where Cipher Suites are actually respected

* **HTTP/S endpoints** → defined by the app server config (Tomcat, JBoss, WebSphere).
* **JMS (MQ, ActiveMQ, WebLogic)** → defined by **queue manager/broker TLS settings**.
* **JVM-based clients (TAFJ)** → JVM-level security properties must match broker/server requirements.

---

✅ **Summary:**
You don’t configure cipher suites *inside* T24 itself.
You configure them in:

1. **Application server** (for HTTP/S).
2. **JMS provider (MQ, ActiveMQ, etc.)** (for OFS over JMS).
3. **JVM options / JNDI properties** (to force TLS version/cipher used by T24 when acting as client).

---

👉 Do you want me to give you a **step-by-step example for IBM MQ + T24 JMS adapter with TLS 1.2 enforced**? (That’s the most common real-world setup.)
