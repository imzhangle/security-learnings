Got it — you’re asking specifically about **T24 Transact (Temenos core banking system)** and how RBAC should be structured there.

In banking environments like **T24**, the decision leans strongly toward **granular, least-privilege roles that can be combined** — but with careful governance. Here’s why:

---

### Why granular roles are better for T24:

1. **Regulatory & Compliance (e.g., SOX, PCI DSS, GDPR)**

   * Auditors expect *least privilege* and *segregation of duties (SoD)*.
   * If you create “monolithic” roles that bundle too many permissions, you risk toxic combinations (e.g., “create customer” + “approve customer”), which is a compliance red flag.

2. **Segregation of Duties (SoD)**

   * In banking, SoD is critical — e.g., one role to *initiate* a payment, another to *authorize* it.
   * Granular roles allow you to enforce these cleanly, whereas monolithic roles may accidentally bypass them.

3. **Auditability**

   * With granular roles, you can trace exactly which permission was granted and why.
   * This makes user access reviews much easier and helps during regulatory audits.

4. **Flexibility for Business Functions**

   * T24 often supports multiple business lines (Retail, Corporate, Treasury, etc.).
   * Granular roles let you mix-and-match for different functions, instead of duplicating a giant “super role” for each department.

---

### Risks of granular roles in T24

* **Too many roles**: Without governance, you could end up with hundreds of tiny roles, making it hard to manage.
* **Operational overhead**: Business users might not understand which roles they need.

**Mitigation:**

* Use a **hybrid model** → define granular roles at the permission level, then bundle them into **job-function roles** like *Teller*, *Relationship Manager*, *Treasury Back Office*.
* Apply **SoD rules** to prevent assigning conflicting bundles.
* Automate access reviews and provisioning via an IAM/IGA tool (SailPoint, Azure AD, Oracle Identity, etc.).

---

✅ **Best practice for T24 access:**

* **Granular least-privilege roles** (aligned to system capabilities).
* **Composite/job roles** for assignment (aligned to business functions).
* Enforce **SoD policies** and regularly review access.

---

Would you like me to give you a **sample T24 RBAC role hierarchy** (granular → composite → user) for something like *Retail Teller* vs *Back Office Clerk*? That way you can see how it looks in practice.
