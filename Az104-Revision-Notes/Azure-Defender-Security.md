<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Microsoft Defender for Cloud & Azure Security — AZ-104 Revision Notes

---

## 1. What is Microsoft Defender for Cloud?

- **Cloud-native Application Protection Platform (CNAPP)** for Azure, hybrid, and multi-cloud
- Provides: **Security posture management (CSPM)** + **Cloud workload protection (CWP)**
- Three core pillars: **Assess** → **Secure** → **Defend**
- Formerly: Azure Security Center + Azure Defender (merged)
- Built into Azure — enabled **by default** (free tier)

---

## 2. Key Components

| Component | Description |
|---|---|
| **Secure Score** | Numerical measure of security posture (0–100%) |
| **Security Recommendations** | Actionable guidance to fix misconfigurations |
| **Security Alerts** | Threat detection notifications (requires Defender plans) |
| **Regulatory Compliance** | Dashboard showing compliance with standards |
| **Workload Protections** | Per-resource Defender plans (Servers, Storage, SQL, etc.) |
| **Cloud Security Posture Management (CSPM)** | Free — continuous assessment |
| **Cloud Workload Protection (CWP)** | Paid — advanced threat detection |
| **Azure Security Benchmark** | Default policy initiative (auto-assigned) |

---

## 3. Free (CSPM) vs Paid (Defender Plans) Comparison

| Feature | **Free (CSPM)** | **Enhanced (Defender Plans)** |
|---|---|---|
| **Secure Score** | ✅ | ✅ |
| **Security Recommendations** | ✅ | ✅ |
| **Azure Security Benchmark** | ✅ (auto-assigned) | ✅ |
| **Regulatory Compliance** | ❌ (ASB only) | ✅ (PCI DSS, ISO 27001, SOC TSP, NIST, etc.) |
| **Threat detection / Security Alerts** | ❌ | ✅ |
| **Vulnerability assessment** | ❌ | ✅ |
| **Just-In-Time VM access** | ❌ | ✅ (Defender for Servers) |
| **Adaptive application controls** | ❌ | ✅ |
| **Adaptive network hardening** | ❌ | ✅ |
| **File integrity monitoring** | ❌ | ✅ |
| **Container image scanning** | ❌ | ✅ |
| **Agentless scanning** | ❌ | ✅ (CSPM enhanced) |
| **Multi-cloud (AWS, GCP)** | ✅ (basic) | ✅ (full) |
| **Cost** | **Free** | Per-resource pricing |

> ⚠️ **EXAM TIP:** **Free tier** gives you Secure Score + Recommendations + Azure Security Benchmark. For threat detection, JIT access, vulnerability scanning → need **paid Defender plans**.

> ⚠️ **EXAM TIP:** **Azure Security Benchmark** is auto-assigned to every subscription. It cannot be deleted but can be customized.

---

## 4. Secure Score

- **Percentage** (0–100%) representing overall security posture
- Based on how many recommendations are resolved
- Higher score = more secure
- Score calculated: `(controls achieved / total possible controls) × 100`

### Controls
- Recommendations grouped into **security controls** (logical groups)
- Each control has **max score points**
- Must fix ALL recommendations in a control to earn its points
- Some recommendations = **Preview** (don't affect score)

### Score Impact

| Action | Effect |
|---|---|
| Fix all items in a control | ✅ Earn full control points |
| Fix some items in a control | ❌ No partial credit |
| Exempt a recommendation | ⚠️ Points removed from denominator |
| Unhealthy resources | ↓ Lowers score |

#### Portal Path
```
Defender for Cloud → Overview → Secure Score
Defender for Cloud → Secure Score → View details per control
```

> ⚠️ **EXAM TIP:** Secure Score gives **no partial credit** — ALL recommendations in a control must be resolved to earn that control's points. Partial fixes = 0 points for that control.

> ⚠️ **EXAM TIP:** **Exempting** a recommendation removes it from score calculation (both numerator and denominator). Use for accepted risks or mitigated-elsewhere scenarios.

---

## 5. Security Recommendations

### Recommendation Properties

| Property | Description |
|---|---|
| **Severity** | High / Medium / Low |
| **Freshness interval** | 12 hours (assessment runs every 12h) |
| **Status** | Healthy / Unhealthy / Not applicable |
| **Remediation** | Quick fix (automated) or Manual steps |
| **Exempt** | Mark as waived / mitigated |

### Common AZ-104 Recommendations

| Recommendation | Category |
|---|---|
| MFA should be enabled on accounts with owner permissions | Identity |
| Disk encryption should be applied on VMs | Compute |
| Storage accounts should restrict network access | Data |
| Subnets should be associated with an NSG | Networking |
| Management ports should be closed | Networking |
| System updates should be installed on your machines | Compute |
| Vulnerabilities should be remediated | Compute |
| Secure transfer to storage accounts should be enabled | Data |

#### Portal Path — View Recommendations
```
Defender for Cloud → Recommendations →
Filter by: Severity / Resource type / Environment →
Select recommendation → View affected resources → Remediate
```

#### Portal Path — Exempt a Recommendation
```
Defender for Cloud → Recommendations → Select recommendation →
Select resource → Exempt → Reason: Waiver / Mitigated / Risk accepted →
Expiration (optional) → Save
```

> ⚠️ **EXAM TIP:** Recommendations run every **12 hours**. Changes take up to 12h to reflect in Secure Score.

> ⚠️ **EXAM TIP:** **Quick Fix** remediation = one-click automated fix (not all recommendations have this). Check for the ⚡ icon.

---

## 6. Regulatory Compliance

- Dashboard showing compliance against industry standards
- **Azure Security Benchmark (ASB)** = default (auto-assigned, free)
- Additional standards (paid): PCI DSS, ISO 27001, SOC 2 TSP, NIST 800-53, HIPAA, CIS

### Standards Available

| Standard | Free? | Description |
|---|---|---|
| **Azure Security Benchmark** | ✅ Free (auto-assigned) | Microsoft's Azure-specific best practices |
| **PCI DSS 3.2.1 / 4.0** | ❌ Defender Plans | Payment card industry |
| **ISO 27001** | ❌ Defender Plans | Information security management |
| **SOC 2 Type 2** | ❌ Defender Plans | Service organization controls |
| **NIST SP 800-53** | ❌ Defender Plans | US government standard |
| **HIPAA HITRUST** | ❌ Defender Plans | Healthcare compliance |
| **CIS Benchmarks** | ❌ Defender Plans | Center for Internet Security |

#### Portal Path — Add Compliance Standard
```
Defender for Cloud → Regulatory compliance →
Manage compliance policies → Select subscription →
Industry & regulatory standards → + Add more standards →
Select standard → Add
```

#### Portal Path — View Compliance
```
Defender for Cloud → Regulatory compliance →
Select standard → View controls →
Select control → View assessments → Remediate
```

> ⚠️ **EXAM TIP:** **ASB (Azure Security Benchmark)** is automatically assigned as a **policy initiative** to every subscription. Additional standards require Defender plans.

> ⚠️ **EXAM TIP:** Regulatory compliance is mapped to **Azure Policy** — each control maps to policy definitions. Non-compliant = policy evaluation failed.

---

## 7. Defender Plans (Cloud Workload Protection)

### Available Plans

| Plan | Protects | Key Features |
|---|---|---|
| **Defender for Servers** | VMs (Azure, hybrid, multi-cloud) | JIT access, vulnerability assessment, FIM, adaptive controls |
| **Defender for Storage** | Storage accounts | Malware scanning, sensitive data threat detection |
| **Defender for SQL** | Azure SQL, SQL on VMs | Vulnerability assessment, advanced threat protection |
| **Defender for App Service** | App Service apps | Threat detection, dangling DNS |
| **Defender for Key Vault** | Key Vaults | Unusual access patterns, suspicious operations |
| **Defender for ARM** | Azure Resource Manager | Suspicious management operations |
| **Defender for DNS** | Azure DNS | Malicious DNS activity |
| **Defender for Containers** | AKS, container registries | Image scanning, runtime threat protection |
| **Defender for Databases** | Cosmos DB, MySQL, PostgreSQL, MariaDB | Threat protection |
| **Defender CSPM** | Posture management (enhanced) | Attack path analysis, agentless scanning, governance |

#### Portal Path — Enable Defender Plans
```
Defender for Cloud → Environment settings →
Select subscription → Defender plans →
Toggle each plan: On / Off → Save
```

> ⚠️ **EXAM TIP:** Each Defender plan is **per-resource, per-month** pricing. You can enable plans individually — don't need all plans.

> ⚠️ **EXAM TIP:** **Defender for Servers** is required for: JIT VM access, adaptive application controls, file integrity monitoring, and vulnerability assessment.

---

## 8. Just-In-Time (JIT) VM Access

- **Reduces attack surface** by locking down management ports (RDP, SSH, WinRM)
- Opens ports **only when needed**, for **limited time**, to **specific IPs**
- Creates **NSG rules** (and Azure Firewall rules if applicable) to allow temporary access
- Requires: **Defender for Servers Plan 2**

### How It Works

| Step | Action |
|---|---|
| 1. Enable JIT | Locks management ports in NSG (Deny All inbound) |
| 2. Request access | User requests access for specific port/IP/duration |
| 3. Approval | Auto-approved or requires manual approval |
| 4. Access granted | NSG allow rule added for specified time/IP |
| 5. Expiry | After duration, NSG rule automatically removed |

### Default JIT Settings

| Setting | Default |
|---|---|
| Ports | 22 (SSH), 3389 (RDP), 5985 (WinRM), 5986 (WinRM-S) |
| Max request time | **3 hours** |
| Permitted source IPs | Per request |
| Protocol | TCP |

#### Portal Path — Enable JIT on VM
```
Defender for Cloud → Workload protections → JIT VM access →
Not configured tab → Select VM → Enable JIT →
Configure ports (defaults: 22, 3389, 5985, 5986) →
Max time, Allowed source IPs → Save

OR: VM → Settings → Connect → Enable JIT
```

#### Portal Path — Request JIT Access
```
Defender for Cloud → Workload protections → JIT VM access →
Configured tab → Select VM → Request access →
Toggle ports → Set source IP: My IP / IP range →
Time range → Open ports
```

> ⚠️ **EXAM TIP:** JIT requires **Defender for Servers** (not free tier). Creates **Deny rules** in NSG when enabled, then **Allow rules** when access is requested.

> ⚠️ **EXAM TIP:** JIT adds NSG rules with priority **lower number** (higher priority) than existing deny rules. Rules auto-removed after time expires.

> ⚠️ **EXAM TIP:** For exam: "Reduce attack surface for management ports" → Answer = **JIT VM access**. "Allow RDP only when needed" → **JIT**.

---

## 9. Adaptive Application Controls

- **Whitelisting** of applications allowed to run on VMs
- ML-based — learns normal patterns and recommends allowed applications
- Alerts on violations (unauthorized application execution)
- Requires: **Defender for Servers**

#### Portal Path
```
Defender for Cloud → Workload protections → Adaptive application controls →
Select VM group → Review recommendations →
Configure allowed applications → Audit / Enforce → Save
```

> ⚠️ **EXAM TIP:** Adaptive Application Controls = application **whitelisting**. "Prevent unauthorized applications" → this feature.

---

## 10. Adaptive Network Hardening

- Recommends tighter **NSG rules** based on actual traffic patterns
- Compares actual traffic vs current NSG rules
- Suggests: remove overly permissive rules, restrict source IPs, reduce port ranges
- Requires: **Defender for Servers**

#### Portal Path
```
Defender for Cloud → Workload protections →
Adaptive network hardening →
Select VM → View recommendations →
Apply recommended rules → Enforce
```

> ⚠️ **EXAM TIP:** Adaptive Network Hardening = ML-based NSG recommendations. "Tighten NSG rules automatically" → this feature.

---

## 11. File Integrity Monitoring (FIM)

- Monitors changes to: **OS files**, **Windows registry**, **application files**, **Linux system files**
- Alerts on unexpected changes (potential tampering/compromise)
- Requires: **Defender for Servers Plan 2**
- Uses **Log Analytics agent** or **Azure Monitor Agent**

### Monitored Items

| Platform | What's Monitored |
|---|---|
| **Windows** | System files, registry, installed software |
| **Linux** | /etc/\*, /bin/\*, /sbin/\*, /usr/bin/\*, PEM files |
| **Custom** | User-defined paths and registry keys |

#### Portal Path
```
Defender for Cloud → Workload protections →
File integrity monitoring →
Select Log Analytics workspace → Enable FIM →
Configure: Windows files / Linux files / Registry → Save
```

---

## 12. Security Alerts

- Generated by **Defender plans** when threats detected
- Severity: **High / Medium / Low / Informational**
- Include: description, affected resource, attack tactics (MITRE ATT&CK), remediation steps

### Alert Lifecycle

| Status | Description |
|---|---|
| **Active** | New unresolved alert |
| **In Progress** | Being investigated |
| **Resolved** | Alert addressed |
| **Dismissed** | False positive or accepted risk |

#### Portal Path
```
Defender for Cloud → Security alerts →
Filter: Severity / Status / Time range →
Select alert → View details → Take action
```

### Suppression Rules
- Suppress false-positive alerts automatically
- Filter by: alert type, resource, IP, entity

#### Portal Path
```
Defender for Cloud → Security alerts → Suppression rules →
+ Create suppression rule →
Alert type, Entities, Reason → Expiration → Create
```

> ⚠️ **EXAM TIP:** Security alerts are only available with **paid Defender plans**. Free tier = recommendations only, no threat alerts.

---

## 13. Workflow Automation

- Trigger **Logic Apps** based on: security alerts, recommendations, regulatory compliance changes
- Automate: notification (email, Teams, Slack), remediation, ticketing

#### Portal Path
```
Defender for Cloud → Environment settings →
Select subscription → Workflow automation → + Add workflow automation →
Trigger: Security alert / Recommendation / Compliance change →
Severity filter: High / Medium →
Logic App: Select / Create → Save
```

> ⚠️ **EXAM TIP:** Workflow automation uses **Logic Apps** — not Functions, not Runbooks. If exam says "automate response to security alert" → Logic App via Workflow Automation.

---

## 14. Continuous Export

- Export Defender for Cloud data to **Log Analytics** or **Event Hub**
- Data types: security alerts, recommendations, secure score, regulatory compliance
- For SIEM integration (Sentinel, Splunk, etc.)

#### Portal Path
```
Defender for Cloud → Environment settings → Select subscription →
Continuous export →
Export target: Log Analytics workspace / Event Hub →
Data types: Security alerts ✅, Recommendations ✅, Secure score ✅ →
Export frequency: Streaming / Snapshots →
Save
```

---

## 15. Azure Security Benchmark (ASB)

- Microsoft's **best-practice framework** for Azure security
- Auto-assigned to every subscription as a **policy initiative**
- Maps to: NIST, CIS, PCI DSS controls
- Controls organized by areas: Network Security, Identity Management, Data Protection, etc.

### ASB Control Families

| Control | Description |
|---|---|
| **NS** (Network Security) | NSGs, firewalls, DDoS, private endpoints |
| **IM** (Identity Management) | MFA, conditional access, PIM |
| **PA** (Privileged Access) | Admin accounts, JIT, PIM |
| **DP** (Data Protection) | Encryption at rest, in transit, key management |
| **AM** (Asset Management) | Resource inventory, tagging |
| **LT** (Logging and Threat Detection) | Diagnostic logging, Defender |
| **IR** (Incident Response) | Alert response, automation |
| **PV** (Posture and Vulnerability Management) | Vulnerability scanning, patching |
| **ES** (Endpoint Security) | EDR, antimalware |
| **BR** (Backup and Recovery) | Azure Backup, geo-redundancy |
| **DS** (DevOps Security) | IaC, CI/CD security |
| **GS** (Governance and Strategy) | Policies, roles, responsibilities |

> ⚠️ **EXAM TIP:** ASB = **automatically assigned** as a policy initiative (Microsoft Cloud Security Benchmark). It's the default regulatory standard — cannot be removed but can be supplemented with additional standards.

---

## 16. Azure Policy Integration

- Defender for Cloud assessments are powered by **Azure Policy**
- Each recommendation maps to an **Azure Policy definition**
- **Policy initiative** = collection of policies (ASB = built-in initiative)
- Non-compliant resources → appear as unhealthy recommendations

| Concept | Defender for Cloud | Azure Policy |
|---|---|---|
| **Standard** | Regulatory compliance standard | Policy initiative |
| **Control** | Security control grouping | Policy set |
| **Assessment** | Recommendation | Policy definition |
| **Status** | Healthy / Unhealthy | Compliant / Non-compliant |

> ⚠️ **EXAM TIP:** Defender for Cloud regulatory compliance = **Azure Policy initiatives** under the hood. Adding a compliance standard = assigning a policy initiative.

---

## 17. Microsoft Defender for Cloud RBAC Roles

| Role | Permissions |
|---|---|
| **Security Reader** | View Defender for Cloud data (scores, alerts, recommendations). Read-only |
| **Security Admin** | View + update security policies, dismiss alerts, apply recommendations |
| **Contributor** | Can modify resources but NOT security policies |
| **Owner** | Full access including security policies |
| **Security Assessment Contributor** | Submit assessment results |

> ⚠️ **EXAM TIP:** **Security Reader** = read-only access to Defender data. **Security Admin** = read + write + dismiss alerts + update policies. Neither role manages Azure resources.

> ⚠️ **EXAM TIP:** **Security Admin** can: view recommendations, dismiss alerts, edit security policies. **Cannot**: create/manage Azure resources (need Contributor/Owner for that).

---

## 18. Azure Security — Additional Features

### 18.1 Microsoft Sentinel (Brief — AZ-104 Awareness)

| Feature | Description |
|---|---|
| **Type** | Cloud-native SIEM + SOAR |
| **Purpose** | Collect, detect, investigate, respond to threats |
| **Data sources** | Defender for Cloud, Azure AD, Office 365, firewalls |
| **Integration** | Receives alerts from Defender for Cloud via continuous export |

> ⚠️ **EXAM TIP:** Sentinel = SIEM (collects + analyzes security data). Defender for Cloud = CSPM + CWP (assesses + protects). They complement each other.

### 18.2 Azure DDoS Protection

| Tier | Features | Cost |
|---|---|---|
| **DDoS Network Protection** | Auto-tuned mitigation, alerts, metrics, DDoS Rapid Response | ~$2,944/month + overage |
| **DDoS IP Protection** | Per-IP protection, basic alerts | ~$199/month per IP |
| **Infrastructure Protection** | Basic (always-on, free) | ✅ Free |

#### Portal Path
```
Virtual Network → DDoS protection → Enable →
DDoS protection plan: Select / Create → Save
```

> ⚠️ **EXAM TIP:** **DDoS Infrastructure Protection** = always on, free, covers all Azure services. **DDoS Network/IP Protection** = paid, provides advanced features (metrics, alerting, rapid response).

### 18.3 Azure Firewall

| Feature | Details |
|---|---|
| **Type** | Managed cloud-based network firewall (PaaS) |
| **SKUs** | Basic, Standard, Premium |
| **Rules** | NAT rules, Network rules, Application rules |
| **Threat intelligence** | Alert/deny known malicious IPs/domains |
| **Integration** | Defender for Cloud adaptive network hardening |

### 18.4 Azure Bastion

| Feature | Details |
|---|---|
| **Purpose** | Secure RDP/SSH via browser (no public IP on VM) |
| **SKUs** | Basic, Standard, Premium |
| **Port** | Uses HTTPS (443) — no port 3389/22 exposure |
| **Exam relevance** | "Secure management access without public IP" → Bastion |

---

## 19. Network Security Best Practices (Exam Framework)

| Requirement | Solution |
|---|---|
| Block inbound internet to VMs | NSG deny rules |
| Allow management only when needed | **JIT VM access** |
| Secure RDP/SSH without public IP | **Azure Bastion** |
| Lock down NSGs based on traffic | **Adaptive Network Hardening** |
| Protect against DDoS | **DDoS Protection Plan** |
| Centralized firewall | **Azure Firewall** |
| Private access to PaaS | **Private Endpoints** |
| Reduce service exposure | **Service Endpoints** |

---

## 20. Monitoring & Alerts

### Defender for Cloud Dashboards

| Dashboard | Content |
|---|---|
| **Overview** | Secure score, active alerts, resource health |
| **Secure Score** | Detailed breakdown by control |
| **Recommendations** | All recommendations with status |
| **Security Alerts** | Active threats and incidents |
| **Regulatory Compliance** | Compliance posture per standard |
| **Workload Protections** | Status of Defender plans |
| **Inventory** | All discovered resources with security state |

#### Portal Path — Overview Dashboard
```
Defender for Cloud → Overview
Defender for Cloud → Security Alerts → Filter → Drill down
Defender for Cloud → Inventory → Filter by: unhealthy / unmonitored
```

### Alert Notifications

#### Email Notifications
```
Defender for Cloud → Environment settings → Select subscription →
Email notifications →
Email address: admin@company.com →
Notification types: High severity alerts ✅ →
Also notify: Subscription owners ✅ / Contributors ✅ →
Save
```

> ⚠️ **EXAM TIP:** Email notifications are configured per-subscription. Can notify specific emails + subscription owners/contributors. Default = subscription owner gets high-severity alerts.

---

## 21. Pricing Key Points

| Component | Cost |
|---|---|
| **Free tier (CSPM)** | ✅ Free — Secure Score, Recommendations, ASB |
| **Defender for Servers P1** | ~$5/server/month |
| **Defender for Servers P2** | ~$15/server/month (JIT, FIM, vulnerability) |
| **Defender for Storage** | ~$10/storage account/month or per-transaction |
| **Defender for SQL** | ~$15/server/month |
| **Defender for App Service** | ~$15/App Service instance/month |
| **Defender for Key Vault** | ~$0.02/10K transactions |
| **Defender for ARM** | ~$4/subscription/month |
| **Defender for DNS** | ~$0.7/million queries |
| **Defender CSPM** | ~$5/server/month (enhanced posture) |
| **30-day free trial** | ✅ All plans included |

> ⚠️ **EXAM TIP:** Defender for Cloud offers a **30-day free trial** for all Defender plans. After trial, each plan billed individually per resource.

> ⚠️ **EXAM TIP:** Free tier is **always free** — no trial needed. Paid plans can be enabled/disabled individually per subscription.

---

## 22. Limitations & Key Constraints

| Constraint | Detail |
|---|---|
| Secure Score update | Every **12 hours** |
| Partial credit | ❌ No — all recommendations in control must be fixed |
| JIT access | Requires **Defender for Servers** |
| FIM | Requires **Defender for Servers P2** |
| Multi-cloud | Supported (AWS, GCP) with connectors |
| Custom policies | ✅ Supported (custom initiative) |
| ASB removal | ❌ Cannot remove default ASB initiative |
| Alert suppression | ✅ Supported (suppression rules) |
| Continuous export targets | Log Analytics or Event Hub only |
| Workflow automation | Uses Logic Apps only |

---

## 23. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| List Defender settings | `az security pricing list -o table` |
| Enable Defender for Servers | `az security pricing create -n VirtualMachines --tier Standard` |
| Disable Defender plan | `az security pricing create -n VirtualMachines --tier Free` |
| List security alerts | `az security alert list -o table` |
| Get secure score | `az security secure-scores list -o table` |
| List recommendations | `az security assessment list -o table` |
| List contacts | `az security contact list` |
| Set email notification | `az security contact create -n default --email admin@co.com --alert-notifications on --alerts-admins on` |
| List JIT policies | `az security jit-policy list` |

### PowerShell

| Action | Command |
|---|---|
| Get pricing tier | `Get-AzSecurityPricing` |
| Set Defender plan | `Set-AzSecurityPricing -Name "VirtualMachines" -PricingTier "Standard"` |
| Get alerts | `Get-AzSecurityAlert` |
| Get secure score | `Get-AzSecuritySecureScore` |
| Get recommendations | `Get-AzSecurityAssessment` |
| Set contact | `Set-AzSecurityContact -Name "default" -Email "admin@co.com" -AlertAdmin -NotifyAdmin` |

---

## 24. Quick-Fire Exam Points ⚡

1. Defender for Cloud = **CSPM** (free) + **CWP** (paid Defender plans)
2. **Free tier**: Secure Score + Recommendations + Azure Security Benchmark — always free
3. **Paid plans**: Threat detection, JIT access, FIM, vulnerability assessment, adaptive controls
4. **Azure Security Benchmark (ASB)** = auto-assigned policy initiative to every subscription — cannot remove
5. Additional compliance standards (PCI, ISO, NIST) require **paid Defender plans**
6. **Secure Score** = 0–100%, **no partial credit** per control — all recommendations in control must be fixed
7. Recommendations refresh every **12 hours**
8. **Exempting** a recommendation removes it from score calculation (numerator + denominator)
9. **Quick Fix** remediation = one-click automated fix (not available for all recommendations)
10. **JIT VM access** = locks management ports, opens temporarily on request — requires **Defender for Servers**
11. JIT creates/removes **NSG rules** dynamically — Deny when locked, Allow when requested
12. Default JIT max request time = **3 hours**
13. **Adaptive Application Controls** = application whitelisting (ML-based)
14. **Adaptive Network Hardening** = ML-based NSG rule recommendations
15. **File Integrity Monitoring** = monitors OS/registry/app file changes — requires **Defender for Servers P2**
16. Security alerts = **paid plans only**. Alert severity: High / Medium / Low / Informational
17. **Suppression rules** filter out false positive alerts
18. **Workflow automation** uses **Logic Apps** (not Functions, not Runbooks)
19. **Continuous export** sends data to Log Analytics or Event Hub (for SIEM)
20. **Security Reader** = read-only Defender access. **Security Admin** = read + write + dismiss
21. **Security Admin** CANNOT manage Azure resources — only security policies/alerts
22. Email notifications: configured per-subscription, can notify owners + specific emails
23. **30-day free trial** for all Defender plans
24. Defender for Cloud integrates with **Azure Policy** — each recommendation = a policy definition
25. Adding a compliance standard = assigning a **policy initiative**
26. **Sentinel** = SIEM + SOAR. **Defender for Cloud** = CSPM + CWP. They complement each other
27. DDoS Infrastructure Protection = free, always on. DDoS Network/IP Protection = paid, advanced features
28. "Reduce attack surface for RDP/SSH" → **JIT**. "Secure access without public IP" → **Bastion**
29. Defender for Cloud works with **multi-cloud** (AWS, GCP) via environment connectors
30. Defender plans can be enabled/disabled **individually per subscription**

---

## 25. Step-by-Step Configuration Mind Maps 🗺️

---

### 25.1 Enable Defender Plans on a Subscription

> **Portal:** `Defender for Cloud → Environment settings → Select subscription`

```
Enable Defender Plans
│
├── Portal: Defender for Cloud → Environment settings →
│   Select subscription → Defender plans
│
├── RBAC Required: Security Admin or Owner on subscription
│
├── Available Plans (toggle On/Off individually):
│   │
│   ├── Defender for Servers
│   │   ├── Plan 1 (~$5/server/month): Basic threat detection
│   │   └── Plan 2 (~$15/server/month): JIT, FIM, vulnerability assessment, adaptive controls
│   │   ⚠️ JIT access requires Plan 2 or higher
│   │
│   ├── Defender for Storage
│   │   └── Malware scanning, sensitive data threat detection
│   │
│   ├── Defender for SQL
│   │   └── Azure SQL + SQL Server on VMs
│   │
│   ├── Defender for App Service
│   │   └── Threat detection for web apps
│   │
│   ├── Defender for Key Vault
│   │   └── Unusual access, suspicious operations
│   │
│   ├── Defender for ARM
│   │   └── Suspicious management plane operations
│   │
│   ├── Defender for DNS
│   │   └── Malicious DNS activity
│   │
│   ├── Defender for Containers
│   │   └── AKS runtime + image scanning
│   │
│   ├── Defender for Databases
│   │   └── Cosmos DB, MySQL, PostgreSQL, MariaDB
│   │
│   └── Defender CSPM (Enhanced)
│       └── Attack path analysis, agentless scanning, governance
│
├── Save
│   ⚠️ 30-day free trial available for first-time enablement
│   ⚠️ Each plan billed individually per resource after trial
│
└── ⚠️ Notes
    ├── Free tier (CSPM basic) = always free, cannot disable
    ├── Plans can be enabled/disabled independently
    ├── Takes a few hours for full assessment after enabling
    └── Enabling at subscription = covers ALL resources of that type
```

---

### 25.2 Configure Just-In-Time VM Access

> **Portal:** `Defender for Cloud → Workload protections → JIT VM access`

```
Configure JIT VM Access
│
├── Prerequisites
│   ├── Defender for Servers Plan 2 enabled on subscription
│   │   ⚠️ JIT NOT available in free tier
│   ├── VM has NSG or Azure Firewall associated
│   └── RBAC: Security Admin (configure) / Security Reader (request access)
│
├── Step 1: Enable JIT on VM
│   │
│   ├── Method 1: Defender for Cloud
│   │   │   Defender for Cloud → Workload protections → JIT VM access
│   │   ├── Not configured tab → Select VM(s) → Enable JIT
│   │   ├── Configure ports:
│   │   │   ├── Port 22 (SSH): Max time 3h, Allowed: Any / My IP / CIDR
│   │   │   ├── Port 3389 (RDP): Max time 3h, Allowed: Any / My IP / CIDR
│   │   │   ├── Port 5985 (WinRM): Max time 3h
│   │   │   ├── Port 5986 (WinRM-S): Max time 3h
│   │   │   └── + Add custom port
│   │   └── Save
│   │
│   └── Method 2: VM Blade
│       VM → Settings → Connect → ⚡ Enable JIT → Configure
│
├── Step 2: Request Access (When Needed)
│   │   Defender for Cloud → Workload protections → JIT VM access →
│   │   Configured tab → Select VM → Request access
│   ├── Toggle ports to open: ✅ 3389
│   ├── Source IP:
│   │   ├── My IP (auto-detected)
│   │   ├── IP range (CIDR)
│   │   └── Per port configuration
│   ├── Time range: 1 hour (max 3 hours default, max 24h configurable)
│   └── Open ports
│       ⚠️ NSG Allow rule auto-added with higher priority
│       ⚠️ After time expires → Allow rule auto-removed
│
│   CLI:
│   az security jit-policy show -g <rg> --name <vm> -l <location>
│
├── Step 3: Connect to VM
│   ├── RDP: mstsc /v:<VM-IP>:3389
│   ├── SSH: ssh user@<VM-IP>
│   └── ⚠️ Must connect within the approved time window
│
├── What Happens in NSG:
│   │
│   ├── JIT Enabled (no active request):
│   │   Rule: Deny TCP 3389 from Any → Priority 1000
│   │
│   ├── Access Requested and Approved:
│   │   Rule: Allow TCP 3389 from <My-IP> → Priority 100 (higher than deny)
│   │   ⚠️ Allow rule removed automatically after time expires
│   │
│   └── Access Expired:
│       Rule: Allow deleted → Deny rule remains active
│
└── ⚠️ Exam Notes
    ├── JIT = exam favorite for "reduce attack surface"
    ├── Requires Defender for Servers (paid)
    ├── Works by manipulating NSG rules dynamically
    ├── Default max time = 3 hours (configurable up to 24h)
    ├── Can configure approval workflow (Azure AD role-based)
    └── Alternative to exposing RDP/SSH permanently
```

---

### 25.3 Configure Email Notifications

> **Portal:** `Defender for Cloud → Environment settings → Email notifications`

```
Configure Email Notifications
│
├── Portal: Defender for Cloud → Environment settings →
│   Select subscription → Email notifications
│
├── Settings:
│   ├── Email recipients:
│   │   ├── Additional email addresses: admin@company.com; soc@company.com
│   │   │   (semicolon-separated)
│   │   └── All users with the following roles:
│   │       ├── ✅ Account owner
│   │       ├── ✅ Service admin
│   │       ├── ✅ Contributor
│   │       └── Custom roles
│   │
│   ├── Notification types:
│   │   ├── Notify about alerts with the following severity:
│   │   │   ├── ✅ High (recommended minimum)
│   │   │   ├── ✅ Medium
│   │   │   └── ✅ Low
│   │   └── ⚠️ Always enable at least High severity
│   │
│   └── Save
│
└── ⚠️ Notes
    ├── Per-subscription configuration
    ├── Default: subscription owner gets high-severity alerts
    ├── Add SOC/security team emails for operational coverage
    └── Can also use Workflow Automation for richer integrations
```

---

### 25.4 Add Regulatory Compliance Standard

> **Portal:** `Defender for Cloud → Regulatory compliance → Manage compliance policies`

```
Add Compliance Standard
│
├── Prerequisites
│   ├── Defender plans enabled (additional standards require paid plans)
│   │   ⚠️ ASB (Azure Security Benchmark) is free and auto-assigned
│   └── RBAC: Security Admin or Owner
│
├── Step 1: Navigate
│   Portal: Defender for Cloud → Regulatory compliance →
│   Manage compliance policies → Select subscription
│
├── Step 2: Add Standard
│   ├── Industry & regulatory standards section
│   ├── Available standards:
│   │   ├── PCI DSS 3.2.1 / 4.0
│   │   ├── ISO 27001:2013
│   │   ├── SOC 2 Type 2
│   │   ├── NIST SP 800-53 Rev 4/5
│   │   ├── HIPAA HITRUST
│   │   ├── CIS Azure Benchmark
│   │   └── Others (region-specific)
│   ├── + Add more standards → Select → Add
│   └── ⚠️ Each standard = policy initiative assigned to subscription
│
├── Step 3: View Compliance
│   │   Defender for Cloud → Regulatory compliance
│   ├── Select standard → View controls
│   ├── Each control: Passed / Failed assessments
│   ├── Select failed assessment → View affected resources
│   └── Remediate → Follow recommendation steps
│
├── Step 4: Export Compliance Report
│   ├── Defender for Cloud → Regulatory compliance →
│   │   Download report → Select standard → PDF / CSV
│   └── Use for: audit evidence, management reporting
│
└── ⚠️ Notes
    ├── Standards mapped to Azure Policy initiatives
    ├── ASB always present — cannot remove
    ├── Adding standard = assigning policy initiative
    ├── Non-compliant controls = policy evaluation failures
    └── Some controls require manual attestation (no auto-assessment)
```

---

### 25.5 Configure Continuous Export

> **Portal:** `Defender for Cloud → Environment settings → Continuous export`

```
Configure Continuous Export
│
├── Portal: Defender for Cloud → Environment settings →
│   Select subscription → Continuous export
│
├── Export Target Options:
│   │
│   ├── Option 1: Log Analytics Workspace
│   │   ├── Select workspace (or create new)
│   │   └── Data available via KQL queries in Log Analytics
│   │
│   └── Option 2: Event Hub
│       ├── Select Event Hub namespace + Event Hub
│       └── For: SIEM integration (Sentinel, Splunk, QRadar)
│
├── Data Types to Export:
│   ├── ✅ Security recommendations
│   ├── ✅ Secure score & secure score controls
│   ├── ✅ Security alerts
│   ├── ✅ Regulatory compliance
│   └── ✅ Attack paths (Defender CSPM)
│
├── Export Frequency:
│   ├── Streaming updates: Real-time as changes occur
│   └── Snapshots: Periodic snapshot of current state
│
├── Export Scope:
│   ├── Selected subscription
│   └── All recommendations / specific severity filter
│
├── Save
│
└── ⚠️ Notes
    ├── Only two targets: Log Analytics or Event Hub
    ├── For Sentinel integration: use Log Analytics connector
    ├── Enable for SIEM, long-term retention, custom dashboards
    ├── Streaming = near real-time. Snapshots = periodic
    └── RBAC: Security Admin to configure
```

---

### 25.6 Set Up Workflow Automation

> **Portal:** `Defender for Cloud → Workflow automation → + Add`

```
Set Up Workflow Automation
│
├── Prerequisites
│   ├── Logic App exists (or create during setup)
│   │   ⚠️ Only Logic Apps supported — not Functions or Runbooks
│   ├── Logic App has appropriate trigger/actions configured
│   └── RBAC: Security Admin + Logic App Contributor
│
├── Portal: Defender for Cloud → Environment settings →
│   Select subscription → Workflow automation → + Add workflow automation
│
├── Configuration:
│   ├── Name: "high-alert-notification"
│   ├── Trigger type:
│   │   ├── When a security alert is created or triggered
│   │   │   Filter: Severity ≥ Medium, Alert name contains "..."
│   │   ├── When a recommendation is created or triggered
│   │   │   Filter: Specific recommendation, severity
│   │   └── When regulatory compliance assessment changes
│   │       Filter: Standard, control
│   │
│   ├── Actions:
│   │   ├── Logic App: Select from subscription
│   │   └── Logic App examples:
│   │       ├── Send email via Office 365 connector
│   │       ├── Post to Teams channel
│   │       ├── Create ServiceNow ticket
│   │       ├── Trigger Azure Function for remediation
│   │       └── Send to Slack webhook
│   │
│   └── Create
│
├── Verify:
│   ├── Trigger a test alert (or wait for real alert)
│   ├── Check Logic App run history
│   └── Verify notification received
│
└── ⚠️ Notes
    ├── Workflow automation = Logic Apps ONLY
    ├── Can create multiple automations for different triggers
    ├── Filter by severity to avoid alert fatigue
    ├── Logic App must be in same subscription
    └── Common exam scenario: "auto-notify SOC team on high-severity alert"
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
