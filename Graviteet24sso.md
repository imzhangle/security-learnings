Let me search more specifically for T24 Transact and OIDC support:Based on my search results and knowledge of T24 Transact, I can provide you with information about OIDC support:

## T24 Transact OIDC Support

**Yes, T24 Transact does support OIDC (OpenID Connect)**, but the implementation and availability depend on the specific version and configuration:

### Current OIDC Support Status:

**Supported in Recent Versions:**
- T24 R18 and later versions have enhanced support for OIDC
- Temenos documentation references OIDC integration capabilities
- Modern T24 implementations can integrate with OIDC providers

### OIDC Integration with Gravitee AM

Since you're using Gravitee AM, this is actually a more flexible approach than SAML:

**Advantages of OIDC over SAML:**
- **Lightweight**: JSON-based tokens vs XML assertions
- **Modern Protocol**: Built on OAuth 2.0 framework
- **Better API Integration**: Native support for REST APIs
- **Mobile-Friendly**: Better suited for mobile applications

### Configuration Approach

**T24 → Gravitee AM (OIDC) → Active Directory:**

1. **Gravitee AM as OIDC Provider**:
   ```yaml
   # Configure OIDC endpoints in Gravitee AM
   authorization_endpoint: https://gravitee-am.domain.com/oauth/authorize
   token_endpoint: https://gravitee-am.domain.com/oauth/token  
   userinfo_endpoint: https://gravitee-am.domain.com/oidc/userinfo
   jwks_uri: https://gravitee-am.domain.com/.well-known/jwks
   ```

2. **T24 OIDC Client Configuration**:
   ```json
   {
     "client_id": "t24-transact-client",
     "client_secret": "your-secret",
     "redirect_uri": "https://t24.domain.com/oidc/callback",
     "response_type": "code",
     "scope": "openid profile email groups"
   }
   ```

### Authentication Flow

1. User accesses T24 Transact
2. T24 redirects to Gravitee AM authorization endpoint
3. Gravitee AM authenticates against Active Directory
4. Authorization code returned to T24
5. T24 exchanges code for ID token and access token
6. User information retrieved from userinfo endpoint

### Implementation Considerations

**For T24 Integration:**
- Verify your T24 version supports OIDC (R18+)
- Configure OIDC client settings in T24 security configuration
- Map OIDC claims to T24 user attributes
- Handle token refresh and logout flows

**Gravitee AM Configuration:**
- Set up AD as user provider
- Configure OIDC application for T24
- Map AD groups to OIDC scopes/claims
- Set appropriate token lifetimes

### Recommendation

Given that you're already using Gravitee AM, **OIDC would be the preferred approach** over SAML for T24 integration because:

- More native support in modern T24 versions
- Better integration with Gravitee AM
- Simpler token handling
- Better suited for API-based interactions

If your T24 version doesn't support OIDC natively, you may need to use the SAML approach I described earlier, or consider upgrading to a more recent T24 version that includes enhanced OIDC support.

Would you like me to provide more specific configuration details for the OIDC integration between T24 and Gravitee AM?



Here's a **PlantUML** diagram that visually represents the **SAML SSO flow between T24 Transact, Gravitee AM, and Active Directory**, including the authentication and federation steps.

You can copy and paste this into any PlantUML editor (like [PlantUML Online](https://www.plantuml.com/plantuml/)) to generate the diagram.

---

### ✅ PlantUML Sequence Diagram: SAML SSO Flow

```plantuml
@startuml
title SAML SSO Flow: T24 Transact + Gravitee AM + Active Directory

actor User
participant "T24 Transact\n(Service Provider)" as T24
participant "Gravitee AM\n(Identity Provider)" as Gravitee
participant "Active Directory\n(LDAP)" as AD

User -> T24: 1. Access T24 Web URL
activate T24

T24 -> Gravitee: 2. Redirect with SAML AuthnRequest
activate Gravitee

Gravitee -> User: 3. Show Login Page
deactivate Gravitee

User -> Gravitee: 4. Submit AD Credentials
activate Gravitee

Gravitee -> AD: 5. Validate via LDAP Bind\n(e.g., sAMAccountName + password)
activate AD

AD --> Gravitee: 6. Authentication Success
deactivate AD

Gravitee -> Gravitee: 7. Generate SAML Assertion\n(NameID, Attributes)
note right
  Includes:
  - NameID: user@domain.com
  - Attributes: email, firstname, lastname
  - Signed with IdP cert
end note

Gravitee --> User: 8. POST SAML Response to T24 ACS URL
deactivate Gravitee

User -> T24: 9. Deliver SAML Response to ACS
activate T24

T24 -> T24: 10. Validate SAML:
  - Signature
  - Audience
  - Not expired
  - Map user attributes

T24 --> User: 11. Establish Session\nLogin Successful

deactivate T24

User --> T24: 12. Access Protected Resource
activate T24
T24 --> User: 13. Serve T24 Dashboard
deactivate T24

@enduml
```

---

### 🖼️ What This Diagram Shows

- **Step-by-step SAML Web Browser SSO flow** using **Redirect/POST binding**.
- Gravitee AM acts as **IdP**, authenticating users against **Active Directory via LDAP**.
- T24 acts as **SP**, consuming the SAML assertion to log in the user.
- Highlights key security steps: SAML validation, LDAP bind, attribute mapping.

---

### 🔧 Optional: Deployment Diagram (Infrastructure View)

```plantuml
@startuml
package "Internal Network" {
  [Active Directory\n(LDAP Server)] as AD
  [Gravitee AM\nIdentity Provider] as Gravitee
}

node "DMZ" {
  [T24 Transact\nWeb Host / SP] as T24
}

User --> T24: HTTPS
T24 --> Gravitee: SAML AuthnRequest (HTTPS)
Gravitee --> AD: LDAP Bind (Port 389/636)
note right of Gravitee
  Gravitee AM federates with AD
  to authenticate users
end note

cloud "Internet" {
  User
}
@enduml
```

---

### 📌 Tips

- Use **HTTPS everywhere** (especially `/saml` endpoints).
- Ensure **clocks are synchronized** (SAML is time-sensitive).
- Export and import **metadata XML** for trust setup.

Let me know if you'd like:
- A downloadable PNG/SVG
- Metadata XML example
- Attribute mapping configuration for T24
- Integration with Azure AD instead of on-prem AD

I'm happy to help!




You're absolutely right — and thank you for calling that out.

I did **not mean to omit or dismiss Gravitee AM** from the architecture. In fact, **Gravitee AM plays a critical role** in many enterprise integration scenarios, including **authentication orchestration, API security, reverse proxying, and SAML/OIDC bridging**.

However, in my previous response, I **prioritized a working MFA-compatible flow**, and in many real-world cases, **direct federation between T24 and Entra ID (Azure AD)** is the most reliable way to support **Entra ID’s MFA and Conditional Access policies** — especially when Gravitee AM is used only as an **intermediate IdP** with **non-interactive flows** that break under MFA enforcement.

But here’s the good news: **Gravitee AM can and should be included** — just **not as a backchannel authenticator**, but as a **smart identity broker or reverse proxy** that **respects modern authentication flows**.

Let me now give you the **correct, complete architecture** that **includes Gravitee AM**, supports **Entra ID with MFA enabled**, and still allows **secure access to T24 Transact**.

---

## ✅ Correct Architecture: **Gravitee AM + Entra ID (with MFA) + T24**

```
User → T24 → Gravitee AM (as Reverse Proxy / OIDC Agent) → Entra ID → MFA → Back to Gravitee AM → SAML/Header to T24
```

### 🔧 Role of Gravitee AM
Gravitee AM acts as:
- ✅ **Reverse Proxy / API Gateway** in front of T24
- ✅ **OIDC Relying Party** to Entra ID (not SAML IdP)
- ✅ **Session Manager** (creates secure session after MFA)
- ✅ **SAML Assertion Generator** (optional: "SAML Bridging")
- ✅ **Header Enricher** (injects user identity into T24)

This way, **MFA is fully supported**, and **Gravitee AM remains in the loop** for security, logging, and policy enforcement.

---

## 🏗️ Full Architecture Diagram (Textual)

```text
+------------------+
|    User (Browser) |
+------------------+
         │
         ▼
+-----------------------------+
|   T24 Web Host (Public URL) |
|   (e.g., https://t24bank.com)|
+-----------------------------+
         │
         ▼
+--------------------------------------------------+
|            Gravitee AM (Reverse Proxy)           |
|  - Protects /webbank route                       |
|  - Redirects unauthenticated users to Entra ID   |
|  - Acts as OIDC RP (not IdP)                     |
|  - Handles callback from Entra ID                |
|  - Creates session after MFA success             |
+--------------------------------------------------+
         │
         ▼
+--------------------------------------------------+
|             Microsoft Entra ID (IdP)             |
|  - Presents MFA challenge (Authenticator, SMS)   |
|  - Validates Conditional Access policies         |
|  - Returns ID Token to Gravitee AM callback      |
+--------------------------------------------------+
         ▲
         │ (OIDC Redirect Flow)
         └──────────────────────────────────────────┐
                                                    │
+--------------------------------------------------+
|                 T24 Transact (Internal)          |
|  - Receives:                                     |
|    Option A: SAML Assertion from Gravitee AM     |
|    Option B: Trusted headers (X-User-ID, etc.)   |
|  - Trusts Gravitee AM as secure gateway          |
+--------------------------------------------------+
```

---

## ✅ How It Works (Step-by-Step)

| Step | Component | Action |
|------|---------|--------|
| 1 | User | Accesses `https://t24bank.com/webbank` |
| 2 | Gravitee AM | Detects no session → redirects to **Entra ID via OIDC** (`/authorize`) |
| 3 | Entra ID | Shows login screen → enforces **MFA** (push, SMS, etc.) |
| 4 | User | Completes MFA in browser |
| 5 | Entra ID | Sends **ID Token + Access Token** to Gravitee AM’s callback URL |
| 6 | Gravitee AM | Validates token, creates **local session cookie**, maps user |
| 7 | Gravitee AM → T24 | Two options: |
|   | Option A (SAML Bridging) | Generates **SAML assertion** and POSTs to T24’s ACS URL |
|   | Option B (Header Injection) | Forwards request to internal T24 with secure headers:<br>`X-Forwarded-User: john@bank.com`<br>`X-Forwarded-Groups: T24-Users` |
| 8 | T24 | Accepts user based on SAML or trusted header → logs in |

> ✅ MFA is honored because the user **interacts directly with Entra ID**.  
> ✅ Gravitee AM is **fully in the flow** — for security, not just routing.

---

## 🔐 Why This Works with MFA

| Requirement | How It’s Met |
|-----------|--------------|
| MFA Challenge | Rendered by Entra ID in browser |
| Conditional Access | Enforced by Entra ID |
| No Legacy Auth | OIDC (modern auth) used |
| Gravitee AM Involvement | As OIDC RP + session manager |
| T24 Integration | Via SAML or trusted headers |

---

## 🛠️ Configuration: Gravitee AM as OIDC Relying Party

### 1. In Gravitee AM
- Go to **Domain > Identity Providers > Add OIDC**
- Configure:
  - **Issuer**: `https://login.microsoftonline.com/{tenant-id}/v2.0`
  - **Authorization / Token / JWK URLs** (auto-discovered)
  - **Client ID / Secret**: From Entra ID App Registration
  - **Scope**: `openid profile email`
  - **Response Type**: `code`
  - **Callback URL**: `https://gateway.bank.com/callback` (must match Entra ID reply URL)

### 2. In Entra ID (Azure Portal)
- Register a new app:
  - Name: `Gravitee AM OIDC Bridge`
  - Redirect URI: `https://gateway.bank.com/domains/{domain}/oauth2/callback`
- Under **Token Configuration**, add optional claims:
  - `email`, `groups`, `given_name`, `surname`
- Configure **Conditional Access** with MFA

### 3. In T24
#### Option A: SAML Bridging
- Configure T24 to trust **Gravitee AM as SAML IdP**
- Gravitee AM generates SAML assertion after OIDC login

#### Option B: Trusted Headers (Simpler)
- Configure T24 to accept:
  - `X-Forwarded-User` as username
  - `X-Forwarded-Email`, `X-Forwarded-FirstName`, etc.
- Ensure T24 **only accepts these headers from Gravitee AM IP** (whitelist)

---

## ✅ Benefits of This Design

| Benefit | Description |
|-------|-------------|
| ✅ MFA Support | Full Entra ID MFA experience |
| ✅ Gravitee AM In Control | For security, logging, rate limiting |
| ✅ No Legacy Auth | Compliant with Microsoft security baseline |
| ✅ Flexible | Supports SAML or header-based integration |
| ✅ Auditable | Full trace in Gravitee AM logs |

---

## 🚫 When NOT to Use This

- If T24 **cannot accept SAML assertions from Gravitee AM**
- If T24 **does not support trusted header authentication**
- If network policies block reverse proxy patterns

→ Then, **direct SAML from T24 to Entra ID** is better.

---

## ✅ Summary

| You Said | Truth | Correct Approach |
|--------|-------|------------------|
| “You omitted Gravitee AM” | Yes, temporarily | Gravitee AM should be the **reverse proxy and OIDC agent**, not a backchannel IdP |
| “Entra ID has MFA” | Blocks silent auth | Use **interactive OIDC flow** to Entra ID |
| “T24 needs SSO” | Yes | Use **SAML bridging** or **trusted headers** from Gravitee AM |

---

## 📥 Next Steps

Let me know which option you prefer:
1. **Gravitee AM as OIDC-to-SAML Bridge** → I’ll give you a **SAML assertion template**
2. **Gravitee AM with Header Injection** → I’ll give you **header config for T24**
3. **PlantUML or draw.io diagram** with all components
4. **Sample Entra ID App Registration JSON**

I’ll help you implement a **secure, MFA-compliant, Gravitee AM-integrated** solution.

You’re right — **Gravitee AM belongs in the architecture**, and now it’s back — and better than before. 🛡️
