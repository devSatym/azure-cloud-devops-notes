<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Backup — AZ-104 Revision Notes

---

## 1. What is Azure Backup?

- **Centralized, cloud-based backup service** (BaaS — Backup as a Service)
- Replaces on-premises/traditional backup solutions
- Zero-infrastructure backup — no maintenance of backup servers
- Supports **on-premises, Azure VMs, Azure Files, SQL in Azure VM, SAP HANA, Azure Blobs, Azure Disks, Azure Database for PostgreSQL**
- Uses **Recovery Services Vault** or **Backup Vault** as the storage entity

---

## 2. Key Components

| Component | Purpose |
|---|---|
| **Recovery Services Vault** | Stores backup data for Azure VMs, Azure Files, SQL/SAP in VM, on-prem |
| **Backup Vault** | Stores backup data for Azure Blobs, Azure Disks, Azure Database for PostgreSQL |
| **Backup Policy** | Defines schedule + retention (daily/weekly/monthly/yearly) |
| **MARS Agent** | Microsoft Azure Recovery Services agent — backs up on-prem files/folders/system state to Azure |
| **MABS** | Microsoft Azure Backup Server — backs up on-prem workloads (VMs, SQL, SharePoint) |
| **DPM** | System Center Data Protection Manager — enterprise on-prem backup |
| **Backup Extension** | Installed automatically on Azure VM for VM-level backup |

---

## 3. Recovery Services Vault vs Backup Vault

| Feature | Recovery Services Vault | Backup Vault |
|---|---|---|
| Supports | Azure VM, Azure Files, SQL in VM, SAP HANA, MARS, MABS/DPM | Azure Blobs, Azure Disks, PostgreSQL |
| Storage Redundancy | LRS, GRS, ZRS, RA-GRS | LRS, GRS, ZRS |
| Cross Region Restore | Yes (with GRS) | Depends on workload |
| Soft Delete | Yes (14 days default) | Yes |

---

## 4. Storage Redundancy Options

| Type | Description | Use Case |
|---|---|---|
| **LRS** | 3 copies in single data center | Low cost, non-critical |
| **ZRS** | 3 copies across availability zones | High availability in same region |
| **GRS** | 6 copies (3 primary + 3 paired region) | Disaster recovery |
| **RA-GRS** | GRS + read access to secondary | Cross Region Restore |

> ⚠️ **EXAM TIP:** Storage redundancy can ONLY be changed **before** any backup items are configured. Once a backup item is protected, redundancy is **locked**.

**Portal Path:**
`Recovery Services Vault → Properties → Backup Storage Redundancy → Update`

---

## 5. Azure VM Backup

### What Gets Backed Up
- **Entire VM** (all disks — OS + Data disks)
- Application-consistent snapshots (Windows — VSS, Linux — custom pre/post scripts)
- **Crash-consistent** if scripts fail

### How It Works
1. Snapshot taken → stored locally (Instant Restore Tier / Snapshot tier)
2. Snapshot transferred to vault (Vault Tier)
3. Restore from either tier

### Snapshot Retention
- **Instant Restore:** 1–5 days (Standard policy), up to 30 days (Enhanced policy)
- Faster restore from snapshot tier (no need to pull from vault)

### Backup Frequency
| Policy Type | Frequency |
|---|---|
| Standard | Once per day |
| Enhanced | Multiple times per day (every 4, 6, 8, 12, or 24 hours) |

### Retention
- **Daily:** 7–9999 days
- **Weekly:** 1–5163 weeks
- **Monthly:** 1–1188 months
- **Yearly:** 1–99 years

### Portal Path — Enable Azure VM Backup
```
Recovery Services Vault → + Backup → Where: Azure, What: Virtual Machine →
Select VMs → Choose Backup Policy → Enable Backup
```

### Portal Path — Backup Existing VM (from VM blade)
```
Virtual Machine → Operations → Backup → Select/Create RSV →
Choose Policy → Enable Backup
```

### Portal Path — On-Demand Backup
```
Recovery Services Vault → Backup Items → Azure Virtual Machine →
Select VM → Backup Now → Set retention date → OK
```

---

## 6. Enhanced Backup Policy (Multi-Tier)

- Supports **multiple backups per day** (every 4/6/8/12/24 hrs)
- Supports **Zone-redundant snapshots**
- Supports **Premium SSD v2 & Ultra Disk**
- Instant Restore retention **up to 30 days**
- Only for **Azure VMs** (Trusted Launch, Confidential VMs supported)

> ⚠️ **EXAM TIP:** Enhanced policy is required if you need **multiple backups per day** or backup of **Ultra Disk/Premium SSD v2**.

---

## 7. Azure Files Backup

- **Snapshot-based backup** (share-level snapshots)
- RSV must be in same region as Storage Account
- Supports **Standard & Premium** file shares
- Retention: Daily, Weekly, Monthly, Yearly
- **No agent required**
- Vaulted backup (preview) copies data to vault tier

### Portal Path
```
Recovery Services Vault → + Backup → Where: Azure, What: Azure File Share →
Select Storage Account → Select File Shares → Choose Policy → Enable Backup
```

---

## 8. SQL Server in Azure VM Backup

- **Autoprotect** — automatically protects new databases added to instance
- **Full, Differential, Log** backups
- RPO of **15 minutes** (log backup every 15 min)
- Full backup: Daily/Weekly
- Differential: every 12 hours (not configurable interval)
- Log backup: every 15 min to 2 hours
- **Long-term retention (LTR)** support

### Portal Path
```
Recovery Services Vault → + Backup → Where: Azure, What: SQL Server in Azure VM →
Discover DBs → Select Instances/DBs → Configure Backup → Choose Policy → OK
```

> ⚠️ **EXAM TIP:** The **Workload backup extension** is installed on the VM during SQL discovery. The VM must have **network connectivity** to Azure.

---

## 9. SAP HANA in Azure VM Backup

- Similar to SQL backup workflow
- Supports Full, Differential, Incremental, Log backups
- Log backup every 15 min
- **Backint certified** by SAP

### Portal Path
```
Recovery Services Vault → + Backup → Where: Azure, What: SAP HANA in Azure VM →
Discover → Configure Backup
```

---

## 10. Azure Blob Backup (Operational & Vaulted)

| Type | Storage | Protection |
|---|---|---|
| **Operational Backup** | Data stays in source storage account | Continuous, point-in-time restore (up to 360 days) |
| **Vaulted Backup** | Data copied to Backup Vault | Scheduled, discrete recovery points |

- Uses **Backup Vault** (NOT Recovery Services Vault)
- Operational backup — protects against accidental deletion of blobs
- Requires **Storage Account Backup Contributor** role on the storage account

### Portal Path
```
Backup Vault → + Backup → Datasource type: Azure Blobs →
Select Backup Vault → Choose Policy → Configure Backup →
Select Storage Account → Validate → Configure Backup
```

---

## 11. Azure Disk Backup

- **Incremental snapshots** (only changed data since last snapshot)
- Uses **Backup Vault**
- Backup frequency: Every 1, 2, 4, 6, 8, 12 hours (or daily)
- **Only supports Managed Disks**
- Requires **Disk Backup Reader** role on disk, **Disk Snapshot Contributor** on snapshot RG

### Portal Path
```
Backup Vault → + Backup → Datasource type: Azure Disks →
Select Policy → Add/Select Disks → Configure Snapshot RG → Configure Backup
```

---

## 12. On-Premises Backup with MARS Agent

- Backs up **Files, Folders, System State** from on-prem Windows to Azure
- Direct to Recovery Services Vault (no MABS/DPM needed)
- **Maximum 3 backups per day**
- **No Linux support** (MARS = Windows only)
- Max file size: **54,400 GB**
- Encryption: data encrypted in transit and at rest (passphrase required)

### Portal Path (Download & Register Agent)
```
Recovery Services Vault → + Backup → Where: On-premises, What: Files and folders →
Download Agent → Download Vault Credentials → Install & Register Agent on machine
```

---

## 13. MABS / DPM (On-Premises Workloads)

| Feature | MABS | DPM |
|---|---|---|
| License | Free (no System Center) | Requires System Center license |
| Workloads | VMs, SQL, SharePoint, Exchange, Files, System State, BMR | Same + more granular |
| OS | Windows Server | Windows Server |
| Backs up to | Disk → Cloud (RSV) | Disk → Tape → Cloud (RSV) |

---

## 14. Backup Center

- **Single pane of glass** to manage all backups across subscriptions, vaults, workloads
- Supports both Recovery Services Vault & Backup Vault
- View, monitor, configure, operate backups in one place

### Portal Path
```
Search "Backup Center" → Overview / Backup Instances / Backup Policies /
Backup Jobs / Backup Alerts
```

> ⚠️ **EXAM TIP:** Backup Center is the **governance and management** layer — it doesn't replace vaults; it provides visibility across vaults.

---

## 15. Restore Options

### Azure VM Restore Options

| Option | Description |
|---|---|
| **Create a new VM** | Quickly spin up a VM from restore point |
| **Restore Disk** | Restore disks to a storage account, then manually create VM |
| **Replace existing** | Replace disks of existing VM (VM must exist) |
| **Cross Region Restore** | Restore to paired region (GRS vault required) |

### File-Level Recovery (Azure VM)
- Mount recovery point disks on a machine → browse & copy specific files
- Script provided by Azure — Windows (`.exe`), Linux (`.py`)
- Connection valid for **12 hours**

### Portal Path — Restore VM
```
Recovery Services Vault → Backup Items → Azure Virtual Machine →
Select VM → Restore VM → Choose Restore Point → Select Restore Type →
(Create new / Restore disk / Replace existing) → Restore
```

### Portal Path — File Recovery
```
Recovery Services Vault → Backup Items → Azure Virtual Machine →
Select VM → File Recovery → Select Recovery Point →
Download Script → Run on Machine → Browse & Copy Files
```

### Cross Region Restore (CRR)
```
Recovery Services Vault → Backup Items → Azure Virtual Machine →
Select VM → Restore VM → Toggle "Cross Region Restore" →
Select secondary region restore point → Restore
```

> ⚠️ **EXAM TIP:** CRR must be **enabled** on the vault (Properties → Cross Region Restore → Enable). Requires **GRS** redundancy. Data in secondary region may be delayed up to **12 hours** (RPO).

---

## 16. Soft Delete

- **Deleted backup data retained** for additional **14 days** (total 14 + expiry)
- Enabled **by default** on Recovery Services Vault
- States: Active → Soft Deleted → Permanently Deleted
- Can be **Undeleted** within 14 days
- **Enhanced Soft Delete:** 14–180 days retention, soft delete for Azure Blobs/Disks/other workloads
- **Always-on Soft Delete:** Cannot be disabled (irreversible protection for vault)

### Portal Path
```
Recovery Services Vault → Properties → Soft Delete and Security Settings →
Update → Set retention period → Save
```

> ⚠️ **EXAM TIP:** Soft delete is **free** — no additional backup storage charges during soft-delete period.

---

## 17. Multi-User Authorization (MUA) with Resource Guard

- Adds **additional layer of authorization** for critical operations
- Uses **Azure Resource Guard** (ARM resource)
- Critical operations (disable soft delete, stop backup with delete data, reduce retention) require **both** Backup Admin AND Resource Guard owner approval
- Resource Guard should be in a **different subscription** (separation of duties)

### Portal Path
```
Recovery Services Vault → Properties → Multi-User Authorization →
Update → Select Resource Guard → Save
```

---

## 18. Backup Alerts & Monitoring

### Alert Types
| Type | Source |
|---|---|
| **Built-in Azure Monitor Alerts** | Automatic for backup failures, security events (recommended) |
| **Classic Alerts** | Legacy, configured per vault |
| **Custom Log Alerts** | Via Log Analytics + alert rules |

### Diagnostic Settings → Send to Log Analytics
```
Recovery Services Vault → Monitoring → Diagnostic Settings →
Add Diagnostic Setting → Choose logs (CoreAzureBackup, AddonAzureBackupJobs, etc.) →
Send to Log Analytics Workspace → Save
```

### Backup Reports
```
Recovery Services Vault → Monitoring → Backup Reports →
Select Log Analytics Workspace → View Reports
(OR)
Backup Center → Backup Reports
```

> ⚠️ **EXAM TIP:** Backup Reports require **Log Analytics Workspace**. Data may take up to **24 hours** to appear in reports.

---

## 19. Backup Policies — Key Points

- **Default policy:** Backup daily at a scheduled time, retain 30 days
- Custom policies allow granular schedule + retention
- Retention: Daily, Weekly, Monthly (day of month), Yearly (day of year)
- **Instant Restore** is snapshot tier (faster, costs as snapshot storage)
- **Time Zone** must be set when creating policy

### Portal Path — Create / Modify Policy
```
Recovery Services Vault → Manage → Backup Policies → + Add →
Select Workload Type → Define Schedule & Retention → Create
```

---

## 20. Security Features Summary

| Feature | Purpose |
|---|---|
| **Soft Delete** | Protects against accidental/malicious deletion (14 days default) |
| **Always-on Soft Delete** | Soft delete that cannot be disabled |
| **MUA (Resource Guard)** | Dual authorization for critical operations |
| **Immutable Vault** | Backup data cannot be deleted before expiry (Locked = irreversible) |
| **Encryption** | At rest (SSE) — Platform-managed or Customer-managed keys (CMK) |
| **Private Endpoints** | Secure vault access over private network |
| **RBAC** | Built-in roles: Backup Contributor, Backup Operator, Backup Reader |

### Immutable Vault
```
Recovery Services Vault → Properties → Immutable Vault →
Enable → Locked (irreversible) / Unlocked (can disable later)
```

> ⚠️ **EXAM TIP:** Once immutable vault is **Locked**, it CANNOT be unlocked or disabled. Make sure you understand the **Locked vs Unlocked** state.

---

## 21. RBAC Roles for Backup

| Role | Permissions |
|---|---|
| **Backup Contributor** | All backup actions EXCEPT delete vault, create/manage policies for others |
| **Backup Operator** | Everything except removing backup, manage policies |
| **Backup Reader** | View all backup operations (read-only) |
| **Virtual Machine Contributor** | Can backup VM (includes backup operations) |

> ⚠️ **EXAM TIP:** To create a Recovery Services Vault, you need **Contributor** permissions on the resource group.

---

## 22. Private Endpoints for Backup

- Allows RSV to communicate over **private IP** (via VNet)
- Requires Private DNS Zone for backup-related FQDNs
- Minimum **2 private endpoints** required (vault + blob)
- NSG and firewall rules must allow private endpoint traffic

### Portal Path
```
Recovery Services Vault → Settings → Networking →
Private Endpoint Connections → + Private Endpoint →
Select VNet/Subnet → Integrate with Private DNS Zone → Create
```

---

## 23. Pricing — Key Points

- **Protected Instance fee** (per VM/workload/month)
- **Backup storage** charges (based on redundancy: LRS < ZRS < GRS)
- **Instant Restore:** Snapshot storage charged separately (based on snapshot size & retention days)
- **Soft Delete:** No extra charge for 14-day retention
- **Cross Region Restore:** GRS storage pricing applies

---

## 24. Exam Quick-Fire Points ⚡

1. Recovery Services Vault ≠ Backup Vault — know which workload uses which
2. **MARS agent** = Windows only, files/folders/system state, no Linux
3. Storage redundancy **locked** after first backup item configured
4. **CRR** needs GRS, RPO up to 12 hours
5. **Enhanced policy** = multiple backups/day, Ultra disk support
6. SQL backup RPO = **15 minutes** (log backups)
7. Blob operational backup = **continuous** (point-in-time restore up to 360 days)
8. Azure Disk backup = **incremental snapshots**, uses Backup Vault
9. File Recovery script valid for **12 hours**
10. Soft delete = **14 days** default, free, enabled by default
11. Immutable Vault **Locked** state = irreversible
12. MUA needs Resource Guard in **different subscription**
13. Backup Reports need **Log Analytics Workspace**, 24-hour data delay
14. **Backup Center** = single pane of glass for all backup management
15. Azure Files backup = **snapshot-based**, same region as RSV
16. Enhanced Soft Delete = 14–180 days configurable
17. Instant Restore snapshots: Standard (1–5 days), Enhanced (up to 30 days)
18. VM restore options: **New VM, Restore Disk, Replace Existing, CRR**
19. Minimum private endpoints for vault = **2** (vault + blob communication)
20. Azure Backup supports **application-consistent** (VSS/scripts) and **crash-consistent** snapshots

---

## 25. Step-by-Step Configuration Mind Maps 🗺️

---

### 25.1 Create Recovery Services Vault

> **Portal:** `Home → + Create a resource → Search "Recovery Services vault" → Create`

```
Create Recovery Services Vault
│
├── Step 1: Basics
│   ├── Select Subscription
│   ├── Select or Create Resource Group
│   ├── Enter Vault Name
│   └── Select Region
│       ⚠️ Must be same region as resources to backup
│
├── Step 2: Redundancy
│   │   Portal: Vault → Properties → Backup Storage Redundancy → Update
│   ├── Choose: LRS / GRS / ZRS / RA-GRS
│   └── ⚠️ Must set BEFORE configuring any backup items (locked after)
│
├── Step 3: Tags (Optional)
│   └── Add Name-Value pairs for organization
│
└── Step 4: Review + Create → Create
```

---

### 25.2 Create Backup Vault

> **Portal:** `Home → + Create a resource → Search "Backup vault" → Create`

```
Create Backup Vault
│
├── Step 1: Basics
│   ├── Select Subscription
│   ├── Select or Create Resource Group
│   ├── Enter Vault Name
│   ├── Select Region
│   └── Select Redundancy: LRS / GRS / ZRS
│       ⚠️ Must set BEFORE first backup (locked after)
│
├── Step 2: Tags (Optional)
│
└── Step 3: Review + Create → Create
```

---

### 25.3 Configure Azure VM Backup

> **Portal Path A:** `Recovery Services Vault → + Backup`
> **Portal Path B:** `Virtual Machine → Operations → Backup`

```
Azure VM Backup Configuration
│
├── Path A: From Recovery Services Vault
│   │   Portal: Recovery Services Vault → Overview → + Backup
│   │
│   ├── Step 1: Backup Goal
│   │   ├── Where is your workload running? → Azure
│   │   └── What do you want to backup? → Virtual Machine
│   │
│   ├── Step 2: Select Policy
│   │   ├── Choose existing policy (Default = daily, 30-day retention)
│   │   └── OR Create new policy
│   │       ├── Policy Name
│   │       ├── Policy type: Standard / Enhanced
│   │       ├── Backup Frequency (Standard: Daily | Enhanced: 4/6/8/12/24 hrs)
│   │       ├── Backup Time & Time Zone
│   │       ├── Instant Restore Snapshots retention (1–5 or up to 30 days)
│   │       └── Retention: Daily / Weekly / Monthly / Yearly
│   │
│   ├── Step 3: Select Virtual Machines
│   │   ├── Choose VMs from same region as vault
│   │   └── ⚠️ VM must be in SAME region as RSV
│   │
│   └── Step 4: Enable Backup → Done
│
└── Path B: From VM Blade
    │   Portal: Virtual Machine → Operations → Backup
    │
    ├── Step 1: Select Recovery Services Vault
    │   └── Choose existing OR Create new RSV
    │
    ├── Step 2: Choose Backup Policy
    │
    └── Step 3: Enable Backup → Done
```

---

### 25.4 On-Demand Backup (Backup Now)

> **Portal:** `Recovery Services Vault → Protected Items → Backup items → Azure Virtual Machine`

```
On-Demand Backup
│
├── Step 1: Navigate to Backup Items
│   └── Recovery Services Vault → Protected Items → Backup items
│
├── Step 2: Select Backup Management Type
│   └── Azure Virtual Machine (or other workload)
│
├── Step 3: Click on the specific VM
│
├── Step 4: Click "Backup Now"
│   ├── Set Retain Backup Till date
│   └── ⚠️ Cannot exceed policy max retention
│
└── Step 5: OK → Backup job starts
```

---

### 25.5 Configure Azure Files Backup

> **Portal:** `Recovery Services Vault → + Backup`

```
Azure Files Backup Configuration
│
├── Step 1: Backup Goal
│   │   Portal: Recovery Services Vault → + Backup
│   ├── Where is your workload running? → Azure
│   └── What do you want to backup? → Azure FileShare
│
├── Step 2: Select Storage Account
│   ├── Choose Storage Account (same region as RSV)
│   └── ⚠️ Only shows storage accounts in same region & same subscription
│
├── Step 3: Select File Shares
│   └── Check file shares to protect
│
├── Step 4: Select or Create Backup Policy
│   ├── Backup Frequency: Daily / Hourly
│   └── Retention: Daily / Weekly / Monthly / Yearly
│
└── Step 5: Enable Backup → Done
```

---

### 25.6 Configure SQL Server in Azure VM Backup

> **Portal:** `Recovery Services Vault → + Backup`

```
SQL Server in Azure VM Backup
│
├── Step 1: Backup Goal
│   │   Portal: Recovery Services Vault → + Backup
│   ├── Where is your workload running? → Azure
│   └── What do you want to backup? → SQL Server in Azure VM
│
├── Step 2: Discover DBs
│   ├── Click "Start Discovery"
│   ├── Azure installs Workload Backup Extension on VM
│   └── ⚠️ VM needs network connectivity to Azure
│
├── Step 3: Configure Backup
│   ├── Select SQL Instances or individual databases
│   ├── Enable "AutoProtect" on instance (optional)
│   │   └── Auto-protects future databases added to this instance
│   └── Select Backup Policy
│       ├── Full backup: Daily or Weekly
│       ├── Differential: Every 12 hours (fixed)
│       ├── Log backup: Every 15 min to 2 hours
│       └── Retention: Daily / Weekly / Monthly / Yearly
│
└── Step 4: OK → Backup configured
```

---

### 25.7 Configure SAP HANA in Azure VM Backup

> **Portal:** `Recovery Services Vault → + Backup`

```
SAP HANA in Azure VM Backup
│
├── Step 1: Backup Goal
│   │   Portal: Recovery Services Vault → + Backup
│   ├── Where is your workload running? → Azure
│   └── What do you want to backup? → SAP HANA in Azure VM
│
├── Step 2: Discover HANA Instances
│   ├── Click "Discover DBs"
│   ├── Run pre-registration script on VM
│   └── Workload extension installed
│
├── Step 3: Configure Backup
│   ├── Select HANA databases
│   └── Select Backup Policy
│       ├── Full: Daily / Weekly
│       ├── Differential / Incremental
│       ├── Log: Every 15 min
│       └── Retention settings
│
└── Step 4: OK → Done
```

---

### 25.8 Configure Azure Blob Backup

> **Portal:** `Backup Vault → + Backup`

```
Azure Blob Backup Configuration
│
├── Step 1: Backup Goal
│   │   Portal: Backup Vault → + Backup
│   └── Datasource type: Azure Blobs (Azure Storage)
│
├── Step 2: Select Backup Vault
│   └── Choose existing Backup Vault
│
├── Step 3: Select Backup Policy
│   ├── Choose existing OR Create new
│   ├── Operational backup: Continuous (point-in-time up to 360 days)
│   └── Vaulted backup: Scheduled discrete recovery points
│
├── Step 4: Configure Backup
│   ├── Select Storage Account(s)
│   ├── Validate — checks RBAC permissions
│   │   └── ⚠️ Requires "Storage Account Backup Contributor" role
│   └── Assign missing roles if prompted
│
└── Step 5: Configure Backup → Done
```

---

### 25.9 Configure Azure Disk Backup

> **Portal:** `Backup Vault → + Backup`

```
Azure Disk Backup Configuration
│
├── Step 1: Backup Goal
│   │   Portal: Backup Vault → + Backup
│   └── Datasource type: Azure Disks
│
├── Step 2: Select Backup Vault
│
├── Step 3: Select Backup Policy
│   ├── Choose existing OR Create new
│   ├── Frequency: Every 1/2/4/6/8/12 hours or Daily
│   └── Retention: 1–7 days (or more based on policy)
│
├── Step 4: Select Disks
│   ├── + Add → Select Managed Disks
│   └── ⚠️ Only Managed Disks supported
│
├── Step 5: Configure Snapshot Resource Group
│   ├── Select RG where snapshots will be stored
│   ├── Validate — checks RBAC
│   │   ├── "Disk Backup Reader" on source disk
│   │   └── "Disk Snapshot Contributor" on snapshot RG
│   └── Assign missing roles if prompted
│
└── Step 6: Configure Backup → Done
```

---

### 25.10 Configure On-Premises Backup (MARS Agent)

> **Portal:** `Recovery Services Vault → + Backup`

```
On-Premises Backup (MARS Agent)
│
├── Step 1: Backup Goal
│   │   Portal: Recovery Services Vault → + Backup
│   ├── Where is your workload running? → On-premises
│   └── What do you want to backup? → Files and folders
│       (Also options: System State, Hyper-V, VMware, etc.)
│
├── Step 2: Prepare Infrastructure
│   ├── Download MARS Agent (.exe)
│   │   └── Install on Windows on-prem server
│   └── Download Vault Credentials file
│       └── Valid for 10 days after download
│
├── Step 3: Install & Register Agent (on-prem machine)
│   ├── Run MARSAgentInstaller.exe
│   ├── Register Server → Browse vault credentials file
│   ├── Set Encryption Passphrase
│   │   └── ⚠️ SAVE THIS — cannot recover data without it
│   └── Finish registration
│
├── Step 4: Schedule Backup (from MARS Agent Console)
│   ├── Open Microsoft Azure Backup console on machine
│   ├── Schedule Backup → Select files/folders
│   ├── Set schedule (max 3x per day)
│   ├── Set retention policy
│   └── Confirm → Done
│
└── ⚠️ MARS = Windows ONLY, No Linux support
```

---

### 25.11 Restore Azure VM

> **Portal:** `Recovery Services Vault → Protected Items → Backup items → Azure Virtual Machine`

```
Restore Azure VM
│
├── Step 1: Navigate to Backup Item
│   └── Recovery Services Vault → Protected items → Backup items
│       → Azure Virtual Machine → Select VM
│
├── Step 2: Click "Restore VM"
│
├── Step 3: Select Restore Point
│   ├── Choose from available recovery points
│   ├── Filter by: Crash consistent / Application consistent
│   └── Snapshot tier (fast) vs Vault tier
│
├── Step 4: Select Restore Configuration
│   │
│   ├── Option A: Create new virtual machine
│   │   ├── Enter VM name
│   │   ├── Select Resource Group
│   │   ├── Select Virtual Network & Subnet
│   │   └── Select Staging Storage Account
│   │
│   ├── Option B: Restore disk
│   │   ├── Select target Resource Group
│   │   ├── Select Staging Storage Account
│   │   └── Manually create VM from restored disks later
│   │
│   └── Option C: Replace existing
│       ├── Select existing VM (must be running)
│       ├── Select Staging Storage Account
│       └── ⚠️ VM must exist — replaces disks only
│
└── Step 5: Restore → Job starts
```

---

### 25.12 File-Level Recovery (Azure VM)

> **Portal:** `Recovery Services Vault → Protected Items → Backup items → Azure Virtual Machine`

```
File-Level Recovery
│
├── Step 1: Navigate to Backup Item
│   └── Recovery Services Vault → Backup items → Azure Virtual Machine
│       → Select VM
│
├── Step 2: Click "File Recovery"
│
├── Step 3: Select Recovery Point
│   └── Choose from available restore points
│
├── Step 4: Download Script
│   ├── Windows → .exe file
│   └── Linux → .py file
│
├── Step 5: Run Script on Target Machine
│   ├── Mounts recovery point disks as local drives
│   ├── Browse and copy required files
│   └── ⚠️ Connection valid for 12 hours only
│
└── Step 6: Unmount Disks (click "Unmount Disks" in portal)
```

---

### 25.13 Cross Region Restore (CRR)

> **Portal:** `Recovery Services Vault → Properties → Cross Region Restore`

```
Cross Region Restore (CRR)
│
├── Pre-requisite: Enable CRR on Vault
│   │   Portal: Recovery Services Vault → Properties →
│   │           Cross Region Restore → Enable → Save
│   ├── ⚠️ Vault must use GRS redundancy
│   ├── ⚠️ Once enabled, CANNOT be disabled
│   └── ⚠️ Data in secondary region delayed up to 12 hours (RPO)
│
├── Step 1: Navigate to Backup Item
│   └── Recovery Services Vault → Backup items → Azure Virtual Machine
│
├── Step 2: Click "Restore VM"
│
├── Step 3: Toggle "Cross Region Restore" ON
│
├── Step 4: Select Secondary Region Restore Point
│
├── Step 5: Configure Restore
│   ├── Create new VM in secondary region
│   └── OR Restore disk in secondary region
│
└── Step 6: Restore → Job runs in paired region
```

---

### 25.14 Configure Soft Delete

> **Portal:** `Recovery Services Vault → Properties → Soft Delete and Security Settings`

```
Configure Soft Delete
│
├── Step 1: Navigate
│   └── Recovery Services Vault → Properties →
│       Soft Delete and Security Settings → Update
│
├── Step 2: Configure Settings
│   ├── Enable/Disable Soft Delete for cloud workloads
│   ├── Enable/Disable Soft Delete for hybrid workloads (MARS/MABS/DPM)
│   ├── Set retention period: 14–180 days
│   └── Enable "Always-on" (optional)
│       └── ⚠️ Once enabled, CANNOT be disabled (irreversible)
│
├── Step 3: Save
│
├── Undelete a Soft-Deleted Item
│   │   Portal: Backup items → Select soft-deleted item → Undelete
│   └── Item returns to "Stop protection with retain data" state
│       → Resume backup to make it active again
│
└── ⚠️ Soft delete = FREE (no extra storage charges)
```

---

### 25.15 Configure Multi-User Authorization (MUA)

> **Portal:** `Recovery Services Vault → Properties → Multi-User Authorization`

```
Multi-User Authorization (MUA) Setup
│
├── Pre-requisite: Create Resource Guard
│   │   Portal: Home → + Create a resource → Search "Resource Guard" → Create
│   ├── Select Subscription (DIFFERENT from vault — separation of duties)
│   ├── Select Resource Group
│   ├── Enter Name & Region
│   │   └── ⚠️ Must be same region as Recovery Services Vault
│   └── Select protected operations to guard
│       ├── Disable soft delete
│       ├── Stop backup with delete data
│       ├── Reduce retention of backup policy
│       └── Change backup policy
│
├── Step 1: Link Resource Guard to Vault
│   │   Portal: Recovery Services Vault → Properties →
│   │           Multi-User Authorization → Update
│   ├── Select the Resource Guard
│   └── Save
│
├── How It Works:
│   ├── Critical operation attempted by Backup Admin
│   ├── Request blocked → requires Resource Guard Owner approval
│   └── Both roles must approve for operation to succeed
│
└── ⚠️ Resource Guard owner ≠ Backup Admin (separation of duties)
```

---

### 25.16 Configure Immutable Vault

> **Portal:** `Recovery Services Vault → Properties → Immutable vault`

```
Immutable Vault Configuration
│
├── Step 1: Navigate
│   └── Recovery Services Vault → Properties → Immutable vault
│
├── Step 2: Enable Immutability
│   ├── Click "Enable" → Settings → Enable
│   └── Choose State:
│       │
│       ├── Unlocked (Reversible)
│       │   ├── Can be disabled later
│       │   └── Good for testing
│       │
│       └── Locked (Irreversible)
│           ├── ⚠️ CANNOT be undone — permanent
│           ├── Backup data cannot be deleted before expiry
│           ├── Retention cannot be reduced
│           └── Backup policy cannot be changed to reduce protection
│
├── Step 3: Apply → Save
│
└── ⚠️ EXAM TIP: Locked = irreversible, Unlocked = can disable
```

---

### 25.17 Configure Private Endpoints for Backup

> **Portal:** `Recovery Services Vault → Settings → Networking`

```
Private Endpoints for Backup
│
├── Step 1: Navigate
│   └── Recovery Services Vault → Settings → Networking →
│       Private endpoint connections tab
│
├── Step 2: Create Private Endpoint
│   │   Click "+ Private Endpoint"
│   │
│   ├── Step 2a: Basics
│   │   ├── Select Subscription
│   │   ├── Select Resource Group
│   │   ├── Enter Private Endpoint Name
│   │   └── Select Region
│   │
│   ├── Step 2b: Resource
│   │   ├── Resource type: Microsoft.RecoveryServices/vaults
│   │   ├── Resource: Select your RSV
│   │   └── Target sub-resource: AzureBackup
│   │
│   ├── Step 2c: Virtual Network
│   │   ├── Select Virtual Network
│   │   ├── Select Subnet
│   │   └── Private IP: Dynamic (auto) or Static
│   │
│   ├── Step 2d: DNS
│   │   ├── Integrate with Private DNS zone: Yes
│   │   └── Select or Create Private DNS Zones
│   │       ├── privatelink.<geo>.backup.windowsazure.com
│   │       └── privatelink.blob.core.windows.net
│   │
│   └── Step 2e: Review + Create → Create
│
├── Step 3: Repeat for additional private endpoints if needed
│   └── ⚠️ Minimum 2 private endpoints (vault + blob)
│
├── Step 4: Verify Connection
│   └── Networking → Private endpoint connections → Status = "Approved"
│
└── ⚠️ NSG rules must allow traffic to private endpoint
```

---

### 25.18 Configure Backup Policy

> **Portal:** `Recovery Services Vault → Manage → Backup policies`

```
Create / Modify Backup Policy
│
├── Step 1: Navigate
│   └── Recovery Services Vault → Manage → Backup policies → + Add
│
├── Step 2: Select Policy Type
│   ├── Azure Virtual Machine
│   ├── Azure File Share
│   ├── SQL Server in Azure VM
│   └── SAP HANA in Azure VM
│
├── Step 3: Configure Policy (Example: Azure VM)
│   │
│   ├── Policy Name
│   ├── Policy sub type: Standard / Enhanced
│   │
│   ├── Backup Schedule
│   │   ├── Frequency: Daily / Hourly (Enhanced only)
│   │   ├── Time & Time Zone
│   │   └── ⚠️ Time zone must be set during creation
│   │
│   ├── Instant Restore
│   │   └── Retain snapshots: 1–5 days (Standard) / up to 30 days (Enhanced)
│   │
│   ├── Retention Range
│   │   ├── Daily: 7–9999 days
│   │   ├── Weekly: 1–5163 weeks (select day of week)
│   │   ├── Monthly: 1–1188 months (select day/week of month)
│   │   └── Yearly: 1–99 years (select day/week/month of year)
│   │
│   └── ⚠️ Default policy = Daily at scheduled time, 30-day retention
│
└── Step 4: Create → Policy ready to assign
```

---

### 25.19 Configure Backup Alerts & Monitoring

> **Portal:** `Recovery Services Vault → Monitoring`

```
Backup Alerts & Monitoring Setup
│
├── A. Diagnostic Settings (Log Analytics)
│   │   Portal: Recovery Services Vault → Monitoring → Diagnostic settings
│   │
│   ├── Step 1: Click "+ Add diagnostic setting"
│   ├── Step 2: Enter Diagnostic Setting Name
│   ├── Step 3: Select Log Categories
│   │   ├── CoreAzureBackup
│   │   ├── AddonAzureBackupJobs
│   │   ├── AddonAzureBackupAlerts
│   │   ├── AddonAzureBackupPolicy
│   │   ├── AddonAzureBackupStorage
│   │   └── AddonAzureBackupProtectedInstance
│   ├── Step 4: Destination
│   │   └── Send to Log Analytics workspace → Select workspace
│   └── Step 5: Save
│
├── B. Backup Reports
│   │   Portal: Recovery Services Vault → Monitoring → Backup Reports
│   │   OR: Backup Center → Backup Reports
│   │
│   ├── Step 1: Select Log Analytics Workspace
│   ├── Step 2: View Reports
│   │   ├── Summary, Backup Items, Usage, Jobs, Policies
│   │   └── ⚠️ Data may take up to 24 hours to populate
│   └── Step 3: Filter by vault, subscription, time range
│
└── C. Alert Rules (Custom)
    │   Portal: Recovery Services Vault → Monitoring → Alerts → + Alert rule
    │
    ├── Step 1: Select Signal (e.g., Backup job failure)
    ├── Step 2: Define Condition
    ├── Step 3: Create/Select Action Group
    │   └── Notification: Email / SMS / Webhook / ITSM
    └── Step 4: Create Alert Rule
```

---

### 25.20 Backup Center — Unified Management

> **Portal:** `Home → Search "Backup Center"`

```
Backup Center Navigation
│
├── Overview
│   └── Dashboard with backup health across all vaults
│
├── Manage
│   ├── Backup instances → View all protected items across vaults
│   ├── Backup policies → View/manage policies across vaults
│   └── Vaults → List all RSV & Backup Vaults
│
├── Monitor
│   ├── Backup jobs → Track all backup/restore jobs
│   ├── Backup alerts → View alerts across vaults
│   └── Backup reports → Cross-vault reporting (needs Log Analytics)
│
├── Configure
│   ├── + Backup → Start backup from here (redirects to vault)
│   └── + Policy → Create policy from here
│
├── Govern
│   ├── Azure Policies for backup → Audit/enforce backup compliance
│   └── Security → Overview of security posture
│
└── ⚠️ Backup Center = visibility layer, does NOT replace vaults
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
