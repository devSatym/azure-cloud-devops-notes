<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Subscriptions & Management Groups — AZ-104 Revision Notes

---

## 1. What are Azure Subscriptions?

- A **logical container** for Azure resources (VMs, Storage, Networks, etc.)
- Provides a **billing boundary** — charges are aggregated per subscription
- Provides an **access control boundary** — RBAC is applied at subscription level and below
- Every Azure resource belongs to **exactly one** subscription
- A subscription is linked to **exactly one** Entra ID (Azure AD) tenant for authentication
- One Entra ID tenant can have **multiple** subscriptions; one subscription → one tenant

> ⚠️ **EXAM TIP:** A subscription = billing boundary + access control boundary. Every resource must be inside a subscription. One tenant → many subscriptions; one subscription → one tenant only.

---

## 2. What are Management Groups?

- **Containers above subscriptions** for organizing subscriptions at scale
- Apply **governance** (RBAC, Azure Policy) across multiple subscriptions at once
- Hierarchical structure — Management Groups can contain other Management Groups and Subscriptions
- Every tenant has a single **Root Management Group** (cannot be deleted or moved)
- All new subscriptions are placed under the Root Management Group by default

> ⚠️ **EXAM TIP:** Management Groups are about **governance at scale**. Policies and RBAC assigned at a MG apply to ALL subscriptions below. Root MG always exists and cannot be deleted.

---

## 3. Subscription Types / Offer Types

| Subscription Type | Key Details | Billing |
|---|---|---|
| **Free Trial** | $200 credit for 30 days + 12 months of free services | Convert to Pay-As-You-Go after |
| **Pay-As-You-Go (PAYG)** | Most common; billed monthly for consumed resources | Per-usage billing |
| **Enterprise Agreement (EA)** | 3-year contract with Microsoft; large orgs; upfront commitment | Negotiated pricing |
| **CSP (Cloud Solution Provider)** | Purchased through a Microsoft partner; partner manages billing | Via partner |
| **Microsoft Customer Agreement (MCA)** | Replaces EA for many orgs; flexible billing profiles | Flexible |
| **MSDN / Visual Studio** | Dev/Test workloads; discounted rates; monthly credit | Dev/Test pricing |
| **Azure for Students** | $100 credit; no credit card required; 12-month free services | Academic |
| **Sponsorship** | Microsoft-sponsored credits (MVPs, events, startups) | Credit-based |

### Free Trial Key Details

| Feature | Detail |
|---|---|
| **Credit** | $200 USD for 30 days |
| **Free services** | 25+ services free for 12 months (limited quantities) |
| **Upgrade** | Can upgrade to PAYG at any time — unused credit carries over for 30-day period |
| **After 30 days** | Disabled if not upgraded; upgrade to keep resources |
| **Spending limit** | $200 — cannot be removed on Free; removed when upgrading |

> ⚠️ **EXAM TIP:** Free Trial = $200 for 30 days. After 30 days or $200 spent → subscription **disabled** unless upgraded. **Upgrading to PAYG removes the spending limit**. Free services (like B1S VMs) continue for 12 months even after upgrade.

---

## 4. Subscription Limits (Important for Exam)

| Resource | Default Limit | Max Limit (with request) |
|---|---|---|
| **Subscriptions per tenant** | No hard limit (practical ~thousands) | Contact support |
| **Resource Groups per subscription** | **980** | Cannot increase |
| **Resources per Resource Group** | **800** (per resource type) | Varies by type |
| **Deployments per Resource Group** | **800** | Auto-deleted after limit |
| **Tags per resource/RG** | **50** | Cannot increase |
| **Tag key length** | 512 characters | — |
| **Tag value length** | 256 characters | — |
| **Role assignments per subscription** | **4,000** | Cannot increase |
| **Management Groups per tenant** | **10,000** | Cannot increase |
| **MG depth (levels below root)** | **6** (7 total including root) | Cannot increase |
| **Subscriptions per MG** | No hard limit | — |
| **Regions per subscription** | All (enabled by default) | — |
| **Co-Admins per subscription** | **200** (classic — legacy) | Cannot increase |

> ⚠️ **EXAM TIP:** Key limits: **980 RGs** per subscription, **800 deployments** per RG (auto-cleanup), **50 tags** per resource/RG, **4,000 role assignments** per subscription, MG depth = **6 levels** below root, **10,000 MGs** per tenant.

---

## 5. Management Group Hierarchy — Details

### Structure

```
Root Management Group (/)
├── MG: Production
│   ├── Sub: Prod-App-01
│   ├── Sub: Prod-App-02
│   └── MG: Prod-Databases
│       └── Sub: Prod-DB-01
├── MG: Development
│   ├── Sub: Dev-01
│   └── Sub: Dev-02
└── MG: Sandbox
    └── Sub: Sandbox-01
```

### Key Facts

| Feature | Details |
|---|---|
| **Root MG** | Auto-created; cannot be deleted or moved; every tenant has exactly 1 |
| **Root MG display name** | Default = "Tenant Root Group"; can be renamed |
| **Max depth** | 6 levels below root (7 total including root) |
| **Max MGs per tenant** | 10,000 |
| **Parent of all** | Root MG is direct or indirect parent of ALL MGs and subscriptions |
| **Default for new subs** | New subscriptions auto-placed under Root MG (unless moved) |
| **RBAC inheritance** | Roles at MG → inherited by ALL child MGs + subscriptions + RGs + resources |
| **Policy inheritance** | Policies at MG → inherited by ALL child subscriptions |
| **Moving subscriptions** | Direct role assignments preserved; inherited roles change |
| **Moving MGs** | RBAC + Policy inheritance follows the new parent |

### Root Management Group — Special Behaviors

- **Global Admin** can elevate to **User Access Administrator** at root scope
- After elevation, can assign any role on root MG → flows to entire tenant
- No user (including Global Admin) has default access to root MG — must elevate explicitly
- Root MG **cannot** be moved or deleted
- Root MG display name **can** be changed

> ⚠️ **EXAM TIP:** Root MG cannot be deleted/moved. No user has default RBAC access to root MG. Global Admin must **elevate** first via `Entra ID → Properties → Access management for Azure resources → Yes`. New subscriptions go to root MG by default.

---

## 6. Management Group vs Subscription vs Resource Group vs Resource

| Level | Purpose | Contains | Governance |
|---|---|---|---|
| **Management Group** | Organize subscriptions | MGs + Subscriptions | RBAC + Policy |
| **Subscription** | Billing + access boundary | Resource Groups | RBAC + Policy + Billing |
| **Resource Group** | Logical grouping of resources | Resources | RBAC + Policy + Tags + Locks |
| **Resource** | Individual Azure service | N/A | Inherits from above + own RBAC |

### Governance Inheritance Direction

```
Management Group
  ↓ RBAC + Policy inherited
Subscription
  ↓ RBAC + Policy inherited
Resource Group
  ↓ RBAC + Policy + Locks inherited
Resource
```

- **RBAC** inherits downward — cannot block inheritance (but deny assignments can override)
- **Policy** inherits downward — child cannot override a parent's deny policy
- **Locks** inherit from RG to resources
- **Tags** do NOT inherit — must be set at each level individually (or use Policy to enforce)

> ⚠️ **EXAM TIP:** **Tags do NOT inherit** from RG to resources. This is a very commonly tested point. Use Azure Policy effect `Modify` or `Append` to auto-apply tags. RBAC and Policy DO inherit downward.

---

## 7. Resource Tags

### What are Tags?

- **Key-value pairs** applied to resources, RGs, and subscriptions for organization
- Used for cost tracking, environment identification, automation, compliance
- **NOT inherited** from RG to resources (critical exam point)

### Tag Limits

| Limit | Value |
|---|---|
| Tags per resource / RG / subscription | **50** |
| Tag key max length | **512** characters |
| Tag value max length | **256** characters |
| Tag key max length (Storage Account) | **128** characters |
| Tag value max length (Storage Account) | **256** characters |

### Common Tag Patterns

| Tag Key | Example Value | Purpose |
|---|---|---|
| `Environment` | `Production` / `Development` / `Test` | Identify environment |
| `CostCenter` | `CC-12345` | Cost allocation |
| `Owner` | `team-devops@contoso.com` | Accountability |
| `Project` | `Project-Alpha` | Project tracking |
| `Department` | `Engineering` | Departmental billing |
| `CreatedBy` | `user@contoso.com` | Audit trail |
| `ExpirationDate` | `2025-12-31` | Lifecycle management |

### Portal Path — Apply Tags

```
Resource / Resource Group / Subscription → Tags (left menu) →
+ Add tag → Name + Value → Save
```

### CLI — Manage Tags

```bash
# Add/update tags on a resource (merge with existing)
az tag update --resource-id <resource-id> --operation Merge \
  --tags Environment=Production CostCenter=CC-123

# Replace ALL tags on a resource
az tag update --resource-id <resource-id> --operation Replace \
  --tags Environment=Production

# Delete ALL tags from a resource
az tag update --resource-id <resource-id> --operation Delete \
  --tags Environment CostCenter

# List tags for a resource
az tag list --resource-id <resource-id>

# Create a predefined tag name (subscription-level)
az tag create --name "Environment"

# Add a value to a predefined tag
az tag add-value --name "Environment" --value "Production"
```

### PowerShell — Manage Tags

```powershell
# Get existing tags
$resource = Get-AzResource -Name "myVM" -ResourceGroupName "myRG"
$resource.Tags

# Add/merge tags
$tags = @{"Environment"="Production"; "CostCenter"="CC-123"}
Update-AzTag -ResourceId $resource.Id -Tag $tags -Operation Merge

# Replace all tags
Update-AzTag -ResourceId $resource.Id -Tag $tags -Operation Replace

# Remove all tags
Remove-AzTag -ResourceId $resource.Id

# Get all resources with a specific tag
Get-AzResource -TagName "Environment" -TagValue "Production"
```

### Enforce Tags with Azure Policy

| Policy Effect | Behavior |
|---|---|
| **Deny** | Block resource creation if required tag is missing |
| **Modify** | Auto-add/modify tag on existing/new resources |
| **Append** | Add tag to resource during creation (legacy — prefer Modify) |
| **Audit** | Flag non-compliant resources but don't block |

Built-in tag policies:
- `Require a tag and its value on resources`
- `Require a tag on resource groups`
- `Inherit a tag from the resource group`
- `Add or replace a tag on resources`

> ⚠️ **EXAM TIP:** Tags do **NOT inherit**. Use policy `Inherit a tag from the resource group` (Modify effect) to auto-copy RG tags to resources. Max 50 tags per resource/RG. Tag operations: **Merge** (add/update), **Replace** (overwrite all), **Delete** (remove).

---

## 8. Resource Groups — Key Properties

| Property | Details |
|---|---|
| **Definition** | Logical container for Azure resources |
| **Location** | RG has a location (stores metadata) — resources inside can be in ANY region |
| **Lifecycle** | Deleting RG = deletes ALL resources inside |
| **Nesting** | RGs CANNOT be nested — flat structure |
| **Moving resources** | Resources can be moved between RGs (some limitations) |
| **Max per subscription** | **980** |
| **Tags** | Supported (up to 50) — NOT inherited by resources |
| **Locks** | Supported — inherit to all resources inside |
| **RBAC** | Supported — inherit to all resources inside |

### Resource Group Location

- RG **metadata** (name, properties, policy evaluations) stored in the RG's region
- If that region is down, RG metadata is unavailable — but resources in other regions still run
- **Choose RG region** that meets compliance/data residency requirements

### Portal Path — Create Resource Group

```
Home → Resource groups → + Create →
Subscription → Resource group name → Region → Tags → Review + Create
```

> ⚠️ **EXAM TIP:** RG location = metadata location, NOT resource location. Resources in an RG can be in ANY region. Deleting an RG deletes ALL resources inside — no confirmation per resource. RGs cannot be nested.

---

## 9. Moving Resources Between Subscriptions / Resource Groups

### What Can Be Moved?

| Resource Type | Between RGs | Between Subscriptions | Between Tenants |
|---|---|---|---|
| **Virtual Machines** | ✅ | ✅ (same tenant) | ⚠️ Limited |
| **Storage Accounts** | ✅ | ✅ | ⚠️ Limited |
| **VNets** | ✅ | ✅ (with all dependents) | ❌ |
| **NSGs** | ✅ | ✅ | ❌ |
| **App Service Plans** | ✅ | ✅ (same region, with constraints) | ❌ |
| **Key Vault** | ✅ | ✅ (same tenant) | ❌ |
| **Azure SQL DB** | ✅ | ✅ (same tenant) | ❌ |
| **Public IP** | ✅ | ✅ | ❌ |
| **Load Balancer** | ✅ (Basic only) | ✅ (Basic only) | ❌ |
| **Managed Disks** | ✅ | ✅ | ❌ |
| **Recovery Services Vault** | ❌ (with active backups) | ❌ | ❌ |
| **ExpressRoute** | ❌ | ❌ | ❌ |

### Move Rules & Constraints

| Rule | Details |
|---|---|
| **Source & destination** | Both RGs locked during move (read-only for move duration) |
| **Resource locks** | Must remove **Delete** locks before move (ReadOnly blocks move) |
| **Dependent resources** | Some resources must be moved together (e.g., VNet + all subnets) |
| **Region** | Move does NOT change the resource's region — only changes RG/subscription |
| **Downtime** | For most resources = **no downtime** during move |
| **Subscription state** | Both source and destination subscriptions must be **Active** |
| **Same tenant** | Cross-subscription moves must be within the same Entra ID tenant |
| **Resource provider** | Must be registered in destination subscription |
| **Quotas** | Destination subscription must have sufficient quota |
| **RBAC** | Need `Microsoft.Resources/subscriptions/resourceGroups/moveResources/action` |

### Portal Path — Move Resources

```
Resource → Overview → Move (top toolbar) →
  Move to another resource group / Move to another subscription →
  Select destination RG / Create new RG → Select resources to move →
  Validation → Move
```

### CLI — Move Resources

```bash
# Move resource to another resource group
az resource move \
  --destination-group <dest-rg> \
  --ids <resource-id-1> <resource-id-2>

# Move resource to another subscription
az resource move \
  --destination-group <dest-rg> \
  --destination-subscription-id <dest-sub-id> \
  --ids <resource-id>

# Validate move (dry run)
az resource move \
  --destination-group <dest-rg> \
  --ids <resource-id> \
  --validate
```

### PowerShell — Move Resources

```powershell
# Move to another RG
Move-AzResource -DestinationResourceGroupName "destRG" `
  -ResourceId <resource-id>

# Move to another subscription
Move-AzResource -DestinationSubscriptionId <dest-sub-id> `
  -DestinationResourceGroupName "destRG" `
  -ResourceId <resource-id>
```

> ⚠️ **EXAM TIP:** Moving a resource does NOT change its region. Both source and destination RGs are **locked** (read-only) during the move. Must remove resource locks before move. Cross-subscription moves must be **same tenant**. Not all resources can be moved — always validate first.

---

## 10. Subscription Transfer (Change Tenant / Billing)

### Transfer Types

| Transfer Type | RBAC Impact | Resource Impact |
|---|---|---|
| **Between billing accounts (same tenant)** | ✅ RBAC preserved | ✅ Resources preserved |
| **Between tenants** | ❌ ALL RBAC deleted | ✅ Resources preserved |
| **EA to MCA** | Varies | ✅ Resources preserved |
| **Between MGs (same tenant)** | ✅ Direct RBAC preserved | ✅ Resources preserved |

### What Happens on Cross-Tenant Transfer

| Item | Impact |
|---|---|
| **RBAC role assignments** | ❌ ALL deleted |
| **Custom roles** | ❌ ALL deleted |
| **System-assigned MI** | ⚠️ Disabled — must re-enable |
| **User-assigned MI** | ⚠️ Must reassign |
| **Azure Policy** | ❌ Removed |
| **Azure Blueprints** | ❌ Removed |
| **Resource locks** | ✅ Preserved |
| **Resources** | ✅ Preserved |
| **Classic admins** | ❌ Removed |
| **Key Vault access policies** | ⚠️ Must update tenant ID |
| **Azure DevOps connections** | ❌ Broken |
| **App registrations** | ⚠️ Orphaned — must re-register |

### Portal Path — Transfer Subscription

```
Subscriptions → Select subscription →
  Manage (or "Transfer billing ownership") →
  Enter recipient email → Send transfer request
```

### Portal Path — Change Directory (Tenant)

```
Subscriptions → Select subscription →
  Overview → Change directory →
  Select target Entra ID directory → Change
```

> ⚠️ **EXAM TIP:** Changing a subscription's directory (tenant) = ALL RBAC assignments deleted. All managed identities disabled. All policies removed. Resources stay intact. This is heavily tested — know what is preserved vs deleted.

---

## 11. Cost Management & Billing

### Key Components

| Component | Description |
|---|---|
| **Billing Account** | Top-level billing container (EA enrollment / MCA / PAYG) |
| **Billing Profile** | Generates invoices; contains payment methods |
| **Invoice Section** | Groups charges within a billing profile |
| **Subscription** | Billed under an invoice section/billing profile |
| **Cost Management** | Tool to analyze, monitor, and optimize costs |
| **Budgets** | Set spending thresholds with alerts |
| **Cost Alerts** | Notifications when spending exceeds thresholds |
| **Advisor Recommendations** | Optimization suggestions (resize, shut down, reserved) |

### Cost Management Features

| Feature | Description | Portal Path |
|---|---|---|
| **Cost Analysis** | View and analyze current/projected costs | `Cost Management → Cost analysis` |
| **Budgets** | Set monthly/quarterly/annual spend limits | `Cost Management → Budgets → + Add` |
| **Cost Alerts** | Auto-notify on budget thresholds | `Cost Management → Cost alerts` |
| **Advisor Recommendations** | Savings suggestions | `Azure Advisor → Cost` |
| **Export** | Schedule CSV exports to Storage Account | `Cost Management → Exports → + Add` |
| **Power BI Integration** | Advanced cost reporting | Via Power BI connector |

### Budget Alert Thresholds

| Alert Type | Trigger | Action |
|---|---|---|
| **Actual cost** | When actual spend reaches % of budget | Email / Action Group |
| **Forecasted cost** | When projected spend is predicted to reach % | Email / Action Group |
| **Credit-based (EA)** | When EA credit balance reaches % | Email notification |

### Portal Path — Create Budget

```
Cost Management → Budgets → + Add →
  Name → Reset period (Monthly / Quarterly / Annually / Billing month/quarter/year) →
  Start date → Expiration date → Budget amount →
  Alert conditions:
    Type: Actual / Forecasted →
    % of budget (e.g., 80%, 100%, 120%) →
    Action group (optional) →
    Email recipients →
  Create
```

### CLI — Cost Management

```bash
# Create a budget
az consumption budget create \
  --budget-name "MonthlyBudget" \
  --amount 1000 \
  --time-grain Monthly \
  --start-date "2025-01-01" \
  --end-date "2025-12-31" \
  --resource-group "myRG"

# List budgets
az consumption budget list

# View current usage
az consumption usage list --top 10
```

> ⚠️ **EXAM TIP:** Budgets do NOT stop spending — they only **alert**. To actually stop/limit spending, use spending limits (Free/MSDN only) or shutdown automation via Action Groups. Budget alerts can trigger Action Groups that run automation (Logic Apps, Azure Functions, runbooks).

---

## 12. Spending Limits

| Feature | Details |
|---|---|
| **What** | Prevents charges beyond credit amount |
| **Available on** | Free Trial, MSDN/Visual Studio, Azure for Students, Sponsorship |
| **NOT available on** | Pay-As-You-Go, EA, CSP |
| **When credit exhausted** | Resources **disabled** (deprovisioned) — not deleted |
| **Re-enable** | Upgrade subscription or wait for next credit period |
| **Remove spending limit** | Can remove permanently or for current billing period only |
| **After removal** | Cannot re-enable spending limit on same subscription |

### Spending Limit vs Budget

| Feature | Spending Limit | Budget |
|---|---|---|
| **Purpose** | Hard stop — disables resources | Soft alert — sends notifications |
| **Available on** | Credit-based subscriptions only | All subscription types |
| **Effect when hit** | Resources disabled | Email/action group triggered |
| **Prevents charges** | ✅ Yes | ❌ No |
| **Configurable threshold** | ❌ Fixed at credit amount | ✅ Any amount and % |
| **Action groups** | ❌ No | ✅ Yes |

### Portal Path — Manage Spending Limit

```
Subscriptions → Select subscription → Manage →
  Spending limit section → Remove spending limit / Turn on spending limit
```

> ⚠️ **EXAM TIP:** Spending limit ≠ Budget. Spending limit actually **stops** resource usage; budgets only fire **alerts**. Spending limits are only for credit-based subscriptions (Free Trial, MSDN). PAYG and EA do NOT have spending limits. Once removed, cannot be re-applied.

---

## 13. Azure Reservations & Savings Plans

| Feature | Azure Reservations | Azure Savings Plans |
|---|---|---|
| **Commitment** | Specific resource type + region + size | Hourly compute spend ($/hr) |
| **Discount** | Up to **72%** vs PAYG | Up to **65%** vs PAYG |
| **Term** | 1-year or 3-year | 1-year or 3-year |
| **Flexibility** | Specific SKU (instance size flexibility available) | Applies across VMs, App Service, Functions, AKS |
| **Scope** | Single subscription / Shared / Management Group / RG | Single subscription / Shared / Management Group |
| **Payment** | All upfront / Monthly / No upfront | All upfront / Monthly |
| **Exchange/Cancel** | Exchange for different reservation; cancel with fee | ❌ Cannot be exchanged or cancelled |
| **Auto-renew** | ✅ Optional | ✅ Optional |

### Reservation Scope Options

| Scope | Applies To |
|---|---|
| **Single subscription** | Only in that subscription |
| **Shared** | All subscriptions in the billing account |
| **Management Group** | All subscriptions under that MG |
| **Resource Group** | Only to resources in that RG |

### Portal Path — Purchase Reservation

```
Reservations → + Add → Select product (VM, SQL, Storage, etc.) →
  Select region → Select size/tier → Term (1 or 3 year) →
  Scope (Single/Shared/MG) → Quantity → Payment → Purchase
```

> ⚠️ **EXAM TIP:** Reservations = up to **72% savings**, 1/3 year terms, specific to resource type + region. Savings Plans = up to **65%**, flexible across compute services. Reservations CAN be exchanged; Savings Plans CANNOT. Know scope options: shared, single sub, MG, RG.

---

## 14. RBAC for Subscriptions & Management Groups

### Key Roles

| Role | Subscription Level | Management Group Level |
|---|---|---|
| **Owner** | Full access + assign roles | Applies to all child subs |
| **Contributor** | Full access - assign roles | Applies to all child subs |
| **Reader** | Read-only | Read-only on all child subs |
| **User Access Administrator** | Manage role assignments only | Manage role assignments on all child subs |
| **Cost Management Reader** | View cost data | View cost data on all child subs |
| **Cost Management Contributor** | View + manage cost configurations | Applies to all child subs |
| **Billing Reader** | View billing data | — |
| **Management Group Contributor** | Manage MGs (create, move, delete) | — |
| **Management Group Reader** | Read MG hierarchy | — |

### Management Group Specific Roles

| Role | Permissions |
|---|---|
| **Management Group Contributor** | Create/update/delete MGs, move subscriptions, manage policies at MG |
| **Management Group Reader** | Read MG hierarchy and properties |

### Portal Path — Assign Role at Subscription

```
Subscriptions → Select subscription → Access control (IAM) →
+ Add → Add role assignment → Role → Members → Review + assign
```

### Portal Path — Assign Role at Management Group

```
Management groups → Select MG → Access control (IAM) →
+ Add → Add role assignment → Role → Members → Review + assign
```

> ⚠️ **EXAM TIP:** RBAC at MG level inherits to ALL child subscriptions, RGs, and resources. **Management Group Contributor** can manage MG structure but NOT resources inside subscriptions. Only **Owner** and **User Access Administrator** can assign roles.

---

## 15. Azure Policy at Subscription & MG Level

- Policies assigned at **Management Group** = apply to ALL child subscriptions
- Policies assigned at **Subscription** = apply to all RGs and resources in that subscription
- Policies **cannot be blocked** by child scopes (no inheritance override)
- Use **exemptions** to exclude specific scopes from policy evaluation

### Key Policy Effects (Subscription-Relevant)

| Effect | Behavior |
|---|---|
| **Deny** | Block non-compliant resource creation/modification |
| **Audit** | Log non-compliance without blocking |
| **Modify** | Auto-modify resources to become compliant |
| **DeployIfNotExists** | Deploy resources to achieve compliance |
| **AuditIfNotExists** | Audit if related resource doesn't exist |
| **Disabled** | Turn off policy evaluation |

### Commonly Tested Built-in Policies

| Policy | Scope | Description |
|---|---|---|
| **Allowed locations** | Subscription / MG | Restrict where resources can be deployed |
| **Allowed virtual machine size SKUs** | Subscription / MG | Restrict VM sizes |
| **Require tag on resources** | RG / Subscription | Enforce tagging |
| **Inherit tag from RG** | Subscription / MG | Auto-apply RG tags to resources |
| **Not allowed resource types** | Subscription / MG | Block specific resource types |

> ⚠️ **EXAM TIP:** **Allowed locations** policy restricts WHERE resources can be created — this is separate from Azure RBAC (which restricts WHO). Know that policy assigned at MG level = affects ALL subscriptions below. Policies cannot be overridden by child scopes.

---

## 16. Subscription Management — Configuration Details

### Rename Subscription

```
Subscriptions → Select subscription → Overview →
  Click subscription name → Edit → Enter new name → Save
```

### Cancel Subscription

```
Subscriptions → Select subscription → Overview →
  Cancel subscription → Select reason → Turn off resources → Cancel
```

- Cancelled subscription = marked for deletion after **90 days** for PAYG, **30 days** for Free
- Resources **immediately disabled** but not deleted
- Data retained for recovery period
- Can **reactivate** during grace period

### Reactivate Subscription

```
Subscriptions → Select subscription (show cancelled) →
  Reactivate subscription
```

### Move to Different MG

```
Management Groups → Select subscription → Move →
  Select target Management Group → Save
```

### Add Subscription to MG

```
Management Groups → Select target MG → + Add subscription →
  Select subscription → Save
```

> ⚠️ **EXAM TIP:** Cancelled PAYG subscription = **90-day** grace period before permanent deletion. Free Trial = **30 days**. Resources are disabled immediately. Can reactivate during grace period. After deletion, data is **permanently lost**.

---

## 17. Microsoft Cost Management + Billing — Detailed Features

### Cost Analysis Views

| View | Description |
|---|---|
| **Accumulated costs** | Running total over time |
| **Daily costs** | Cost per day breakdown |
| **Cost by service** | Group by Azure service |
| **Cost by resource** | Individual resource costs |
| **Cost by resource group** | Group by RG |
| **Cost by tag** | Group by tag (e.g., CostCenter) |
| **Invoice details** | View actual invoice charges |
| **Forecast** | Projected costs based on trends |

### Portal Path — Cost Analysis

```
Cost Management → Cost analysis →
  Scope: Select subscription / RG / MG →
  View: Accumulated costs / Daily costs →
  Group by: Service / Resource / Tag / Location →
  Filter: Date range, Service, Resource type, Tag →
  Download (CSV/PNG/Excel)
```

### Portal Path — Export Cost Data

```
Cost Management → Exports → + Add →
  Name → Export type (Actual cost / Amortized cost / Usage) →
  Schedule: Daily / Weekly / Monthly →
  Storage account → Container → Directory path →
  Create
```

### Advisor Cost Recommendations

| Recommendation | Example |
|---|---|
| **Right-size VMs** | Downgrade underutilized VMs |
| **Shutdown idle VMs** | Identify VMs with low CPU |
| **Delete unused resources** | Unattached disks, idle PIPs |
| **Purchase reservations** | When usage pattern supports it |
| **Use Savings Plans** | Flexible compute commitment |

> ⚠️ **EXAM TIP:** Cost analysis can group by **tags** — why tagging strategy matters. Cost exports go to **Storage Account** (CSV format). Azure Advisor provides cost optimization recommendations — know common recommendations for exam.

---

## 18. Monitoring & Alerts for Subscriptions

### Activity Log

- **All** subscription-level operations logged in Activity Log
- Categories: Administrative, Service Health, Alert, Recommendation, Policy, Autoscale, Security
- Default retention: **90 days**
- Export to Log Analytics for longer retention

### Key Subscription Events in Activity Log

| Event | Description |
|---|---|
| `Microsoft.Resources/subscriptions/write` | Subscription created/modified |
| `Microsoft.Authorization/roleAssignments/write` | Role assignment created |
| `Microsoft.Authorization/policyAssignments/write` | Policy assigned |
| `Microsoft.Resources/subscriptions/resourceGroups/write` | RG created |
| `Microsoft.Resources/subscriptions/resourceGroups/delete` | RG deleted |

### Portal Path — View Activity Log

```
Azure Monitor → Activity Log →
  Filter: Subscription, Timespan, Event severity, Category →
  View details
```

### Portal Path — Export Activity Log

```
Azure Monitor → Activity Log → Export Activity Logs →
  + Add diagnostic setting →
  Categories: Administrative, Security, Policy →
  Send to: Log Analytics workspace / Storage Account / Event Hub →
  Save
```

### Service Health

```
Azure Monitor → Service Health →
  Service issues → View current outages affecting your subscription
  Planned maintenance → Upcoming planned changes
  Health advisories → Service changes that may affect you
  Health history → Past incidents
```

### Resource Health

```
Resource → Help → Resource Health →
  View current and historical health status
```

> ⚠️ **EXAM TIP:** Activity Log retention = **90 days** by default. Export to Log Analytics for longer. Service Health shows issues affecting **your** specific subscriptions (not all Azure). Resource Health = specific resource status.

---

## 19. Azure Subscription Governance — Best Practices

| Practice | Details |
|---|---|
| **Use Management Groups** | Organize subscriptions by business unit, environment, compliance |
| **Apply policies at MG level** | Governance cascades to all subscriptions |
| **Use naming conventions** | Standard names for MGs, subscriptions, RGs |
| **Assign RBAC to groups** | Not individual users — reduces assignment count |
| **Use tags** | Enforce via Policy; critical for cost management |
| **Separate subscriptions** | By environment (Prod/Dev/Test) or by team |
| **Set budgets** | Budget per subscription with alerts |
| **Enable cost exports** | Regular export to Storage for analysis |
| **Use resource locks** | Protect critical subscriptions/RGs |
| **Regular access reviews** | PIM + Access Reviews (P2) |
| **Limit Owner role** | Minimize Owner assignments at subscription level |
| **Use PIM** | Just-in-time activation for privileged roles |

---

## 20. Pricing Key Points

| Item | Cost |
|---|---|
| **Subscriptions** | Free to create — you pay for resources inside |
| **Management Groups** | **Free** |
| **Resource Groups** | **Free** |
| **Tags** | **Free** |
| **Cost Management** | **Free** (for Azure resources) |
| **Budgets & Alerts** | **Free** |
| **Azure Advisor** | **Free** |
| **Cost exports** | Free service — but Storage Account charges apply |
| **RBAC** | **Free** |
| **Azure Policy** | **Free** for Azure resources; charges for Arc-connected resources |
| **Azure Reservations** | Discount commitment — prepay vs PAYG rates |
| **PIM** | Requires **P2 license** |
| **Access Reviews** | Requires **P2 license** |

> ⚠️ **EXAM TIP:** Management Groups, RBAC, Tags, Budgets, Cost Management, Policy = ALL **free**. You only pay for the actual Azure resources. PIM and Access Reviews require P2.

---

## 21. CLI & PowerShell Commands

### Azure CLI — Subscription Commands

```bash
# List all subscriptions
az account list --output table

# Show current subscription
az account show

# Set active subscription
az account set --subscription "<sub-name-or-id>"

# List subscription locations
az account list-locations --output table

# Create resource group
az group create --name "myRG" --location "eastus"

# Delete resource group
az group delete --name "myRG" --yes --no-wait

# List resource groups
az group list --output table

# List resources in a resource group
az resource list --resource-group "myRG" --output table

# Show subscription limits/quotas
az vm list-usage --location "eastus" --output table
```

### Azure CLI — Management Group Commands

```bash
# List management groups
az account management-group list --output table

# Create management group
az account management-group create --name "MG-Production" --display-name "Production"

# Create child management group under parent
az account management-group create --name "MG-ProdApps" \
  --parent "MG-Production" --display-name "Prod Applications"

# Move subscription to management group
az account management-group subscription add \
  --name "MG-Production" --subscription "<sub-id>"

# Remove subscription from management group
az account management-group subscription remove \
  --name "MG-Production" --subscription "<sub-id>"

# Delete management group (must be empty)
az account management-group delete --name "MG-Production"

# Show management group details
az account management-group show --name "MG-Production" --expand --recurse
```

### Azure PowerShell — Subscription & MG Commands

```powershell
# List subscriptions
Get-AzSubscription

# Set active subscription
Set-AzContext -SubscriptionId "<sub-id>"

# Create resource group
New-AzResourceGroup -Name "myRG" -Location "eastus"

# Delete resource group
Remove-AzResourceGroup -Name "myRG" -Force

# List management groups
Get-AzManagementGroup

# Create management group
New-AzManagementGroup -GroupName "MG-Production" -DisplayName "Production"

# Create child MG
New-AzManagementGroup -GroupName "MG-ProdApps" `
  -DisplayName "Prod Apps" -ParentId "/providers/Microsoft.Management/managementGroups/MG-Production"

# Move subscription under MG
New-AzManagementGroupSubscription -GroupName "MG-Production" -SubscriptionId "<sub-id>"

# Remove subscription from MG
Remove-AzManagementGroupSubscription -GroupName "MG-Production" -SubscriptionId "<sub-id>"

# Delete MG
Remove-AzManagementGroup -GroupName "MG-Production"
```

### Azure CLI — Tag Commands

```bash
# Apply tags (merge)
az tag update --resource-id <id> --operation Merge --tags Env=Prod Dept=IT

# List tags on subscription
az tag list --resource-id /subscriptions/<sub-id>

# Apply tags to resource group
az tag update --resource-id /subscriptions/<sub-id>/resourceGroups/myRG \
  --operation Merge --tags Env=Prod
```

> ⚠️ **EXAM TIP:** `az account set` switches active subscription. `az account management-group create` creates MGs. `az account management-group subscription add` moves a subscription into a MG. MG must be **empty** before it can be deleted. `--expand --recurse` shows full hierarchy.

---

## 22. Quick-Fire Exam Points ⚡

1. **Subscription** = billing boundary + access control boundary
2. Every resource belongs to **exactly one** subscription
3. One subscription → **one tenant**; one tenant → **many subscriptions**
4. **Management Group** = container above subscriptions for governance at scale
5. Every tenant has **one Root Management Group** — cannot be deleted or moved
6. **Max MG depth** = 6 levels below root (7 total including root)
7. **Max MGs per tenant** = 10,000
8. **Max RGs per subscription** = 980
9. **Max deployments per RG** = 800 (auto-cleanup when exceeded)
10. **Max tags per resource/RG** = 50; key = 512 chars; value = 256 chars
11. **Tags do NOT inherit** from RG to resources — use Azure Policy to enforce inheritance
12. RBAC + Policy **inherit downward** through the hierarchy (MG → Sub → RG → Resource)
13. Policies cannot be overridden at child scopes — use exemptions instead
14. **Root MG** — no user has default access; Global Admin must **elevate** explicitly
15. **Global Admin elevation** = User Access Administrator at root scope (`/`)
16. Moving subscription between MGs: **direct RBAC preserved**, **inherited RBAC changes**
17. **Cross-tenant subscription transfer** = ALL RBAC, policies, managed identities, custom roles **deleted**
18. Resources and locks are **preserved** in cross-tenant transfer
19. Cancelled subscription: PAYG = **90-day** grace, Free = **30-day** grace, then permanently deleted
20. Resources **immediately disabled** on cancellation but data retained during grace period
21. **Budget** = alert only, does NOT stop spending
22. **Spending limit** = hard stop, only for credit-based subscriptions (Free/MSDN)
23. Spending limit once removed → **cannot be re-enabled**
24. **Cost Management, Budgets, Tags, MGs, RGs, RBAC, Policy** = all **FREE**
25. **PIM and Access Reviews** = require **P2 license**
26. **Allowed locations** policy = restricts WHERE resources can be deployed (not WHO)
27. RG location = **metadata** location; resources inside can be in **any region**
28. Deleting an RG = deletes **ALL resources** inside
29. RGs **cannot be nested** — flat structure only
30. Moving resources between RGs/subscriptions does NOT change the resource's **region**
31. During resource move, both source and destination RGs are **read-only locked**
32. Cross-subscription resource move must be within **same tenant**
33. Must remove **resource locks** before moving resources
34. Resource provides must be **registered** in destination subscription before move
35. **Azure Reservations** = up to **72% savings**, 1/3 year, can exchange
36. **Azure Savings Plans** = up to **65% savings**, 1/3 year, **cannot** exchange or cancel
37. Reservations scope: Single sub / Shared / MG / RG
38. **Activity Log** retention = 90 days; export to Log Analytics for longer
39. **Service Health** = Azure issues affecting YOUR subscriptions specifically
40. **Cost analysis** can group by tags — key reason for proper tagging strategy
41. `az account set --subscription` changes active subscription in CLI
42. `az account management-group create` creates MGs; `--parent` sets hierarchy
43. MG must be **empty** (no child MGs or subs) before deletion
44. **Billing Reader** role = view billing data without resource access
45. **Management Group Contributor** = manage MG structure (create/move/delete MGs)
46. Tag operations: **Merge** (add/update), **Replace** (overwrite all), **Delete** (remove)
47. Built-in policy `Inherit a tag from the resource group` = auto-apply RG tags to resources
48. **Co-Admins** (classic) = legacy; max 200 per subscription; use RBAC instead
49. Azure Advisor provides FREE cost optimization recommendations (right-size, shutdown, reservations)
50. Cost exports are saved to a **Storage Account** in CSV format — export service is free, storage charges apply

---

## 23. Step-by-Step Configuration Mind Maps 🗺️

---

### 23.1 Create a Management Group

> **Portal:** `Management Groups → + Add`

```
Create Management Group
│
├── Prerequisites
│   ├── Permissions: Microsoft.Management/managementGroups/write
│   │   └── Any user (if protection enabled = only MG Contributor / Owner)
│   └── MG hierarchy protection may be enabled (requires admin)
│
├── Step 1: Navigate
│   └── Home → Management groups
│
├── Step 2: + Add management group
│
├── Step 3: Configure
│   ├── Management group ID (unique, immutable after creation)
│   │   └── ⚠️ Cannot be changed after creation!
│   ├── Display name (can be changed later)
│   └── Parent: Select parent MG (default = Root MG)
│
├── Step 4: Save
│
├── Required Role: varies by tenant settings
│   ├── Default: Any Entra ID user can create MGs
│   └── If protected: Requires MG Contributor or similar
│
└── ⚠️ Key Points
    ├── MG ID is immutable — choose carefully
    ├── Max 10,000 MGs per tenant
    ├── Max 6 levels below root
    ├── New MG creation can take up to 15 minutes to propagate
    └── MG must be empty to delete
```

---

### 23.2 Move Subscription to Management Group

> **Portal:** `Management Groups → Select MG → + Add subscription`

```
Move Subscription to Management Group
│
├── Step 1: Navigate
│   └── Home → Management groups → Select target MG
│
├── Step 2: + Add subscription
│   └── Select subscription from dropdown → Save
│
│   OR (from subscription side):
│   └── Subscriptions → Select sub → Move → Select target MG → Save
│
├── Required Role:
│   ├── On the subscription: Owner
│   ├── On the target MG: MG Contributor or Owner
│   └── On the source MG: MG Contributor or Owner
│
├── Step 3: Verify
│   └── Check MG hierarchy shows subscription under new parent
│
└── ⚠️ Key Points
    ├── Direct RBAC on subscription = PRESERVED
    ├── Inherited RBAC from old MG = LOST
    ├── Inherited RBAC from new MG = GAINED
    ├── Policies from new MG apply immediately
    ├── Policies from old MG no longer apply
    └── Cannot move subscription to MG if depth would exceed 6 levels
```

---

### 23.3 Create a Subscription

> **Portal:** `Subscriptions → + Add`

```
Create Subscription
│
├── Step 1: Navigate
│   └── Home → Subscriptions → + Add
│
├── Step 2: Select offer
│   ├── Pay-As-You-Go
│   ├── Dev/Test (Pay-As-You-Go)
│   ├── Enterprise Agreement (if applicable)
│   └── Other available offers
│
├── Step 3: Configure
│   ├── Subscription name
│   ├── Billing account
│   ├── Billing profile / Invoice section (MCA)
│   ├── Directory (Entra ID tenant)
│   └── Management group (optional — default = Root MG)
│
├── Step 4: Review + Create
│
├── Required Role: Billing account role (varies by account type)
│   ├── EA: Account Owner on enrollment account
│   ├── MCA: Azure subscription creator role on invoice section
│   └── PAYG: Account admin
│
└── ⚠️ Key Points
    ├── Sub placed under Root MG by default if no MG selected
    ├── Sub linked to the selected Entra ID tenant
    ├── One sub → one tenant (can change later, with consequences)
    └── Programmatic creation requires specific billing roles
```

---

### 23.4 Create a Resource Group

> **Portal:** `Resource Groups → + Create`

```
Create Resource Group
│
├── Step 1: Navigate
│   └── Home → Resource groups → + Create
│
├── Step 2: Basics
│   ├── Subscription: Select target subscription
│   ├── Resource group name (unique within subscription)
│   │   ├── 1-90 characters
│   │   ├── Alphanumeric, underscore, hyphen, period, parenthesis
│   │   └── Cannot end with a period
│   └── Region (where metadata is stored)
│       └── ⚠️ RG region ≠ resource region; resources can be in ANY region
│
├── Step 3: Tags (optional)
│   └── Add key-value pairs (max 50)
│       └── ⚠️ Tags do NOT inherit to resources inside
│
├── Step 4: Review + Create
│
├── Required Role: Contributor or Owner at subscription level
│   (needs Microsoft.Resources/subscriptions/resourceGroups/write)
│
└── ⚠️ Key Points
    ├── Max 980 RGs per subscription
    ├── RGs cannot be nested
    ├── Deleting RG = deletes ALL resources inside
    ├── RG region = metadata location only
    └── RG name must be unique within the subscription
```

---

### 23.5 Apply and Manage Tags

> **Portal:** `Resource / RG / Subscription → Tags`

```
Apply and Manage Tags
│
├── Step 1: Navigate to target
│   └── Resource / Resource Group / Subscription → Tags (left menu)
│
├── Step 2: Add Tags
│   ├── + Add tag
│   │   ├── Name (key) — max 512 chars (128 for Storage)
│   │   └── Value — max 256 chars
│   ├── Add multiple tags (up to 50 total)
│   └── Save
│
├── Step 3: Edit/Delete Tags
│   ├── Modify value → Save
│   └── Click X to remove → Save
│
├── Required Role: Tag Contributor or Contributor/Owner
│   (needs Microsoft.Resources/tags/write)
│
├── Enforce via Policy:
│   ├── Azure Policy → Assignments → + Assign policy →
│   │   ├── "Require a tag and its value on resources" (Deny)
│   │   ├── "Inherit a tag from the resource group" (Modify)
│   │   └── "Require a tag on resource groups" (Deny)
│   └── Scope: Subscription / MG / RG
│
└── ⚠️ Key Points
    ├── Tags do NOT inherit (critical exam point)
    ├── Max 50 tags per resource/RG/subscription
    ├── Use Policy with Modify effect to auto-inherit from RG
    ├── Tags are essential for cost management (group costs by tag)
    └── Storage accounts: key max 128 chars (shorter than standard 512)
```

---

### 23.6 Set Up a Budget

> **Portal:** `Cost Management → Budgets → + Add`

```
Set Up Budget
│
├── Step 1: Navigate
│   └── Cost Management → Budgets → + Add
│
├── Step 2: Budget Details
│   ├── Name
│   ├── Scope: Subscription / Resource Group / MG
│   ├── Reset period:
│   │   ├── Monthly / Quarterly / Annually
│   │   └── Billing month / Billing quarter / Billing year (EA)
│   ├── Creation date
│   ├── Expiration date
│   └── Budget amount ($)
│
├── Step 3: Alert Conditions
│   ├── + Add alert condition
│   │   ├── Type: Actual / Forecasted
│   │   ├── % of budget: e.g., 50%, 80%, 100%, 120%
│   │   └── Can add multiple thresholds
│   ├── Alert recipients: email addresses
│   └── Action group (optional — for automation)
│       └── Can trigger: Email, SMS, Logic App, Azure Function, Runbook, Webhook
│
├── Step 4: Create
│
├── Required Role: Cost Management Contributor or higher
│
└── ⚠️ Key Points
    ├── Budgets = ALERTS ONLY — do NOT stop spending
    ├── Use Action Groups to trigger automation (e.g., shutdown VMs)
    ├── Forecasted alerts = warns BEFORE budget is exceeded
    ├── Actual alerts = warns AFTER threshold crossed
    ├── Budget alerts typically fire within 8-12 hours of threshold
    └── Budgets are FREE
```

---

### 23.7 Transfer Subscription to Different Tenant

> **Portal:** `Subscriptions → Select → Change directory`

```
Transfer Subscription to Different Tenant
│
├── Prerequisites
│   ├── Global Admin or Owner on the subscription
│   ├── Must be a member of the destination tenant
│   ├── No active Azure Lighthouse delegations
│   └── ⚠️ UNDERSTAND WHAT WILL BE DELETED (see below)
│
├── Step 1: Navigate
│   └── Subscriptions → Select subscription → Overview
│
├── Step 2: Change directory
│   ├── Select destination Entra ID directory
│   └── ⚠️ Review all warnings about what will be removed
│
├── Step 3: Change
│   └── Subscription moves to new tenant
│
├── Step 4: Post-Transfer Actions (CRITICAL)
│   ├── Reassign ALL RBAC role assignments (all were deleted)
│   ├── Recreate ALL custom roles (all were deleted)
│   ├── Re-enable system-assigned managed identities
│   ├── Reassign user-assigned managed identities
│   ├── Reassign Azure Policy assignments
│   ├── Update Key Vault tenant IDs
│   ├── Reconfigure service connections (DevOps, CI/CD)
│   └── Re-register resource providers if needed
│
├── Required Role: Owner on subscription + Global Admin or User Admin in destination tenant
│
└── ⚠️ Key Points
    ├── ALL RBAC assignments = DELETED
    ├── ALL custom roles = DELETED
    ├── ALL Policy assignments = REMOVED
    ├── Managed identities = DISABLED
    ├── Resources = PRESERVED
    ├── Resource locks = PRESERVED
    ├── This is a HIGH-IMPACT operation
    └── Plan for significant post-transfer reconfiguration work
```

---

### 23.8 Move Resources Between Resource Groups

> **Portal:** `Resource → Overview → Move`

```
Move Resources Between RGs
│
├── Prerequisites
│   ├── Remove resource locks (ReadOnly/Delete) from source and destination
│   ├── Both source and destination subscriptions must be Active
│   ├── Resource provider registered in destination (if cross-sub)
│   ├── Sufficient quota in destination subscription (if cross-sub)
│   └── ⚠️ Not all resource types support move — validate first
│
├── Step 1: Navigate to resource
│   └── Resource → Overview → Move (top toolbar)
│
├── Step 2: Select move type
│   ├── Move to another resource group
│   └── Move to another subscription
│
├── Step 3: Configure
│   ├── Destination subscription (if cross-sub)
│   ├── Destination resource group (existing or create new)
│   └── Select resources to move (can move dependent resources together)
│
├── Step 4: Validation
│   └── Azure validates the move — shows errors if not supported
│
├── Step 5: Move
│   └── Confirm → Move starts
│
├── Required Role:
│   ├── Contributor+ on source RG
│   ├── Contributor+ on destination RG
│   └── Microsoft.Resources/subscriptions/resourceGroups/moveResources/action
│
└── ⚠️ Key Points
    ├── Both source + destination RGs are READ-ONLY during move
    ├── Resource region does NOT change
    ├── Cross-subscription = must be same tenant
    ├── Some resources cannot be moved (ExpressRoute, Recovery Services w/ backups)
    ├── VNet move = must move ALL dependent resources (subnets, peers stay)
    ├── No downtime for most resources
    └── Always validate before moving (CLI: az resource move --validate)
```

---

### 23.9 Cancel and Reactivate Subscription

> **Portal:** `Subscriptions → Select → Cancel subscription / Reactivate`

```
Cancel and Reactivate Subscription
│
├── Cancel Subscription
│   │
│   ├── Step 1: Navigate
│   │   └── Subscriptions → Select subscription → Overview
│   │
│   ├── Step 2: Cancel subscription
│   │   ├── Select reason for cancellation
│   │   ├── Turn off resources
│   │   └── Confirm cancellation
│   │
│   ├── Required Role: Owner or Account Admin
│   │
│   └── ⚠️ Effects
│       ├── Resources immediately DISABLED (inaccessible)
│       ├── Data retained for grace period
│       ├── PAYG grace period = 90 days
│       ├── Free Trial grace period = 30 days
│       ├── EA grace period = 90 days
│       ├── After grace → permanently deleted (no recovery)
│       └── Other subscriptions in tenant are NOT affected
│
├── Reactivate Subscription
│   │
│   ├── Step 1: Navigate
│   │   └── Subscriptions → Show cancelled subscriptions → Select
│   │
│   ├── Step 2: Reactivate
│   │   └── Click "Reactivate"
│   │
│   ├── Required Role: Owner or Account Admin
│   │
│   └── ⚠️ Key Points
│       ├── Only possible during grace period
│       ├── All resources restored to previous state
│       ├── Data that was retained becomes available again
│       └── RBAC assignments preserved (if same tenant)
│
└── ⚠️ Overall Key Points
    ├── Cancellation ≠ immediate deletion
    ├── Always export critical data before cancelling
    ├── Resource locks do NOT prevent subscription cancellation
    └── Consider moving resources instead of cancelling
```

---

### 23.10 Configure Cost Alerts and Exports

> **Portal:** `Cost Management → Cost alerts / Exports`

```
Configure Cost Alerts and Exports
│
├── Cost Alerts
│   │
│   ├── Step 1: Navigate
│   │   └── Cost Management → Cost alerts
│   │
│   ├── Step 2: View/manage active alerts
│   │   ├── Budget alerts (from budget thresholds)
│   │   ├── Credit alerts (EA — when credits running low)
│   │   ├── Department spending quota alerts (EA)
│   │   └── Anomaly alerts (unexpected cost spikes)
│   │
│   └── ⚠️ Alerts are created through Budgets, not directly here
│
├── Cost Exports
│   │
│   ├── Step 1: Navigate
│   │   └── Cost Management → Exports → + Add
│   │
│   ├── Step 2: Configure
│   │   ├── Export name
│   │   ├── Metric type:
│   │   │   ├── Actual cost (as billed)
│   │   │   ├── Amortized cost (reservation costs spread over term)
│   │   │   └── Usage (raw consumption data)
│   │   ├── Export type:
│   │   │   ├── Daily export of month-to-date costs
│   │   │   ├── Weekly export of cost for last 7 days
│   │   │   └── Monthly export of last month's costs
│   │   └── Storage account + Container + Directory
│   │
│   ├── Step 3: Create
│   │
│   ├── Required Role: Cost Management Contributor or higher
│   │
│   └── ⚠️ Key Points
│       ├── Export service = FREE; Storage Account charges apply
│       ├── Data format = CSV
│       ├── Can export at Subscription, RG, or MG scope
│       └── Use for integration with Power BI, external tools
│
└── Anomaly Detection
    ├── Automatically detects unexpected cost patterns
    ├── Uses machine learning to identify anomalies
    ├── Accessible via: Cost Management → Cost analysis → Anomaly detection tab
    └── Results visible in Cost alerts
```

---

### 23.11 Manage Subscription Resource Providers

> **Portal:** `Subscriptions → Select → Settings → Resource providers`

```
Manage Resource Providers
│
├── Step 1: Navigate
│   └── Subscriptions → Select subscription → Settings → Resource providers
│
├── Step 2: View registered/not registered providers
│   ├── Search for provider (e.g., Microsoft.Compute, Microsoft.Storage)
│   ├── Status: Registered / NotRegistered / Registering
│   └── Many providers auto-registered when first resource created
│
├── Step 3: Register a provider (if needed for resource move or new service)
│   ├── Select provider → Register
│   └── Wait for status to change to "Registered"
│
├── Required Role: Contributor or Owner
│   (needs Microsoft.Resources/providers/register/action)
│
├── CLI:
│   ├── az provider register --namespace Microsoft.Compute
│   ├── az provider list --output table
│   └── az provider show --namespace Microsoft.Compute
│
└── ⚠️ Key Points
    ├── Some providers are auto-registered; others must be manually registered
    ├── Cross-sub resource move may fail if provider not registered in destination
    ├── Common exam issue: "Resource type not available" = provider not registered
    └── Registration is per-subscription
```


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
