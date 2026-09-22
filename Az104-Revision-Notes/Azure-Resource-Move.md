<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Resource Move & Limitations — AZ-104 Revision Notes

---

## 1. What is Azure Resource Move?

- Ability to **move Azure resources** between Resource Groups, Subscriptions, or Regions
- Moves the **resource's logical placement** — NOT the physical region (for RG/Sub moves)
- Three types of move: **Between RGs**, **Between Subscriptions**, **Between Regions**
- Not all resources support all move types — must validate before moving
- During move, both source and destination groups experience a **brief read-only lock**

> ⚠️ **EXAM TIP:** Moving a resource between RGs or subscriptions does **NOT change the resource's region**. Moving between regions is a completely different operation that involves **recreating** the resource.

---

## 2. Types of Resource Move

| Move Type | What Changes | Region Change? | Downtime? |
|---|---|---|---|
| **Between Resource Groups** (same sub) | RG membership | ❌ No | Minimal (brief lock) |
| **Between Subscriptions** (same tenant) | Subscription + RG | ❌ No | Minimal (brief lock) |
| **Between Tenants** (directory change) | Tenant + all IAM | ❌ No | ⚠️ Yes (RBAC disruption) |
| **Between Regions** | Physical location | ✅ Yes | ⚠️ Yes (resource recreated) |

---

## 3. Move Between Resource Groups — Rules

### What Happens

- Resource moves from **source RG** to **destination RG** within **same subscription**
- Resource ID changes (new RG name in path)
- Resource region stays the **same**
- Dependent resources may need to move together

### Key Rules

| Rule | Details |
|---|---|
| **Source RG** | Locked (read-only) during move |
| **Destination RG** | Locked (read-only) during move |
| **Lock duration** | Up to **4 hours** max; usually minutes |
| **Write operations blocked** | Cannot create/update/delete in locked RGs |
| **Read operations** | ✅ Continue working |
| **Resource region** | Does NOT change |
| **Resource tags** | Move with the resource |
| **Resource locks** | Must be **removed** before move |
| **RBAC** | **Preserved** — no change |

### Portal Path

```
Resource → Overview → Move (toolbar) →
  Move to another resource group →
  Select destination RG (or create new) →
  Select resources to move →
  Acknowledge check → OK
```

> ⚠️ **EXAM TIP:** Both source AND destination RGs are **locked** (read-only) during the move. You CANNOT create, update, or delete resources in either RG during the move. The lock can last up to **4 hours**.

---

## 4. Move Between Subscriptions — Rules

### Prerequisites

| Requirement | Details |
|---|---|
| **Same Entra ID tenant** | Both subscriptions must be in the same tenant |
| **Both subs Active** | Both source and destination must be Active state |
| **Resource provider registered** | Must be registered in destination subscription |
| **Sufficient quota** | Destination sub must have enough quota for the resource |
| **Resource locks** | Must remove locks from source resources |
| **Permissions** | Need move permissions on both source AND destination |
| **Dependent resources** | Some must move together (VNet + subnets) |
| **Region** | Does NOT change during cross-sub move |

### What Changes

| Item | Impact |
|---|---|
| **Resource ID** | Changes (new subscription ID in path) |
| **Region** | ❌ No change |
| **RBAC on resource** | ❌ **Cleared** at resource level — inherited RBAC from new RG/Sub applies |
| **Resource tags** | ✅ Preserved |
| **Linked resources** | Must verify dependencies |
| **Billing** | Moves to destination subscription's billing |
| **Activity Log** | Move operation logged in both source and destination |

### Portal Path

```
Resource → Overview → Move (toolbar) →
  Move to another subscription →
  Select destination subscription →
  Select destination RG (or create new) →
  Select resources → Validate → OK
```

> ⚠️ **EXAM TIP:** Cross-subscription move requires **same tenant**. RBAC at resource level is **cleared** — the resource inherits RBAC from the new RG/subscription. Resource providers must be **registered** in the destination subscription. Remove **all locks** before moving.

---

## 5. Move Between Regions

- **Completely different** from RG/Sub moves
- Resource is **recreated** in the new region (not physically moved)
- Uses **Azure Resource Mover** service or manual re-deployment
- Involves **downtime** for most resources
- Not all resources support cross-region move

### Azure Resource Mover

| Feature | Details |
|---|---|
| **What** | Managed service to move resources between Azure regions |
| **Supports** | VMs, VNets, NSGs, Availability Sets, SQL databases, and more |
| **Process** | Prepare → Initiate move → Commit → Clean up source |
| **Dependencies** | Automatically identifies dependent resources |
| **Validation** | Pre-move validation checks |

### Portal Path — Azure Resource Mover

```
Search: "Azure Resource Mover" →
  + Create → Source region → Target region →
  + Add resources → Select resources →
  Validate dependencies → Prepare → Initiate move →
  Commit → Delete source resources (optional)
```

> ⚠️ **EXAM TIP:** Region move = resource is **recreated**, not physically relocated. Uses **Azure Resource Mover**. Involves **downtime**. This is fundamentally different from RG/subscription moves which are instant metadata changes.

---

## 6. Complete Resource Move Support Matrix (AZ-104 Resources)

### Compute Resources

| Resource | Between RGs | Between Subs | Between Regions | Key Limitations |
|---|---|---|---|---|
| **Virtual Machines** | ✅ | ✅ | ✅ (Resource Mover) | All dependent resources must move together |
| **VM Scale Sets (VMSS)** | ✅ | ✅ | ✅ (Resource Mover) | With all dependencies |
| **Availability Sets** | ✅ | ✅ | ✅ (Resource Mover) | Must move VMs separately |
| **Managed Disks** | ✅ | ✅ | ✅ (copy/snapshot) | If attached, move with VM |
| **Unmanaged Disks** | ❌ | ❌ | ❌ | Must convert to managed first |
| **Snapshots** | ✅ | ✅ | ⚠️ Copy only | Copy to new region, not move |
| **Images** | ✅ | ✅ | ⚠️ Copy only | Copy, not move |
| **Dedicated Hosts** | ❌ | ❌ | ❌ | Cannot be moved |
| **Proximity Placement Groups** | ✅ | ✅ | ❌ | Cannot cross regions |

### VM Move — Specific Requirements

| Scenario | Move Supported? | Notes |
|---|---|---|
| **VM in Availability Set** | ✅ | Move all VMs in set together |
| **VM with Managed Disks** | ✅ | Disks move with VM |
| **VM with Azure Backup** | ⚠️ Conditional | Must stop backup or delete recovery points |
| **VM with Azure Site Recovery** | ❌ | Cannot move while ASR is enabled |
| **VM in VNet** | ✅ | Move VNet + all subnets together |
| **VM with Public IP** | ✅ | Basic SKU only; Standard = depends |
| **VM with Key Vault** | ✅ | KV must be in same target or accessible |
| **Spot VM** | ✅ | Moves like regular VM |
| **VM with extensions** | ✅ | Extensions move with VM |
| **VM with encryption (ADE)** | ⚠️ | Complex; Key Vault access must be maintained |

> ⚠️ **EXAM TIP:** VMs with **Azure Backup enabled** cannot be directly moved — you must **stop backup** first (or keep data). VMs with **Azure Site Recovery** replication **cannot** be moved. These are frequent exam scenarios.

### Networking Resources

| Resource | Between RGs | Between Subs | Between Regions | Key Limitations |
|---|---|---|---|---|
| **Virtual Network (VNet)** | ✅ | ✅ | ✅ (Resource Mover) | ALL dependent resources must move together |
| **Subnet** | N/A (part of VNet) | N/A | N/A | Moves with VNet |
| **Network Security Group (NSG)** | ✅ | ✅ | ✅ (Resource Mover) | Moves independently |
| **Network Interface (NIC)** | ✅ | ✅ | ⚠️ Recreate | Move with VM |
| **Public IP Address** | ✅ | ✅ | ❌ | **Basic SKU**: can move; **Standard SKU**: can move |
| **Load Balancer** | ✅ (Basic) | ✅ (Basic) | ❌ | **Standard SKU**: ❌ Cannot move across sub |
| **Application Gateway** | ❌ | ❌ | ❌ | Cannot be moved at all |
| **Azure Firewall** | ❌ | ❌ | ❌ | Cannot be moved — must delete and recreate |
| **VPN Gateway** | ❌ | ❌ | ❌ | Cannot be moved |
| **ExpressRoute Circuit** | ❌ | ❌ | ❌ | Cannot be moved |
| **ExpressRoute Gateway** | ❌ | ❌ | ❌ | Cannot be moved |
| **Azure Bastion** | ❌ | ❌ | ❌ | Cannot be moved |
| **Traffic Manager Profile** | ✅ | ✅ | N/A (global) | Global service — no region |
| **Azure DNS Zone** | ✅ | ✅ | N/A (global) | Global service |
| **Private DNS Zone** | ✅ | ✅ | N/A (global) | Must unlink VNet links if cross-sub |
| **Private Endpoint** | ❌ | ❌ | ❌ | Cannot be moved |
| **NAT Gateway** | ✅ | ✅ | ❌ | — |
| **Route Table (UDR)** | ✅ | ✅ | ❌ | — |
| **DDoS Protection Plan** | ✅ | ✅ | ❌ | — |

### VNet Move — Critical Dependencies

```
Virtual Network Move
│
├── MUST move together:
│   ├── All Subnets (auto-included)
│   ├── All VNet peerings (must be deleted and recreated)
│   │   └── ⚠️ Peerings are DELETED during cross-sub move
│   ├── Network Security Groups (if associated)
│   ├── Route Tables (if associated)
│   └── NAT Gateways (if associated)
│
├── MUST deal with before move:
│   ├── VNet Gateway → ❌ Cannot move with VNet
│   │   └── Must delete gateway → move VNet → recreate gateway
│   ├── VNet Peerings → Must delete before move, recreate after
│   └── Service Endpoints → Reconfigure in new context
│
└── ⚠️ Cannot move if:
    ├── VNet has a VPN/ER Gateway — delete gateway first
    ├── VNet has peering — delete peering first
    └── VNet has Private Endpoints — cannot move
```

> ⚠️ **EXAM TIP:** VNet move requires **deleting all peerings** first, then recreating after move. **VPN/ExpressRoute Gateways** must be deleted before moving the VNet. **Private Endpoints** block the move entirely. This is heavily tested.

### Storage Resources

| Resource | Between RGs | Between Subs | Between Regions | Key Limitations |
|---|---|---|---|---|
| **Storage Account (GPv2/GPv1)** | ✅ | ✅ | ❌ (must copy data) | Move is metadata only; data stays |
| **Blob Storage** | Part of SA | Part of SA | ❌ | Moves with Storage Account |
| **Azure Files** | Part of SA | Part of SA | ❌ | Moves with Storage Account |
| **Storage Account (Blob)** | ✅ | ✅ | ❌ | Same as GPv2 |
| **Azure Data Lake Storage Gen2** | ✅ | ✅ | ❌ | Hierarchical namespace |
| **Azure NetApp Files** | ❌ | ❌ | ❌ | Cannot move |
| **Managed Disks** | ✅ | ✅ | ⚠️ Copy | Copy snapshot to new region |

### Storage Move Notes

- Storage Account move (RG/Sub) = **instant** — metadata change only
- **Data stays in the same region** — only the billing/management scope changes
- Cross-region = must **copy data** (AzCopy, Storage replication, etc.)
- **Storage Account name** is globally unique — name preserved on move
- If SA has **private endpoints**, it **cannot** be moved across subscriptions

> ⚠️ **EXAM TIP:** Storage Account move between RG/Sub = fast metadata change. Data does NOT physically move. Cross-region Storage = must **copy data manually** (AzCopy). SA with **private endpoints** = cannot cross-sub move.

### Database Resources

| Resource | Between RGs | Between Subs | Between Regions | Key Limitations |
|---|---|---|---|---|
| **Azure SQL Database** | ✅ | ✅ | ⚠️ (geo-replication) | Must move server + all DBs together |
| **Azure SQL Server (logical)** | ✅ | ✅ | ❌ | All DBs on server must move together |
| **Azure SQL Managed Instance** | ❌ | ❌ | ❌ | Cannot be moved |
| **Azure SQL Elastic Pool** | ✅ | ✅ | ❌ | Moves with server |
| **Azure Database for MySQL** | ✅ | ✅ | ❌ | — |
| **Azure Database for PostgreSQL** | ✅ | ✅ | ❌ | — |
| **Azure Cosmos DB** | ✅ | ✅ | ❌ | — |
| **Azure Cache for Redis** | ✅ | ✅ | ❌ | — |

### SQL Move — Critical Rules

- **SQL Server + ALL databases** must move **together** (cannot split)
- SQL Server across subscriptions = **same tenant required**
- **Elastic Pools** move with the server
- **SQL Managed Instance** = ❌ **cannot be moved at all**

> ⚠️ **EXAM TIP:** Azure SQL Server + ALL its databases must move **together** — cannot move individual databases separately. **SQL Managed Instance** cannot be moved. These are commonly tested scenarios.

### Web / App Service Resources

| Resource | Between RGs | Between Subs | Between Regions | Key Limitations |
|---|---|---|---|---|
| **App Service Plan** | ✅ | ⚠️ Conditional | ❌ | Complex rules — see below |
| **Web App** | ✅ | ⚠️ Conditional | ❌ | Must move with App Service Plan |
| **Function App** | ✅ | ⚠️ Conditional | ❌ | Must move with App Service Plan |
| **App Service Certificate** | ✅ | ⚠️ | ❌ | Limitations with Key Vault bindings |
| **App Service Domain** | ✅ | ❌ | N/A (global) | Cannot cross-sub |
| **API Management** | ✅ | ✅ | ❌ | — |

### App Service Move Rules (Heavily Tested)

| Rule | Detail |
|---|---|
| **Same region required** | Source and destination must be in same region for cross-sub |
| **All apps in plan** | All apps in the App Service Plan must move together |
| **All plans in RG** | ALL App Service Plans in source RG must move together |
| **ASE** | Apps in ASE cannot be moved across subscriptions |
| **Certificates** | App Service Managed Certificates cannot be moved |
| **Slots** | Deployment slots move with the app |
| **Custom domains** | TXT verification may need to be re-done |

```
App Service Move Checklist
│
├── ✅ Must be met:
│   ├── Same region (source and destination)
│   ├── ALL apps in the plan move together
│   ├── ALL plans in the source RG move together
│   ├── Web App + Plan in same destination RG
│   └── Destination RG has no existing App Service resources
│       from different region
│
├── ❌ Cannot move if:
│   ├── App is in App Service Environment (ASE) — cross-sub
│   ├── App Service Managed Certificate bound
│   └── Custom domain with TXT record not matching
│
└── ⚠️ After move:
    ├── Verify custom domain bindings
    ├── Verify SSL certificate bindings
    └── Update any CI/CD pipelines with new resource IDs
```

> ⚠️ **EXAM TIP:** App Service cross-subscription move requires: **same region**, **all apps in plan move together**, **all plans in source RG move together**, destination RG **cannot have App Service resources from a different region**. This is one of the most complex move scenarios and is heavily tested.

### Monitoring & Management Resources

| Resource | Between RGs | Between Subs | Between Regions | Key Limitations |
|---|---|---|---|---|
| **Log Analytics Workspace** | ✅ | ✅ | ❌ | Linked Automation Account must be in same sub |
| **Application Insights** | ✅ | ✅ | ❌ | — |
| **Automation Account** | ✅ | ✅ | ❌ | Unlink from Log Analytics first if cross-sub |
| **Recovery Services Vault** | ❌ | ❌ | ❌ | **Cannot move** (with active backups) |
| **Key Vault** | ✅ | ✅ | ❌ | Soft-delete must be considered |
| **Managed Identity (user-assigned)** | ✅ | ✅ | ❌ | — |
| **Alert Rules** | ✅ | ✅ | ❌ | — |

### Recovery Services Vault — Critical

- **Cannot be moved** if it contains active backup items
- To move: stop all backup jobs → delete backup data → delete vault → recreate
- ASR (Site Recovery) vaults also cannot be moved with active replications

> ⚠️ **EXAM TIP:** **Recovery Services Vault** with active backups = **CANNOT** be moved. Must stop and delete all backups first. This is one of the most tested "cannot move" scenarios on AZ-104.

---

## 7. Resources That CANNOT Be Moved (Master List)

| Resource | Between RGs | Between Subs | Reason |
|---|---|---|---|
| **Application Gateway** | ❌ | ❌ | Architecture limitation |
| **Azure Firewall** | ❌ | ❌ | Must delete/recreate |
| **Azure Bastion** | ❌ | ❌ | Tied to VNet/subnet |
| **VPN Gateway** | ❌ | ❌ | Must delete/recreate |
| **ExpressRoute Circuit** | ❌ | ❌ | Cannot be moved |
| **ExpressRoute Gateway** | ❌ | ❌ | Cannot be moved |
| **Private Endpoint** | ❌ | ❌ | Tied to specific resource + VNet |
| **Azure SQL Managed Instance** | ❌ | ❌ | Cannot be moved |
| **Dedicated Host** | ❌ | ❌ | Cannot be moved |
| **Recovery Services Vault** (active) | ❌ | ❌ | Active backups prevent move |
| **Azure NetApp Files** | ❌ | ❌ | Cannot be moved |
| **Unmanaged Disks** | ❌ | ❌ | Convert to managed first |
| **Azure Kubernetes Service (AKS)** | ✅ RG | ❌ Sub | Limited sub move |
| **Azure AD Domain Services** | ❌ | ❌ | Cannot be moved |

> ⚠️ **EXAM TIP:** Memorize the "cannot move" resources: **Application Gateway, Azure Firewall, Bastion, VPN Gateway, ExpressRoute, Private Endpoints, SQL Managed Instance, Recovery Services Vault (with backups)**. These frequently appear as exam distractors.

---

## 8. General Move Prerequisites & Constraints

| Prerequisite | Details |
|---|---|
| **Resource locks** | ❌ Must remove ALL locks (ReadOnly AND Delete) before move |
| **Subscription state** | Both source and destination subs must be **Active** |
| **Same tenant** | Cross-sub moves require same Entra ID tenant |
| **Resource provider** | Must be registered in destination subscription |
| **Quotas & limits** | Destination must have sufficient quota |
| **Dependent resources** | Some resources must move together |
| **No active operations** | Resource must not have pending async operations |
| **Validation** | Always validate before moving (`az resource move --validate`) |
| **Role assignments** | Cleared at resource level on cross-sub move |
| **Tags** | Preserved during all move types |

---

## 9. RBAC for Resource Moves

### Required Permissions

| Permission | Where | Description |
|---|---|---|
| `Microsoft.Resources/subscriptions/resourceGroups/moveResources/action` | Source RG | Permission to initiate move |
| `Microsoft.Resources/subscriptions/resourceGroups/write` | Destination RG | Permission to write to target |
| `Microsoft.Resources/subscriptions/resourceGroups/validateMoveResources/action` | Source | Permission to validate move |

### Simplified Role Requirements

| Action | Minimum Role |
|---|---|
| Move within same subscription | **Contributor** on both source and destination RGs |
| Move across subscriptions | **Contributor** on both source and destination subs |
| Validate a move | **Contributor** on source RG |
| Remove locks before move | **Owner** or **User Access Administrator** |

> ⚠️ **EXAM TIP:** Need **Contributor** (or higher) on **BOTH** source and destination. If resource locks exist, need **Owner** to remove them first. Cross-sub also requires the resource provider to be registered in the destination.

---

## 10. CLI & PowerShell Commands

### Azure CLI

```bash
# Validate move (dry run — ALWAYS do this first)
az resource move \
  --destination-group "destRG" \
  --ids "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/myVM" \
  --validate

# Move resource to another RG (same subscription)
az resource move \
  --destination-group "destRG" \
  --ids <space-separated-resource-ids>

# Move resource to another subscription
az resource move \
  --destination-group "destRG" \
  --destination-subscription-id <dest-sub-id> \
  --ids <resource-id>

# Move multiple resources at once
az resource move \
  --destination-group "destRG" \
  --ids <id1> <id2> <id3>

# Check resource provider registration
az provider show --namespace Microsoft.Compute --query "registrationState"

# Register resource provider in destination sub
az provider register --namespace Microsoft.Compute

# List resource locks (check before move)
az lock list --resource-group "sourceRG"

# Delete a lock
az lock delete --name "myLock" --resource-group "sourceRG"
```

### Azure PowerShell

```powershell
# Validate move
$resources = @(
  "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/myVM"
)
Move-AzResource -DestinationResourceGroupName "destRG" `
  -ResourceId $resources -WhatIf   # -WhatIf = validate

# Move to another RG
Move-AzResource -DestinationResourceGroupName "destRG" `
  -ResourceId $resources

# Move to another subscription
Move-AzResource -DestinationSubscriptionId <dest-sub-id> `
  -DestinationResourceGroupName "destRG" `
  -ResourceId $resources

# Check locks
Get-AzResourceLock -ResourceGroupName "sourceRG"

# Remove lock
Remove-AzResourceLock -LockName "myLock" -ResourceGroupName "sourceRG" -Force

# Register provider
Register-AzResourceProvider -ProviderNamespace "Microsoft.Compute"
```

> ⚠️ **EXAM TIP:** Always **validate** before moving (`--validate` in CLI, `-WhatIf` in PowerShell). Move uses **resource IDs** (not names). Multiple resources can be moved in one operation. Check `az provider show` to verify registration.

---

## 11. Monitoring & Activity Log

### Move Operations in Activity Log

| Event | Operation |
|---|---|
| **Move started** | `Microsoft.Resources/subscriptions/resourceGroups/moveResources/action` |
| **Move validated** | `Microsoft.Resources/subscriptions/resourceGroups/validateMoveResources/action` |
| **Move completed** | Success/failure logged with details |

### Portal Path — View Move Activity

```
Azure Monitor → Activity Log →
  Filter: Operation = "Move resources" →
  View: Who initiated, when, status, affected resources
```

---

## 12. Pricing Key Points

| Item | Cost |
|---|---|
| **Resource move (RG/Sub)** | **FREE** — no charge for the move operation |
| **Region move (Resource Mover)** | **FREE** service — but you pay for resources in new region |
| **Data transfer** | ⚠️ Egress charges may apply for cross-region data copy |
| **Duplicate resources** | During region move, you may have resources in BOTH regions temporarily |
| **Storage copy** | AzCopy transfer = egress charges for cross-region |

> ⚠️ **EXAM TIP:** RG/Subscription moves are **FREE**. Region moves = free service but **egress charges** apply for data transfer. During region moves, resources exist in **both** regions temporarily = double cost until source is cleaned up.

---

## 13. Quick-Fire Exam Points ⚡

1. Resource move between RGs/Subs = **metadata change** — region does NOT change
2. Region move = resource is **recreated** in new region (different process)
3. During RG/Sub move, **both** source and destination RGs are **read-only locked** (up to 4 hours)
4. Cross-subscription move requires **same Entra ID tenant**
5. Both subscriptions must be **Active** state
6. **Resource provider** must be registered in destination subscription
7. **Resource locks** MUST be removed before any move
8. Tags are **preserved** during moves
9. RBAC at resource level is **cleared** on cross-subscription moves
10. Need **Contributor** on both source and destination RGs/subs
11. Always **validate** before moving (`--validate` / `-WhatIf`)
12. **VMs with Azure Backup** = must stop backup before move
13. **VMs with Site Recovery** = cannot move while replication is active
14. **VNet** move = delete peerings + delete VPN gateway + delete private endpoints first
15. **Application Gateway** = ❌ cannot be moved at all
16. **Azure Firewall** = ❌ cannot be moved — delete and recreate
17. **VPN Gateway** = ❌ cannot be moved
18. **ExpressRoute** = ❌ cannot be moved
19. **Azure Bastion** = ❌ cannot be moved
20. **Private Endpoints** = ❌ cannot be moved — block VNet moves too
21. **SQL Server + ALL databases** must move together — cannot split
22. **SQL Managed Instance** = ❌ cannot be moved
23. **Recovery Services Vault** (active backups) = ❌ cannot be moved
24. **App Service** cross-sub move: same region + all apps in plan + all plans in RG together
25. App Service destination RG must NOT have App Service resources from different region
26. **Storage Account** RG/Sub move = instant metadata change; data stays in same region
27. Storage Account with **private endpoints** = cannot cross-sub move
28. Cross-region Storage = must **copy data manually** (AzCopy)
29. **Managed Disks** = can move between RGs/subs; cross-region = copy via snapshot
30. **Unmanaged Disks** = ❌ cannot be moved — convert to managed first
31. **Log Analytics Workspace** — linked Automation Account must be in same subscription
32. **Key Vault** — can move between RGs/subs; consider soft-delete state
33. **Azure Resource Mover** = managed service for cross-region moves
34. Resource Mover flow: **Prepare → Initiate → Commit → Clean up source**
35. RG/Sub moves are **FREE**; region moves = free service but **egress charges** apply
36. **Load Balancer Standard SKU** cannot be moved cross-subscription
37. **Load Balancer Basic SKU** can be moved across RGs and subscriptions
38. Move operations logged in **Activity Log** under Administrative category
39. Multiple resources can be moved in a **single operation**
40. **App Service Environment (ASE)** apps cannot be moved cross-subscription
41. Deployment slots move **with** the Web App
42. **Cosmos DB** can be moved between RGs and subscriptions
43. **Azure Cache for Redis** can be moved between RGs and subscriptions
44. After cross-sub move, update any **CI/CD pipelines** with new resource IDs
45. **Network Interface** moves with its associated VM
46. **Public IP** can be moved between RGs and subscriptions
47. **NSG** can be moved independently between RGs and subscriptions
48. **Traffic Manager** can be moved (global service — no region constraint)
49. **DNS Zones** can be moved (global service)
50. **Dedicated Hosts** cannot be moved

---

## 14. Step-by-Step Configuration Mind Maps 🗺️

---

### 14.1 Move Resources Between Resource Groups

> **Portal:** `Resource → Overview → Move → Move to another resource group`

```
Move Between Resource Groups
│
├── Prerequisites
│   ├── Contributor+ on source RG
│   ├── Contributor+ on destination RG
│   ├── Remove ALL resource locks (ReadOnly + Delete)
│   ├── No pending async operations on resources
│   └── Dependent resources identified (move together)
│
├── Step 1: Navigate
│   └── Resource → Overview → Move → Move to another resource group
│
├── Step 2: Select Destination
│   ├── Select existing RG from dropdown
│   └── OR Create new resource group
│
├── Step 3: Select Resources
│   ├── Primary resource auto-selected
│   ├── Check dependent resources (auto-detected)
│   │   ├── VM → NIC, Disk, PIP, NSG
│   │   ├── VNet → Subnets (auto-included)
│   │   └── SQL Server → All databases
│   └── ⚠️ Some dependents MUST move together
│
├── Step 4: Validation
│   └── Azure validates move — shows errors if unsupported
│
├── Step 5: Acknowledge
│   ├── Check "I understand..." checkbox
│   │   └── Resource IDs will change
│   └── OK to start move
│
├── Step 6: Wait
│   ├── Both source + destination RGs are READ-ONLY
│   ├── Duration: seconds to minutes (max 4 hours)
│   └── Resources remain accessible during move
│
├── Post-Move
│   ├── Verify resource in new RG
│   ├── Update any scripts/templates with new resource IDs
│   ├── Verify dependent resources functioning
│   └── Tags preserved — verify
│
└── ⚠️ Key Points
    ├── Region does NOT change
    ├── Both RGs locked (read-only) during move
    ├── RBAC preserved (same subscription)
    └── Tags preserved
```

---

### 14.2 Move Resources Between Subscriptions

> **Portal:** `Resource → Overview → Move → Move to another subscription`

```
Move Between Subscriptions
│
├── Prerequisites
│   ├── Both subscriptions in SAME Entra ID tenant
│   ├── Both subscriptions in ACTIVE state
│   ├── Contributor+ on source sub/RG
│   ├── Contributor+ on destination sub/RG
│   ├── Resource provider registered in destination sub
│   │   └── Check: Subscription → Resource providers → Verify status
│   ├── Sufficient quota in destination subscription
│   ├── Remove ALL resource locks
│   ├── Stop Azure Backup on VMs (if applicable)
│   └── Delete VNet peerings (if moving VNet)
│
├── Step 1: Navigate
│   └── Resource → Overview → Move → Move to another subscription
│
├── Step 2: Select Destination
│   ├── Destination subscription (dropdown)
│   ├── Destination resource group (existing or create new)
│   └── ⚠️ Both must be same tenant
│
├── Step 3: Select Resources
│   ├── Select all resources to move
│   └── ⚠️ All dependents must be included
│       ├── SQL Server → ALL databases on that server
│       ├── App Service Plan → ALL apps in the plan
│       └── VM → NIC, Disk, PIP (recommended)
│
├── Step 4: Validation
│   ├── Azure validates compatibility
│   ├── Checks provider registration
│   ├── Checks quotas
│   └── Shows detailed errors if any
│
├── Step 5: Move
│   ├── Both source + destination RGs locked
│   └── Duration: seconds to hours depending on resources
│
├── Post-Move
│   ├── RBAC at resource level = CLEARED → reassign roles
│   ├── Update billing references
│   ├── Register resource providers if needed
│   ├── Update CI/CD pipelines (new resource IDs)
│   ├── Recreate VNet peerings (if applicable)
│   ├── Recreate VPN/ER gateways (if deleted for the move)
│   └── Verify all dependent resources
│
└── ⚠️ Key Points
    ├── Same tenant required
    ├── RBAC on resources CLEARED — must reassign
    ├── Provider must be registered in destination
    ├── Resource locks must be removed first
    └── Billing moves to destination subscription
```

---

### 14.3 Move Resources Between Regions (Azure Resource Mover)

> **Portal:** `Search → Azure Resource Mover → + Create`

```
Move Between Regions
│
├── Prerequisites
│   ├── Contributor+ on source resources
│   ├── Contributor+ on destination region/RG
│   ├── Sufficient quota in destination region
│   ├── Target region supports the resource types
│   └── ⚠️ This RECREATES resources — not a metadata change
│
├── Step 1: Navigate
│   └── Azure Resource Mover → + Create
│
├── Step 2: Configure
│   ├── Source region (where resources are now)
│   ├── Target region (where you want them)
│   └── Subscription
│
├── Step 3: Add Resources
│   ├── + Add resources → Select resource types
│   ├── Select individual resources
│   └── Resource Mover auto-detects dependencies
│
├── Step 4: Resolve Dependencies
│   ├── View dependency warnings
│   ├── Add missing dependent resources
│   └── Configure target settings (names, SKUs, etc.)
│
├── Step 5: Prepare
│   ├── Creates target resources in destination region
│   ├── Sets up replication (for supported types)
│   └── Status: "Prepare pending" → "Prepare in progress" → "Initiate move pending"
│
├── Step 6: Initiate Move
│   ├── Triggers the actual resource move/recreation
│   ├── ⚠️ DOWNTIME occurs during this phase
│   └── Status: "Move in progress" → "Commit pending"
│
├── Step 7: Commit
│   ├── Finalizes the move in destination
│   └── Status: "Commit pending" → "Delete source pending"
│
├── Step 8: Clean Up Source (optional)
│   ├── Delete source resources (to stop paying for them)
│   └── ⚠️ Double-cost until source is deleted
│
├── Required Role: Contributor on source and destination
│
└── ⚠️ Key Points
    ├── Resources are RECREATED, not physically moved
    ├── Downtime expected during initiation
    ├── Egress data transfer charges apply
    ├── Resources in BOTH regions until cleanup = double cost
    ├── Not all resources supported by Resource Mover
    └── Always test in non-production first
```

---

### 14.4 Move a VM and Its Dependencies

> **Portal:** `Virtual Machine → Overview → Move`

```
Move VM and Dependencies
│
├── Prerequisites
│   ├── Remove resource locks on VM and all dependents
│   ├── Stop Azure Backup (if enabled)
│   │   └── Recovery Services Vault → Backup items → Stop backup
│   ├── Disable Site Recovery (if enabled)
│   │   └── ⚠️ Cannot move VM with active ASR replication
│   ├── VM can remain RUNNING during RG/Sub move
│   └── Identify all dependent resources
│
├── Step 1: Identify Dependencies
│   ├── Network Interface (NIC) — required
│   ├── Managed Disks (OS + Data) — moves with VM
│   ├── Public IP Address — recommended
│   ├── NSG (if directly attached) — recommended
│   ├── Virtual Network — move if all VMs moving
│   ├── Availability Set — if VM is in one
│   └── Key Vault (if ADE encrypted) — ensure access
│
├── Step 2: Navigate
│   └── VM → Overview → Move → Select move type
│
├── Step 3: Select All Dependencies
│   ├── Checkmark each dependent resource
│   └── ⚠️ Missing a dependency = move may fail or resource orphaned
│
├── Step 4: Validate → Move
│
├── Post-Move
│   ├── Verify VM is running and accessible
│   ├── Re-enable Azure Backup in new context
│   ├── Reassign RBAC (if cross-sub)
│   └── Update DNS records if IP changed
│
└── ⚠️ Key Points
    ├── VM stays RUNNING during RG/Sub move
    ├── Azure Backup MUST be stopped first
    ├── Site Recovery MUST be disabled first
    ├── All dependents should move together
    └── VM region does NOT change (RG/Sub move)
```

---

### 14.5 Move App Service Plan and Web Apps

> **Portal:** `App Service Plan → Overview → Move`

```
Move App Service Plan + Web Apps
│
├── Prerequisites
│   ├── Source and destination in SAME REGION (for cross-sub)
│   ├── ALL apps in the plan must move together
│   ├── ALL App Service Plans in source RG must move together
│   ├── Destination RG must NOT contain App Service resources
│   │   from a DIFFERENT region
│   ├── Not using App Service Environment (ASE) for cross-sub
│   └── Remove resource locks
│
├── Step 1: Navigate
│   └── App Service Plan → Overview → Move → Select type
│
├── Step 2: Select Resources
│   ├── App Service Plan (required)
│   ├── All Web Apps in the plan (auto-selected)
│   ├── All Function Apps in the plan (auto-selected)
│   └── Deployment Slots (auto-included with apps)
│
├── Step 3: Validate
│   ├── Azure checks:
│   │   ├── Same region? ✅
│   │   ├── All apps included? ✅
│   │   ├── All plans in RG included? ✅
│   │   └── Destination RG compatible? ✅
│   └── Shows errors if constraints violated
│
├── Step 4: Move
│
├── Post-Move
│   ├── Verify custom domains still bound
│   ├── Verify SSL certificates
│   ├── Verify deployment slots
│   ├── Update CI/CD pipeline resource references
│   └── Test all apps are responding
│
└── ⚠️ Key Points
    ├── SAME REGION required for cross-sub
    ├── ALL apps in plan MUST move together
    ├── ALL plans in source RG MUST move together
    ├── Destination RG must not have conflicts
    ├── ASE apps cannot cross-sub move
    └── This is the MOST COMPLEX move scenario on the exam
```


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
