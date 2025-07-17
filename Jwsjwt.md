You're absolutely right — Gravitee's naming for these policies can be **very confusing**. Let's break down the **difference between the "JSON Web Tokens" policy and the "JSON Web Signature" policy**, clearly and directly:

---

### 🔐 **1. JSON Web Tokens (JWT) Policy**

**Purpose:**
Validates a **JWT token** — typically in the Authorization header (Bearer token) — to authenticate or authorize a request.

**What it does:**

* Parses the JWT.
* Verifies the **signature** using a secret or a public key (depending on the algorithm).
* Optionally checks:

  * Expiration (`exp`)
  * Issuer (`iss`)
  * Audience (`aud`)
  * Custom claims.
* Can extract claims and **inject them into headers**, context variables, etc.

✅ **Use this when:**

* You want to **validate an incoming JWT token** (from a user or app).
* Common in OAuth2/OpenID Connect scenarios.

---

### 🧾 **JSON Web Signature (JWS) Policy**

**Purpose:**
Verifies the **signature** of a JWS (JSON Web Signature) object, which **may or may not be a JWT**.

**What it does:**

* Takes a JWS string (e.g., from a header or body).
* Verifies its signature using a key or JWKS.
* Outputs verification status (valid/invalid).

**Key difference:**
While JWTs are a type of JWS, this policy is **lower-level**. It doesn’t care about token claims or standard JWT validation steps.

✅ **Use this when:**

* You want to **verify a raw JWS** signature, **not necessarily a JWT**.
* You’re working with **non-auth use cases**, e.g., validating message integrity, webhook signatures, or signed payloads.

---

### 🤔 So why both?

| Feature                   | **JWT Policy**              | **JWS Policy**                       |
| ------------------------- | --------------------------- | ------------------------------------ |
| Validates signature       | ✅ Yes                       | ✅ Yes                                |
| Parses claims             | ✅ Yes                       | ❌ No                                 |
| Checks expiration, issuer | ✅ Yes                       | ❌ No                                 |
| Handles Bearer tokens     | ✅ Yes (default use case)    | ❌ No (you must extract JWS manually) |
| Use case                  | Auth (OAuth2, OpenID, etc.) | Signed payload verification          |

---

### 🧠 TL;DR

* **JWT Policy = Auth-centric, high-level JWT validation**.
* **JWS Policy = Signature verification only**, more generic.

Let me know if you want an example configuration of either!
