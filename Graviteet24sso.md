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
