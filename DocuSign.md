If you're currently using **DocuSign** and are considering a switch to **DigiCert** or **SwissSign** for **Qualified Electronic Signatures (QES)** under the **eIDAS Regulation**, there are several important considerations. Let's break this down step by step.

---

### 🔍 1. **Understand What You’re Using Today**
You're currently using **DocuSign**, which primarily offers **electronic signatures (e-signatures)** that comply with the **eIDAS regulation** as **Advanced Electronic Signatures (AES)** or **Simple Electronic Signatures (SES)** depending on the product tier.

- DocuSign **eSignature** = Advanced Electronic Signature (AES)
- DocuSign **with ID verification + QES compliance** = Qualified Electronic Signature (QES) in some countries

DocuSign partners with Trust Service Providers (TSPs) in the EU to offer QES in specific jurisdictions like Germany, Spain, etc., but it’s not universally available globally.

---

### 🧾 2. **What is a Qualified Electronic Signature (QES)?**

A **QES** is the highest level of electronic signature under **eIDAS**, offering the same legal standing as a handwritten signature across all EU member states.

To qualify:
- It must be based on a **qualified certificate for electronic signatures**
- Created by a **secure signature creation device**
- Issued by a **Trusted Service Provider (TSP)** listed in the **EU Trusted List**

---

### 🔄 3. **Switching from DocuSign to DigiCert or SwissSign for QES**

#### ✅ Can DigiCert or SwissSign provide QES?
Yes — both **DigiCert** and **SwissSign** are **Trusted Service Providers (TSPs)** under eIDAS and can issue **qualified certificates** for use in creating **QES**, but typically:

- **DigiCert**: Offers qualified certificates (QSCD), especially for code signing, document signing, and server authentication. They may offer QES solutions via their global TSP partnerships.
- **SwissSign**: A Swiss-based TSP, part of the Swiss Confederation, offers qualified certificates and QES services, particularly strong in Switzerland and EU via mutual recognition agreements.

However, they do **not provide an end-user platform** like DocuSign. They issue the **certificates**, not the full signing workflow.

---

### 🛠️ 4. **What Would Change If You Use DigiCert/SwissSign Instead?**

| Feature | DocuSign | DigiCert / SwissSign |
|--------|----------|----------------------|
| Type of Signature | SES, AES, QES (in certain regions) | QES (via qualified certificates) |
| Signing Platform | Yes (UI, API, workflows) | No (just certificate issuance) |
| Integration | Easy integration with apps | Requires technical setup for use in your own system |
| Legal Compliance | Global compliance, including eIDAS | Full eIDAS compliance for QES |
| Certificate Issuance | Not applicable | Yes (qualified certificates for QES) |
| Secure Signature Creation Device (SSCD) | Managed via DocuSign partner TSPs | Required; you must use a secure environment (e.g., HSM or smart card) |

---

### 🧩 5. **How to Implement QES with DigiCert or SwissSign**

If you want to move to QES using DigiCert or SwissSign, you would need to:

#### Step 1: Choose a QES Solution
- Obtain **qualified certificates** from DigiCert or SwissSign.
- Ensure they support **document signing certificates** under eIDAS.

#### Step 2: Use a Secure Signature Creation Device (SSCD)
- This could be a hardware security module (HSM), smart card, or USB token.
- The private key used for signing must never leave the secure environment.

#### Step 3: Integrate with Your Own Signing System
- You'll need to build or integrate with a system that uses these certificates for signing documents.
- Tools like Adobe PDF signing, Java/Python libraries (e.g., iText, DigiDoc4j), or custom software may be required.

#### Step 4: Maintain Audit Trail & Timestamping
- Ensure time-stamping from a qualified TSA (Time Stamp Authority).
- Keep logs for auditability and non-repudiation.

---

### 📌 6. **Do You Need to Replace DocuSign Entirely?**

Not necessarily. Here are options:

#### Option A: Continue Using DocuSign
- For most business use cases (contracts, HR docs), DocuSign’s AES is sufficient.
- Only use QES where legally required (e.g., real estate, public procurement in some EU countries).

#### Option B: Use DocuSign + QES Add-on
- Some DocuSign plans allow you to upload qualified certificates for signing (e.g., via DocuSign SignServer or local integrations).
- You can sign documents using your own QES infrastructure while using DocuSign as a workflow engine.

#### Option C: Replace DocuSign Entirely
- Build your own QES-compliant signing system using DigiCert/SwissSign certificates.
- More complex, requires development, testing, and compliance management.

---

### 🧭 7. **Recommendation Based on Use Case**

| Scenario | Recommendation |
|---------|----------------|
| Most business contracts | Stick with DocuSign (AES) |
| Legal requirement for QES | Use QES via DigiCert/SwissSign (requires internal integration) |
| Hybrid approach | Use DocuSign + QES for specific document types |
| In-house control needed | Build your own system with QES certificates |

---

### 📞 Next Steps

- Contact **DigiCert** or **SwissSign** sales/support to request **qualified document signing certificates**.
- Evaluate if your current IT team has the expertise to manage **PKI**, **SSCDs**, and **QES signing systems**.
- Consider working with vendors like **OpenTrust**, **Ubisign**, or **DocBridge** who offer full QES platforms with integrated workflows.

---

Would you like help drafting a message to DigiCert or SwissSign requesting information about qualified certificates for QES?

Sure! Here's a **diagram** showing how **DocuSign SignServer** fits into your workflow when integrating with a **Trusted Service Provider (TSP)** like **DigiCert or SwissSign**, especially for **Qualified Electronic Signature (QES)** use cases.

Since I can’t draw images directly, I’ll provide you with a **text-based diagram** that you can visualize or convert into an actual flowchart using tools like **Lucidchart**, **draw.io (diagrams.net)**, or **Microsoft Visio**.

---

## 🧾 DocuSign SignServer Architecture Diagram (Text Version)

```
+---------------------+
|    DocuSign UI      |
|  (Envelope Creation)|
+----------+----------+
           |
           | 1. Prepare and send envelope
           v
+-----------------------------+
|     DocuSign Platform       |
|  (Routing, Workflow, etc.)  |
+--------------+--------------+
               |
               | 2. Detect QES requirement
               v
+-------------------------------+
|   DocuSign SignServer         |
|  (On-premises or Cloud)       |
+-------------------------------+
               |
               | 3. Request signature using
               |    qualified certificate
               v
+-------------------------------+
|   Secure Signature Device     |
|  (HSM / Smart Card / USB Token)|
+-------------------------------+
               |
               | 4. Certificate from TSP
               |    (e.g., DigiCert, SwissSign)
               v
+-------------------------------+
|   Qualified Trust Services    |
|  (Time Stamp, Validation, etc.)|
+-------------------------------+
               |
               | 5. Return signed document
               v
+-----------------------------+
|     DocuSign Platform       |
|  (Finalize & Notify Users)  |
+-----------------------------+
```

---

## 🔍 Explanation of Each Step

### 1. **User creates an envelope in the DocuSign UI**
- The user prepares a document and defines signers.
- Some documents may require **QES compliance** based on template, metadata, or business rules.

### 2. **Document sent to DocuSign Platform**
- Envelope is processed by DocuSign’s system.
- If QES is required, the system routes the document to **SignServer** instead of using standard signing.

### 3. **DocuSign SignServer handles the advanced signing**
- SignServer connects to secure infrastructure (like an HSM).
- It applies a digital signature using a **qualified certificate** stored securely.

### 4. **Secure Signature Creation Device (SSCD)**
- A hardware security module (HSM), smart card, or token stores the private key.
- The key never leaves this device — a requirement under eIDAS for QES.

### 5. **Trust services (timestamping, validation)**
- A **qualified Time Stamp Authority (TSA)** ensures long-term validity.
- Optional: Validation via OCSP or CRL to verify certificate status at time of signing.

### 6. **Signed document returned to DocuSign**
- Final PDF with full QES metadata is uploaded back to DocuSign.
- All users are notified as usual.

---

## 📦 Tools Used in This Setup

| Component | Description |
|----------|-------------|
| **DocuSign UI** | Front-end for preparing and sending documents |
| **DocuSign Platform** | Core DocuSign system managing workflows and envelopes |
| **DocuSign SignServer** | Backend signing engine supporting QES |
| **HSM (e.g., Thales, Utimaco, AWS CloudHSM)** | Secure storage of private keys |
| **DigiCert / SwissSign** | Issuers of qualified certificates |
| **Qualified TSA** | Provides trusted timestamping |

---

## 📨 Want a Visual Version?

You can copy the structure above into any diagramming tool. Here’s how:

1. Go to [https://app.diagrams.net](https://app.diagrams.net) (free online tool).
2. Create rectangles for each component.
3. Connect them with arrows according to the steps.
4. Save or export as PNG/PDF.

---

## 📩 Next Steps

Would you like:
- 📄 A downloadable version of this diagram (as a `.drawio` or `.vsdx` file)?
- 📋 Sample configuration settings for deploying SignServer?
- 🧰 Help choosing between different HSM vendors?

Let me know what you'd like next!
