Excellent question — you're diving into the **mobile app hardening** space, and these three vendors — **Build38**, **Appdome**, and **Digital.ai (formerly Arxan)** — are all major players in mobile application security for financial services and Open Banking.

Let’s analyze whether they are **concurrent (competing) or complementary**.

---

## 🔍 Overview of Each Vendor

| Vendor | Focus | Key Features |
|--------|-------|--------------|
| **Build38** | Mobile App Hardening & Secure Communication | SDK-based protection, secure communication layer (SEAL), fraud detection integration |
| **Appdome** | No-code Mobile App Integration & Security | Drag-and-drop SDK integrations, app hardening, encryption, MFA, WAF |
| **Digital.ai** | Mobile App Hardening & Runtime Protection | Code obfuscation, anti-tampering, anti-debugging, jailbreak/root detection |

---

## 🤝 Are They Concurrent or Complementary?

### ✅ **Short Answer:**
They are **mostly concurrent** — meaning they **compete directly** in the **mobile app hardening** space — but there are **nuances** depending on your use case.

---

## 🧠 Detailed Comparison

| Feature / Vendor | Build38 | Appdome | Digital.ai |
|------------------|---------|---------|------------|
| **Code Obfuscation** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Anti-Tampering** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Anti-Debugging** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Jailbreak/Root Detection** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Runtime Integrity Checks** | ✅ Yes | ✅ Yes | ✅ Yes |
| **No-code / Low-code Integration** | ❌ Mostly SDK | ✅ Yes (drag-and-drop) | ❌ Mostly SDK |
| **Secure Communication Layer (e.g., TLS + SEAL)** | ✅ Yes (Build38 SEAL) | ⚠️ Limited | ⚠️ Standard TLS only |
| **Integration with Fraud Detection Tools** | ✅ Yes (e.g., BioCatch) | ⚠️ Depends on third-party SDKs | ⚠️ Requires custom work |
| **Open Banking Use Case Fit** | ✅ Strong (Germany-focused, used by Deutsche Bank, SolarisBank) | ✅ Good (global use cases) | ✅ Strong (global enterprise) |
| **Compliance Support** | PSD2, GDPR, ISO 27001 | General compliance support | PCI DSS, NIST, ISO 27001 |

---

## 🧩 When Are They *Concurrent*?

All three offer **similar core functionality**:

- Protecting apps from reverse engineering
- Detecting rooted/jailbroken devices
- Preventing runtime tampering
- Securing API communication

👉 So yes, they **compete head-to-head** in the **mobile app hardening** market.

---

## 🧩 When Are They *Complementary*?

While mostly concurrent, there are **some areas where one might complement another**, especially when building a full-stack solution:

| Use Case | Best Fit | Could Be Complemented By |
|----------|----------|---------------------------|
| **Quick Integration Without Coding** | Appdome | Build38 or Digital.ai for deeper protections |
| **Secure Communication Layer (TLS+, SEAL)** | Build38 | Digital.ai for code-level protection |
| **Enterprise-Grade Protection (code-level + runtime)** | Digital.ai | Appdome for faster integration or DevOps automation |
| **Fraud Detection Integration** | Build38 (native integration with fraud engines) | BioCatch or other behavioral tools |

---

## 📊 Summary Table

| Aspect | Concurrent? | Complementary? |
|--------|-------------|----------------|
| **Core App Hardening Features** | ✅ Yes | ❌ No |
| **Integration Method (SDK vs No-code)** | ✅ Yes (different approaches) | ⚠️ Partially |
| **Secure Communication (e.g., SEAL)** | ❌ No | ✅ Yes (Build38 complements others) |
| **Fraud Engine Integration** | ❌ No | ✅ Yes (Build38 offers more native support) |
| **DevOps Automation / CI/CD Friendly** | ⚠️ Varies | ✅ Appdome is easier to automate |
| **Open Banking / Financial Services Use Case** | ✅ All are strong | ⚠️ Build38 has edge in EU fintech adoption |

---

## 🎯 Strategic Recommendation

Depending on your priorities:

| If You Need... | Recommended Vendor | Why |
|----------------|---------------------|-----|
| **Fast, no-code implementation** | Appdome | Easy to integrate without deep dev involvement |
| **Deep code protection + runtime defense** | Digital.ai | Industry leader in runtime integrity and obfuscation |
| **Secure comms + fraud integration (especially in EU fintech)** | Build38 | Strong in Germany/EU, integrates well with fraud engines like BioCatch |

You can even consider using **two of them together**, e.g.:

- Use **Appdome** for quick integration of basic protections
- Add **Digital.ai** or **Build38** for deeper runtime/code-level protections

However, this could lead to **overlapping features** and increased app complexity, so it's best to pick **one primary vendor** unless you have a very specific reason to combine.

---

## 📥 Next Steps

Would you like me to help you:
- Draft an RFP comparing these vendors?
- Create a side-by-side feature matrix?
- Recommend a vendor based on your region, team size, or regulatory needs?

Let me know!
Great! As a **Security Architect**, evaluating tools like **Horizon Mobile Banking**, **BioCatch**, and **Digital.ai Application Protection (formerly Arxan)** requires a **structured, risk-based, and compliance-driven approach**.

Each of these platforms serves a **different layer in the mobile security stack**, so your evaluation criteria should reflect their **unique roles** while ensuring they can be integrated into a **cohesive, enterprise-grade Open Banking solution**.

---

## ✅ Evaluation Criteria Overview

| Category | Description |
|---------|-------------|
| **1. Security Capabilities** | Core protections offered by the tool |
| **2. Compliance & Standards Support** | Alignment with regulations like PSD2, GDPR, ISO 27001, PCI DSS |
| **3. Integration Complexity** | How easy is it to integrate into existing or new apps? |
| **4. Performance Impact** | Does the tool affect app performance or user experience? |
| **5. Threat Detection & Response** | How does it detect and respond to threats in real-time? |
| **6. Management & Monitoring** | Reporting, analytics, alerting, dashboards |
| **7. Vendor Maturity & Support** | Track record, SLAs, customer support, roadmap |
| **8. Scalability & Maintenance** | Can it scale with your business? Is maintenance manageable? |
| **9. Cryptographic Controls** | Use of secure encryption, key management, etc. |
| **10. DevOps & CI/CD Compatibility** | Can it be automated in pipelines? |

---

## 🧠 Detailed Evaluation Questions by Tool

### 🔐 A. Horizon Mobile Banking (by Finastra)

#### ⚙️ Focus: Secure Mobile Banking Platform

##### 1. **Security Capabilities**
- Does Horizon include built-in jailbreak/root detection?
- What authentication mechanisms are supported (e.g., biometrics, OTP, OAuth)?
- Does it support certificate pinning for API communication?

##### 2. **Compliance**
- Is Horizon certified or compliant with:
  - PSD2 / RTS
  - ISO 27001
  - SOC 2
  - OWASP MASVS L2 or above?

##### 3. **Integration**
- What SDKs or APIs are available for extending security functionality?
- Can Horizon be integrated with third-party mobile app protection tools (like Digital.ai or Appdome)?

##### 4. **Threat Detection**
- Does Horizon provide device integrity checks or runtime tamper detection?
- Are there logs or alerts on suspicious behavior (e.g., rooted device access attempts)?

##### 5. **DevOps Compatibility**
- Can Horizon components be integrated into CI/CD pipelines?
- Is there automation support for updates or configuration changes?

---

### 🧠 B. BioCatch (Behavioral Biometrics)

#### ⚙️ Focus: Behavioral Authentication & Fraud Detection

##### 1. **Security Capabilities**
- What behavioral signals does BioCatch monitor (e.g., typing rhythm, swipe pressure)?
- Can it perform continuous authentication during a session?
- Does it support passive liveness detection?

##### 2. **Compliance**
- Does BioCatch comply with:
  - GDPR
  - eIDAS
  - PSD2 fraud detection requirements
  - CCPA (if operating in the US)?

##### 3. **Integration**
- How is the SDK integrated (iOS, Android, Web)?
- Can it work alongside other security tools like Digital.ai or WSO2 Identity Server?
- Does it support SAML/OAuth2 integration for federated identity?

##### 4. **Threat Detection**
- How does BioCatch differentiate between legitimate users and impostors?
- What kind of alerts or reports are generated upon anomaly detection?
- Does it integrate with SIEM systems (e.g., Splunk, QRadar)?

##### 5. **Performance**
- What is the impact on app load time or CPU usage?
- Is data collection done locally or sent to cloud services?

##### 6. **Vendor Maturity**
- How many financial institutions use BioCatch globally?
- What certifications exist (e.g., ISO 27001, SOC 2)?

---

### 🔒 C. Digital.ai Application Protection (formerly Arxan)

#### ⚙️ Focus: Mobile App Hardening & Runtime Protection

##### 1. **Security Capabilities**
- What hardening features are included?
  - Code obfuscation
  - Anti-debugging
  - Anti-tampering
  - Jailbreak/root detection
  - Certificate pinning
- Does it protect against memory scraping or hooking attacks (e.g., Frida, Xposed)?
- Does it offer control flow obfuscation or string encryption?

##### 2. **Compliance**
- Is Digital.ai used in regulated environments like:
  - Financial services
  - Government
  - Healthcare
- Does it support standards like:
  - NIST SP 800-53
  - ISO 27001
  - OWASP MASVS L2+
  - PCI DSS

##### 3. **Integration**
- How is the protection applied (SDK, build-time transformation)?
- Does it require manual code changes or can it be fully automated?
- Is it compatible with hybrid apps (React Native, Flutter)?

##### 4. **Threat Detection**
- Does it offer real-time tamper detection and response?
- Can it block execution if tampering is detected?
- Can it report violations back to a backend system (SIEM or backend)?

##### 5. **Management & Monitoring**
- Is there a centralized dashboard for policy management?
- Can you push updated policies or protections without recompiling the app?
- Does it log incidents and send alerts?

##### 6. **DevOps Compatibility**
- Can it be integrated into CI/CD pipelines (e.g., Jenkins, GitHub Actions)?
- Is there support for containerized builds or cloud-native deployment?

##### 7. **Cryptographic Controls**
- Does it support secure key storage (e.g., Keystore, Keychain)?
- Are cryptographic algorithms FIPS-compliant?

---

## 📋 Summary Table: Comparison Criteria

| Criteria | Horizon | BioCatch | Digital.ai |
|--------|---------|----------|------------|
| **Code Obfuscation** | ❌ | ❌ | ✅ |
| **Anti-Tampering** | ⚠️ Limited | ❌ | ✅ |
| **Jailbreak/Root Detection** | ⚠️ Basic | ❌ | ✅ |
| **Certificate Pinning** | ⚠️ Yes | ❌ | ✅ |
| **Behavioral Analytics** | ❌ | ✅ | ❌ |
| **Passive Authentication** | ❌ | ✅ | ❌ |
| **Session Monitoring** | ❌ | ✅ | ❌ |
| **API Gateway Integration** | ✅ | ❌ | ❌ |
| **Regulatory Compliance** | ✅ | ✅ | ✅ |
| **Integration Difficulty** | ⚠️ Moderate | ⚠️ Moderate | ⚠️ High |
| **DevOps Friendly** | ✅ | ⚠️ Moderate | ⚠️ High |
| **Centralized Dashboard** | ✅ | ✅ | ✅ |
| **Scalability** | ✅ | ✅ | ✅ |

---

## 🧩 Strategic Recommendation

As a **Security Architect**, your goal is not just to evaluate each tool individually, but to **integrate them into a layered defense model**:

### 🔐 Layered Defense Strategy

| Layer | Recommended Tool |
|-------|------------------|
| **Mobile App Framework** | Horizon Mobile Banking |
| **App Hardening** | Digital.ai Application Protection |
| **User Behavior Monitoring** | BioCatch |
| **Backend API Security** | ODS (Open Digital Service), OAuth2 provider |
| **Monitoring & Logging** | SIEM (Splunk, QRadar) + vendor dashboards |

---

## 📄 Bonus: RFP Template Snippet

If you're preparing an **RFP** or internal evaluation document, here’s a sample question format:

> **Question:** Please describe how your platform supports OWASP Mobile Application Security Verification Standard (MASVS) Level 2.
>
> **Expected Answer Format:**  
> - List of verified controls  
> - Attestation or certification status  
> - Sample implementation guidance

Would you like me to help draft a full **RFP template**, **evaluation matrix (Excel or PDF)**, or even a **vendor comparison presentation** based on this?


### 🔐 What Is **Secure Communication Layer Depth**?

**Secure Communication Layer Depth** refers to the **number of security-enhanced layers and mechanisms** used in the communication between a mobile app (or client) and a backend server.

In simple terms:
> It’s how **deeply encrypted, authenticated, and protected** the data is as it travels across the network — especially from threats like:
> - Man-in-the-Middle (MITM) attacks
> - Session hijacking
> - Certificate pinning bypasses
> - Network-level tampering

---

## 🧠 Why Does "Layer Depth" Matter?

In high-security environments like **Open Banking**, **financial apps**, or **enterprise mobile apps**, **standard TLS/SSL** may not be enough because:

- Attackers can sometimes bypass standard certificate pinning using tools like **Burp Suite**, **Frida**, or **rooted devices**.
- Standard TLS doesn’t always verify the **integrity of the client app itself** during communication.
- There's no **session-specific encryption** or **device integrity validation** built into the protocol.

That’s where **deep secure communication layers** come in — they add **custom, proprietary protections** on top of standard TLS to make network communication **far more resilient** to these advanced attacks.

---

## 🛡️ Components That Contribute to Secure Communication Layer Depth

Here are key features that indicate **deeper secure communication capabilities**:

| Feature | Description | Increases Depth? |
|--------|-------------|------------------|
| **Standard TLS/SSL** | Basic HTTPS communication | ❌ (Baseline) |
| **Certificate Pinning** | Ensures only trusted certificates are accepted | ⚠️ Moderate |
| **Custom Encryption Layer** | Adds an extra layer beyond TLS (e.g., SEAL by Build38) | ✅ Yes |
| **Session Token Binding** | Binds API calls to session tokens for replay protection | ✅ Yes |
| **Device Integrity Check During Comm** | Validates the app hasn't been tampered with before allowing communication | ✅ Yes |
| **Runtime MITM Detection** | Detects if a proxy or attacker is intercepting traffic | ✅ Yes |
| **Dynamic Key Exchange** | Uses runtime-generated keys instead of static ones | ✅ Yes |
| **Anti-Replay Protection** | Prevents reuse of intercepted requests | ✅ Yes |
| **Custom Transport Protocol** | Uses a non-standard transport layer (not just HTTP/HTTPS) | ✅ Yes |
| **Integration with Fraud Engines** | Communicates device risk score to backend | ✅ Yes |

---

## 🔍 Examples of Vendors with Deep vs. Shallow Layer Depth

| Vendor | Secure Comm Layer Depth | Description |
|--------|--------------------------|-------------|
| **Build38 (SEAL)** | ✅ Deep | Proprietary SEAL layer adds encryption, integrity checks, and fraud integration; used in EU fintech |
| **Digital.ai** | ⚠️ Moderate | Strong code obfuscation, includes TLS pinning but no custom protocol |
| **Appdome** | ⚠️ Moderate | Offers TLS pinning and WAF, but no deep custom layer |
| **Custom App with OWASP MASVS L2+** | ✅ Deep (if implemented) | Can include all depth features but requires expert development |

---

## 🧩 Real-World Example: Build38 SEAL

**SEAL (Secure Execution & Attack Localization)** by Build38 is a great example of **deep secure communication**.

It does the following:
- Wraps communication in a **custom protocol** on top of TLS
- Embeds **device attestation** into every request
- Provides **runtime MITM detection**
- Supports **dynamic key exchange**
- Integrates with fraud engines like BioCatch

This makes it **much harder for attackers** to inspect or manipulate API traffic — even if they control the device.

---

## 📊 Summary Table

| Level of Depth | Features Included | Use Case |
|----------------|-------------------|----------|
| **Shallow** | TLS + basic pinning | General-purpose apps |
| **Moderate** | TLS + pinning + WAF | Mid-tier enterprise apps |
| **Deep** | Custom protocol + runtime checks + fraud integration | Open Banking, fintech, regulated apps |

---

## 🎯 When You Should Care About Secure Communication Layer Depth

You should care about **secure communication layer depth** when:

- You're in a **regulated industry** (e.g., finance, healthcare)
- Your app handles **sensitive data** (e.g., payments, PII)
- You must comply with standards like:
  - **PSD2 RTS**
  - **OWASP MASVS L2+**
  - **ISO 27001**
- You want to protect against **advanced reverse engineering or MITM attacks**

---

Would you like me to help you:
- Create a **secure communication architecture diagram**?
- Draft a **vendor evaluation checklist** based on layer depth?
- Compare vendors like Build38 SEAL vs. Digital.ai vs. Appdome in detail?

Let me know!
