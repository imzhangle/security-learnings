**Yes, Unique AI (unique.ai), a Zurich-based Swiss company founded in 2021, provides an enterprise agentic AI platform specifically designed for financial services institutions.** It includes pre-built and customizable **financial AI agents** for tasks like KYC/AML, investment research/insights, due diligence, RFP/DDQ analysis, compliance, meeting insights, and more. These agents are deployed by major Swiss and European financial players (e.g., Pictet, UBP, SIX, LGT, Partners Group). The platform runs in secure Swiss Azure environments (multi-tenant or single-tenant/self-hosted options) and is FINMA-compliant, SOC 2 Type 2 certified, and aligned with the EU AI Act.

### Who accesses the agents?
Authenticated employees of the client financial institution (e.g., relationship managers, analysts, compliance teams, advisors). Access is strictly role-based and controlled via:
- SAML/OIDC SSO with MFA
- SCIM provisioning for automated user/group management
- Layered roles (Zitadel, Unique Roles, Space Access, Scope Access)

Users interact via the Unique AI frontend, embedded tools, or custom integrations. External access (e.g., by Unique staff) is limited and audited.

### Who has permissions to parameterize (configure/customize) agents?
**Client-side admins, IT/security teams, Space admins, and developers/engineers** (within the client’s organization). 
- Clients fully control boundaries and permissions in the **MCP Hub** (their secure control plane).
- Customization happens in the **AI Workspace / Spaces** (intuitive interface for defining prompts, steps, inputs, constraints, outputs, tools, and sub-agents).
- Developers use the platform’s API/SDK/toolkit for advanced logic.
- Unique provides training/guidance but does not hold parameterization rights—clients maintain full authority.

### Who supervises critical decisions made by agents?
**Client-side supervisors, risk/compliance officers, managers, and data owners** (again, within the financial institution). 
- Full observability via audit logs, traceability (every tool call, prompt, and action is logged and traceable back to the source/user), benchmarking dashboards, and feedback mechanisms.
- The **MCP Governance Framework** and overall AI Governance Framework enforce client-led oversight. Unique does not supervise client decisions.

### Are agents autonomous?
**Partially / agentic, but not fully autonomous.**  
Agents are “agentic” — they can autonomously orchestrate multi-step workflows, collaborate (main agent + sub-agents), use tools, adapt, research, summarize, extract data, and execute routine tasks within strict guardrails (RAG-grounded on client data, no hallucinations via benchmarking). However, the **MCP Governance Framework** explicitly requires **human-in-the-loop** for any critical or non-routine action. No process runs silently; agents pause for explicit user confirmation on data sharing or high-stakes steps.

### Which tools can agents access?
Agents access tools securely via the **MCP Hub’s MCP Connectors** (centralized, governed orchestration layer). Examples include:
- CRMs
- Document management systems
- Research databases
- Compliance tools
- Internal SharePoint/Excel/docs (via secure RAG)
- Custom/legacy systems
- Code execution (admin-enabled)
- External knowledge (controlled)

Access is per-agent and fully configurable.

### Is agent access to APIs restricted following least privilege?
**Yes — explicitly designed this way.**  
The MCP Hub enforces **privilege boundary enforcement** and **least-privilege principles**:
- Agents/MCPs can only perform actions the authenticated user/role is allowed to do.
- Granular per-agent controls over systems, tools, datasets, and prompts.
- Approved Server Registry (only pre-approved servers/connectors work).
- OAuth authorization + dynamic consent.
- Full audit logs and traceability.

This meets FINMA, SOC 2, ISO 27001/42001, and other financial-grade standards.

### Which actions execute by agent directly and what requires human?
| Category | Agent executes directly (autonomous within guardrails) | Requires explicit human approval / supervision |
|----------|-------------------------------------------------------|-----------------------------------------------|
| **Routine / Low-risk** | Data retrieval, summarization, document analysis, investment research insights, table/portfolio analysis, draft generation, sentiment extraction, meeting insights | N/A |
| **Workflow automation** | Multi-step orchestration (e.g., KYC data collection + initial checks, RFP analysis) | N/A (if within approved scope) |
| **Critical / High-risk** | Suggestions or proposals only | Final decisions (e.g., client onboarding approval, investment recommendations, compliance sign-off, data disclosure, any action with financial/legal impact) |
| **Any action involving external systems or sensitive data** | Preparation and proposal | Explicit user consent/confirmation (MCP pauses and asks) |

**Bottom line**: Agents act as highly capable digital teammates for productivity, but **clients (financial institutions) retain full control and responsibility**. Critical decisions always stay human-supervised to meet regulatory requirements. All governance is client-configurable in the MCP Hub.

This information is based on Unique AI’s publicly available platform documentation, blog posts on their MCP Governance Framework, and official site (as of 2026). For deployment-specific details, clients work directly with Unique’s team under NDA. If you are a client or prospect and need deeper internal docs, contact them via unique.ai.
