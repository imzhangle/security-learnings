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
