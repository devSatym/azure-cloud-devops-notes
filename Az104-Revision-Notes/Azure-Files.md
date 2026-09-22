<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Files — AZ-104 Revision Notes

---

## 1. What is Azure Files?

- **Fully managed file shares** in the cloud accessible via **SMB** and **NFS** protocols
- Replace or supplement on-premises file servers
- Mount concurrently on **Windows, Linux, and macOS**
- Accessible from anywhere via **port 445** (SMB) or private endpoint
- Endpoint: `https://<storage-account>.file.core.windows.net/<share-name>`
- Stored within a **Storage Account** (GPv2 for Standard, FileStorage for Premium)

---

## 2. Key Components

| Component | Description |
|---|---|
| **Storage Account** | Parent container — GPv2 (Standard) or FileStorage (Premium) |
| **File Share** | SMB or NFS share with a quota (size limit) |
| **Directory** | Folder structure within the share (true hierarchy) |
| **File** | Individual file (max 4 TiB per file) |
| **Snapshot** | Read-only point-in-time copy of the entire share |
| **Azure File Sync** | Hybrid service to sync on-prem Windows servers with Azure Files |

---

## 3. Protocols: SMB vs NFS

| Feature | **SMB** | **NFS** |
|---|---|---|
| **Protocol versions** | SMB 2.1, 3.0, 3.1.1 | NFS 4.1 |
| **Account type** | Standard (GPv2) or Premium (FileStorage) | **Premium (FileStorage) ONLY** |
| **OS support** | Windows, Linux, macOS | **Linux only** |
| **Authentication** | AD DS, Azure AD DS, Azure AD Kerberos, Storage Key | **No authentication** (host-based, IP/VNet) |
| **Encryption in transit** | ✅ (SMB 3.0+ encryption) | ❌ Not supported |
| **Public access** | ✅ (port 445 open) | ❌ Requires **Private Endpoint** |
| **ACLs** | ✅ NTFS-style Windows ACLs | ✅ POSIX permissions (uid/gid) |
| **Soft delete** | ✅ Supported | ❌ Not supported |
| **Snapshots** | ✅ Supported | ✅ Supported |
| **Cross-region access** | ✅ | ❌ Same region only |

> ⚠️ **EXAM TIP:** **NFS** requires: (1) **Premium FileStorage** account, (2) **Private Endpoint** (no public access), (3) **Linux only**. If exam asks about NFS → these 3 constraints.

> ⚠️ **EXAM TIP:** **SMB 3.0+** provides encryption in transit. SMB 2.1 does NOT support encryption in transit — only available when connecting from same Azure region.

> ⚠️ **EXAM TIP:** **Port 445** must be open for SMB over the internet. Many ISPs/corporate networks **block port 445**. Workarounds: VPN, ExpressRoute, Azure File Sync, or Private Endpoint.

---

## 4. Storage Account Types for Azure Files

| Account Type | Performance | Protocol | Redundancy | Access Tiers |
|---|---|---|---|---|
| **GPv2 (Standard)** | Standard (HDD-backed) | SMB only | LRS, ZRS, GRS, RA-GRS, GZRS, RA-GZRS | Transaction optimized, Hot, Cool |
| **FileStorage (Premium)** | Premium (SSD-backed) | SMB and NFS | **LRS, ZRS only** | Premium (no tiering) |

> ⚠️ **EXAM TIP:** **Premium file shares** = LRS and ZRS only — **NO GRS/RA-GRS** options. If geo-redundancy needed → Standard (GPv2) only.

> ⚠️ **EXAM TIP:** You **CANNOT mix** Standard and Premium file shares in the same storage account. Standard = GPv2 account. Premium = FileStorage account (dedicated).

---

## 5. File Share Tiers (Standard Only)

| Tier | Storage Cost | Transaction Cost | Best For |
|---|---|---|---|
| **Transaction optimized** | Higher | Lowest | Heavy read/write, migration, default |
| **Hot** | Medium | Medium | General-purpose file sharing |
| **Cool** | Lowest | Highest | Archival, infrequent access |

- Premium shares do NOT have tiers — always high-performance SSD
- Tier can be changed **anytime** (no minimum retention, no early deletion fees)

#### Portal Path — Set/Change Share Tier
```
Storage Account → Data storage → File shares → Select share →
Change tier → Transaction optimized / Hot / Cool → OK
```

> ⚠️ **EXAM TIP:** Unlike blob tiers, file share tiers have **NO minimum retention** period and **NO early deletion fees**. You can switch tiers anytime without penalty.

> ⚠️ **EXAM TIP:** **Transaction optimized** is the default tier — best for heavy read/write and migration workloads.

---

## 6. File Share Quota & Sizing

### Standard File Shares

| Setting | Default | Max |
|---|---|---|
| Share size (without large file shares) | 5 TiB | **5 TiB** |
| Share size (with large file shares enabled) | 5 TiB | **100 TiB** |
| Max file size | — | **4 TiB** |
| Max IOPS | — | **10,000** |
| Throughput | — | Up to **300 MiB/s** |

### Premium File Shares

| Setting | Details |
|---|---|
| Share size | 100 GiB to **100 TiB** |
| Max file size | **4 TiB** |
| Baseline IOPS | **400 + 1 per GiB** provisioned |
| Max IOPS | **100,000** |
| Throughput | **100 MiB/s + 0.06 MiB/s per GiB** up to ~10 GiB/s |

### Large File Shares
- Enables Standard shares to go up to **100 TiB**
- Must be **explicitly enabled** on the storage account
- **Irreversible** — cannot downgrade back to 5 TiB shares
- Restricts redundancy to **LRS and ZRS only** (no GRS after enabling)

#### Portal Path — Enable Large File Shares
```
Storage Account → Settings → Configuration →
Large file shares: Enabled → Save
⚠️ IRREVERSIBLE — disables GRS/RA-GRS redundancy options
```

> ⚠️ **EXAM TIP:** Large file shares = Standard accounts up to **100 TiB**. Enabling is **IRREVERSIBLE** and **disables GRS** (only LRS/ZRS after). Premium is always 100 TiB by default.

> ⚠️ **EXAM TIP:** Max individual file size = **4 TiB** (both Standard and Premium).

---

## 7. Authentication & Identity-Based Access

### SMB Authentication Methods

| Method | On-Prem AD? | Azure AD? | Use Case |
|---|---|---|---|
| **On-premises AD DS** | ✅ | Synced via AD Connect | Hybrid (domain-joined VMs accessing shares) |
| **Azure AD DS** | ❌ | ✅ (managed domain) | Cloud-native (Azure AD DS joined VMs) |
| **Azure AD Kerberos** | ❌ | ✅ (hybrid identities) | Hybrid users accessing from Azure AD joined VMs |
| **Storage Account Key** | N/A | N/A | Full access (admin/scripting) |

### Two-Level Permission Model (Identity-Based)

| Level | Type | Managed Via |
|---|---|---|
| **Share-level** | RBAC roles (Azure) | IAM on the file share |
| **File/Directory-level** | NTFS ACLs (Windows) | Windows Explorer / icacls |

> ⚠️ **EXAM TIP:** Identity-based access uses a **two-tier model**: (1) **Share-level** permissions via Azure RBAC roles, then (2) **File/directory-level** permissions via Windows NTFS ACLs. BOTH must be configured.

> ⚠️ **EXAM TIP:** **Storage account key** bypasses all RBAC and NTFS permissions — it has full access. Use identity-based auth for security.

### Share-Level RBAC Roles

| Role | Permissions |
|---|---|
| **Storage File Data SMB Share Reader** | Read access to files/directories |
| **Storage File Data SMB Share Contributor** | Read/write/delete files and directories |
| **Storage File Data SMB Share Elevated Contributor** | Read/write/delete + **modify NTFS ACLs** |

> ⚠️ **EXAM TIP:** **Elevated Contributor** = same as Contributor but can also **modify NTFS ACLs**. Regular Contributor cannot change permissions at the file/directory level.

#### Portal Path — Share-Level RBAC
```
Storage Account → Data storage → File shares → Select share →
Access Control (IAM) → + Add role assignment →
Role: Storage File Data SMB Share Contributor →
Members: Select users/groups → Assign
```

---

## 8. Mounting File Shares

### Mount on Windows
```cmd
net use Z: \\<storage-account>.file.core.windows.net\<share-name> /user:Azure\<storage-account> <storage-account-key>
```
- OR use the **Connect** button in Portal (generates mount script)
- Persistent mount: Add `/persistent:yes`
- Requires: port 445 open

### Mount on Linux (SMB)
```bash
sudo mount -t cifs //<storage-account>.file.core.windows.net/<share-name> /mnt/share \
  -o vers=3.0,username=<storage-account>,password=<key>,dir_mode=0777,file_mode=0777,serverino
```

### Mount on Linux (NFS)
```bash
sudo mount -t nfs <storage-account>.file.core.windows.net:/<storage-account>/<share-name> /mnt/share \
  -o vers=4,minorversion=1,sec=sys
```

#### Portal Path — Get Mount Script
```
Storage Account → File shares → Select share → Connect →
Select OS (Windows/Linux/macOS) → Authentication method →
Copy mount script → Run on client machine
```

> ⚠️ **EXAM TIP:** The **Connect** button in the portal generates the correct mount command for the selected OS. It includes the storage account key.

> ⚠️ **EXAM TIP:** For Windows: check `Test-NetConnection -ComputerName <sa>.file.core.windows.net -Port 445` — if fails, port 445 is blocked.

---

## 9. Snapshots

- **Share-level** read-only snapshot of entire file share at a point in time
- Incremental — only changes from previous snapshot stored
- Max **200 snapshots** per share
- Retention up to **10 years**
- Can browse and restore individual files from a snapshot
- Windows: accessible via **Previous Versions** tab (right-click → Properties → Previous Versions)

#### Portal Path — Create Snapshot
```
Storage Account → File shares → Select share →
Snapshots → + Add snapshot → Comment (optional) → OK
```

#### Portal Path — Browse/Restore from Snapshot
```
Storage Account → File shares → Select share →
Snapshots → Select snapshot → Browse files →
Select file → Restore (overwrites current) / Download
```

> ⚠️ **EXAM TIP:** File share snapshots are **read-only** and **incremental**. Max **200 per share**. Snapshots must be deleted before deleting the file share.

> ⚠️ **EXAM TIP:** On Windows mapped drives, snapshots appear as **"Previous Versions"** — allows end-users to self-service restore files.

---

## 10. Soft Delete

- Retain deleted file shares for a configurable period (1–365 days)
- Recover entire deleted share within retention window
- Default: **Enabled, 7 days** (for new storage accounts)
- Applies to **SMB shares only** (NOT NFS)

#### Portal Path
```
Storage Account → Data protection →
Enable soft delete for file shares: ✅ →
Retention period: 7 days (1–365) → Save
```

> ⚠️ **EXAM TIP:** File share soft delete = **enabled by default (7 days)**. Protects against accidental share deletion. Does NOT protect individual file deletion within a share — use snapshots for that.

> ⚠️ **EXAM TIP:** Soft delete does NOT work for **NFS** shares. SMB only.

---

## 11. Azure File Sync

### What It Does
- Synchronizes on-premises **Windows file servers** with Azure File shares
- Centralizes file shares in Azure while keeping **local access performance**
- Enables **cloud tiering** — frequently accessed files cached locally, rest in cloud
- Supports **multi-site sync** — same share synced across multiple on-prem servers

### Architecture Components

| Component | Description |
|---|---|
| **Storage Sync Service** | Top-level Azure resource managing sync relationships |
| **Sync Group** | Defines the sync topology (which endpoints sync together) |
| **Cloud Endpoint** | Azure File share (1 per sync group) |
| **Server Endpoint** | Folder path on a registered Windows Server |
| **Registered Server** | Windows Server with Azure File Sync agent installed |
| **Azure File Sync Agent** | Software installed on Windows Server |

### Cloud Tiering

| Setting | Description |
|---|---|
| **Volume free space policy** | Maintain X% free space on local volume (e.g., 20%) |
| **Date policy** | Cache files accessed within last X days locally |
| **Tiered files** | Replaced with **stubs** (reparse points) — appear as normal files |
| **Recall** | Tiered file automatically downloaded from Azure when opened |

### Requirements

| Requirement | Detail |
|---|---|
| **Server OS** | Windows Server 2016, 2019, 2022 |
| **PowerShell** | 5.1 (Windows Server) or 7+ |
| **.NET Framework** | 4.7.2+ |
| **File system** | NTFS (ReFS not supported) |
| **Azure share size** | ≤ 100 TiB |
| **File size** | No max (entire file tree can sync) |
| **Max file size for tiering** | 100 GiB per file |
| **Max files per sync group** | 100 million files |
| **Max server endpoints per sync group** | 100 |
| **Max sync groups per Storage Sync Service** | 200 |
| **Max registered servers per SSS** | 99 |

### Sync Topology

| Topology | Description |
|---|---|
| **Hub and spoke** | Azure share is the hub; multiple servers sync to it |
| **Multi-site** | Multiple server endpoints in one sync group → all stay in sync |
| **Cloud tiering + local cache** | Most common — full namespace visible, frequently accessed files local |

> ⚠️ **EXAM TIP:** Only **1 cloud endpoint** per sync group. Multiple server endpoints can sync to the same cloud endpoint. Azure File share = single source of truth.

> ⚠️ **EXAM TIP:** **Cloud tiering** creates **stubs** (reparse points) on the server. Files look normal but are actually pointers to Azure. Full file downloaded transparently when accessed.

> ⚠️ **EXAM TIP:** Azure File Sync requires **NTFS** — ReFS is NOT supported. Windows Server 2016+ only.

> ⚠️ **EXAM TIP:** Each server endpoint folder must be on an **NTFS volume** and must **NOT be nested** within another server endpoint.

---

## 12. Azure File Sync vs Standard Backup Comparison

| Feature | Azure File Sync | Azure Backup (File Shares) |
|---|---|---|
| **Purpose** | Sync + cache on-prem ↔ cloud | Backup + restore file shares |
| **Cloud tiering** | ✅ | ❌ |
| **Multi-site sync** | ✅ | ❌ |
| **Agent** | Required on Windows Server | No agent needed |
| **Recovery** | Recall from cloud / snapshots | Restore from backup vault |
| **RPO** | Near real-time sync | Per backup schedule |

---

## 13. Azure Backup for File Shares

- Snapshot-based backup for Azure File shares
- Managed by **Recovery Services vault** or **Backup vault**
- Supports: instant restore from snapshots, file-level restore
- Backup up to **4× per day** (every 4 hours minimum)
- Retention: Daily (up to 200 days), Weekly/Monthly/Yearly (up to 10 years)

#### Portal Path — Enable Backup
```
Storage Account → File shares → Select share →
Backup → Configure backup →
Recovery Services vault: Select/Create →
Backup policy: Select → Enable backup
```

> ⚠️ **EXAM TIP:** Azure Backup for file shares uses **snapshots** — max 200 snapshots per share. Backup counts toward the 200 snapshot limit.

---

## 14. Networking

### Access Options

| Method | Description |
|---|---|
| **Public endpoint** | `<sa>.file.core.windows.net` — accessible over internet (port 445) |
| **Service endpoint** | Optimized route from VNet — still uses public IP |
| **Private endpoint** | Private IP in your VNet — fully private access |
| **VPN / ExpressRoute** | Tunnel from on-prem to Azure VNet + private endpoint |

### Firewall Rules

#### Portal Path
```
Storage Account → Security + networking → Networking →
Public network access: Enabled from selected VNets and IPs →
VNets: Add existing VNet + subnet →
Firewall: Add client IP → Save
```

> ⚠️ **EXAM TIP:** **NFS shares** REQUIRE private endpoint — no public endpoint access. SMB shares can use public or private endpoints.

> ⚠️ **EXAM TIP:** When firewall rules are active, ensure "Allow trusted Microsoft services" is checked for Azure Backup and File Sync to work.

---

## 15. Security

### Encryption

| Feature | Description |
|---|---|
| **Encryption at rest** | AES-256, always enabled (cannot disable) |
| **Microsoft-managed keys** | Default |
| **Customer-managed keys** | Key Vault (CMK) — optional |
| **SMB encryption in transit** | ✅ (SMB 3.0+ — enabled by default via "Secure transfer required") |
| **NFS encryption in transit** | ❌ Not supported (use VPN for encrypted tunnel) |

### Secure Transfer

#### Portal Path
```
Storage Account → Settings → Configuration →
Secure transfer required: Enabled (default) → Save
```

> ⚠️ **EXAM TIP:** **Secure transfer required** = default ON. Enforces SMB 3.0 encryption + HTTPS for REST APIs. SMB 2.1 connections are **rejected** when this is enabled.

> ⚠️ **EXAM TIP:** NFS does NOT support encryption in transit natively. Secure transfer setting does NOT apply to NFS. Use VPN/ExpressRoute tunnel for NFS encryption.

### RBAC Roles

| Role | Permissions |
|---|---|
| **Storage Account Contributor** | Manage account (NOT data access) |
| **Storage File Data SMB Share Reader** | Read files/dirs via SMB |
| **Storage File Data SMB Share Contributor** | Read/write/delete via SMB |
| **Storage File Data SMB Share Elevated Contributor** | Read/write/delete + modify NTFS ACLs |
| **Storage File Data Privileged Contributor** | Read/write/delete + override ACLs (service principal) |
| **Storage File Data Privileged Reader** | Read + override ACLs (service principal) |

---

## 16. Monitoring & Diagnostics

### Key Metrics

| Metric | Description |
|---|---|
| **FileCapacity** | Total storage used by file shares |
| **FileCount** | Number of files |
| **FileShareCount** | Number of file shares |
| **Transactions** | Number of requests |
| **Ingress / Egress** | Data in / out |
| **SuccessE2ELatency** | End-to-end latency |
| **Availability** | Percentage of successful requests |

#### Portal Path
```
Storage Account → Monitoring → Metrics →
Metric namespace: File →
Metric: FileCapacity / Transactions / Egress → Apply
```

### Diagnostic Logs
```
Storage Account → Monitoring → Diagnostic settings →
+ Add diagnostic setting → file →
Logs: StorageRead / StorageWrite / StorageDelete →
Destination: Log Analytics → Save
```

---

## 17. Pricing Key Points

### Standard (GPv2)

| Component | Notes |
|---|---|
| **Data at rest** | Per GiB/month — varies by tier (Cool cheapest, Transaction optimized highest) |
| **Transactions** | Per 10,000 ops — varies by tier (Cool highest, Transaction optimized lowest) |
| **Data transfer** | Ingress free, Egress charged |
| **Snapshots** | Charged at Cool tier rate (incremental) |
| **Soft delete** | Retained data billed at share tier rate |
| **Redundancy** | LRS < ZRS < GRS < RA-GRS |

### Premium (FileStorage)

| Component | Notes |
|---|---|
| **Provisioned capacity** | Per GiB/month (pay for provisioned, not used) |
| **No transaction charges** | Included in provisioned cost |
| **Snapshots** | Charged incrementally |
| **Redundancy** | LRS or ZRS only |

> ⚠️ **EXAM TIP:** **Standard** = pay for what you USE. **Premium** = pay for what you PROVISION (even if unused). Premium includes transactions; Standard charges per-transaction.

> ⚠️ **EXAM TIP:** Standard file share tiers have **no early deletion fees** — unlike blob tiers. Switch tiers anytime without penalty.

---

## 18. Limitations & Constraints

| Constraint | Standard | Premium |
|---|---|---|
| Max share size | **100 TiB** (with large file shares) | **100 TiB** |
| Max file size | **4 TiB** | **4 TiB** |
| Max share IOPS | **10,000** | **100,000** |
| Max throughput | **300 MiB/s** | ~**10 GiB/s** |
| Max shares per account | **Unlimited** | **Unlimited** |
| Min share size | N/A | **100 GiB** provisioned |
| Snapshots per share | **200** | **200** |
| Max open handles per file | **2,000** | **2,000** |
| Max directory depth | **Unlimited** | **Unlimited** |
| Max file name length | **255 chars** | **255 chars** |
| Max path length | **2,048 chars** | **2,048 chars** |
| Max files per share | **~100 billion** | **~100 billion** |

### Azure File Sync Limits

| Constraint | Limit |
|---|---|
| Sync groups per Storage Sync Service | **200** |
| Registered servers per SSS | **99** |
| Server endpoints per sync group | **100** |
| Cloud endpoints per sync group | **1** |
| Files per sync group | **100 million** |
| File size for tiering | No limit (syncs any size) |

---

## 19. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| Create share | `az storage share create --account-name <sa> -n <share> --quota 100` |
| List shares | `az storage share list --account-name <sa> -o table` |
| Delete share | `az storage share delete --account-name <sa> -n <share>` |
| Show share | `az storage share show --account-name <sa> -n <share>` |
| Set quota | `az storage share update --account-name <sa> -n <share> --quota 200` |
| Upload file | `az storage file upload --account-name <sa> -s <share> --source file.txt` |
| Download file | `az storage file download --account-name <sa> -s <share> -p file.txt --dest ./file.txt` |
| List files | `az storage file list --account-name <sa> -s <share> -o table` |
| Create directory | `az storage directory create --account-name <sa> -s <share> -n mydir` |
| Delete file | `az storage file delete --account-name <sa> -s <share> -p file.txt` |
| Create snapshot | `az storage share snapshot --account-name <sa> -n <share>` |
| List snapshots | `az storage share list --account-name <sa> --include-snapshots` |
| Restore soft-deleted | `az storage share restore --account-name <sa> -n <share> --deleted-version <version>` |

### PowerShell

| Action | Command |
|---|---|
| Get context | `$ctx = New-AzStorageContext -StorageAccountName <sa> -StorageAccountKey <key>` |
| Create share | `New-AzStorageShare -Name <share> -Context $ctx` |
| Set quota | `Set-AzStorageShareQuota -ShareName <share> -Quota 100 -Context $ctx` |
| Upload file | `Set-AzStorageFileContent -ShareName <share> -Source file.txt -Path file.txt -Context $ctx` |
| Download file | `Get-AzStorageFileContent -ShareName <share> -Path file.txt -Destination ./file.txt -Context $ctx` |
| Create snapshot | `$share = Get-AzStorageShare -Name <share> -Context $ctx; $share.CloudFileShare.Snapshot()` |

---

## 20. Quick-Fire Exam Points ⚡

1. Azure Files = **managed file shares** via SMB (2.1/3.0/3.1.1) and NFS (4.1)
2. **SMB** = Windows/Linux/macOS. **NFS** = Linux only
3. **NFS** requires: (1) **Premium FileStorage** account, (2) **Private endpoint**, (3) Linux only
4. Standard shares = GPv2 account. Premium shares = **FileStorage** account (dedicated)
5. **Cannot mix** Standard and Premium shares in the same storage account
6. Standard tiers: **Transaction optimized** (default), Hot, Cool — no early deletion fees
7. Premium: **no tiers** — always SSD-backed, pay for **provisioned** capacity
8. Standard redundancy: LRS, ZRS, GRS, RA-GRS, GZRS, RA-GZRS. Premium: **LRS, ZRS only**
9. Max file size = **4 TiB** (Standard and Premium)
10. Standard max share size = **5 TiB** default → **100 TiB** with large file shares enabled
11. Enabling large file shares is **IRREVERSIBLE** and **disables GRS** (only LRS/ZRS)
12. Premium max share = **100 TiB** by default (no separate enablement needed)
13. Premium IOPS: baseline **400 + 1 per GiB** provisioned, max **100,000**
14. **Port 445** required for SMB over internet — often blocked by ISPs
15. **Connect** button in portal generates OS-specific mount command
16. Identity-based auth = **two levels**: (1) Share-level RBAC, (2) File/directory NTFS ACLs
17. Share-level roles: Reader / Contributor / **Elevated Contributor** (can modify ACLs)
18. **Storage account key** bypasses ALL permissions — full access
19. **Secure transfer required** = default ON → enforces SMB 3.0+ and HTTPS
20. SMB encryption in transit = ✅. NFS encryption in transit = ❌ (use VPN)
21. Snapshots: share-level, incremental, max **200 per share**, read-only
22. Windows **Previous Versions** tab = browse/restore from snapshots
23. Soft delete: **enabled by default (7 days)**, **SMB only** (not NFS), 1–365 day retention
24. Soft delete protects **shares** from deletion, NOT individual files — use snapshots for files
25. **Azure File Sync** = sync on-prem Windows servers with Azure Files
26. File Sync agent: Windows Server **2016+**, **NTFS only** (not ReFS)
27. **Cloud tiering** = stubs (reparse points) on server, full files in Azure, auto-recalled
28. File Sync: **1 cloud endpoint** per sync group, up to **100 server endpoints**
29. Max **100 million files** per sync group
30. Azure Backup for Files = snapshot-based, managed by Recovery Services vault
31. Backup: up to **4×/day**, max **200 snapshots** (shared with manual snapshots)
32. **Encryption at rest** = always on (AES-256), cannot disable
33. Standard = pay for used capacity. Premium = pay for **provisioned** capacity (even if unused)
34. Ingress = FREE. Egress = charged
35. Max open handles per file = **2,000**

---

## 21. Step-by-Step Configuration Mind Maps 🗺️

---

### 21.1 Create Azure File Share

> **Portal:** `Storage Account → Data storage → File shares → + File share`

```
Create Azure File Share
│
├── Prerequisites
│   ├── Storage account exists:
│   │   ├── SMB Standard: GPv2 account
│   │   ├── SMB Premium: FileStorage (Premium) account
│   │   └── NFS: FileStorage (Premium) account
│   │       ⚠️ NFS = Premium only + Linux only + Private endpoint only
│   └── RBAC: Storage Account Contributor or Contributor
│
├── Step 1: (If Standard > 5 TiB needed) Enable Large File Shares
│   │   Portal: Storage Account → Settings → Configuration
│   ├── Large file shares: Enabled → Save
│   └── ⚠️ IRREVERSIBLE — disables GRS/RA-GRS
│
├── Step 2: Create File Share
│   │   Portal: Storage Account → Data storage → File shares → + File share
│   ├── Name: lowercase letters, numbers, hyphens (3–63 chars)
│   ├── Provisioned capacity (Premium only): GiB (min 100)
│   │   ⚠️ Premium: billed for provisioned, not used
│   ├── Tier (Standard only):
│   │   ├── Transaction optimized (default) — heavy I/O
│   │   ├── Hot — general purpose
│   │   └── Cool — archival
│   │   ⚠️ No early deletion fees for tier changes
│   ├── Protocol:
│   │   ├── SMB (default)
│   │   └── NFS (Premium FileStorage only)
│   │       ⚠️ NFS + Secure transfer required = conflict
│   │       Secure transfer must be disabled for NFS shares
│   ├── Quota (Standard): Max GiB (1 GiB – 100 TiB)
│   └── Create
│
├── Step 3: (If NFS) Configure Private Endpoint
│   │   Portal: Storage Account → Networking → Private endpoint connections → + Private endpoint
│   ├── Target sub-resource: file
│   ├── VNet and Subnet
│   ├── DNS integration: ✅
│   └── Create
│       ⚠️ NFS shares REQUIRE private endpoint (no public access)
│
└── ⚠️ Notes
    ├── Large file shares = 100 TiB (Standard). Premium = 100 TiB default
    ├── Premium: IOPS = 400 + 1/GiB. More capacity = more IOPS
    ├── File share name = lowercase only (unlike Blob containers)
    └── Quota is a soft limit — prevents share from growing beyond it
```

---

### 21.2 Mount Azure File Share on Windows

> **Portal:** `File share → Connect → Windows`

```
Mount on Windows
│
├── Prerequisites
│   ├── Port 445 open (outbound)
│   │   Test: Test-NetConnection -ComputerName <sa>.file.core.windows.net -Port 445
│   │   ⚠️ Many ISPs block port 445 — use VPN or Private Endpoint
│   ├── Windows 10/11 or Windows Server 2016+ (for SMB 3.x)
│   │   ⚠️ SMB 2.1 only from Windows 8.1/Server 2012 (same region only)
│   └── Storage account key or AD credentials
│
├── Method 1: Portal (Connect Script)
│   │   Portal: Storage Account → File shares → Select share → Connect
│   ├── Drive letter: Select (e.g., Z:)
│   ├── Authentication:
│   │   ├── Storage account key (default)
│   │   └── Active Directory (if configured)
│   ├── → Script generated → Copy → Run in PowerShell
│   └── Script auto-stores credentials in Windows Credential Manager
│
├── Method 2: Manual net use
│   │   net use Z: \\<sa>.file.core.windows.net\<share> /user:Azure\<sa> <key> /persistent:yes
│   ├── /persistent:yes → reconnects after reboot
│   └── ⚠️ Credentials visible in command history
│
├── Method 3: PowerShell (cmdkey + New-PSDrive)
│   │   cmdkey /add:<sa>.file.core.windows.net /user:Azure\<sa> /pass:<key>
│   │   New-PSDrive -Name Z -PSProvider FileSystem -Root "\\<sa>.file.core.windows.net\<share>" -Persist
│   └── Survives reboot with -Persist
│
├── Verify
│   ├── dir Z:\ → list files
│   └── net use → shows mapped drive
│
└── ⚠️ Notes
    ├── Port 445 blocked = mount fails. Use File Sync or VPN as workaround
    ├── Secure transfer required = SMB 3.0+ only (no SMB 2.1)
    ├── Max 1 mount per storage account from same machine with key-based auth
    └── For AD auth: machine must be domain-joined
```

---

### 21.3 Mount Azure File Share on Linux

> **Portal:** `File share → Connect → Linux`

```
Mount on Linux
│
├── SMB Mount
│   │
│   ├── Prerequisites
│   │   ├── cifs-utils package installed: sudo apt install cifs-utils
│   │   ├── Port 445 open
│   │   └── Storage account key or credential file
│   │
│   ├── Step 1: Create credential file (security best practice)
│   │   sudo mkdir /etc/smbcredentials
│   │   sudo bash -c 'echo "username=<sa>" > /etc/smbcredentials/<sa>.cred'
│   │   sudo bash -c 'echo "password=<key>" >> /etc/smbcredentials/<sa>.cred'
│   │   sudo chmod 600 /etc/smbcredentials/<sa>.cred
│   │
│   ├── Step 2: Create mount point
│   │   sudo mkdir -p /mnt/<share>
│   │
│   ├── Step 3: Mount
│   │   sudo mount -t cifs //<sa>.file.core.windows.net/<share> /mnt/<share> \
│   │     -o credentials=/etc/smbcredentials/<sa>.cred,\
│   │     dir_mode=0777,file_mode=0777,serverino,nosharesock,actimeo=30
│   │
│   └── Step 4: Persistent mount (fstab)
│       sudo echo "//<sa>.file.core.windows.net/<share> /mnt/<share> cifs \
│         credentials=/etc/smbcredentials/<sa>.cred,dir_mode=0777,... 0 0" >> /etc/fstab
│
├── NFS Mount
│   │
│   ├── Prerequisites
│   │   ├── Premium FileStorage account
│   │   ├── NFS share created (protocol = NFS)
│   │   ├── Private endpoint configured
│   │   │   ⚠️ NFS has NO public access — private endpoint required
│   │   ├── nfs-common package: sudo apt install nfs-common
│   │   └── Client VM in same VNet (or peered/VPN)
│   │
│   ├── Mount
│   │   sudo mount -t nfs <sa>.file.core.windows.net:/<sa>/<share> /mnt/<share> \
│   │     -o vers=4,minorversion=1,sec=sys
│   │
│   └── Persistent (fstab)
│       <sa>.file.core.windows.net:/<sa>/<share> /mnt/<share> nfs \
│         vers=4,minorversion=1,sec=sys 0 0
│
└── ⚠️ Notes
    ├── NFS = Premium only, Private endpoint only, Linux only
    ├── NFS has NO authentication — access controlled by VNet/IP
    ├── NFS does NOT encrypt in transit — use VPN tunnel
    └── SMB 3.0+ recommended for encryption support
```

---

### 21.4 Configure Identity-Based Authentication (AD DS)

> **Portal:** `Storage Account → File shares → Active Directory`

```
Configure AD DS Authentication for SMB
│
├── Prerequisites
│   ├── On-premises Active Directory Domain Services (AD DS)
│   ├── Azure AD Connect syncing identities to Azure AD
│   ├── Client machines domain-joined to same AD
│   ├── Network connectivity to storage account (port 445)
│   └── RBAC: Storage Account Contributor
│
├── Step 1: Enable AD DS Authentication
│   │   Portal: Storage Account → Data storage → File shares →
│   │   Active Directory: Not configured → Set up
│   │   Select: On-premises Active Directory Domain Services
│   │
│   │   OR via PowerShell (recommended):
│   │   # Download AzFilesHybrid module
│   │   Install-Module -Name AzFilesHybrid
│   │   # Join storage account to AD domain
│   │   Join-AzStorageAccount -ResourceGroupName <rg> \
│   │     -StorageAccountName <sa> \
│   │     -DomainAccountType ComputerAccount \
│   │     -OrganizationalUnitDistinguishedName "OU=FileShares,DC=corp,DC=com"
│   │   ⚠️ Creates a computer account in AD for the storage account
│   │
│   ├── Verify: Storage Account → File shares → Active Directory
│   │   Status: Configured ✅
│
├── Step 2: Assign Share-Level RBAC
│   │   Portal: File share → Access Control (IAM) → + Add role assignment
│   ├── Storage File Data SMB Share Reader → for read-only users
│   ├── Storage File Data SMB Share Contributor → for read/write users
│   ├── Storage File Data SMB Share Elevated Contributor → for admins (modify ACLs)
│   ├── Assign to: AD-synced Azure AD users/groups
│   └── ⚠️ Both share-level RBAC and file-level NTFS ACLs must be configured
│
├── Step 3: Configure NTFS Permissions (File/Directory Level)
│   ├── Mount share using storage account key (as admin):
│   │   net use Z: \\<sa>.file.core.windows.net\<share> /user:Azure\<sa> <key>
│   ├── Set NTFS permissions:
│   │   icacls Z:\folder /grant "DOMAIN\Group:(OI)(CI)M"
│   │   OR right-click → Properties → Security → Edit → Add users/groups
│   └── ⚠️ Storage key mount = full access (for initial ACL setup)
│       After setup, users connect with AD credentials
│
├── Step 4: Mount with AD Credentials (End Users)
│   │   net use Z: \\<sa>.file.core.windows.net\<share>
│   │   (no key needed — Kerberos ticket used automatically)
│   └── ⚠️ Machine must be domain-joined + user must be in synced group
│
└── ⚠️ Notes
    ├── Two-level: Share-level RBAC + File-level NTFS ACLs
    ├── Storage key bypasses ALL permissions (admin only)
    ├── Elevated Contributor role = needed to SET initial NTFS ACLs
    └── AD DS = on-prem AD. Azure AD DS = managed domain. AD Kerberos = hybrid
```

---

### 21.5 Configure Azure File Sync

> **Portal:** `Home → Storage Sync Services → + Create`

```
Configure Azure File Sync
│
├── Prerequisites
│   ├── Azure File share exists (Standard GPv2 or Premium FileStorage)
│   ├── Windows Server 2016/2019/2022
│   ├── .NET Framework 4.7.2+ installed on server
│   ├── NTFS formatted volume (NOT ReFS)
│   │   ⚠️ ReFS is NOT supported
│   ├── PowerShell 5.1+
│   └── RBAC: Contributor on RG + Storage Account Contributor
│
├── Step 1: Create Storage Sync Service
│   │   Portal: Home → Storage Sync Services → + Create
│   ├── Subscription, Resource Group
│   ├── Name: e.g., "corp-sync-service"
│   ├── Region: Same region as storage account (recommended)
│   └── Create
│
├── Step 2: Install Azure File Sync Agent on Windows Server
│   ├── Download: Microsoft Download Center → "Azure File Sync Agent"
│   ├── Install StorageSyncAgent.msi
│   ├── Server Registration Wizard opens:
│   │   ├── Sign in to Azure
│   │   ├── Select Subscription → Resource Group → Storage Sync Service
│   │   └── Register
│   └── ⚠️ Server appears in: Storage Sync Service → Registered servers
│
├── Step 3: Create Sync Group
│   │   Portal: Storage Sync Service → Sync groups → + Sync group
│   ├── Sync group name: e.g., "file-sync-group"
│   ├── Cloud endpoint:
│   │   ├── Subscription → Storage Account → File Share
│   │   └── ⚠️ Only 1 cloud endpoint per sync group
│   └── Create
│
├── Step 4: Add Server Endpoint
│   │   Portal: Sync group → + Add server endpoint
│   ├── Registered server: Select server
│   ├── Path: e.g., D:\SharedFiles (local folder to sync)
│   │   ⚠️ Cannot be nested in another server endpoint
│   │   ⚠️ Must be on NTFS volume
│   ├── Cloud Tiering: Enabled / Disabled
│   │   ├── Volume Free Space Policy: e.g., 20% (keep 20% free on local volume)
│   │   ├── Date Policy: Cache files accessed in last X days
│   │   └── ⚠️ Tiered files = stubs. Opening triggers cloud recall
│   ├── Initial Download: Namespace only / Full download
│   │   ├── Namespace only: download file names/structure, tier content
│   │   └── Full download: download all file content
│   └── Add
│
├── Step 5: Verify Sync
│   ├── Portal: Sync group → server endpoint → Sync status
│   ├── Check: Upload activity / Download activity
│   ├── Event Viewer (server): Applications and Services Logs → Microsoft → FileSync
│   └── Test: Create file on server → appears in Azure File share (and vice versa)
│
└── ⚠️ Key Points
    ├── 1 cloud endpoint per sync group
    ├── Up to 100 server endpoints per sync group
    ├── Max 100 million files per sync group
    ├── Cloud tiering = exam favorite concept
    ├── Initial sync uploads ALL local data to Azure
    ├── Conflicts: last write wins (cloud = truth)
    └── Agent updates: auto-update policy available
```

---

### 21.6 Configure Azure Backup for File Shares

> **Portal:** `File share → Backup`

```
Configure Azure Backup for File Shares
│
├── Prerequisites
│   ├── Azure File share exists (SMB, Standard or Premium)
│   │   ⚠️ NFS shares: backup NOT supported
│   ├── Recovery Services vault (same region as storage account)
│   └── RBAC: Backup Contributor on vault + Storage Account Contributor
│
├── Step 1: Configure Backup
│   │   Portal: Storage Account → File shares → Select share →
│   │   Backup → Recovery Services vault: Select/Create new
│   │
│   │   OR: Recovery Services vault → + Backup →
│   │   Workload: Azure → What to back up: Azure FileShare →
│   │   Select storage account → Select shares → Configure backup
│   │
│   ├── Backup policy:
│   │   ├── Default policy or Create new
│   │   ├── Frequency: Daily (up to 4× per day)
│   │   ├── Retention:
│   │   │   ├── Daily: 1–200 days
│   │   │   ├── Weekly: 1–200 weeks
│   │   │   ├── Monthly: 1–120 months
│   │   │   └── Yearly: 1–10 years
│   │   └── ⚠️ Backup uses share snapshots — count toward 200 limit
│   └── Enable backup
│
├── Step 2: Run On-Demand Backup (optional)
│   │   Portal: Recovery Services vault → Backup items →
│   │   Azure Storage (Azure Files) → Select share → Backup now
│   └── Retain until: Select date
│
├── Step 3: Restore
│   │   Portal: Recovery Services vault → Backup items →
│   │   Select share → Restore
│   ├── Restore Type:
│   │   ├── Full Share Restore: → Original location / Alternate location
│   │   └── File Level Restore: → Select individual files/folders
│   ├── Restore Point: Select snapshot
│   ├── Conflict resolution: Overwrite / Skip
│   └── Restore
│
└── ⚠️ Notes
    ├── Backup = snapshot-based — instant restore
    ├── Snapshots count toward 200 max per share
    ├── NFS shares NOT supported for backup
    ├── Cannot delete storage account while backup is active
    │   → Stop backup → Delete backup data → then delete account
    └── Retention up to 10 years for compliance
```

---

### 21.7 Configure Snapshots & Previous Versions

> **Portal:** `Storage Account → File shares → Select share → Snapshots`

```
Configure Snapshots
│
├── Create Snapshot (Manual)
│   │   Portal: File share → Snapshots → + Add snapshot
│   ├── Comment: Optional description
│   └── OK
│       ⚠️ Max 200 snapshots per share (includes backup snapshots)
│
├── Browse Snapshot
│   │   Portal: File share → Snapshots → Select snapshot
│   ├── Browse directories and files
│   ├── Download individual files
│   └── Restore: Overwrites current file with snapshot version
│
├── Windows Previous Versions (End-User Self-Service)
│   │   Prerequisite: Share mounted as drive on Windows
│   ├── Right-click folder/file → Properties → Previous Versions tab
│   ├── Select snapshot version → Open / Restore / Copy
│   └── ⚠️ Each share snapshot = one "Previous Version" entry
│
├── Delete Snapshot
│   │   Portal: File share → Snapshots → Select → Delete
│   └── ⚠️ Must delete ALL snapshots before deleting the share
│
├── CLI
│   │   # Create
│   │   az storage share snapshot --account-name <sa> -n <share>
│   │   # List
│   │   az storage share list --account-name <sa> --include-snapshots -o table
│   │   # Delete
│   │   az storage share delete --account-name <sa> -n <share> --snapshot <datetime>
│
└── ⚠️ Notes
    ├── Snapshots = share-level (entire share, not individual files)
    ├── Incremental — only differences stored (cost-efficient)
    ├── Read-only — cannot modify snapshot content
    ├── Used by Azure Backup (count toward 200 limit)
    └── Previous Versions: great for user self-service restore
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
