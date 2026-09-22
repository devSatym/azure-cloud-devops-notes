<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Blob Storage — AZ-104 Revision Notes

---

## 1. What is Azure Blob Storage?

- **Object storage** service for unstructured data (images, videos, backups, logs, documents)
- Massively scalable — stores petabytes of data
- Accessed via **REST API (HTTP/HTTPS)**, CLI, PowerShell, SDKs, AzCopy, Storage Explorer
- Endpoint: `https://<storage-account>.blob.core.windows.net/<container>/<blob>`
- Supports: access tiers, lifecycle management, versioning, soft delete, immutability, replication, CDN

---

## 2. Key Components

| Component | Description |
|---|---|
| **Storage Account** | Top-level namespace — globally unique name |
| **Container** | Logical grouping of blobs (like a folder). Flat structure internally |
| **Blob** | Individual object (file) stored in a container |
| **Virtual Directory** | Use `/` in blob name to simulate folders (e.g., `logs/2026/jan/file.txt`) |

### Hierarchy
```
Storage Account
 └── Container 1
      ├── blob1.jpg
      ├── blob2.pdf
      └── logs/2026/file.txt  ← virtual directory (part of blob name)
 └── Container 2
      └── data.csv
```

> ⚠️ **EXAM TIP:** Blob storage is **flat** — no real folders. Virtual directories are simulated using `/` in blob names. Only **Data Lake Gen2** (hierarchical namespace) has true directories.

---

## 3. Blob Types

| Type | Description | Operations | Max Size | Use Case |
|---|---|---|---|---|
| **Block Blob** | Composed of blocks (up to 50,000 blocks) | Upload, replace blocks | **190.7 TiB** (4,000 MiB × 50,000 blocks) | Files, images, videos, backups — **most common** |
| **Append Blob** | Optimized for append-only operations | Append blocks only | **195 GiB** (4 MiB × ~50,000 blocks) | Logging, audit trails, telemetry |
| **Page Blob** | Random read/write access in 512-byte pages | Read/write any page | **8 TiB** | VM disks (VHDs), databases |

> ⚠️ **EXAM TIP:** **Block blob** = default, most common. **Append blob** = log files (cannot modify existing, only append). **Page blob** = Azure managed disks / VHDs stored as page blobs.

> ⚠️ **EXAM TIP:** Blob type is set at **upload time** and **CANNOT be changed** after creation. Must re-upload as a new blob.

> ⚠️ **EXAM TIP:** Only **Block Blobs** support all access tiers (Hot/Cool/Cold/Archive). Page Blobs and Append Blobs do NOT support Archive tier.

---

## 4. Access Tiers

| Tier | Storage Cost | Access Cost | Min Retention | Availability SLA | Status |
|---|---|---|---|---|---|
| **Hot** | Highest | Lowest | None | 99.9% (RA: 99.99%) | Online |
| **Cool** | Lower | Higher | **30 days** | 99% (RA: 99.9%) | Online |
| **Cold** | Lower than Cool | Higher than Cool | **90 days** | 99% (RA: 99.9%) | Online |
| **Archive** | Lowest | Highest | **180 days** | N/A | **Offline** |

### Tier Setting Scope

| Scope | Options | Notes |
|---|---|---|
| **Account-level default** | Hot or Cool only | Set during or after creation |
| **Blob-level override** | Hot, Cool, Cold, Archive | Overrides account default for individual blobs |

#### Portal Path — Set Account Default Tier
```
Storage Account → Settings → Configuration →
Default access tier: Hot / Cool → Save
```

#### Portal Path — Set Individual Blob Tier
```
Storage Account → Containers → Select container →
Select blob → Change tier → Hot / Cool / Cold / Archive → Save
```

> ⚠️ **EXAM TIP:** Account default = **Hot or Cool** only. Cold and Archive are **blob-level only** — cannot be set as account default.

> ⚠️ **EXAM TIP:** **Archive** = blob is **OFFLINE**. Cannot be read, downloaded, or modified. Must **rehydrate** first (change tier to Hot/Cool/Cold).

> ⚠️ **EXAM TIP:** **Early deletion fee**: Deleting/moving a blob before minimum retention incurs a charge equal to the remaining days. Cool = 30 days, Cold = 90 days, Archive = 180 days.

### Cost Trade-off Summary
```
Storage cost:    Hot > Cool > Cold > Archive
Access cost:     Archive > Cold > Cool > Hot
Retrieval cost:  Archive > Cold > Cool > Hot (free)
```

---

## 5. Rehydration from Archive

| Method | Description |
|---|---|
| **Change tier (in-place)** | Set blob tier from Archive → Hot/Cool/Cold. Blob stays at same URL |
| **Copy blob** | Copy archive blob to a new blob in Hot/Cool/Cold tier. Original stays archived |

### Rehydration Priority

| Priority | Duration | Cost |
|---|---|---|
| **Standard** | Up to **15 hours** | Lower |
| **High** | Under **1 hour** (for blobs < 10 GB) | Higher |

#### Portal Path
```
Storage Account → Containers → Select blob (Archive) →
Change tier → Select Hot/Cool/Cold →
Rehydrate priority: Standard / High → Save
```

> ⚠️ **EXAM TIP:** Standard rehydration = **up to 15 hours**. **High priority** = sub-1 hour but costs more. Can **check status**: blob properties → `x-ms-archive-status: rehydrate-pending-to-hot`.

> ⚠️ **EXAM TIP:** **Copy rehydration** creates a NEW blob — original remains in Archive. **Change tier** rehydrates the same blob in-place.

---

## 6. Lifecycle Management

- **Automate** blob tier transitions and deletion based on age/access patterns
- Rules run **once per day** (not real-time)
- Apply to: **Block Blobs** and **Append Blobs** (Append: only delete actions)
- Filter by: container name, blob prefix, blob index tags

### Available Actions

| Action | Applicable To | Description |
|---|---|---|
| `tierToCool` | Block blob (base/snapshot/version) | Move to Cool |
| `tierToCold` | Block blob (base/snapshot/version) | Move to Cold |
| `tierToArchive` | Block blob (base/snapshot/version) | Move to Archive |
| `delete` | Block blob, Append blob | Delete blob |
| `enableAutoTierToHotFromCool` | Block blob (base only) | Auto-move to Hot when accessed |

### Conditions (Triggers)

| Condition | Description |
|---|---|
| `daysAfterModificationGreaterThan` | Days since last modified |
| `daysAfterCreationGreaterThan` | Days since created |
| `daysAfterLastAccessTimeGreaterThan` | Days since last accessed (requires access tracking) |
| `daysAfterLastTierChangeGreaterThan` | Days since tier was changed |

### Example Rule Logic
```
IF blob not modified for 30 days → Move to Cool
IF blob not modified for 90 days → Move to Cold  
IF blob not modified for 180 days → Move to Archive
IF blob not modified for 365 days → Delete
```

#### Portal Path
```
Storage Account → Data management → Lifecycle management →
+ Add rule → Rule name →
  Filter: Limit to containers/prefixes (optional) →
  Base blobs tab:
    Move to Cool after 30 days
    Move to Archive after 180 days
    Delete after 365 days →
  Snapshots tab: (optional actions) →
  Versions tab: (optional actions) →
Add
```

> ⚠️ **EXAM TIP:** Lifecycle management is THE exam answer for "automatically optimize storage costs" or "move infrequently accessed data to cheaper tier."

> ⚠️ **EXAM TIP:** `daysAfterLastAccessTimeGreaterThan` requires **Last Access Time Tracking** enabled. Portal: `Storage Account → Data management → Lifecycle management → Enable access tracking`.

> ⚠️ **EXAM TIP:** Moving from a **cooler** to a **hotter** tier (e.g., Cool → Hot) incurs **data retrieval charges** but not early deletion fees.

---

## 7. Blob Versioning

- Automatically maintains **previous versions** of a blob when it's overwritten or deleted
- Each version has a **unique version ID** (timestamp-based)
- Previous versions are read-only
- Current version = the blob itself; previous versions accessible by version ID

#### Portal Path
```
Storage Account → Data management → Data protection →
Enable versioning for blobs: ✅ → Save
```

### Behavior

| Action | Result |
|---|---|
| Overwrite blob | Old content saved as previous version. New content = current version |
| Delete blob | Current version becomes a previous version (soft state) |
| Access version | Append `?versionid=<id>` to blob URL |

> ⚠️ **EXAM TIP:** Blob versioning creates a **new version for every write operation** — can significantly increase storage costs. Use **Lifecycle Management** to auto-delete old versions.

> ⚠️ **EXAM TIP:** Versioning requires **GPv2** or **BlobStorage** account. Not supported on accounts with hierarchical namespace (Data Lake Gen2).

---

## 8. Soft Delete

### 8.1 Blob Soft Delete
- Deleted blobs retained for a configurable retention period
- Recover deleted blobs within the window
- Default: **Enabled, 7 days** retention

#### Portal Path
```
Storage Account → Data management → Data protection →
Enable soft delete for blobs: ✅ →
Retention period: 7 days (1–365) → Save
```

### 8.2 Container Soft Delete
- Recover deleted containers within retention window
- Default: **Disabled**

#### Portal Path
```
Storage Account → Data management → Data protection →
Enable soft delete for containers: ✅ →
Retention period: 7 days (1–365) → Save
```

### Soft Delete Comparison

| Feature | Blob Soft Delete | Container Soft Delete |
|---|---|---|
| Default | ✅ Enabled (7 days) | ❌ Disabled |
| Retention | 1–365 days | 1–365 days |
| Recovery | Undelete blob | Undelete container |
| Versioning interaction | Works with versions | Independent |

> ⚠️ **EXAM TIP:** Blob soft delete = **enabled by default (7 days)**. Container soft delete = **disabled by default**. Both are independent — enabling one doesn't enable the other.

> ⚠️ **EXAM TIP:** Soft-deleted data **occupies storage** and is billed — factor into cost calculations.

---

## 9. Point-in-Time Restore

- Restore **block blobs** to a previous state (within retention window)
- Works at **container** or **blob prefix** level
- **NOT** full account restore — only block blobs

### Prerequisites (ALL required)
1. **Blob versioning** — Enabled
2. **Blob soft delete** — Enabled
3. **Change feed** — Enabled
4. **No** hierarchical namespace (Data Lake Gen2)

#### Portal Path
```
Storage Account → Data management → Data protection →
Enable point-in-time restore for block blobs: ✅ →
Retention period → Save
```

> ⚠️ **EXAM TIP:** Point-in-time restore requires THREE features enabled: **versioning + soft delete + change feed**. If exam asks "what must be enabled for PITR" → all three.

> ⚠️ **EXAM TIP:** PITR restores block blobs ONLY — not append blobs, page blobs, file shares, queues, or tables.

---

## 10. Snapshots

- **Read-only** copy of a blob at a specific point in time
- Created **manually** (unlike versioning which is automatic)
- Snapshot URL: `<blob-url>?snapshot=<datetime>`
- Stored as **differences** from base blob (incremental cost)
- Deleting base blob with snapshots requires `include=snapshots` header

#### Portal Path
```
Storage Account → Containers → Select blob →
Create snapshot (via ... menu or REST API)
```

### Snapshots vs Versioning

| Feature | Snapshots | Versioning |
|---|---|---|
| **Trigger** | Manual | Automatic (on every write) |
| **Read-only?** | ✅ Yes | ✅ Yes (previous versions) |
| **Cost** | Incremental (differences) | Full version stored |
| **Restore** | Copy snapshot over current | Promote version to current |
| **Identification** | `?snapshot=<datetime>` | `?versionid=<id>` |

> ⚠️ **EXAM TIP:** Snapshots = **manual**. Versioning = **automatic**. Both are read-only copies. Use versioning for automatic protection; snapshots for point-in-time manual backups.

---

## 11. Change Feed

- Provides a **log of all changes** to blobs and blob metadata
- Stored as **append blobs** in `$blobchangefeed` container
- Records: Create, Update, Delete operations
- **Required** for point-in-time restore and object replication

#### Portal Path
```
Storage Account → Data management → Data protection →
Enable blob change feed: ✅ → Save
```

> ⚠️ **EXAM TIP:** Change feed is required for **Point-in-Time Restore** and **Object Replication** (source account).

---

## 12. Object Replication

- **Asynchronous** replication of block blobs between storage accounts
- Cross-region, cross-subscription, cross-tenant
- Source → Destination (one direction per rule)
- Use cases: latency reduction, compliance, data distribution

### Prerequisites

| Requirement | Source | Destination |
|---|---|---|
| **Blob versioning** | ✅ Required | ✅ Required |
| **Change feed** | ✅ Required | ❌ Not required |
| **Blob type** | Block blobs only | Block blobs only |
| **Account kind** | GPv2 or BlobStorage | GPv2 or BlobStorage |

#### Portal Path
```
Storage Account → Data management → Object replication →
+ Set up replication rules →
Source storage account, Destination storage account →
Source container → Destination container →
Copy over filter: All / Prefix →
Create
```

### What Is NOT Replicated

| Item | Replicated? |
|---|---|
| Blob data (block blobs) | ✅ |
| Blob metadata and tags | ✅ |
| Snapshots | ❌ |
| Blob in Archive tier | ❌ |
| Append / Page blobs | ❌ |
| Immutable blobs | Depends on policy |

> ⚠️ **EXAM TIP:** Object replication: **versioning on BOTH** + **change feed on SOURCE**. Only **Block Blobs**. Snapshots and Archive blobs NOT replicated.

---

## 13. Immutable Blob Storage (WORM)

- **Write Once, Read Many** — prevents modification and deletion
- Compliance: SEC 17a-4(f), CFTC, FINRA, HIPAA

### Policy Types

| Policy | Expiry | Description |
|---|---|---|
| **Time-Based Retention** | Set period (days) | Data immutable for specified duration |
| **Legal Hold** | No expiry | Data locked until all hold tags removed |

### Time-Based Retention States

| State | Modify Policy | Delete Blobs | Delete Container |
|---|---|---|---|
| **Unlocked** | ✅ Can modify/delete policy | ❌ | ❌ |
| **Locked** | Only **extend** retention | ❌ | ❌ |

### Version-Level vs Container-Level

| Feature | Container-Level | Version-Level |
|---|---|---|
| **Scope** | All blobs in container | Individual blob versions |
| **Granularity** | Coarse | Fine |
| **Account setup** | Any GPv2 | Must enable at account creation |
| **Default policy** | On container | On account + override per version |

#### Portal Path
```
Storage Account → Containers → Select container →
Access policy → + Add policy →
Type: Time-based retention / Legal hold →
Retention period (days) → OK

Lock policy (time-based only):
Select policy → Lock → Confirm
⚠️ IRREVERSIBLE — cannot unlock, shorten, or delete
```

> ⚠️ **EXAM TIP:** **Locked** time-based retention = **IRREVERSIBLE**. Can only **extend**, never shorten or remove. Test with **Unlocked** first.

> ⚠️ **EXAM TIP:** Cannot delete a storage account or container with a **locked immutable policy** — must wait for retention to expire or remove all legal holds.

---

## 14. Blob Index Tags

- **Key-value metadata tags** stored with the blob (up to 10 tags per blob)
- **Queryable** across the storage account using tag filter expressions
- Used for: discovery, categorization, lifecycle management filters

#### Portal Path
```
Storage Account → Containers → Select blob →
Blob index tags → Add tag (key=value) → Save
```

### Tag Properties

| Property | Limit |
|---|---|
| Tags per blob | **10** |
| Key length | 1–128 characters |
| Value length | 0–256 characters |
| Characters | Alphanumeric, space, +, -, ., /, :, =, _ |

### Query Syntax
```
"Department" = 'Finance' AND "Status" = 'Active'
```

#### CLI
```bash
az storage blob list --account-name <name> --container-name <c> \
  --tag-filter "\"Department\"='Finance'"
```

> ⚠️ **EXAM TIP:** Blob Index Tags are **queryable** across the entire storage account. Regular blob **metadata** (custom headers) is NOT queryable. Tags = findable, metadata = per-blob only.

---

## 15. Blob Lease

- **Exclusive lock** on a blob or container — prevents modification/deletion
- Lease duration: **15–60 seconds** or **infinite** (-1)
- Must acquire lease before modifying a leased blob
- Used for: distributed locking, preventing concurrent writes

### Lease States

| State | Description |
|---|---|
| **Available** | No active lease |
| **Leased** | Active lock (must include lease ID to modify) |
| **Expired** | Lease duration passed |
| **Breaking** | Lease break requested, waiting to expire |
| **Broken** | Lease broken, available for new lease |

### Lease Operations

| Operation | Description |
|---|---|
| **Acquire** | Get a new lease on blob/container |
| **Renew** | Extend an active lease |
| **Change** | Swap lease ID while maintaining lock |
| **Release** | End the lease voluntarily |
| **Break** | Force-end the lease |

> ⚠️ **EXAM TIP:** A blob with an **active lease** cannot be deleted or modified without providing the lease ID. Break or release the lease first.

> ⚠️ **EXAM TIP:** **Container lease** prevents the container from being deleted. **Blob lease** prevents the blob from being modified/deleted.

---

## 16. Containers

### Container Access Levels

| Level | Anonymous Access |
|---|---|
| **Private** (default) | ❌ No anonymous access |
| **Blob** | ✅ Read individual blobs (must know exact URL) |
| **Container** | ✅ Read + list all blobs in container |

> ⚠️ **EXAM TIP:** Public access must be enabled at **BOTH** account level AND container level. Account-level disabled → all containers private regardless of container setting.

### Container Operations

| Operation | CLI |
|---|---|
| Create | `az storage container create --account-name <sa> -n <name>` |
| List | `az storage container list --account-name <sa> -o table` |
| Delete | `az storage container delete --account-name <sa> -n <name>` |
| Set access | `az storage container set-permission --account-name <sa> -n <name> --public-access blob` |

#### Portal Path — Set Container Access
```
Storage Account → Data storage → Containers → Select container →
Change access level → Private / Blob / Container → OK
```

---

## 17. Security

### 17.1 Authorization Methods

| Method | Scope | Security Level | Works With |
|---|---|---|---|
| **Azure AD (Entra ID)** | RBAC roles | ✅ Most secure | Blob, Queue |
| **Storage Account Key** | Full account access | ❌ Risky (full access) | All services |
| **SAS (Shared Access Signature)** | Scoped & time-limited | ⚠️ Moderate | All services |
| **Anonymous access** | Public read | ❌ Least secure | Blob only |

### 17.2 SAS Types

| SAS Type | Signed With | Scope | Best For |
|---|---|---|---|
| **User Delegation SAS** | Azure AD credentials | Blob only | ✅ Most secure SAS |
| **Service SAS** | Account key | Single service | Container/blob-level access |
| **Account SAS** | Account key | Multiple services | Broad access |

### 17.3 Stored Access Policy
- Defined on container, queue, table, or share
- SAS references the policy → modify policy to change SAS behavior
- Can **revoke** SAS by deleting/modifying the policy
- Max **5 stored access policies** per resource

> ⚠️ **EXAM TIP:** **User Delegation SAS** = most secure (Azure AD signed). Only for **Blob service**. Eliminates dependency on account keys.

> ⚠️ **EXAM TIP:** Individual SAS tokens **cannot be revoked**. Options: (1) regenerate signing key, (2) modify/delete stored access policy, (3) wait for expiry.

### 17.4 Encryption

| Feature | Description | Default |
|---|---|---|
| **Encryption at rest** | AES-256, always enabled | ✅ Cannot disable |
| **Microsoft-managed keys** | Azure manages keys | ✅ Default |
| **Customer-managed keys** | Keys in Key Vault or Managed HSM | Optional |
| **Encryption scopes** | Different keys per container/blob | Optional |
| **Infrastructure encryption** | Double encryption (service + infra) | Must enable at creation |
| **Encryption in transit** | TLS 1.2 for HTTPS | ✅ Default (require secure transfer) |

### 17.5 Encryption Scopes
- Apply **different encryption keys** to different containers or blobs
- Useful for multi-tenant scenarios (different keys per customer)
- Scope = named encryption configuration (MMK or CMK)

#### Portal Path
```
Storage Account → Security + networking → Encryption →
Encryption scopes → + Add → Name, Type (Microsoft/Customer), Key → Create
```

> ⚠️ **EXAM TIP:** **Encryption scopes** let you use different keys for different containers — exam tests this for compliance scenarios.

---

## 18. Data Transfer Tools

| Tool | Type | Auth Methods | Best For |
|---|---|---|---|
| **AzCopy** | CLI | Azure AD, SAS | Scripted bulk copy/sync |
| **Storage Explorer** | GUI (desktop) | Azure AD, SAS, Keys | Interactive management |
| **Azure Portal** | Web UI | Azure AD | Small file uploads, config |
| **Azure Data Box** | Physical device | N/A | Offline bulk transfer (TBs) |
| **Import/Export** | Physical disks | N/A | Ship your own disks |

### AzCopy Commands

| Command | Description |
|---|---|
| `azcopy login` | Authenticate with Azure AD |
| `azcopy copy "<src>" "<dst>"` | Copy files/blobs |
| `azcopy sync "<src>" "<dst>"` | Sync (mirror source to dest) |
| `azcopy copy "<local>" "<blob-url>?<SAS>"` | Upload with SAS |
| `azcopy copy "<blob-url>?<SAS>" "<local>"` | Download with SAS |
| `azcopy copy "<src-url>?<SAS>" "<dst-url>?<SAS>"` | Copy between accounts (server-side) |
| `azcopy remove "<url>?<SAS>"` | Delete blobs |
| `azcopy bench "<url>?<SAS>"` | Benchmark performance |

### AzCopy Copy vs Sync

| Feature | `azcopy copy` | `azcopy sync` |
|---|---|---|
| Direction | Source → Destination | Source → Destination |
| Existing files | Always copies all | Skips unchanged files |
| Deletes extras | ❌ | ✅ (with `--delete-destination`) |
| Use case | Full copy/migration | Ongoing synchronization |

> ⚠️ **EXAM TIP:** `azcopy copy` = copies everything (overwrite). `azcopy sync` = incremental (only changed files). Sync is more efficient for recurring transfers.

> ⚠️ **EXAM TIP:** AzCopy can copy **between storage accounts** (server-side copy — data doesn't pass through local machine). Use SAS or Azure AD on both sides.

---

## 19. Static Website Hosting

- Host **static sites** (HTML, CSS, JS, images) directly from Blob Storage
- Content served from the **`$web`** container (auto-created)
- Endpoint: `https://<account>.z<region-code>.web.core.windows.net`

### Configuration

| Setting | Description |
|---|---|
| Index document | Default page (e.g., `index.html`) |
| Error document | Custom 404 page (e.g., `404.html`) |
| Custom domain | Supported (CNAME mapping) |
| SSL for custom domain | Requires **Azure CDN** |

#### Portal Path
```
Storage Account → Data management → Static website →
Status: Enabled →
Index document name: index.html →
Error document path: 404.html → Save
```

> ⚠️ **EXAM TIP:** Static website files go in the **`$web`** container. For custom domain with HTTPS → place **Azure CDN** in front. Can't use Azure-managed SSL on custom domain without CDN.

---

## 20. Monitoring & Diagnostics

### Key Metrics

| Metric | Description |
|---|---|
| **BlobCapacity** | Total blob data stored (bytes) |
| **BlobCount** | Number of blobs |
| **ContainerCount** | Number of containers |
| **Transactions** | Number of API requests |
| **Ingress** | Data received (bytes) |
| **Egress** | Data sent out (bytes) |
| **SuccessE2ELatency** | End-to-end latency |
| **SuccessServerLatency** | Server processing time |
| **Availability** | % of successful requests |

#### Portal Path — Metrics
```
Storage Account → Monitoring → Metrics →
Metric Namespace: Blob →
Metric: BlobCapacity / Transactions / Ingress / Egress → Apply
```

### Diagnostic Logs
- Log: read, write, delete operations
- Destination: Log Analytics, Storage Account, Event Hub

#### Portal Path
```
Storage Account → Monitoring → Diagnostic settings →
+ Add diagnostic setting → blob →
Logs: StorageRead / StorageWrite / StorageDelete →
Metrics: Transaction →
Destination: Log Analytics workspace → Save
```

> ⚠️ **EXAM TIP:** Diagnostic settings for blobs are configured under `blob` resource. Each service (blob/file/queue/table) has separate diagnostic settings.

---

## 21. RBAC Roles for Blob

| Role | Permissions |
|---|---|
| **Storage Blob Data Owner** | Full read/write/delete + manage POSIX ACLs (Data Lake) |
| **Storage Blob Data Contributor** | Read/write/delete blobs |
| **Storage Blob Data Reader** | Read-only access to blobs |
| **Storage Blob Delegator** | Get user delegation key (for User Delegation SAS) |
| **Storage Account Contributor** | Manage account (NOT blob data) |
| **Reader and Data Access** | Read everything + list keys |

> ⚠️ **EXAM TIP:** **Storage Blob Data Contributor** = read/write blob data. **Storage Account Contributor** = manage the account but CANNOT access blob data. This difference is exam-critical.

> ⚠️ **EXAM TIP:** **Storage Blob Delegator** role is required to create **User Delegation SAS** — it allows getting the delegation key from Azure AD.

---

## 22. Pricing Key Points

| Factor | Details |
|---|---|
| **Storage capacity** | Per GB/month — varies by tier (Hot most, Archive least) |
| **Operations** | Per 10,000 ops — varies by tier (Archive most, Hot least) |
| **Read access (retrieval)** | Free for Hot; charged for Cool/Cold/Archive |
| **Data transfer** | Ingress = FREE; Egress = per GB charged |
| **Rehydration** | Archive → Hot/Cool incurs per-GB retrieval cost |
| **Early deletion** | Cool: 30d, Cold: 90d, Archive: 180d remaining cost |
| **Redundancy** | LRS < ZRS < GRS < RA-GRS |
| **Snapshots/Versions** | Billed for additional data stored |
| **Lifecycle management** | Free (the rules themselves) |

### Tier Cost Comparison (relative)

| | Storage $/GB | Read Ops $/10K | Write Ops $/10K | Retrieval $/GB |
|---|---|---|---|---|
| **Hot** | $$$ | $ | $ | Free |
| **Cool** | $$ | $$ | $$ | $ |
| **Cold** | $ | $$$ | $$$ | $$ |
| **Archive** | ¢ | $$$$ | $$$$ | $$$ |

> ⚠️ **EXAM TIP:** "Cheapest storage for rarely accessed data" → **Archive**. "Cheapest to read frequently" → **Hot**. Lifecycle management optimizes cost over time.

---

## 23. Limitations & Constraints

| Constraint | Limit |
|---|---|
| Max block blob size | **190.7 TiB** (4,000 MiB blocks × 50,000) |
| Max single upload (Put Blob) | **5,000 MiB** (5 GiB) |
| Max append blob size | **195 GiB** |
| Max page blob size | **8 TiB** |
| Max block size | **4,000 MiB** (Block Blob) |
| Max blocks per blob | **50,000** |
| Containers per account | **Unlimited** |
| Blobs per container | **Unlimited** |
| Max blob index tags per blob | **10** |
| Max stored access policies per container | **5** |
| Max IOPS per blob (block) | **500 requests/sec** |
| Max account capacity | **5 PiB** |
| Snapshots per blob | **200** |
| Immutable policies per container | **1** time-based + multiple legal holds |
| Blob name max length | **1,024 characters** |
| Lease duration | **15–60 seconds** or **infinite** |

---

## 24. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| Create container | `az storage container create --account-name <sa> -n <container>` |
| Upload blob | `az storage blob upload --account-name <sa> -c <container> -f file.txt -n blob.txt` |
| Upload directory | `az storage blob upload-batch --account-name <sa> -d <container> -s ./local-dir` |
| Download blob | `az storage blob download --account-name <sa> -c <container> -n blob.txt -f output.txt` |
| Download all | `az storage blob download-batch --account-name <sa> -s <container> -d ./local-dir` |
| List blobs | `az storage blob list --account-name <sa> -c <container> -o table` |
| Delete blob | `az storage blob delete --account-name <sa> -c <container> -n blob.txt` |
| Set tier | `az storage blob set-tier --account-name <sa> -c <container> -n blob.txt --tier Cool` |
| Copy blob | `az storage blob copy start --account-name <sa> --destination-container <c> --destination-blob <b> --source-uri <url>` |
| Show properties | `az storage blob show --account-name <sa> -c <container> -n blob.txt` |
| Snapshot | `az storage blob snapshot --account-name <sa> -c <container> -n blob.txt` |
| Undelete | `az storage blob undelete --account-name <sa> -c <container> -n blob.txt` |
| Set metadata | `az storage blob metadata update --account-name <sa> -c <container> -n blob.txt --metadata key1=value1` |
| Lease acquire | `az storage blob lease acquire --account-name <sa> -c <container> -n blob.txt --lease-duration 60` |
| Lease break | `az storage blob lease break --account-name <sa> -c <container> -n blob.txt` |

### PowerShell

| Action | Command |
|---|---|
| Get context | `$ctx = New-AzStorageContext -StorageAccountName <sa> -StorageAccountKey <key>` |
| Upload blob | `Set-AzStorageBlobContent -Container <c> -File file.txt -Blob blob.txt -Context $ctx` |
| Download blob | `Get-AzStorageBlobContent -Container <c> -Blob blob.txt -Destination output.txt -Context $ctx` |
| List blobs | `Get-AzStorageBlob -Container <c> -Context $ctx` |
| Set tier | `Set-AzStorageBlobTier -Container <c> -Blob blob.txt -Tier Cool -Context $ctx` |
| Copy blob | `Start-AzStorageBlobCopy -SrcBlob <src> -SrcContainer <c> -DestContainer <dc> -DestBlob <db> -Context $ctx` |

---

## 25. Quick-Fire Exam Points ⚡

1. Blob storage = **object store** for unstructured data (images, videos, backups, logs)
2. Three blob types: **Block** (files, most common), **Append** (logging), **Page** (VHDs)
3. Blob type set at **upload time** — CANNOT be changed after creation
4. Only **Block Blobs** support all access tiers (Hot/Cool/Cold/Archive)
5. Account default tier = **Hot or Cool** only. Cold/Archive = **blob-level only**
6. **Archive** = blob is **OFFLINE** — must rehydrate before reading
7. Rehydration: **Standard = 15 hours**, **High Priority = <1 hour**
8. Can rehydrate by **changing tier** (in-place) or **copying** to a new blob
9. Early deletion fees: Cool = **30 days**, Cold = **90 days**, Archive = **180 days**
10. **Lifecycle Management** automates tier transitions — rules run **once per day**
11. `daysAfterLastAccessTimeGreaterThan` requires **access time tracking** enabled
12. **Blob versioning** = automatic on every write. **Snapshots** = manual point-in-time
13. Blob soft delete: **enabled by default (7 days)**. Container soft delete: **disabled by default**
14. **Point-in-Time Restore** requires: versioning + soft delete + change feed (all three)
15. PITR works for **Block Blobs ONLY** — not append, page, files, queues, tables
16. **Change feed** = log of all blob changes in `$blobchangefeed` container
17. **Object replication**: versioning on **BOTH** accounts + change feed on **SOURCE**
18. Object replication: Block Blobs only. No snapshots, no Archive tier blobs
19. **Immutable storage**: Locked time-based retention = **IRREVERSIBLE** (can only extend)
20. **Legal hold** = no expiry — locked until ALL tags manually removed
21. Blob storage is **flat** — virtual directories via `/` in blob name (not real folders)
22. Container access levels: **Private** (default) / Blob (read only) / Container (read + list)
23. Public access needs BOTH **account-level** and **container-level** enabled
24. **User Delegation SAS** = most secure SAS (Azure AD). Blob service only
25. **Stored access policies** = revoke SAS by modifying policy. Max **5** per container
26. **Storage Blob Delegator** role = create User Delegation SAS
27. **Storage Blob Data Contributor** = data access. **Storage Account Contributor** = management only
28. **Encryption at rest** = always on, AES-256, cannot disable
29. **Encryption scopes** = different keys for different containers/blobs
30. **Infrastructure encryption** (double encryption) = must enable at **creation time**
31. `$web` container = **static website** hosting. Custom domain HTTPS needs Azure CDN
32. **AzCopy copy** = full copy. **AzCopy sync** = incremental (skip unchanged)
33. AzCopy can copy **between storage accounts** (server-side, no local download)
34. **Blob lease** = exclusive lock. Duration: **15–60 seconds** or **infinite**
35. **Blob index tags** = queryable (max 10 per blob). Metadata = NOT queryable
36. Max block blob = **190.7 TiB**. Max single upload (Put Blob) = **5 GiB**
37. Max snapshots per blob = **200**
38. Ingress = **FREE**. Egress = charged. Archive retrieval = high cost
39. Diagnostic settings: configure separately for each service (blob/file/queue/table)
40. `az storage blob set-tier` = change individual blob tier via CLI

---

## 26. Step-by-Step Configuration Mind Maps 🗺️

---

### 26.1 Upload Blobs to Container

> **Portal:** `Storage Account → Containers → Select → Upload`

```
Upload Blobs
│
├── Prerequisites
│   ├── Storage account exists (GPv2 recommended)
│   ├── Container exists (or create one)
│   └── RBAC: Storage Blob Data Contributor (Azure AD) or Account Key/SAS
│
├── Method 1: Azure Portal
│   │   Storage Account → Data storage → Containers → Select container → Upload
│   ├── Browse for files → Select
│   ├── Advanced options:
│   │   ├── Blob type: Block blob (default) / Page blob / Append blob
│   │   │   ⚠️ Cannot change type after upload
│   │   ├── Block size: Auto / 64 KiB – 4,000 MiB
│   │   ├── Access tier: Hot / Cool / Cold / Archive
│   │   │   ⚠️ Archive = immediately offline after upload
│   │   ├── Upload to folder: virtual directory path (optional)
│   │   ├── Encryption scope: Default / Custom scope
│   │   └── Overwrite if exists: ✅ / ❌
│   └── Upload
│
├── Method 2: Azure CLI
│   │   # Single file
│   │   az storage blob upload --account-name <sa> -c <container> \
│   │     -f localfile.txt -n blob.txt --tier Cool
│   │   # Directory (batch)
│   │   az storage blob upload-batch --account-name <sa> \
│   │     -d <container> -s ./local-folder
│   └── ⚠️ For files > 256 MiB, CLI auto-uses block upload
│
├── Method 3: AzCopy
│   │   azcopy copy "localfile.txt" "https://<sa>.blob.core.windows.net/<container>/blob.txt?<SAS>"
│   │   # Upload entire directory
│   │   azcopy copy "./localdir/*" "https://<sa>.blob.core.windows.net/<container>?<SAS>" --recursive
│   └── ⚠️ Fastest for bulk uploads (parallel transfers)
│
├── Method 4: PowerShell
│   │   $ctx = New-AzStorageContext -StorageAccountName <sa> -StorageAccountKey <key>
│   │   Set-AzStorageBlobContent -Container <c> -File file.txt -Blob blob.txt -Context $ctx
│
└── ⚠️ Notes
    ├── Max single upload (Put Blob) = 5 GiB → use block upload for larger
    ├── Set tier at upload to avoid tier-change fees
    └── Virtual directories = just "/" in the blob name (no real folders)
```

---

### 26.2 Configure Access Tiers & Rehydrate Archive Blobs

> **Portal:** `Storage Account → Containers → Select blob → Change tier`

```
Configure Access Tiers & Rehydration
│
├── Set Account Default Tier
│   │   Portal: Storage Account → Settings → Configuration
│   ├── Default access tier: Hot / Cool
│   │   ⚠️ Account default = Hot or Cool ONLY
│   │   ⚠️ New blobs without explicit tier inherit account default
│   └── Save
│
├── Set Individual Blob Tier
│   │   Portal: Storage Account → Containers → Select blob → Change tier
│   ├── Access tier: Hot / Cool / Cold / Archive
│   ├── For Archive: blob goes OFFLINE immediately
│   └── Save
│       ⚠️ Changing to cooler tier: check early deletion fees
│       ⚠️ Changing to hotter tier: data retrieval charges apply
│
├── Rehydrate Archive Blob (Method 1: Change Tier)
│   │   Portal: Select archived blob → Change tier → Hot / Cool / Cold
│   ├── Rehydrate priority:
│   │   ├── Standard: Up to 15 hours
│   │   └── High: Under 1 hour (blobs < 10 GB)
│   │       ⚠️ High priority costs significantly more
│   ├── Status: Check blob properties → Archive Status = "rehydrate-pending-to-hot"
│   └── Save
│
├── Rehydrate Archive Blob (Method 2: Copy)
│   │   Creates NEW blob in desired tier; original stays archived
│   ├── CLI: az storage blob copy start --source-uri <archive-url?SAS> \
│   │     --destination-container <c> --destination-blob <new-name> \
│   │     --tier Hot --rehydrate-priority High
│   └── ⚠️ Original blob remains in Archive (pay for both)
│
├── Bulk Tier Change (CLI)
│   │   az storage blob set-tier --account-name <sa> -c <container> \
│   │     -n blobname --tier Cool
│   └── For batch: use lifecycle management rules instead
│
└── ⚠️ Key Exam Points
    ├── Archive → Online: MUST rehydrate (cannot read directly)
    ├── Standard rehydration = up to 15 hours
    ├── High priority = under 1 hour (blobs < 10 GB), higher cost
    ├── Early deletion: Cool 30d, Cold 90d, Archive 180d
    └── Moving Cool → Hot incurs retrieval cost (not early deletion)
```

---

### 26.3 Configure Lifecycle Management Rules

> **Portal:** `Storage Account → Data management → Lifecycle management`

```
Configure Lifecycle Management
│
├── Portal: Storage Account → Data management → Lifecycle management
│
├── + Add rule
│   ├── Details tab:
│   │   ├── Rule name: e.g., "move-old-to-archive"
│   │   ├── Rule scope:
│   │   │   ├── Apply rule to all blobs in account
│   │   │   └── Limit blobs with filters (prefix, index tags)
│   │   ├── Blob type: Block blobs ✅ / Append blobs ✅
│   │   └── Blob subtype: Base blobs / Snapshots / Versions
│   │
│   ├── Base blobs tab:
│   │   ├── Condition: Last modified / Created / Last accessed
│   │   │   ⚠️ "Last accessed" requires access tracking enabled
│   │   ├── Action chain (example):
│   │   │   ├── blob last modified > 30 days → Move to Cool
│   │   │   ├── blob last modified > 90 days → Move to Cold
│   │   │   ├── blob last modified > 180 days → Move to Archive
│   │   │   └── blob last modified > 365 days → Delete
│   │   └── Enable auto-tiering from Cool to Hot on access (optional)
│   │
│   ├── Snapshots tab (optional):
│   │   └── Delete snapshots older than X days
│   │
│   ├── Versions tab (optional):
│   │   ├── Move to Cool/Cold/Archive after X days
│   │   └── Delete versions older than X days
│   │
│   └── Filter set tab (if scoped):
│       ├── Blob prefix: logs/, images/2025/
│       └── Blob index tags: "Department"='Finance'
│
├── Add → Save
│
└── ⚠️ Notes
    ├── Rules run ONCE PER DAY (not instant)
    ├── Rule evaluation may take up to 24–48 hours after creation
    ├── Lifecycle management itself is FREE
    ├── Tier transition costs still apply (retrieval fees)
    ├── Cannot move from Archive to Cool/Cold directly via lifecycle
    │   (lifecycle can only tier DOWN, not UP)
    └── Use for cost optimization: exam favorite topic
```

---

### 26.4 Configure Blob Versioning, Soft Delete & PITR

> **Portal:** `Storage Account → Data management → Data protection`

```
Configure Data Protection Features
│
├── Portal: Storage Account → Data management → Data protection
│
├── Section 1: Recovery
│   ├── Enable point-in-time restore for block blobs: ✅
│   │   ├── Retention: X days (max 365)
│   │   └── ⚠️ Requires: versioning + soft delete + change feed (auto-enabled)
│   │       ⚠️ Block blobs ONLY — not append/page/files
│   │
│   ├── Enable soft delete for blobs: ✅ (default ON)
│   │   ├── Retention: 7 days default (1–365 range)
│   │   └── ⚠️ Soft-deleted blobs count toward storage/cost
│   │
│   └── Enable soft delete for containers: ✅ (default OFF)
│       └── Retention: 1–365 days
│
├── Section 2: Tracking
│   ├── Enable versioning for blobs: ✅
│   │   ├── ⚠️ Creates new version on EVERY write → can increase costs
│   │   └── Use lifecycle management to auto-delete old versions
│   │
│   └── Enable blob change feed: ✅
│       ├── Records all create/update/delete operations
│       └── ⚠️ Required for: PITR and Object Replication
│
├── Save
│
├── Perform Point-in-Time Restore
│   │   Portal: Storage Account → Data management → Point-in-time restore
│   ├── Select restore point (datetime within retention window)
│   ├── Scope: All containers / Specific containers / Blob prefix ranges
│   ├── Restore → Confirm
│   └── ⚠️ Creates a new state — does not create a parallel copy
│
└── ⚠️ Notes
    ├── PITR prerequisites: versioning + soft delete + change feed
    ├── Blob soft delete = ON by default. Container soft delete = OFF by default
    ├── Versioning increases storage (every write = new version)
    ├── Combine with lifecycle management to manage old version costs
    └── PITR restore time depends on data size (can take hours)
```

---

### 26.5 Configure Object Replication

> **Portal:** `Storage Account → Data management → Object replication`

```
Configure Object Replication
│
├── Prerequisites
│   ├── Source and Destination = GPv2 or BlobStorage accounts
│   ├── Source: Blob versioning ✅ + Change feed ✅
│   ├── Destination: Blob versioning ✅
│   ├── Block blobs only (no append, page, snapshots)
│   ├── Blobs NOT in Archive tier
│   └── RBAC: Storage Blob Data Contributor on both accounts
│
├── Portal: Storage Account (source or destination) →
│   Data management → Object replication → + Set up replication rules
│
├── Configuration
│   ├── Source storage account: Select
│   ├── Destination storage account: Select
│   ├── + Add rule:
│   │   ├── Source container
│   │   ├── Destination container
│   │   ├── Filter:
│   │   │   ├── Copy over: All objects / With prefix filter
│   │   │   └── Prefix: e.g., "logs/"
│   │   └── Copy over: Objects created after a specific time (optional)
│   ├── Can add multiple rules (different container pairs)
│   └── Create
│
├── Monitoring
│   │   Source: Check replication status per blob
│   └── az storage blob show → "objectReplicationSourceProperties"
│
└── ⚠️ Notes
    ├── Replication is ASYNCHRONOUS — no SLA on timing
    ├── Cross-region, cross-subscription, cross-tenant supported
    ├── Snapshots are NOT replicated
    ├── Archive tier blobs are NOT replicated
    ├── Does NOT replicate retroactively (only new/modified after rule creation)
    │   Unless "Copy over: All objects" is selected with a time filter
    └── One-way: Source → Destination (for bidirectional, create two rules)
```

---

### 26.6 Configure Immutable Storage (WORM)

> **Portal:** `Storage Account → Containers → Select → Access policy`

```
Configure Immutable Storage (WORM)
│
├── Option 1: Time-Based Retention Policy
│   │   Portal: Container → Access policy → + Add policy → Time-based retention
│   ├── Retention interval: X days (e.g., 365)
│   ├── Save (creates in UNLOCKED state)
│   │   ⚠️ Unlocked = test mode: can modify, shorten, or delete policy
│   │   ⚠️ Blobs still cannot be deleted/modified while policy exists
│   │
│   ├── Lock Policy (when ready):
│   │   Select policy → Lock icon → Confirm
│   │   ⚠️ IRREVERSIBLE — CANNOT unlock
│   │   ⚠️ Can only EXTEND retention after locking
│   │   ⚠️ Cannot delete container or storage account with locked policy
│   │
│   └── Extend Retention (locked policy):
│       Edit policy → increase retention period → Save
│
├── Option 2: Legal Hold
│   │   Portal: Container → Access policy → + Add policy → Legal hold
│   ├── Tag name: e.g., "litigation-2026"
│   ├── Can add multiple tags
│   ├── Data locked until ALL tags manually removed
│   │   ⚠️ No automatic expiry — must remove tags explicitly
│   └── Save
│
├── Behavior Summary:
│   │
│   │ Policy Active (locked/legal hold):
│   ├── Create/Upload new blobs: ✅ (allowed)
│   ├── Read existing blobs: ✅ (allowed)
│   ├── Modify existing blobs: ❌ (blocked)
│   ├── Delete existing blobs: ❌ (blocked)
│   ├── Delete container: ❌ (blocked)
│   └── Delete storage account: ❌ (blocked)
│
└── ⚠️ Exam Points
    ├── LOCKED retention = permanent, irreversible — test with unlocked first
    ├── Legal hold has NO auto-expiry
    ├── Time-based + Legal hold: both can coexist on same container
    ├── Even storage account cannot be deleted with locked policies
    └── Compliance: SEC 17a-4(f), CFTC 1.31, HIPAA
```

---

### 26.7 Generate and Use SAS for Blob Access

> **Portal:** `Storage Account → Shared access signature` or `Container → Shared access tokens`

```
Generate SAS for Blob Access
│
├── Option 1: Account SAS (broad access)
│   │   Portal: Storage Account → Security + networking → Shared access signature
│   ├── Allowed services: Blob ✅ (select as needed)
│   ├── Allowed resource types: Service / Container / Object
│   ├── Allowed permissions: Read / Write / Delete / List / Add / Create
│   ├── Start and expiry date/time
│   │   ⚠️ Use shortest possible expiry
│   ├── Allowed IP addresses (optional)
│   ├── Allowed protocols: HTTPS only ✅ (recommended)
│   ├── Signing key: key1 / key2
│   │   ⚠️ Regenerating this key invalidates ALL SAS signed with it
│   └── Generate SAS and connection string → Copy immediately
│       ⚠️ SAS shown ONCE — not stored by Azure
│
├── Option 2: Service SAS (container/blob level)
│   │   Portal: Container → Settings → Shared access tokens
│   ├── Signing method: Account key / User delegation key
│   │   ⚠️ User delegation = Azure AD (most secure, Blob only)
│   ├── Permissions, Start/Expiry, IP, Protocol
│   └── Generate SAS token and URL
│
├── Option 3: User Delegation SAS (CLI — most secure)
│   │   # Step 1: Login with Azure AD
│   │   az login
│   │   # Step 2: Generate User Delegation SAS
│   │   az storage blob generate-sas --account-name <sa> \
│   │     -c <container> -n <blob> --permissions r \
│   │     --expiry 2026-12-31 --auth-mode login --as-user
│   └── ⚠️ Requires Storage Blob Delegator role
│
├── Using SAS Token
│   ├── Append to blob URL: https://<sa>.blob.core.windows.net/<c>/<b>?<SAS>
│   ├── AzCopy: azcopy copy "source" "dest?<SAS>"
│   └── Storage Explorer: Connect using SAS URI
│
├── Stored Access Policy (for revocation)
│   │   Portal: Container → Access policy → Stored access policies
│   ├── Add policy: Name, Permissions, Start/Expiry
│   ├── Create SAS referencing this policy
│   └── Revoke: Modify or delete the policy → SAS invalidated
│       ⚠️ Max 5 stored access policies per container
│
└── ⚠️ Notes
    ├── User Delegation SAS = most secure (Azure AD, no account key)
    ├── SAS CANNOT be individually revoked
    ├── Revoke via: regenerate key, modify stored access policy, or expiry
    ├── HTTPS only recommended (never http)
    └── Always use minimum permissions and shortest expiry
```

---

### 26.8 Configure Static Website Hosting

> **Portal:** `Storage Account → Data management → Static website`

```
Configure Static Website
│
├── Prerequisites
│   ├── GPv2 or BlockBlobStorage account
│   └── RBAC: Storage Blob Data Contributor (to upload files to $web)
│
├── Step 1: Enable Static Website
│   │   Portal: Storage Account → Data management → Static website
│   ├── Status: Enabled
│   ├── Index document name: index.html
│   ├── Error document path: 404.html (optional)
│   └── Save
│       → Primary endpoint displayed: https://<sa>.z<code>.web.core.windows.net
│       → $web container auto-created
│
├── Step 2: Upload Website Files
│   │   Portal: Storage Account → Containers → $web → Upload
│   ├── Upload: index.html, 404.html, CSS, JS, images
│   ├── CLI: az storage blob upload-batch -d '$web' -s ./website --account-name <sa>
│   └── AzCopy: azcopy copy "./website/*" "https://<sa>.blob.core.windows.net/\$web?<SAS>" --recursive
│
├── Step 3: Test
│   └── Browse to: https://<sa>.z<code>.web.core.windows.net
│
├── Step 4: Custom Domain (optional)
│   │   Portal: Storage Account → Settings → Custom domain
│   ├── Direct CNAME: www.example.com → <sa>.z<code>.web.core.windows.net
│   │   ⚠️ No HTTPS with direct CNAME (HTTP only)
│   └── For HTTPS: Use Azure CDN
│       ├── Create CDN profile and endpoint
│       ├── Origin: Static website endpoint
│       ├── Add custom domain to CDN
│       └── Enable HTTPS on CDN custom domain
│       ⚠️ Custom domain + HTTPS requires Azure CDN
│
└── ⚠️ Notes
    ├── Content served from $web container (auto-created)
    ├── $web = publicly accessible (read-only for visitors)
    ├── Static only — NO server-side processing (no PHP, no Node.js)
    ├── For dynamic apps use App Service or Static Web Apps
    └── CDN improves performance + enables custom domain HTTPS
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
