As a **Security Architect** responsible for an Oracle FCCM (OFSAA) + EDQ + TF/SAA environment in a bank, here is the exact list of network flows you must open in firewalls (and justify in your architecture diagrams).  
These are the **real production flows** used globally in Tier-1 banks in 2025.

| # | Source Component                | Destination Component               | Port(s) – Typical & Secure | Protocol | Direction | Mandatory? | Purpose & Justification |
|---|----------------------------------|--------------------------------------|----------------------------|----------|-----------|------------|-------------------------|
| 1 | User browsers / Citrix / VDI     | WebLogic Managed Servers (FCCM UI)  | 443 (HTTPS)                | TCP      | Inbound   | Yes        | FCCM UI, ECM, KYC, Alert workbench |
| 2 | WebLogic Admin/Managed Servers   | Oracle Database (FCCM + OFSAA schema) | 1521 or 2484 (TLS)        | TCP      | Outbound  | Yes        | All metadata, alerts, cases, transactions |
| 3 | OFSAA Processing Servers (Batch nodes) | Oracle Database                  | 1521 or 2484 (TLS)         | TCP      | Bidirectional | Yes    | Run-rule, data load, WE Logic batch screening |
| 4 | EDQ Server(s)                    | Oracle Database (FCCM staging area) | 1521/2484                  | TCP      | Outbound  | Yes        | Write cleansed/standardized customer data |
| 5 | Core Banking / Data Warehouse / DWH | OFSAA Data Ingestion (DIH or AAI) | 22 (SFTP) or 443 (HTTPS)   | TCP      | Inbound   | Yes        | Daily transaction & customer feeds |
| 6 | OFSAA DIH / AAI servers          | Oracle Database                     | 1521/2484                  | TCP      | Outbound  | Yes        | Load staged data into FCCM tables |
| 7 | TF or SAA (real-time screening)  | WebLogic (FCCM Inline Screening service) | 443 or custom (e.g., 7001) | HTTPS/TCP | Outbound  | Yes (if TF/SAA used) | Send screening result back to FCCM |
| 8 | WebLogic / OFSAA servers         | TF or SAA screening engine          | 443 (most common) or 8443  | HTTPS    | Outbound  | Yes (if used) | Real-time payment/customer screening |
| 9 | All servers (WebLogic, EDQ, OFSAA batch, TF/SAA) | LDAP / Active Directory         | 636 (LDAPS) or 389 (STARTTLS) | TCP   | Outbound  | Yes        | User authentication & group sync |
|10 | All servers                      | NTP servers                         | 123                        | UDP      | Outbound  | Yes        | Time sync – critical for audit & sequence |
|11 | All servers                      | DNS                                 | 53                         | TCP/UDP  | Outbound  | Yes        |
|12 | All servers                      | SIEM / Syslog (Splunk, QRradar, etc.) | 514 or 6514 (TLS)        | TCP/UDP  | Outbound  | Yes        | Audit logs & alerts |
|13 | WebLogic / OFSAA                 | Oracle Support / My Oracle Support  | 443                        | HTTPS    | Outbound  | Recommended| Patch downloads, CSR |
|14 | EDQ Server                       | External address/watchlist providers (optional) | 443            | HTTPS    | Outbound  | If EDQ does real-time enrichment |
|15 | Management jump hosts / Bastion  | All servers (WebLogic, DB, OFSAA batch, EDQ) | 22 (SSH)          | TCP      | Inbound   | Yes        | Admin access (lock down source IPs!) |

### Recommended Secure Configuration (2025 standards)

| Setting                        | Recommended Value                              | Why |
|--------------------------------|------------------------------------------------|-----|
| Database listener              | TLS-only (2484) – disable plain 1521           | PCI-DSS, FFIEC, MAS TRM |
| WebLogic front-end             | Terminate SSL at F5/NGINX → backend HTTP only is acceptable inside DMZ if network is segmented | Common pattern |
| OFSAA batch → DB               | Always direct, never through WebLogic           | Performance + security |
| Real-time screening (TF/SAA)   | Mutual TLS (mTLS) if possible                  | Prevents spoofing |
| All outbound internet          | Via authenticated forward proxy (Zscaler, etc.)| DLP + threat protection |

### Typical Network Zones & Example Segmentation

```
Internet → F5 Load Balancer (443) → DMZ Web Tier (WebLogic Managed Servers)
                                           → App Tier (OFSAA Processing + EDQ + TF/SAA)
                                           → Database Tier (Oracle DB with TDE + TLS)
```

### Quick Checklist You Can Copy into Your Firewall Rule Request

| Source Zone       | Destination Zone | Ports          | Justification |
|-------------------|------------------|----------------|-------------|
| End-User VLAN     | DMZ-Web          | TCP 443        | FCCM UI access |
| DMZ-Web           | App Tier         | TCP 1521/2484  | DB access |
| App Tier          | App Tier (DB)    | TCP 1521/2484  | Batch & metadata |
| Integration Zone  | App Tier         | TCP 22/443     | Data feeds |
| App Tier          | Screening Zone   | TCP 443        | TF/SAA real-time |
| All zones         | Corp LDAP        | TCP 636        | Auth |
| All zones         | SIEM             | TCP 6514       | Logging |

Open exactly these flows (and nothing more) and you will have a secure, compliant, and fully functional FCCM + EDQ + TF/SAA landscape. Let me know your exact versions or if you are on OCI – the flows are 99% the same, but OCI uses VNICs and security lists instead of traditional firewalls.
