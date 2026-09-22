<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Network Watcher — AZ-104 Revision Notes

---

## 1. What is Azure Network Watcher?

- **Regional network monitoring & diagnostics service** for Azure IaaS resources
- Provides tools to **monitor, diagnose, view metrics, and enable/disable logs** for Azure virtual network resources
- **Automatically enabled** when you create or update a VNet in your subscription (creates a `NetworkWatcherRG` resource group)
- One Network Watcher instance **per region per subscription**
- Does NOT monitor PaaS or Web Analytics — **IaaS only** (VMs, VNets, NSGs, Load Balancers, VPN Gateways, Application Gateways)

> ⚠️ **EXAM TIP:** Network Watcher is **automatically enabled per region** when a VNet is created. It is NOT a global service — it is **regional**. You get one instance per region per subscription.

> ⚠️ **EXAM TIP:** Network Watcher is created in a resource group called **NetworkWatcherRG** by default. Do NOT delete this RG.

---

## 2. Key Components / Tools

| Category | Tool | Purpose |
|---|---|---|
| **Monitoring** | Connection Monitor | End-to-end connection monitoring (multi-source/destination) |
| **Monitoring** | Topology | Visual map of VNet resources and relationships |
| **Diagnostics** | IP Flow Verify | Test if packet allowed/denied by NSG rules |
| **Diagnostics** | Next Hop | Determine next hop for a packet from a VM |
| **Diagnostics** | Connection Troubleshoot | One-time connectivity check (point-to-point) |
| **Diagnostics** | Packet Capture | Capture packets to/from a VM NIC |
| **Diagnostics** | VPN Troubleshoot | Diagnose VPN Gateway & connection issues |
| **Diagnostics** | NSG Diagnostics | Evaluate NSG rules applied to traffic |
| **Logging** | NSG Flow Logs | Log NSG allow/deny decisions for network flows |
| **Logging** | VNet Flow Logs | Log traffic flows at the VNet level (newer) |
| **Logging** | Traffic Analytics | Analyze NSG/VNet flow logs for insights |
| **Diagnostics** | Effective Security Rules | View all NSG rules applied to a NIC |

---

## 3. Monitoring Tools

---

### 3.1 Connection Monitor

- **Continuous, end-to-end connectivity monitoring** between sources and destinations
- Measures **latency, reachability, packet loss** over time
- Sources: **Azure VMs, on-prem machines** (with Log Analytics agent)
- Destinations: **Azure VMs, URLs, IPs, FQDNs, any endpoint** (port + protocol)
- Supports **multi-source → multi-destination** test groups
- **Replaces** the legacy Connection Monitor (Classic)
- Requires **Network Watcher Agent VM extension** on source VMs
- Checks run at configurable intervals (default: **30 seconds**)
- Stores results in **Log Analytics workspace**

#### Portal Path — Create Connection Monitor
```
Network Watcher → Connection monitor → + Create →
Name, Region, Workspace →
Test Groups: Add sources, destinations, test configurations →
Protocol: TCP/HTTP/ICMP, Port, Threshold →
Create
```

> ⚠️ **EXAM TIP:** Connection Monitor requires the **Network Watcher Agent VM extension** installed on source VMs. Without it, monitoring won't work.

> ⚠️ **EXAM TIP:** Connection Monitor is for **continuous** monitoring. For a **one-time** check, use **Connection Troubleshoot** instead.

---

### 3.2 Topology

- **Visual, interactive diagram** of VNet resources and their relationships
- Shows: VNets, subnets, NICs, VMs, NSGs, route tables, public IPs, load balancers
- Scoped to a **resource group** (select RG to see its network topology)
- Read-only — for visualization only, no configuration
- Can be **downloaded as SVG** for documentation

#### Portal Path — View Topology
```
Network Watcher → Topology →
Subscription, Resource Group, VNet →
View diagram → Download SVG (optional)
```

> ⚠️ **EXAM TIP:** Topology is scoped by **resource group**. You must select a specific RG to view its network layout. It only shows resources within that RG.

---

## 4. Network Diagnostic Tools

---

### 4.1 IP Flow Verify

- **Tests if a specific packet (5-tuple) is allowed or denied** by NSG rules
- Input: VM, NIC, direction (inbound/outbound), protocol (TCP/UDP), local IP/port, remote IP/port
- Output: **Access allowed** or **Access denied** + **which NSG rule** caused the decision
- Checks rules at **NIC level** and **subnet level** NSGs
- **Use case:** Troubleshoot why traffic to/from a VM is blocked

#### Portal Path — IP Flow Verify
```
Network Watcher → IP flow verify →
VM, Network interface, Protocol: TCP/UDP,
Direction: Inbound/Outbound,
Local IP, Local port, Remote IP, Remote port →
Check
```

> ⚠️ **EXAM TIP:** IP Flow Verify tells you **which NSG rule** is blocking/allowing traffic — it is the #1 tool for NSG troubleshooting. If the exam says "a VM can't receive traffic," think IP Flow Verify first.

> ⚠️ **EXAM TIP:** IP Flow Verify checks BOTH the **NIC-level NSG** and **subnet-level NSG** rules. It reports the effective result.

---

### 4.2 Next Hop

- **Shows the next hop type and IP address** for a packet leaving a VM
- Input: VM, source IP, destination IP
- Output: Next hop type + IP address
- Next hop types returned:

| Next Hop Type | Meaning |
|---|---|
| **VirtualAppliance** | Routes through an NVA (with IP shown) |
| **VNetGateway** | Routes through VPN/ExpressRoute gateway |
| **VNetLocal** | Within the same VNet |
| **Internet** | Routes to internet |
| **None** | Packet is dropped (no route) |
| **VirtualNetworkServiceEndpoint** | Routes to a service endpoint |

#### Portal Path — Next Hop
```
Network Watcher → Next hop →
VM, Network interface,
Source IP address, Destination IP address →
Next hop
```

> ⚠️ **EXAM TIP:** Next Hop = routing troubleshooting. If traffic is going to the wrong place (e.g., dropped or misrouted by UDR), use **Next Hop** to check the effective route.

> ⚠️ **EXAM TIP:** Next Hop result of **None** = packet will be **dropped**. This typically means no matching route or a route with next hop type "None."

---

### 4.3 Connection Troubleshoot

- **One-time connectivity test** between a source and destination
- Source: Azure VM (requires Network Watcher Agent extension)
- Destination: VM, URI, FQDN, or IP address + port
- Shows: **Reachability status, latency, number of hops, each hop details**
- Shows if connection is **Reachable, Unreachable, or Degraded**
- Checks: NSGs, UDRs, firewalls, DNS resolution in the path

#### Portal Path — Connection Troubleshoot
```
Network Watcher → Connection troubleshoot →
Source: VM, NIC →
Destination: VM / URI / IP + Port →
Protocol: TCP/ICMP →
Check
```

> ⚠️ **EXAM TIP:** Connection Troubleshoot = **one-time** check. Connection Monitor = **continuous** monitoring. Know when to use which.

---

### 4.4 Packet Capture

- **Capture network packets on a VM NIC** for analysis
- Requires **Network Watcher Agent VM extension** on the VM
- Capture saved to: **VM local disk**, **Storage account**, or **both**
- File format: **.cap** (can be analyzed in Wireshark)
- Supports **filters**: protocol, local/remote IP, local/remote port
- Max capture duration: configurable (default: **5 hours / 18000 seconds**)
- Max file size: configurable (default: **1 GB**)
- Max packets: configurable

#### Portal Path — Start Packet Capture
```
Network Watcher → Packet capture → + Add →
VM, Capture name →
Storage account: Yes/No, File path on VM: Yes/No →
Max bytes per packet, Max bytes per session, Time limit →
Filters: Protocol, Local IP, Local port, Remote IP, Remote port →
Start
```

> ⚠️ **EXAM TIP:** Packet capture requires **Network Watcher Agent VM extension**. If the extension is not installed, packet capture will fail.

> ⚠️ **EXAM TIP:** Packet capture files are **.cap** format (not .pcap). Can be opened with Wireshark or Network Monitor.

---

### 4.5 VPN Troubleshoot

- Diagnose **VPN Gateway** and **VPN connection** health issues
- Checks: gateway health, connection configuration, throughput, errors
- Returns diagnostic results in a **storage account** (logs)
- Provides: **error code, error message, recommended action**
- Works with **Site-to-Site, Point-to-Site, and VNet-to-VNet** VPN connections

#### Portal Path — VPN Troubleshoot
```
Network Watcher → VPN troubleshoot →
Select VPN Gateway or Connection →
Storage account (for logs) →
Troubleshoot
```

> ⚠️ **EXAM TIP:** VPN Troubleshoot stores diagnostic logs in a **storage account** — you must specify one. It cannot display results without storage.

---

### 4.6 NSG Diagnostics

- **Evaluates NSG rules** for a specific traffic flow
- Similar to IP Flow Verify but provides **more detailed analysis**
- Shows all matching NSG rules (both NIC and subnet level)
- Input: VM, direction, protocol, source/destination IP and port
- Output: List of all evaluated NSG rules and their effect

#### Portal Path — NSG Diagnostics
```
Network Watcher → NSG diagnostics →
Target VM, Direction,
Protocol, Source, Destination →
Run diagnostics
```

---

### 4.7 Effective Security Rules

- View **all NSG rules effectively applied to a VM NIC** (combined NIC + subnet NSGs)
- Shows the aggregated, ordered list of rules as the platform evaluates them
- Useful to understand why specific traffic is allowed or blocked

#### Portal Path — Effective Security Rules
```
Network Watcher → Effective security rules →
Select VM → View effective rules
```

**Alternative path (from VM):**
```
VM → Networking → Effective security rules tab
```

> ⚠️ **EXAM TIP:** Effective Security Rules shows the **merged view** of NIC-level + subnet-level NSG rules. Helpful when multiple NSGs are layered.

---

## 5. IP Flow Verify vs Next Hop vs Connection Troubleshoot vs NSG Diagnostics

| Feature | **IP Flow Verify** | **Next Hop** | **Connection Troubleshoot** | **NSG Diagnostics** |
|---|---|---|---|---|
| **Purpose** | Check NSG allow/deny | Check routing path | End-to-end connectivity test | Detailed NSG rule evaluation |
| **Checks** | NSG rules only | Route tables (UDR, system, BGP) | NSGs + Routes + Firewalls + DNS | All NSG rules in detail |
| **Output** | Allow/Deny + rule name | Next hop type + IP | Reachable/Unreachable + hop trace | Full rule evaluation list |
| **Use when** | Traffic blocked by NSG? | Traffic misrouted? | Can't connect at all? | Need detailed NSG analysis |
| **Scope** | Single packet (5-tuple) | Single source→dest IP | Source VM → destination endpoint | Single traffic flow |
| **One-time/Continuous** | One-time | One-time | One-time | One-time |

> ⚠️ **EXAM TIP:** Scenario-based question: "VM can't reach the internet" → use **Next Hop** (routing). "VM can't receive RDP" → use **IP Flow Verify** (NSG). "VM can't connect to SQL" → use **Connection Troubleshoot** (end-to-end).

---

## 6. Logging Tools

---

### 6.1 NSG Flow Logs

- Log **all IP traffic (flows)** passing through an **NSG**
- Captures: source IP, dest IP, source port, dest port, protocol, action (allow/deny)
- **Two versions:**

| Feature | **Version 1** | **Version 2** |
|---|---|---|
| **Flow info** | Basic (5-tuple + action) | Basic + **bytes, packets, flow state** |
| **Throughput info** | ❌ | ✅ Bytes and packets per flow |
| **Flow state** | ❌ | ✅ Begin, Continuing, End |
| **Recommended** | Legacy | ✅ Always use V2 |

- Stored as **JSON** in a **Storage Account** (in `insights-logs-networksecuritygroupflowevent` container)
- Retention: **0–365 days** (0 = forever)
- Can be analyzed with **Traffic Analytics** or exported to SIEM

#### Portal Path — Enable NSG Flow Logs
```
Network Watcher → NSG flow logs → + Create →
Select NSG →
Storage account: Select →
Retention days: 0–365 →
Flow log version: Version 2 →
Enable Traffic Analytics: Yes/No →
Traffic Analytics interval: 10 min / 60 min →
Log Analytics workspace (for Traffic Analytics) →
Create
```

> ⚠️ **EXAM TIP:** NSG Flow Logs are stored in a **Storage Account** (NOT Log Analytics directly). Traffic Analytics reads from Storage Account and sends insights to Log Analytics.

> ⚠️ **EXAM TIP:** Always use **Version 2** for NSG Flow Logs — it includes byte/packet counts and flow state. Version 1 is legacy.

> ⚠️ **EXAM TIP:** NSG Flow Logs are attached to the **NSG**, not to the NIC or subnet. One flow log configuration per NSG.

---

### 6.2 VNet Flow Logs

- **Newer replacement** for NSG Flow Logs — logs traffic at the **VNet level**
- Captures all traffic flowing through the VNet (not just NSG-evaluated traffic)
- Logged per **Virtual Network** (not per NSG)
- Includes traffic that bypasses NSGs (e.g., traffic within a subnet using Azure default rules)
- Also stored in **Storage Account**
- Supports **Traffic Analytics** integration

| Feature | **NSG Flow Logs** | **VNet Flow Logs** |
|---|---|---|
| **Scope** | Per NSG | Per VNet |
| **Coverage** | Only NSG-evaluated flows | All VNet traffic (including inter-subnet) |
| **Granularity** | NSG level | VNet level |
| **Encrypted VNets** | ❌ | ✅ |
| **Recommended** | Legacy | ✅ Preferred going forward |

#### Portal Path — Enable VNet Flow Logs
```
Network Watcher → VNet flow logs → + Create →
Select VNet →
Storage account →
Retention days →
Enable Traffic Analytics: Yes/No →
Create
```

> ⚠️ **EXAM TIP:** VNet Flow Logs are the **newer, recommended** approach over NSG Flow Logs. They capture broader traffic at the VNet level.

---

### 6.3 Traffic Analytics

- **Analytics solution** that processes NSG/VNet Flow Logs for visualized insights
- Shows: top talkers, traffic distribution, open ports, geo distribution, malicious IPs
- Requires: **Log Analytics workspace** + **NSG/VNet Flow Logs enabled**
- Processing interval: **10 minutes** or **60 minutes**
- Visualizations in Azure portal (dashboard) + Log Analytics queries (KQL)

#### Portal Path — Enable Traffic Analytics
```
Network Watcher → Traffic Analytics →
(Enabled during flow log creation, or)
NSG flow log → Edit → Enable Traffic Analytics: Yes →
Processing interval: Every 10 min / Every 60 min →
Log Analytics workspace: Select →
Save
```

> ⚠️ **EXAM TIP:** Traffic Analytics requires **both** a Storage Account (for flow logs) AND a Log Analytics workspace (for analysis). Processing interval of **10 min** gives faster insights but costs more.

> ⚠️ **EXAM TIP:** Traffic Analytics can identify **malicious IP addresses** connecting to your resources — this is a security insight.

---

## 7. Network Watcher Agent VM Extension

- **Required for:** Packet Capture, Connection Monitor, Connection Troubleshoot
- Must be installed on the **source VM** (the VM being tested)
- Extension name:
  - **Windows:** `Microsoft.Azure.NetworkWatcher` (`AzureNetworkWatcherExtension`)
  - **Linux:** `Microsoft.Azure.NetworkWatcher.Linux` (`NetworkWatcherAgentLinux`)
- Auto-installed when using features from the portal (sometimes)
- Can be manually installed via CLI/PowerShell/Portal

| Tool | Extension Required? |
|---|---|
| IP Flow Verify | ❌ No |
| Next Hop | ❌ No |
| Effective Security Rules | ❌ No |
| NSG Diagnostics | ❌ No |
| **Packet Capture** | ✅ Yes |
| **Connection Monitor** | ✅ Yes |
| **Connection Troubleshoot** | ✅ Yes |
| Topology | ❌ No |
| NSG Flow Logs | ❌ No |
| VPN Troubleshoot | ❌ No |

#### CLI — Install Extension
```bash
# Windows VM
az vm extension set --vm-name <vm> -g <rg> \
  --name NetworkWatcherAgentWindows \
  --publisher Microsoft.Azure.NetworkWatcher

# Linux VM
az vm extension set --vm-name <vm> -g <rg> \
  --name NetworkWatcherAgentLinux \
  --publisher Microsoft.Azure.NetworkWatcher
```

> ⚠️ **EXAM TIP:** Know which tools require the VM extension: **Packet Capture, Connection Monitor, Connection Troubleshoot**. IP Flow Verify and Next Hop do NOT require it.

---

## 8. Security & RBAC

### RBAC Roles

| Role | Permissions |
|---|---|
| **Network Contributor** | Full access to Network Watcher and all tools |
| **Reader** | View-only access to Network Watcher diagnostics and topology |
| **Monitoring Contributor** | Access to monitoring features |
| **Classic Network Contributor** | Classic networking only (not recommended) |

### Required Permissions (Granular)

| Action | Required Permission |
|---|---|
| Enable/disable Network Watcher | `Microsoft.Network/networkWatchers/write` |
| IP Flow Verify | `Microsoft.Network/networkWatchers/ipFlowVerify/action` |
| Next Hop | `Microsoft.Network/networkWatchers/nextHop/action` |
| Packet Capture | `Microsoft.Network/networkWatchers/packetCaptures/write` |
| Connection Monitor | `Microsoft.Network/networkWatchers/connectionMonitors/write` |
| NSG Flow Logs | `Microsoft.Network/networkWatchers/configureFlowLog/action` |
| Topology | `Microsoft.Network/networkWatchers/topology/action` |
| VPN Troubleshoot | `Microsoft.Network/networkWatchers/troubleshoot/action` |

### Custom RBAC Role for Network Watcher

- Create custom role with only specific Network Watcher actions
- Useful for giving helpdesk staff access to diagnostic tools only (no modify permissions on the network itself)

> ⚠️ **EXAM TIP:** To use Network Watcher tools, a user needs **specific action permissions** on `Microsoft.Network/networkWatchers/*`. The **Network Contributor** role includes all of these.

> ⚠️ **EXAM TIP:** Network Watcher operates in the **NetworkWatcherRG** resource group. RBAC must be scoped to include access to this RG.

---

## 9. Monitoring & Alerts

### Key Metrics Available

| Metric | Description |
|---|---|
| Connection Monitor checks | Pass/fail status of connection tests |
| Round-trip time | Latency from source to destination |
| Packet loss | % of packets lost in connection tests |

### Alerts via Connection Monitor
```
Network Watcher → Connection monitor →
Select monitor → Alerts → + Create alert rule →
Condition: Test Failed / Latency threshold →
Action Group: Email/SMS/Webhook →
Create
```

### Integration with Azure Monitor
- Connection Monitor metrics available in **Azure Monitor**
- Create **metric alerts** on connectivity and latency
- Log Analytics queries (KQL) for flow log analysis
- Workbooks for custom dashboards

#### Portal Path — Network Watcher Diagnostic Logs
```
Network Watcher → Diagnostic settings →
+ Add diagnostic setting →
Send to: Log Analytics workspace / Storage Account / Event Hub →
Select log categories →
Save
```

---

## 10. Pricing Key Points

| Component | Cost Model |
|---|---|
| **Network Watcher** | Free to enable (no charge for the service itself) |
| **NSG Flow Logs** | Per GB of logs collected (flow log data stored in storage account) |
| **VNet Flow Logs** | Per GB of logs collected |
| **Traffic Analytics** | Per GB processed + Log Analytics ingestion costs |
| **Packet Capture** | Free (storage costs apply if stored in storage account) |
| **Connection Monitor** | Per test per month (first 360 tests/month free) |
| **IP Flow Verify** | Free (included) |
| **Next Hop** | Free (included) |
| **Connection Troubleshoot** | Free (included) |
| **VPN Troubleshoot** | Per troubleshoot check |
| **Topology** | Free |

> ⚠️ **EXAM TIP:** Network Watcher itself is **free**. Costs come from **flow logs storage**, **Traffic Analytics processing**, and **Connection Monitor** (beyond 360 free tests/month).

> ⚠️ **EXAM TIP:** Connection Monitor first **360 tests/month are free**. After that, per-test charges apply.

---

## 11. CLI / PowerShell Commands

| Action | CLI Command |
|---|---|
| Enable Network Watcher | `az network watcher configure -g <rg> -l <location> --enabled true` |
| List Network Watchers | `az network watcher list` |
| IP Flow Verify | `az network watcher test-ip-flow --vm <vm> --direction <Inbound/Outbound> --protocol <TCP/UDP> --local <ip:port> --remote <ip:port>` |
| Next Hop | `az network watcher show-next-hop --vm <vm> --source-ip <src> --dest-ip <dest>` |
| Connection Troubleshoot | `az network watcher test-connectivity --source-resource <vm-id> --dest-address <ip/fqdn> --dest-port <port>` |
| Start Packet Capture | `az network watcher packet-capture create --vm <vm> -g <rg> --name <capture-name> --storage-account <sa-id>` |
| Stop Packet Capture | `az network watcher packet-capture stop --name <capture-name> -l <location>` |
| Enable NSG Flow Log | `az network watcher flow-log create --nsg <nsg-id> --storage-account <sa-id> --enabled true --log-version 2` |
| Show Topology | `az network watcher show-topology -g <rg>` |
| VPN Troubleshoot | `az network watcher troubleshooting start --resource <vpn-gw-id> --storage-account <sa-id> --storage-path <blob-uri>` |
| Show Effective Security Rules | `az network watcher show-security-group-view --vm <vm>` |

### PowerShell Equivalents

| Action | PowerShell Command |
|---|---|
| Enable Network Watcher | `New-AzNetworkWatcher -Name <name> -ResourceGroupName NetworkWatcherRG -Location <location>` |
| IP Flow Verify | `Test-AzNetworkWatcherIPFlow -NetworkWatcher <nw> -TargetVMResourceId <vm-id> -Direction <Inbound/Outbound> -Protocol <TCP/UDP> -LocalIPAddress <ip> -LocalPort <port> -RemoteIPAddress <ip> -RemotePort <port>` |
| Next Hop | `Get-AzNetworkWatcherNextHop -NetworkWatcher <nw> -TargetVMResourceId <vm-id> -SourceIPAddress <ip> -DestinationIPAddress <ip>` |
| Packet Capture | `New-AzNetworkWatcherPacketCapture -NetworkWatcher <nw> -TargetVMResourceId <vm-id> -PacketCaptureName <name> -StorageAccountId <sa-id>` |

> ⚠️ **EXAM TIP:** CLI flow log command is `az network watcher flow-log create` with `--log-version 2`. PowerShell uses `New-AzNetworkWatcherPacketCapture` for packet captures.

---

## 12. Limitations & Constraints

| Constraint | Limit |
|---|---|
| Network Watcher instances per region per subscription | **1** |
| Connection Monitor tests per monitor | **Varies by configuration** |
| Connection Monitor free tests per month | **360** |
| Packet capture max size | **1 GB** default (configurable) |
| Packet capture max duration | **5 hours (18,000 seconds)** default |
| Packet capture max sessions per region | **10,000** |
| NSG Flow Log retention | **0–365 days** (0 = indefinite) |
| Flow log storage | Must be in **same region** as NSG |
| Traffic Analytics processing interval | **10 min** or **60 min** |
| Network Watcher Agent extension | Required on **source VM** for packet capture, connection monitor, and connection troubleshoot |

> ⚠️ **EXAM TIP:** Flow log storage account must be in the **same region** as the NSG being logged. Cross-region flow log storage is NOT supported.

> ⚠️ **EXAM TIP:** There is only **1 Network Watcher per region per subscription**. It is automatically created in `NetworkWatcherRG`.

---

## 13. Quick-Fire Exam Points ⚡

1. Network Watcher = **regional** network monitoring & diagnostics service for **IaaS** resources
2. **Automatically enabled** when a VNet is created/updated — creates `NetworkWatcherRG`
3. **One Network Watcher per region per subscription** — cannot have multiples
4. **IP Flow Verify** = checks if NSG allows/denies a specific packet 5-tuple → reports which rule
5. **Next Hop** = shows routing decision for a packet → identifies route type (Internet, VNet, NVA, None)
6. **Next Hop = None** means packet will be **dropped** (no matching route)
7. **Connection Troubleshoot** = one-time end-to-end connectivity check (NSGs + routes + DNS)
8. **Connection Monitor** = **continuous** monitoring with alerts (latency, packet loss, reachability)
9. Connection Monitor requires **Network Watcher Agent VM extension** on source VMs
10. **Packet Capture** = captures packets on VM NIC → stored as **.cap** file → needs VM extension
11. Packet capture default max: **5 hours** duration, **1 GB** file size
12. **NSG Flow Logs** = logs all traffic through an NSG → stored as **JSON in Storage Account**
13. Always use NSG Flow Log **Version 2** (includes bytes, packets, flow state)
14. **VNet Flow Logs** = newer, logs at VNet level (broader coverage than NSG Flow Logs)
15. Flow log storage account must be in the **same region** as the NSG/VNet
16. **Traffic Analytics** = analyzes flow logs → requires Log Analytics workspace + Storage Account
17. Traffic Analytics processing interval: **10 min** (faster, more expensive) or **60 min**
18. Traffic Analytics can detect **malicious IP connections** (security insight)
19. **Topology** = visual diagram of VNet resources → scoped to **resource group** → download as SVG
20. **VPN Troubleshoot** = diagnoses VPN Gateway health → requires a **storage account** for logs
21. **Effective Security Rules** = merged view of NIC + subnet NSG rules applied to a VM
22. Tools requiring VM extension: **Packet Capture, Connection Monitor, Connection Troubleshoot**
23. Tools NOT requiring VM extension: **IP Flow Verify, Next Hop, Topology, NSG Flow Logs, Effective Security Rules**
24. Network Watcher itself is **free** — costs from flow logs storage, Traffic Analytics, Connection Monitor
25. Connection Monitor: first **360 tests/month free**
26. NSG Flow Log retention: **0–365 days** (0 = keep forever)
27. Network Watcher uses resource group **NetworkWatcherRG** — do NOT delete it
28. **Network Contributor** role provides full access to all Network Watcher tools
29. NSG Flow Logs stored in: `insights-logs-networksecuritygroupflowevent` container in storage account
30. "VM can't receive traffic" → **IP Flow Verify**. "Traffic goes wrong path" → **Next Hop**. "Can't connect at all" → **Connection Troubleshoot**

---

## 14. Step-by-Step Configuration Mind Maps 🗺️

---

### 14.1 Enable Network Watcher

> **Portal:** `Network Watcher → Overview`

```
Enable Network Watcher
│
├── Automatic Enablement
│   ├── Network Watcher auto-enabled when creating/updating a VNet
│   ├── Creates resource group: NetworkWatcherRG
│   └── One instance per region per subscription
│       ⚠️ Do NOT delete NetworkWatcherRG
│
├── Manual Enable (if disabled)
│   │   Portal: Network Watcher → Overview → Regions
│   ├── Select subscription
│   ├── Select region
│   ├── Status: Enabled / Disabled
│   └── Toggle to Enabled
│   ⚠️ RBAC: Network Contributor or higher
│
├── CLI
│   └── az network watcher configure -g NetworkWatcherRG -l eastus --enabled true
│
└── PowerShell
    └── New-AzNetworkWatcher -Name NetworkWatcher_eastus -ResourceGroupName NetworkWatcherRG -Location eastus
```

---

### 14.2 Run IP Flow Verify

> **Portal:** `Network Watcher → IP flow verify`

```
IP Flow Verify
│
├── Prerequisites
│   ├── Network Watcher enabled in VM's region
│   ├── Target VM must be running
│   └── NSG attached to VM NIC or subnet
│       ⚠️ VM extension NOT required for this tool
│
├── Step 1: Select Target
│   │   Portal: Network Watcher → IP flow verify
│   ├── Subscription
│   ├── Resource group
│   ├── Virtual machine
│   └── Network interface (if VM has multiple NICs)
│
├── Step 2: Configure Packet Details
│   ├── Protocol: TCP / UDP
│   ├── Direction: Inbound / Outbound
│   ├── Local IP address (VM's private IP)
│   ├── Local port
│   ├── Remote IP address
│   └── Remote port
│
├── Step 3: Check
│   └── Click "Check"
│
└── Output
    ├── Access: Allowed / Denied
    ├── NSG rule name that matched
    └── NSG where rule exists (NIC-level or subnet-level)
        ⚠️ Checks BOTH NIC and subnet NSGs
```

---

### 14.3 Run Next Hop Diagnostic

> **Portal:** `Network Watcher → Next hop`

```
Next Hop
│
├── Prerequisites
│   ├── Network Watcher enabled in VM's region
│   └── Target VM must be running
│       ⚠️ VM extension NOT required
│
├── Step 1: Select Target
│   │   Portal: Network Watcher → Next hop
│   ├── Subscription, Resource group
│   ├── Virtual machine
│   └── Network interface
│
├── Step 2: Enter Addresses
│   ├── Source IP address (VM's private IP)
│   └── Destination IP address (target you're testing)
│
├── Step 3: Next hop
│   └── Click "Next hop"
│
└── Output
    ├── Next hop type: Internet / VNetLocal / VirtualAppliance /
    │                  VNetGateway / None
    ├── Next hop IP address (if applicable)
    └── Route table ID (which route table matched)
        ⚠️ "None" = packet DROPPED (no route)
        ⚠️ Checks system routes, UDRs, and BGP routes
```

---

### 14.4 Configure Packet Capture

> **Portal:** `Network Watcher → Packet capture`

```
Configure Packet Capture
│
├── Prerequisites
│   ├── Network Watcher enabled in VM's region
│   ├── ⚠️ Network Watcher Agent VM extension installed on target VM
│   │   ├── Windows: AzureNetworkWatcherExtension
│   │   └── Linux: NetworkWatcherAgentLinux
│   ├── VM must be running
│   └── Storage account (if storing captures externally)
│
├── Step 1: Create Packet Capture
│   │   Portal: Network Watcher → Packet capture → + Add
│   ├── Target VM: Select
│   └── Packet capture name
│
├── Step 2: Storage Configuration
│   ├── Storage account: Yes → Select account
│   │   ⚠️ Storage account recommended for persistent captures
│   ├── File path on VM: Yes → Enter local path (e.g., C:\captures\)
│   └── Can enable both simultaneously
│
├── Step 3: Capture Limits
│   ├── Maximum bytes per packet: default 0 (entire packet)
│   ├── Maximum bytes per session: default 1073741824 (1 GB)
│   └── Time limit in seconds: default 18000 (5 hours)
│       ⚠️ Capture auto-stops when any limit is reached
│
├── Step 4: Filters (Optional)
│   ├── Protocol: TCP / UDP / Any
│   ├── Local IP address
│   ├── Local port
│   ├── Remote IP address
│   └── Remote port
│       Note: Multiple filters = OR logic
│
├── Step 5: Start
│   └── Click "Add" to start capture
│
└── Managing Captures
    ├── Stop: Network Watcher → Packet capture → Select → Stop
    ├── Download: From storage account (blob) or VM local disk
    └── Analyze: Open .cap file in Wireshark
        ⚠️ File format is .cap
```

---

### 14.5 Configure NSG Flow Logs

> **Portal:** `Network Watcher → NSG flow logs`

```
Configure NSG Flow Logs
│
├── Prerequisites
│   ├── Network Watcher enabled in NSG's region
│   ├── NSG exists
│   ├── Storage account in same region as NSG
│   │   ⚠️ Cross-region storage NOT supported
│   └── Microsoft.Insights provider registered
│       (az provider register --namespace Microsoft.Insights)
│
├── Step 1: Create Flow Log
│   │   Portal: Network Watcher → NSG flow logs → + Create
│   ├── Select NSG
│   └── Flow log name
│
├── Step 2: Storage Configuration
│   ├── Storage account: Select (same region as NSG)
│   └── Retention (days): 0–365
│       ├── 0 = keep forever
│       └── 90 = common exam default recommendation
│       ⚠️ Storage costs increase with retention
│
├── Step 3: Flow Log Version
│   ├── Version 1: Basic (5-tuple + action)
│   └── Version 2: Extended (+ bytes, packets, flow state)
│       ⚠️ Always select Version 2
│
├── Step 4: Traffic Analytics (Optional but Recommended)
│   ├── Enable Traffic Analytics: Yes
│   ├── Processing interval:
│   │   ├── Every 10 minutes (faster, more expensive)
│   │   └── Every 60 minutes (slower, cheaper)
│   └── Log Analytics workspace: Select
│       ⚠️ Requires Log Analytics workspace
│       ⚠️ Additional costs for processing + ingestion
│
└── Step 5: Create
    └── RBAC: Network Contributor or
        Microsoft.Network/networkWatchers/configureFlowLog/action
```

---

### 14.6 Configure VNet Flow Logs

> **Portal:** `Network Watcher → VNet flow logs`

```
Configure VNet Flow Logs
│
├── Prerequisites
│   ├── Network Watcher enabled in VNet's region
│   ├── VNet exists
│   ├── Storage account in same region as VNet
│   └── Microsoft.Insights provider registered
│
├── Step 1: Create Flow Log
│   │   Portal: Network Watcher → VNet flow logs → + Create
│   ├── Select VNet (not NSG — this is VNet-level)
│   └── Flow log name
│
├── Step 2: Storage & Retention
│   ├── Storage account: Select
│   └── Retention (days): 0–365
│
├── Step 3: Traffic Analytics
│   ├── Enable: Yes/No
│   ├── Processing interval: 10 min / 60 min
│   └── Log Analytics workspace
│
└── Step 4: Create
    ⚠️ VNet Flow Logs are newer and capture broader
       traffic than NSG Flow Logs
    ⚠️ Can coexist with NSG Flow Logs (but not recommended
       to avoid double logging)
```

---

### 14.7 Configure Connection Monitor

> **Portal:** `Network Watcher → Connection monitor`

```
Configure Connection Monitor
│
├── Prerequisites
│   ├── Network Watcher enabled
│   ├── ⚠️ Network Watcher Agent VM extension on source VMs
│   ├── Log Analytics workspace
│   └── Source VM(s) running
│
├── Step 1: Create Connection Monitor
│   │   Portal: Network Watcher → Connection monitor → + Create
│   ├── Name
│   ├── Region
│   └── Log Analytics workspace
│
├── Step 2: Add Test Groups
│   │   Click "+ Add test group"
│   ├── Test group name
│   │
│   ├── Sources
│   │   ├── + Add sources
│   │   ├── Type: Azure VMs / Azure VNets / On-prem (with agent)
│   │   └── Select VMs / endpoints
│   │
│   ├── Destinations
│   │   ├── + Add destinations
│   │   ├── Type: Azure VM / External address / Azure endpoint
│   │   └── Enter IP / FQDN / URL + port
│   │
│   └── Test Configurations
│       ├── + Add test configuration
│       ├── Name
│       ├── Protocol: TCP / HTTP / ICMP
│       ├── Destination port (for TCP)
│       ├── Test frequency: Default 30 seconds
│       │   (options: 30s, 60s, 300s, 600s, 1800s)
│       └── Success thresholds:
│           ├── % checks failed → alert
│           └── Round-trip time threshold (ms)
│
├── Step 3: Alerts (Optional)
│   ├── Configure alert on test failure
│   └── Action group: Email / SMS / Webhook
│
└── Step 4: Create
    ⚠️ First 360 tests/month = free
    ⚠️ Continuous monitoring (not one-time)
    ⚠️ Replaces legacy Connection Monitor (Classic)
```

---

### 14.8 VPN Gateway Troubleshoot

> **Portal:** `Network Watcher → VPN troubleshoot`

```
VPN Gateway Troubleshoot
│
├── Prerequisites
│   ├── Network Watcher enabled in gateway's region
│   ├── VPN Gateway or connection exists
│   └── Storage account (for diagnostic logs)
│       ⚠️ Storage account is REQUIRED
│
├── Step 1: Select Resource
│   │   Portal: Network Watcher → VPN troubleshoot
│   ├── Resource type: VPN Gateway / Connection
│   └── Select the gateway or connection
│
├── Step 2: Storage Account
│   ├── Select storage account for log output
│   └── Container auto-created for results
│
├── Step 3: Start Troubleshoot
│   └── Click "Start troubleshooting"
│       (Takes several minutes)
│
└── Output
    ├── Status: Healthy / Unhealthy
    ├── Error code (if unhealthy)
    ├── Error message
    ├── Recommended action
    └── Detailed logs in storage account
        ⚠️ Works with S2S, P2S, and VNet-to-VNet connections
```

---

### 14.9 View Network Topology

> **Portal:** `Network Watcher → Topology`

```
View Network Topology
│
├── Step 1: Navigate
│   └── Portal: Network Watcher → Topology
│
├── Step 2: Select Scope
│   ├── Subscription
│   ├── Resource group (required)
│   │   ⚠️ Topology is scoped by resource group
│   └── Virtual network (optional filter)
│
├── Step 3: View Diagram
│   ├── Interactive visual map showing:
│   │   ├── VNets and subnets
│   │   ├── VMs and NICs
│   │   ├── NSGs
│   │   ├── Route tables
│   │   ├── Public IPs
│   │   ├── Load balancers
│   │   └── VPN/ExpressRoute gateways
│   └── Click any resource to see details
│
└── Step 4: Export
    └── Download as SVG
        ⚠️ Read-only visualization — no changes can be made here
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
