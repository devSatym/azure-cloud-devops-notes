<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Log Analytics — AZ-104 Revision Notes

---

## 1. What is Azure Log Analytics?

- **Query and analytics engine** for Azure Monitor Logs
- Central **Log Analytics Workspace (LAW)** stores all log data from Azure, on-prem, and multi-cloud
- Uses **Kusto Query Language (KQL)** to query data
- **Not a separate service** — it is the **query and storage layer** within Azure Monitor
- Workspace = the **data repository** for log data; Log Analytics = the **query tool** (in portal)
- All logs collected via Diagnostic Settings, Agents (AMA), or direct API go into a workspace

---

## 2. Key Components

| Component | Purpose |
|---|---|
| **Log Analytics Workspace** | Central store/repository for all log data |
| **KQL (Kusto Query Language)** | Query language to search, filter, aggregate log data |
| **Tables** | Data stored in structured tables (e.g., Heartbeat, Perf, Event, Syslog) |
| **Data Collection Rules (DCR)** | Define what data to collect and where to send it |
| **Azure Monitor Agent (AMA)** | Collects guest OS data and sends to workspace |
| **Solutions / Insights** | Pre-built analytics (VM Insights, Container Insights, etc.) |
| **Saved Queries** | Reusable KQL queries stored in workspace |
| **Query Packs** | Grouping of saved queries, shareable across workspaces |
| **Workbooks** | Interactive visual reports powered by KQL queries |
| **Log Search Alerts** | Alerts triggered by KQL queries on log data |

---

## 3. Log Analytics Workspace — Architecture

- **One workspace per region** is common best practice (but multi-workspace is supported)
- Workspace is a **unique Azure resource** (lives in a resource group + region)
- Can ingest data from **multiple subscriptions and tenants**
- Workspace **name must be globally unique**
- **Cannot move workspace** between regions after creation
- Each workspace has its own set of **tables, permissions, retention settings, pricing tier**

### Workspace Design Considerations

| Factor | Guidance |
|---|---|
| **Region** | Same region as resources to reduce latency & egress costs |
| **Access control** | Resource-context vs workspace-context RBAC |
| **Data volume** | Higher volume → consider commitment tiers |
| **Regulatory / compliance** | Separate workspaces for data sovereignty |
| **Network isolation** | Use Private Link for secure access |
| **Multi-tenant** | Separate workspaces per tenant recommended |

> ⚠️ **EXAM TIP:** Workspace name = **globally unique across all Azure subscriptions**. Cannot be reused for 14 days after deletion (soft-delete period).

---

## 4. Data Sources → Log Analytics Workspace

| Data Source | How It Gets to Workspace | Agent/Setting Needed |
|---|---|---|
| **Platform Metrics** | Diagnostic Settings | Diagnostic Settings (per resource) |
| **Resource Logs** | Diagnostic Settings | Diagnostic Settings (per resource) |
| **Activity Log** | Diagnostic Settings (subscription-level) | Diagnostic Setting export |
| **Guest OS Perf Counters** | Azure Monitor Agent (AMA) | AMA + DCR |
| **Windows Event Logs** | Azure Monitor Agent (AMA) | AMA + DCR |
| **Linux Syslog** | Azure Monitor Agent (AMA) | AMA + DCR |
| **Custom Text Logs** | Azure Monitor Agent (AMA) | AMA + DCR |
| **IIS Logs** | Azure Monitor Agent (AMA) | AMA + DCR |
| **Application Insights** | Workspace-based App Insights | App Insights resource config |
| **Microsoft Sentinel** | Connects to workspace | Sentinel enabled on workspace |
| **Azure AD / Entra ID Logs** | Diagnostic Settings | Diagnostic Settings |
| **Custom Logs API** | HTTP Data Collector API / Logs Ingestion API | DCR + endpoint |

> ⚠️ **EXAM TIP:** **Diagnostic Settings** route platform/resource logs. **AMA + DCR** route guest OS data. These are the two primary ingestion paths for AZ-104.

---

## 5. Log Tables — Types & Categories

### Table Types

| Table Type | Description | Retention | Query Cost |
|---|---|---|---|
| **Analytics** | Default; full KQL, alerts, visualizations | Configurable (30–730 days interactive) | Standard |
| **Basic** | Reduced cost; limited KQL (8 days query, no alerts) | 8 days interactive + 30 days total minimum | Lower ingestion, per-query charge |
| **Archive** | Long-term cold storage; restored on-demand | Up to **12 years** (4,383 days) total | Restore job or search job charge |

### Key Standard Tables

| Table | Data | Source |
|---|---|---|
| `Heartbeat` | Agent connectivity (heartbeat every 1 min) | AMA / MMA |
| `Perf` | Performance counters (CPU, Memory, Disk, Network) | AMA / MMA |
| `Event` | Windows Event Log entries | AMA / MMA |
| `Syslog` | Linux Syslog messages | AMA / MMA |
| `SecurityEvent` | Windows Security Events (logon, process, etc.) | AMA / MMA |
| `AzureActivity` | Azure Activity Log (control-plane operations) | Diagnostic Settings |
| `AzureDiagnostics` | Resource diagnostic logs (legacy single table) | Diagnostic Settings |
| `AzureMetrics` | Platform metrics sent to workspace | Diagnostic Settings |
| `InsightsMetrics` | VM Insights / Container Insights metrics | AMA |
| `Update` | Update management data | Update Management |
| `Alert` | Fired alert records | Azure Monitor |
| `Usage` | Workspace data volume usage per table | System |
| `Operation` | Workspace operational events | System |

> ⚠️ **EXAM TIP:** **Basic Logs** = cheaper ingestion, only 8 days queryable, no alert rules, per-query scan charge. Good for verbose/debug logs. **Analytics** = full-featured, default.

---

## 6. Retention & Archive

### Retention Tiers

| Tier | Duration | Description |
|---|---|---|
| **Interactive retention (hot)** | 30–730 days | Full KQL query, alerts, visualizations |
| **Total retention (archive/cold)** | Up to **12 years** (4,383 days) | Must restore or use search job to query |

### Key Retention Facts

- **Default interactive retention** = **30 days**
- **First 31 days** of retention = **FREE** (included in ingestion cost)
- Beyond 31 days → charged per GB/month for interactive retention
- Archive = lower cost than interactive; data must be **restored** to query
- Retention is configured **per workspace** (applies to all tables by default)
- Can set **per-table retention** (override workspace default)
- **SecurityEvent & Usage tables** = can have different retention than workspace

### Restore & Search Jobs

| Feature | Restore | Search Job |
|---|---|---|
| **Purpose** | Bring archived data back to hot tier | Search archived data without full restore |
| **Duration** | Creates temp table (2–730 days) | Returns results as a table |
| **Query** | Full KQL on restored data | Limited KQL during search |
| **Cost** | Per GB restored + compute | Per GB scanned |
| **Max data** | Up to 60 TB per restore | Up to entire archive |
| **SLA** | Restore can take hours | Async; results in a table |

### Portal Path — Set Retention
```
Log Analytics Workspace → Settings → Usage and estimated costs →
Data Retention → Set slider (30–730 days) → OK
```

### Portal Path — Per-Table Retention
```
Log Analytics Workspace → Settings → Tables →
Select table → Configure retention (interactive + total) → Save
```

> ⚠️ **EXAM TIP:** Default retention = **30 days**. Free = first **31 days**. Max interactive = **730 days**. Max total (archive) = **12 years (4,383 days)**. Per-table override is possible.

---

## 7. KQL (Kusto Query Language) — Exam Essentials

### Core KQL Operators

| Operator | Purpose | Example |
|---|---|---|
| `where` | Filter rows | `Heartbeat \| where TimeGenerated > ago(1h)` |
| `summarize` | Aggregate data | `Perf \| summarize avg(CounterValue) by Computer` |
| `project` | Select/rename columns | `Event \| project TimeGenerated, Computer` |
| `extend` | Add calculated column | `Perf \| extend GB = CounterValue / 1024` |
| `count` | Count rows | `SecurityEvent \| count` |
| `top` | First N results | `Perf \| top 10 by CounterValue desc` |
| `order by` / `sort by` | Sort results | `Event \| order by TimeGenerated desc` |
| `distinct` | Unique values | `Heartbeat \| distinct Computer` |
| `render` | Visualize | `\| render timechart` / `\| render barchart` |
| `join` | Join two tables | `Heartbeat \| join (Perf) on Computer` |
| `union` | Combine tables | `union Event, Syslog` |
| `take` / `limit` | Return N random rows | `Event \| take 10` |
| `search` | Full-text search across tables | `search "error"` |
| `let` | Declare variables | `let threshold = 90;` |
| `ago()` | Relative time | `ago(24h)`, `ago(7d)`, `ago(30m)` |
| `bin()` | Time bucketing | `bin(TimeGenerated, 1h)` |
| `between` | Range filter | `where CounterValue between (50 .. 100)` |
| `contains` / `has` | String matching | `where Message contains "error"` |
| `startswith` / `endswith` | String prefix/suffix | `where Computer startswith "Web"` |
| `!contains` / `!has` | Negation | `where Message !contains "info"` |

### KQL Query Structure
```
TableName
| where <condition>
| summarize <aggregation> by <grouping>
| project <columns>
| order by <column> desc
| render <visualization>
```

### Common Exam KQL Patterns

```kql
// Count VMs with heartbeat in last 1 hour
Heartbeat | where TimeGenerated > ago(1h) | distinct Computer | count

// Average CPU by VM
Perf | where CounterName == "% Processor Time"
| summarize avg(CounterValue) by Computer | order by avg_CounterValue desc

// Find failed logons
SecurityEvent | where EventID == 4625
| summarize count() by Account, Computer

// Data ingested per table (workspace usage)
Usage | summarize DataGB = sum(Quantity) / 1024 by DataType
| order by DataGB desc

// Activity log - resource deletions
AzureActivity | where OperationNameValue contains "DELETE"
| project TimeGenerated, Caller, ResourceGroup, OperationNameValue
```

> ⚠️ **EXAM TIP:** KQL is **case-sensitive** for table and column names. Pipe `|` chains operators. `has` is faster than `contains` (word boundary match vs substring). `ago()` for relative time.

---

## 8. Data Collection Rules (DCR)

- **Central configuration** defining what data to collect and where to send
- Used with **Azure Monitor Agent (AMA)**
- DCR is an **Azure resource** itself (has resource ID, lives in a resource group)
- **Many-to-many** relationship: one DCR → many VMs, one VM → many DCRs
- Supports **transformations** (KQL-based filtering/modification of data before ingestion)

### DCR Components

| Component | Description |
|---|---|
| **Data Sources** | What to collect (Perf counters, Event logs, Syslog, Custom text logs, IIS logs) |
| **Destinations** | Where to send (Log Analytics workspace, Azure Monitor Metrics) |
| **Data Flows** | Maps sources to destinations |
| **Transformations** | KQL queries to filter/modify data in-stream (reduce ingestion cost) |

### Portal Path — Create DCR
```
Azure Monitor → Settings → Data Collection Rules → + Create →
Rule Name, Subscription, RG, Region, Platform Type (Windows/Linux) →
Add Resources (select VMs — auto-installs AMA) →
Add Data Source (Perf counters / Win Event Logs / Syslog / Custom logs / IIS logs) →
Add Destination (Log Analytics Workspace) →
Review + Create → Create
```

> ⚠️ **EXAM TIP:** When you associate a VM with a DCR, AMA is **automatically installed** if not already present. DCR **transformations** can filter data before ingestion to reduce cost.

---

## 9. Azure Monitor Agent (AMA) — Log Analytics Context

### AMA vs Legacy Log Analytics Agent (MMA)

| Feature | AMA (Current) | MMA / OMS (Deprecated) |
|---|---|---|
| **Status** | ✅ Current, recommended | ❌ Deprecated (Aug 2024) |
| **Configuration** | Data Collection Rules (DCR) | Workspace-level (connected sources) |
| **Multi-homing** | Multiple DCRs → multiple workspaces | Manual multi-homing |
| **Identity** | Managed Identity (System or User) | Workspace key-based |
| **Filtering** | XPath for Event Logs, DCR transforms | Limited |
| **Custom Logs** | DCR-based custom text logs | Custom log wizard |
| **Supports** | Azure VMs, Arc, VMSS | Azure VMs, on-prem (direct connect) |
| **Extension name** | `AzureMonitorWindowsAgent` / `AzureMonitorLinuxAgent` | `MicrosoftMonitoringAgent` / `OmsAgentForLinux` |

### Legacy Agent Workspace Connection (still exam-relevant)
```
Log Analytics Workspace → Settings → Agents →
Windows/Linux → Download Agent → Workspace ID + Primary Key →
Install on machine → Agent connects to workspace
```

> ⚠️ **EXAM TIP:** MMA is **deprecated since Aug 2024** but still tested for migration awareness. AZ-104 focuses on **AMA + DCR**. When a VM is associated with a DCR, AMA is **auto-installed**.

---

## 10. Workspace Access Control

### Access Control Modes

| Mode | Description | Scope |
|---|---|---|
| **Workspace-context** | Permissions set at workspace level; user sees ALL data in workspace | Workspace-level RBAC |
| **Resource-context** | Permissions based on Azure resource RBAC; users see logs only for resources they have access to | Resource-level RBAC |

- **Resource-context** = default for new workspaces
- Allows fine-grained access without managing workspace-level permissions
- Users query from the **resource blade** → see only that resource's logs
- Users with workspace-level access → can query via workspace blade and see all data

### Portal Path — Set Access Control Mode
```
Log Analytics Workspace → Settings → Properties →
Access control mode: "Use resource or workspace permissions" (resource-context)
                  or "Require workspace permissions" (workspace-context)
→ Save
```

### Table-Level RBAC

- Grant/deny access to **specific tables** within a workspace
- Useful when certain tables contain sensitive data (e.g., SecurityEvent)
- Configured via custom RBAC roles with `Microsoft.OperationalInsights/workspaces/tables/read`

> ⚠️ **EXAM TIP:** **Resource-context** = users see only logs for resources they have RBAC access to. **Workspace-context** = users see ALL data if they have workspace role. Resource-context is the default.

---

## 11. Security & RBAC

### Built-in Roles

| Role | Permissions |
|---|---|
| **Log Analytics Reader** | Read log data, query logs, view saved searches, view workspace settings |
| **Log Analytics Contributor** | Read + configure workspace (retention, pricing tier, table config, solutions, DCR association) |
| **Monitoring Reader** | Read all monitoring data (metrics, logs, alerts, settings) |
| **Monitoring Contributor** | Read + create/modify alert rules, action groups, diagnostic settings, DCRs |

### Permission Matrix

| Action | Required Role |
|---|---|
| Run KQL queries (read logs) | Log Analytics Reader (or Monitoring Reader) |
| View workspace properties | Log Analytics Reader |
| Change retention settings | Log Analytics Contributor |
| Change pricing tier | Log Analytics Contributor |
| Create/modify DCRs | Monitoring Contributor |
| Configure Diagnostic Settings | Monitoring Contributor (or resource Contributor) |
| Install AMA on VMs | Virtual Machine Contributor |
| Create workspace | Contributor on RG |
| Delete workspace | Contributor on workspace |
| Create saved queries | Log Analytics Contributor |
| Manage query packs | Log Analytics Contributor |

### Workspace Network Isolation (Private Link)

- Use **Azure Private Link** to connect workspace to VNet
- **Azure Monitor Private Link Scope (AMPLS)** = resource that groups workspaces + App Insights for private connectivity
- Data ingestion and query go through private endpoints
- Can block public access after Private Link is configured

### Portal Path — Private Link
```
Azure Monitor → Settings → Private Link Scopes → + Create →
Name, Subscription, RG → Create →
Add Resources (Log Analytics workspace / App Insights) →
Create Private Endpoint (select VNet, Subnet) → Create
```

> ⚠️ **EXAM TIP:** **Log Analytics Reader** = read/query only. **Log Analytics Contributor** = full workspace config. For **network isolation**, use **AMPLS + Private Endpoint**. Know the RBAC difference from Monitoring Reader/Contributor.

---

## 12. Workspace Pricing

### Pricing Tiers

| Tier | Description | Commitment |
|---|---|---|
| **Pay-As-You-Go (PerGB2018)** | Per GB ingested; default tier | None |
| **Commitment Tier 100 GB/day** | Fixed daily rate for 100 GB | 31-day minimum |
| **Commitment Tier 200 GB/day** | Fixed daily rate for 200 GB | 31-day minimum |
| **Commitment Tier 300 GB/day** | Fixed daily rate for 300 GB | 31-day minimum |
| **Commitment Tier 400 GB/day** | Fixed daily rate for 400 GB | 31-day minimum |
| **Commitment Tier 500 GB/day** | Fixed daily rate for 500 GB | 31-day minimum |
| **Commitment Tier 1000+ GB/day** | Higher tiers available (1000, 2000, 5000) | 31-day minimum |

### Cost Components

| Component | Pricing |
|---|---|
| **Data Ingestion (Analytics)** | Per GB (Pay-As-You-Go) or Commitment Tier |
| **Data Ingestion (Basic Logs)** | ~50% cheaper than Analytics tables |
| **Interactive Retention** | First **31 days FREE**; then per GB/month |
| **Archive (Total Retention)** | Much cheaper than interactive; per GB/month |
| **Restore** | Per GB restored + compute charge |
| **Search Job** | Per GB scanned |
| **Data Export** | Per GB exported |
| **Log query (Basic Logs)** | Per GB scanned during query |

### Portal Path — Change Pricing Tier
```
Log Analytics Workspace → Settings → Usage and estimated costs →
Pricing Tier → Select tier → OK
```

### Daily Cap

- **Daily cap** = maximum GB ingested per day (stops ingestion when reached)
- Default = **no cap** (unlimited)
- Reset time = **UTC midnight** (or custom time)
- Use to prevent unexpected cost spikes
- **Not recommended for production** — you lose data when cap is hit

### Portal Path — Set Daily Cap
```
Log Analytics Workspace → Settings → Usage and estimated costs →
Daily Cap → Set daily cap (GB) → Set reset time → OK
```

> ⚠️ **EXAM TIP:** **Commitment tiers** require **31-day minimum**. Can only upgrade tier before 31 days; downgrade only after 31 days. **Daily cap** stops ingestion (data loss!) — avoid in production.

---

## 13. Querying Logs — Portal Access Points

### Where to Run KQL Queries

| Location | Portal Path | Scope |
|---|---|---|
| **Workspace** | `Log Analytics Workspace → Logs` | All data in workspace |
| **Resource** | `Resource → Monitoring → Logs` | Only that resource's data (resource-context) |
| **Azure Monitor** | `Azure Monitor → Logs` | Select workspace(s) to query |
| **Resource Group** | `Resource Group → Monitoring → Logs` | All resources in RG (if in same workspace) |

### Saved Queries & Query Packs

- **Saved Queries** = saved in workspace, visible to all workspace users
- **Query Packs** = saved as Azure resources, shareable across workspaces/subscriptions
- **Default Query Pack** = created automatically per subscription
- Categorized by type (Performance, Security, etc.)

### Portal Path — Save Query
```
Log Analytics Workspace → Logs → Run query →
Save → Save as query →
Name, Description, Category → Select Query Pack → Save
```

### Portal Path — Manage Query Packs
```
Home → Search "Log Analytics query packs" →
Select query pack → View/Edit queries
```

> ⚠️ **EXAM TIP:** You can query from **3 access points** — Workspace (all data), Resource (resource-context), Azure Monitor (select workspace). Resource-context respects Azure RBAC.

---

## 14. Log Analytics Workspace — Key Settings

### Settings Summary

| Setting | Location | Default |
|---|---|---|
| **Retention** | Usage and estimated costs → Data Retention | 30 days |
| **Daily Cap** | Usage and estimated costs → Daily Cap | No cap |
| **Pricing Tier** | Usage and estimated costs → Pricing Tier | Pay-As-You-Go |
| **Access Control Mode** | Properties | Resource or workspace permissions |
| **Network Isolation** | Network Isolation | Public access enabled |
| **Data Export** | Settings → Data Export | Not configured |
| **Tables** | Settings → Tables | All Analytics type |
| **Agents** | Settings → Agents | Shows connected agents |
| **Connected Sources** | Legacy (Agents section) | VMs, Arc servers |

### Data Export Rules

- **Continuously export** workspace data to Storage Account or Event Hub
- Near real-time export (streaming)
- Can export specific tables only
- Max **10 export rules** per workspace
- Data types that **cannot be exported**: Usage, Heartbeat, AzureActivity (some)

### Portal Path — Data Export
```
Log Analytics Workspace → Settings → Data Export →
+ New export rule → Name → Select tables →
Select destination (Storage Account / Event Hub) → Save
```

---

## 15. Solutions & Integrations

### Common Solutions (Exam-Relevant)

| Solution | Purpose | Installed From |
|---|---|---|
| **VM Insights** | VM performance and dependency mapping | Azure Monitor → Insights → VMs |
| **Container Insights** | AKS/container monitoring | Azure Monitor → Insights → Containers |
| **Network Insights** | Network topology and performance | Azure Monitor → Insights → Networks |
| **Microsoft Sentinel** | SIEM & SOAR; connects to workspace | Separate service, same workspace |
| **Update Management** | Patch compliance and deployment | Automation Account linked to workspace |
| **Change Tracking** | Track file and registry changes on VMs | Automation Account linked to workspace |

### Cross-Workspace Queries

- Query **multiple workspaces** in a single KQL query
- Syntax: `workspace("workspaceName").TableName` or `workspace("workspaceId").TableName`
- Also supports cross-resource queries with Application Insights: `app("appName").requests`

```kql
// Cross-workspace query example
union
  workspace("Workspace1").Heartbeat,
  workspace("Workspace2").Heartbeat
| distinct Computer
```

> ⚠️ **EXAM TIP:** **Cross-workspace queries** use `workspace("name").Table` syntax. Useful when data is in multiple workspaces. User needs **Reader** on all queried workspaces.

---

## 16. CLI & PowerShell Commands (Exam-Relevant)

### Azure CLI

```bash
# Create workspace
az monitor log-analytics workspace create \
  --resource-group <RG> --workspace-name <Name> --location <Region>

# Show workspace details
az monitor log-analytics workspace show \
  --resource-group <RG> --workspace-name <Name>

# List workspaces in subscription
az monitor log-analytics workspace list

# Update retention
az monitor log-analytics workspace update \
  --resource-group <RG> --workspace-name <Name> --retention-time 90

# Delete workspace (soft-delete for 14 days)
az monitor log-analytics workspace delete \
  --resource-group <RG> --workspace-name <Name>

# Recover soft-deleted workspace (within 14 days)
az monitor log-analytics workspace recover \
  --resource-group <RG> --workspace-name <Name>

# Run a KQL query
az monitor log-analytics query \
  --workspace <WorkspaceId> --analytics-query "Heartbeat | count"

# List tables
az monitor log-analytics workspace table list \
  --resource-group <RG> --workspace-name <Name>

# Set table retention
az monitor log-analytics workspace table update \
  --resource-group <RG> --workspace-name <Name> \
  --name <TableName> --retention-time 90 --total-retention-time 365

# Create DCR (Data Collection Rule)
az monitor data-collection rule create \
  --resource-group <RG> --name <DCR-Name> --location <Region> \
  --rule-file <dcr-config.json>
```

### Azure PowerShell

```powershell
# Create workspace
New-AzOperationalInsightsWorkspace -ResourceGroupName <RG> `
  -Name <Name> -Location <Region> -Sku PerGB2018

# Get workspace
Get-AzOperationalInsightsWorkspace -ResourceGroupName <RG> -Name <Name>

# Set retention
Set-AzOperationalInsightsWorkspace -ResourceGroupName <RG> `
  -Name <Name> -RetentionInDays 90

# Delete workspace
Remove-AzOperationalInsightsWorkspace -ResourceGroupName <RG> -Name <Name>

# Run query
Invoke-AzOperationalInsightsQuery -WorkspaceId <Id> `
  -Query "Heartbeat | distinct Computer | count"

# List linked solutions
Get-AzOperationalInsightsIntelligencePack -ResourceGroupName <RG> `
  -WorkspaceName <Name>
```

> ⚠️ **EXAM TIP:** `az monitor log-analytics query` runs KQL from CLI. Workspace deletion = **soft-delete for 14 days** (recoverable). Pay-As-You-Go SKU = `PerGB2018`.

---

## 17. Monitoring the Workspace Itself

### Key Monitoring Points

| What to Monitor | How | Table/Metric |
|---|---|---|
| **Data ingestion volume** | `Usage` table or Usage and estimated costs blade | `Usage \| summarize sum(Quantity) by DataType` |
| **Ingestion latency** | Metrics → Ingestion latency | Platform Metric |
| **Agent health** | `Heartbeat` table (look for missing heartbeats) | `Heartbeat \| summarize max(TimeGenerated) by Computer` |
| **Daily cap reached** | Activity Log → look for operation `Data collection stopped` | AzureActivity |
| **Query performance** | `LAQueryLogs` table (if enabled) | Enable via Diagnostic Settings |

### Workspace Health Alerts (Recommended)

| Alert | Condition |
|---|---|
| **Ingestion spike** | Daily data volume > baseline + threshold |
| **Agent disconnect** | No heartbeat for > 15 min from a computer |
| **Daily cap approaching** | Data volume approaching configured cap |
| **Ingestion latency** | Latency metric > threshold |

### Portal Path — View Workspace Usage
```
Log Analytics Workspace → Settings → Usage and estimated costs →
View data usage, daily cap status, pricing tier summary
```

---

## 18. Workspace Deletion & Recovery

| Aspect | Detail |
|---|---|
| **Soft-delete** | Workspace enters soft-delete state for **14 days** |
| **Recovery** | Can recover workspace and all its data within 14 days |
| **Permanent delete** | After 14 days, workspace + data permanently deleted |
| **Name lock** | Workspace name cannot be reused during soft-delete (14 days) |
| **Force delete** | Can force permanent deletion (bypasses soft-delete — data lost) |

### Portal Path — Delete Workspace
```
Log Analytics Workspace → Overview → Delete → Confirm name → Delete
```

### Recover Workspace (CLI)
```bash
az monitor log-analytics workspace recover \
  --resource-group <RG> --workspace-name <Name>
```

> ⚠️ **EXAM TIP:** Deleted workspaces are **soft-deleted for 14 days** — recoverable with all data. Workspace name **cannot be reused** during soft-delete period.

---

## 19. Comparison Tables

### Analytics vs Basic vs Archive Logs

| Feature | Analytics Logs | Basic Logs | Archive Logs |
|---|---|---|---|
| **Ingestion cost** | Standard (per GB) | ~50% cheaper | Same as interactive tier |
| **Retention (interactive)** | 30–730 days | 8 days fixed | N/A (cold storage) |
| **Total retention** | Up to 12 years | Up to 12 years | Up to 12 years |
| **Full KQL** | ✅ Yes | ❌ Limited (basic operators) | ❌ Restore or Search Job |
| **Alerts** | ✅ Yes | ❌ No | ❌ No |
| **Workbooks** | ✅ Yes | ❌ No | ❌ No |
| **Data export** | ✅ Yes | ❌ No | ❌ No |
| **Query cost** | Included in ingestion | Per GB scanned | Per GB scanned / restored |
| **Best for** | Primary operational data | Verbose/debug logs | Compliance / long-term |

### Workspace-Context vs Resource-Context Access

| Feature | Workspace-Context | Resource-Context |
|---|---|---|
| **Scope** | Entire workspace | Per-resource |
| **RBAC** | Workspace-level roles | Azure resource RBAC |
| **User sees** | ALL data in workspace | Only data for resources they have access to |
| **Query from** | Workspace blade | Resource blade or workspace (with filtering) |
| **Default** | No (legacy) | ✅ Yes (default for new workspaces) |
| **Best for** | Security teams needing all data | Distributed teams with resource ownership |

### AMA vs MMA (Legacy Agent)

| Feature | AMA | MMA (Deprecated Aug 2024) |
|---|---|---|
| **Configuration** | Data Collection Rules (DCR) | Workspace config |
| **Multi-homing** | Multiple DCRs | Manual |
| **Identity** | Managed Identity | Workspace Key + ID |
| **Filtering** | XPath, DCR transforms | Limited |
| **Custom Logs** | DCR-based | Custom Log Wizard |
| **OS support** | Windows + Linux | Windows + Linux |
| **Arc support** | ✅ Yes | ✅ Yes |
| **Extension** | AzureMonitorWindowsAgent | MicrosoftMonitoringAgent |

---

## 20. Quick-Fire Exam Points ⚡

1. **Log Analytics Workspace** = central data store for Azure Monitor Logs; queried via **KQL**
2. Workspace name must be **globally unique** across all Azure subscriptions
3. **Default retention** = **30 days**; configurable **30–730 days** (interactive)
4. **First 31 days** retention = **FREE** (included in ingestion cost)
5. **Total retention** (archive) = up to **12 years (4,383 days)**
6. **Soft-delete** = **14 days** after workspace deletion; recoverable with all data
7. Workspace name **locked for 14 days** after deletion (cannot reuse)
8. **Pay-As-You-Go** = default pricing tier; **Commitment Tiers** start at 100 GB/day (31-day minimum)
9. **Daily cap** stops ingestion when hit — causes **data loss**; avoid in production
10. **Analytics logs** = full KQL + alerts; **Basic logs** = 50% cheaper, 8-day query, no alerts
11. **Resource-context** = default access mode; users see only their resources' logs
12. **Workspace-context** = user sees ALL workspace data with workspace-level role
13. **Table-level RBAC** = restrict access to specific tables (e.g., SecurityEvent)
14. **AMA + DCR** = current data collection method; **MMA deprecated** (Aug 2024)
15. Associating a VM to a DCR **auto-installs AMA**
16. **DCR many-to-many**: one DCR → many VMs, one VM → many DCRs
17. **Cross-workspace query**: `workspace("name").TableName`
18. **KQL is case-sensitive** for table and column names; `|` separates operators
19. `has` is **faster** than `contains` (word boundary vs substring matching)
20. **Log search alerts** = KQL-based; can be **stateful or stateless**
21. **Heartbeat** table = check agent connectivity (1-min intervals)
22. **Usage** table = check data ingestion volume per table
23. **Data Export** to Storage or Event Hub — max **10 rules** per workspace
24. **AMPLS (Private Link Scope)** = secure workspace access via private endpoint
25. **Log Analytics Reader** = query/read only; **Log Analytics Contributor** = full config
26. **Monitoring Reader** overlaps with Log Analytics Reader but broader (all Azure Monitor data)
27. `ago(1h)` = 1 hour ago, `ago(7d)` = 7 days ago — used in `where` time filters
28. **Diagnostic Settings** = per resource, max **5 per resource**, route logs to workspace
29. **Per-table retention** can override workspace-level default retention
30. **Restore** brings archived data to hot tier; **Search Job** scans archive without full restore
31. **Cannot move** workspace to a different **region** after creation
32. Workspace can ingest from **multiple subscriptions and tenants**
33. **Query Packs** = shareable saved queries across workspaces (Azure resource)
34. **Commitment tier** = can upgrade anytime, downgrade only **after 31 days**
35. SKU for Pay-As-You-Go = `PerGB2018` in CLI/ARM templates

---

## 21. Step-by-Step Configuration Mind Maps 🗺️

---

### 21.1 Create Log Analytics Workspace

> **Portal:** `Home → + Create a resource → Search "Log Analytics workspace" → Create`

```
Create Log Analytics Workspace
│
├── Step 1: Basics
│   ├── Select Subscription
│   ├── Select or Create Resource Group
│   ├── Enter Workspace Name
│   │   ⚠️ Must be globally unique across all Azure subscriptions
│   └── Select Region
│       ⚠️ Same region as monitored resources (reduces latency + egress)
│       ⚠️ Cannot move workspace to different region after creation
│
├── Step 2: Pricing Tier
│   ├── Pay-As-You-Go (PerGB2018) — default
│   └── Commitment Tiers (100, 200, 300, 400, 500, 1000+ GB/day)
│       ⚠️ 31-day minimum commitment; upgrade anytime, downgrade after 31 days
│
├── Step 3: Tags (optional)
│
├── Step 4: Review + Create → Create
│
├── Required RBAC: Contributor on Resource Group
│
└── Post-Creation Tasks
    ├── Set Retention
    │   Portal: Workspace → Usage and estimated costs → Data Retention
    │   ⚠️ Default 30 days; first 31 days FREE; max 730 days interactive
    │
    ├── Set Daily Cap (optional)
    │   Portal: Workspace → Usage and estimated costs → Daily Cap
    │   ⚠️ Stops ingestion when hit — data loss risk!
    │
    ├── Set Access Control Mode
    │   Portal: Workspace → Properties → Access control mode
    │   ⚠️ Default = "Use resource or workspace permissions" (resource-context)
    │
    └── Configure Network Isolation (optional)
        Portal: Workspace → Network Isolation
        ⚠️ Use AMPLS + Private Endpoint for secure access
```

---

### 21.2 Configure Data Collection Rule (DCR) + AMA

> **Portal:** `Azure Monitor → Settings → Data Collection Rules → + Create`

```
Create Data Collection Rule
│
├── Step 1: Basics
│   ├── Rule Name
│   ├── Subscription
│   ├── Resource Group
│   ├── Region
│   └── Platform Type: Windows / Linux / Custom
│
├── Step 2: Resources
│   ├── + Add resources → Select VMs / VMSS / Arc servers
│   │   ⚠️ Adding a VM auto-installs AMA (if not present)
│   └── Enable Data Collection Endpoints (optional — for private link)
│
├── Step 3: Collect and deliver
│   ├── + Add data source
│   │   ├── Performance Counters (CPU, Memory, Disk, Network — customize intervals)
│   │   ├── Windows Event Logs (System, Application, Security — XPath filter)
│   │   ├── Linux Syslog (facilities: auth, syslog, daemon, etc.)
│   │   ├── Custom Text Logs (file path pattern)
│   │   └── IIS Logs
│   │
│   └── Destination
│       ├── Select Log Analytics Workspace
│       └── (Optional) Azure Monitor Metrics
│
├── Step 4: Review + Create → Create
│
├── Required RBAC: Monitoring Contributor + VM Contributor (for AMA install)
│
└── ⚠️ Key Points
    ├── One DCR → many VMs; One VM → many DCRs (many-to-many)
    ├── DCR transformations can filter data before ingestion (reduce cost)
    └── Prerequisite: Log Analytics Workspace must exist
```

---

### 21.3 Configure Diagnostic Settings (Send Resource Logs to Workspace)

> **Portal:** `Resource → Monitoring → Diagnostic settings → + Add diagnostic setting`

```
Configure Diagnostic Settings
│
├── Step 1: Navigate to Resource
│   └── Select resource (VM, Storage, Key Vault, NSG, App Service, etc.)
│       → Monitoring → Diagnostic settings
│
├── Step 2: + Add diagnostic setting
│
├── Step 3: Setting Name
│
├── Step 4: Select Categories
│   ├── Log categories (resource-specific, e.g., AuditEvent, SignInLogs)
│   ├── allLogs (select all log categories)
│   └── Metrics → AllMetrics
│
├── Step 5: Select Destination(s) — can choose multiple
│   ├── Send to Log Analytics workspace → Select workspace
│   ├── Archive to a storage account → Select account
│   ├── Stream to an event hub → Select namespace + hub
│   └── Send to partner solution → Select partner
│
├── Step 6: Save
│
├── Required RBAC: Monitoring Contributor or resource-level Contributor
│
└── ⚠️ Key Points
    ├── Max 5 diagnostic settings per resource
    ├── Must configure per resource individually (no subscription-wide toggle)
    ├── Activity Log has its own export path (subscription-level)
    └── Data appears in AzureDiagnostics or resource-specific tables
```

---

### 21.4 Export Activity Log to Workspace

> **Portal:** `Azure Monitor → Activity Log → Export Activity Logs`

```
Export Activity Log
│
├── Step 1: Navigate
│   └── Azure Monitor → Activity Log → Export Activity Logs
│
├── Step 2: Select Subscription
│
├── Step 3: + Add diagnostic setting
│
├── Step 4: Enter Name
│
├── Step 5: Select Log Categories
│   ├── Administrative
│   ├── Security
│   ├── Service Health
│   ├── Alert
│   ├── Recommendation
│   ├── Policy
│   ├── Autoscale
│   └── Resource Health
│
├── Step 6: Select Destination
│   ├── Send to Log Analytics workspace → Select workspace
│   ├── Archive to storage account
│   └── Stream to event hub
│
├── Step 7: Save
│
├── Required RBAC: Monitoring Contributor on subscription
│
└── ⚠️ Key Points
    ├── Activity Log default retention in portal = 90 days
    ├── Once in workspace → query from AzureActivity table
    └── This is a subscription-level diagnostic setting (not resource-level)
```

---

### 21.5 Configure Per-Table Retention

> **Portal:** `Log Analytics Workspace → Settings → Tables`

```
Configure Per-Table Retention
│
├── Step 1: Navigate
│   └── Log Analytics Workspace → Settings → Tables
│
├── Step 2: Select Table
│   └── Click on table name (e.g., SecurityEvent, Perf, Syslog)
│
├── Step 3: Configure Retention
│   ├── Interactive retention: 30–730 days (or workspace default)
│   └── Total retention: interactive + archive (up to 4,383 days / 12 years)
│
├── Step 4: Save
│
├── Required RBAC: Log Analytics Contributor
│
└── ⚠️ Key Points
    ├── Per-table overrides workspace-level default
    ├── Total retention must be ≥ interactive retention
    └── Archive data requires Restore or Search Job to access
```

---

### 21.6 Change Table Plan (Analytics ↔ Basic)

> **Portal:** `Log Analytics Workspace → Settings → Tables`

```
Change Table Plan
│
├── Step 1: Navigate
│   └── Log Analytics Workspace → Settings → Tables
│
├── Step 2: Select Table → Change Plan
│   ├── Analytics (full KQL, alerts, higher ingestion cost)
│   └── Basic (limited KQL, no alerts, ~50% cheaper ingestion)
│
├── Step 3: Confirm change
│
├── Required RBAC: Log Analytics Contributor
│
└── ⚠️ Key Points
    ├── Basic → Analytics change takes effect immediately
    ├── Analytics → Basic has 30-day commitment once changed
    ├── Basic logs: 8 days interactive, then archive
    ├── Not all tables support Basic plan
    └── Basic logs have per-query scan charge
```

---

### 21.7 Set Up Cross-Workspace Query

> **Portal:** `Log Analytics Workspace → Logs`

```
Cross-Workspace Query
│
├── Step 1: Open Logs in any workspace
│   └── Log Analytics Workspace → Logs
│
├── Step 2: Write KQL with workspace() function
│   ├── By name: workspace("WorkspaceName").TableName
│   ├── By ID:   workspace("workspace-guid").TableName
│   └── By resource ID: workspace("/subscriptions/.../workspaces/Name").TableName
│
├── Example:
│   union workspace("Workspace1").Heartbeat,
│         workspace("Workspace2").Heartbeat
│   | distinct Computer | count
│
├── Required RBAC: Log Analytics Reader on ALL queried workspaces
│
└── ⚠️ Key Points
    ├── Supports up to 100 workspaces in a single query
    ├── Cross-resource: app("AppInsightsName").requests
    └── Performance may be slower than single-workspace queries
```

---

### 21.8 Configure Workspace Network Isolation (Private Link)

> **Portal:** `Azure Monitor → Settings → Private Link Scopes`

```
Configure Private Link for Workspace
│
├── Step 1: Create Azure Monitor Private Link Scope (AMPLS)
│   └── Azure Monitor → Settings → Private Link Scopes → + Create
│       ├── Name, Subscription, Resource Group
│       └── Create
│
├── Step 2: Add Resources to AMPLS
│   └── AMPLS → Azure Monitor Resources → + Add
│       ├── Select Log Analytics Workspace(s)
│       └── Select Application Insights resource(s) (if needed)
│
├── Step 3: Create Private Endpoint
│   └── AMPLS → Private Endpoint Connections → + New
│       ├── Name, Subscription, RG, Region
│       ├── Select Virtual Network + Subnet
│       ├── DNS integration → Yes (recommended)
│       └── Create
│
├── Step 4: (Optional) Block Public Access
│   └── Log Analytics Workspace → Network Isolation
│       ├── Accept data ingestion from public networks: No
│       └── Accept queries from public networks: No
│
├── Required RBAC: Contributor on AMPLS + Network Contributor for endpoint
│
└── ⚠️ Key Points
    ├── AMPLS limits: max 10 Private Endpoints, 50 resources per AMPLS
    ├── One workspace can be in only ONE AMPLS (for ingestion)
    └── DNS resolution must point to private IP (Private DNS Zone)
```

---

### 21.9 Create Log Search Alert from Workspace

> **Portal:** `Azure Monitor → Alerts → + Create → Alert Rule`

```
Create Log Search Alert
│
├── Step 1: Select Scope
│   └── Select Log Analytics Workspace (or specific resource)
│
├── Step 2: Condition
│   ├── Signal type: Custom log search
│   ├── Enter KQL query
│   │   e.g., Heartbeat | summarize LastHB = max(TimeGenerated) by Computer
│   │        | where LastHB < ago(15m)
│   ├── Measurement: Table rows / Metric measurement
│   ├── Aggregation type + granularity
│   ├── Operator + Threshold value
│   ├── Frequency of evaluation (5 min — 24 hrs)
│   └── Lookback period (5 min — 48 hrs)
│
├── Step 3: Actions
│   └── Select Action Group (Email / SMS / Webhook / Runbook / Logic App)
│
├── Step 4: Details
│   ├── Alert Rule Name
│   ├── Severity (Sev 0–4)
│   ├── Region
│   ├── Enable upon creation: Yes/No
│   └── Auto-resolve: Stateful / Stateless
│
├── Step 5: Review + Create → Create
│
├── Required RBAC: Monitoring Contributor
│
└── ⚠️ Key Points
    ├── Log alerts incur cost based on evaluation frequency
    ├── Cannot create alerts on Basic Logs tables
    ├── Stateful = fires once, auto-resolves; Stateless = fires every evaluation
    └── Lookback must be ≥ frequency
```

---

### 21.10 Recover Deleted Workspace

> **CLI:** `az monitor log-analytics workspace recover`

```
Recover Deleted Workspace
│
├── Prerequisites
│   ├── Workspace deleted within last 14 days (soft-delete)
│   ├── Know workspace name + resource group
│   └── Resource group must still exist (or recreate it)
│
├── Option 1: Azure CLI
│   └── az monitor log-analytics workspace recover \
│       --resource-group <RG> --workspace-name <Name>
│
├── Option 2: PowerShell
│   └── Restore-AzOperationalInsightsWorkspace -ResourceGroupName <RG> `
│       -Name <Name> -Location <OriginalRegion>
│
├── Option 3: Portal
│   └── Create new workspace with SAME name + SAME resource group + SAME region
│       → Azure auto-recovers the soft-deleted workspace
│
├── Required RBAC: Contributor on Resource Group
│
└── ⚠️ Key Points
    ├── All data + configuration is restored
    ├── Must use SAME subscription, resource group, name, and region
    ├── After 14 days → permanent deletion, no recovery
    └── Workspace name locked during 14-day soft-delete period
```


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
