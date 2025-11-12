In Temenos Transact (T24) R11, the SAML authentication layer—typically handled through the T24 Browser (on JBoss or similar) or integrated with Temenos Fabric/API Gateway—doesn't natively perform dynamic consultations of a user mapping table directly within the core SAML filter. Instead, it relies on configurable attribute mappings and extensible hooks (e.g., custom routines, identity services, or post-authentication processing) to handle variable external identifiers like local nicknames, emails, or sub IDs from Entra ID. This allows translation to a local T24 USER record ID without requiring passwords. Below, I'll elaborate on the flow, how variability in identifiers is managed, and the role of database consultations, based on standard Temenos SSO patterns and customization options.

### SAML Authentication Flow Overview with Mapping
The process follows standard SAML 2.0 SP-initiated or IdP-initiated flows, but with Temenos-specific extensions for banking security/compliance:

1. **Initiation and Assertion Receipt**:
   - User attempts login via T24 Browser/UI, redirecting to Entra ID (IdP) for authentication.
   - Entra ID authenticates (e.g., via email/password or MFA) and returns a SAML assertion (XML) to the T24 SP endpoint (e.g., `/saml/SSO` in Fabric or `/j_spring_security_check` in Browser).
   - The assertion includes claims/attributes configured in Entra ID, such as:
     - `NameID`: Often the primary identifier (e.g., email format like `user@example.com` or persistent sub ID).
     - Custom attributes: e.g., `email`, `nickname`, `sub`, `objectId`, or groups/roles.
   - Variability: If users authenticate with emails sometimes and nicknames others, Entra ID must be configured to emit consistent or multiple attributes (e.g., always include both if available). T24 doesn't dictate this; it's IdP-side setup.

2. **Validation and Attribute Extraction**:
   - T24's SAML filter (e.g., Spring Security SAML in Browser WAR or Fabric's SAML provider) validates the assertion (signature, audience, expiry).
   - Extracts attributes into a security context/principal. Standard mapping ties `NameID` directly to T24's USER.ID if identical.
   - For variability: If `NameID` isn't reliable (e.g., email vs. nickname), configure attribute mappings in the SAML config file (e.g., `saml-servlet.xml` or Fabric UI) to pull from alternative claims:
     | External Attribute (from Assertion) | Local Mapping Target | Handling Variability |
     |-------------------------------------|----------------------|----------------------|
     | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier` (sub ID) | Principal/User ID | Use if present; fallback to email if null. |
     | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` (email) | USER.EMAIL or custom field | Match if nickname absent. |
     | Custom (e.g., `nickname` or `preferred_username`) | USER.EXTERNAL.ID (LOCAL.REF) | For local nicknames; prioritize based on routine logic. |

     - This is set in Fabric's Identity Service > Mapping of IDP SAML Attributes (as in my prior response) or Browser's SAML beans.

3. **Custom Mapping and Database Consultation**:
   - **When Direct Mapping Isn't Enough**: If identifiers vary (e.g., email for some users, nickname for others), the SAML layer invokes a custom extension to consult a mapping table. This isn't automatic—it's implemented via:
     - **Custom Identity Service in Fabric**: Preferred for R11+ integrations. In the service config (Identity > Custom or SAML type), define a **Post Authentication URL** or **User Attributes Endpoint** that points to a backend orchestration/integration service. This service:
       - Receives extracted attributes (e.g., as JSON: `{"sub": "abc123", "email": "user@example.com", "nickname": "userNick"}`).
       - Determines the identifier type (e.g., via if-else in Groovy/script: if email present, use it; else nickname).
       - Queries the T24 database for mapping.
     - **Routine Hook in T24 Core**: For non-Fabric setups, attach a jBASE/BASIC routine to the authentication process (e.g., via USER application's AUTH.ROUTINE field or Browser's JAAS custom module). The routine runs post-assertion validation.
     - **Timing**: Consultation happens immediately after attribute extraction, before session creation, to avoid unauthorized access.
   - **Database Table Consultation**:
     - Use a custom T24 application/table like `USER.MAPPING` (created via T24's application designer with fields: ID, EXTERNAL.TYPE, EXTERNAL.VALUE, LOCAL.USER.ID, STATUS, ROLES).
       - EXTERNAL.TYPE: Enum-like (e.g., 'SUB', 'EMAIL', 'NICKNAME') to handle variability.
       - EXTERNAL.VALUE: The actual identifier (e.g., 'user@example.com' or 'userNick').
       - LOCAL.USER.ID: Maps to T24's USER record (e.g., 'BANKUSER01').
     - How it's consulted:
       - The custom service/routine issues a T24 enquiry or direct read (e.g., via `READ` command in jBASE or REST API call to T24).
       - Example jBASE pseudocode in a routine (attached to post-auth hook):
         ```
         SUBROUTINE RESOLVE.USER.MAPPING(ATTRIBUTES, LOCAL.USER, ERR)
         * ATTRIBUTES is parsed assertion attrs (e.g., from SAML context)
         IF ATTRIBUTES<EMAIL> NE '' THEN
           SEL = 'SELECT USER.MAPPING WITH EXTERNAL.TYPE EQ "EMAIL" AND EXTERNAL.VALUE EQ ' : ATTRIBUTES<EMAIL>
         END ELSE IF ATTRIBUTES<NICKNAME> NE '' THEN
           SEL = 'SELECT USER.MAPPING WITH EXTERNAL.TYPE EQ "NICKNAME" AND EXTERNAL.VALUE EQ ' : ATTRIBUTES<NICKNAME>
         END ELSE IF ATTRIBUTES<SUB> NE '' THEN
           SEL = 'SELECT USER.MAPPING WITH EXTERNAL.TYPE EQ "SUB" AND EXTERNAL.VALUE EQ ' : ATTRIBUTES<SUB>
         END ELSE
           ERR = 'No valid identifier'
           RETURN
         END
         EXECUTE SEL CAPTURING OUTPUT RETURNING IDS
         IF IDS THEN
           READ REC FROM 'USER.MAPPING', IDS<1> ELSE ERR = 'No mapping'
           LOCAL.USER = REC<LOCAL.USER.POS>  ;* e.g., field 3
         END
         END
         ```
       - Database access: Via T24's jBASE (hashed files) or relational DB if migrated (e.g., Oracle/SQL). The routine connects internally—no external DB calls needed.
       - If no match: Fall back to just-in-time provisioning (create new USER record) or error/redirect.
     - In Fabric: The integration service (Groovy/Java) calls T24 via middleware (e.g., `invokeT24Enquiry("USER.MAPPING.LOOKUP", params)`), passing the selected identifier.

4. **Authorization and Session Setup**:
   - Once mapped, set the security context (e.g., `x-kony-user-id` header in Fabric) to the local USER.ID.
   - Apply roles/groups from assertion (mapped to T24's USER.SMS.GROUP or profiles).
   - Log the consultation in T24's audit logs (SECURITY.LOG) for compliance.

### Handling Edge Cases for Variable Identifiers
- **Prioritization**: Configure the IdP to always send a primary identifier, or use routine logic to prioritize (e.g., sub > email > nickname).
- **Multiple Matches**: Routine can enforce uniqueness or log conflicts.
- **Updates**: If identifier changes (e.g., email update), sync via SCIM provisioning (Entra ID supports; integrate with T24's user API) or manual mapping updates.
- **Security**: All DB consultations are internal/authenticated; use encrypted assertions and validate issuers.

This setup ensures flexibility without modifying core SAML code. For exact R11 configs, refer to Temenos docs like "T24 Browser SAML Setup" or Fabric guides (V8.x for R11 compatibility). If you share config snippets or errors, I can advise more precisely.
