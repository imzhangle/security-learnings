Got it — **on-prem T24 Private Banking** changes the answer quite a bit 👍
In *that* setup, the **recommended and safest choice is: 👉 SAML 2.0**.

Let me explain why (no fluff, just real-world T24 reality).

---

## ✅ Recommendation: **SAML 2.0**

For **T24 R25 on-prem (Private Banking)**, **SAML is the de-facto standard** and what Temenos implementations actually expect in production.

### Why SAML wins for on-prem T24

#### 1️⃣ Native + proven support in T24

* T24 on-prem deployments **officially and commonly support SAML 2.0** for SSO
* Most Temenos reference architectures, Basecamp docs, and SI implementations use **SAML SP → IdP**
* OIDC support (if present) is usually:

  * Limited
  * UI-layer only
  * Not battle-tested in regulated banking environments

👉 If you ask Temenos or an SI, **they will almost always say SAML**.

---

#### 2️⃣ Better fit for private banking security models

Private banking environments usually require:

* Centralized authentication
* Strong MFA
* Conditional access
* Clear user-to-T24 mapping

**SAML handles this very cleanly**:

* Attribute-based identity mapping (USERNAME, COMPANY, ROLE, etc.)
* Certificate-based trust (no token secrets rotating unexpectedly)
* Works perfectly with **Entra ID Conditional Access + MFA**

---

#### 3️⃣ No internet-facing OAuth flows needed

OIDC works best when:

* Apps are internet-facing
* APIs are exposed
* Tokens are consumed by multiple services

Your setup is:

* On-prem
* Internal users
* Thick / browser-based T24 UI

👉 SAML avoids:

* OAuth token lifecycle complexity
* Refresh token storage
* API token validation logic on-prem

---

#### 4️⃣ Operational stability (this matters in banks)

SAML gives you:

* Long-lived signing certs
* Predictable flows
* Fewer moving parts

OIDC adds:

* Token expiry handling
* Clock skew issues
* More troubleshooting when something breaks at 2am 😅

---

## 🚫 When OIDC would make sense (and why it usually doesn’t here)

OIDC is **not wrong**, but it’s usually **not recommended** for **on-prem T24** unless:

* Temenos explicitly confirmed OIDC support in *your exact R25 build*
* You are integrating T24 APIs with cloud-native services
* You have a Temenos Identity Manager or modern digital layer in front of T24

If none of those are true → **don’t pick OIDC just because it’s “modern”**.

---

## 🧠 Final decision table (for your case)

| Factor              | Your Environment | Best Choice |
| ------------------- | ---------------- | ----------- |
| T24 R25             | On-prem          | **SAML**    |
| Banking / regulated | Yes              | **SAML**    |
| Private Banking     | Yes              | **SAML**    |
| Entra ID as IdP     | Yes              | **SAML**    |
| Internal users      | Yes              | **SAML**    |
| API / microservices | No               | **SAML**    |

---

## 🔧 Typical architecture (what most banks do)

```
User Browser
   ↓
Microsoft Entra ID (IdP)
   ↓  SAML Assertion
T24 Web / Browser Login (SP)
   ↓
T24 User Session
```

* Entra ID handles:

  * MFA
  * Conditional Access
  * Password policy
* T24 handles:

  * Authorization
  * Roles
  * Company / application mapping

---

## 💡 Practical advice

If you want the **least risk**, **best vendor support**, and **fastest sign-off from security & audit**:

👉 **Choose SAML 2.0**

If you want, I can:

* Walk you through **exact Entra ID SAML settings**
* Help map **Entra attributes → T24 user fields**
* Share **common mistakes banks hit during T24 SAML SSO**

Just say the word.
