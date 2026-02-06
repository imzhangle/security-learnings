**No, challenging (binding/authenticating against) LDAP for literally every single authentication request is usually not the best long-term approach** — especially in a core banking system like Temenos T24 (now Transact), where login volume, performance, resilience, and security all matter.

Here’s a breakdown of why, and what is more commonly considered good practice:

### Why "LDAP bind on every login" can be problematic
- **Performance & latency** — Each login causes a network round-trip + bind operation on the LDAP/AD server. In high-concurrency environments (many tellers / branch users logging in during peak morning hours), this adds noticeable latency and load on the LDAP infrastructure.
- **Availability dependency** — T24 login becomes completely dependent on LDAP server reachability and performance. If LDAP is slow, down, or network partitioned → nobody can log into T24.
- **Scalability limits** — LDAP is optimized for reads, but frequent bind operations still create more load than necessary.

### Recommended / common patterns in Temenos T24 environments
Modern Temenos setups (especially Browser / temenos explorer / Transact with UXP / T24 Transact) strongly favor these approaches instead of direct per-login LDAP bind from T24:

1. **Use an Identity Provider / SSO layer (most recommended today)**  
   - Keycloak (Temenos officially supports / recommends this in many recent architectures)  
   - Other IAM solutions (Okta, Azure AD, Ping, ForgeRock, etc.)  
   - Flow: User → Keycloak / IdP → LDAP/AD bind happens only once → IdP issues JWT / OAuth2 / OIDC token → T24 validates the token (no LDAP call on every T24 login)  
   → **Result**: LDAP bind happens only at IdP login → subsequent T24 authentications are token-based (fast, no LDAP dependency).

2. **Application server / container-level LDAP / Kerberos SSO** (very common in older/mid setups)  
   - JBoss EAP / WildFly configured with LDAP security domain or Kerberos (SPNEGO)  
   - Browser SSO with Kerberos ticket → no password re-entry and no per-request LDAP bind from T24 itself  
   - Or LDAP JAAS module with connection pooling + caching at the app server level

3. **T24-level LDAP with caching / connection pooling (if you must stay with direct integration)**  
   - Many T24 implementations use Temenos Connector or OFS authentication modules configured against LDAP.  
   - These usually support connection pooling (keep-alive LDAP connections) and sometimes simple result caching.  
   - Still not ideal to do full bind+search every time — better if the product caches credentials hashes or uses "pass-through" mode with pooled binds.

### Quick decision table

| Scenario                              | Challenge LDAP every time? | Recommendation                  | Why |
|---------------------------------------|-----------------------------|---------------------------------|-----|
| Very low user count (<50)             | Acceptable                  | OK to do direct bind            | Low load |
| Normal branch / teller environment    | No                          | IdP (Keycloak) or app-server SSO| Performance + resilience |
| High-frequency / API / microservices  | No                          | Token-based (OIDC/JWT)          | Must be fast |
| Strict security (zero caching allowed)| Yes (but rare)              | Direct bind + LDAPS + monitoring| Compliance edge case |
| Already suffering slow logins         | Almost certainly no         | Move to SSO layer               | Immediate improvement |

### Bottom line (2026 perspective)
Direct "challenge LDAP on every single T24 authentication" is **not good practice** in most production Temenos environments anymore.

The industry (and Temenos reference architectures) moved toward:
- **Keycloak + OIDC/JWT** (preferred modern way)
- **Kerberos SSO** (great for Windows/AD heavy banks)
- **App-server pooled LDAP** (acceptable interim solution)

These reduce LDAP bind frequency dramatically while still getting full password policy enforcement, account lockouts, password expiry, etc. from AD/LDAP.

If your current setup is already direct bind-per-login and you're not seeing performance issues → it might be "good enough" short-term. But plan to evolve toward an IdP/SSO model — especially if you're on recent T24 Transact / Explorer releases.

If you can share more context (T24 version, using Browser/Classic/Explorer/UXP?, AD or OpenLDAP?, any existing SSO/IAM product?), I can give more targeted advice.
