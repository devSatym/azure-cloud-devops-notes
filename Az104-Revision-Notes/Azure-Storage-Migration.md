<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Live Migration of Storage — AZ-104 Revision Notes

---

## 1. What is Live Migration of Storage?

- Moving or migrating storage **data**, **redundancy**, **account type**, or **region** without downtime
- Covers multiple scenarios: redundancy changes, cross-region moves, cross-subscription moves, data copy/migration
- Azure supports both **live (in-place) migrations** and **manual (copy-based) migrations** depending on the scenario
- Key tools: Portal redundancy change, AzCopy, Azure Data Box, Object Replication, Import/Export

---

## 2. Storage Migration Scenarios Overview

| Scenario | Method | Downtime | Data Copy? |
|---|---|---|---|
| **Change redundancy** (e.g., LRS → GRS) | Portal / CLI (live) | ❌ None | ❌ Azure handles internally |
| **Change redundancy** (e.g., LRS → ZRS) | Support request (live migration) | ❌ None | ❌ Azure handles internally |
| **Move to different region** | Manual — AzCopy / Data Box | ⚠️ Cutover | ✅ Must copy data |
| **Move to different subscription** | Resource Move | ❌ None | ❌ Metadata move |
| **Move to different resource group** | Resource Move | ❌ None | ❌ Metadata move |
| **Upgrade V1 → V2** | Portal / CLI (in-place) | ❌ None | ❌ In-place upgrade |
| **Standard → Premium** | Manual — create new + AzCopy | ⚠️ Cutover | ✅ Must copy data |
| **Premium → Standard** | Manual — create new + AzCopy | ⚠️ Cutover | ✅ Must copy data |
| **Copy data between accounts** | AzCopy / Object Replication | Depends | ✅ |

> ⚠️ **EXAM TIP:** Changing redundancy from LRS/GRS to **ZRS/GZRS** requires a **Microsoft support request** for live migration. Other redundancy changes (LRS ↔ GRS, GRS ↔ RA-GRS) can be done directly in the portal.

---

## 3. Redundancy Change (Live Migration)

### 3.1 Portal-Supported Redundancy Changes (Instant/Background)

These can be changed directly in Portal/CLI — Azure handles data replication internally:

| From | To | Method |
|---|---|---|
| **LRS** → **GRS** | Portal/CLI | Azure copies to secondary region |
| **LRS** → **RA-GRS** | Portal/CLI | Azure copies + enables read on secondary |
| **GRS** → **LRS** | Portal/CLI | Stops geo-replication |
| **GRS** → **RA-GRS** | Portal/CLI | Enables read access on secondary |
| **RA-GRS** → **GRS** | Portal/CLI | Disables read access on secondary |
| **RA-GRS** → **LRS** | Portal/CLI | Stops geo-replication |
| **ZRS** → **GZRS** | Portal/CLI | Adds geo-replication to ZRS |
| **ZRS** → **RA-GZRS** | Portal/CLI | Adds geo-replication + read secondary |
| **GZRS** → **ZRS** | Portal/CLI | Stops geo-replication |
| **GZRS** → **RA-GZRS** | Portal/CLI | Enables read access |
| **RA-GZRS** → **GZRS** | Portal/CLI | Disables read access |
| **RA-GZRS** → **ZRS** | Portal/CLI | Stops geo-replication |

#### Portal Path
```
Storage Account → Settings → Redundancy →
Select new redundancy option → Save
```

> ⚠️ **EXAM TIP:** Switching between LRS ↔ GRS, GRS ↔ RA-GRS, ZRS ↔ GZRS, GZRS ↔ RA-GZRS can be done directly in the **portal with no downtime**.

### 3.2 Migrations Requiring Support Request (LRS/GRS → ZRS/GZRS)

| From | To | Method |
|---|---|---|
| **LRS** → **ZRS** | Microsoft support request (live migration) | Azure migrates across zones |
| **LRS** → **GZRS** | Support request OR manual | Two-step or live |
| **GRS** → **ZRS** | Change to LRS first → then request ZRS | Two-step |
| **GRS** → **GZRS** | Support request OR manual | Live or manual |
| **RA-GRS** → **ZRS** | Change to LRS first → then request ZRS | Two-step |
| **RA-GRS** → **GZRS** | Support request OR Change to GRS then request | Multi-step |
| **RA-GRS** → **RA-GZRS** | Support request | Live migration |

### Live Migration Process (Support Request)

| Step | Detail |
|---|---|
| 1. Submit request | Azure Portal → Support + troubleshooting → New support request |
| 2. Type | "Live migration to ZRS" |
| 3. Azure schedules | Microsoft schedules the migration |
| 4. Data moved | Azure replicates data to 3 Availability Zones |
| 5. Completion | No downtime — endpoint stays the same |

> ⚠️ **EXAM TIP:** Live migration from LRS → ZRS is performed by **Microsoft** — you submit a support request. There is **no self-service portal option** for this. No application downtime.

> ⚠️ **EXAM TIP:** During live migration to ZRS, storage account is **fully accessible** — no downtime. Endpoint and access keys do NOT change.

### 3.3 Manual Migration (Alternative to Live Migration)

- Create new storage account with desired redundancy → Copy data → Switch apps → Delete old
- Use when live migration is not available or you need immediate migration
- Tools: **AzCopy**, **Azure Data Box**, **Storage Explorer**, or **Object Replication**

> ⚠️ **EXAM TIP:** "Move from LRS to ZRS" → Two options: (1) **Live migration** (support request, no downtime) or (2) **Manual migration** (create new account, copy, switch). Exam may test both paths.

---

## 4. Account Type Upgrade (V1 → V2)

- **Free** and **non-disruptive** — no data copy needed
- Enables: access tiers, lifecycle management, all redundancy options
- **One-way** — cannot downgrade V2 → V1

#### Portal Path
```
Storage Account → Settings → Configuration →
Account kind: Upgrade to StorageV2 → Upgrade
```

#### CLI
```bash
az storage account update -g <rg> -n <name> --set kind=StorageV2
```

> ⚠️ **EXAM TIP:** GPv1 → GPv2 upgrade is **free, instant, no downtime, irreversible** (cannot go back). Unlocks access tiers and lifecycle management.

---

## 5. Performance Tier Change (Standard ↔ Premium)

| Scenario | Method |
|---|---|
| **Standard → Premium** | Manual: Create new Premium account → Copy data → Switch |
| **Premium → Standard** | Manual: Create new Standard account → Copy data → Switch |

- **CANNOT change in-place** — different underlying infrastructure (HDD vs SSD)
- Must create a new storage account of the target performance tier
- Copy data with AzCopy, Storage Explorer, or Azure Data Box
- Update application connection strings after migration

> ⚠️ **EXAM TIP:** You **CANNOT convert** Standard ↔ Premium in-place. Must create a new account and copy data. This is a common exam question.

> ⚠️ **EXAM TIP:** Premium accounts (BlockBlobStorage, FileStorage) only support **LRS and ZRS** — no GRS. Consider this when planning migration.

---

## 6. Cross-Region Storage Migration

- Azure does NOT support moving a storage account to a different region directly
- Must create new account in target region → copy data → update apps → delete old

### Method Comparison

| Method | Best For | Speed | Data Size |
|---|---|---|---|
| **AzCopy** | Online copy, any size | Fast (parallel) | Any |
| **Azure Data Box** | Offline, massive datasets | Ship device | 40–100 TB per device |
| **Import/Export** | Offline, ship your own disks | Slower | < 40 TB |
| **Object Replication** | Ongoing async replication (Blobs only) | Background | Any |
| **Storage Explorer** | Interactive, smaller datasets | Medium | Small–medium |

### AzCopy for Cross-Region Migration
```bash
# Copy all blobs from source to destination
azcopy copy "https://source.blob.core.windows.net/container?<SAS>" \
  "https://dest.blob.core.windows.net/container?<SAS>" --recursive

# Sync (incremental)
azcopy sync "https://source.blob.core.windows.net/container?<SAS>" \
  "https://dest.blob.core.windows.net/container?<SAS>" --recursive
```

> ⚠️ **EXAM TIP:** **AzCopy** = preferred tool for online storage migration. Supports **server-to-server copy** (data flows directly between storage accounts, not through local machine).

> ⚠️ **EXAM TIP:** **Azure Data Box** = for massive offline transfers (>40 TB). Microsoft ships a physical device. Data copied to device → shipped to Azure datacenter → uploaded.

---

## 7. Cross-Subscription / Cross-Resource Group Move

### Resource Move (Metadata Only)
- Moves the storage account **resource** to another subscription or RG
- **No data copy** — just changes the management scope
- Endpoint and keys remain the same
- Resource ID changes (new subscription/RG path)

#### Portal Path
```
Storage Account → Overview → Resource group → Move →
Move to another subscription / Move to another resource group →
Select target subscription/RG → Validate → Move
```

#### CLI
```bash
# Move to another resource group
az resource move --destination-group <target-rg> \
  --ids /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<name>

# Move to another subscription
az resource move --destination-group <target-rg> --destination-subscription-id <target-sub> \
  --ids /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<name>
```

### Move Constraints

| Constraint | Details |
|---|---|
| **Storage account locks** | ❌ Must remove locks before move |
| **Resource locks** | ❌ Must remove all locks (ReadOnly and Delete) |
| **Data copy** | NOT needed — metadata move only |
| **Downtime** | ❌ None |
| **Endpoint change** | ❌ No change |
| **Access keys** | ❌ No change |
| **Connection strings** | ❌ No change (unless resource ID is used) |
| **Cross-region** | ❌ Not supported via resource move — same region only |
| **Azure File Sync** | May need reconfiguration after move |
| **Azure Backup** | May need reconfiguration after move |

> ⚠️ **EXAM TIP:** Moving storage account between subscriptions/RGs is a **metadata-only** operation — keys, endpoints, data are unchanged. But **resource locks must be removed first**.

> ⚠️ **EXAM TIP:** Cross-subscription move does NOT move the data — just the resource pointer. The physical storage location is unchanged.

---

## 8. Storage Account Failover (Geo-Redundancy)

- For **GRS/RA-GRS/GZRS/RA-GZRS** accounts only
- **Customer-initiated failover** — makes secondary become new primary
- Data in the secondary region (which may be slightly behind) becomes the new primary
- After failover, account becomes **LRS** in the failed-over region

### Failover Types

| Type | Trigger | RPO |
|---|---|---|
| **Customer-initiated failover** | Portal/CLI manually | ~15 minutes data loss (async replication lag) |
| **Microsoft-initiated failover** | Major regional disaster | Varies |

#### Portal Path
```
Storage Account → Settings → Redundancy →
"Prepare for failover" link → Failover → Confirm
(Only visible for GRS/RA-GRS/GZRS/RA-GZRS)
```

#### CLI
```bash
az storage account failover -g <rg> -n <name>
```

### After Failover

| Item | State |
|---|---|
| Primary region | New region (was secondary) |
| Redundancy | Becomes **LRS** |
| Endpoint | Same (DNS re-maps) |
| Access keys | **Regenerated** — update connection strings |
| Geo-replication | Must reconfigure manually |
| Data loss | Possible — last sync time indicates RPO |

> ⚠️ **EXAM TIP:** After failover: (1) Account becomes **LRS** (must reconfigure GRS), (2) Access keys are **regenerated**, (3) Possible **data loss** from replication lag (~15 min RPO).

> ⚠️ **EXAM TIP:** Check **"Last Sync Time"** property before failover to estimate potential data loss. Portal: `Storage Account → Redundancy → Last sync time`.

> ⚠️ **EXAM TIP:** Failover is only available for **geo-redundant** accounts (GRS/RA-GRS/GZRS/RA-GZRS). LRS and ZRS do NOT support failover.

---

## 9. Data Transfer Tools Comparison

| Tool | Type | Direction | Auth | Best For |
|---|---|---|---|---|
| **AzCopy** | CLI | Online | Azure AD, SAS | Scripted copy/sync between accounts |
| **Storage Explorer** | GUI | Online | Azure AD, SAS, Keys | Interactive management and transfer |
| **Azure Data Box** | Device | Offline | N/A | Massive data (40–100 TB per device) |
| **Azure Data Box Disk** | Disks | Offline | N/A | Up to 35 TB |
| **Azure Data Box Heavy** | Device | Offline | N/A | Up to 1 PB |
| **Import/Export** | Your disks | Offline | N/A | Ship your own drives |
| **Object Replication** | Service | Online | Managed | Async blob replication (ongoing) |

### AzCopy Key Features

| Feature | Details |
|---|---|
| **Server-to-server copy** | ✅ Data flows between Azure storage, not through local machine |
| **Incremental sync** | `azcopy sync` — only copies changed files |
| **Concurrent transfers** | Multiple parallel connections |
| **Authentication** | Azure AD (`azcopy login`) or SAS tokens |
| **Supported services** | Blob, Files |
| **Bandwidth throttput** | `--cap-mbps` flag to limit bandwidth |
| **Log files** | Auto-generated in `~/.azcopy/` |

### AzCopy Commands for Migration

| Action | Command |
|---|---|
| Login (Azure AD) | `azcopy login` |
| Copy blob | `azcopy copy "source?SAS" "dest?SAS"` |
| Copy entire container | `azcopy copy "https://src.blob.../<container>?SAS" "https://dst.blob.../<container>?SAS" --recursive` |
| Sync (incremental) | `azcopy sync "source?SAS" "dest?SAS" --recursive` |
| Copy between accounts (server-side) | `azcopy copy "srcURL?SAS" "dstURL?SAS" --recursive` |
| Copy to different sub/region | Same syntax — both accounts need SAS or Azure AD auth |

> ⚠️ **EXAM TIP:** `azcopy copy` = full copy (always overwrites). `azcopy sync` = incremental (only changed/new files). For ongoing sync → use `azcopy sync`.

---

## 10. Object Replication (Online Blob Migration)

- Asynchronous replication of **block blobs** between accounts
- Cross-region, cross-subscription
- Requires: **Versioning on both** + **Change Feed on source**

| Feature | Details |
|---|---|
| **Direction** | One-way (source → destination) |
| **Blob types** | Block blobs only |
| **Not replicated** | Snapshots, Archive tier, Append/Page blobs |
| **Timing** | Asynchronous — no SLA on timing |
| **Use for migration** | Set up replication → wait for sync → switch apps → delete rule |

> ⚠️ **EXAM TIP:** Object replication is useful for **gradual migration** — set up rules, let Azure replicate in the background, then cutover. But it's **Block Blobs only** and does NOT replicate snapshots.

---

## 11. Import/Export Service

- Ship **physical disks** (HDDs/SSDs) to Azure datacenter
- **Import**: Local → Azure (upload data from your disks to Blob/File storage)
- **Export**: Azure → Local (Azure copies data to your disks, ships to you)
- Use for: Large data migrations where network transfer is impractical

### Import Job Flow

| Step | Action |
|---|---|
| 1. Prepare disks | Format + copy data using **WAImportExport** tool |
| 2. Create Import job | Portal: `Home → Import/Export jobs → + Create` |
| 3. Ship disks | Ship to Azure datacenter (tracking number required) |
| 4. Azure copies | Data uploaded to Blob/File storage |
| 5. Disks returned | Azure ships disks back to you |

### Export Job Flow

| Step | Action |
|---|---|
| 1. Create Export job | Portal: `Home → Import/Export jobs → + Create` |
| 2. Ship disks | Ship empty disks to Azure |
| 3. Azure copies | Data from Blob storage copied to your disks |
| 4. Disks returned | Azure returns disks with data |

> ⚠️ **EXAM TIP:** Import/Export supports **Blob storage and Azure Files** (import only for Files). Export uses **BitLocker encryption** for security.

---

## 12. Azure Data Box Family

| Product | Capacity | Use Case |
|---|---|---|
| **Data Box Disk** | Up to **35 TB** (5 SSDs × 7 TB) | Small–medium offline migration |
| **Data Box** | **100 TB** per device | Large offline migration |
| **Data Box Heavy** | **1 PB** per device | Massive datacenter migration |
| **Data Box Gateway** | Virtual device | Ongoing transfer (online) |

- Microsoft ships the device → You copy data → Ship back → Azure uploads
- Encrypted with AES-256
- Portal: `Home → Data Box → + Create order`

> ⚠️ **EXAM TIP:** Data Box = Microsoft-owned device shipped to you. Import/Export = You ship your **own disks**. Both for offline large-scale transfer.

---

## 13. Security & RBAC for Migration

| Role | Permissions |
|---|---|
| **Storage Account Contributor** | Manage accounts, change redundancy, move resources |
| **Contributor** | Full management including resource move |
| **Owner** | Full access + role assignment |
| **Storage Blob Data Contributor** | Read/write blob data (for AzCopy with Azure AD) |
| **Reader** | View redundancy and failover status |

### Migration Security

| Aspect | Detail |
|---|---|
| **AzCopy auth** | Azure AD (`azcopy login`) or SAS token |
| **Server-to-server copy** | Data stays within Azure backbone |
| **Data Box encryption** | AES-256 BitLocker on device |
| **Import/Export encryption** | BitLocker on disks |
| **Failover** | Keys regenerated after failover |
| **Secure transfer** | Enforce HTTPS during online migration |

---

## 14. Monitoring & Tracking Migration

| Scenario | Monitoring |
|---|---|
| **Redundancy change (Portal)** | Portal: Storage Account → Redundancy → Status |
| **Live migration (ZRS)** | Support ticket status → Azure portal notifications |
| **Failover** | Portal: Redundancy → Last sync time, Failover status |
| **AzCopy** | `azcopy jobs list`, `azcopy jobs show <id>`, log files in `~/.azcopy/` |
| **Object Replication** | Portal: Storage Account → Object replication → Replication status per blob |
| **Import/Export** | Portal: Import/Export jobs → Job status |
| **Data Box** | Portal: Data Box → Order status |

---

## 15. Pricing Key Points

| Scenario | Cost |
|---|---|
| **Redundancy change** | Different rate for new redundancy (pay ongoing storage rate) |
| **Live migration to ZRS** | **Free** (no migration charge from Microsoft) |
| **V1 → V2 upgrade** | **Free** |
| **AzCopy transfer** | Egress charges if cross-region; operations charges |
| **Object Replication** | Operations + egress charges |
| **Failover** | Free (but account becomes LRS — reconfigure GRS for cost) |
| **Data Box** | Per-device fee + shipping + egress for export |
| **Import/Export** | Per-drive handling fee + shipping |
| **Resource move (sub/RG)** | **Free** |

> ⚠️ **EXAM TIP:** Live migration to ZRS = **free**. V1 → V2 upgrade = **free**. Resource move = **free**. These are common exam distractors.

---

## 16. Limitations & Constraints

| Constraint | Detail |
|---|---|
| Standard ↔ Premium conversion | ❌ Cannot convert in-place — must create new + copy |
| Cross-region move | ❌ No direct move — must copy data to new account |
| LRS → ZRS (portal) | ❌ Cannot do in portal — requires support request |
| Failover data loss | ⚠️ Possible — async replication lag (~15 min RPO) |
| After failover redundancy | Becomes **LRS** — must reconfigure geo-replication |
| After failover keys | **Regenerated** — must update connection strings |
| Resource move + locks | ❌ Locks must be removed before move |
| V2 → V1 downgrade | ❌ Not possible (one-way upgrade) |
| Large file shares enabled | ❌ Cannot disable — restricts to LRS/ZRS only |
| Object replication | Block blobs only, no snapshots, no archive |
| Data Box max | 1 PB (Heavy), 100 TB (Standard), 35 TB (Disk) |

---

## 17. Quick-Fire Exam Points ⚡

1. **LRS ↔ GRS, GRS ↔ RA-GRS** = change directly in portal, no downtime
2. **LRS → ZRS** = requires **Microsoft support request** (live migration) — cannot do in portal
3. Live migration to ZRS = **no downtime**, **no data copy**, endpoint stays the same, **free**
4. During live migration: storage account is **fully accessible** — no restrictions
5. Alternative to live migration: **manual migration** — create new account + AzCopy + switch apps
6. **GPv1 → GPv2** upgrade = **free, instant, no downtime, irreversible** (cannot downgrade)
7. **Standard ↔ Premium** = CANNOT convert in-place — must create new account + copy data
8. **Cross-region** storage move = NOT supported directly — copy data to new account in target region
9. **Cross-subscription / RG** move = **metadata-only** move, no data copy, no downtime, keys/endpoint unchanged
10. **Resource locks** must be **removed** before moving storage account (sub or RG move)
11. **Failover** = geo-redundant accounts only (GRS/RA-GRS/GZRS/RA-GZRS)
12. After failover: account becomes **LRS**, keys **regenerated**, possible **data loss** (~15 min RPO)
13. Check **Last Sync Time** before failover to estimate data loss
14. **AzCopy** = preferred online migration tool. `azcopy copy` = full copy, `azcopy sync` = incremental
15. AzCopy supports **server-to-server** copy — data flows directly between Azure accounts
16. **Azure Data Box** = physical device from Microsoft (100 TB). Your disks = Import/Export service
17. Object replication = async, block blobs only, versioning on both + change feed on source
18. **Large file shares** enablement is **irreversible** — disables GRS (LRS/ZRS only after)
19. Premium storage accounts only support **LRS and ZRS** — no GRS options
20. All live migrations and upgrades are **free** — you pay the new ongoing redundancy rate

---

## 18. Step-by-Step Configuration Mind Maps 🗺️

---

### 18.1 Change Storage Redundancy (Portal-Supported)

> **Portal:** `Storage Account → Settings → Redundancy`

```
Change Redundancy (Portal)
│
├── Portal: Storage Account → Settings → Redundancy
│
├── Supported Direct Changes:
│   ├── LRS → GRS / RA-GRS
│   ├── GRS → LRS / RA-GRS
│   ├── RA-GRS → LRS / GRS
│   ├── ZRS → GZRS / RA-GZRS
│   ├── GZRS → ZRS / RA-GZRS
│   └── RA-GZRS → ZRS / GZRS
│
├── Steps:
│   ├── Select new redundancy from dropdown
│   ├── Review change summary
│   │   ⚠️ GRS → LRS = STOPS geo-replication (secondary data deleted)
│   │   ⚠️ LRS → GRS = async replication begins (takes time)
│   └── Save
│
├── Timing:
│   ├── GRS ↔ RA-GRS: Near instant (just enables/disables read on secondary)
│   ├── LRS → GRS: Background replication (hours to days depending on data size)
│   └── GRS → LRS: Instant (stops replication)
│
├── RBAC: Storage Account Contributor or Contributor
│
├── NOT Available in Portal:
│   ├── LRS → ZRS: ❌ Requires support request
│   ├── GRS → ZRS: ❌ Must first change to LRS, then request ZRS
│   ├── LRS → GZRS: ❌ Requires support request
│   └── RA-GRS → RA-GZRS: ❌ Requires support request
│
└── ⚠️ Notes
    ├── Premium accounts: LRS and ZRS only (no GRS option visible)
    ├── Large file shares enabled: LRS and ZRS only (no GRS option)
    ├── Changing redundancy does NOT change endpoint or keys
    └── Ongoing cost changes to new redundancy rate
```

---

### 18.2 Live Migration to ZRS (Support Request)

> **Portal:** `Help + support → New support request`

```
Live Migration: LRS → ZRS
│
├── Prerequisites
│   ├── Storage account in a region that supports ZRS
│   │   ⚠️ Not all regions support ZRS — check availability
│   ├── Account type: GPv2 (Standard) or FileStorage/BlockBlobStorage (Premium)
│   │   ⚠️ GPv1 must be upgraded to GPv2 first
│   ├── No: immutable storage policies, point-in-time restore,
│   │   versioning with certain features, or NFS file shares
│   └── RBAC: Account owner (to submit support request)
│
├── Step 1: Submit Support Request
│   │   Portal: Help + support → + New support request
│   ├── Issue type: Service and subscription limits (quotas)
│   ├── Subscription: Select
│   ├── Quota type: Storage: Account Limits
│   ├── Description: "Request live migration from LRS to ZRS for account <name>"
│   ├── Include: Storage account name, region, current redundancy, target redundancy
│   └── Submit
│
├── Step 2: Microsoft Reviews and Schedules
│   ├── Microsoft validates account eligibility
│   ├── Schedules migration (may take days to weeks to start)
│   └── ⚠️ No guaranteed timeline — plan accordingly
│
├── Step 3: Migration Executes
│   ├── Azure moves data across 3 Availability Zones
│   ├── Storage account remains FULLY ACCESSIBLE during migration
│   ├── No downtime, no endpoint change, no key change
│   ├── Portal shows migration progress (Redundancy section)
│   └── ⚠️ May take hours to days depending on data size
│
├── Step 4: Migration Complete
│   ├── Redundancy shows: ZRS
│   ├── All data now in 3 AZs
│   └── Notification received
│
├── Alternative: Manual Migration
│   ├── Create new account with ZRS
│   ├── Copy data: azcopy copy "source?SAS" "dest?SAS" --recursive
│   ├── Update application connection strings
│   ├── Verify data integrity
│   └── Delete old LRS account
│
└── ⚠️ Notes
    ├── Live migration = FREE (no migration fee)
    ├── Ongoing cost changes to ZRS rate (higher than LRS)
    ├── GPv1 must upgrade to GPv2 before requesting ZRS migration
    ├── Can go LRS → ZRS → GZRS (ZRS first, then add geo in portal)
    └── Cannot go directly from GRS → ZRS — must go GRS → LRS → ZRS
```

---

### 18.3 Migrate Storage Account Cross-Region

> **Method:** Create new account + Copy data with AzCopy

```
Cross-Region Storage Migration
│
├── Prerequisites
│   ├── Target region has desired storage features available
│   ├── AzCopy installed (or use Azure Cloud Shell)
│   ├── SAS tokens for both source and destination (or Azure AD)
│   └── RBAC: Storage Account Contributor (create new) + Blob Data Contributor (copy data)
│
├── Step 1: Create Destination Storage Account
│   │   Portal: Home → Storage accounts → + Create
│   ├── Region: Target region
│   ├── Same configuration: Kind, Performance, Redundancy, settings
│   │   ⚠️ Match all settings to avoid feature gaps
│   └── Create
│
├── Step 2: Recreate Containers/Shares
│   ├── Create matching containers in destination
│   ├── Set access levels, policies, metadata
│   └── CLI: az storage container create --account-name <dest> -n <container>
│
├── Step 3: Copy Data with AzCopy
│   │
│   ├── Blob Data (server-to-server):
│   │   azcopy copy \
│   │     "https://source.blob.core.windows.net/?<SAS>" \
│   │     "https://dest.blob.core.windows.net/?<SAS>" \
│   │     --recursive
│   │   ⚠️ Server-to-server: data flows within Azure backbone
│   │
│   ├── File Share Data:
│   │   azcopy copy \
│   │     "https://source.file.core.windows.net/<share>?<SAS>" \
│   │     "https://dest.file.core.windows.net/<share>?<SAS>" \
│   │     --recursive
│   │
│   ├── Monitor: azcopy jobs list
│   └── Retry failed: azcopy jobs resume <job-id>
│
├── Step 4: Validate Data
│   ├── Compare blob/file counts and sizes
│   ├── Check: azcopy jobs show <id> → verify 0 failures
│   └── Spot-check critical files
│
├── Step 5: Cutover
│   ├── Final sync: azcopy sync (catches any changes since initial copy)
│   ├── Update application connection strings to destination account
│   ├── Update DNS, SAS tokens, connection strings
│   ├── Test application functionality
│   └── ⚠️ Brief cutover window — minimize with final azcopy sync
│
├── Step 6: Cleanup
│   ├── Monitor destination for a period
│   ├── Delete source account when confident
│   └── ⚠️ Keep source as backup until fully validated
│
└── ⚠️ Notes
    ├── Cross-region egress charges apply (data leaving source region)
    ├── No direct "move" — must copy + switch
    ├── Preserve tier settings, access policies, RBAC on destination
    ├── For very large data (>10 TB): consider Data Box
    └── For ongoing replication (Blobs): use Object Replication first, then cutover
```

---

### 18.4 Move Storage Account Between Subscriptions/RGs

> **Portal:** `Storage Account → Overview → Move`

```
Move Storage Account (Subscription/RG)
│
├── Prerequisites
│   ├── Source and target subscription/RG exist
│   ├── RBAC: Contributor on BOTH source AND target scope
│   ├── No resource locks on storage account or resource group
│   │   ⚠️ Remove ALL locks (ReadOnly + Delete) before move
│   └── Target sub must have the same resource provider registered
│
├── Step 1: Remove Locks (if any)
│   │   Portal: Storage Account → Settings → Locks → Delete locks
│   │   Also check: Resource Group → Locks
│   └── ⚠️ Locks on RG level also block resource move
│
├── Step 2: Initiate Move
│   │   Portal: Storage Account → Overview →
│   │   Resource group (click) → Move →
│   ├── Move to another resource group
│   │   ├── Select target RG (or create new)
│   │   ├── Select resources to move
│   │   ├── ✅ "I understand..." checkbox
│   │   └── Move
│   │
│   └── Move to another subscription
│       ├── Select target subscription
│       ├── Select target RG (or create new)
│       ├── Select resources to move
│       ├── ✅ "I understand..." checkbox
│       └── Move
│
├── Step 3: Verify
│   ├── Azure validates the move (may take minutes)
│   ├── Check: target RG/subscription → storage account appears
│   ├── Keys: Unchanged ✅
│   ├── Endpoint: Unchanged ✅
│   └── Connection strings: Still work ✅ (unless using resource ID)
│
├── Post-Move:
│   ├── Re-apply locks if needed
│   ├── Update any references using resource ID (it changes)
│   ├── Verify Azure Backup, File Sync, Policy assignments
│   └── Update RBAC if role assignments were at RG level
│
└── ⚠️ Notes
    ├── Metadata-only move — NO data is copied
    ├── NO downtime
    ├── Resource ID changes (subscription/RG in the path changes)
    ├── Both source and target are locked during move (brief, few minutes)
    ├── Cannot move cross-region this way (must copy data instead)
    └── Dependent resources may also need to move together
```

---

### 18.5 Initiate Storage Account Failover

> **Portal:** `Storage Account → Settings → Redundancy → Prepare for failover`

```
Storage Account Failover
│
├── Prerequisites
│   ├── Redundancy: GRS / RA-GRS / GZRS / RA-GZRS
│   │   ⚠️ LRS and ZRS do NOT support failover
│   ├── Understand data loss risk (RPO ~15 minutes)
│   └── RBAC: Storage Account Contributor or Contributor
│
├── Step 1: Check Last Sync Time
│   │   Portal: Storage Account → Settings → Redundancy
│   ├── "Last sync time" = most recent data guaranteed in secondary
│   ├── Any data written after this time may be LOST
│   └── ⚠️ Async replication — typical lag = ~15 minutes
│
├── Step 2: Initiate Failover
│   │   Portal: Storage Account → Settings → Redundancy →
│   │   "Prepare for failover" link (bottom of page)
│   ├── Confirmation: "Yes, fail over"
│   │   ⚠️ This promotes secondary to primary — CANNOT undo easily
│   │
│   │   CLI: az storage account failover -g <rg> -n <name>
│   │
│   └── ⚠️ Failover may take up to 1 hour
│
├── Step 3: Post-Failover State
│   ├── Secondary region → becomes new Primary
│   ├── Redundancy → becomes **LRS** (geo-replication stopped)
│   ├── Access keys → **REGENERATED** (must update apps)
│   ├── Endpoint → Same DNS name (re-pointed)
│   ├── Secondary endpoint → ❌ No longer available
│   └── Data → Available from new primary (possible last-minute loss)
│
├── Step 4: Reconfigure (Post-Failover)
│   ├── Update connection strings with new keys
│   ├── Reconfigure geo-replication:
│   │   Portal: Redundancy → Change from LRS to GRS/RA-GRS → Save
│   │   ⚠️ New secondary region may differ from original primary
│   ├── Re-verify Azure Backup, File Sync, dependent services
│   └── Test application end-to-end
│
└── ⚠️ Exam-Critical Notes
    ├── After failover = LRS. Must manually reconfigure GRS
    ├── Keys REGENERATED — old keys stop working
    ├── Data loss possible — check Last Sync Time before failover
    ├── Failback = initiate another failover (LRS → GRS first, then failover again)
    ├── Only for GRS/RA-GRS/GZRS/RA-GZRS — not LRS/ZRS
    └── RA-GRS: can READ secondary before failover (no failover needed for reads)
```

---

### 18.6 Migrate Data Using Azure Data Box

> **Portal:** `Home → Azure Data Box → + Create order`

```
Migrate Data Using Azure Data Box
│
├── Prerequisites
│   ├── Azure subscription
│   ├── Storage account in target region
│   ├── Physical address for device shipment
│   └── RBAC: Contributor on subscription
│
├── Step 1: Create Data Box Order
│   │   Portal: Home → Azure Data Box → + Create order
│   ├── Transfer type: Import to Azure / Export from Azure
│   ├── Source country/region
│   ├── Device type:
│   │   ├── Data Box Disk: up to 35 TB (5 SSDs)
│   │   ├── Data Box: up to 100 TB (1 device)
│   │   └── Data Box Heavy: up to 1 PB (1 device)
│   │   ⚠️ Choose based on data volume
│   ├── Subscription, Resource Group
│   ├── Destination: Storage account(s) → Blob / File / Managed Disks
│   ├── Shipping address
│   └── Create
│
├── Step 2: Receive Device
│   ├── Microsoft ships device to you
│   ├── Unbox, cable, and power on
│   └── Connect to local network
│
├── Step 3: Copy Data to Device
│   ├── Connect via SMB/NFS (credentials on the device/portal)
│   ├── Copy data to appropriate shares:
│   │   ├── Block blob share → for block blobs
│   │   ├── Page blob share → for page blobs / VHDs
│   │   └── Azure Files share → for file shares
│   ├── Use Robocopy (Windows) or rsync (Linux) for bulk copy
│   ├── Validate data: checksums verified
│   └── ⚠️ Do NOT rename or move the pre-created share folders
│
├── Step 4: Ship Device to Azure
│   ├── Prepare to ship (Portal: order → Prepare to ship)
│   ├── Device generates manifest + checksum
│   ├── Power off, disconnect cables
│   ├── Schedule pickup with carrier
│   └── Update tracking number in portal
│
├── Step 5: Azure Ingests Data
│   ├── Device received at Azure datacenter
│   ├── Data uploaded to storage account(s)
│   ├── Device securely wiped (NIST 800-88)
│   └── Portal: order status → Data copy complete
│
├── Step 6: Verify
│   ├── Check storage account for uploaded data
│   ├── Verify file counts and sizes
│   └── Copy log files available in storage account
│
└── ⚠️ Notes
    ├── Encrypted with AES-256 (BitLocker)
    ├── Import: data goes TO Azure. Export: data comes FROM Azure
    ├── Pricing: per-device service fee + shipping
    ├── Ingress (upload) is FREE. Export incurs egress charges
    ├── Data Box device wiped after upload (NIST standard)
    └── Ideal for: no reliable network, >10 TB, initial seeding, datacenter migration
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
