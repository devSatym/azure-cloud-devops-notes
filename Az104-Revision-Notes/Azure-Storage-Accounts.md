<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Storage Accounts — AZ-104 Revision Notes

---

## 1. What is an Azure Storage Account?

- **Unified namespace** for Azure's core storage services (Blobs, Files, Queues, Tables)
- Provides a **unique endpoint** accessible via HTTP/HTTPS
- Globally unique name: `<name>.blob.core.windows.net`, `<name>.file.core.windows.net`, etc.
- Supports **redundancy**, **encryption**, **access tiers**, **lifecycle management**, **networking rules**
- Storage account name: **3–24 characters**, lowercase letters and numbers only, globally unique

> ⚠️ **EXAM TIP:** Storage account name = **3–24 chars**, **lowercase + numbers only**, **globally unique** across all of Azure. No hyphens, no uppercase.

---

## 2. Storage Services

| Service | Description | Endpoint | Protocol |
|---|---|---|---|
| **Blob Storage** | Unstructured data (images, videos, backups, logs) | `<name>.blob.core.windows.net` | REST/HTTP(S) |
| **Azure Files** | Managed SMB/NFS file shares | `<name>.file.core.windows.net` | SMB 3.x / NFS 4.1 |
| **Queue Storage** | Message queuing (up to 64 KB per message) | `<name>.queue.core.windows.net` | REST/HTTP(S) |
| **Table Storage** | NoSQL key-value store | `<name>.table.core.windows.net` | REST/HTTP(S) |
| **Azure Data Lake Storage Gen2** | Hierarchical namespace on Blob Storage for analytics | `<name>.dfs.core.windows.net` | REST/HTTP(S) |

---

## 3. Storage Account Types (Kinds)

| Kind | Services | Access Tiers | Redundancy | Use Case |
|---|---|---|---|---|
| **StorageV2 (General-purpose v2)** | Blob, File, Queue, Table, Data Lake | Hot, Cool, Cold, Archive | LRS, ZRS, GRS, RA-GRS, GZRS, RA-GZRS | ✅ Recommended for most scenarios |
| **BlobStorage** | Blob only | Hot, Cool, Archive | LRS, GRS, RA-GRS | Legacy — use StorageV2 |
| **BlockBlobStorage (Premium)** | Block Blobs + Append Blobs | Premium (no tiering) | LRS, ZRS | Low-latency, high-transaction workloads |
| **FileStorage (Premium)** | Files only | Premium (no tiering) | LRS, ZRS | High-perf file shares (SMB/NFS) |
| **StorageV1 (General-purpose v1)** | Blob, File, Queue, Table | No tiering | LRS, GRS, RA-GRS | Legacy — upgrade to V2 |

> ⚠️ **EXAM TIP:** **StorageV2 (GPv2)** is the recommended account type for ALL scenarios. Microsoft recommends upgrading V1 to V2. V1 → V2 upgrade is **free and non-disruptive**.

> ⚠️ **EXAM TIP:** **Premium** (BlockBlobStorage / FileStorage) does NOT support access tiers (Hot/Cool/Archive). Premium has **only LRS and ZRS** redundancy — no GRS.

> ⚠️ **EXAM TIP:** **NFS file shares** require **Premium FileStorage** account + **private endpoint** (no public access). SMB works with Standard and Premium.

---

## 4. Redundancy Options

| Option | Copies | Scope | Durability | Read from Secondary |
|---|---|---|---|---|
| **LRS** | 3 copies | Single datacenter | 11 nines | ❌ |
| **ZRS** | 3 copies | 3 Availability Zones (same region) | 12 nines | ❌ |
| **GRS** | 6 copies | Primary DC + Secondary region DC | 16 nines | ❌ |
| **RA-GRS** | 6 copies | Primary DC + Secondary region DC | 16 nines | ✅ Read-only |
| **GZRS** | 6 copies | 3 AZs (primary) + Secondary DC | 16 nines | ❌ |
| **RA-GZRS** | 6 copies | 3 AZs (primary) + Secondary DC | 16 nines | ✅ Read-only |

### Redundancy Diagram
```
LRS:     [ DC1: Copy1, Copy2, Copy3 ]
ZRS:     [ Zone1: Copy1 ] [ Zone2: Copy2 ] [ Zone3: Copy3 ]
GRS:     [ DC1: 3 copies ] ────→ [ Secondary DC: 3 copies ]
RA-GRS:  [ DC1: 3 copies ] ────→ [ Secondary DC: 3 copies (readable) ]
GZRS:    [ Zone1 ] [ Zone2 ] [ Zone3 ] ────→ [ Secondary DC: 3 copies ]
RA-GZRS: [ Zone1 ] [ Zone2 ] [ Zone3 ] ────→ [ Secondary DC: 3 copies (readable) ]
```

### Secondary Region Read Access
- **RA-GRS** and **RA-GZRS** = read-only access to secondary
- Secondary endpoint: `<name>-secondary.blob.core.windows.net`
- Data is **asynchronously replicated** — may have slight lag (RPO ~15 min)

#### Portal Path — Change Redundancy
```
Storage Account → Settings → Redundancy →
Select: LRS / ZRS / GRS / RA-GRS / GZRS / RA-GZRS → Save
```

> ⚠️ **EXAM TIP:** **LRS** = cheapest but lowest durability. **RA-GRS** = most common exam answer when "read access to secondary region" is needed. The "-secondary" endpoint is read-only.

> ⚠️ **EXAM TIP:** **GRS** replicates to secondary but you **CANNOT read** from it until failover. **RA-GRS** lets you read from secondary **without failover**.

> ⚠️ **EXAM TIP:** Failover to secondary makes it the new primary. After failover, account becomes **LRS** in the new primary region. Must reconfigure geo-redundancy.

> ⚠️ **EXAM TIP:** **Premium** storage accounts (BlockBlobStorage, FileStorage) only support **LRS** and **ZRS** — no GRS options.

---

## 5. Blob Storage

### 5.1 Blob Types

| Type | Description | Modifiable | Max Size |
|---|---|---|---|
| **Block Blob** | Default — for files, images, videos, text | ✅ (overwrite blocks) | **190.7 TiB** |
| **Append Blob** | Optimized for append operations | Append only | **195 GiB** |
| **Page Blob** | Random read/write — for VHDs | ✅ (random access) | **8 TiB** |

> ⚠️ **EXAM TIP:** **Block blobs** = most common (general-purpose storage). **Append blobs** = logging/audit trails (append-only). **Page blobs** = VM disk VHDs.

### 5.2 Access Tiers

| Tier | Storage Cost | Access Cost | Min Retention | Availability | Use Case |
|---|---|---|---|---|---|
| **Hot** | Highest | Lowest | None | 99.9% (RA-GRS: 99.99%) | Frequently accessed data |
| **Cool** | Lower | Higher | **30 days** | 99% (RA-GRS: 99.9%) | Infrequently accessed |
| **Cold** | Lower than Cool | Higher than Cool | **90 days** | 99% (RA-GRS: 99.9%) | Rarely accessed |
| **Archive** | Lowest | Highest | **180 days** | Offline | Long-term compliance |

### Tier Setting Levels

| Level | Setting | Options |
|---|---|---|
| **Account level** | Default access tier | Hot or Cool only |
| **Blob level** | Individual blob tier | Hot, Cool, Cold, or Archive |

#### Portal Path — Set Default Tier
```
Storage Account → Settings → Configuration →
Default access tier: Hot / Cool → Save
```

#### Portal Path — Set Blob Tier
```
Storage Account → Containers → Select container →
Select blob → Change tier → Hot/Cool/Cold/Archive → Save
```

> ⚠️ **EXAM TIP:** **Account default** can only be Hot or Cool. **Blob-level** can be Hot, Cool, Cold, or Archive. Cold and Archive are blob-level only.

> ⚠️ **EXAM TIP:** **Archive tier** = blob is **offline**. Must **rehydrate** to Hot or Cool before accessing. Rehydration takes up to **15 hours** (Standard) or **1 hour** (High Priority).

> ⚠️ **EXAM TIP:** **Early deletion fee** if a blob is deleted/moved before minimum retention: Cool = 30 days, Cold = 90 days, Archive = 180 days.

### 5.3 Rehydration from Archive

| Priority | Time | Cost |
|---|---|---|
| **Standard** | Up to **15 hours** | Lower |
| **High** | Under **1 hour** (for blobs < 10 GB) | Higher |

- Can rehydrate by: **changing tier** (in-place) or **copying** to a different blob in Hot/Cool

### 5.4 Lifecycle Management

- **Automate** tier transitions and deletion based on age/conditions
- Rules apply to: Block Blobs and Append Blobs
- Actions: Move to Cool → Cold → Archive → Delete
- Based on: **last modified date**, **created date**, **last accessed date** (with access tracking)

| Action | Description |
|---|---|
| **tierToCool** | Move from Hot to Cool |
| **tierToCold** | Move to Cold tier |
| **tierToArchive** | Move to Archive |
| **delete** | Delete the blob |
| **enableAutoTierToHotFromCool** | Move back to Hot when accessed |

#### Portal Path
```
Storage Account → Data management → Lifecycle management →
+ Add rule → Name → Conditions (days since modified) →
Actions (Move to Cool / Archive / Delete) → Add
```

> ⚠️ **EXAM TIP:** Lifecycle management is the exam-favorite for cost optimization. "Automatically move infrequently accessed data to Cool/Archive" → Lifecycle Management rule.

### 5.5 Blob Versioning and Soft Delete

| Feature | Description | Default |
|---|---|---|
| **Soft Delete (Blobs)** | Recover deleted blobs within retention period | Enabled, **7 days** default |
| **Soft Delete (Containers)** | Recover deleted containers | Disabled by default |
| **Blob Versioning** | Maintain previous versions of blobs automatically | Disabled by default |
| **Point-in-Time Restore** | Restore block blobs to a previous state | Requires: Versioning + Change Feed + Soft Delete |

#### Portal Path — Soft Delete
```
Storage Account → Data protection →
Enable soft delete for blobs: ✅ → Retention: 7–365 days → Save
Enable soft delete for containers: ✅ → Retention: 1–365 days → Save
```

> ⚠️ **EXAM TIP:** Blob soft delete default = **7 days**. Container soft delete is **disabled** by default. Both are independent settings.

### 5.6 Blob Snapshots and Versions
- **Snapshot**: Read-only copy of a blob at a point in time (manual)
- **Versioning**: Automatic — maintains all previous versions when blob is modified/deleted
- Snapshots and versions consume storage (additional cost)

---

## 6. Azure Files

- **Managed file shares** accessible via **SMB 3.x** and **NFS 4.1**
- Mount on Windows, Linux, macOS
- Replace on-prem file servers
- Supports: **Azure File Sync** for hybrid scenarios

### SMB vs NFS

| Feature | SMB | NFS |
|---|---|---|
| Protocol | SMB 2.1, 3.0, 3.1.1 | NFS 4.1 |
| Account type | Standard (GPv2) or Premium | **Premium only** |
| Auth | AD DS, Azure AD DS, Storage Key | No auth (host-based) |
| Access | Windows, Linux, macOS | Linux only |
| Public access | ✅ (port 445) | ❌ (Private endpoint required) |
| Encryption in transit | ✅ (SMB 3.0+) | ❌ |

### File Share Tiers (Standard)

| Tier | Use Case |
|---|---|
| **Transaction optimized** | Heavy read/write (default for GPv2) |
| **Hot** | General-purpose sharing |
| **Cool** | Archival, infrequently accessed |

> ⚠️ **EXAM TIP:** **NFS** file shares require **Premium** FileStorage account and **Private Endpoint** (no public access). SMB shares can use Standard or Premium.

> ⚠️ **EXAM TIP:** Mounting Azure Files over internet requires **port 445** open. Many ISPs block port 445 — use VPN or Azure File Sync as workaround.

#### Portal Path — Create File Share
```
Storage Account → Data storage → File shares → + File share →
Name, Tier (Transaction optimized / Hot / Cool), Quota (GiB) → Create
```

---

## 7. Azure File Sync

- Synchronizes on-prem Windows file servers with Azure File shares
- Enables **cloud tiering** — frequently accessed files local, rest in cloud
- Multi-site sync — sync the same share across multiple on-prem servers
- Requires: **Storage Sync Service** resource + **Azure File Sync agent** on Windows Server

### Components

| Component | Description |
|---|---|
| **Storage Sync Service** | Azure resource that manages sync relationships |
| **Sync Group** | Defines which endpoints sync together |
| **Cloud Endpoint** | Azure File share |
| **Server Endpoint** | Folder path on a registered Windows Server |
| **Registered Server** | Windows Server with File Sync agent installed |

### Cloud Tiering
- Infrequently accessed files replaced with **stubs** (placeholders) on local server
- Full file recalled from cloud when accessed
- Saves local disk space while providing full namespace visibility

> ⚠️ **EXAM TIP:** **Cloud tiering** = files accessed locally but stored in Azure Files. Stubs (placeholders) on local server. Full file downloaded on demand. Saves on-prem storage.

---

## 8. Queue Storage

- **Message queuing** service for asynchronous communication
- Max message size: **64 KB**
- Max queue size: **Unlimited** (total per storage account)
- Messages have TTL (Time to Live): default **7 days**, max **unlimited** (`-1`)
- Supports at-least-once delivery

> ⚠️ **EXAM TIP:** Queue message max = **64 KB**. For larger messages, store content in Blob Storage and put the blob reference in the queue message.

---

## 9. Table Storage

- **NoSQL key-value** store for semi-structured data
- Each entity: **PartitionKey** + **RowKey** (composite primary key)
- Max entity size: **1 MB**
- Alternative: **Azure Cosmos DB Table API** (premium, globally distributed)

> ⚠️ **EXAM TIP:** Table entity max size = **1 MB**. PartitionKey + RowKey = unique identifier. For global distribution → migrate to Cosmos DB Table API.

---

## 10. Security

### 10.1 Authorization Methods

| Method | Description | Use Case |
|---|---|---|
| **Storage Account Keys** | Full access (2 keys per account) | Admin / backend (rotate regularly) |
| **Shared Access Signatures (SAS)** | Delegated access with constraints | Time-limited, scoped access |
| **Azure AD (Entra ID)** | RBAC-based identity access | Recommended for Blob and Queue |
| **Anonymous/Public** | Public read for blobs | Public assets (CDN, websites) |

### 10.2 Storage Account Keys
- **2 keys** per account (Key1 and Key2) for rotation
- Each key grants **complete access** to the storage account
- Rotate regularly — use Key2 while regenerating Key1, then switch

#### Portal Path
```
Storage Account → Security + networking → Access keys →
View keys → Rotate key → Confirm
```

> ⚠️ **EXAM TIP:** Storage account keys grant **full access** — treat like root passwords. Prefer Azure AD RBAC or SAS for fine-grained access.

### 10.3 Shared Access Signatures (SAS)

| SAS Type | Scope | Description |
|---|---|---|
| **Account SAS** | Entire account | Access to multiple services (Blob, File, Queue, Table) |
| **Service SAS** | Single service | Scoped to one service (e.g., just Blob) |
| **User Delegation SAS** | Blob only | Signed with Azure AD credentials (most secure) |

### SAS Parameters

| Parameter | Description |
|---|---|
| **sv** | Signed version (API version) |
| **ss** | Signed services (b=Blob, f=File, q=Queue, t=Table) |
| **srt** | Signed resource types (s=Service, c=Container, o=Object) |
| **sp** | Signed permissions (r=Read, w=Write, d=Delete, l=List) |
| **se** | Signed expiry (UTC datetime) |
| **st** | Signed start (UTC datetime) |
| **sip** | Signed IP (restrict to IP range) |
| **spr** | Signed protocol (https or https,http) |
| **sig** | Signature (HMAC-SHA256) |

#### Portal Path — Generate SAS
```
Storage Account → Security + networking → Shared access signature →
Allowed services, resource types, permissions →
Start/Expiry date → Allowed IP → Protocol: HTTPS only →
Generate SAS and connection string
```

> ⚠️ **EXAM TIP:** **User Delegation SAS** = most secure (signed with Azure AD, not account key). Only works with **Blob** service. Recommended over Account/Service SAS.

> ⚠️ **EXAM TIP:** SAS tokens **cannot be revoked** individually. To invalidate: **regenerate the storage account key** that signed it, or use a **Stored Access Policy** to revoke.

> ⚠️ **EXAM TIP:** **Stored Access Policies** allow you to modify/revoke SAS tokens after creation. Up to **5** stored policies per container/queue/table/share.

### 10.4 Stored Access Policies
- Defined on a container, queue, table, or file share (not on account)
- SAS token references the policy → modifying the policy changes SAS behavior
- Can set: start time, expiry, permissions
- Max **5 policies** per resource

### 10.5 Encryption

| Feature | Default | Description |
|---|---|---|
| **Encryption at rest (SSE)** | ✅ Always on | AES-256, automatic for all data |
| **Microsoft-managed keys (MMK)** | ✅ Default | Azure manages encryption keys |
| **Customer-managed keys (CMK)** | Optional | Keys stored in Azure Key Vault |
| **Encryption in transit** | ✅ Default (HTTPS) | TLS 1.2 enforced by default |
| **Infrastructure encryption** | Optional | Double encryption (service + infra layer) |

> ⚠️ **EXAM TIP:** **Encryption at rest** is ALWAYS enabled — cannot be disabled. Default = Microsoft-managed keys. You can switch to **Customer-managed keys** (Key Vault required).

> ⚠️ **EXAM TIP:** **Infrastructure encryption** = double encryption. Must be enabled at **account creation time** — cannot be added later.

### 10.6 Network Security

| Feature | Description |
|---|---|
| **Firewalls & VNets** | Restrict access to specific VNets/IPs |
| **Private Endpoints** | Private IP in VNet for storage access |
| **Service Endpoints** | Optimized route from VNet to storage |
| **Secure transfer** | Require HTTPS (default: enabled) |
| **Minimum TLS version** | TLS 1.2 (default) |
| **Public access level** | Disabled / Blob / Container |

#### Portal Path — Networking
```
Storage Account → Security + networking → Networking →
Public network access: Enabled from all / Enabled from selected VNets and IPs / Disabled →
Firewall: Add VNet/IP → Save
```

> ⚠️ **EXAM TIP:** Default: **public access enabled from all networks**. For production: restrict to specific VNets/IPs or use private endpoints. "Storage accessible only from VNet" → Private Endpoint or Service Endpoint + firewall rules.

> ⚠️ **EXAM TIP:** **Secure transfer required** is enabled by default. Enforces HTTPS for REST API calls and SMB 3.0+ encryption for file shares.

---

## 11. Blob Public Access

| Level | Access |
|---|---|
| **Disabled** (account-level) | No anonymous access to any container/blob |
| **Private** (container-level) | No anonymous access (default for containers) |
| **Blob** (container-level) | Anonymous read for blobs only (must know URL) |
| **Container** (container-level) | Anonymous read + list blobs in container |

#### Portal Path — Account-Level Public Access
```
Storage Account → Settings → Configuration →
Allow Blob public access: Enabled / Disabled → Save
```

#### Portal Path — Container-Level Access
```
Storage Account → Containers → Select → Change access level →
Private / Blob / Container → OK
```

> ⚠️ **EXAM TIP:** Public access must be enabled at **BOTH** the account level AND the container level. If account-level is disabled, container-level settings are ignored.

> ⚠️ **EXAM TIP:** Default for new containers = **Private** (no anonymous access). You must explicitly set Blob or Container access level.

---

## 12. Immutable Storage (WORM)

- **Write Once, Read Many** — prevents modification and deletion
- For compliance: SEC, FINRA, HIPAA

| Policy | Description |
|---|---|
| **Time-based retention** | Data cannot be modified/deleted for a set period |
| **Legal hold** | Data locked until all legal holds removed (no expiry) |

| State | Can Modify | Can Delete |
|---|---|---|
| **Unlocked** | ✅ (can test) | ✅ |
| **Locked** | ❌ | ❌ (irreversible) |

#### Portal Path
```
Storage Account → Containers → Select → Access policy →
Add policy → Time-based retention / Legal hold → Configure
```

> ⚠️ **EXAM TIP:** Once a time-based retention policy is **locked**, it CANNOT be unlocked or shortened — only **extended**. This is **irreversible**.

---

## 13. AzCopy, Storage Explorer & Data Transfer

### AzCopy
- **Command-line tool** for copying data to/from Azure Storage
- High-performance, parallel transfer
- Supports: Blob, Files
- Authenticate: SAS tokens or Azure AD (`azcopy login`)

| Command | Description |
|---|---|
| `azcopy copy <source> <dest>` | Copy files |
| `azcopy sync <source> <dest>` | Sync (mirror) |
| `azcopy make <container-url>` | Create container |
| `azcopy remove <url>` | Delete blobs |
| `azcopy login` | Authenticate with Azure AD |

### Storage Explorer
- **GUI application** — manage storage across subscriptions
- Supports: Blobs, Files, Queues, Tables, Data Lake, Cosmos DB
- Works with: SAS, Azure AD, storage account keys

### Import/Export Service
- **Physical disk shipping** for large data transfers
- Ship HDDs to Azure datacenter
- **Azure Data Box** = Microsoft ships you a device for offline transfer

> ⚠️ **EXAM TIP:** **AzCopy** = command-line (scripted, CI/CD). **Storage Explorer** = GUI (interactive). **Import/Export** = physical disks for very large datasets (offline transfer).

---

## 14. Static Website Hosting

- Host **static HTML/CSS/JS** directly from Blob Storage
- Endpoint: `<name>.z<region>.web.core.windows.net`
- Requires: **GPv2** or **BlockBlobStorage** account
- Index document (e.g., `index.html`) + Error document (e.g., `404.html`)
- Content served from `$web` container

#### Portal Path
```
Storage Account → Data management → Static website →
Enable: Enabled → Index document: index.html →
Error document: 404.html → Save
```

> ⚠️ **EXAM TIP:** Static website content is stored in the **$web** container (auto-created). Custom domain + SSL requires **Azure CDN** in front.

---

## 15. Object Replication

- **Asynchronous** copy of block blobs between storage accounts
- Source → Destination (cross-region, cross-subscription)
- Requires: **Blob versioning** enabled on both source and destination
- Requires: **Change feed** enabled on source

#### Portal Path
```
Storage Account → Data management → Object replication →
+ Set up replication rules → Source/Destination → Containers → Create
```

> ⚠️ **EXAM TIP:** Object replication requires: **versioning enabled on BOTH** accounts + **change feed on source**. Does not replicate snapshots or blobs in Archive tier.

---

## 16. Security & RBAC

### Key RBAC Roles

| Role | Permissions |
|---|---|
| **Storage Account Contributor** | Manage storage accounts (NOT data access) |
| **Storage Blob Data Owner** | Full blob access + manage ACLs |
| **Storage Blob Data Contributor** | Read/write/delete blobs |
| **Storage Blob Data Reader** | Read-only blob access |
| **Storage File Data SMB Share Contributor** | Read/write/delete files via SMB |
| **Storage File Data SMB Share Reader** | Read-only file access via SMB |
| **Storage File Data SMB Share Elevated Contributor** | Read/write/delete + modify ACLs |
| **Storage Queue Data Contributor** | Read/write/delete queue messages |
| **Storage Queue Data Reader** | Read queue messages |
| **Storage Table Data Contributor** | Read/write/delete table entities |
| **Storage Table Data Reader** | Read table entities |
| **Reader and Data Access** | Read access + list storage keys |

> ⚠️ **EXAM TIP:** **Storage Account Contributor** can manage the account (create, delete, configure) but **CANNOT read/write data**. For data access, use **Storage Blob Data Contributor/Reader** etc.

> ⚠️ **EXAM TIP:** Azure AD RBAC for data access works with **Blob** and **Queue** only (and File SMB). **Table** data RBAC is now supported. Key-based auth works for all.

---

## 17. Monitoring & Diagnostics

### Metrics

| Metric | Description |
|---|---|
| **UsedCapacity** | Total storage consumed |
| **Transactions** | Number of requests |
| **Ingress / Egress** | Data in / out |
| **SuccessE2ELatency** | End-to-end latency |
| **Availability** | Percentage of successful requests |

#### Portal Path
```
Storage Account → Monitoring → Metrics →
Metric: Used Capacity / Transactions / Ingress / Egress → Apply
```

### Diagnostic Settings
```
Storage Account → Monitoring → Diagnostic settings →
+ Add diagnostic setting → Select logs/metrics →
Destination: Log Analytics / Storage Account / Event Hub → Save
```

---

## 18. Pricing Key Points

| Factor | Impact |
|---|---|
| **Redundancy** | LRS < ZRS < GRS < RA-GRS < GZRS < RA-GZRS |
| **Access tier** | Hot: high storage cost, low access. Archive: lowest storage, highest access |
| **Capacity** | Per GB/month stored |
| **Operations** | Per 10,000 operations (reads, writes, lists) |
| **Data transfer** | Egress charged, ingress free |
| **Data retrieval** | Cool/Cold/Archive retrieval charges |
| **Replication** | Geo-replication data transfer charges |
| **Early deletion** | Penalty if blob deleted before min retention |
| **Rehydration** | Archive → Hot/Cool incurs rehydration cost |

> ⚠️ **EXAM TIP:** **Ingress** is always **FREE**. **Egress** is charged per GB. Archive tier has the lowest storage cost but highest read/access cost.

---

## 19. Limitations & Constraints

| Constraint | Limit |
|---|---|
| Storage accounts per subscription per region | **250** |
| Max storage account capacity | **5 PiB** |
| Max blob size (Block Blob) | **190.7 TiB** |
| Max blob size (Page Blob) | **8 TiB** |
| Max file share size (Standard) | **100 TiB** (with large file shares enabled) |
| Max file share size (Premium) | **100 TiB** |
| Max file size | **4 TiB** |
| Max queue message | **64 KB** |
| Max table entity | **1 MB** |
| Stored access policies per resource | **5** |
| Max IOPS per account | **20,000** (for standard) |
| Max ingress (US regions, GPv2) | **10 Gbps** (with large file shares) |
| Storage account name | **3–24 chars**, lowercase + numbers |
| Account key count | **2** |
| Containers per account | **Unlimited** |
| Blobs per container | **Unlimited** |

---

## 20. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| Create account | `az storage account create -g <rg> -n <name> -l eastus --sku Standard_LRS --kind StorageV2` |
| List accounts | `az storage account list -g <rg> -o table` |
| Show account | `az storage account show -g <rg> -n <name>` |
| Show keys | `az storage account keys list -g <rg> -n <name>` |
| Regenerate key | `az storage account keys renew -g <rg> -n <name> --key key1` |
| Create container | `az storage container create --account-name <name> -n <container> --public-access off` |
| Upload blob | `az storage blob upload --account-name <name> -c <container> -n blobname -f file.txt` |
| Download blob | `az storage blob download --account-name <name> -c <container> -n blobname -f output.txt` |
| List blobs | `az storage blob list --account-name <name> -c <container> -o table` |
| Set blob tier | `az storage blob set-tier --account-name <name> -c <container> -n blobname --tier Cool` |
| Create file share | `az storage share create --account-name <name> -n <share> --quota 100` |
| Generate SAS | `az storage account generate-sas --account-name <name> --permissions rwdl --services bfqt --resource-types sco --expiry 2026-12-31` |
| Delete account | `az storage account delete -g <rg> -n <name>` |

### PowerShell

| Action | Command |
|---|---|
| Create account | `New-AzStorageAccount -ResourceGroupName <rg> -Name <name> -Location eastus -SkuName Standard_LRS -Kind StorageV2` |
| Get key | `Get-AzStorageAccountKey -ResourceGroupName <rg> -Name <name>` |
| Create context | `$ctx = New-AzStorageContext -StorageAccountName <name> -StorageAccountKey <key>` |
| Create container | `New-AzStorageContainer -Name <container> -Context $ctx -Permission Off` |
| Upload blob | `Set-AzStorageBlobContent -Container <container> -File file.txt -Blob blobname -Context $ctx` |
| Copy blob (AzCopy) | `azcopy copy "source" "dest?SAS"` |

---

## 21. Quick-Fire Exam Points ⚡

1. Storage account name: **3–24 chars**, lowercase letters + numbers only, **globally unique**
2. **StorageV2 (GPv2)** = recommended for all scenarios. V1 → V2 upgrade is **free**
3. **LRS** = 3 copies in 1 DC. **ZRS** = 3 copies across 3 AZs. **GRS** = 6 copies (2 regions)
4. **RA-GRS** = read-only secondary at `<name>-secondary.blob.core.windows.net`
5. **GRS** secondary is NOT readable until failover. **RA-GRS** secondary is always readable
6. After failover, account becomes **LRS** in new primary — must reconfigure geo-replication
7. **Premium** accounts support only **LRS and ZRS** — no GRS
8. Blob types: **Block** (general), **Append** (logging), **Page** (VHDs)
9. Access tiers: **Hot** (frequent), **Cool** (30-day min), **Cold** (90-day min), **Archive** (180-day min, offline)
10. Account default tier = **Hot or Cool** only. Blob-level = Hot/Cool/Cold/Archive
11. **Archive** blob is **offline** — must rehydrate (Standard: 15h, High Priority: <1h)
12. **Lifecycle management** automates tier transitions based on blob age
13. Soft delete default retention: **7 days** (blobs). Container soft delete: **disabled** by default
14. **Encryption at rest** (SSE) is ALWAYS on — AES-256, cannot be disabled
15. **Infrastructure encryption** = double encryption, set at **creation time only**
16. **Secure transfer required** = default enabled (enforces HTTPS)
17. Default: public access enabled from all networks. Recommend: restrict to VNets/IPs
18. **Public blob access** must be enabled at BOTH account level AND container level
19. Default container access = **Private** (no anonymous access)
20. **2 account keys** — rotate regularly. Regenerating key invalidates SAS signed with that key
21. SAS types: **Account**, **Service**, **User Delegation** (most secure, Blob only, Azure AD)
22. SAS cannot be revoked — regenerate key or use **Stored Access Policy** (max **5** per resource)
23. **User Delegation SAS** = signed with Azure AD credentials, most secure
24. **Storage Account Contributor** = manage account but NOT data access
25. **Storage Blob Data Contributor** = read/write blob data
26. Azure AD RBAC data access: Blob, Queue (+ File SMB, Table)
27. Azure Files: SMB = Standard/Premium. **NFS = Premium only + Private Endpoint**
28. Port **445** required for SMB over internet (often blocked by ISPs)
29. Azure File Sync: cloud tiering = stubs on-prem, full files in cloud
30. Object replication: requires **versioning** on both + **change feed** on source
31. Immutable (WORM): locked retention policy = **irreversible** (cannot shorten, only extend)
32. Static website hosting: content in **$web** container. Custom domain needs Azure CDN
33. Queue message max = **64 KB**. Table entity max = **1 MB**
34. Max storage accounts per subscription per region = **250**
35. Max account capacity = **5 PiB**
36. **Ingress = FREE**. Egress = charged. Archive retrieval = high cost
37. AzCopy = CLI for fast parallel copy. Storage Explorer = GUI. Import/Export = physical disks
38. **AzCopy login** = authenticate with Azure AD (no account key needed)
39. Max file share size = **100 TiB** (with large file shares enabled)
40. Failover RPO for GRS = approximately **15 minutes** (asynchronous replication)

---

## 22. Step-by-Step Configuration Mind Maps 🗺️

---

### 22.1 Create Storage Account

> **Portal:** `Home → Storage accounts → + Create`

```
Create Storage Account
│
├── Basics
│   ├── Subscription, Resource Group
│   ├── Storage account name
│   │   ⚠️ 3–24 chars, lowercase + numbers only, globally unique
│   ├── Region
│   ├── Performance:
│   │   ├── Standard: GPv2 (Blob, File, Queue, Table)
│   │   └── Premium: Choose below
│   │       ├── Block blobs: Low-latency blobs
│   │       ├── File shares: High-perf file shares (SMB/NFS)
│   │       └── Page blobs: VHD storage
│   │       ⚠️ Premium = LRS or ZRS only (no GRS)
│   ├── Redundancy: LRS / ZRS / GRS / RA-GRS / GZRS / RA-GZRS
│   │   ⚠️ RA-GRS/RA-GZRS = read from secondary
│   │   ⚠️ LRS = cheapest. RA-GZRS = most durable
│
├── Advanced
│   ├── Require secure transfer: ✅ (default, enforces HTTPS)
│   ├── Allow enabling anonymous access on containers: Enable/Disable
│   │   ⚠️ Must also set container-level access
│   ├── Enable storage account key access: ✅ (default)
│   ├── Default access tier: Hot / Cool
│   │   ⚠️ Account level = Hot or Cool ONLY
│   ├── Enable infrastructure encryption: ✅ (optional, double encryption)
│   │   ⚠️ CANNOT be changed after creation
│   ├── Enable hierarchical namespace: For Data Lake Storage Gen2
│   │   ⚠️ CANNOT be changed after creation
│   └── Minimum TLS version: 1.2 (default)
│
├── Networking
│   ├── Network access:
│   │   ├── Enable public access from all networks (default)
│   │   ├── Enable public access from selected VNets and IPs
│   │   │   ├── Add VNet/Subnet
│   │   │   └── Add IP range
│   │   └── Disable public access (private endpoints only)
│   ├── Private endpoint: + Add (optional)
│   └── Network routing: Microsoft / Internet
│
├── Data Protection
│   ├── Enable point-in-time restore: Off (requires versioning + change feed)
│   ├── Enable soft delete for blobs: ✅ 7 days (default)
│   ├── Enable soft delete for containers: Off (default)
│   ├── Enable soft delete for file shares: ✅ 7 days (default)
│   ├── Enable versioning: Off (default)
│   └── Enable change feed: Off
│
├── Encryption
│   ├── Encryption type: Microsoft-managed keys / Customer-managed keys
│   ├── Infrastructure encryption: Enable (optional)
│   │   ⚠️ Must be set NOW — cannot add later
│   └── Enable encryption for: All services
│
├── Tags → Review + Create
│
└── RBAC: Storage Account Contributor or Contributor
```

---

### 22.2 Configure Blob Access Tiers & Lifecycle Management

> **Portal:** `Storage Account → Data management → Lifecycle management`

```
Configure Lifecycle Management
│
├── Prerequisites
│   ├── Storage account: GPv2 or BlobStorage
│   └── Works on: Block blobs and Append blobs
│
├── Portal: Storage Account → Data management → Lifecycle management
│
├── + Add rule
│   ├── Rule name
│   ├── Rule scope: Apply to all blobs / Limit with filters
│   ├── Blob type: Block blobs / Append blobs
│   │
│   ├── Base blob conditions:
│   │   ├── Last modified: More than X days ago
│   │   │   ├── → Move to Cool tier (after 30 days)
│   │   │   ├── → Move to Cold tier (after 60 days)
│   │   │   ├── → Move to Archive tier (after 90 days)
│   │   │   └── → Delete blob (after 365 days)
│   │   └── ⚠️ Archive = offline, must rehydrate to access
│   │
│   ├── Snapshot conditions (optional):
│   │   └── Delete snapshots after X days
│   │
│   ├── Version conditions (optional):
│   │   └── Move/delete previous versions
│   │
│   └── Filter set (if scoped):
│       └── Prefix match: container1/logs/
│
└── Add → Save
    ⚠️ Rules run once per day (not instant)
    ⚠️ Moving from Cool → Hot incurs data retrieval cost
    ⚠️ Early deletion fees: Cool 30d, Cold 90d, Archive 180d
```

---

### 22.3 Generate and Use SAS Token

> **Portal:** `Storage Account → Shared access signature`

```
Generate SAS Token
│
├── Method 1: Account SAS (Portal)
│   │   Portal: Storage Account → Security + networking → Shared access signature
│   ├── Allowed services: ✅ Blob / ✅ File / ✅ Queue / ✅ Table
│   ├── Allowed resource types: ✅ Service / ✅ Container / ✅ Object
│   ├── Allowed permissions: Read / Write / Delete / List / Add / Create
│   ├── Start and expiry date/time
│   │   ⚠️ Set shortest possible expiry
│   ├── Allowed IP addresses (optional): restrict to specific IPs
│   ├── Allowed protocols: HTTPS only (recommended) / HTTPS and HTTP
│   ├── Signing key: key1 / key2
│   │   ⚠️ Regenerating this key invalidates SAS
│   └── Generate SAS and connection string
│       ⚠️ Save immediately — SAS shown ONCE (portal doesn't store it)
│
├── Method 2: Service SAS (Container level)
│   │   Portal: Container → Shared access tokens
│   ├── Permissions: Read / Add / Create / Write / Delete / List
│   ├── Start/Expiry
│   ├── Allowed IPs, Protocol
│   └── Generate SAS token and URL
│
├── Method 3: User Delegation SAS (CLI)
│   │   ⚠️ Most secure — signed with Azure AD (not account key)
│   │   ⚠️ Blob service ONLY
│   └── az storage blob generate-sas --account-name <name> \
│         --container-name <c> --name <blob> \
│         --permissions r --expiry 2026-12-31 \
│         --auth-mode login --as-user
│
├── Using SAS
│   └── Append SAS token to URL:
│       https://<name>.blob.core.windows.net/<container>/<blob>?<SAS-token>
│
└── ⚠️ Revoking SAS
    ├── Cannot revoke individual SAS tokens
    ├── Option 1: Regenerate signing key → invalidates ALL SAS signed with that key
    ├── Option 2: Use Stored Access Policy → modify or delete policy
    │   Max 5 policies per container/share/queue/table
    └── Option 3: Let SAS expire naturally
```

---

### 22.4 Configure Network Security (Firewall & Private Endpoint)

> **Portal:** `Storage Account → Security + networking → Networking`

```
Configure Network Security
│
├── Option 1: Restrict to VNets/IPs
│   │   Portal: Storage Account → Networking → Firewalls and virtual networks
│   ├── Public network access: Enabled from selected virtual networks and IP addresses
│   ├── Virtual networks: + Add existing VNet/subnet
│   │   ⚠️ Enables Service Endpoint on the subnet automatically
│   ├── Firewall: Add client IP address / IP range
│   │   ⚠️ Your IP must be listed or you lose portal access
│   ├── Exceptions:
│   │   └── Allow trusted Microsoft services: ✅ (recommended)
│   │       ⚠️ Allows Azure Backup, Monitor, etc. to access storage
│   └── Save
│
├── Option 2: Private Endpoint
│   │   Portal: Storage Account → Networking → Private endpoint connections → + Private endpoint
│   ├── Subscription, RG, Name
│   ├── Target sub-resource: blob / file / queue / table / web / dfs
│   ├── VNet and Subnet
│   ├── DNS integration: ✅ (creates Private DNS zone)
│   └── Create
│       ⚠️ Each service (blob, file, etc.) needs separate private endpoint
│
├── Option 3: Disable Public Access Entirely
│   │   Portal: Storage Account → Networking
│   ├── Public network access: Disabled
│   └── Access only via Private Endpoints
│       ⚠️ Portal access also blocked unless you are on the VNet
│
└── ⚠️ Notes
    ├── Service Endpoint = optimized route, still public IP
    ├── Private Endpoint = private IP in VNet (fully private)
    ├── "Allow trusted services" bypass = critical for Azure services
    └── NFS file shares REQUIRE private endpoint (no public access)
```

---

### 22.5 Configure Azure File Sync

> **Portal:** `Home → Storage Sync Service → + Create`

```
Configure Azure File Sync
│
├── Prerequisites
│   ├── Azure File share exists (Standard or Premium)
│   ├── Windows Server 2016+ with File Sync agent installed
│   ├── Agent download: Microsoft Download Center
│   ├── .NET Framework 4.7.2+ on the server
│   └── RBAC: Contributor on Storage Account + Storage Sync Service
│
├── Step 1: Create Storage Sync Service
│   │   Portal: Home → Storage Sync Services → + Create
│   ├── Subscription, RG, Name, Region
│   └── Create
│
├── Step 2: Install Agent on Windows Server
│   ├── Download Azure File Sync agent (StorageSyncAgent.msi)
│   ├── Install on Windows Server
│   └── Register server with Storage Sync Service
│       ⚠️ Server appears under "Registered servers" in portal
│
├── Step 3: Create Sync Group
│   │   Portal: Storage Sync Service → Sync groups → + Sync group
│   ├── Sync group name
│   ├── Cloud endpoint:
│   │   ├── Subscription → Storage Account → File Share
│   │   └── ⚠️ One cloud endpoint per sync group
│   └── Create
│
├── Step 4: Add Server Endpoint
│   │   Portal: Sync group → + Add server endpoint
│   ├── Registered server: Select
│   ├── Path: D:\SharedFolder (local folder to sync)
│   ├── Cloud Tiering: Enable / Disable
│   │   ├── Volume free space policy: e.g., 20% (keep 20% free)
│   │   └── Date policy: tier files not accessed in X days
│   │   ⚠️ Cloud tiering = stubs local, full files in cloud
│   └── Add
│
└── ⚠️ Notes
    ├── Max 1 cloud endpoint per sync group
    ├── Multiple server endpoints (multi-site sync) per sync group
    ├── Cloud tiering is the key exam concept
    └── Initial sync uploads all files to Azure File share
```

---

### 22.6 Configure Immutable Storage (WORM)

> **Portal:** `Storage Account → Containers → Select → Access policy`

```
Configure Immutable Storage
│
├── Portal: Storage Account → Containers → Select container → Access policy
│
├── Option 1: Time-Based Retention Policy
│   │   + Add policy → Time-based retention
│   ├── Retention period: X days (e.g., 365)
│   ├── State: Unlocked (testing) → Locked (production)
│   │   ⚠️ UNLOCKED: Can modify retention, test policies
│   │   ⚠️ LOCKED: IRREVERSIBLE — cannot unlock, shorten, or delete
│   │   Only action on locked: EXTEND retention period
│   ├── Allow protected append writes: Optional
│   │   (allows appending to append blobs in immutable container)
│   └── Save
│
├── Option 2: Legal Hold
│   │   + Add policy → Legal hold
│   ├── Tag name: e.g., "case-2026-001"
│   ├── Multiple tags supported
│   ├── Data locked until ALL tags removed
│   │   ⚠️ No expiry — manually remove tags when hold lifted
│   └── Save
│
├── Version-Level Immutability (Alternative)
│   ├── Enabled at account level during creation
│   ├── Set policies per blob version (not per container)
│   └── More granular than container-level
│
└── ⚠️ Notes
    ├── WORM = compliance (SEC Rule 17a-4, HIPAA, etc.)
    ├── Test with Unlocked policy first
    ├── Locking is PERMANENT — triple-check before locking
    ├── Cannot delete storage account with locked immutable policy
    └── Legal hold overrides retention — data locked indefinitely
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
