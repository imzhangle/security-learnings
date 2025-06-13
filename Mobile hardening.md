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
