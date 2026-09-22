<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Automation — AZ-104 Revision Notes

---

## 1. What is Azure Automation?

- Cloud-based **automation and configuration management** service
- Automate: repetitive tasks, VM management, patching, compliance, resource lifecycle
- Key features: **Runbooks** (PowerShell/Python scripts), **State Configuration (DSC)**, **Update Management**, **Change Tracking**, **Inventory**
- Runs scripts in Azure sandbox or on **Hybrid Runbook Workers** (on-prem/other clouds)

---

## 2. Key Components

| Component | Description |
|---|---|
| **Automation Account** | Container for all automation resources (runbooks, credentials, schedules) |
| **Runbooks** | Scripts that execute automation tasks (PowerShell, Python, Graphical) |
| **Hybrid Runbook Worker** | Agent on on-prem/Azure VM to run runbooks locally |
| **Schedules** | Time-based triggers for runbooks |
| **Variables** | Shared config values accessible by runbooks |
| **Credentials** | Stored username/password for runbook authentication |
| **Connections** | Pre-defined connection objects (service principals, etc.) |
| **Certificates** | X.509 certificates for authentication |
| **Modules** | PowerShell/Python modules available to runbooks |
| **Webhooks** | HTTP endpoints to trigger runbooks externally |
| **State Configuration (DSC)** | Desired State Configuration for VMs |
| **Update Management** | OS patching for Azure/hybrid VMs |
| **Change Tracking & Inventory** | Track file, registry, software, service changes |

---

## 3. Runbook Types

| Runbook Type | Language | Execution | Use Case |
|---|---|---|---|
| **PowerShell** | PowerShell 5.1 / 7.2 | Azure sandbox or Hybrid Worker | Most common — Azure resource management |
| **PowerShell Workflow** | PowerShell Workflow | Azure sandbox or Hybrid Worker | Parallel execution, checkpoints (legacy) |
| **Python** | Python 2 / 3 | Azure sandbox or Hybrid Worker | Cross-platform, Linux automation |
| **Graphical** | Drag-and-drop UI | Azure sandbox or Hybrid Worker | Visual authoring (no code) |
| **Graphical PowerShell Workflow** | Graphical + workflow | Azure sandbox or Hybrid Worker | Visual + checkpoints (legacy) |

> ⚠️ **EXAM TIP:** **PowerShell Workflow** supports **checkpoints** and **parallel execution** — standard PowerShell runbook does NOT. If exam asks about "long-running scripts with checkpoints" → PowerShell Workflow.

> ⚠️ **EXAM TIP:** **Graphical runbooks** = visual drag-and-drop authoring. Cannot be edited as text. Created/edited only in the portal.

### Runbook Execution Environment

| Environment | Description |
|---|---|
| **Azure Sandbox** | Multi-tenant, shared Azure infrastructure |
| **Hybrid Runbook Worker** | Your VM (Azure or on-prem) running the agent |

> ⚠️ **EXAM TIP:** Azure sandbox has **limitations**: max **3 hours** fair-share runtime, no access to on-prem resources, limited disk/memory. Use **Hybrid Runbook Worker** for: longer jobs, on-prem access, resource-intensive tasks.

---

## 4. Runbook Execution Limits (Azure Sandbox)

| Limit | Value |
|---|---|
| **Max runtime (fair share)** | **3 hours** |
| **Max output stream** | **1 MB** |
| **Max memory per sandbox** | **400 MB** |
| **Max concurrent jobs per account** | **Varies** (Basic: unlimited queued) |
| **Max child runbooks (nesting)** | **Depth of 5** |
| **Max module import** | **Varies** |
| **Runbook retention** | Job logs retained for **30 days** |

> ⚠️ **EXAM TIP:** **3-hour fair share** limit on Azure sandbox. After 3 hours, runbook is suspended and restarted. Use Hybrid Worker for jobs > 3 hours (no fair share limit on Hybrid Worker).

> ⚠️ **EXAM TIP:** Runbook job output = max **1 MB**. If exceeded, output is truncated. Use `Write-Output` sparingly; write to Log Analytics for large outputs.

---

## 5. Azure Sandbox vs Hybrid Runbook Worker

| Feature | **Azure Sandbox** | **Hybrid Runbook Worker** |
|---|---|---|
| **Infrastructure** | Azure-managed (shared) | Your VM (Azure/on-prem/other cloud) |
| **Max runtime** | **3 hours** (fair share) | **No limit** |
| **Access to on-prem** | ❌ | ✅ |
| **Access to Azure resources** | ✅ (via managed identity) | ✅ (via managed identity / Run As) |
| **Agent required** | ❌ | ✅ (Log Analytics agent or Extension-based) |
| **OS support** | N/A (cloud) | Windows / Linux |
| **Cost** | Included (job minutes) | VM cost + Free HRW feature |
| **Use case** | Cloud-only Azure tasks | On-prem access, long-running, resource-heavy |
| **Disk access** | ❌ Limited | ✅ Local disk |
| **Network** | Azure public only | Your network (VNet, on-prem) |

> ⚠️ **EXAM TIP:** "Run script against on-premises servers" → **Hybrid Runbook Worker**. "Run script > 3 hours" → **Hybrid Runbook Worker**. "Cloud-only Azure task" → **Azure Sandbox** is fine.

---

## 6. Hybrid Runbook Worker

### Types

| Type | Description |
|---|---|
| **System Hybrid Worker** | Installed on VM for Update Management — auto-created |
| **User Hybrid Worker** | Manually deployed for custom runbook execution |

### Deployment Methods

| Method | Agent |
|---|---|
| **Extension-based** (recommended) | Azure VM extension (`HybridWorker`) — no Log Analytics needed |
| **Agent-based** (legacy) | Log Analytics agent + Automation Hybrid Worker solution |

#### Portal Path — Add Hybrid Worker
```
Automation Account → Process Automation → Hybrid worker groups →
+ Create hybrid worker group →
Name: my-worker-group →
Use Hybrid Worker Credentials: (optional Run As) →
+ Add machines → Select VMs → Add → Create
```

#### CLI
```bash
# Extension-based (recommended)
az automation hrwg create --automation-account-name <acct> \
  --resource-group <rg> --name <group-name>

az automation hrwg hrw create --automation-account-name <acct> \
  --resource-group <rg> --hybrid-runbook-worker-group-name <group> \
  --name <worker-id> --vm-resource-id <vm-resource-id>
```

> ⚠️ **EXAM TIP:** **Extension-based** Hybrid Worker = recommended (no Log Analytics agent needed). **Agent-based** = legacy approach. Exam may reference both.

---

## 7. Runbook Authentication

### Methods

| Method | Description | Recommended? |
|---|---|---|
| **System-assigned Managed Identity** | Auto-created identity for Automation Account | ✅ Recommended |
| **User-assigned Managed Identity** | Shared identity across multiple resources | ✅ Recommended |
| **Run As Account** | Service principal + certificate (deprecated Jan 2025) | ❌ Deprecated |
| **Credentials** | Stored username/password | For Hybrid Worker / on-prem |
| **Certificates** | Certificate-based authentication | For specific scenarios |

> ⚠️ **EXAM TIP:** **Run As Account is DEPRECATED** (retired Sep 2023). Microsoft now recommends **Managed Identity** (system or user-assigned) for Automation Account authentication.

> ⚠️ **EXAM TIP:** To authenticate a runbook to Azure: (1) Enable Managed Identity on Automation Account, (2) Assign RBAC role (e.g., Contributor) to the managed identity, (3) Use `Connect-AzAccount -Identity` in runbook.

#### Portal Path — Enable Managed Identity
```
Automation Account → Account Settings → Identity →
System assigned: Status = On → Save → Yes →
Copy Object ID → Assign RBAC at resource/RG/subscription scope
```

---

## 8. Shared Resources

### 8.1 Variables

| Feature | Details |
|---|---|
| **Types** | String, Integer, Boolean, DateTime, Null, Object (complex) |
| **Encrypted** | ✅ Optional — cannot be read back once encrypted |
| **Access in runbook** | `Get-AutomationVariable`, `Set-AutomationVariable` |

#### Portal Path
```
Automation Account → Shared Resources → Variables →
+ Add a variable → Name, Type, Value, Encrypted → Create
```

> ⚠️ **EXAM TIP:** **Encrypted variables** = cannot retrieve value from portal or cmdlets (write-only). Unencrypted = readable. Use encrypted for sensitive config values.

### 8.2 Credentials

| Feature | Details |
|---|---|
| **Stores** | Username + Password (PSCredential object) |
| **Encrypted** | ✅ Always encrypted at rest |
| **Access** | `Get-AutomationPSCredential -Name <name>` |

#### Portal Path
```
Automation Account → Shared Resources → Credentials →
+ Add a credential → Name, Username, Password → Create
```

### 8.3 Certificates

| Feature | Details |
|---|---|
| **Format** | .cer or .pfx |
| **Use** | RunAs account auth (legacy), certificate-based access |
| **Exportable** | PFX can be exported; CER = public only |

#### Portal Path
```
Automation Account → Shared Resources → Certificates →
+ Add a certificate → Name, Upload .pfx/.cer, Password → Create
```

### 8.4 Connections

| Feature | Details |
|---|---|
| **Types** | Azure (service principal), Azure Classic, etc. |
| **Use** | Predefined connection parameters for runbooks |
| **Deprecated** | ⚠️ Used by Run As Account (deprecated) |

### 8.5 Modules

| Feature | Details |
|---|---|
| **Default** | Az modules (Az.Accounts, Az.Compute, etc.) |
| **Custom** | Upload .zip from PowerShell Gallery or local |
| **Import** | `Az.Accounts` must be imported first (dependency) |

#### Portal Path
```
Automation Account → Shared Resources → Modules →
+ Add a module → Browse Gallery / Upload file →
Runtime version: 5.1 / 7.2 → Import
```

> ⚠️ **EXAM TIP:** **Az.Accounts** must be imported FIRST — all other Az modules depend on it. If runbook fails with "module not found" → check module imports.

---

## 9. Schedules

- Time-based triggers to start runbooks automatically
- One-time or recurring (hourly, daily, weekly, monthly)
- Can link **multiple schedules** to one runbook
- Can link **one schedule** to multiple runbooks
- Supports **parameters** passed at schedule-runbook link

| Setting | Options |
|---|---|
| **Frequency** | Once, Hourly, Daily, Weekly, Monthly |
| **Recurrence** | Every X hours/days/weeks/months |
| **Start time** | Minimum **5 minutes** from creation |
| **Expiration** | Optional end date (or never) |
| **Time zone** | Configured per schedule |

#### Portal Path — Create Schedule
```
Automation Account → Shared Resources → Schedules →
+ Add a schedule → Name →
Starts: date/time → Time zone →
Recurrence: Once / Recurring →
Recur every: X hours/days/weeks/months →
Set expiration (optional) → Create
```

#### Portal Path — Link Schedule to Runbook
```
Automation Account → Runbooks → Select runbook →
Schedules → + Add a schedule →
Link a schedule to your runbook: Select existing / Create new →
Configure parameters → OK
```

> ⚠️ **EXAM TIP:** Schedule start time must be **at least 5 minutes** in the future. Cannot schedule in the past.

> ⚠️ **EXAM TIP:** A runbook can have **multiple schedules**. A schedule can trigger **multiple runbooks**. Many-to-many relationship.

---

## 10. Webhooks

- **HTTP POST endpoint** to trigger a runbook from external sources
- Unique URL generated at creation — **cannot be retrieved later**
- Supports: CI/CD pipelines, monitoring alerts, third-party integrations
- Max expiry: **10 years** (default: 1 year)

| Feature | Details |
|---|---|
| **URL** | Unique, generated once — copy immediately |
| **Method** | HTTP POST only |
| **Auth** | URL contains token (no separate auth needed) |
| **Input** | Request body = runbook parameter (JSON) |
| **Expiry** | Configurable (max 10 years) |
| **Disable** | Can enable/disable without deleting |

#### Portal Path
```
Automation Account → Runbooks → Select runbook →
Webhooks → + Add webhook →
Create new webhook → Name →
⚠️ COPY THE URL NOW — cannot retrieve later →
Expiry date → Enabled: Yes →
Configure parameters → OK → Create
```

> ⚠️ **EXAM TIP:** Webhook URL = **one-time view** at creation. If you lose it, must create a new webhook. URL contains the auth token — treat as secret.

> ⚠️ **EXAM TIP:** Webhook input goes to the runbook as `$WebhookData` parameter. The body is in `$WebhookData.RequestBody`.

---

## 11. Update Management

- Assess and deploy **OS patches** to Azure VMs, on-prem VMs, other clouds
- Supports: **Windows** (Windows Update / WSUS) and **Linux** (apt/yum/zypper)
- Uses **Log Analytics workspace** for assessment data
- Schedule-based: define maintenance windows and update classifications

### Components

| Component | Role |
|---|---|
| **Log Analytics agent** | Collects update assessment data from VMs |
| **Automation Account** | Orchestrates update deployment |
| **Log Analytics workspace** | Stores assessment results |
| **Update deployment** | Scheduled update installation |

### Update Classifications

| Windows | Linux |
|---|---|
| Critical Updates | Critical Updates |
| Security Updates | Security Updates |
| Update Rollups | Other Updates |
| Feature Packs | — |
| Service Packs | — |
| Definition Updates | — |
| Tools | — |
| Updates | — |

#### Portal Path — Enable Update Management
```
Automation Account → Update Management →
Enable → Select Log Analytics workspace →
Select subscription and region → Enable
```

#### Portal Path — Schedule Update Deployment
```
Automation Account → Update Management →
Schedule update deployment →
Name: monthly-patches →
OS: Windows / Linux →
Machines: Select VMs →
Update classifications: Critical ✅, Security ✅ →
Exclude updates: optional KB IDs →
Schedule: Recurring monthly →
Maintenance window: 120 minutes (default) →
Reboot options: If required / Never / Always →
Pre/post scripts: optional runbooks →
Create
```

> ⚠️ **EXAM TIP:** Update Management **maintenance window** = max **6 hours** (default 2 hours = 120 min). Updates installation stops at 20 minutes before window end to allow reboot.

> ⚠️ **EXAM TIP:** Update Management requires: **Log Analytics workspace** + **Log Analytics agent** on VMs + **Automation Account** linked to workspace.

> ⚠️ **EXAM TIP:** Update Management is being replaced by **Azure Update Manager** (no Automation Account needed). AZ-104 may test both — Update Manager is agent-based (Azure VM extension), Update Management is legacy.

---

## 12. Azure Update Manager (New — Replacement)

| Feature | **Update Management (Legacy)** | **Azure Update Manager (New)** |
|---|---|---|
| **Dependency** | Automation Account + Log Analytics | ❌ No dependencies (standalone) |
| **Agent** | Log Analytics agent | Azure VM guest agent / Arc agent |
| **Scope** | Azure + Hybrid | Azure + Hybrid + multi-cloud |
| **Scheduling** | Via Automation Account | Maintenance configurations |
| **Portal** | Automation Account blade | VM blade + Azure Update Manager |
| **Assessment** | Periodic (agent reports) | On-demand + periodic |
| **Recommended** | ❌ Legacy | ✅ Microsoft recommended |

#### Portal Path — Azure Update Manager
```
VM → Operations → Updates → Assess now / Install one-time / Create maintenance configuration
```

```
Home → Azure Update Manager → Overview → Machines →
Select VMs → Check for updates / One-time update / Schedule updates
```

> ⚠️ **EXAM TIP:** **Azure Update Manager** = modern replacement (no Automation Account needed). But AZ-104 still tests legacy **Update Management** (with Automation Account). Know both.

---

## 13. Change Tracking & Inventory

### Change Tracking
- Tracks changes to: **files**, **Windows registry**, **Windows services**, **Linux daemons**, **software**
- Alerts when unauthorized changes occur
- Uses: Log Analytics agent + Log Analytics workspace

### Inventory
- Collects installed **software**, **files**, **Windows services**, **Linux daemons**, **registry keys**
- Browse per-VM inventory in portal

#### Portal Path
```
Automation Account → Configuration Management →
Change tracking → Enable → Select workspace → Enable

Automation Account → Configuration Management →
Inventory → Enable
```

#### Portal Path — Configure Tracked Items
```
Automation Account → Change tracking → Edit Settings →
Windows Files / Linux Files / Windows Registry / File Content →
+ Add → Path, Recursion, Upload content → Save
```

| Tracked Item | Windows | Linux |
|---|---|---|
| Files | ✅ | ✅ |
| Registry | ✅ | ❌ |
| Software | ✅ (.msi, .exe) | ✅ (rpm, dpkg) |
| Services/Daemons | ✅ (Windows services) | ✅ (Linux daemons) |
| File content | ✅ | ✅ |

> ⚠️ **EXAM TIP:** Change Tracking monitors **changes over time**. Inventory shows **current state**. Both use the same Log Analytics workspace.

---

## 14. State Configuration (DSC)

- **Desired State Configuration** — ensure VMs maintain a specific configuration state
- Define **what** the machine should look like (packages, files, services, registry)
- Azure Automation = **pull server** — nodes pull configuration periodically
- Supports: Windows + Linux (Azure + on-prem via Hybrid)

### Key Concepts

| Concept | Description |
|---|---|
| **Configuration** | PowerShell DSC script defining desired state |
| **Node Configuration** | Compiled configuration (MOF file) |
| **Node** | Managed VM registered with Automation DSC |
| **Pull server** | Azure Automation acts as pull endpoint |
| **Compliance** | Node's actual state vs desired state |
| **Configuration Mode** | ApplyOnly / ApplyAndMonitor / ApplyAndAutoCorrect |

### Configuration Modes

| Mode | Behavior |
|---|---|
| **ApplyOnly** | Apply config once. No monitoring. No re-apply |
| **ApplyAndMonitor** | Apply config + report drift but don't fix |
| **ApplyAndAutoCorrect** | Apply config + detect drift + **automatically fix** |

#### Portal Path — Add DSC Configuration
```
Automation Account → State Configuration (DSC) →
Configurations tab → + Add → Upload .ps1 DSC script → Import
```

#### Portal Path — Compile Configuration
```
Automation Account → State Configuration (DSC) →
Configurations → Select config → Compile → Yes
→ Creates Node Configuration (MOF)
```

#### Portal Path — Register Node
```
Automation Account → State Configuration (DSC) →
Nodes tab → + Add →
Select VM → Connect →
Node configuration: Select compiled config →
Configuration mode: ApplyAndAutoCorrect →
Reboot if needed: Yes / No →
Frequency: 15 min (default) →
OK
```

> ⚠️ **EXAM TIP:** **ApplyAndAutoCorrect** = most strict mode — detects drift and automatically remediates. Exam loves this. ApplyAndMonitor = detects but does NOT fix.

> ⚠️ **EXAM TIP:** DSC configuration must be **compiled** before it can be assigned to nodes. Compilation produces a **MOF (Managed Object Format)** file.

> ⚠️ **EXAM TIP:** Default **refresh frequency** = **30 minutes**. Default **configuration frequency** = **15 minutes**. Node checks pull server every 15 min.

---

## 15. Source Control Integration

- Sync runbooks from **GitHub**, **Azure DevOps (TFVC or Git)**
- Auto-sync or manual sync
- Version control for runbooks

#### Portal Path
```
Automation Account → Source Control → + Add →
Source type: GitHub / Azure DevOps (Git) / Azure DevOps (TFVC) →
Repository: Select → Branch →
Folder path: /runbooks →
Auto sync: ✅ / ❌ →
OK
```

---

## 16. Security & RBAC

### Built-in Roles

| Role | Permissions |
|---|---|
| **Automation Operator** | Start, stop, resume, suspend jobs. Read runbooks. Cannot edit |
| **Automation Runbook Operator** | Read runbook properties. Start runbook jobs |
| **Automation Job Operator** | Create and manage jobs |
| **Automation Contributor** | Full management of Automation Account (CRUD runbooks, schedules, etc.) |
| **Reader** | View Automation Account and resources |
| **Contributor** | Full access at resource level |

> ⚠️ **EXAM TIP:** **Automation Operator** = can START/STOP runbooks but CANNOT edit or create them. "User needs to run runbooks but not modify" → Automation Operator.

> ⚠️ **EXAM TIP:** **Automation Contributor** is NOT a default Azure built-in role — it grants full control over Automation Account resources.

### Managed Identity Permissions

| Pattern | Steps |
|---|---|
| Runbook manages Azure resources | 1. Enable managed identity on Automation Account → 2. Assign RBAC role (e.g., VM Contributor) at required scope → 3. Use `Connect-AzAccount -Identity` in runbook |

---

## 17. Monitoring & Diagnostics

### Job Monitoring

#### Portal Path
```
Automation Account → Process Automation → Jobs →
Filter: Status / Runbook name / Time range →
Select job → Output / Errors / Warnings / All logs
```

### Job Statuses

| Status | Description |
|---|---|
| **New** | Job created, not yet queued |
| **Queued** | Waiting for worker |
| **Starting** | Worker picked up, initializing |
| **Running** | Executing |
| **Completed** | Finished successfully |
| **Failed** | Error occurred |
| **Stopped** | Manually stopped |
| **Suspended** | Suspended (checkpoint for PowerShell Workflow) |
| **Disconnected** | Hybrid Worker lost connection |

### Diagnostic Logs

#### Portal Path
```
Automation Account → Monitoring → Diagnostic settings →
+ Add diagnostic setting →
Logs: JobLogs ✅, JobStreams ✅, DscNodeStatus ✅, AuditEvent ✅ →
Destination: Log Analytics / Storage / Event Hub →
Save
```

### Activity Log

#### Portal Path
```
Automation Account → Monitoring → Activity log →
Filter: operations, timespan
```

---

## 18. Pricing Key Points

| Component | Cost |
|---|---|
| **Job runtime** | **500 free minutes/month** (Azure sandbox). Then ~$0.002/minute |
| **Watchers** | ~$0.002/hour when enabled |
| **DSC node management** | ~$6/node/month |
| **Update Management** | **Free** (but Log Analytics ingestion costs apply) |
| **Change Tracking** | **Free** features (Log Analytics costs apply) |
| **Hybrid Runbook Worker** | **Free** (VM costs are yours) |
| **Basic tier** | All included (retired — now all accounts are Free or Basic) |

> ⚠️ **EXAM TIP:** First **500 minutes** of job runtime free per month. Hybrid Runbook Worker itself is free — you pay for the VM.

> ⚠️ **EXAM TIP:** DSC node management = **~$6/node/month**. Update Management itself is free, but you pay for Log Analytics data ingestion.

---

## 19. Limitations & Constraints

| Constraint | Limit |
|---|---|
| **Sandbox max runtime** | **3 hours** (fair share) |
| **Job output** | **1 MB** max |
| **Runbook nesting depth** | **5 levels** |
| **Webhook URL** | View **once** at creation only |
| **Webhook expiry** | **Max 10 years** |
| **Schedule minimum start** | **5 minutes** from now |
| **Variables per account** | **256** |
| **Credentials per account** | **256** |
| **Certificates per account** | **256** |
| **Modules per account** | No hard limit (practical ~500) |
| **Job log retention** | **30 days** |
| **DSC default refresh** | Every **30 minutes** |
| **DSC default config frequency** | Every **15 minutes** |
| **File size for runbook** | **1 MB** |
| **Update maint. window max** | **6 hours** |

---

## 20. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| Create Automation Account | `az automation account create -g <rg> -n <acct> --location <region>` |
| List accounts | `az automation account list -g <rg> -o table` |
| Create runbook | `az automation runbook create -g <rg> --automation-account-name <acct> -n <name> --type PowerShell` |
| Import runbook | `az automation runbook replace-content -g <rg> --automation-account-name <acct> -n <name> --content @script.ps1` |
| Publish runbook | `az automation runbook publish -g <rg> --automation-account-name <acct> -n <name>` |
| Start runbook | `az automation runbook start -g <rg> --automation-account-name <acct> -n <name>` |
| List jobs | `az automation job list -g <rg> --automation-account-name <acct> -o table` |
| Show job output | `az automation job show -g <rg> --automation-account-name <acct> -n <job-name>` |

### PowerShell

| Action | Command |
|---|---|
| Create account | `New-AzAutomationAccount -ResourceGroupName <rg> -Name <acct> -Location <region>` |
| Import runbook | `Import-AzAutomationRunbook -ResourceGroupName <rg> -AutomationAccountName <acct> -Path ./script.ps1 -Type PowerShell -Name <name>` |
| Publish runbook | `Publish-AzAutomationRunbook -ResourceGroupName <rg> -AutomationAccountName <acct> -Name <name>` |
| Start runbook | `Start-AzAutomationRunbook -ResourceGroupName <rg> -AutomationAccountName <acct> -Name <name>` |
| Get job | `Get-AzAutomationJob -ResourceGroupName <rg> -AutomationAccountName <acct>` |
| Create schedule | `New-AzAutomationSchedule -ResourceGroupName <rg> -AutomationAccountName <acct> -Name "daily" -StartTime (Get-Date).AddMinutes(10) -DayInterval 1` |
| Link schedule | `Register-AzAutomationScheduledRunbook -ResourceGroupName <rg> -AutomationAccountName <acct> -RunbookName <rb> -ScheduleName "daily"` |
| Set variable | `New-AzAutomationVariable -ResourceGroupName <rg> -AutomationAccountName <acct> -Name "myVar" -Value "hello" -Encrypted $false` |
| Create credential | `New-AzAutomationCredential -ResourceGroupName <rg> -AutomationAccountName <acct> -Name "myCred" -Value (Get-Credential)` |
| Register DSC node | `Register-AzAutomationDscNode -ResourceGroupName <rg> -AutomationAccountName <acct> -AzureVMName <vm> -NodeConfigurationName "Config.Server"` |
| Compile DSC config | `Start-AzAutomationDscCompilationJob -ResourceGroupName <rg> -AutomationAccountName <acct> -ConfigurationName "Config"` |

---

## 21. Quick-Fire Exam Points ⚡

1. Azure Automation = **Runbooks** (PowerShell/Python) + **DSC** + **Update Management** + **Change Tracking**
2. Runbook types: **PowerShell**, **PowerShell Workflow**, **Python**, **Graphical**
3. **PowerShell Workflow** = supports checkpoints + parallel execution. Standard PowerShell = does NOT
4. **Graphical runbooks** = visual authoring in portal only — cannot edit as text
5. **Azure sandbox** max runtime = **3 hours** (fair share). **Hybrid Worker** = no limit
6. **Hybrid Runbook Worker** needed for: on-prem access, jobs > 3 hours, local resources
7. **Extension-based** Hybrid Worker = recommended. **Agent-based** = legacy
8. **Run As Account** = DEPRECATED (Sep 2023). Use **Managed Identity** instead
9. Managed Identity auth: `Connect-AzAccount -Identity` in runbook
10. **Encrypted variables** = write-only. Cannot retrieve value once encrypted
11. **Az.Accounts** module must be imported first — dependency for all other Az modules
12. Schedule start = minimum **5 minutes** from now
13. Many-to-many: runbook ↔ schedules (multiple schedules per runbook, multiple runbooks per schedule)
14. Webhook URL = **one-time view**. Lose it = create new webhook
15. Webhook max expiry = **10 years**. Input → `$WebhookData.RequestBody`
16. **Update Management** requires: Automation Account + Log Analytics workspace + Log Analytics agent
17. Update maintenance window max = **6 hours** (default 2 hours). Stops installing 20 min before window end
18. **Azure Update Manager** = new replacement (no Automation Account or Log Analytics needed)
19. DSC modes: **ApplyOnly** (once) / **ApplyAndMonitor** (detect drift) / **ApplyAndAutoCorrect** (fix drift)
20. DSC default refresh = **30 min**. Default config frequency = **15 min**
21. DSC config must be **compiled** (creates MOF file) before assigning to nodes
22. **Automation Operator** = can run runbooks but CANNOT edit/create
23. First **500 minutes/month** free. DSC = **~$6/node/month**
24. Hybrid Worker = **free** feature (pay for VM only)
25. Job logs retained for **30 days**. Job output max = **1 MB**
26. Runbook file max size = **1 MB**
27. Runbook nesting max depth = **5 levels**
28. Change Tracking: tracks files, registry (Windows), services, software, daemons (Linux)
29. Source control: GitHub, Azure DevOps Git, Azure DevOps TFVC
30. Diagnostic logs: **JobLogs**, **JobStreams**, **DscNodeStatus**, **AuditEvent** → send to Log Analytics

---

## 22. Step-by-Step Configuration Mind Maps 🗺️

---

### 22.1 Create Automation Account & Run a Runbook

> **Portal:** `Home → Automation Accounts → + Create`

```
Create Automation Account & Runbook
│
├── Step 1: Create Automation Account
│   │   Portal: Home → Automation Accounts → + Create
│   ├── Basics tab:
│   │   ├── Subscription: Select
│   │   ├── Resource Group: Select / Create
│   │   ├── Automation account name: my-automation
│   │   └── Region: Select
│   ├── Advanced tab:
│   │   ├── System assigned managed identity: ✅ (recommended)
│   │   └── User assigned managed identity: + Add (optional)
│   ├── Networking tab:
│   │   ├── Public access: Enabled (default) / Disabled
│   │   └── Private endpoints: + Create (optional)
│   ├── Tags → optional
│   └── Review + create → Create
│       ⚠️ Run As Account deprecated — use Managed Identity
│
├── Step 2: Assign RBAC to Managed Identity
│   │   For runbook to manage Azure resources:
│   │   Target resource/RG/subscription → Access Control (IAM) →
│   │   + Add role assignment →
│   │   Role: Contributor / VM Contributor / etc. →
│   │   Members: Managed identity → Select Automation Account identity →
│   │   Review + assign
│
├── Step 3: Import Required Modules
│   │   Automation Account → Shared Resources → Modules →
│   │   + Add a module → Browse Gallery →
│   │   Search: Az.Accounts → Import (do this FIRST)
│   │   Then: Az.Compute, Az.Resources, etc.
│   │   ⚠️ Az.Accounts must be imported first (dependency)
│   │   Select Runtime version: 5.1 / 7.2
│
├── Step 4: Create Runbook
│   │   Automation Account → Process Automation → Runbooks →
│   │   + Create a runbook
│   ├── Name: stop-idle-vms
│   ├── Runbook type: PowerShell
│   ├── Runtime version: 5.1 / 7.2
│   ├── Description: Stop idle VMs nightly
│   └── Create
│
├── Step 5: Author Runbook
│   │   Runbook opens in editor:
│   ├── Write PowerShell code:
│   │   Connect-AzAccount -Identity
│   │   Get-AzVM -ResourceGroupName "prod" | Stop-AzVM -Force
│   ├── Test pane → Start → Verify output
│   └── Publish (top bar) → Yes
│       ⚠️ Runbook must be Published to be scheduled/started
│
├── Step 6: Test & Run
│   │   Automation Account → Runbooks → Select runbook →
│   │   Start → Parameters (if any) → OK
│   │   Monitor: Jobs tab → Select job → Output / Errors
│
└── ⚠️ Notes
    ├── Draft runbooks cannot be scheduled or triggered
    ├── Always test in Test pane before publishing
    ├── Published = live version. Draft = work-in-progress
    └── Max sandbox runtime = 3 hours
```

---

### 22.2 Schedule a Runbook

> **Portal:** `Automation Account → Runbooks → Select → Schedules`

```
Schedule a Runbook
│
├── Prerequisites
│   ├── Runbook exists and is **Published**
│   │   ⚠️ Draft runbooks CANNOT be scheduled
│   └── RBAC: Automation Contributor or higher
│
├── Step 1: Create Schedule
│   │   Automation Account → Shared Resources → Schedules →
│   │   + Add a schedule
│   ├── Name: nightly-cleanup
│   ├── Description: Run cleanup at 11 PM
│   ├── Starts: 2026-02-21 23:00
│   │   ⚠️ Must be at least 5 minutes in the future
│   ├── Time zone: (UTC+05:30) Chennai, Kolkata
│   ├── Recurrence:
│   │   ├── Once
│   │   └── Recurring:
│   │       ├── Recur every: 1 Day
│   │       ├── On these days: Mon-Fri (weekly option)
│   │       └── Monthly: Day 1 / Last day / specific weekday
│   ├── Set expiration: ✅ 2026-12-31 (optional)
│   └── Create
│
├── Step 2: Link Schedule to Runbook
│   │   Automation Account → Runbooks → Select runbook →
│   │   Schedules → + Add a schedule
│   ├── Link a schedule to your runbook → Select schedule: nightly-cleanup
│   ├── Parameters and run settings:
│   │   ├── Parameter values: fill in required params
│   │   └── Run on: Azure (sandbox) / Hybrid Worker group
│   │   ⚠️ On-prem tasks → select Hybrid Worker group
│   └── OK
│
├── Step 3: Monitor
│   │   Automation Account → Jobs → Filter by runbook name
│   ├── Check status: Completed / Failed
│   └── View output/errors
│
└── ⚠️ Notes
    ├── One runbook can have multiple schedules
    ├── One schedule can trigger multiple runbooks
    ├── Schedule change affects ALL linked runbooks
    ├── Deleteing a schedule stops future executions
    └── Schedule fires even if previous job still running (new job queued)
```

---

### 22.3 Configure Hybrid Runbook Worker

> **Portal:** `Automation Account → Hybrid worker groups → + Create`

```
Configure Hybrid Runbook Worker
│
├── Prerequisites
│   ├── Azure VM (Windows or Linux) or Arc-enabled server
│   ├── VM networking: outbound to Azure Automation (HTTPS 443)
│   └── RBAC: Automation Contributor + VM Contributor
│
├── Step 1: Create Hybrid Worker Group
│   │   Automation Account → Process Automation →
│   │   Hybrid worker groups → + Create hybrid worker group
│   ├── Name: onprem-workers
│   ├── Use Hybrid Worker Credentials:
│   │   ├── Default (Automation Account managed identity)
│   │   └── Custom: Select credential asset
│   └── Next
│
├── Step 2: Add Machines
│   ├── + Add machines → Select VMs
│   │   ⚠️ VM must be in same subscription (or Arc-enabled for hybrid)
│   ├── Select VM(s) → Add
│   └── Review + create → Create
│       ⚠️ Azure VM extension "HybridWorker" auto-installed
│
├── Step 3: Run Runbook on Hybrid Worker
│   │   Automation Account → Runbooks → Select runbook → Start
│   ├── Run on: Hybrid Worker
│   ├── Hybrid Worker group: Select → onprem-workers
│   └── OK
│       ⚠️ No 3-hour fair share limit on Hybrid Worker
│
├── Verify
│   │   Automation Account → Hybrid worker groups →
│   │   Select group → View workers → Status: Online
│
└── ⚠️ Notes
    ├── Extension-based = recommended (no Log Analytics needed)
    ├── Agent-based = legacy (requires MMA + LA workspace)
    ├── Hybrid Worker = FREE (VM cost is yours)
    ├── No sandbox time limit — run as long as needed
    ├── Access local resources (disk, network, on-prem)
    └── Update Management auto-creates System Hybrid Worker
```

---

### 22.4 Configure State Configuration (DSC)

> **Portal:** `Automation Account → State Configuration (DSC)`

```
Configure DSC
│
├── Prerequisites
│   ├── Automation Account exists
│   ├── DSC PowerShell configuration script (.ps1) prepared
│   └── Target VMs accessible (Azure or Hybrid)
│
├── Step 1: Import Configuration
│   │   Automation Account → State Configuration (DSC) →
│   │   Configurations → + Add
│   ├── Upload configuration file: WebServer.ps1
│   │   Example content:
│   │     Configuration WebServer {
│   │       Node "localhost" {
│   │         WindowsFeature IIS {
│   │           Ensure = "Present"
│   │           Name = "Web-Server"
│   │         }
│   │       }
│   │     }
│   └── Import
│
├── Step 2: Compile Configuration
│   │   State Configuration → Configurations → Select "WebServer" →
│   │   Compile → Yes
│   ├── Compilation creates: Node Configuration = "WebServer.localhost"
│   │   ⚠️ Must compile before assigning to nodes
│   └── Monitor: Compilation jobs tab → Status: Completed
│
├── Step 3: Register Node (VM)
│   │   State Configuration → Nodes → + Add
│   ├── Select VM → Connect
│   ├── Configuration:
│   │   ├── Node configuration: WebServer.localhost
│   │   ├── Configuration mode:
│   │   │   ├── ApplyOnly (apply once, no monitoring)
│   │   │   ├── ApplyAndMonitor (apply + detect drift, no fix)
│   │   │   └── ApplyAndAutoCorrect ✅ (apply + detect + FIX drift)
│   │   │   ⚠️ ApplyAndAutoCorrect = most common exam answer
│   │   ├── Reboot if needed: true / false
│   │   ├── Configuration frequency: 15 min (default)
│   │   ├── Refresh frequency: 30 min (default)
│   │   └── Allow Module Override: true / false
│   └── OK
│
├── Step 4: Monitor Compliance
│   │   State Configuration → Nodes →
│   │   Status: Compliant ✅ / Not Compliant ❌ / Pending / Failed
│   └── Click node → View details → Drift report
│
└── ⚠️ Notes
    ├── Compile BEFORE assigning (MOF creation)
    ├── ApplyAndAutoCorrect = detects + fixes drift automatically
    ├── Cost: ~$6/node/month
    ├── Pull model: nodes pull config from Automation Account
    ├── DSC requires LCM (Local Configuration Manager) on each node
    └── PowerShell DSC = Windows native; Linux uses omi + nxtools
```

---

### 22.5 Configure Update Management

> **Portal:** `Automation Account → Update Management`

```
Configure Update Management (Legacy)
│
├── Prerequisites
│   ├── Automation Account exists
│   ├── Log Analytics workspace exists
│   └── RBAC: Contributor on Automation Account + VMs
│
├── Step 1: Enable Update Management
│   │   Automation Account → Update Management →
│   │   Enable
│   ├── Log Analytics workspace: Select / Create
│   ├── Subscription: Select
│   ├── Region: Select
│   └── Enable
│       ⚠️ Links Automation Account to Log Analytics workspace
│
├── Step 2: Onboard VMs
│   │   Automation Account → Update Management →
│   │   + Add Azure VMs → Select VMs → Enable
│   ├── OR: + Add Non-Azure machine → Download agent →
│   │   Install LA agent on on-prem VM
│   └── ⚠️ VMs need Log Analytics agent (MMA) installed
│
├── Step 3: View Assessment
│   │   Update Management dashboard:
│   │   Missing updates tab → View OS / Critical / Security
│   │   Machines tab → Status per machine
│   └── ⚠️ Initial assessment may take 2-24 hours
│
├── Step 4: Schedule Update Deployment
│   │   Update Management → Schedule update deployment
│   ├── Name: monthly-windows-patches
│   ├── Operating system: Windows / Linux
│   ├── Machines to update:
│   │   ├── All available
│   │   ├── Selected machines
│   │   └── Dynamic groups (Azure query)
│   ├── Update classifications:
│   │   └── Critical ✅, Security ✅, Update Rollups ✅
│   ├── Include/Exclude updates: KB IDs
│   ├── Schedule:
│   │   ├── Once / Recurring
│   │   ├── Start time + time zone
│   │   └── Recurrence: Daily / Weekly / Monthly
│   ├── Maintenance window: 120 minutes (30 min – 6 hours)
│   │   ⚠️ Max 6 hours. Stops 20 min before end for reboot
│   ├── Reboot options:
│   │   ├── If required (default)
│   │   ├── Never reboot
│   │   ├── Always reboot
│   │   └── Only reboot (install nothing)
│   └── Pre/Post scripts: Select runbooks (optional)
│       ⚠️ Pre-script: e.g., create snapshot. Post-script: e.g., start services
│
├── Step 5: Monitor
│   │   Update Management → History tab →
│   │   View deployment runs → Status per VM
│   └── Succeeded / Failed / In progress / Not started
│
└── ⚠️ Notes
    ├── Update Management = free (LA ingestion costs apply)
    ├── Requires: Automation Account + LA workspace + LA agent
    ├── Being replaced by Azure Update Manager
    ├── Max maintenance window = 6 hours
    ├── Pre/post scripts = runbooks in same Automation Account
    └── Dynamic groups = Azure Resource Graph queries
```

---

### 22.6 Create and Use a Webhook

> **Portal:** `Automation Account → Runbooks → Select → Webhooks`

```
Create and Use a Webhook
│
├── Prerequisites
│   ├── Runbook exists and is **Published**
│   └── RBAC: Automation Contributor
│
├── Step 1: Create Webhook
│   │   Automation Account → Runbooks → Select runbook →
│   │   Webhooks → + Add webhook
│   ├── Create new webhook:
│   │   ├── Name: alert-trigger-webhook
│   │   ├── Enabled: Yes
│   │   ├── Expires: Set date (max 10 years from now)
│   │   └── ⚠️⚠️ COPY THE URL NOW ⚠️⚠️
│   │       → URL shown ONCE. Cannot retrieve later
│   │       → Contains authentication token
│   │       → Treat as a secret
│   ├── Next: Configure parameters and run settings
│   │   ├── Parameters: fill defaults (or leave for request body)
│   │   └── Run on: Azure / Hybrid Worker
│   └── Create
│
├── Step 2: Use Webhook (External Trigger)
│   │   HTTP POST to webhook URL:
│   │   curl -X POST "https://s1events.azure-automation.net/webhooks?token=..."
│   │     -H "Content-Type: application/json"
│   │     -d '{"vmName":"myVM","action":"restart"}'
│   │
│   │   Runbook receives:
│   │   param([object]$WebhookData)
│   │   $body = ConvertFrom-Json $WebhookData.RequestBody
│   │   $vmName = $body.vmName
│
├── Step 3: Integrate with Azure Alerts
│   │   Monitor → Alerts → Action groups →
│   │   Action type: Automation Runbook →
│   │   Select runbook → Use webhook or direct
│   └── ⚠️ Azure Monitor alerts can trigger runbooks directly
│
├── Step 4: Monitor
│   │   Automation Account → Jobs → Filter →
│   │   View job triggered by webhook
│   └── Check: Start method = "Webhook"
│
└── ⚠️ Notes
    ├── URL = one-time view. Lose it = create new webhook
    ├── Max expiry = 10 years
    ├── HTTP POST only (not GET)
    ├── Input via $WebhookData parameter in runbook
    ├── Can be disabled without deleting
    └── Common scenario: Alert → Webhook → Runbook → Remediate
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
