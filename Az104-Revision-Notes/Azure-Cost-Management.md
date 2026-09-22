<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Cost Management — AZ-104 Revision Notes

---

## 1. What is Azure Cost Management?

- **Free built-in** Azure service for analyzing, monitoring, and optimizing cloud costs
- Provides cost analysis, budgets, alerts, exports, and recommendations
- Works at **Management Group**, **Subscription**, **Resource Group**, and **Resource** scope
- Integrated with **Azure Advisor** for optimization recommendations
- Supports Azure, AWS (via connector), and GCP (limited)

> ⚠️ **EXAM TIP:** Cost Management is **FREE** for Azure resources. You only pay for the resources themselves. Cost Management for AWS requires a connector and additional setup.

---

## 2. Key Components

| Component | Description | Portal Path |
|---|---|---|
| **Cost Analysis** | View and analyze current/historical/forecasted costs | `Cost Management → Cost analysis` |
| **Budgets** | Set spending thresholds with alerts | `Cost Management → Budgets` |
| **Cost Alerts** | View all triggered cost alerts | `Cost Management → Cost alerts` |
| **Exports** | Schedule automatic CSV exports to Storage Account | `Cost Management → Exports` |
| **Advisor Recommendations** | Cost savings suggestions | `Azure Advisor → Cost` |
| **Reservations** | Prepaid resource commitments for discounts | `Reservations` |
| **Savings Plans** | Flexible compute commitment for discounts | `Savings plans` |
| **Billing** | Invoices, payment methods, billing accounts | `Cost Management + Billing` |
| **Power BI Integration** | Advanced reporting via connector | Via Power BI Desktop |

---

## 3. Cost Analysis — Detailed

### Views Available

| View | Description |
|---|---|
| **Accumulated costs** | Running total over selected period |
| **Daily costs** | Per-day cost breakdown |
| **Cost by service** | Grouped by Azure service name |
| **Cost by resource** | Individual resource costs |
| **Cost by resource group** | Grouped by RG |
| **Cost by location** | Grouped by Azure region |
| **Cost by tag** | Grouped by tag key/value (e.g., CostCenter) |
| **Cost by meter** | Grouped by billing meter |
| **Invoice details** | Actual invoice line items |
| **Forecast** | Projected future costs based on trends |

### Granularity Options

| Granularity | Description |
|---|---|
| **Daily** | Cost per day |
| **Monthly** | Cost per month |
| **None** | Total for entire period |

### Group By Options

- Service name, Service tier, Meter, Meter category
- Resource, Resource group, Resource type, Resource location
- Subscription, Tag, Pricing model, Charge type
- Publisher type, Frequency, Invoice section, Billing profile

### Portal Path — Cost Analysis

```
Cost Management → Cost analysis →
  Scope: Select MG / Subscription / RG →
  View: Select predefined or custom view →
  Date range: Last 7 days / This month / Last month / Custom →
  Granularity: Daily / Monthly / None →
  Group by: Service / Resource / Tag / Location etc. →
  Add filter: Tag, Resource type, Location, etc. →
  Chart type: Area / Column / Line / Table →
  Download: CSV / PNG / Excel
```

### Save Custom Views

```
Cost Management → Cost analysis → Customize view →
  Save → Name → Scope → Pin to dashboard (optional) → Save
```

> ⚠️ **EXAM TIP:** Cost analysis can group by **tags** — this is why proper tagging strategy is critical. Forecasting uses machine learning to predict future costs. Data availability: PAYG = near real-time; EA = up to 24-hour delay.

---

## 4. Budgets — Detailed

### What is a Budget?

- Set **spending thresholds** and receive alerts when thresholds are approached/exceeded
- Budgets **do NOT stop spending** — they only send notifications
- Can trigger **Action Groups** for automation (shutdown VMs, send webhooks, etc.)
- Scope: Management Group, Subscription, or Resource Group

### Budget Configuration

| Setting | Options |
|---|---|
| **Name** | Unique name within scope |
| **Scope** | MG / Subscription / RG |
| **Reset period** | Monthly / Quarterly / Annually / Billing Month / Billing Quarter / Billing Year |
| **Start date** | When budget tracking begins |
| **Expiration date** | When budget stops evaluating |
| **Amount** | Dollar amount for the period |
| **Filters** | Resource group, Resource, Meter, Tag, etc. |

### Alert Types

| Alert Type | When It Fires | Use Case |
|---|---|---|
| **Actual** | When actual spend **reaches** the % threshold | Know when you've spent X% |
| **Forecasted** | When projected spend **is predicted to reach** % threshold | Early warning before overspend |

### Alert Thresholds

- Set **multiple thresholds** per budget (e.g., 50%, 80%, 100%, 120%)
- Each threshold can have different notification recipients
- Thresholds expressed as **% of budget amount**
- Can set thresholds **above 100%** (e.g., 120% = alert when 20% over budget)

### Action Groups with Budgets

- Trigger automation when budget threshold is reached
- Common actions: Email, SMS, Push notification, Voice call
- Automation: Azure Function, Logic App, Automation Runbook, Webhook
- Example: Auto-shutdown dev VMs when 90% of budget consumed

### Portal Path — Create Budget

```
Cost Management → Budgets → + Add →
  Name → Reset period → Start date → Expiration date → Amount →
  Filters (optional: RG, Resource, Tag, Meter) →
  Next: Alert conditions →
    + Add condition →
    Type: Actual / Forecasted →
    % of budget → Alert recipients (email) →
    Action group (optional) →
    Language preference →
  Create
```

### CLI — Create Budget

```bash
# Create budget at subscription scope
az consumption budget create \
  --budget-name "MonthlyBudget" \
  --amount 1000 \
  --time-grain Monthly \
  --start-date "2025-01-01" \
  --end-date "2025-12-31" \
  --category Cost

# List budgets
az consumption budget list --output table

# Delete budget
az consumption budget delete --budget-name "MonthlyBudget"
```

### PowerShell — Create Budget

```powershell
# Create budget
New-AzConsumptionBudget -Name "MonthlyBudget" `
  -Amount 1000 `
  -TimeGrain Monthly `
  -StartDate "2025-01-01" `
  -EndDate "2025-12-31" `
  -Category Cost

# Get budgets
Get-AzConsumptionBudget

# Remove budget
Remove-AzConsumptionBudget -Name "MonthlyBudget"
```

> ⚠️ **EXAM TIP:** Budgets = **alerts only**, NOT spending caps. To actually stop spending, use spending limits (credit-based subs only) or automation via Action Groups. **Forecasted** alerts warn BEFORE threshold is hit. Budget alerts typically fire within **8–24 hours** of threshold being crossed. Budgets are **FREE**.

---

## 5. Cost Alerts

### Alert Types

| Type | Source | Description |
|---|---|---|
| **Budget alerts** | Budgets | Triggered when cost/forecast hits budget threshold |
| **Credit alerts** | EA | When EA monetary commitment balance reaches 90% and 100% consumed |
| **Department spending quota alerts** | EA | When department spending reaches configured % |
| **Anomaly alerts** | Cost Management | AI-detected unexpected cost spikes |

### Anomaly Detection

- Uses **machine learning** to detect unexpected cost patterns
- Automatically enabled — no configuration needed
- Evaluates daily costs for irregularities
- Shows root cause analysis (which resource/service caused the anomaly)

### Portal Path — View Cost Alerts

```
Cost Management → Cost alerts →
  Filter: Type (Budget / Credit / Quota / Anomaly) →
  Status (Active / Dismissed) →
  View details → Root cause analysis
```

> ⚠️ **EXAM TIP:** **Anomaly alerts** are automatic (ML-based) and require no setup. **Credit alerts** are EA-only and fire at 90% and 100% credit consumption. Budget alerts support Action Groups for automation; credit alerts do NOT.

---

## 6. Cost Exports

- Schedule **automatic CSV exports** of cost data to Azure Storage Account
- Used for external reporting, Power BI, third-party tools
- Export service = **FREE**; Storage Account charges apply

### Export Types

| Export Type | Data |
|---|---|
| **Actual cost** | Charges as billed (incl. RI purchases as lump sum) |
| **Amortized cost** | RI/Savings Plan costs spread evenly over term |
| **Usage** | Raw resource consumption data |

### Schedule Options

| Schedule | Description |
|---|---|
| **Daily export of month-to-date** | Every day, exports costs from month start to current |
| **Weekly export** | Every week, exports last 7 days |
| **Monthly export of last month** | First of month, exports previous full month |
| **One-time export** | Single export for custom date range |

### Portal Path — Create Export

```
Cost Management → Exports → + Add →
  Name → Metric: Actual cost / Amortized cost / Usage →
  Export type: Daily / Weekly / Monthly / One-time →
  Start date →
  Storage account → Subscription → Storage account → Container →
  Directory (folder path) →
  Create
```

### CLI — Create Export

```bash
# Create daily cost export
az costmanagement export create \
  --name "DailyCostExport" \
  --scope "/subscriptions/{sub-id}" \
  --type "ActualCost" \
  --timeframe "MonthToDate" \
  --storage-account-id "<storage-id>" \
  --storage-container "exports" \
  --storage-directory "daily" \
  --schedule-recurrence "Daily" \
  --schedule-status "Active"
```

> ⚠️ **EXAM TIP:** Exports go to a **Storage Account** in CSV format. The export service is free; you pay for storage. **Amortized** view spreads reservation purchases over the term. Use exports for Power BI or external analysis.

---

## 7. Spending Limits

| Feature | Details |
|---|---|
| **What** | Hard cap that prevents charges beyond credit |
| **Available on** | Free Trial, Visual Studio/MSDN, Azure for Students, Sponsorship |
| **NOT on** | Pay-As-You-Go, EA, CSP, MCA |
| **When hit** | Resources **disabled/deallocated** (not deleted) |
| **Remove** | Can remove for current period or permanently |
| **Re-enable** | Cannot re-enable after permanent removal |

### Spending Limit vs Budget

| Feature | Spending Limit | Budget |
|---|---|---|
| **Purpose** | Hard stop — disables resources | Soft alert — notifications only |
| **Available** | Credit-based subs only | All subscription types |
| **Effect** | Resources disabled | Email / Action Group fires |
| **Prevents charges** | ✅ Yes | ❌ No |
| **Configurable threshold** | ❌ Fixed at credit amount | ✅ Any amount / % |
| **Automation support** | ❌ No | ✅ Action Groups |

### Portal Path — Spending Limit

```
Subscriptions → Select subscription →
  Manage → Spending limit → Remove / Turn on
```

> ⚠️ **EXAM TIP:** Spending limit ≠ Budget. Spending limits **stop resources**; budgets only **alert**. Only available on credit-based subscriptions. Once permanently removed, **cannot** be re-enabled. PAYG has **no** spending limit.

---

## 8. Azure Reservations

- **Pre-purchase** specific Azure resources for 1 or 3 years at discounted rates
- Up to **72% savings** compared to PAYG pricing
- Applies to specific **resource type + region + size** (with flexibility options)

### Supported Resource Types

| Resource | Reservation Available |
|---|---|
| **Virtual Machines** | ✅ (most common) |
| **Azure SQL Database** | ✅ (vCore) |
| **Azure Cosmos DB** | ✅ (throughput RU/s) |
| **Azure Storage** | ✅ (blob capacity) |
| **Azure Synapse Analytics** | ✅ |
| **Azure Databricks** | ✅ |
| **App Service** | ✅ (Isolated stamp fee) |
| **Azure Dedicated Host** | ✅ |
| **Azure Disk Storage** | ✅ (Premium SSD) |
| **Azure Cache for Redis** | ✅ |
| **Azure Data Explorer** | ✅ |

### Reservation Scope

| Scope | Applies To |
|---|---|
| **Shared** | All subscriptions in billing account |
| **Single subscription** | Only that subscription |
| **Single resource group** | Only resources in that RG |
| **Management Group** | All subscriptions under that MG |

### Payment Options

| Option | Description |
|---|---|
| **All upfront** | Full payment at purchase — greatest discount |
| **Monthly** | Spread over term — slightly less discount |
| **No upfront** | Pay nothing upfront (available for some RIs) — least discount |

### Instance Size Flexibility

- Within same VM series, reservation applies to **different sizes** in same region
- Example: Reserved D4s_v3 → applies to D2s_v3, D8s_v3 (proportionally)
- Enabled by default for VMs
- Ratio-based: smaller VM = uses fraction; larger VM = uses multiple units

### Exchange and Cancellation

| Action | Policy |
|---|---|
| **Exchange** | ✅ Can exchange for different reservation of **equal or greater** value |
| **Cancel (return)** | ✅ Up to **$50,000** per rolling 12-month window |
| **Early termination fee** | 12% of remaining unused amount (prorated) |
| **Cancel refund** | Refunded to original payment method |

### Portal Path — Purchase Reservation

```
Reservations → + Add →
  Product type (VM, SQL, Storage, etc.) → Select →
  Scope: Shared / Single sub / MG / RG →
  Region → Size / SKU → Term: 1 year / 3 years →
  Quantity → Billing frequency: All upfront / Monthly →
  Review + Purchase
```

### Portal Path — Exchange / Cancel

```
Reservations → Select reservation →
  Exchange: Exchange → Select new reservation → Review + exchange
  Refund: Return → Confirm → Submit
```

### CLI — View Reservations

```bash
# List all reservations
az reservations reservation list --reservation-order-id <order-id>

# List reservation orders
az reservations reservation-order list --output table
```

> ⚠️ **EXAM TIP:** Reservations = up to **72% savings**, 1/3 year. Can **exchange** (equal or greater value). Can **cancel** (up to $50K/12 months, 12% fee). Instance size flexibility = ON by default for VMs. Scope determines which subscriptions benefit. **Shared scope** = all subs in billing account.

---

## 9. Azure Savings Plans

- **Flexible** compute commitment — commit to $/hour spend for 1 or 3 years
- Up to **65% savings** vs PAYG
- Applies across **multiple compute services** (not locked to specific SKU/region)

### Covered Services

| Service | Covered |
|---|---|
| **Azure VMs** | ✅ All series and sizes |
| **Azure App Service** | ✅ Premium v3, Isolated v2 |
| **Azure Functions** | ✅ Premium plan |
| **Azure Container Instances** | ✅ |
| **Azure Kubernetes Service** | ✅ (compute) |
| **Azure Dedicated Host** | ✅ |

### Reservations vs Savings Plans

| Feature | Reservations | Savings Plans |
|---|---|---|
| **Commitment** | Specific resource type + region + size | $/hour compute spend |
| **Max discount** | Up to **72%** | Up to **65%** |
| **Flexibility** | Limited (same series, instance flexibility) | High (across services, sizes, regions) |
| **Term** | 1 or 3 years | 1 or 3 years |
| **Exchange** | ✅ Yes | ❌ No |
| **Cancel/Return** | ✅ (with fee, $50K cap) | ❌ No |
| **Auto-renew** | ✅ Optional | ✅ Optional |
| **Scope** | Shared / Sub / MG / RG | Shared / Sub / MG |
| **Best for** | Predictable, stable workloads with known SKU | Variable workloads across services |

### Benefit Application Order

```
1. Reservations applied first (most specific)
2. Savings Plans applied next
3. Remaining = billed at PAYG rates
```

### Portal Path — Purchase Savings Plan

```
Savings plans → + Add →
  Select compute service → Term: 1 / 3 years →
  Scope: Shared / Single sub / MG →
  Hourly commitment amount ($) →
  Billing frequency: All upfront / Monthly →
  Review + Purchase
```

> ⚠️ **EXAM TIP:** Savings Plans = more flexible, less discount (65% max). Reservations = less flexible, more discount (72% max). Savings Plans **cannot be exchanged or cancelled** (key difference). Reservation applied **first**, then Savings Plan, then PAYG.

---

## 10. Azure Advisor — Cost Recommendations

### What is Azure Advisor?

- **Free** built-in recommendation engine
- Provides personalized best practice recommendations
- Categories: Cost, Security, Reliability, Operational Excellence, Performance

### Cost Recommendation Types

| Recommendation | Action |
|---|---|
| **Right-size underutilized VMs** | Downgrade or resize VMs with low CPU/memory |
| **Shut down unused VMs** | Identify VMs with near-zero usage |
| **Delete unattached disks** | Remove orphaned managed disks |
| **Delete unused public IPs** | Remove unassociated static PIPs |
| **Purchase reservations** | Buy RIs based on usage patterns |
| **Purchase savings plans** | Commit to savings plan based on compute trends |
| **Use Spot VMs** | Switch interruptible workloads to Spot pricing |
| **Reduce ExpressRoute overprovisioning** | Downgrade underutilized circuits |
| **Reconfigure idle VPN gateways** | Identify low-traffic gateways |
| **Delete unused ExpressRoute circuits** | Remove inactive circuits |

### Portal Path — View Advisor Cost Recommendations

```
Azure Advisor → Cost →
  View all recommendations →
  Select recommendation → View details →
  Impact: High / Medium / Low →
  Estimated savings →
  Quick fix (some) / Remediate
```

### Advisor Alerts

```
Azure Advisor → Alerts → + Create Advisor alert →
  Category: Cost →
  Impact level →
  Action group → Create
```

> ⚠️ **EXAM TIP:** Advisor is **FREE**. Cost recommendations include right-sizing VMs, deleting unused resources, and purchasing reservations. Advisor shows **estimated annual savings** per recommendation. Exam may ask which tool to use for cost optimization → Advisor.

---

## 11. Azure Pricing Tools

| Tool | Purpose | URL/Path |
|---|---|---|
| **Azure Pricing Calculator** | Estimate costs BEFORE deployment | pricing.azure.com |
| **TCO Calculator** | Compare on-prem vs Azure costs | azure.microsoft.com/pricing/tco |
| **Cost Management** | Analyze CURRENT costs | Portal → Cost Management |
| **Azure Advisor** | Optimize EXISTING resources | Portal → Advisor |

### Azure Pricing Calculator

- Estimate costs for resources **before** deploying
- Configure: Region, SKU, tier, quantity, hours/month
- Export estimates as Excel
- Share via link
- NOT inside Azure portal — external website

### TCO (Total Cost of Ownership) Calculator

- Compare **on-premises** infrastructure costs vs **Azure** costs
- Input current workloads (servers, databases, storage, networking)
- Factors: hardware, software, electricity, labor, data center costs
- Generates multi-year cost comparison report
- NOT inside Azure portal — external website

> ⚠️ **EXAM TIP:** **Pricing Calculator** = estimate future costs. **TCO Calculator** = compare on-prem vs Azure. **Cost Management** = analyze current costs. **Advisor** = optimize existing costs. Know which tool for which scenario — commonly tested.

---

## 12. Tags for Cost Management

- Tags **essential** for cost allocation and tracking
- Cost analysis can **group by tags** (e.g., CostCenter, Project, Environment)
- Tags **do NOT inherit** from RG to resources — use Azure Policy to enforce

### Cost-Relevant Tags

| Tag | Purpose |
|---|---|
| `CostCenter` | Allocate costs to business units |
| `Department` | Track spending per department |
| `Project` | Track project-specific costs |
| `Environment` | Separate Prod / Dev / Test costs |
| `Owner` | Accountability for resource costs |
| `Application` | Track per-application spend |

### Enforce Tags with Policy

| Policy | Effect |
|---|---|
| `Require a tag and its value on resources` | Deny — block creation without tag |
| `Inherit a tag from the resource group` | Modify — auto-apply RG tag to resources |
| `Add or replace a tag on resources` | Modify — enforce specific tag value |

> ⚠️ **EXAM TIP:** Tags are **essential** for showback/chargeback. Tags do NOT inherit — use Policy (Modify effect) to auto-inherit. Cost analysis can filter and group by tag. Without tags, cost allocation across teams/projects is very difficult.

---

## 13. RBAC Roles for Cost Management

| Role | Permissions |
|---|---|
| **Cost Management Reader** | View cost data, budgets, exports |
| **Cost Management Contributor** | View + create/manage budgets, exports, recommendations |
| **Billing Reader** | View billing and payment info |
| **Billing Account Reader** | View billing account details |
| **Invoice Section Reader** | View invoice section properties |
| **Owner** | Full access including cost management |
| **Contributor** | Full access including cost management |
| **Reader** | View cost data (via Cost Management Reader inherited) |

### Who Can Do What

| Action | Required Role |
|---|---|
| View cost analysis | Cost Management Reader+ |
| Create/manage budgets | Cost Management Contributor+ |
| Create/manage exports | Cost Management Contributor+ |
| Purchase reservations | Owner or Contributor on subscription |
| View reservations | Reader+ |
| Manage reservations | Owner on reservation order |
| View Advisor recommendations | Reader+ |
| View billing invoices | Billing Reader+ |

### Portal Path — Assign Cost Role

```
Subscription / RG → Access control (IAM) →
  + Add → Add role assignment →
  Role: Cost Management Reader / Cost Management Contributor →
  Members → Select → Review + assign
```

> ⚠️ **EXAM TIP:** **Cost Management Reader** = view only. **Cost Management Contributor** = view + manage budgets/exports. To **purchase** reservations, need **Owner** or **Contributor** on the subscription. Billing Reader is for invoice/payment info, not cost analysis.

---

## 14. Pricing Key Points

| Item | Cost |
|---|---|
| **Cost Management + Billing** | **FREE** for Azure resources |
| **Budgets** | **FREE** |
| **Cost alerts** | **FREE** |
| **Cost exports** | **FREE** service (Storage charges apply) |
| **Azure Advisor** | **FREE** |
| **Azure Pricing Calculator** | **FREE** |
| **TCO Calculator** | **FREE** |
| **Reservations** | Discounted prepaid pricing (up to 72% off) |
| **Savings Plans** | Discounted commitment pricing (up to 65% off) |
| **Cost Management for AWS** | Requires connector — additional charges |

---

## 15. Quick-Fire Exam Points ⚡

1. **Cost Management** is **FREE** for Azure resources — no additional charges
2. **Budget** = alert only, does **NOT stop** spending
3. **Spending limit** = hard stop, disables resources — only for credit-based subs (Free/MSDN)
4. Spending limit once removed permanently → **cannot be re-enabled**
5. PAYG, EA, CSP = **no spending limit** available
6. Budget alert types: **Actual** (after) and **Forecasted** (before threshold hit)
7. Budget alerts can trigger **Action Groups** for automation
8. **Anomaly alerts** = ML-based, automatic, no setup required
9. **Credit alerts** = EA only, fire at 90% and 100% of credit consumed
10. Cost exports go to **Storage Account** in **CSV** format
11. Export types: **Actual cost**, **Amortized cost**, **Usage**
12. **Amortized** = reservation costs spread evenly over the term
13. **Azure Reservations** = up to **72% savings**, 1/3 year terms
14. **Azure Savings Plans** = up to **65% savings**, 1/3 year terms
15. Reservations **can be exchanged** (equal or greater value)
16. Reservations **can be cancelled** (up to $50K/12 months, 12% fee)
17. Savings Plans **cannot be exchanged or cancelled** (key difference!)
18. Reservation benefit application order: **Reservations → Savings Plans → PAYG**
19. Instance size flexibility = ON by default for VMs — reservation applies within same series
20. Reservation scope: **Shared** / Single sub / MG / RG
21. Savings Plan scope: **Shared** / Single sub / MG (no RG scope)
22. **Azure Advisor** = FREE, provides cost optimization recommendations
23. Advisor cost recommendations: right-size VMs, shutdown unused, delete unattached disks, buy RIs
24. **Pricing Calculator** = estimate costs BEFORE deployment (external website)
25. **TCO Calculator** = compare on-prem vs Azure costs (external website)
26. **Cost Management** = analyze CURRENT/past costs (inside Azure portal)
27. **Tags** are essential for cost allocation — Cost Analysis can group by tag
28. Tags **do NOT inherit** from RG to resources — use Azure Policy (Modify)
29. **Cost Management Reader** = view cost data only
30. **Cost Management Contributor** = view + manage budgets, exports
31. To purchase reservations = **Owner or Contributor** on subscription
32. Budget alert processing delay = typically **8–24 hours**
33. Cost analysis granularity: **Daily**, **Monthly**, **None** (total)
34. Cost analysis can save **custom views** and pin to dashboards
35. Forecast uses **machine learning** to predict future costs
36. EA cost data delay = up to **24 hours**; PAYG = near real-time
37. Budget reset periods: Monthly / Quarterly / Annually / Billing variants
38. Budget filters: Resource group, Resource, Meter, Tag, etc.
39. Budgets can be set at MG, Subscription, or RG scope
40. Max tags per resource = **50**; key = 512 chars; value = 256 chars
41. Common cost tags: CostCenter, Department, Project, Owner, Environment
42. Policy `Require a tag on resources` (Deny) = block untagged deployments
43. Policy `Inherit a tag from the resource group` (Modify) = auto-copy RG tag
44. Reservation payment: **All upfront** (best discount) / **Monthly** / **No upfront** (some RIs)
45. Savings Plan payment: **All upfront** or **Monthly** only
46. Spot VMs = up to **90% savings** but can be evicted any time (Advisor may recommend)
47. Cost Management supports **AWS** via connector (not in AZ-104 scope but know it exists)
48. **Delete unused Public IPs** = common Advisor recommendation (static PIPs cost money)
49. **Unattached managed disks** continue incurring charges — delete if unused
50. Cost exports can be created at **Subscription**, **RG**, or **MG** scope

---

## 16. Step-by-Step Configuration Mind Maps 🗺️

---

### 16.1 Create a Budget with Alerts

> **Portal:** `Cost Management → Budgets → + Add`

```
Create Budget
│
├── Prerequisites
│   ├── Cost Management Contributor (or Contributor/Owner) at target scope
│   └── Target scope exists (Subscription / RG / MG)
│
├── Step 1: Navigate
│   └── Cost Management → Budgets → + Add
│
├── Step 2: Budget Details
│   ├── Name (unique within scope)
│   ├── Scope: Subscription / Resource Group / Management Group
│   ├── Reset period:
│   │   ├── Monthly / Quarterly / Annually
│   │   └── Billing month / Billing quarter / Billing year (EA)
│   ├── Start date (first day of tracking)
│   ├── Expiration date (when to stop)
│   └── Budget amount ($)
│
├── Step 3: Filters (optional)
│   ├── Resource group (limit to specific RGs)
│   ├── Resource (limit to specific resources)
│   ├── Meter category / Meter
│   └── Tag (key:value filter)
│
├── Step 4: Alert Conditions (Next tab)
│   ├── + Add alert condition
│   │   ├── Type: Actual / Forecasted
│   │   │   ├── Actual = fires AFTER threshold crossed
│   │   │   └── Forecasted = fires BEFORE (predicted to cross)
│   │   └── % of budget (e.g., 50, 80, 100, 120)
│   ├── Alert recipients (email addresses — up to 20)
│   ├── Action group (optional)
│   │   └── ⚠️ Use to trigger automation: Logic App, Function, Runbook
│   └── Language preference for alerts
│
├── Step 5: Create
│
├── Required Role: Cost Management Contributor+
│
└── ⚠️ Key Points
    ├── Budget does NOT stop spending — alerts only
    ├── Forecasted alerts give early warning
    ├── Add Action Group to automate responses (e.g., VM shutdown)
    ├── Alerts typically fire within 8-24 hours of threshold
    ├── Can set multiple thresholds (50%, 80%, 100%)
    └── Budgets are FREE
```

---

### 16.2 Create a Cost Export

> **Portal:** `Cost Management → Exports → + Add`

```
Create Cost Export
│
├── Prerequisites
│   ├── Cost Management Contributor+ at scope
│   ├── Storage Account exists in same Azure tenant
│   └── Contributor on Storage Account (to write data)
│
├── Step 1: Navigate
│   └── Cost Management → Exports → + Add
│
├── Step 2: Export Details
│   ├── Name
│   ├── Metric type:
│   │   ├── Actual cost (charges as billed)
│   │   ├── Amortized cost (RI costs spread over term)
│   │   └── Usage (raw consumption)
│   └── Export type:
│       ├── Daily export of month-to-date costs
│       ├── Weekly export
│       ├── Monthly export of last month
│       └── One-time export (custom date range)
│
├── Step 3: Storage Destination
│   ├── Subscription (where Storage Account lives)
│   ├── Storage Account
│   ├── Container
│   └── Directory (folder path within container)
│
├── Step 4: Create
│
├── Required Role: Cost Management Contributor + Contributor on Storage Account
│
└── ⚠️ Key Points
    ├── Export service = FREE; you pay for Storage Account
    ├── Data exported as CSV files
    ├── Amortized view useful when using reservations
    ├── Can trigger manually via "Run now"
    └── Use for Power BI, external tools, long-term retention
```

---

### 16.3 Purchase an Azure Reservation

> **Portal:** `Reservations → + Add`

```
Purchase Reservation
│
├── Prerequisites
│   ├── Owner or Contributor role on subscription
│   ├── EA: must have reserved instance purchasing enabled
│   └── Understand your usage patterns first (check Advisor)
│
├── Step 1: Navigate
│   └── Home → Reservations → + Add
│
├── Step 2: Select Product
│   ├── Virtual Machines / SQL Database / Cosmos DB /
│   │   Storage / App Service / Dedicated Host / etc.
│   └── Select → Configure
│
├── Step 3: Configure
│   ├── Scope:
│   │   ├── Shared (all subs in billing account)
│   │   ├── Single subscription
│   │   ├── Single resource group
│   │   └── Management Group
│   ├── Region
│   ├── Size / SKU
│   │   └── ⚠️ Instance size flexibility ON by default for VMs
│   ├── Term: 1 year / 3 years
│   │   └── ⚠️ 3-year = greater discount than 1-year
│   ├── Quantity
│   └── Billing frequency: All upfront / Monthly / No upfront
│       └── ⚠️ All upfront = greatest discount
│
├── Step 4: Review + Purchase
│
├── Required Role: Owner or Contributor on subscription + billing permissions
│
└── ⚠️ Key Points
    ├── Up to 72% savings vs PAYG
    ├── Can exchange for equal or greater value
    ├── Can cancel with 12% fee (up to $50K/12 months)
    ├── Instance flexibility allows coverage across sizes in same series
    ├── Benefit applied automatically to matching resources
    ├── Shared scope = discount applies across all subscriptions
    └── Check Advisor for purchase recommendations first
```

---

### 16.4 Purchase a Savings Plan

> **Portal:** `Savings plans → + Add`

```
Purchase Savings Plan
│
├── Prerequisites
│   ├── Owner or Contributor role on subscription
│   └── Understand compute spending patterns
│
├── Step 1: Navigate
│   └── Home → Savings plans → + Add
│
├── Step 2: Configure
│   ├── Compute service: All compute services / Specific
│   ├── Term: 1 year / 3 years
│   ├── Scope:
│   │   ├── Shared (all subs in billing account)
│   │   ├── Single subscription
│   │   └── Management Group
│   │   └── ⚠️ No RG scope (unlike Reservations)
│   ├── Hourly commitment amount ($/hr)
│   └── Billing frequency: All upfront / Monthly
│       └── ⚠️ No "No upfront" option for Savings Plans
│
├── Step 3: Review + Purchase
│
├── Required Role: Owner or Contributor on subscription
│
└── ⚠️ Key Points
    ├── Up to 65% savings vs PAYG
    ├── CANNOT be exchanged or cancelled (critical!)
    ├── More flexible than reservations (across services, sizes, regions)
    ├── Applied AFTER reservations in discount hierarchy
    ├── Best for variable compute workloads
    └── Auto-renew available
```

---

### 16.5 View and Act on Advisor Cost Recommendations

> **Portal:** `Azure Advisor → Cost`

```
View Advisor Cost Recommendations
│
├── Step 1: Navigate
│   └── Home → Azure Advisor → Cost tab
│
├── Step 2: View Recommendations
│   ├── Grouped by type:
│   │   ├── Right-size or shutdown underutilized VMs
│   │   ├── Purchase reserved instances
│   │   ├── Delete unattached managed disks
│   │   ├── Delete unused public IP addresses
│   │   ├── Reconfigure idle VPN gateways
│   │   └── Other service-specific recommendations
│   │
│   ├── Each shows:
│   │   ├── Impact: High / Medium / Low
│   │   ├── Estimated annual savings ($)
│   │   ├── Affected resources
│   │   └── Quick Fix (for some — auto-remediate)
│   │
│   └── Select recommendation → View details → Take action
│
├── Step 3: Set Up Advisor Alerts (optional)
│   └── Advisor → Alerts → + Create Advisor alert →
│       Category: Cost → Impact → Action group → Create
│
├── Required Role: Reader+ (to view); Contributor+ (to remediate)
│
└── ⚠️ Key Points
    ├── Advisor is FREE
    ├── Recommendations refresh periodically (not real-time)
    ├── Can dismiss / postpone recommendations
    ├── Can configure alert rules for new recommendations
    └── Advisor also covers Security, Reliability, Performance, OpsExcellence
```

---

### 16.6 Analyze Costs with Cost Analysis

> **Portal:** `Cost Management → Cost analysis`

```
Analyze Costs
│
├── Step 1: Navigate
│   └── Cost Management → Cost analysis
│
├── Step 2: Select Scope
│   ├── Management Group / Subscription / Resource Group
│   └── ⚠️ Scope determines what data you see
│
├── Step 3: Configure View
│   ├── Date range: Last 7 days / This month / Last 3 months / Custom
│   ├── Granularity: Daily / Monthly / None
│   ├── View type:
│   │   ├── Accumulated costs (running total)
│   │   ├── Cost by service
│   │   ├── Cost by resource
│   │   └── Daily costs
│   ├── Group by:
│   │   ├── Service name / Resource / Resource group
│   │   ├── Tag / Location / Meter
│   │   └── Subscription (at MG scope)
│   ├── Add filter: Tag, Resource type, Location, RG
│   └── Chart type: Area / Column / Line / Table / Donut
│
├── Step 4: Actions
│   ├── Download: CSV / PNG / Excel
│   ├── Save view: Name → Save (reusable)
│   ├── Pin to dashboard
│   └── Share: Generate link
│
├── Step 5: Forecast tab
│   └── View ML-based projected costs
│
├── Required Role: Cost Management Reader+
│
└── ⚠️ Key Points
    ├── Group by TAG = how you do showback/chargeback
    ├── Forecast uses ML — shows predicted monthly spend
    ├── Save custom views for recurring analysis
    ├── PAYG data = near real-time; EA = up to 24 hours delay
    └── Data available for last 13 months of billing history
```



---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
