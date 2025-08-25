Perfect — let’s walk through this step by step 👌.
We’re setting up **SAML 2.0 SSO between Temenos BrowserWeb (SP) and Entra ID (IdP)**.

---

## 🔹 1. Prepare in Entra ID (Azure AD)

1. **Register BrowserWeb as an Enterprise Application**

   * Go to **Entra ID → Enterprise Applications → New Application → Create your own application**.
   * Choose **Integrate any other application you don’t find in gallery (non-gallery)**.

2. **Configure SAML-based Sign-on**

   * Upload **BrowserWeb metadata** if available, or configure manually.
   * Set **Identifier (Entity ID)** → usually the BrowserWeb app identifier (e.g. `https://<browserweb-server>/browserweb`).
   * Set **Reply URL (ACS URL)** → BrowserWeb Assertion Consumer Service endpoint (e.g. `https://<browserweb-server>/browserweb/saml/acs`).
   * Set **Sign-on URL** (optional) → BrowserWeb landing page.

3. **Download SAML metadata** from Entra ID

   * This XML contains the IdP entity ID, SSO URL, and the signing certificate.

4. **Configure user attributes & claims**

   * Map `NameID` to the T24 username (often `userPrincipalName` or `sAMAccountName`).
   * Add roles or group claims if needed for T24 mapping.

---

## 🔹 2. Configure BrowserWeb (Service Provider)

1. **Enable SAML in BrowserWeb**

   * Go into BrowserWeb’s **sso-config.xml** (or equivalent configuration file in your version).
   * Import the IdP metadata (from Entra ID).
   * Define BrowserWeb’s **Entity ID** and **ACS URL** to match what you registered in Entra ID.

2. **Configure SP signing & encryption (optional)**

   * Generate a keypair for BrowserWeb.
   * Upload SP metadata back to Entra ID if required (so it knows BrowserWeb’s certificate).

3. **Trusted login setup (BrowserWeb → T24)**

   * In `browserweb.xml` / application config: enable **trusted login**.
   * Map the SAML `NameID` or claim (like UPN) to a **T24 user ID**.
   * Ensure the T24 user exists and is configured for trusted login (no password prompt).

---

## 🔹 3. Test the Flow

1. Open BrowserWeb in a browser.
2. You should be redirected to Entra ID login.
3. After authentication, Entra ID issues a **SAMLResponse** → BrowserWeb ACS endpoint.
4. BrowserWeb validates signature, extracts `NameID`.
5. BrowserWeb passes user via trusted login to T24/TAFJ.
6. User lands in T24 without password prompt.

---

## 🔹 4. Troubleshooting

* **Invalid SAMLResponse** → check certificates & ACS URL.
* **User not found in T24** → check mapping between SAML claim and T24 user ID.
* **Looping login** → usually ACS URL mismatch or clock skew.

---

✅ Summary:

* Configure Entra ID as IdP (enterprise app, SAML).
* Configure BrowserWeb as SP (ACS, EntityID, IdP metadata).
* Enable **trusted login** in BrowserWeb to pass identity to T24.

---

👉 Do you want me to also create a **diagram showing Entra ID ↔ BrowserWeb ↔ T24 with the SAML steps** so you can visualize the flow?
