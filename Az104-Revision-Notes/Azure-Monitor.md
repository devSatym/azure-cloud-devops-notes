<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Monitor — AZ-104 Revision Notes

---

## 1. What is Azure Monitor?

- **Full-stack monitoring service** for collecting, analyzing, and acting on telemetry from Azure & on-prem resources
- Collects **Metrics** (numeric, time-series) and **Logs** (structured, query-based)
- Central hub: Azure Monitor → feeds into Alerts, Dashboards, Workbooks, Insights, Autoscale, etc.
- Enabled **by default** for platform metrics — no additional config needed

---

## 2. Key Components

| Component | Purpose |
|---|---|
| **Metrics** | Numeric time-series data (CPU %, memory, disk, network) — stored for **93 days** |
| **Logs (Log Analytics)** | Structured log data — stored in **Log Analytics Workspace**, queried via **KQL** |
| **Alerts** | Automated notifications/actions when conditions are met |
| **Action Groups** | Define WHO gets notified and WHAT action to take |
| **Workbooks** | Interactive visual reports combining metrics + logs |
| **Insights** | Pre-built monitoring experiences (VM Insights, Network Insights, etc.) |
| **Diagnostic Settings** | Route platform logs/metrics to Log Analytics, Storage, Event Hubs |
| **Application Insights** | APM for web apps (performance, failures, dependencies) |
| **Autoscale** | Automatically scale resources based on metrics or schedules |
| **Service Health** | Azure platform health, planned maintenance, advisories |
| **Azure Advisor** | Best practice recommendations (cost, security, reliability, performance) |

---

## 3. Data Sources

| Source | Data Type | Example |
|---|---|---|
| **Platform Metrics** | Metrics (auto-collected) | VM CPU %, Storage transactions |
| **Activity Log** | Logs (auto-collected) | Who created/deleted a resource, RBAC changes |
| **Resource Logs** | Logs (need Diagnostic Settings) | NSG flow logs, Key Vault access logs |
| **Guest OS Metrics/Logs** | Metrics + Logs (need agent) | Memory %, event logs, syslog |
| **Application Data** | Logs (Application Insights SDK) | Request rates, exceptions, dependencies |
| **Custom Metrics** | Metrics (API / Application Insights) | Business-specific metrics |

> ⚠️ **EXAM TIP:** **Platform metrics** = auto-collected, no config. **Resource logs** = require **Diagnostic Settings** to be enabled. **Guest OS data** = requires **Azure Monitor Agent (AMA)**.

---

## 4. Metrics

- **Numeric, time-series data** — lightweight, near real-time
- Stored for **93 days** (platform metrics)
- Accessible from **Metrics Explorer**
- Supports: Aggregation (Avg, Min, Max, Sum, Count), Filtering, Splitting
- **Custom metrics** via Application Insights or REST API (retained 93 days)
- **Namespace** = logical grouping (e.g., `Microsoft.Compute/virtualMachines`)

### Portal Path — Metrics Explorer
```
Resource → Monitoring → Metrics
OR
Azure Monitor → Metrics
```

### Key Metric Operations
| Operation | Description |
|---|---|
| **Filter** | Show metric for specific dimension (e.g., Disk = OS Disk only) |
| **Split** | Break metric by dimension into separate lines |
| **Aggregation** | Avg / Sum / Min / Max / Count |
| **Time range** | Last 1hr to 30 days (custom) |
| **Pin to Dashboard** | Pin chart to Azure Dashboard |

> ⚠️ **EXAM TIP:** Metrics retention = **93 days**. To retain longer → route to **Log Analytics** via Diagnostic Settings (then retained per workspace retention policy, default 30 days, max 730 days).

---

## 5. Logs & Log Analytics

### Log Analytics Workspace
- **Central repository** for log data from multiple sources
- Data queried using **KQL (Kusto Query Language)**
- Default retention: **30 days** (configurable: 30–730 days)
- **Interactive retention** (hot) → up to 730 days
- **Total retention** (archive/cold) → up to **12 years** (4383 days)
- Workspace can collect from multiple subscriptions and tenants
- Pricing: **Pay-As-You-Go** or **Commitment Tiers** (100, 200, 300, 400, 500 GB/day)

### Portal Path — Create Log Analytics Workspace
```
Home → + Create a resource → Search "Log Analytics workspace" → Create
```

### Portal Path — Query Logs
```
Log Analytics Workspace → Logs → Write KQL query → Run
OR
Resource → Monitoring → Logs
OR
Azure Monitor → Logs
```

### Key KQL Commands (Exam-Relevant)

| Command | Purpose | Example |
|---|---|---|
| `where` | Filter rows | `Heartbeat \| where TimeGenerated > ago(1h)` |
| `summarize` | Aggregate | `Perf \| summarize avg(CounterValue) by Computer` |
| `project` | Select columns | `Event \| project TimeGenerated, EventLog, Computer` |
| `count` | Count rows | `SecurityEvent \| count` |
| `extend` | Add calculated column | `Perf \| extend GB = CounterValue / 1024` |
| `order by` / `sort by` | Sort results | `Event \| order by TimeGenerated desc` |
| `top` | First N results | `Perf \| top 10 by CounterValue desc` |
| `render` | Visualization | `Perf \| summarize avg(CounterValue) by bin(TimeGenerated, 1h) \| render timechart` |
| `join` | Join tables | `Heartbeat \| join (Perf) on Computer` |
| `union` | Combine tables | `union Event, Syslog` |
| `ago()` | Relative time | `ago(1h)`, `ago(7d)`, `ago(30m)` |
| `bin()` | Time bucketing | `bin(TimeGenerated, 5m)` |
| `distinct` | Unique values | `Heartbeat \| distinct Computer` |

> ⚠️ **EXAM TIP:** KQL is **case-sensitive** for table and column names. `|` (pipe) separates operators. Read KQL queries left to right, top to bottom.

### Common Log Tables

| Table | Data |
|---|---|
| **Heartbeat** | Agent connectivity (1 min intervals) |
| **Perf** | Performance counters (CPU, Memory, Disk, Network) |
| **Event** | Windows Event Log |
| **Syslog** | Linux Syslog |
| **SecurityEvent** | Windows Security events |
| **AzureActivity** | Activity Log entries |
| **AzureDiagnostics** | Resource diagnostic logs |
| **InsightsMetrics** | Metrics from VM Insights / Container Insights |
| **Update** | Update management data |
| **Alert** | Fired alert records |

---

## 6. Activity Log

- Records **control-plane operations** on Azure resources (ARM-level)
- Auto-collected, **no Diagnostic Settings needed** to view
- Retained for **90 days** (in portal)
- To retain longer → route to **Log Analytics** or **Storage Account** via Diagnostic Settings
- Covers: Write operations (PUT, POST, DELETE), RBAC assignments, policy events, service health
- Does NOT capture read (GET) operations

### Activity Log Categories

| Category | Examples |
|---|---|
| **Administrative** | Create VM, Delete RG, Assign role |
| **Service Health** | Azure outages, planned maintenance |
| **Resource Health** | Resource availability changes |
| **Alert** | Alert fired events |
| **Autoscale** | Scale up/down events |
| **Recommendation** | Azure Advisor recommendations |
| **Security** | Microsoft Defender alerts |
| **Policy** | Azure Policy evaluation events |

### Portal Path
```
Azure Monitor → Activity Log
OR
Resource → Activity Log
OR
Resource Group → Activity Log
```

### Export Activity Log
```
Azure Monitor → Activity Log → Export Activity Logs →
+ Add Diagnostic Setting → Select categories →
Send to: Log Analytics / Storage Account / Event Hub → Save
```

> ⚠️ **EXAM TIP:** Activity Log default retention = **90 days**. To keep longer, export via **Diagnostic Settings**. Once in Log Analytics, query from **AzureActivity** table.

---

## 7. Diagnostic Settings

- Route **platform logs and metrics** from Azure resources to destinations
- Must be configured **per resource** (not auto-enabled)
- Each resource can have **up to 5 diagnostic settings**

### Destinations

| Destination | Purpose |
|---|---|
| **Log Analytics Workspace** | Query with KQL, use in alerts, workbooks |
| **Storage Account** | Long-term archival (cheapest) |
| **Event Hubs** | Stream to 3rd party SIEM / external tools |
| **Partner Solution** | Send to partner monitoring tools (Datadog, Elastic, etc.) |

### Portal Path
```
Resource → Monitoring → Diagnostic settings → + Add diagnostic setting →
Name → Select log categories + metrics → Select destination(s) → Save
```

> ⚠️ **EXAM TIP:** Diagnostic settings are **resource-level** — you must enable them on EACH resource individually. There is no subscription-wide toggle (except Activity Log export).

---

## 8. Azure Monitor Agent (AMA)

- **Replacement** for legacy agents (Log Analytics Agent/MMA, Diagnostics Extension)
- Collects **guest OS** data (performance counters, event logs, syslog, custom logs)
- Uses **Data Collection Rules (DCR)** to define what to collect and where to send
- Supports **Windows & Linux** (Azure VMs, Arc-enabled servers, VMSS)
- Installed as a **VM extension**
- Supports **multiple destinations** (multiple workspaces via DCR)

### AMA vs Legacy Agents

| Feature | Azure Monitor Agent (AMA) | Log Analytics Agent (MMA) |
|---|---|---|
| Status | **Current / Recommended** | **Deprecated (Aug 2024)** |
| Config method | Data Collection Rules (DCR) | Workspace-level config |
| Multi-homing | Yes (via multiple DCRs) | Yes (manual) |
| Windows Event filtering | Granular (XPath queries) | Limited |
| Linux Syslog | Yes | Yes |
| Extension-based | Yes | Yes |
| Managed Identity | Yes (System or User) | No |

### Data Collection Rules (DCR)
- Define **what data** to collect (which counters, logs, events)
- Define **where to send** (which Log Analytics workspace)
- Can associate **multiple VMs** to one DCR
- Can associate **multiple DCRs** to one VM
- DCR is an **Azure resource** (centrally managed)

### Portal Path — Install AMA & Create DCR
```
Azure Monitor → Settings → Data Collection Rules → + Create →
Rule Name, Subscription, RG, Region, Platform Type (Windows/Linux) →
Add Resources (select VMs) →
Add Data Source (Performance counters / Windows Event Logs / Syslog / Custom logs) →
Add Destination (Log Analytics Workspace) →
Review + Create → Create
```

> ⚠️ **EXAM TIP:** Legacy **Log Analytics Agent (MMA)** is deprecated since Aug 2024. AZ-104 focuses on **Azure Monitor Agent (AMA)** with **Data Collection Rules (DCR)**.

---

## 9. Alerts

### Alert Components

| Component | Purpose |
|---|---|
| **Alert Rule** | Condition that triggers the alert |
| **Action Group** | Who/what to notify when alert fires |
| **Alert Processing Rules** | Suppress or modify alerts (e.g., during maintenance) |
| **Alert State** | New → Acknowledged → Closed |

### Alert Types

| Type | Signal Source | Use Case |
|---|---|---|
| **Metric Alert** | Platform/custom metrics | CPU > 80%, Disk > 90% |
| **Log Search Alert** | Log Analytics (KQL query) | Error count > 10 in last 1hr |
| **Activity Log Alert** | Activity Log | VM deleted, role assigned |
| **Service Health Alert** | Service Health | Azure outage in your region |
| **Resource Health Alert** | Resource Health | Your VM became unavailable |
| **Smart Detection (AI)** | Application Insights | Anomaly detection |

### Metric Alerts

- Evaluate metrics at **regular intervals**
- Types: **Static** (fixed threshold) vs **Dynamic** (ML-based, auto-adjusts)
- Frequency: 1 min, 5 min, 15 min, 30 min, 1 hr
- Can alert on **multiple metrics** (multi-resource metric alerts)
- **Multi-resource alerts:** One rule → monitors multiple resources of same type in same region
- Supports **dimensions** (e.g., alert per VM in a resource group)
- **Stateful** — fires once, resolves when condition clears (no repeated alerts)

### Log Search Alerts

- Run a **KQL query** at defined frequency
- Evaluation: **Number of results** or **Metric measurement**
- Frequency: 5 min to 24 hrs
- Lookback period: 5 min to 48 hrs
- **Stateful** or **Stateless** (configurable)

### Activity Log Alerts

- Trigger on specific **control-plane events**
- Examples: VM created, role assigned, resource deleted
- Scope: Subscription or Resource Group level
- **No cost** for Activity Log alerts

### Service Health Alerts

- Trigger on: **Service issues**, **Planned maintenance**, **Health advisories**, **Security advisories**
- Scope: Subscription + Region + Service specific
- **Free** — no cost
- **Highly recommended** to set up

### Portal Path — Create Alert Rule
```
Azure Monitor → Alerts → + Create → Alert Rule →
Select Scope (resource/RG/subscription) →
Select Condition (signal type + threshold) →
Select Action Group →
Alert Rule Name, Severity, Enable on creation →
Review + Create
```

### Alert Severity Levels

| Severity | Level | Meaning |
|---|---|---|
| **Sev 0** | Critical | Immediate attention |
| **Sev 1** | Error | Needs attention soon |
| **Sev 2** | Warning | Potential issue |
| **Sev 3** | Informational | FYI / awareness |
| **Sev 4** | Verbose | Detailed/debug |

> ⚠️ **EXAM TIP:** **Metric alerts** are **stateful** (fire once, auto-resolve). **Log alerts** can be stateful or stateless. **Activity log alerts** are **free**.

---

## 10. Action Groups

- Define **notifications** and **actions** when an alert fires
- Reusable across multiple alert rules
- Max **10 action groups per alert rule**

### Notification Types

| Type | Detail |
|---|---|
| **Email** | Up to 1000 emails/hr, max 100 email recipients per action group |
| **SMS** | Max 1 SMS every 5 min per phone number |
| **Push Notification** | Azure mobile app |
| **Voice** | Max 1 voice call every 5 min per phone number |

### Action Types

| Action | Purpose |
|---|---|
| **Azure Function** | Trigger a function |
| **Logic App** | Trigger a Logic App workflow |
| **Webhook** | Call an external HTTP endpoint |
| **ITSM** | Create incident in ITSM tool (ServiceNow, etc.) |
| **Automation Runbook** | Run Azure Automation runbook |
| **Event Hub** | Stream alert data |
| **Secure Webhook** | Webhook with Azure AD auth |

### Portal Path — Create Action Group
```
Azure Monitor → Alerts → Action groups → + Create →
Subscription, RG, Action Group Name, Display Name →
Notifications (Email/SMS/Push/Voice) →
Actions (Function/Logic App/Webhook/Runbook/ITSM) →
Review + Create
```

> ⚠️ **EXAM TIP:** Rate limits — Email: **100 emails/hr**. SMS: **1 per 5 min**. Voice: **1 per 5 min**. Webhook: **10 calls per minute** (timeout 10 sec, retry up to 2 times).

---

## 11. Alert Processing Rules

- Apply actions or suppress alerts **without modifying alert rules**
- Use cases: **Suppress alerts during maintenance windows**, **Add action groups at scale**
- Can filter by: Resource type, Resource group, Severity, Alert name, etc.
- Supports **one-time** or **recurring** schedules
- Replaces classic **Action Rules** (legacy)

### Portal Path
```
Azure Monitor → Alerts → Alert processing rules → + Create →
Select Scope → Add Filters → Select Rule Type
(Apply action group OR Suppression) → Scheduling → Create
```

---

## 12. VM Insights

- Pre-built monitoring for **Azure VMs and VMSS**
- Shows: Performance (CPU, Memory, Disk, Network), Dependencies (map), Health
- Requires **Azure Monitor Agent (AMA)** + **Log Analytics Workspace**
- **Dependency Map** requires **Dependency Agent** (+ AMA)
- Map shows processes, connections, TCP ports between machines

### Portal Path — Enable VM Insights
```
Virtual Machine → Monitoring → Insights → Enable →
Select Log Analytics Workspace →
Enable (installs AMA + Dependency Agent)
OR
Azure Monitor → Insights → Virtual Machines → Not Monitored →
Select VM → Enable
```

### What VM Insights Collects

| Data | Details |
|---|---|
| **Performance** | CPU %, Available Memory, Disk IOPS, Disk latency, Network bytes |
| **Map** | Running processes, inbound/outbound connections, ports, dependencies |
| **Health** | (Preview) Health state of VM |

> ⚠️ **EXAM TIP:** VM Insights **Performance** tab = needs AMA only. **Map** tab = needs AMA **+ Dependency Agent**. Both need Log Analytics Workspace.

---

## 13. Network Watcher

- **Regional service** — automatically enabled in each region where you have a VNet
- Network diagnostics and monitoring tools

### Key Tools

| Tool | Purpose |
|---|---|
| **IP Flow Verify** | Check if packet allowed/denied between VM and endpoint (tests NSG rules) |
| **Next Hop** | Shows next hop for a route from a VM |
| **Connection Troubleshoot** | Test connectivity between source and destination (TCP/ICMP) |
| **NSG Flow Logs** | Log traffic flowing through NSG (stored in Storage Account) |
| **Packet Capture** | Capture network packets on a VM (requires Network Watcher extension) |
| **Connection Monitor** | Continuous connectivity monitoring between endpoints |
| **Traffic Analytics** | Visualize NSG flow logs (requires Log Analytics) |
| **VPN Troubleshoot** | Diagnose VPN Gateway/connection issues |
| **NSG Diagnostic** | Check which NSG rules apply to traffic |
| **Topology** | Visual map of VNet resources |

### Portal Path
```
Search "Network Watcher" →
Select tool (IP Flow Verify / Next Hop / Connection Troubleshoot, etc.) →
Select VM / resource → Run
```

### NSG Flow Logs
```
Network Watcher → NSG Flow Logs → + Create →
Select NSG → Select Storage Account → Set retention (0–365 days) →
Enable Traffic Analytics (optional) → Select Log Analytics Workspace →
Processing interval: 10 min or 1 hr → Create
```

> ⚠️ **EXAM TIP:** **IP Flow Verify** = tests NSG allow/deny for specific traffic. **Next Hop** = shows routing path. **Connection Troubleshoot** = end-to-end connectivity test. Know the difference!

---

## 14. Application Insights

- **APM (Application Performance Management)** for web apps
- Monitors: Requests, failures, exceptions, dependencies, page views, performance
- Supports: .NET, Java, Node.js, Python, JavaScript
- Two modes: **Workspace-based** (logs → Log Analytics) vs **Classic** (deprecated)
- **Availability Tests** — ping from Azure locations to check uptime
- **Live Metrics Stream** — real-time telemetry (no delay)
- **Application Map** — visual dependency map of app components
- **Smart Detection** — AI-driven anomaly detection

### Availability Tests

| Type | Description |
|---|---|
| **Standard test** | Single URL ping from multiple Azure locations (HTTP GET/HEAD) |
| **Custom Track Availability** | Custom code for complex test scenarios |

### Portal Path
```
Application Insights → Overview (for app health)
Application Insights → Performance / Failures / Availability / Live Metrics
Application Insights → Application Map
```

> ⚠️ **EXAM TIP:** Application Insights **Availability Tests** check if your web app is reachable from **multiple global Azure locations**. Supports **SSL cert validation** and **custom headers**.

---

## 15. Autoscale

- **Automatically scale** resources based on metrics or schedule
- Supports: **VMSS, App Service, Cloud Services**, and more
- Types: **Metric-based** (reactive) and **Schedule-based** (predictive)
- Scale **Out** (add instances) / Scale **In** (remove instances)
- Configure: Min, Max, and Default instance count

### Autoscale Rules

| Setting | Description |
|---|---|
| **Metric Source** | Any Azure Monitor metric (e.g., CPU %, HTTP Queue, custom) |
| **Operator** | Greater than, Less than, Equal, etc. |
| **Threshold** | e.g., CPU > 70% |
| **Duration** | Time window to evaluate metric (e.g., 10 min) |
| **Time grain** | Metric aggregation interval (1 min, 5 min, etc.) |
| **Time aggregation** | Average, Min, Max, Sum, Count, Last |
| **Action** | Increase count by / Increase count to / Increase percent by |
| **Cool down** | Wait period after a scale action (default: **5 minutes**) |

### Flapping Prevention
- **Cool down period** — prevents rapid scale in/out oscillation
- Default: **5 minutes**
- Scale-in evaluates: if scaling in would cause scale-out again, it **skips** the scale-in

### Scale Conditions
- **Default condition** — always active (fallback)
- **Custom conditions** — metric-based or schedule-based (override default)
- Multiple conditions supported, evaluated in order

### Portal Path — Configure Autoscale
```
Resource (e.g., VMSS / App Service Plan) → Settings → Scale out (Autoscale) →
Custom autoscale →
Set Min / Max / Default instance count →
+ Add a rule (Metric-based) → Set metric, threshold, action, cool down →
+ Add a rule (Scale-in rule) →
OR Schedule-based → Set specific date/time or recurring →
Save
```

> ⚠️ **EXAM TIP:** Autoscale **default cool down = 5 minutes**. Always create **both** scale-out AND scale-in rules. Without scale-in, resources never scale back down. **Flapping** = rapid oscillation, prevented by cool down.

---

## 16. Azure Dashboards

- **Custom visual dashboards** — pin metrics, logs, workbooks tiles
- Shared or private
- **RBAC** controlled — share via dashboard sharing (Reader role to view)
- Supports: Metric charts, Log query results, Markdown text, Resource lists, Workbooks
- Max **100 dashboards** per subscription (shared)

### Portal Path
```
Home → Dashboard → + New Dashboard → Blank Dashboard →
Drag tiles (Metrics chart, Markdown, Log query, etc.) →
Pin charts from Metrics Explorer / Log Analytics → Save → Share
```

### Share Dashboard
```
Dashboard → Share → Publish to "dashboards" resource group →
Assign RBAC (Reader) to users/groups → OK
```

> ⚠️ **EXAM TIP:** Shared dashboards are **Azure resources** stored in a resource group. They need **RBAC** permissions (at minimum **Reader** role on the dashboard + **Reader** on underlying resources to see data).

---

## 17. Workbooks

- **Interactive reports** combining metrics, logs, text, parameters
- More powerful than dashboards — supports parameters, links, conditional formatting
- Template gallery available (pre-built workbooks)
- Saved to a **resource group** (shared) or **My Workbooks** (private)
- Can be pinned to **dashboards**

### Portal Path
```
Azure Monitor → Workbooks → + New →
Add elements (Metric, Query, Text, Parameters, Links, Groups) →
Save → Choose shared or private
```

---

## 18. Service Health

- Tracks Azure platform health **specific to YOUR resources**
- Three components:

| Component | Scope |
|---|---|
| **Azure Status** | Global Azure health (status.azure.com) |
| **Service Health** | Health of Azure services YOU use in YOUR regions |
| **Resource Health** | Health of YOUR specific resources (VM, DB, etc.) |

### Resource Health States

| State | Meaning |
|---|---|
| **Available** | Resource is healthy |
| **Unavailable** | Platform or user-initiated event detected |
| **Unknown** | No health signal received for > 10 min |
| **Degraded** | Resource detecting performance loss but still available |

### Portal Path
```
Search "Service Health" → Service Issues / Planned Maintenance / 
Health Advisories / Security Advisories / Resource Health / Health History
```

### Create Service Health Alert
```
Service Health → + Create service health alert →
Select Services, Regions, Event types →
Select Action Group → Alert Rule Name → Create
```

> ⚠️ **EXAM TIP:** **Service Health alerts are FREE**. Always set them up. **Resource Health** = YOUR resource level. **Service Health** = Azure service level. **Azure Status** = global.

---

## 19. Azure Advisor

- **Personalized best practices** recommendation engine
- Categories: **Reliability, Security, Performance, Cost, Operational Excellence**
- Recommendations are **free**
- Can **dismiss, postpone, or act** on recommendations
- Integrates with Azure Monitor alerts

### Portal Path
```
Search "Advisor" → Overview → Select category →
View recommendations → Follow remediation steps
```

### Advisor Alerts
```
Advisor → Alerts → + New Advisor alert →
Select category (Cost, Security, etc.) →
Select Action Group → Create
```

> ⚠️ **EXAM TIP:** Advisor **Cost** recommendations include: right-sizing VMs, unused resources, reserved instance savings. Advisor **Security** = mirrors Microsoft Defender recommendations.

---

## 20. Security & RBAC

### Built-in Roles for Monitoring

| Role | Permissions |
|---|---|
| **Monitoring Reader** | Read all monitoring data (metrics, logs, alerts, diagnostics) |
| **Monitoring Contributor** | Read monitoring data + create/modify alert rules, action groups, diagnostic settings |
| **Log Analytics Reader** | Read Log Analytics data, search logs |
| **Log Analytics Contributor** | Read + configure Log Analytics (add solutions, configure collection, etc.) |

### Key Permissions

| Action | Required Role |
|---|---|
| View metrics / alerts | Monitoring Reader |
| Create alert rules | Monitoring Contributor |
| Create/modify diagnostic settings | Monitoring Contributor or resource Contributor |
| Query Log Analytics | Log Analytics Reader |
| Change workspace retention | Log Analytics Contributor |
| Create DCR | Monitoring Contributor |
| Manage Action Groups | Monitoring Contributor |

> ⚠️ **EXAM TIP:** **Monitoring Reader** = view only. **Monitoring Contributor** = create alerts, action groups, diagnostic settings. Know the difference.

---

## 21. Pricing Key Points

| Component | Pricing Model |
|---|---|
| **Platform Metrics** | **Free** (auto-collected) |
| **Activity Log** | **Free** (90-day portal retention) |
| **Activity Log → Log Analytics** | Charged per GB ingested |
| **Metric Alerts** | Charged per **alert rule** monitored per month |
| **Log Search Alerts** | Charged per evaluation frequency |
| **Activity Log Alerts** | **Free** |
| **Service Health Alerts** | **Free** |
| **Log Analytics Ingestion** | Per GB (Pay-As-You-Go) or Commitment Tiers |
| **Log Analytics Retention** | First **31 days included free**, then per GB/month after |
| **Archive (Total Retention)** | Lower cost than interactive retention |
| **Basic Logs** | Cheaper ingestion, limited query (for verbose/debug logs) |
| **Action Groups** | Email = free, SMS/Voice = per notification |
| **Application Insights** | Per GB ingested (first 5 GB/month free per billing account) |

> ⚠️ **EXAM TIP:** **Free items:** Platform metrics, Activity Log (90 days), Activity Log alerts, Service Health alerts, Advisor. **Paid items:** Log ingestion, metric alerts, log alerts, retention beyond 31 days, SMS/Voice notifications.

---

## 22. Quick-Fire Exam Points ⚡

1. **Platform metrics** = auto-collected, retained **93 days**, free
2. **Activity Log** = retained **90 days**, free; export via Diagnostic Settings for longer
3. **Log Analytics default retention** = **30 days** (configurable 30–730 days interactive, up to 12 years total)
4. **Log Analytics first 31 days retention = free**
5. **AMA** replaces legacy MMA agent — uses **Data Collection Rules (DCR)**
6. **DCR** = what to collect + where to send; one DCR → many VMs, one VM → many DCRs
7. **Metric alerts** = **stateful** (fire once, auto-resolve)
8. **Activity Log alerts** = **free**
9. **Service Health alerts** = **free**
10. **Action group limits:** 100 email recipients, SMS = 1 per 5 min, Voice = 1 per 5 min
11. **Alert severity:** Sev 0 (Critical) → Sev 4 (Verbose)
12. **Diagnostic settings** = per resource, max 5 per resource
13. Destinations: Log Analytics, Storage Account, Event Hub, Partner Solution
14. **VM Insights Map** requires **Dependency Agent** + AMA
15. **KQL** is case-sensitive; pipe `|` separates operators
16. **Autoscale cool down** default = **5 minutes**
17. Create **both** scale-out AND scale-in rules to avoid one-way scaling
18. **Shared dashboards** = Azure resources, need RBAC (Reader) to view
19. **Network Watcher** = regional, auto-enabled per region
20. **IP Flow Verify** = NSG test, **Next Hop** = routing test, **Connection Troubleshoot** = E2E test
21. **NSG Flow Logs** → stored in **Storage Account**, analyzed by **Traffic Analytics** (needs Log Analytics)
22. **Resource Health** = your resource, **Service Health** = Azure service, **Azure Status** = global
23. **Advisor categories:** Reliability, Security, Performance, Cost, Operational Excellence
24. **Monitoring Reader** = view only, **Monitoring Contributor** = create/edit alerts, diagnostics
25. **Application Insights** first **5 GB/month free** per billing account
26. **Basic Logs** = cheaper ingestion for verbose logs, limited query capability
27. Alert Processing Rules = suppress alerts during maintenance without modifying alert rules
28. **Dynamic metric alerts** use ML to auto-adjust thresholds based on historical patterns
29. **Max 100 shared dashboards** per subscription
30. **Workspace-based Application Insights** = recommended (logs go to Log Analytics workspace)

---

## 23. Step-by-Step Configuration Mind Maps 🗺️

---

### 23.1 Create Log Analytics Workspace

> **Portal:** `Home → + Create a resource → Search "Log Analytics workspace" → Create`

```
Create Log Analytics Workspace
│
├── Step 1: Basics
│   ├── Select Subscription
│   ├── Select or Create Resource Group
│   ├── Enter Workspace Name (globally unique)
│   └── Select Region
│       ⚠️ Choose same region as monitored resources for lower latency
│
├── Step 2: Pricing Tier
│   ├── Pay-As-You-Go (default, per GB)
│   └── Commitment Tiers (100, 200, 300, 400, 500 GB/day)
│       ⚠️ Commitment tier = 31-day minimum commitment
│
├── Step 3: Tags (Optional)
│
├── Step 4: Review + Create → Create
│
└── Post-Creation: Set Retention
    │   Portal: Workspace → Settings → Usage and estimated costs →
    │           Data Retention → Set slider (30–730 days) → OK
    └── ⚠️ First 31 days FREE, charged per GB/month after
```

---

### 23.2 Configure Diagnostic Settings

> **Portal:** `Resource → Monitoring → Diagnostic settings`

```
Configure Diagnostic Settings
│
├── Step 1: Navigate to Resource
│   └── Select resource (VM, Storage, Key Vault, NSG, etc.)
│       → Monitoring → Diagnostic settings
│
├── Step 2: Click "+ Add diagnostic setting"
│
├── Step 3: Enter Setting Name
│
├── Step 4: Select Log Categories
│   ├── Resource-specific logs (e.g., AuditEvent for Key Vault)
│   ├── allLogs (all available log categories)
│   └── Metrics → AllMetrics
│
├── Step 5: Select Destination(s) (can choose multiple)
│   ├── Send to Log Analytics workspace → Select workspace
│   ├── Archive to a storage account → Select account + retention
│   ├── Stream to an event hub → Select namespace + hub
│   └── Send to partner solution → Select partner
│
├── Step 6: Save
│
└── ⚠️ Max 5 diagnostic settings per resource
    ⚠️ Must configure per resource individually
    ⚠️ Required Role: Monitoring Contributor or resource Contributor
```

---

### 23.3 Export Activity Log

> **Portal:** `Azure Monitor → Activity Log → Export Activity Logs`

```
Export Activity Log
│
├── Step 1: Navigate
│   └── Azure Monitor → Activity Log → Export Activity Logs
│
├── Step 2: Select Subscription
│
├── Step 3: Click "+ Add diagnostic setting"
│
├── Step 4: Enter Name
│
├── Step 5: Select Categories
│   ├── Administrative
│   ├── Security
│   ├── ServiceHealth
│   ├── Alert
│   ├── Recommendation
│   ├── Policy
│   ├── Autoscale
│   └── ResourceHealth
│
├── Step 6: Select Destination
│   ├── Send to Log Analytics workspace
│   ├── Archive to Storage Account
│   └── Stream to Event Hub
│
├── Step 7: Save
│
└── ⚠️ Default retention (portal) = 90 days
    ⚠️ Once exported to Log Analytics → query via "AzureActivity" table
```

---

### 23.4 Install Azure Monitor Agent & Create DCR

> **Portal:** `Azure Monitor → Settings → Data Collection Rules → + Create`

```
Create Data Collection Rule (DCR) + Install AMA
│
├── Step 1: Basics
│   │   Portal: Azure Monitor → Settings → Data Collection Rules → + Create
│   ├── Rule Name
│   ├── Subscription
│   ├── Resource Group
│   ├── Region
│   ├── Platform Type: Windows / Linux / Custom
│   └── Data Collection Endpoint (optional — for custom/IIS logs)
│
├── Step 2: Resources
│   ├── + Add resources → Select VMs / VMSS / Arc servers
│   ├── ⚠️ AMA extension auto-installed on selected VMs
│   └── Optionally enable Data Collection Endpoint
│
├── Step 3: Collect and Deliver
│   ├── + Add data source
│   │   ├── Data source type:
│   │   │   ├── Performance Counters (CPU, Memory, Disk, Network)
│   │   │   │   └── Choose: Basic (pre-set) or Custom (specific counters + intervals)
│   │   │   ├── Windows Event Logs
│   │   │   │   └── Select levels: Critical, Error, Warning, Information, Verbose
│   │   │   │       OR custom XPath queries
│   │   │   ├── Linux Syslog
│   │   │   │   └── Select facilities + log levels (emerg to debug)
│   │   │   ├── Custom Text Logs
│   │   │   └── IIS Logs
│   │   │
│   │   └── Destination
│   │       └── Azure Monitor Logs → Select Log Analytics Workspace
│   │
│   └── + Add more data sources if needed
│
├── Step 4: Review + Create → Create
│
└── ⚠️ AMA = current agent (MMA deprecated)
    ⚠️ One DCR → many VMs, One VM → many DCRs
    ⚠️ Required Role: Monitoring Contributor
```

---

### 23.5 Create Metric Alert

> **Portal:** `Azure Monitor → Alerts → + Create → Alert rule`

```
Create Metric Alert
│
├── Step 1: Select Scope
│   │   Portal: Azure Monitor → Alerts → + Create → Alert rule
│   ├── Select resource(s) — VM, Storage, App Service, etc.
│   └── ⚠️ Multi-resource: select multiple resources of same type in same region
│
├── Step 2: Condition
│   ├── Select Signal: e.g., "Percentage CPU", "Used Capacity", etc.
│   ├── Alert Logic:
│   │   ├── Threshold: Static or Dynamic
│   │   ├── Operator: Greater than / Less than / Equal
│   │   ├── Threshold value: e.g., 80 (for Static)
│   │   │   OR Sensitivity: Low / Medium / High (for Dynamic)
│   │   ├── Aggregation type: Average / Min / Max / Sum / Count
│   │   └── Aggregation granularity: 1 min / 5 min / 15 min / 1 hr
│   ├── Evaluation Frequency: Every 1 / 5 / 15 min
│   └── ⚠️ Static = fixed threshold, Dynamic = ML-based auto-adjusting
│
├── Step 3: Actions
│   ├── Select existing Action Group
│   └── OR + Create action group (see 23.6)
│
├── Step 4: Details
│   ├── Alert Rule Name
│   ├── Description
│   ├── Severity: Sev 0–4
│   ├── Enable upon creation: Yes
│   └── Auto resolve: Yes (stateful) — ⚠️ fires once, auto-resolves
│
└── Step 5: Review + Create → Create
```

---

### 23.6 Create Action Group

> **Portal:** `Azure Monitor → Alerts → Action groups → + Create`

```
Create Action Group
│
├── Step 1: Basics
│   │   Portal: Azure Monitor → Alerts → Action groups → + Create
│   ├── Subscription
│   ├── Resource Group
│   ├── Action Group Name
│   └── Display Name (max 12 chars — shown in SMS/email)
│
├── Step 2: Notifications
│   ├── Notification type:
│   │   ├── Email/SMS/Push/Voice
│   │   │   ├── Email: enter email address (max 100 recipients)
│   │   │   ├── SMS: country code + phone (rate: 1 per 5 min)
│   │   │   ├── Push: Azure mobile app notification
│   │   │   └── Voice: phone call (rate: 1 per 5 min)
│   │   └── Azure Resource Manager Role
│   │       └── Notify all users with specific role (e.g., Owner, Contributor)
│   └── Enter Notification Name
│
├── Step 3: Actions
│   ├── Action type:
│   │   ├── Automation Runbook
│   │   ├── Azure Function
│   │   ├── Event Hub
│   │   ├── ITSM (ServiceNow, etc.)
│   │   ├── Logic App
│   │   ├── Webhook (timeout: 10 sec)
│   │   └── Secure Webhook (Azure AD auth)
│   └── Enter Action Name + configure details
│
├── Step 4: Tags (Optional)
│
└── Step 5: Review + Create → Create
    ⚠️ Rate limits: Email = 100/hr, SMS = 1/5min, Voice = 1/5min
    ⚠️ Max 10 action groups per alert rule
```

---

### 23.7 Create Log Search Alert

> **Portal:** `Azure Monitor → Alerts → + Create → Alert rule`

```
Create Log Search Alert
│
├── Step 1: Select Scope
│   └── Select Log Analytics Workspace (or specific resource)
│
├── Step 2: Condition
│   ├── Signal type: Custom log search
│   ├── Write KQL query
│   │   └── Example: Heartbeat | where TimeGenerated > ago(5m) | summarize count() by Computer
│   ├── Measurement:
│   │   ├── Measure: Table rows / Metric (numeric column)
│   │   └── Aggregation granularity: 5 min to 1 day
│   ├── Alert Logic:
│   │   ├── Operator: Greater than / Less than / Equal
│   │   └── Threshold value: e.g., 0
│   ├── Evaluation Frequency: 5 min to 24 hrs
│   ├── Lookback period: 5 min to 48 hrs
│   └── ⚠️ Frequency cannot be longer than lookback period
│
├── Step 3: Actions → Select Action Group
│
├── Step 4: Details
│   ├── Alert Rule Name, Severity
│   ├── Stateful / Stateless toggle
│   │   └── Stateful = fires once, auto-resolves when condition clears
│   │       Stateless = fires every time condition is met
│   └── Auto mitigation: Resolve when condition clears
│
└── Step 5: Review + Create → Create
```

---

### 23.8 Create Activity Log Alert

> **Portal:** `Azure Monitor → Alerts → + Create → Alert rule`

```
Create Activity Log Alert
│
├── Step 1: Select Scope
│   └── Subscription or Resource Group level
│
├── Step 2: Condition
│   ├── Signal type: Activity Log
│   ├── Select signal:
│   │   ├── e.g., "Create or Update Virtual Machine"
│   │   ├── "Delete Resource Group"
│   │   ├── "Create Role Assignment"
│   │   └── Any ARM operation
│   ├── Event Level: Critical / Error / Warning / Info / Verbose
│   ├── Status: Succeeded / Failed / Started
│   └── Initiated by: specific user (optional)
│
├── Step 3: Actions → Select Action Group (optional)
│
├── Step 4: Details
│   ├── Alert Rule Name
│   └── Enable upon creation: Yes
│
├── Step 5: Review + Create → Create
│
└── ⚠️ Activity Log alerts = FREE (no cost)
    ⚠️ Max 100 Activity Log alert rules per subscription
```

---

### 23.9 Create Service Health Alert

> **Portal:** `Service Health → + Create service health alert`

```
Create Service Health Alert
│
├── Step 1: Navigate
│   └── Search "Service Health" → Alerts → + Create service health alert
│
├── Step 2: Condition
│   ├── Select Subscription
│   ├── Select Service(s): e.g., Virtual Machines, Storage, SQL Database
│   ├── Select Region(s): e.g., East US, West Europe
│   └── Select Event Type(s):
│       ├── Service issue (outage)
│       ├── Planned maintenance
│       ├── Health advisories
│       └── Security advisories
│
├── Step 3: Actions → Select Action Group
│
├── Step 4: Alert Rule Name → Create
│
└── ⚠️ Service Health alerts = FREE
    ⚠️ Only notifies for services + regions YOU select
    ⚠️ Best practice: always configure these
```

---

### 23.10 Create Alert Processing Rule

> **Portal:** `Azure Monitor → Alerts → Alert processing rules → + Create`

```
Create Alert Processing Rule
│
├── Step 1: Scope
│   │   Portal: Azure Monitor → Alerts → Alert processing rules → + Create
│   └── Select scope: Subscription / Resource Group / Resource
│
├── Step 2: Filter (Optional)
│   ├── Alert rule ID
│   ├── Alert rule name
│   ├── Severity: Sev 0–4
│   ├── Monitor condition
│   ├── Resource type
│   └── ⚠️ Filters narrow which alerts are affected
│
├── Step 3: Rule Type
│   ├── Apply action group → select action group to apply
│   └── Suppression → suppress all matching alert notifications
│
├── Step 4: Scheduling
│   ├── Always (apply rule 24/7)
│   ├── One-time: specific start + end datetime
│   └── Recurring: daily / weekly time windows
│       └── ⚠️ Use for maintenance windows
│
├── Step 5: Details
│   ├── Rule Name
│   ├── Subscription, Resource Group
│   └── Enable upon creation
│
└── Step 6: Review + Create → Create
```

---

### 23.11 Enable VM Insights

> **Portal:** `Virtual Machine → Monitoring → Insights`

```
Enable VM Insights
│
├── Path A: From VM Blade
│   │   Portal: Virtual Machine → Monitoring → Insights
│   │
│   ├── Step 1: Click "Enable"
│   ├── Step 2: Select Data Collection Rule
│   │   ├── Use existing DCR
│   │   └── OR Create new DCR
│   │       ├── Select Log Analytics Workspace
│   │       └── Enable Processes and Dependencies (for Map tab)
│   │           └── ⚠️ Installs Dependency Agent (required for Map)
│   ├── Step 3: Configure → VM extensions auto-installed:
│   │   ├── Azure Monitor Agent (AMA)
│   │   └── Dependency Agent (if Map enabled)
│   └── Step 4: Wait for data collection → View Performance / Map tabs
│
└── Path B: From Azure Monitor
    │   Portal: Azure Monitor → Insights → Virtual Machines
    │
    ├── Step 1: "Not Monitored" tab → Select VMs
    ├── Step 2: Click "Enable" → Select DCR
    └── Step 3: Configure → Done

⚠️ Performance tab = AMA only
⚠️ Map tab = AMA + Dependency Agent
⚠️ Both require Log Analytics Workspace
```

---

### 23.12 Configure Autoscale

> **Portal:** `Resource → Settings → Scale out (Autoscale)`

```
Configure Autoscale
│
├── Step 1: Navigate
│   └── VMSS / App Service Plan → Settings → Scale out (Autoscale)
│
├── Step 2: Select "Custom autoscale"
│
├── Step 3: Default Scale Condition
│   ├── Set Instance Limits:
│   │   ├── Minimum: e.g., 2
│   │   ├── Maximum: e.g., 10
│   │   └── Default: e.g., 2 (used when metrics unavailable)
│   │
│   ├── + Add Scale-Out Rule
│   │   ├── Metric source: Current resource or other resource
│   │   ├── Metric: e.g., Percentage CPU
│   │   ├── Time aggregation: Average
│   │   ├── Operator: Greater than
│   │   ├── Threshold: e.g., 70
│   │   ├── Duration: e.g., 10 min
│   │   ├── Action: Increase count by 1
│   │   └── Cool down: 5 min (default)
│   │       ⚠️ Cool down prevents flapping (rapid scale in/out)
│   │
│   └── + Add Scale-In Rule
│       ├── Metric: Percentage CPU
│       ├── Operator: Less than
│       ├── Threshold: e.g., 25
│       ├── Duration: 10 min
│       ├── Action: Decrease count by 1
│       └── Cool down: 5 min
│       ⚠️ Always create BOTH scale-out and scale-in rules
│
├── Step 4: (Optional) + Add Schedule-Based Condition
│   ├── Specify start/end dates OR recurring days
│   ├── Set Min / Max / Default for that schedule
│   └── Add metric rules specific to schedule (optional)
│
├── Step 5: Notifications (Optional)
│   ├── Email co-administrators
│   ├── Additional email addresses
│   └── Webhook URL
│
└── Step 6: Save
    ⚠️ Default cool down = 5 minutes
    ⚠️ Without scale-in rule, resources never scale down
    ⚠️ Default instance count used when metrics are unavailable
```

---

### 23.13 Configure NSG Flow Logs

> **Portal:** `Network Watcher → NSG Flow Logs`

```
Configure NSG Flow Logs
│
├── Step 1: Navigate
│   └── Search "Network Watcher" → Logs → NSG Flow Logs → + Create
│
├── Step 2: Basics
│   ├── Select Subscription
│   ├── Select NSG (or multiple NSGs)
│   ├── Select Storage Account (same region as NSG)
│   │   └── ⚠️ Storage account must be in same region as NSG
│   └── Set Retention: 0–365 days (0 = forever)
│
├── Step 3: Configuration
│   ├── Flow Log Version: Version 1 or Version 2
│   │   └── ⚠️ Version 2 includes throughput info (bytes/packets)
│   │
│   └── Enable Traffic Analytics (optional)
│       ├── Select Log Analytics Workspace
│       ├── Processing Interval: 10 min or 1 hr
│       │   └── ⚠️ 10 min = more granular, higher cost
│       └── Traffic Analytics provides: Geo-map, Top talkers, Security threats
│
├── Step 4: Tags (Optional)
│
└── Step 5: Review + Create → Create
    ⚠️ NSG flow logs stored in Storage Account (JSON)
    ⚠️ Traffic Analytics needs Log Analytics Workspace
```

---

### 23.14 Create Azure Dashboard

> **Portal:** `Home → Dashboard`

```
Create Azure Dashboard
│
├── Step 1: Navigate
│   └── Home → Dashboard → + New Dashboard → Blank Dashboard
│
├── Step 2: Add Tiles
│   ├── From Tile Gallery (left panel):
│   │   ├── Clock, Markdown, Metrics chart
│   │   ├── Resource groups, All resources
│   │   └── Video, Web content links
│   │
│   └── Pin from other blades:
│       ├── Metrics Explorer → Pin to dashboard
│       ├── Log Analytics → Pin query result to dashboard
│       ├── Workbooks → Pin to dashboard
│       └── Any resource overview → Pin tiles
│
├── Step 3: Arrange and Resize Tiles
│   └── Drag, resize, reorder as needed
│
├── Step 4: Save
│   └── Enter Dashboard Name → Save
│
├── Step 5: Share (Optional)
│   │   Portal: Dashboard → Share
│   ├── Publish to resource group: "dashboards" (default)
│   ├── Assign RBAC access: Reader role to view
│   └── ⚠️ Viewers need Reader on dashboard + data-source resources
│
└── ⚠️ Max 100 shared dashboards per subscription
    ⚠️ Shared dashboards = Azure resources (ARM)
```

---

### 23.15 Configure Application Insights

> **Portal:** `Home → + Create a resource → Search "Application Insights" → Create`

```
Configure Application Insights
│
├── Step 1: Create Application Insights Resource
│   │   Portal: Home → + Create a resource → Application Insights → Create
│   ├── Subscription
│   ├── Resource Group
│   ├── Name
│   ├── Region
│   └── Resource Mode: Workspace-based (recommended)
│       └── Select Log Analytics Workspace
│       ⚠️ Workspace-based = logs go to Log Analytics (recommended)
│       ⚠️ Classic mode = deprecated
│
├── Step 2: Instrument Your App
│   ├── Get Connection String / Instrumentation Key
│   │   └── Portal: Application Insights → Overview → Connection String
│   ├── Install SDK or enable auto-instrumentation:
│   │   ├── .NET / Java / Node.js / Python → Add SDK
│   │   └── App Service → Enable via portal (auto-instrumentation)
│   │       Portal: App Service → Settings → Application Insights → Turn On
│   └── ⚠️ Connection String (preferred) over Instrumentation Key
│
├── Step 3: Configure Availability Test
│   │   Portal: Application Insights → Investigate → Availability → + Add Test
│   ├── Test Type: Standard test
│   ├── URL: Enter web app URL
│   ├── Test frequency: 5 / 10 / 15 min
│   ├── Test locations: Select Azure regions to test from
│   ├── Success criteria: HTTP status code, response time, content match
│   └── Alerts: automatically created for failures
│
├── Step 4: Use Built-in Features
│   ├── Performance → Request duration, server response times
│   ├── Failures → Failed requests, exceptions
│   ├── Application Map → Visual dependency map
│   ├── Live Metrics → Real-time telemetry (no delay)
│   └── Smart Detection → AI anomaly detection (auto-enabled)
│
└── ⚠️ First 5 GB/month free per billing account
    ⚠️ Workspace-based is the recommended mode
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
