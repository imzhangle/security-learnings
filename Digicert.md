When evaluating **DigiCert** as a provider for **Qualified Digital Signatures (QES)** under the **eIDAS Regulation**, particularly for **Santander Private Bank customers**, your security team needs to ensure that DigiCert meets not only compliance and cryptographic standards, but also integrates securely into Santander’s existing infrastructure. Below is a structured response addressing each of the 8 points you've raised:

---

### **1. SSO for Employees**

- **Evaluation Focus**: Ensure that DigiCert supports secure **Single Sign-On (SSO)** options such as **SAML 2.0**, **OAuth 2.0**, or **OpenID Connect**, which can integrate with Santander's Identity Provider (IdP), such as Microsoft Azure AD or another enterprise IdP.
- **Recommendations**:
  - Validate that DigiCert allows federated authentication for administrative access.
  - Confirm support for **MFA** in conjunction with SSO.
  - Review session management and token expiration policies.

---

### **2. User Certification**

- **Evaluation Focus**: DigiCert should offer **qualified certificates for electronic signatures (QES)** compliant with **eIDAS Article 3**, issued by a **Trusted Third Party (TTP)** under strict identity verification procedures.
- **Key Checks**:
  - Does DigiCert provide **Qualified Website Authentication Certificates (QWACs)** and **Qualified Electronic Signature Certificates (QESCs)**?
  - Are they listed in the **EU Trusted List** via the eIDAS Network?
  - Is there a documented and auditable process for user identity verification prior to issuing QES certificates?

---

### **3. Third-Party Risk Assessment**

- **Evaluation Focus**: Assess DigiCert as a critical third-party vendor using Santander’s **Third-Party Risk Management (TPRM)** framework.
- **Key Considerations**:
  - Obtain and review **SOC 2 Type II**, **ISO 27001**, and/or **ETSI EN 319 401** certifications.
  - Evaluate contractual clauses regarding data handling, incident reporting, and audit rights.
  - Perform a **risk assessment** covering dependency risks, business continuity, and exit strategies.
  - Ensure alignment with **EBA/ECB requirements** for outsourcing and third-party risk.

---

### **4. Multi-Tier Architecture**

- **Evaluation Focus**: Verify that DigiCert’s architecture aligns with a multi-tiered model for separation of concerns and defense-in-depth.
- **Checkpoints**:
  - Do they separate presentation, application, and data tiers?
  - Are certificate issuance, validation, and revocation systems logically separated?
  - What are their **availability SLAs**, redundancy mechanisms, and failover capabilities?

---

### **5. SIEM / Syslog Integration**

- **Evaluation Focus**: Ensure DigiCert provides logs and alerts that can be ingested into Santander’s **SIEM** (e.g., Splunk, QRadar) for real-time monitoring.
- **Integration Requirements**:
  - Support for **syslog**, **JSON over HTTP(S)**, or **API-based log forwarding**.
  - Availability of **audit logs** for certificate lifecycle events (issuance, revocation, renewal).
  - Ability to tag, filter, and enrich logs for correlation within the SIEM.

---

### **6. Application Logs**

- **Evaluation Focus**: Ensure DigiCert maintains comprehensive logging of all system activities related to digital signature services.
- **Expectations**:
  - Full audit trail of actions taken by users and systems.
  - Retention period aligned with regulatory requirements (minimum 6–12 months).
  - Logs must be tamper-evident and available for export upon request.
  - Access to logs must be role-based and logged themselves.

---

### **7. Privileged Access Management (PAM)**

- **Evaluation Focus**: Ensure DigiCert follows strong PAM practices internally and allows integration with Santander’s PAM tools if needed.
- **Key Areas**:
  - Use of **least privilege** for administrative functions.
  - Just-in-Time (JIT) access controls and session monitoring where applicable.
  - Integration with enterprise PAM solutions like CyberArk or BeyondTrust (if required).
  - Strong password and MFA enforcement for admin accounts.

---

### **8. PAM at External Host**

- **Evaluation Focus**: If DigiCert hosts any components externally, assess how privileged access is managed on those external systems.
- **Considerations**:
  - Are external systems monitored and audited independently?
  - Do they restrict and monitor access to root/administrator-level accounts?
  - Is there segregation between internal and external hosted environments?
  - Can DigiCert provide evidence of hardened OS configurations, patching cycles, and access control policies?

---

### **Additional Recommendations:**

- **Penetration Testing & Red Teaming**: Request recent reports from DigiCert or conduct your own through approved vendors.
- **Vendor Due Diligence Questionnaire (VDDQ)**: Use Santander’s standard VDDQ template to collect detailed responses.
- **Contractual Safeguards**: Ensure GDPR, eIDAS, and other relevant regulations are covered in the contract terms.
- **Exit Strategy**: Define processes for transitioning to another provider without service disruption.

---

Would you like a **customizable checklist or RFP template** based on these points to send to DigiCert?
