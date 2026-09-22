<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Site Recovery (ASR) — AZ-104 Revision Notes

---

## 1. What is Azure Site Recovery?

- **Disaster Recovery as a Service (DRaaS)** for Azure and on-premises workloads
- Replicates workloads → failover to secondary location during outage → failback when resolved
- Ensures **business continuity** with automated replication, failover, and recovery
- Orchestrated via **Recovery Services Vault** (same vault type used for Azure Backup)
- NOT the same as Azure Backup — **Backup = data protection**, **ASR = disaster recovery**

---

## 2. Key Components

| Component | Purpose |
|---|---|
| **Recovery Services Vault** | Container for replication configuration, policies, recovery plans |
| **Replication Policy** | Defines RPO, recovery point retention, app-consistent snapshot frequency |
| **Recovery Plan** | Groups machines for ordered failover (with manual/automated steps) |
| **Configuration Server** | On-prem server that coordinates communication (VMware/Physical only) |
| **Process Server** | Handles replication data, caching, compression (VMware/Physical) |
| **Master Target Server** | Receives replication data during failback (VMware/Physical) |
| **Mobility Service** | Agent installed on source VMs being replicated |
| **Azure Site Recovery Provider** | Installed on Hyper-V hosts / VMM servers |

---

## 3. Supported Scenarios

| Source | Target | Use Case |
|---|---|---|
| **Azure VM → Azure** (different region) | Azure (paired/non-paired region) | Azure-to-Azure DR |
| **On-prem VMware VM → Azure** | Azure | VMware DR to cloud |
| **On-prem Hyper-V VM → Azure** | Azure | Hyper-V DR to cloud |
| **On-prem Physical Server → Azure** | Azure | Physical server DR |
| **Hyper-V VM → Hyper-V** (secondary site) | Secondary on-prem site | Site-to-site DR (with VMM) |
| **VMware VM → VMware** (secondary site) | Secondary on-prem site | Deprecated (use Azure) |

> ⚠️ **EXAM TIP:** AZ-104 focuses primarily on **Azure VM to Azure** replication. Know the on-prem scenarios at a high level.

---

## 4. Azure-to-Azure Replication (Primary Focus for AZ-104)

### How It Works
1. **Mobility Service extension** auto-installed on source Azure VM
2. VM disk writes replicated continuously to **cache storage account** in source region
3. Data sent to **target region** → written to **managed disks** (replica disks)
4. **Crash-consistent** recovery points generated every **5 minutes**
5. **App-consistent** recovery points generated per policy (default: every 4 hours)

### What Gets Replicated
- **OS disk + all data disks** (managed disks)
- VM configuration, NICs, public IPs (re-created in target)
- Does NOT replicate: VM extensions, Azure Backup data, temporary disk, page file

### Requirements
- Source and target must be **different Azure regions**
- Source VM must be **running** to enable replication
- **Managed disks** supported (both Standard and Premium)
- **Unmanaged disks** — must convert to managed first
- Source VM must have **outbound connectivity** to ASR service URLs

### Portal Path — Enable Replication (Azure VM)
```
Recovery Services Vault → Protected Items → Replicated items →
+ Replicate → Azure virtual machines →
Select Source Region, Subscription, RG, VMs →
Select Target Region, Subscription, RG, VNet, Storage →
Configure Replication Policy →
Enable Replication
```

### Portal Path — Enable from VM Blade
```
Virtual Machine → Operations → Disaster recovery →
Select Target Region → Review + Start replication
```

---

## 5. Replication Policy

- Defines **how** replication behaves
- Attached to Recovery Services Vault

| Setting | Default | Range |
|---|---|---|
| **Recovery Point Retention** | 24 hours | 0–72 hours |
| **App-consistent Snapshot Frequency** | 4 hours | 1–12 hours (0 = disabled) |
| **Crash-consistent Recovery Points** | Every 5 min | Not configurable (always 5 min) |

### Portal Path — Create/Modify Replication Policy
```
Recovery Services Vault → Manage → Site Recovery Infrastructure →
Replication Policies → + Replication Policy →
Name, Recovery Point Retention, App-consistent frequency → OK
```

> ⚠️ **EXAM TIP:** **Crash-consistent** points = every **5 minutes** (auto, cannot change). **App-consistent** = configurable (default 4 hrs). **RPO = 5 minutes** minimum for Azure-to-Azure.

---

## 6. Recovery Points

| Type | Description | Frequency |
|---|---|---|
| **Crash-consistent** | Captures disk data at exact point-in-time (like power failure) | Every 5 min (auto) |
| **App-consistent** | Captures disk + in-memory data + pending I/O (uses VSS on Windows) | Configurable (1–12 hrs) |
| **Latest processed** | Latest crash-consistent point processed by ASR (low RTO) | — |
| **Latest app-consistent** | Latest app-consistent recovery point | — |
| **Latest** | Process all pending data first, then create recovery point (lowest RPO, higher RTO) | — |
| **Custom** | Select a specific past recovery point | — |

> ⚠️ **EXAM TIP:** **Latest processed** = fastest failover (low RTO), slightly higher RPO. **Latest** = lowest RPO but takes longer (higher RTO). **App-consistent** = cleanest state but less frequent.

---

## 7. Failover Types

| Type | When to Use | Data Loss? |
|---|---|---|
| **Test Failover** | DR drill — no impact on production | No (isolated network) |
| **Planned Failover** | Known event (e.g., planned maintenance) — Hyper-V only | No (VMs shut down first) |
| **Failover** | Actual disaster — source region unavailable | Possible (depends on RPO) |
| **Forced Failover** | Source unreachable, failover to latest available point | Possible |

### Test Failover
- Creates a **test VM** in target region using selected recovery point
- Uses **isolated virtual network** (no impact on production or replication)
- Must **clean up** test failover after validation
- **Best practice:** Run test failover regularly

### Failover
- Stops replication from source
- Creates VM in target region from recovery point
- After validation → **Commit** (finalizes) or **Change recovery point** (try different point)
- After commit → replication direction is reversed (target becomes source)

### Portal Path — Test Failover
```
Recovery Services Vault → Replicated items → Select VM →
Test Failover → Select Recovery Point → Select Target VNet →
OK → Validate → Clean up Test Failover
```

### Portal Path — Failover
```
Recovery Services Vault → Replicated items → Select VM →
Failover → Select Recovery Point → Shut down source (optional) →
OK → Validate target VM → Commit
```

> ⚠️ **EXAM TIP:** Always **Commit** after failover. If you don't commit, you can still change recovery point. After commit, you can **Re-protect** (reverse replication) and then **Failback**.

---

## 8. Re-protect & Failback

### Workflow
```
1. Failover to target region (during disaster)
2. Commit failover
3. Re-protect (reverse replication: target → source)
4. Wait for replication to complete
5. Failback (failover from target → back to source)
6. Commit failback
7. Re-protect (source → target again — back to normal)
```

### Portal Path — Re-protect
```
Recovery Services Vault → Replicated items → Select VM →
Re-protect → Confirm target (original source) settings → OK
```

### Portal Path — Failback
```
Recovery Services Vault → Replicated items → Select VM →
Failover (from target back to source) → Select Recovery Point → OK →
Commit → Re-protect (original direction)
```

> ⚠️ **EXAM TIP:** After failover you MUST **Re-protect** before you can **Failback**. Re-protect reverses the replication direction.

---

## 9. Recovery Plans

- **Group multiple VMs** for coordinated failover
- Define **failover order** using groups (Group 1 fails over first, then Group 2, etc.)
- Add **manual actions** (e.g., validate, approve) or **automated scripts** (Azure Automation runbooks)
- Max **100 VMs** per recovery plan
- Supports **test failover, planned failover, and failover**

### Portal Path — Create Recovery Plan
```
Recovery Services Vault → Manage → Recovery Plans → + Recovery Plan →
Name → Select Source & Target →
Add VMs → Organize into Groups (Group 1, 2, 3…) →
Add Pre/Post actions (manual or runbook) → OK
```

### Portal Path — Execute Recovery Plan
```
Recovery Services Vault → Recovery Plans → Select Plan →
Test Failover / Failover → Select Recovery Point → OK
```

> ⚠️ **EXAM TIP:** Recovery plans = **ordered failover**. Group 1 completes before Group 2 starts. Add **Azure Automation runbooks** for automated steps (e.g., add NSG, change DNS).

---

## 10. Network Mapping (Azure-to-Azure)

- Maps **source VNet** → **target VNet**
- During failover, VMs placed in mapped target VNet
- IP addresses can be **customized** per VM (static or dynamic in target)
- Target subnet mapping configurable
- Public IP re-created in target (new IP; option to attach)

### Portal Path
```
Recovery Services Vault → Manage → Site Recovery Infrastructure →
Network Mapping → + Network Mapping →
Source Network → Target Network → OK
```

> ⚠️ **EXAM TIP:** Network mapping is per-direction. If you re-protect (reverse), the **reverse mapping is auto-created**.

---

## 11. Capacity Reservation & Target Resources

### Target Region Resources (Auto-created during Enable Replication)

| Resource | Purpose |
|---|---|
| **Cache Storage Account** | In source region — caches replication data before sending to target |
| **Replica Managed Disks** | In target region — replica of source disks |
| **Target Resource Group** | Contains failed-over VMs and resources |
| **Target VNet** | Target virtual network for failed-over VMs |
| **Target Availability Set** | If source VM is in an availability set |

> ⚠️ **EXAM TIP:** Cache storage account is in the **source region**, NOT target. Managed disks replicas are in the **target region**.

---

## 12. Azure Site Recovery vs Azure Backup

| Feature | Azure Site Recovery | Azure Backup |
|---|---|---|
| **Purpose** | Disaster recovery (DR) | Data protection (backup/restore) |
| **Scope** | Entire VM / workload replication | VM, files, databases, disks |
| **RPO** | Minutes (5 min crash-consistent) | Hours (daily/hourly backup) |
| **RTO** | Minutes to hours | Hours to days |
| **Data Location** | Replicated to another Azure region | Stored in vault (same/different region) |
| **Vault** | Recovery Services Vault | Recovery Services Vault / Backup Vault |
| **Use Case** | Region failure, site outage | Accidental deletion, corruption, ransomware |
| **Failover** | Yes (automated/manual) | No (restore operation) |
| **Continuous Replication** | Yes | No (scheduled snapshots) |

> ⚠️ **EXAM TIP:** **ASR = DR (keep workloads running)**. **Backup = Protect data (restore when needed)**. They use the **same Recovery Services Vault** but serve different purposes. Can use BOTH together.

---

## 13. On-Premises to Azure Replication (Overview)

### VMware to Azure

| Component | Location | Purpose |
|---|---|---|
| **Configuration Server** | On-prem VMware VM | Coordinates replication, manages communication |
| **Process Server** | On-prem (with Config Server or standalone) | Caches, compresses, encrypts replication data |
| **Mobility Service** | Each replicated VM | Captures disk writes and sends to Process Server |
| **Master Target Server** | On-prem (for failback) | Receives data during failback |

### Hyper-V to Azure

| Component | Location | Purpose |
|---|---|---|
| **Azure Site Recovery Provider** | On Hyper-V host | Orchestrates replication with ASR |
| **Recovery Services Agent** | On Hyper-V host | Handles data replication |
| **VMM (Optional)** | System Center VMM server | Centralized management of Hyper-V hosts |

### Physical Servers to Azure
- Same architecture as VMware (Config Server + Process Server + Mobility Service)
- **Cannot failback to physical** — fails back as VMware VM

> ⚠️ **EXAM TIP:** Physical server failback = **VMware VM** (not physical). VMware & Physical use **Configuration Server**. Hyper-V uses **ASR Provider**.

---

## 14. ASR Networking Requirements

### Outbound Connectivity (Source VMs)
- VMs must reach ASR service URLs (login.microsoftonline.com, *.hypervrecoverymanager.windowsazure.com, *.blob.core.windows.net, etc.)
- Options: **NSG rules**, **Azure Firewall**, **Proxy server**, **Service Tags**
- **Service Tag:** `SiteRecovery` (allows ASR traffic in NSG rules)

### NSG Rule for ASR
```
Source: Any → Destination: Service Tag "SiteRecovery" →
Port: 443 → Protocol: TCP → Allow
```

### Portal Path
```
NSG → Settings → Outbound security rules → + Add →
Destination: Service Tag → Service Tag: SiteRecovery →
Port: 443 → Allow → Add
```

> ⚠️ **EXAM TIP:** Outbound port **443 (HTTPS)** must be open for ASR replication. Use **Service Tag: SiteRecovery** in NSGs for simplified rules.

---

## 15. Monitoring ASR

### ASR Dashboard
```
Recovery Services Vault → Overview → Site Recovery Dashboard →
Replicated Items / Recovery Plans / Fabric / Jobs
```

### Replication Health States

| State | Meaning |
|---|---|
| **Normal** | Replication healthy, on schedule |
| **Warning** | Replication lagging, minor issue |
| **Critical** | Replication broken, manual intervention needed |

### Key Monitoring Points
- **RPO breach** — replication lag exceeding defined threshold
- **Replication health** — Normal / Warning / Critical
- **Failover readiness** — checks if target resources are ready
- **Test failover** — last test failover date & status

### Portal Path — Monitor Replication
```
Recovery Services Vault → Protected Items → Replicated items →
Select VM → Overview (Health, RPO, Failover readiness)
```

### Configure ASR Alerts
```
Recovery Services Vault → Monitoring → Alerts →
+ Create Alert Rule → Signal: Site Recovery events →
Configure condition → Select Action Group → Create
```

### ASR Events in Activity Log
- Failover events, replication health changes, policy changes
- Queryable in Log Analytics (if Activity Log exported)

---

## 16. Security & RBAC

### Built-in Roles

| Role | Permissions |
|---|---|
| **Site Recovery Contributor** | All ASR operations EXCEPT vault create/delete and role assignment |
| **Site Recovery Operator** | Failover and failback operations (no enable/disable replication) |
| **Site Recovery Reader** | View all ASR operations (read-only) |

### Key Permission Mapping

| Action | Required Role |
|---|---|
| Enable replication | Site Recovery Contributor |
| Failover / Test Failover | Site Recovery Operator or Contributor |
| Create Recovery Plan | Site Recovery Contributor |
| Create/modify replication policy | Site Recovery Contributor |
| View replication status | Site Recovery Reader |
| Create Recovery Services Vault | Contributor (on RG) |

> ⚠️ **EXAM TIP:** **Site Recovery Operator** can perform failover/failback but CANNOT enable/disable replication. **Site Recovery Contributor** can do everything except vault management.

---

## 17. Pricing Key Points

| Component | Cost |
|---|---|
| **ASR License (per protected instance)** | Per VM per month (first 31 days free per new instance) |
| **Storage (replica disks)** | Standard/Premium managed disk cost in target region |
| **Cache Storage Account** | Standard storage cost in source region |
| **Network egress** | Outbound data transfer from source to target region |
| **Compute (during failover)** | VM compute charges ONLY when VMs are running in target |
| **Test Failover** | Compute + storage charges for test VM duration |
| **Recovery Plans** | No additional cost (included) |

### Cost Optimization
- No compute costs **until failover** (only storage + replication)
- Clean up **test failover** VMs promptly to avoid compute charges
- Use **Standard HDD** for replica disks if acceptable performance

> ⚠️ **EXAM TIP:** You are NOT charged for **target VM compute** until you actually **failover**. Replication = storage cost only. First **31 days free** per new protected instance.

---

## 18. Azure Site Recovery with Availability Zones

- Can replicate Azure VMs **within the same region** to a different **Availability Zone**
- Use case: **Zone-to-Zone DR** (not just region-to-region)
- Protects against zone-level failures
- Source in Zone 1 → replicate to Zone 2 or 3 (same region)

### Portal Path
```
VM → Operations → Disaster recovery →
Target region: Same region → Select Target Availability Zone →
Start Replication
```

> ⚠️ **EXAM TIP:** ASR supports **Zone-to-Zone** replication within the same region (not just cross-region). This is a commonly tested point.

---

## 19. Automation & Integration

### Azure Automation Runbooks in Recovery Plans
- Add **pre-action** (before failover) or **post-action** (after failover)
- Examples: Attach NSG, Update DNS, Add Load Balancer rule, Run custom script
- Runbooks must be in an **Azure Automation Account**

### Azure Policy for ASR
- Enforce ASR enablement on VMs via Azure Policy
- Built-in policy: "Azure virtual machines should have disaster recovery configured"

### ARM Templates / PowerShell / CLI
- `az site-recovery` CLI commands for automation
- PowerShell: `New-AzRecoveryServicesAsrReplicationProtectedItem`
- ARM templates for infrastructure-as-code DR configuration

---

## 20. Limitations & Constraints

| Constraint | Limit |
|---|---|
| Max VMs per Recovery Plan | **100** |
| Max Recovery Plans per vault | **200** |
| Max replicated VMs per vault | **2000** (Azure-to-Azure) |
| Max disks per VM | **64** managed disks |
| Max disk size | **32 TB** per managed disk |
| Max data churn rate | **54 MB/s per disk** (Premium), **2 MB/s** (Standard HDD) |
| RPO (crash-consistent) | **5 minutes** |
| Recovery Point Retention | **0–72 hours** |
| App-consistent snapshot frequency | **1–12 hours** |
| Source & Target | Must be **different regions** (or different zones in same region) |

> ⚠️ **EXAM TIP:** Max **100 VMs per recovery plan**, **2000 VMs per vault**, **64 disks per VM**, **72 hours max recovery point retention**.

---

## 21. Quick-Fire Exam Points ⚡

1. ASR = **Disaster Recovery**, Azure Backup = **Data Protection** — different purposes, same vault type
2. **Crash-consistent** recovery points = every **5 minutes** (cannot change)
3. **App-consistent** snapshots = default **every 4 hours** (configurable 1–12 hrs)
4. **Recovery point retention** = default 24 hours, max **72 hours**
5. **Cache storage account** = in **source region**, replica disks = in **target region**
6. Always **Commit** after failover → then **Re-protect** → then **Failback**
7. **Re-protect** = reverses replication direction (required before failback)
8. **Test Failover** uses **isolated network** — no impact on production replication
9. Always **clean up test failover** to avoid charges
10. **Recovery Plans** = ordered failover across groups, max **100 VMs** per plan
11. **Site Recovery Operator** = failover/failback only (no enable/disable replication)
12. **Site Recovery Contributor** = all ASR ops except vault create/delete
13. **No compute charges** until actual failover (only storage during replication)
14. First **31 days FREE** per new protected instance
15. **Zone-to-Zone** replication supported (same region, different AZ)
16. Physical server failback = **VMware VM** (not physical machine)
17. VMware/Physical = **Configuration Server + Process Server + Mobility Service**
18. Hyper-V = **ASR Provider + Recovery Services Agent**
19. Outbound port **443** required, use **Service Tag: SiteRecovery** in NSGs
20. Max **2000 VMs per vault**, **200 recovery plans per vault**
21. Max disk size = **32 TB**, max disks per VM = **64**
22. **Latest processed** = fastest failover (lowest RTO). **Latest** = lowest RPO (highest RTO)
23. Network mapping: source VNet → target VNet (reverse auto-created on re-protect)
24. Azure Automation runbooks = pre/post actions in Recovery Plans
25. **Mobility Service extension** = auto-installed on Azure VMs during enable replication

---

## 22. Step-by-Step Configuration Mind Maps 🗺️

---

### 22.1 Create Recovery Services Vault (for ASR)

> **Portal:** `Home → + Create a resource → Search "Recovery Services vault" → Create`

```
Create Recovery Services Vault
│
├── Step 1: Basics
│   ├── Select Subscription
│   ├── Select or Create Resource Group
│   ├── Enter Vault Name
│   └── Select Region
│       ⚠️ Vault should be in TARGET region (DR destination)
│       ⚠️ Vault region must be DIFFERENT from source VMs
│
├── Step 2: Redundancy
│   ├── Choose: LRS / GRS
│   └── ⚠️ GRS recommended for DR scenarios
│
├── Step 3: Tags (Optional)
│
└── Step 4: Review + Create → Create
```

---

### 22.2 Enable Azure VM Replication (Azure-to-Azure)

> **Portal Path A:** `Recovery Services Vault → + Replicate`
> **Portal Path B:** `VM → Operations → Disaster recovery`

```
Enable Azure-to-Azure Replication
│
├── Path A: From Recovery Services Vault
│   │   Portal: Recovery Services Vault → Protected Items →
│   │           Replicated items → + Replicate → Azure virtual machines
│   │
│   ├── Step 1: Source Settings
│   │   ├── Source Region (auto-detected from selected VMs)
│   │   ├── Subscription
│   │   ├── Resource Group
│   │   └── VM Deployment Model: Resource Manager
│   │
│   ├── Step 2: Select Virtual Machines
│   │   ├── Check VMs to replicate
│   │   └── ⚠️ VMs must be running & use managed disks
│   │
│   ├── Step 3: Replication Settings
│   │   ├── Target Region
│   │   ├── Target Subscription
│   │   ├── Target Resource Group (auto-created or select existing)
│   │   ├── Target Virtual Network (auto-created or select existing)
│   │   ├── Cache Storage Account (auto-created in source region)
│   │   │   └── ⚠️ Cache storage = SOURCE region
│   │   ├── Replica Managed Disks (auto-created in target region)
│   │   ├── Target Availability Set / Zone (if applicable)
│   │   └── Replication Policy (default or custom)
│   │
│   └── Step 4: Enable Replication → Done
│       └── ⚠️ Mobility Service extension auto-installed on VMs
│
└── Path B: From VM Blade
    │   Portal: Virtual Machine → Operations → Disaster recovery
    │
    ├── Step 1: Select Target Region
    │   ├── Target region dropdown
    │   └── Advanced settings (target RG, VNet, availability, policy)
    │
    ├── Step 2: Review + Start replication
    │
    └── Step 3: Wait for initial replication to complete
        └── Monitor: VM → Disaster recovery → Replication health
```

---

### 22.3 Create / Modify Replication Policy

> **Portal:** `Recovery Services Vault → Manage → Site Recovery Infrastructure`

```
Create Replication Policy
│
├── Step 1: Navigate
│   └── Recovery Services Vault → Manage →
│       Site Recovery Infrastructure → Replication Policies
│
├── Step 2: Click "+ Replication Policy"
│
├── Step 3: Configure Policy
│   ├── Policy Name
│   ├── Source Type: Azure virtual machines
│   ├── Recovery Point Retention: 24 hrs (default)
│   │   └── Range: 0–72 hours
│   │       ⚠️ Max 72 hours retention
│   ├── App-consistent Snapshot Frequency: 4 hrs (default)
│   │   └── Range: 1–12 hours (0 = disabled)
│   │       ⚠️ 0 = only crash-consistent points (no app-consistent)
│   └── ⚠️ Crash-consistent = every 5 min (auto, not configurable)
│
└── Step 4: OK → Policy created
    └── Associate with replicated items during Enable Replication
```

---

### 22.4 Perform Test Failover

> **Portal:** `Recovery Services Vault → Replicated items`

```
Test Failover (DR Drill)
│
├── Step 1: Navigate
│   └── Recovery Services Vault → Protected Items →
│       Replicated items → Select VM
│
├── Step 2: Click "Test Failover"
│
├── Step 3: Select Recovery Point
│   ├── Latest processed (fastest — low RTO)
│   ├── Latest (lowest RPO — processes pending data first)
│   ├── Latest app-consistent (cleanest state)
│   └── Custom (specific past point)
│
├── Step 4: Select Azure Virtual Network
│   ├── Choose ISOLATED test network
│   │   └── ⚠️ Do NOT use production VNet
│   └── ⚠️ Test failover does NOT impact production replication
│
├── Step 5: OK → Test VM created in target region
│
├── Step 6: Validate
│   ├── Connect to test VM (RDP/SSH)
│   ├── Verify application works
│   └── Run DR validation tests
│
└── Step 7: Clean up Test Failover
    │   Portal: Replicated items → Select VM → Clean up test failover
    ├── Enter notes
    ├── Check "Testing is complete"
    └── OK → Test resources deleted
    ⚠️ Always clean up — test VMs incur compute charges
```

---

### 22.5 Perform Failover (Actual DR)

> **Portal:** `Recovery Services Vault → Replicated items`

```
Failover (Actual Disaster)
│
├── Step 1: Navigate
│   └── Recovery Services Vault → Protected Items →
│       Replicated items → Select VM
│
├── Step 2: Click "Failover"
│
├── Step 3: Select Recovery Point
│   ├── Latest processed (recommended for speed)
│   ├── Latest (recommended for lowest data loss)
│   ├── Latest app-consistent
│   └── Custom
│
├── Step 4: Options
│   ├── "Shut down machine before beginning failover" (optional)
│   │   └── ⚠️ Ensures latest data replicated before failover
│   │       Only works if source region is still accessible
│   └── ⚠️ If source is down, cannot shut down — use latest available point
│
├── Step 5: OK → Failover starts
│   ├── Source VM stopped (if reachable)
│   ├── Target VM created from recovery point
│   └── New VM starts in target region
│
├── Step 6: Validate Target VM
│   ├── Connect (RDP/SSH)
│   ├── Verify applications
│   └── Update DNS / Traffic Manager if needed
│
├── Step 7: Commit
│   │   Portal: Replicated items → Select VM → Commit
│   ├── Finalizes failover
│   ├── Deletes all recovery points for this VM
│   └── ⚠️ After commit, cannot change recovery point
│
└── ⚠️ If unhappy with results → "Change recovery point" before committing
```

---

### 22.6 Re-protect & Failback

> **Portal:** `Recovery Services Vault → Replicated items`

```
Re-protect & Failback
│
├── Phase 1: Re-protect (reverse replication direction)
│   │   Portal: Replicated items → Select VM → Re-protect
│   │
│   ├── Step 1: Click "Re-protect"
│   ├── Step 2: Confirm settings
│   │   ├── New Source: former target region
│   │   ├── New Target: former source region (original)
│   │   ├── Cache storage account in new source
│   │   └── Target VNet in original region
│   ├── Step 3: OK → Reverse replication starts
│   └── Step 4: Wait for initial replication to complete
│       ⚠️ Takes time — original region resources re-synced
│
├── Phase 2: Failback (failover to original region)
│   │   Portal: Replicated items → Select VM → Failover
│   │
│   ├── Step 1: Click "Failover"
│   ├── Step 2: Select Recovery Point
│   ├── Step 3: OK → VM fails back to original region
│   ├── Step 4: Validate original VM
│   └── Step 5: Commit
│
└── Phase 3: Re-protect again (original direction)
    │   Portal: Replicated items → Select VM → Re-protect
    ├── Restores original replication direction
    │   (source → target, normal DR posture)
    └── ⚠️ Full cycle: Failover → Commit → Re-protect →
        Failback → Commit → Re-protect (back to normal)
```

---

### 22.7 Create Recovery Plan

> **Portal:** `Recovery Services Vault → Manage → Recovery Plans`

```
Create Recovery Plan
│
├── Step 1: Navigate
│   └── Recovery Services Vault → Manage →
│       Recovery Plans (Site Recovery) → + Recovery Plan
│
├── Step 2: Basics
│   ├── Recovery Plan Name
│   ├── Source: Select source location (e.g., East US)
│   ├── Target: Select target location (e.g., West US)
│   └── Deployment Model: Resource Manager
│
├── Step 3: Select Items
│   ├── Select VMs to include in plan
│   └── ⚠️ Max 100 VMs per recovery plan
│
├── Step 4: OK → Plan created with default Group 1
│
├── Step 5: Customize Plan
│   │   Portal: Recovery Plans → Select Plan → Customize
│   │
│   ├── Add Groups (Group 1, Group 2, Group 3…)
│   │   └── Move VMs between groups to set failover order
│   │       ⚠️ Group 1 completes before Group 2 starts
│   │
│   ├── Add Pre-action (before group failover)
│   │   ├── Manual Action: note for operator
│   │   └── Script: Azure Automation Runbook
│   │       ⚠️ Runbook must be in Azure Automation Account
│   │
│   └── Add Post-action (after group failover)
│       ├── Manual Action
│       └── Script: Automation Runbook
│           Examples: Add NSG, Update DNS, Configure LB
│
└── Step 6: Save → Plan ready
    └── Execute: Recovery Plans → Select → Test Failover / Failover
```

---

### 22.8 Configure Network Mapping

> **Portal:** `Recovery Services Vault → Manage → Site Recovery Infrastructure`

```
Configure Network Mapping
│
├── Step 1: Navigate
│   └── Recovery Services Vault → Manage →
│       Site Recovery Infrastructure → Network Mapping
│
├── Step 2: Click "+ Network Mapping"
│
├── Step 3: Add Network Mapping
│   ├── Source: Select source location
│   ├── Target: Select target location
│   ├── Source Virtual Network: Select source VNet
│   └── Target Virtual Network: Select target VNet
│
├── Step 4: OK → Mapping created
│
├── Customize Per-VM Network Settings
│   │   Portal: Replicated items → Select VM →
│   │           Compute and Network → Edit
│   ├── Target VNet / Subnet per NIC
│   ├── Target IP address (static or dynamic)
│   ├── Target VM size
│   ├── Target Availability Set / Zone
│   ├── Public IP: Create new or none
│   └── NSG: Select or none
│
└── ⚠️ Reverse mapping auto-created during Re-protect
    ⚠️ Target VNet must exist before mapping
```

---

### 22.9 Configure NSG for ASR Connectivity

> **Portal:** `NSG → Settings → Outbound security rules`

```
Configure NSG for ASR
│
├── Step 1: Navigate to Source VM's NSG
│   └── NSG → Settings → Outbound security rules
│
├── Step 2: Add Rule for ASR Service Tag
│   ├── + Add
│   ├── Source: Any
│   ├── Source port: *
│   ├── Destination: Service Tag
│   ├── Destination Service Tag: SiteRecovery
│   ├── Destination port: 443
│   ├── Protocol: TCP
│   ├── Action: Allow
│   ├── Priority: e.g., 100
│   └── Name: e.g., Allow-ASR-Outbound
│
├── Step 3: Add Rule for Storage Service Tag
│   ├── Destination Service Tag: Storage.<region>
│   ├── Port: 443
│   └── Action: Allow
│
├── Step 4: Add Rule for Azure AD
│   ├── Destination Service Tag: AzureActiveDirectory
│   ├── Port: 443
│   └── Action: Allow
│
└── ⚠️ Port 443 (HTTPS) for all ASR communication
    ⚠️ Service Tags simplify NSG rules (no IP management)
```

---

### 22.10 Monitor ASR Replication Health

> **Portal:** `Recovery Services Vault → Overview → Site Recovery`

```
Monitor ASR Replication
│
├── Dashboard View
│   │   Portal: Recovery Services Vault → Overview →
│   │           Site Recovery section
│   ├── Replicated items count
│   ├── Failover health status
│   ├── Configuration issues
│   └── Recovery Plan status
│
├── Per-VM Replication Status
│   │   Portal: Replicated items → Select VM
│   ├── Health: Normal / Warning / Critical
│   ├── RPO: Current replication lag
│   ├── Latest recovery point timestamp
│   ├── Failover readiness: ✓ or issues
│   ├── Active/Latest test failover date
│   └── Replication events log
│
├── Site Recovery Jobs
│   │   Portal: Recovery Services Vault → Monitoring →
│   │           Site Recovery Jobs
│   ├── View all ASR jobs (enable, failover, re-protect, etc.)
│   ├── Filter by status: In progress / Succeeded / Failed
│   └── Click job for detailed steps
│
└── Configure Alerts
    │   Portal: Recovery Services Vault → Monitoring → Alerts
    ├── + Create Alert Rule
    ├── Signal: Site Recovery events / RPO breach
    ├── Select Action Group
    └── Create
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
