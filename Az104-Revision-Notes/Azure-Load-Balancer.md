<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Load Balancer — AZ-104 Revision Notes

---

## 1. What is Azure Load Balancer?

- **Layer 4 (TCP/UDP) load balancer** — distributes inbound traffic across backend pool instances
- Operates at the **Transport Layer** (OSI Layer 4) — does NOT inspect HTTP headers, URLs, or cookies
- Supports **inbound + outbound** scenarios
- Provides **high availability**, **scalability**, and **low latency**
- Works with VMs, VMSS (Virtual Machine Scale Sets), and IP addresses
- Fully managed, **no infrastructure to maintain**, zone-resilient (Standard SKU)

---

## 2. Key Components

| Component | Purpose |
|---|---|
| **Frontend IP Configuration** | Public or Private IP that receives incoming traffic |
| **Backend Pool** | Group of VMs / VMSS / IP addresses that serve requests |
| **Health Probes** | Checks backend instance health; unhealthy instances stop receiving traffic |
| **Load Balancing Rules** | Maps frontend IP:port to backend pool:port |
| **Inbound NAT Rules** | Forwards traffic from a specific frontend IP:port to a specific backend VM:port |
| **Outbound Rules** | Controls outbound SNAT (Source NAT) from backend pool to internet (Standard only) |
| **High Availability (HA) Ports** | Load balance ALL TCP/UDP ports simultaneously (Standard Internal LB only) |

---

## 3. SKU Comparison — Basic vs Standard

| Feature | **Basic** | **Standard** |
|---|---|---|
| **Backend pool size** | Up to **300** instances | Up to **1,000** instances |
| **Backend pool type** | VMs in a **single Availability Set** or **VMSS** | Any VMs in a **single VNet** (AS, VMSS, or standalone) |
| **Health probes** | TCP, HTTP | TCP, HTTP, **HTTPS** |
| **HA Ports** | ❌ Not supported | ✅ Supported (Internal LB) |
| **Availability Zones** | ❌ Not supported | ✅ Zone-redundant & zonal |
| **Outbound Rules** | ❌ Not supported | ✅ Declarative outbound NAT |
| **Multiple frontends** | ❌ Inbound only | ✅ Inbound and outbound |
| **Security** | **Open by default** | **Closed by default** (NSG required) |
| **SLA** | ❌ No SLA | ✅ **99.99% SLA** |
| **Diagnostics / Metrics** | ❌ Not supported | ✅ Azure Monitor multi-dimensional metrics |
| **Global / Cross-region** | ❌ | ✅ (Cross-region LB) |
| **Pricing** | **Free** | **Charged** (per rule + data processed) |
| **Upgrade** | Can upgrade Basic → Standard | — |

> ⚠️ **EXAM TIP:** **Basic = open by default** (no NSG needed), **Standard = closed by default** (must attach NSG to allow traffic). This is heavily tested!

> ⚠️ **EXAM TIP:** Basic LB backend pool can ONLY contain VMs from a **single Availability Set or VMSS**. Standard LB can mix VMs freely within a **single VNet**.

> ⚠️ **EXAM TIP:** Basic LB is being **retired on September 30, 2025**. Microsoft recommends Standard SKU for all production workloads.

---

## 4. Types — Public vs Internal

| Feature | **Public Load Balancer** | **Internal (Private) Load Balancer** |
|---|---|---|
| **Frontend IP** | Public IP address | Private IP from VNet subnet |
| **Traffic source** | Internet → backend VMs | Internal VNet / on-prem → backend VMs |
| **Use case** | Internet-facing apps (web servers) | Internal multi-tier apps (app tier, DB tier) |
| **Outbound connectivity** | Provides outbound SNAT for backend VMs | Does NOT provide outbound internet access |
| **Availability Zones** | ✅ (Standard) | ✅ (Standard) |
| **HA Ports** | ❌ | ✅ (Standard Internal only) |

> ⚠️ **EXAM TIP:** An **Internal Standard LB** with HA Ports can load balance **ALL protocols on ALL ports** simultaneously — used for NVAs (Network Virtual Appliances) like firewalls.

---

## 5. Cross-Region (Global) Load Balancer

- **Geo-redundant** HA across multiple Azure regions
- Standard SKU only
- Frontend = **Global tier Public IP**
- Backend pool = **Regional Standard LBs** (not VMs directly)
- Ultra-low latency — routes to **closest healthy regional LB**
- If one region fails → traffic routed to next closest healthy region
- Static anycast IP

| Feature | Regional LB | Cross-Region (Global) LB |
|---|---|---|
| **Scope** | Single region | Multiple regions |
| **Backend** | VMs / VMSS | Regional Standard LBs |
| **Frontend IP** | Regional Public/Private IP | Global tier Public IP |
| **Availability Zones** | Zone-redundant in region | Cross-region redundancy |
| **Protocol** | TCP / UDP | TCP / UDP |

> ⚠️ **EXAM TIP:** Cross-region LB backend = **other Standard LBs**, not VMs directly. Uses a **Global tier public IP**.

---

## 6. Frontend IP Configuration

- **Public LB:** Uses a Public IP address (Static or Dynamic)
  - Standard LB requires a **Standard SKU Public IP** (Basic LB → Basic IP)
  - Static IP recommended for production
- **Internal LB:** Uses a Private IP from a subnet
  - Can be Static (specified) or Dynamic (auto-assigned)
- **Multiple frontends** supported (Standard SKU) — each frontend can have its own rules
- Each frontend IP can have separate **load balancing rules** and **inbound NAT rules**

### Portal Path — Add Frontend IP
```
Load Balancer → Settings → Frontend IP configuration →
+ Add → Name, Type (Public/Private), IP address → Add
```

> ⚠️ **EXAM TIP:** **Standard LB requires Standard SKU Public IP**. Mixing SKUs (Basic IP + Standard LB) is NOT allowed.

---

## 7. Backend Pool

- Group of VMs, VMSS instances, or IP addresses that receive traffic
- **Standard LB:** Backend pool can contain VMs from anywhere in a single VNet (no AS restriction)
- **Basic LB:** Backend pool limited to a single Availability Set or single VMSS
- Can add VMs by **NIC** or by **IP address** (IP-based backend — Standard only)
- IP-based backend supports VMs in peered VNets or even on-premises (via VPN/ER)

### Portal Path — Configure Backend Pool
```
Load Balancer → Settings → Backend pools →
+ Add → Name, VNet, Backend Pool Configuration (NIC / IP Address) →
Add VMs → Save
```

> ⚠️ **EXAM TIP:** IP-based backend pool (Standard SKU) allows adding resources from **peered VNets** or **on-premises** — enables cross-network load balancing.

---

## 8. Health Probes

- Monitors backend instance health — **unhealthy VMs stop receiving new connections**
- Probe checks occur every **interval** seconds; instance marked down after **consecutive failures**

| Probe Type | Basic LB | Standard LB | Details |
|---|---|---|---|
| **TCP** | ✅ | ✅ | SYN handshake check on specified port |
| **HTTP** | ✅ | ✅ | HTTP GET to a path; expects **HTTP 200** for healthy |
| **HTTPS** | ❌ | ✅ | Same as HTTP but over TLS |

### Probe Settings

| Setting | Default | Range |
|---|---|---|
| **Port** | Required | 1–65535 |
| **Interval** | **5 seconds** | 5–2,000,000,000 seconds |
| **Unhealthy threshold** | **2** consecutive failures | 2–429,496,729 |

### How Probes Work
1. LB sends probe to each backend instance at configured interval
2. If instance returns healthy response → continues receiving traffic
3. If instance fails **N consecutive probes** (unhealthy threshold) → marked **Down**
4. Down instance stops receiving **new** connections (existing connections may continue)
5. When instance recovers → marked **Up** again → receives traffic

### Portal Path — Configure Health Probe
```
Load Balancer → Settings → Health probes →
+ Add → Name, Protocol (TCP/HTTP/HTTPS), Port,
Path (for HTTP/HTTPS), Interval, Unhealthy threshold → Add
```

> ⚠️ **EXAM TIP:** **HTTP/HTTPS probe** expects status code **200** only. Any other code = unhealthy. Always specify the **Path** (e.g., `/health`).

> ⚠️ **EXAM TIP:** If **all backend instances** fail health probes, the LB **stops distributing traffic entirely** (no healthy targets). It does NOT send to unhealthy ones.

---

## 9. Load Balancing Rules

- Maps **frontend IP:port** → **backend pool:port** using a health probe
- Defines the distribution algorithm and session persistence
- One rule per frontend IP + port + protocol combination

### Rule Settings

| Setting | Options |
|---|---|
| **Protocol** | TCP or UDP |
| **Frontend IP** | Select frontend |
| **Frontend Port** | 1–65535 (port clients connect to) |
| **Backend Port** | 1–65535 (port backend VMs listen on) |
| **Backend Pool** | Select backend pool |
| **Health Probe** | Select configured probe |
| **Session Persistence** | None / Client IP / Client IP and Protocol |
| **Idle Timeout** | 4–30 minutes (default: **4 minutes**) |
| **TCP Reset** | Enable/Disable (send TCP RST on idle timeout) |
| **Floating IP** | Enable/Disable (Direct Server Return) |
| **HA Ports** | Enable (Standard Internal only — ALL protocols + ALL ports) |

### Portal Path — Create Load Balancing Rule
```
Load Balancer → Settings → Load balancing rules →
+ Add → Name, Frontend IP, Protocol, Frontend Port, Backend Port,
Backend Pool, Health Probe, Session Persistence, Idle Timeout,
TCP Reset, Floating IP → Add
```

> ⚠️ **EXAM TIP:** Default **Idle Timeout = 4 minutes**. Max = 30 minutes. Enable **TCP Reset** to send RST on idle timeout (recommended for predictable connection behavior).

---

## 10. Session Persistence (Distribution Mode)

| Mode | Hash Based On | Behavior |
|---|---|---|
| **None** (default) | 5-tuple: Src IP, Src Port, Dest IP, Dest Port, Protocol | Requests from same client may go to different VMs |
| **Client IP** | 2-tuple: Src IP, Dest IP | All requests from same client IP → same VM |
| **Client IP and Protocol** | 3-tuple: Src IP, Dest IP, Protocol | Same IP + Protocol → same VM |

> ⚠️ **EXAM TIP:** Default = **None (5-tuple hash)**. Use **Client IP** affinity when application requires sticky sessions (e.g., legacy apps, license servers). Also called **Source IP affinity**.

---

## 11. Inbound NAT Rules

- Forwards traffic from a **specific frontend IP:port** to a **specific backend VM:port**
- Used for **direct access** to individual VMs (e.g., RDP port 50001 → VM1:3389, port 50002 → VM2:3389)
- **NOT** load balanced — maps 1-to-1

### Types
| Type | Description |
|---|---|
| **Single VM** | Frontend port → specific backend VM:port |
| **Backend pool-based** | Auto-creates NAT rules for all pool members using port ranges |

### Portal Path — Create Inbound NAT Rule
```
Load Balancer → Settings → Inbound NAT rules →
+ Add → Name, Type (single VM / pool-based), Frontend IP,
Frontend Port (or range), Target VM or Pool, Backend Port → Add
```

> ⚠️ **EXAM TIP:** Inbound NAT rules = **port forwarding** (1:1 mapping). Load balancing rules = **distribution across pool**. Don't confuse them.

---

## 12. Outbound Rules (Standard SKU Only)

- Explicitly configure **outbound SNAT** for backend pool to reach the internet
- Only available on **Standard Public LB**
- Controls: outbound IP, port allocation, idle timeout, TCP reset

### Key Concepts

| Concept | Details |
|---|---|
| **SNAT (Source NAT)** | Backend private IP translated to frontend public IP for outbound connections |
| **SNAT port allocation** | Each frontend IP provides **~64,000 SNAT ports** |
| **Port exhaustion** | When all ports used → outbound connections fail |
| **Allocated ports per instance** | Total ports / number of backend instances (default) |
| **Multiple frontend IPs** | Scale outbound by adding more public IPs (each adds ~64,000 ports) |
| **Idle Timeout** | 4–120 minutes (outbound), default **4 minutes** |

### Portal Path — Create Outbound Rule
```
Load Balancer → Settings → Outbound rules →
+ Add → Name, Frontend IP, Protocol (TCP/UDP/All),
Backend Pool, Port allocation (default/manual),
Allocated ports per instance, Idle timeout, TCP Reset → Add
```

> ⚠️ **EXAM TIP:** If backend VMs need outbound internet access and are behind a **Standard LB**, you MUST configure **outbound rules** or use a **NAT Gateway** or **instance-level Public IPs**. Standard LB does NOT provide default outbound access.

> ⚠️ **EXAM TIP:** **Default outbound access** for VMs is being retired. **NAT Gateway** is the recommended approach for outbound internet access.

---

## 13. HA Ports (High Availability Ports)

- Load balances **ALL TCP and UDP traffic on ALL ports** (1–65535) simultaneously
- Available ONLY on **Standard Internal Load Balancer**
- Single rule covers all protocols and ports — no need for per-port rules
- Use case: **Network Virtual Appliances (NVAs)** — firewalls, WAN optimizers, intrusion detection

### Portal Path — Enable HA Ports
```
Load Balancer (Internal, Standard) → Settings → Load balancing rules →
+ Add → Check "HA Ports" → Backend Pool, Health Probe → Add
```

> ⚠️ **EXAM TIP:** **HA Ports = Standard Internal LB only**. Not available on Public LB or Basic LB. When HA Ports enabled, frontend port and backend port are automatically set to **0** (meaning all).

---

## 14. Floating IP (Direct Server Return)

- When enabled: frontend IP is delivered **directly to the backend VM's NIC** (not DNAT'd)
- VM sees the **Load Balancer's frontend IP** as the destination, not its own IP
- Required for: **SQL Always On**, **HA configurations**, **multiple frontends to same backend port**
- Must configure **loopback interface** on the VM with the frontend IP for traffic to work

> ⚠️ **EXAM TIP:** **Floating IP = Direct Server Return (DSR)**. Required when you need multiple frontend IPs mapped to the same backend port. Also required for **SQL AlwaysOn Availability Groups**.

---

## 15. TCP Reset on Idle

- When enabled, Azure LB sends **TCP RST** packet when the idle timeout expires
- Applies to Load Balancing Rules, Inbound NAT Rules, and Outbound Rules
- Helps applications detect dead connections faster
- Available on **Standard SKU only**

> ⚠️ **EXAM TIP:** TCP Reset is **recommended** for all rules. Without it, idle connections just silently drop and the client/server may not detect the disconnection.

---

## 16. Availability Zones (Standard SKU Only)

| Type | Frontend IP Config | Behavior |
|---|---|---|
| **Zone-redundant** | Spans all zones | Survives any single zone failure; IP served by all zones |
| **Zonal** | Pinned to specific zone (e.g., Zone 1) | Frontend IP dedicated to a single zone |
| **No zone** | Not zone-aware | No zone guarantee |

- Backend pool VMs can be in **any zone** regardless of frontend zone config
- **Zone-redundant** = recommended for highest availability

### Portal Path — Configure Zone Redundancy
```
Create Load Balancer → Basics → Availability Zone →
Select: Zone-redundant / Zone 1 / Zone 2 / Zone 3 / No zone
```

> ⚠️ **EXAM TIP:** Zone selection is **set at creation time** and **cannot be changed** after deployment. Choose **Zone-redundant** for production.

---

## 17. Azure Load Balancer vs Application Gateway vs Traffic Manager vs Front Door

| Feature | **Load Balancer** | **Application Gateway** | **Traffic Manager** | **Front Door** |
|---|---|---|---|---|
| **OSI Layer** | Layer 4 (TCP/UDP) | Layer 7 (HTTP/HTTPS) | DNS-based (Layer 7) | Layer 7 (HTTP/HTTPS) |
| **Scope** | Regional | Regional | Global | Global |
| **Protocol** | TCP / UDP | HTTP, HTTPS, HTTP/2, WebSocket | Any (DNS-level) | HTTP / HTTPS |
| **SSL Offloading** | ❌ | ✅ | ❌ | ✅ |
| **URL-based routing** | ❌ | ✅ | ❌ | ✅ |
| **WAF** | ❌ | ✅ | ❌ | ✅ |
| **Cookie-based affinity** | ❌ | ✅ | ❌ | ✅ |
| **Health Probes** | TCP/HTTP/HTTPS | HTTP/HTTPS | HTTP/HTTPS/TCP | HTTP/HTTPS |
| **Session Persistence** | IP-based (2/3/5 tuple) | Cookie-based | ❌ | Cookie-based |
| **Cross-region** | ✅ (Cross-region LB) | ❌ | ✅ | ✅ |
| **Best For** | Non-HTTP workloads, ultra-low latency | Web apps needing L7 features | DNS routing across regions | Global HTTP apps with WAF |

> ⚠️ **EXAM TIP:** If the question says "Layer 4" or "TCP/UDP traffic" → **Load Balancer**. If it says "URL path-based routing" or "SSL offloading" → **Application Gateway**. If it says "DNS-based global routing" → **Traffic Manager**. If it says "global HTTP with WAF" → **Front Door**.

---

## 18. Security & RBAC

### NSG Requirement
- **Standard LB:** Backend VMs **MUST** have an NSG attached (LB is closed by default)
  - NSG must **allow** probe traffic and load balanced traffic
- **Basic LB:** No NSG required (open by default)

### Required NSG Rules for Standard LB

| Rule | Direction | Source | Destination | Port | Action |
|---|---|---|---|---|---|
| Allow LB probes | Inbound | `AzureLoadBalancer` | Backend Subnet | Probe port | Allow |
| Allow LB traffic | Inbound | Source range | Backend Subnet | Backend port | Allow |

> ⚠️ **EXAM TIP:** Source IP for health probes is always **168.63.129.16** (Azure's magic IP). Service tag `AzureLoadBalancer` covers this. If NSG blocks this IP, probes fail → **all VMs marked unhealthy**.

### RBAC Roles

| Role | Permissions |
|---|---|
| **Network Contributor** | Full management of LB and networking resources |
| **Contributor** | Full access to all resources including LB |
| **Reader** | View LB configuration (read-only) |
| **Owner** | Full access + role assignment |

### Key RBAC Permissions
| Action | Minimum Role |
|---|---|
| Create / Delete Load Balancer | Network Contributor |
| Add/remove backend pool members | Network Contributor |
| Create/modify rules | Network Contributor |
| View LB configuration | Reader |
| Assign roles on LB | Owner / User Access Administrator |

---

## 19. Monitoring & Diagnostics (Standard SKU Only)

### Azure Monitor Metrics (Multi-dimensional)

| Metric | Description |
|---|---|
| **Data Path Availability** | Percentage of time the LB data path is available (end-to-end) |
| **Health Probe Status** | Percentage of successful health probes per backend instance |
| **Byte Count** | Total bytes processed |
| **Packet Count** | Total packets processed |
| **SYN Count** | Number of TCP SYN packets (new connections) |
| **SNAT Connection Count** | Number of outbound SNAT connections |
| **Allocated SNAT Ports** | Number of SNAT ports allocated to each backend instance |
| **Used SNAT Ports** | Number of SNAT ports in use |

### Key Dimensions for Filtering
- Frontend IP address
- Frontend port
- Backend IP address
- Backend port
- Protocol
- Direction (Inbound / Outbound)

### Portal Path — View Metrics
```
Load Balancer → Monitoring → Metrics →
Select Metric (e.g., Data Path Availability) →
Add dimension filter → Apply
```

### Portal Path — Configure Alerts
```
Load Balancer → Monitoring → Alerts → + New alert rule →
Select Signal (e.g., Health Probe Status < 100%) →
Configure Action Group → Create
```

### Diagnostic Logs
- **Standard LB supports** Azure Monitor Diagnostic settings (can send to Log Analytics, Storage, Event Hub)
- Basic LB: **NO diagnostics available**

### Portal Path — Enable Diagnostics
```
Load Balancer → Monitoring → Diagnostic settings →
+ Add diagnostic setting → Select logs/metrics →
Send to: Log Analytics workspace / Storage Account / Event Hub → Save
```

> ⚠️ **EXAM TIP:** **Data Path Availability** = overall LB health. **Health Probe Status** = per-backend health. Both should be monitored. Basic LB has **ZERO** monitoring capabilities.

> ⚠️ **EXAM TIP:** If SNAT Connection Count shows failures → indicates **port exhaustion**. Solution: Add more frontend IPs or use NAT Gateway.

---

## 20. Pricing Key Points

| Component | Cost |
|---|---|
| **Basic LB** | **Free** |
| **Standard LB — Rules** | Charged per load balancing rule + NAT rule (first 5 rules rolled into one) |
| **Standard LB — Data Processed** | Charged per GB processed (inbound + outbound) |
| **Public IP** | Standard Static Public IP charges apply |
| **Cross-region LB** | Higher per-rule and per-GB rates than regional |

### Cost Optimization
- First **5 rules** are charged as **1 rule** (rules 6+ are individual)
- Use fewer frontend IPs where possible
- Consider **NAT Gateway** instead of outbound rules if outbound-only is the need
- Basic LB is free but lacks SLA, zones, HTTPS probes, diagnostics

> ⚠️ **EXAM TIP:** **Basic LB = Free**, **Standard LB = Paid** (per rule + data). First 5 rules counted as 1 for billing.

---

## 21. CLI / PowerShell Commands (Exam-Relevant)

### Azure CLI

| Action | Command |
|---|---|
| Create LB | `az network lb create -g <rg> -n <name> --sku Standard --frontend-ip-name <fe> --public-ip-address <pip>` |
| Create backend pool | `az network lb address-pool create -g <rg> --lb-name <lb> -n <pool>` |
| Create health probe | `az network lb probe create -g <rg> --lb-name <lb> -n <probe> --protocol Tcp --port 80` |
| Create LB rule | `az network lb rule create -g <rg> --lb-name <lb> -n <rule> --frontend-ip <fe> --backend-pool-name <pool> --probe-name <probe> --protocol Tcp --frontend-port 80 --backend-port 80` |
| Create inbound NAT rule | `az network lb inbound-nat-rule create -g <rg> --lb-name <lb> -n <rule> --frontend-ip <fe> --protocol Tcp --frontend-port 50001 --backend-port 3389` |
| Add VM NIC to pool | `az network nic ip-config address-pool add --address-pool <pool> --ip-config-name <ipconfig> --nic-name <nic> -g <rg> --lb-name <lb>` |
| List LB rules | `az network lb rule list -g <rg> --lb-name <lb>` |

### PowerShell

| Action | Command |
|---|---|
| Create LB | `New-AzLoadBalancer -ResourceGroupName <rg> -Name <name> -SKU Standard -Location <loc> -FrontendIpConfiguration <fe> -BackendAddressPool <pool>` |
| Get LB | `Get-AzLoadBalancer -ResourceGroupName <rg> -Name <name>` |
| Create probe | `Add-AzLoadBalancerProbeConfig -LoadBalancer <lb> -Name <probe> -Protocol Tcp -Port 80 -IntervalInSeconds 5 -ProbeCount 2` |
| Create rule | `Add-AzLoadBalancerRuleConfig -LoadBalancer <lb> -Name <rule> -FrontendIpConfiguration <fe> -BackendAddressPool <pool> -Probe <probe> -Protocol Tcp -FrontendPort 80 -BackendPort 80` |

> ⚠️ **EXAM TIP:** When creating Standard LB via CLI, always specify `--sku Standard`. Default SKU may vary. Backend VMs need NSG allowing traffic.

---

## 22. Limitations & Constraints

| Constraint | Limit |
|---|---|
| Max backend pool size (Standard) | **1,000** instances |
| Max backend pool size (Basic) | **300** instances |
| Max frontend IPs per LB | **600** (Public), **600** (Internal) |
| Max rules per LB | **1,500** (all LB rules + inbound NAT rules combined) |
| Max LBs per subscription per region | **1,000** |
| Max backend pools per LB | **1** (Basic), **multiple** (Standard) |
| Backend pool scope | Single VNet (Standard), single AS/VMSS (Basic) |
| SNAT ports per frontend IP | **~64,000** |
| Idle timeout range | **4–30 min** (inbound), **4–120 min** (outbound) |
| HA Ports | Standard Internal LB **only** |
| Cross-region backend | Regional Standard LBs only |
| UDP fragmentation | **Not supported** on Public LB |

> ⚠️ **EXAM TIP:** Backend pool limit: **1,000 (Standard)**, **300 (Basic)**. Max rules: **1,500** total. SNAT ports per IP: **~64,000**.

---

## 23. Quick-Fire Exam Points ⚡

1. Azure Load Balancer = **Layer 4 (TCP/UDP)** — does NOT understand HTTP, URLs, or cookies
2. **Basic SKU = Free, open by default**, no SLA, no zones, no HTTPS probes, no diagnostics
3. **Standard SKU = Paid, closed by default** (NSG required), 99.99% SLA, zone-redundant
4. Basic backend pool = **single Availability Set or single VMSS** only
5. Standard backend pool = **any VMs in a single VNet** (no AS restriction)
6. **Health probe source IP = 168.63.129.16** (Azure infrastructure IP) — must allow in NSG
7. Service tag `AzureLoadBalancer` in NSG rules allows health probe traffic
8. HTTP/HTTPS probes require **HTTP 200** response to mark healthy
9. Default session persistence = **None (5-tuple hash)** — use Client IP for sticky sessions
10. **Idle timeout default = 4 minutes**, max 30 minutes (inbound), 120 minutes (outbound)
11. **Floating IP = Direct Server Return** — needed for SQL AlwaysOn, multiple frontends same port
12. **HA Ports** = Standard Internal LB only — load balances ALL ports and protocols
13. **Outbound rules** = Standard Public LB only — controls SNAT for outbound internet
14. Standard LB **does NOT provide default outbound access** — must use NAT Gateway, outbound rules, or instance-level public IPs
15. **SNAT port exhaustion** → add more frontend public IPs or use NAT Gateway
16. Each public IP provides **~64,000 SNAT ports** (shared across backend pool)
17. **Zone-redundant** = survives zone failure; **Zonal** = pinned to one zone; set at creation, **cannot change later**
18. Front 5 LB rules billed as **1 rule** (rules 6+ billed individually)
19. **Cross-region LB** backend = other **Standard regional LBs** (not VMs)
20. Cross-region LB uses **Global tier public IP**
21. If all probes fail → **no traffic sent** (does NOT fall back to unhealthy VMs)
22. **Standard LB Public IP must be Standard SKU** — cannot mix Basic IP with Standard LB
23. L4 = Load Balancer, L7 = Application Gateway; URL routing / SSL offloading → use App GW, not LB
24. Backend pool max: **1,000 (Standard)**, **300 (Basic)**
25. TCP Reset on idle → sends RST when idle timeout expires (Standard only, recommended)
26. **Basic LB retiring September 30, 2025** — upgrade to Standard
27. Inbound NAT rules = **port forwarding** (1:1), LB rules = **distribution** (1:many)
28. IP-based backend pool (Standard) allows cross-VNet, cross-region, and on-prem backends
29. **Network Contributor** role = minimum to create and manage LBs
30. **Data Path Availability** metric = overall LB health; **Health Probe Status** = per-backend health

---

## 24. Step-by-Step Configuration Mind Maps 🗺️

---

### 24.1 Create Public Standard Load Balancer

> **Portal:** `Home → + Create a resource → Search "Load Balancer" → Create`

```
Create Public Standard Load Balancer
│
├── Step 1: Basics
│   ├── Subscription
│   ├── Resource Group
│   ├── Name
│   ├── Region
│   ├── SKU: Standard
│   │   ⚠️ Cannot change SKU after creation
│   ├── Type: Public
│   └── Tier: Regional (or Global for cross-region)
│
├── Step 2: Frontend IP Configuration
│   ├── + Add Frontend IP
│   ├── Name
│   ├── IP Version: IPv4 / IPv6
│   ├── Public IP Address: Create new or select existing
│   │   ├── SKU: Standard (must match LB SKU)
│   │   ├── Assignment: Static (recommended)
│   │   └── Availability Zone: Zone-redundant / Zonal / No zone
│   │       ⚠️ Zone selection is permanent — cannot change later
│   └── ⚠️ Standard LB requires Standard SKU Public IP
│
├── Step 3: Backend Pools
│   ├── + Add Backend Pool
│   ├── Name
│   ├── Virtual Network
│   ├── Backend Pool Configuration: NIC or IP Address
│   ├── Add VMs / IP Addresses
│   └── ⚠️ VMs must have NSG allowing probe & LB traffic
│
├── Step 4: Inbound Rules
│   ├── Load Balancing Rules
│   │   ├── + Add Rule
│   │   ├── Name, Frontend IP, Protocol, Ports
│   │   ├── Backend Pool, Health Probe
│   │   ├── Session Persistence, Idle Timeout
│   │   └── Floating IP, TCP Reset
│   │
│   └── Inbound NAT Rules
│       ├── + Add NAT Rule
│       ├── Frontend IP, Frontend Port
│       └── Target VM, Backend Port
│
├── Step 5: Outbound Rules (Optional)
│   ├── + Add Outbound Rule
│   ├── Frontend IP, Backend Pool
│   ├── Port Allocation, Idle Timeout
│   └── ⚠️ Required if backend VMs need outbound internet
│
├── Step 6: Tags (Optional)
│
└── Step 7: Review + Create → Create
    └── RBAC: Network Contributor or higher
```

---

### 24.2 Create Internal Standard Load Balancer

> **Portal:** `Home → + Create a resource → Search "Load Balancer" → Create`

```
Create Internal Standard Load Balancer
│
├── Step 1: Basics
│   ├── Subscription, Resource Group, Name, Region
│   ├── SKU: Standard
│   ├── Type: Internal
│   └── Tier: Regional
│
├── Step 2: Frontend IP Configuration
│   ├── + Add Frontend IP
│   ├── Name
│   ├── Virtual Network: Select VNet
│   ├── Subnet: Select subnet
│   ├── IP Assignment: Dynamic / Static
│   │   └── If Static → enter private IP address
│   └── Availability Zone: Zone-redundant / Zonal / No zone
│       ⚠️ Zone selection is permanent
│
├── Step 3: Backend Pools
│   ├── + Add Backend Pool
│   ├── VNet, Add VMs
│   └── ⚠️ VMs must be in same VNet as frontend
│
├── Step 4: Inbound Rules
│   ├── Load Balancing Rules
│   │   ├── + Add Rule
│   │   ├── ⚠️ Can enable HA Ports (Internal Standard only)
│   │   │   └── HA Ports → covers ALL protocols on ALL ports
│   │   └── Standard per-port rules also available
│   └── Inbound NAT Rules
│
├── Step 5: No Outbound Rules (Internal LB)
│   └── ⚠️ Internal LB does NOT provide outbound internet
│
├── Step 6: Tags (Optional)
│
└── Step 7: Review + Create → Create
```

---

### 24.3 Configure Health Probe

> **Portal:** `Load Balancer → Settings → Health probes`

```
Configure Health Probe
│
├── Step 1: Navigate
│   └── Load Balancer → Settings → Health probes
│
├── Step 2: + Add
│
├── Step 3: Configure
│   ├── Name
│   ├── Protocol
│   │   ├── TCP → SYN handshake (no path needed)
│   │   ├── HTTP → HTTP GET request (specify path)
│   │   └── HTTPS → HTTPS GET request (Standard only, specify path)
│   │       ⚠️ HTTPS probe only on Standard SKU
│   ├── Port: e.g., 80, 443, custom
│   ├── Path: e.g., /health (HTTP/HTTPS only)
│   │   ⚠️ Must return HTTP 200 to be considered healthy
│   ├── Interval: 5 seconds (default)
│   └── Unhealthy Threshold: 2 (default)
│
└── Step 4: Add → Probe created
    └── Associate with a Load Balancing Rule
```

---

### 24.4 Configure Load Balancing Rule

> **Portal:** `Load Balancer → Settings → Load balancing rules`

```
Configure Load Balancing Rule
│
├── Step 1: Navigate
│   └── Load Balancer → Settings → Load balancing rules
│
├── Step 2: + Add
│
├── Step 3: Configure Rule
│   ├── Name
│   ├── IP Version: IPv4 / IPv6
│   ├── Frontend IP Address: Select from configured frontends
│   ├── HA Ports: Yes/No
│   │   ⚠️ Only for Internal Standard LB
│   │   If Yes → Protocol = All, Frontend & Backend Port = 0 (all)
│   ├── Protocol: TCP / UDP (if HA Ports disabled)
│   ├── Frontend Port: e.g., 80
│   ├── Backend Port: e.g., 80
│   ├── Backend Pool: Select
│   ├── Health Probe: Select
│   │   ⚠️ Rule without probe = no health checking
│   ├── Session Persistence
│   │   ├── None (5-tuple hash — default)
│   │   ├── Client IP (2-tuple)
│   │   └── Client IP and Protocol (3-tuple)
│   ├── Idle Timeout: 4 min (default), range 4–30 min
│   ├── TCP Reset: Enabled (recommended) / Disabled
│   │   ⚠️ Standard SKU only
│   └── Floating IP: Disabled (default) / Enabled
│       ⚠️ Enable for SQL AlwaysOn or DSR scenarios
│
└── Step 4: Add → Rule created
```

---

### 24.5 Configure Inbound NAT Rule

> **Portal:** `Load Balancer → Settings → Inbound NAT rules`

```
Configure Inbound NAT Rule (Port Forwarding)
│
├── Step 1: Navigate
│   └── Load Balancer → Settings → Inbound NAT rules
│
├── Step 2: + Add
│
├── Step 3: Configure
│   ├── Name
│   ├── Type
│   │   ├── Azure virtual machine → target single VM
│   │   └── Backend pool → auto-map port ranges to all pool VMs
│   ├── Frontend IP Address: Select
│   ├── Frontend Port: e.g., 50001
│   │   └── For pool-based: Frontend Port Range Start
│   ├── Target Virtual Machine (if single VM type)
│   │   └── Or Backend Pool (if pool-based type)
│   ├── Backend Port: e.g., 3389 (RDP) or 22 (SSH)
│   ├── Protocol: TCP / UDP
│   ├── Enable Floating IP: Yes/No
│   └── Enable TCP Reset: Yes/No
│
└── Step 4: Add
    ⚠️ Example: Frontend 50001→VM1:3389, 50002→VM2:3389
    ⚠️ NAT rule = 1:1 mapping, NOT load balanced
```

---

### 24.6 Configure Outbound Rule (Standard Public LB)

> **Portal:** `Load Balancer → Settings → Outbound rules`

```
Configure Outbound Rule
│
├── Step 1: Navigate
│   └── Load Balancer → Settings → Outbound rules
│   ⚠️ Only available on Standard Public LB
│
├── Step 2: + Add
│
├── Step 3: Configure
│   ├── Name
│   ├── Frontend IP Address(es): Select one or more
│   │   └── Each IP adds ~64,000 SNAT ports
│   │       ⚠️ Add more IPs if hitting SNAT port exhaustion
│   ├── Protocol: TCP / UDP / All
│   ├── Backend Pool: Select
│   ├── Port Allocation
│   │   ├── Default: Automatically allocated based on pool size
│   │   └── Manual: Specify ports per instance
│   │       ├── Ports per instance: e.g., 10,000
│   │       └── ⚠️ Ensure enough ports for all instances
│   ├── Idle Timeout: 4 min (default), range 4–120 min
│   │   ⚠️ Outbound max 120 min (vs inbound max 30 min)
│   └── TCP Reset: Enable/Disable
│
└── Step 4: Add → Outbound rule created
    ⚠️ Without outbound rules, Standard LB gives NO default outbound
    ⚠️ Alternative: Use NAT Gateway (recommended by Microsoft)
```

---

### 24.7 Configure NSG for Standard Load Balancer

> **Portal:** `NSG → Settings → Inbound security rules`

```
Configure NSG for Standard LB Backend VMs
│
├── ⚠️ Standard LB = closed by default → NSG REQUIRED
│
├── Step 1: Navigate to Backend VM's NSG
│   └── NSG → Settings → Inbound security rules
│
├── Step 2: Allow Health Probe Traffic
│   ├── + Add
│   ├── Source: Service Tag → AzureLoadBalancer
│   ├── Source port: *
│   ├── Destination: Any (or backend subnet)
│   ├── Destination port: Probe port (e.g., 80)
│   ├── Protocol: TCP
│   ├── Action: Allow
│   ├── Priority: e.g., 100
│   └── Name: e.g., Allow-LB-Probes
│   ⚠️ Probe source IP = 168.63.129.16
│   ⚠️ If denied → all VMs marked unhealthy → no traffic
│
├── Step 3: Allow Application Traffic
│   ├── + Add
│   ├── Source: (Client IP range / Any / Internet)
│   ├── Destination port: Backend port (e.g., 80, 443)
│   ├── Protocol: TCP
│   ├── Action: Allow
│   └── Priority: e.g., 110
│
└── Step 4: Verify
    ├── No deny rules blocking probe or app traffic
    └── Default deny-all inbound still safe if explicit allows added
```

---

### 24.8 Upgrade Basic to Standard Load Balancer

> **Portal:** Automated via PowerShell script (no direct portal button)

```
Upgrade Basic → Standard LB
│
├── Prerequisites
│   ├── Basic LB must exist
│   ├── Backend VMs must NOT be in a Basic availability set
│   │   without Standard LB support
│   ├── Public IPs attached must be upgraded to Standard SKU first
│   │   ⚠️ Basic Public IP → Standard Public IP required
│   └── Backend pool VMs need NSG (Standard = closed by default)
│
├── Option 1: Use Microsoft Upgrade Script
│   ├── PowerShell: Start-AzBasicLoadBalancerUpgrade
│   │   -ResourceGroupName <rg> -BasicLoadBalancerName <name>
│   ├── Handles: SKU upgrade, backend migration, rule migration
│   └── ⚠️ Causes brief downtime during migration
│
├── Option 2: Manual Migration
│   ├── Step 1: Create new Standard LB
│   ├── Step 2: Recreate all rules, probes, pools
│   ├── Step 3: Move VMs to new backend pool
│   ├── Step 4: Add NSG rules for probes/traffic
│   ├── Step 5: Update DNS / Traffic Manager
│   └── Step 6: Delete old Basic LB
│
└── ⚠️ Cannot downgrade Standard → Basic
    ⚠️ Basic LB retiring September 30, 2025
```

---

### 24.9 Configure Cross-Region (Global) Load Balancer

> **Portal:** `Home → + Create a resource → Load Balancer → Tier: Global`

```
Create Cross-Region Load Balancer
│
├── Step 1: Basics
│   ├── Subscription, Resource Group, Name
│   ├── SKU: Standard (only option)
│   ├── Type: Public (only option)
│   └── Tier: Global
│       ⚠️ Global tier = cross-region
│
├── Step 2: Frontend IP
│   ├── Create Global Tier Public IP
│   │   └── SKU: Standard, Tier: Global
│   └── ⚠️ Global tier IP = static anycast IP
│
├── Step 3: Backend Pools
│   ├── + Add Backend Pool
│   ├── Backend pool type: Load Balancer
│   │   └── ⚠️ VMs cannot be added directly
│   ├── Select Regional Standard LBs as backends
│   │   ├── LB 1 (e.g., East US Standard LB)
│   │   └── LB 2 (e.g., West Europe Standard LB)
│   └── ⚠️ Regional LBs must be Standard Public LBs
│
├── Step 4: Load Balancing Rules
│   ├── Frontend IP, Protocol, Ports
│   └── Backend Pool, Health Probe
│
├── Step 5: Tags (Optional)
│
└── Step 6: Review + Create → Create
    ⚠️ If regional LB goes down → traffic routed to next closest
    ⚠️ Ultra-low latency routing to nearest healthy region
```

---

### 24.10 Monitor Load Balancer Health

> **Portal:** `Load Balancer → Monitoring`

```
Monitor Load Balancer
│
├── View Metrics
│   │   Portal: Load Balancer → Monitoring → Metrics
│   │
│   ├── Key Metrics to Monitor
│   │   ├── Data Path Availability (%) → overall LB health
│   │   │   ⚠️ If < 100% → LB data path issue
│   │   ├── Health Probe Status (%) → per-backend health
│   │   │   ⚠️ Filter by Backend IP to identify unhealthy VMs
│   │   ├── SNAT Connection Count → outbound connection tracking
│   │   │   ⚠️ Failed SNAT = port exhaustion
│   │   ├── Used SNAT Ports & Allocated SNAT Ports
│   │   ├── Byte Count → throughput
│   │   └── SYN Count → new connection rate
│   │
│   └── Add Filters / Splitting
│       ├── Split by: Frontend IP, Backend IP
│       └── Time range: Last hour / 24 hrs / 7 days
│
├── Configure Alerts
│   │   Portal: Load Balancer → Monitoring → Alerts
│   ├── + New Alert Rule
│   ├── Signal: Health Probe Status / Data Path Availability
│   ├── Condition: e.g., Health Probe Status < 100%
│   ├── Action Group: Email / SMS / Webhook / Logic App
│   └── Create
│
├── Diagnostic Settings (Standard only)
│   │   Portal: Load Balancer → Monitoring → Diagnostic settings
│   ├── + Add diagnostic setting
│   ├── Select: AllMetrics
│   ├── Destination: Log Analytics / Storage / Event Hub
│   └── Save
│   ⚠️ Basic LB has NO diagnostics
│
└── Resource Health
    │   Portal: Load Balancer → Help → Resource health
    ├── Checks: Available / Degraded / Unavailable
    └── Shows historical health events
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
