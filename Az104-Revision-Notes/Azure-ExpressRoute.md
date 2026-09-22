<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure ExpressRoute — AZ-104 Revision Notes

---

## 1. What is Azure ExpressRoute?

- **Private, dedicated connection** between on-premises and Azure — does **NOT** go over the public internet
- Provided by a **connectivity provider** (e.g., Equinix, AT&T, Megaport)
- Higher **reliability, faster speeds, lower latency, more security** than internet-based VPN
- Supports **BGP** for dynamic routing between on-prem and Azure
- Bandwidth: **50 Mbps to 100 Gbps**
- **Not encrypted by default** — data traverses a private connection but is NOT IPsec encrypted

---

## 2. Key Components

| Component | Purpose |
|---|---|
| **ExpressRoute Circuit** | Logical connection between on-prem and Azure via connectivity provider |
| **Connectivity Provider** | ISP/Telco that provides the physical connection (e.g., Equinix, AT&T) |
| **Peering** | BGP sessions — Private Peering (VNets) and Microsoft Peering (M365/PaaS) |
| **ExpressRoute Gateway** | VNet gateway (type: ExpressRoute) in GatewaySubnet to connect VNet to circuit |
| **GatewaySubnet** | Dedicated subnet (MUST be named `GatewaySubnet`) for the gateway |
| **Route Filter** | Controls which Microsoft services are accessible via Microsoft Peering |
| **ExpressRoute Global Reach** | Connects on-prem sites to each other through ExpressRoute (bypasses internet) |
| **ExpressRoute Direct** | Direct physical connection to Microsoft edge (10 Gbps / 100 Gbps ports) |

---

## 3. Connectivity Models

| Model | Description |
|---|---|
| **Cloud Exchange Co-location** | On-prem located at a co-location facility; connect via provider's L2/L3 exchange |
| **Point-to-Point Ethernet** | Dedicated point-to-point link from on-prem to Azure |
| **Any-to-Any (IPVPN)** | Integrate Azure into existing WAN/MPLS (WAN provider connects to Azure) |
| **ExpressRoute Direct** | Direct physical port connection to Microsoft edge (10G/100G) |

> ⚠️ **EXAM TIP:** Know the 4 connectivity models at a high level. **ExpressRoute Direct** = highest bandwidth (up to 100 Gbps) with direct physical ports.

---

## 4. Peering Types

| Peering | Connects To | Use Case | Address Space |
|---|---|---|---|
| **Azure Private Peering** | Azure VNets (VMs, ILBs, private IPs) | Access resources with private IPs in VNets | Customer-owned private IP ranges (/29 or /30 subnets for BGP) |
| **Microsoft Peering** | Microsoft 365, Dynamics 365, Azure PaaS (Storage, SQL, etc.) | Access Microsoft public services over private connection | Microsoft public IP prefixes (via route filter) |

### Deprecated Peering
- **Azure Public Peering** — **deprecated**, replaced by **Microsoft Peering**

> ⚠️ **EXAM TIP:** **Private Peering** = VNets (private IPs). **Microsoft Peering** = M365 + Azure PaaS (public endpoints over private connection). **Public Peering is DEPRECATED** — if it appears as an option, it's a distractor.

> ⚠️ **EXAM TIP:** Microsoft Peering requires **Route Filters** to select which Microsoft services to advertise. Without route filter = no routes.

---

## 5. ExpressRoute SKUs

| SKU | Scope | Features |
|---|---|---|
| **Local** | Access to 1–2 Azure regions near the peering location | **Free egress** (no data transfer charges), limited to local regions |
| **Standard** | Access to all regions within the **same geopolitical region** | Up to 10 VNet links per circuit |
| **Premium** | Access to all regions **globally** (cross geopolitical) | Up to 100 VNet links, Global Reach, M365 route support |

### Geopolitical Regions

| Geopolitical Region | Example Azure Regions |
|---|---|
| North America | East US, West US, Canada Central |
| Europe | West Europe, North Europe, UK South |
| Asia Pacific | Southeast Asia, East Asia, Australia |
| Others | Brazil, Japan, India, South Africa, UAE, Korea |

> ⚠️ **EXAM TIP:** **Local SKU = free egress**, but limited to nearby regions. **Standard = same geopolitical region**. **Premium = cross geopolitical region + Global Reach**. If connecting US to Europe → **Premium required**.

> ⚠️ **EXAM TIP:** **Standard = max 10 VNet links**. **Premium = max 100 VNet links**. This limit is heavily tested.

---

## 6. ExpressRoute Circuit Bandwidth

| Bandwidth Options |
|---|
| 50 Mbps, 100 Mbps, 200 Mbps, 500 Mbps |
| 1 Gbps, 2 Gbps, 5 Gbps, 10 Gbps |
| **ExpressRoute Direct only:** 40 Gbps, 100 Gbps |

- Can **upgrade bandwidth** without downtime (if provider supports)
- **Cannot downgrade** without recreating the circuit
- Bandwidth is **per circuit** (shared across all peerings)

> ⚠️ **EXAM TIP:** Bandwidth can be **increased** (no downtime) but **NOT decreased** without deleting and recreating the circuit.

---

## 7. ExpressRoute Gateway SKUs

| Gateway SKU | Max Circuits | Max Connections | Throughput | FastPath |
|---|---|---|---|---|
| **Standard (ErGw1Az)** | 4 | 500 | ~1 Gbps | ❌ |
| **High Performance (ErGw2Az)** | 8 | 2,500 | ~2 Gbps | ❌ |
| **Ultra Performance (ErGw3Az)** | 16 | 10,000 | ~10 Gbps | ✅ |
| **ErGwScale (Preview)** | 16 | 10,000 | Up to 40 Gbps | ✅ |

- Gateway deployed in **GatewaySubnet** (same subnet as VPN Gateway if coexisting)
- **AZ** suffix = zone-redundant (recommended for production)

### Portal Path — Create ExpressRoute Gateway
```
Home → + Create a resource → Virtual Network Gateway →
Gateway type: ExpressRoute →
SKU: Standard / High Performance / Ultra Performance →
Virtual Network: Select VNet → GatewaySubnet → Public IP → Create
```

> ⚠️ **EXAM TIP:** **Ultra Performance** gateway required for **FastPath** (bypasses gateway for data plane — lower latency). Standard/High Perf = no FastPath.

> ⚠️ **EXAM TIP:** ExpressRoute Gateway is type **ExpressRoute** (not VPN). A VNet can have **one VPN Gateway + one ExpressRoute Gateway** (coexistence).

---

## 8. ExpressRoute Circuit States

| State | Meaning |
|---|---|
| **Not Provisioned** | Circuit created in Azure but provider hasn't provisioned yet |
| **Provisioning** | Provider is setting up the connection |
| **Provisioned** | Provider has completed provisioning; ready to configure peerings |
| **Deprovisioning** | Provider is tearing down |
| **Enabled/Disabled** | Admin state — can disable without deleting |

### Service Provider Status vs Circuit Status
- **Provider status** = shows provider provisioning state
- **Circuit status** = shows if circuit is enabled/disabled by admin

> ⚠️ **EXAM TIP:** You can only configure peerings AFTER circuit status = **Provisioned**. Create circuit in Azure → send Service Key to provider → provider provisions → status changes to Provisioned.

---

## 9. Connecting VNet to ExpressRoute Circuit

- Create **Connection** between ExpressRoute Gateway and Circuit
- One circuit can connect to **multiple VNets** (up to Standard=10, Premium=100)
- VNets can be in **different subscriptions** (requires authorization)
- VNets can be in **different regions** (Premium SKU required for cross-geopolitical)

### Portal Path — Connect VNet to Circuit
```
ExpressRoute Circuit → Settings → Connections →
+ Add → Name, Connection type: ExpressRoute,
Virtual network gateway: Select ExpressRoute GW →
Authorization key (if cross-subscription) → OK
```

### Portal Path — Authorization (Cross-subscription)
```
ExpressRoute Circuit → Settings → Authorizations →
+ Add → Name → Save → Copy Authorization Key & Circuit Resource ID
```

> ⚠️ **EXAM TIP:** Cross-subscription VNet connection requires **Authorization Key** + **Circuit Resource ID**. The circuit owner creates the authorization; the VNet owner redeems it.

---

## 10. ExpressRoute Global Reach

- Connects **two on-prem sites** to each other **through Microsoft's backbone** via two ExpressRoute circuits
- Traffic goes: On-prem Site A → ER Circuit 1 → Microsoft backbone → ER Circuit 2 → On-prem Site B
- Requires **Premium SKU** on at least one circuit
- Not available in all regions

### Portal Path
```
ExpressRoute Circuit → Settings → Global Reach →
+ Add → Select peer circuit →
Enter /29 subnet for peering → Save
```

> ⚠️ **EXAM TIP:** Global Reach = **on-prem to on-prem** via ExpressRoute (not VPN, not internet). Requires **Premium SKU**.

---

## 11. ExpressRoute Direct

- **Direct physical connection** to Microsoft edge (bypass connectivity provider)
- Port speeds: **10 Gbps or 100 Gbps**
- Enables **MACsec encryption** (Layer 2 encryption)
- Can create **multiple circuits** on a single Direct port
- Use case: massive data ingestion, regulated industries, direct control

> ⚠️ **EXAM TIP:** **ExpressRoute Direct** = only way to get **100 Gbps** and **MACsec encryption**. Regular ExpressRoute max = 10 Gbps (via provider).

---

## 12. ExpressRoute FastPath

- Bypasses the ExpressRoute Gateway for **data plane traffic** (sends directly to VM)
- Reduces latency — gateway only used for **control plane** (route exchange)
- Requires **Ultra Performance** or **ErGwScale** gateway SKU
- Supported on **Private Peering** only
- Does NOT support: VNet Peering (UDR), Basic ILB, Private Link

### Portal Path
```
ExpressRoute Connection → Settings → Configuration →
FastPath: Enabled → Save
```

> ⚠️ **EXAM TIP:** FastPath = **Ultra Performance gateway** only. Removes gateway from data path. Does NOT work with some features (VNet peering gateway transit, Basic ILB).

---

## 13. Redundancy & Resiliency

### Built-in Redundancy
- Every ExpressRoute circuit has **TWO connections** (primary + secondary) to two Microsoft edge routers
- Each connection = separate BGP session
- This provides **active-active** redundancy at the physical layer

### Resiliency Levels

| Level | Setup |
|---|---|
| **Standard** | Single circuit with dual connections (built-in) |
| **High** | Two circuits at same peering location |
| **Maximum** | Two circuits at different peering locations (geo-redundant) |

> ⚠️ **EXAM TIP:** Every circuit has **2 physical connections** (primary + secondary). For **maximum resiliency**, use **two circuits at two different peering locations** + connect both to same VNet.

---

## 14. ExpressRoute + VPN Coexistence

- Same VNet can have **both ExpressRoute Gateway + VPN Gateway**
- VPN Gateway can act as **failover** for ExpressRoute (S2S VPN as backup)
- Both gateways share the **GatewaySubnet** (but are separate resources)
- Use case: HA design — ExpressRoute = primary, S2S VPN = backup

### Requirements
- Route-based VPN Gateway (not policy-based)
- GatewaySubnet large enough for both (recommend **/27 or larger**)
- Create VPN Gateway **after** ExpressRoute Gateway

> ⚠️ **EXAM TIP:** ExpressRoute + VPN can **coexist** in the same VNet. VPN as **backup** for ExpressRoute is a common exam scenario. Both use the **same GatewaySubnet**.

---

## 15. Encryption Options

| Method | Layer | Scope | Setup |
|---|---|---|---|
| **MACsec** | Layer 2 | Port-to-Microsoft edge | ExpressRoute Direct only |
| **IPsec VPN over ER** | Layer 3 | End-to-end (on-prem to Azure) | S2S VPN tunnel through ER Private Peering |

- By default, ExpressRoute = **NOT encrypted** (private but unencrypted)
- For encryption: Use **S2S VPN over ExpressRoute** or **MACsec (Direct only)**

> ⚠️ **EXAM TIP:** ExpressRoute is **private but NOT encrypted by default**. If exam asks for encrypted private connection → VPN over ExpressRoute or MACsec (Direct).

---

## 16. Route Filters (Microsoft Peering)

- Control which **Microsoft service BGP communities** are advertised to on-prem
- Required for **Microsoft Peering** — without it, no routes are advertised
- Can filter by service: Exchange Online, SharePoint Online, Azure Storage by region, etc.

### Portal Path
```
Home → + Create a resource → Route Filter → Create →
Name, Subscription, RG, Region →
Manage rules → Select BGP communities (e.g., Exchange Online, Azure Storage) →
Associate with ExpressRoute circuit → Create
```

> ⚠️ **EXAM TIP:** Route filter = **whitelist** of Microsoft services for Microsoft Peering. No route filter = no routes = no connectivity via Microsoft Peering.

---

## 17. Security & RBAC

### RBAC Roles

| Role | Permissions |
|---|---|
| **Network Contributor** | Full management of ER circuit, gateway, connections |
| **ExpressRoute Circuit Owner** | Creates authorizations for cross-subscription access |
| **Contributor** | Full access to all resources |
| **Reader** | View-only |

| Action | Minimum Role |
|---|---|
| Create ExpressRoute Circuit | Network Contributor |
| Configure peerings | Network Contributor |
| Create authorization (cross-sub) | Network Contributor (on circuit) |
| Create ExpressRoute Gateway | Network Contributor |
| Connect VNet to circuit | Network Contributor (on gateway + authorization key) |

---

## 18. Monitoring & Diagnostics

### Key Metrics

| Metric | Description |
|---|---|
| **Bits In/Out Per Second** | Throughput on circuit |
| **BGP Availability** | BGP session health (%) |
| **ARP Availability** | ARP table reachability (%) |
| **Dropped Packets In/Out** | Packets dropped at the circuit |
| **QoS — Bits In/Out** | Per-class traffic stats |
| **Gateway Bits In/Out** | Throughput at the ER gateway |
| **Gateway CPU** | Gateway instance CPU utilization |

### Portal Path — View Metrics
```
ExpressRoute Circuit → Monitoring → Metrics →
Select: BitsInPerSecond, BGP Availability, etc. → Apply
```

### Portal Path — Diagnostic Logs
```
ExpressRoute Circuit → Monitoring → Diagnostic settings →
+ Add → Select logs → Destination: Log Analytics → Save
```

### Portal Path — Configure Alerts
```
ExpressRoute Circuit → Monitoring → Alerts →
+ New alert rule → Signal: BGP Availability < 100% →
Action Group → Create
```

> ⚠️ **EXAM TIP:** **BGP Availability** and **ARP Availability** = key circuit health indicators. Both should be ~100%. If BGP drops → route exchange fails → connectivity breaks.

---

## 19. Pricing Key Points

| Component | Cost |
|---|---|
| **Circuit (Metered)** | Monthly port fee + outbound data transfer per GB |
| **Circuit (Unlimited)** | Monthly port fee (higher) + unlimited data transfer |
| **Local SKU** | Port fee only — **FREE data egress** (no outbound data charges) |
| **Standard vs Premium** | Premium = additional monthly add-on charge |
| **ExpressRoute Gateway** | Per-hour charge (like VPN Gateway) |
| **Global Reach** | Per-GB charge for data transferred between circuits |
| **ExpressRoute Direct** | Port fee (10G or 100G) + circuit fees |
| **Inbound data** | **Always free** (no ingress charges) |

### Billing Plans

| Plan | Port Fee | Outbound Data |
|---|---|---|
| **Metered** | Lower monthly fee | Charged per GB |
| **Unlimited** | Higher monthly fee | No per-GB charge |

> ⚠️ **EXAM TIP:** **Local SKU = free data egress** (biggest cost saver). **Metered = pay per GB outbound**. **Unlimited = flat rate**. Inbound data = always free for all SKUs.

> ⚠️ **EXAM TIP:** ExpressRoute Gateway runs **24/7 and is charged hourly** (same as VPN Gateway). Cannot be stopped/deallocated.

---

## 20. Limitations & Constraints

| Constraint | Limit |
|---|---|
| VNet links per circuit (Standard) | **10** |
| VNet links per circuit (Premium) | **100** |
| Circuits per ExpressRoute Direct port | **10** (10 Gbps), **100** (100 Gbps) |
| Max bandwidth (provider) | **10 Gbps** |
| Max bandwidth (Direct) | **100 Gbps** |
| BGP routes per Private Peering | **4,000** (Standard), **10,000** (Premium) |
| BGP routes per Microsoft Peering | **200** |
| Max circuits per gateway (Standard) | **4** |
| Max circuits per gateway (Ultra) | **16** |
| GatewaySubnet name | Must be **`GatewaySubnet`** |
| Peerings per circuit | Private + Microsoft (max 2) |

---

## 21. Quick-Fire Exam Points ⚡

1. ExpressRoute = **private dedicated connection** to Azure — **NOT over the internet**
2. **NOT encrypted by default** — private but unencrypted. For encryption: VPN over ER or MACsec
3. **Two peerings**: Private Peering (VNets) + Microsoft Peering (M365/PaaS). **Public Peering = DEPRECATED**
4. **Local SKU** = free data egress, limited to nearby regions
5. **Standard SKU** = same geopolitical region, max **10 VNet links**
6. **Premium SKU** = cross geopolitical region, max **100 VNet links**, Global Reach
7. Bandwidth can be **upgraded** without downtime, but **cannot be downgraded**
8. Every circuit has **2 connections** (primary + secondary) = built-in redundancy
9. **GatewaySubnet** = MUST be named `GatewaySubnet`, recommend /27+
10. ExpressRoute Gateway types: **Standard, High Performance, Ultra Performance**
11. **FastPath** = bypasses gateway for data path, requires **Ultra Performance** gateway
12. **Global Reach** = on-prem-to-on-prem through Microsoft backbone, requires **Premium**
13. **ExpressRoute Direct** = direct physical 10G/100G ports, enables **MACsec** encryption
14. ER + VPN can **coexist** in same VNet. VPN as **backup** for ER = common pattern
15. **Microsoft Peering requires Route Filters** — without it, no routes advertised
16. Circuit states: Not Provisioned → Provisioning → **Provisioned** (then configure peerings)
17. Cross-subscription VNet connection needs **Authorization Key** + **Circuit Resource ID**
18. **BGP Availability** and **ARP Availability** = key health metrics (~100% = healthy)
19. Metered = pay per GB outbound. Unlimited = flat rate. **Inbound = always free**
20. Gateway is **charged per hour** even when idle — cannot stop/deallocate
21. **Network Contributor** = minimum RBAC role for all ER operations
22. 4 connectivity models: Co-location, Point-to-Point, Any-to-Any (IPVPN), Direct
23. ExpressRoute vs VPN: ER = private/faster/not encrypted. VPN = internet/encrypted/cheaper
24. For **maximum resiliency**: 2 circuits at 2 different peering locations
25. Max **4,000 routes** per Private Peering (Standard), **10,000** (Premium)
26. Create circuit → send **Service Key** to provider → provider provisions → configure peerings
27. Max gateway circuits: Standard=4, High Perf=8, Ultra=16
28. ExpressRoute supports **IPv4 and IPv6** dual-stack on Private Peering
29. VNet Peering gateway transit works with ExpressRoute Gateway (share ER with peered VNets)
30. ExpressRoute is a **regional service** — circuit is at a specific peering location

---

## 22. Step-by-Step Configuration Mind Maps 🗺️

---

### 22.1 Create ExpressRoute Circuit

> **Portal:** `Home → + Create a resource → ExpressRoute`

```
Create ExpressRoute Circuit
│
├── Step 1: Basics
│   ├── Subscription, Resource Group
│   ├── Region (Azure region for metadata)
│   ├── Circuit Name
│   ├── Port type: Provider / Direct
│   │   ├── Provider → select connectivity provider
│   │   └── Direct → select ExpressRoute Direct resource
│   ├── Provider: Select from list (e.g., Equinix, AT&T)
│   ├── Peering location: Select (e.g., Washington DC, Amsterdam)
│   ├── Bandwidth: 50 Mbps – 10 Gbps (provider) / up to 100 Gbps (Direct)
│   │   ⚠️ Can increase later but CANNOT decrease
│   └── SKU: Local / Standard / Premium
│       ├── Billing model: Metered / Unlimited
│       │   ⚠️ Local SKU = free egress
│       │   ⚠️ Standard = same geopolitical region
│       │   ⚠️ Premium = global, more VNet links
│       └── ⚠️ Standard = 10 VNet links, Premium = 100
│
├── Step 2: Tags (Optional)
│
├── Step 3: Review + Create → Create
│   ├── RBAC: Network Contributor
│   └── Status: "Not Provisioned"
│
├── Step 4: Get Service Key
│   │   Portal: ExpressRoute Circuit → Overview → Service Key
│   ├── Send Service Key to connectivity provider
│   └── Provider provisions the circuit
│       ⚠️ Wait for Provider Status → "Provisioned"
│
└── Step 5: Configure Peerings (after Provisioned)
    └── See mind map 22.2
```

---

### 22.2 Configure ExpressRoute Peerings

> **Portal:** `ExpressRoute Circuit → Settings → Peerings`

```
Configure ExpressRoute Peerings
│
├── Private Peering (VNet Access)
│   │   Portal: ExpressRoute Circuit → Peerings → + Add →
│   │           Peering type: Azure private
│   │
│   ├── Peer ASN: On-prem BGP ASN (e.g., 65000)
│   │   ⚠️ Must be different from Azure's ASN
│   ├── Primary subnet: /30 (e.g., 10.0.0.0/30)
│   │   ├── First usable IP → Microsoft router
│   │   └── Second usable IP → On-prem router
│   ├── Secondary subnet: /30 (e.g., 10.0.0.4/30)
│   │   └── Same IP assignment pattern
│   ├── VLAN ID: Enter (must match provider config)
│   ├── Shared Key (optional): MD5 hash for BGP auth
│   └── Save
│   ⚠️ /30 subnets required (4 IPs, 2 usable per link)
│   ⚠️ Primary + Secondary for redundancy
│
├── Microsoft Peering (M365 / Azure PaaS)
│   │   Portal: ExpressRoute Circuit → Peerings → + Add →
│   │           Peering type: Microsoft
│   │
│   ├── Peer ASN
│   ├── Primary subnet: /30 (must be PUBLIC IPs you own)
│   │   ⚠️ Microsoft Peering = PUBLIC IP ranges required
│   ├── Secondary subnet: /30 (PUBLIC IPs)
│   ├── VLAN ID
│   ├── Advertised public prefixes: Your public IP ranges
│   │   ⚠️ Must be registered/owned by you
│   │   ⚠️ Validated by Microsoft (may take time)
│   ├── Customer ASN (if different from Peer ASN)
│   ├── Routing Registry: ARIN, RIPE, APNIC, etc.
│   └── Save
│   ⚠️ Requires Route Filter to receive Microsoft routes (see 22.3)
│
└── ⚠️ Public Peering = DEPRECATED (do NOT configure)
```

---

### 22.3 Create Route Filter (Microsoft Peering)

> **Portal:** `Home → + Create a resource → Route Filter`

```
Create Route Filter
│
├── Step 1: Create Route Filter Resource
│   ├── Subscription, Resource Group
│   ├── Name
│   ├── Region (must match circuit)
│   └── Create
│
├── Step 2: Add Filter Rules
│   │   Portal: Route Filter → Settings → Manage rules
│   ├── + Add Rule
│   ├── Name
│   ├── Service communities: Select
│   │   ├── Azure Storage (by region)
│   │   ├── Exchange Online
│   │   ├── SharePoint Online
│   │   ├── Dynamics 365
│   │   └── Other Microsoft services
│   ├── Access: Allow
│   └── Save
│   ⚠️ Without rules → no Microsoft routes advertised to on-prem
│
└── Step 3: Associate with ExpressRoute Circuit
    │   Portal: Route Filter → Settings → Circuit associations
    ├── + Add → Select ExpressRoute Circuit
    └── Save
    ⚠️ Must be associated to circuit's Microsoft Peering
```

---

### 22.4 Connect VNet to ExpressRoute Circuit

> **Portal:** `ExpressRoute Circuit → Connections`

```
Connect VNet to ExpressRoute Circuit
│
├── Prerequisites
│   ├── ExpressRoute Circuit in "Provisioned" state
│   ├── Private Peering configured on circuit
│   ├── ExpressRoute Gateway deployed in VNet's GatewaySubnet
│   └── If cross-subscription: Authorization key required
│
├── Same Subscription
│   │   Portal: ExpressRoute Gateway → Settings → Connections → + Add
│   ├── Name
│   ├── Connection type: ExpressRoute
│   ├── ExpressRoute circuit: Select from dropdown
│   ├── Enable FastPath: Yes/No
│   │   ⚠️ Only with Ultra Performance gateway
│   └── OK → Connection created
│
├── Cross-Subscription
│   │
│   ├── Circuit Owner (Subscription A):
│   │   │   Portal: ExpressRoute Circuit → Authorizations → + Add
│   │   ├── Name → Save
│   │   ├── Copy Authorization Key
│   │   └── Copy Circuit Resource ID
│   │   ⚠️ Each authorization = one VNet connection
│   │
│   └── VNet Owner (Subscription B):
│       │   Portal: ExpressRoute Gateway → Connections → + Add
│       ├── Connection type: ExpressRoute
│       ├── Redeem authorization: Yes
│       ├── Authorization key: paste
│       ├── Peer circuit URI: paste Resource ID
│       └── OK → Connection created
│
└── Verify
    ├── Connection status: "Connected"
    ├── Check effective routes on VNet VMs
    └── ⚠️ Max VNet links: Standard=10, Premium=100
```

---

### 22.5 Configure ExpressRoute + VPN Coexistence

> **Portal:** `Virtual Network → GatewaySubnet + Two Gateways`

```
ExpressRoute + VPN Coexistence
│
├── Prerequisites
│   ├── VNet with GatewaySubnet (/27 or larger)
│   │   ⚠️ Must be large enough for BOTH gateways
│   ├── ExpressRoute circuit provisioned
│   └── On-prem VPN device for S2S VPN
│
├── Step 1: Create ExpressRoute Gateway (FIRST)
│   ├── Gateway type: ExpressRoute
│   ├── SKU: Standard / High Perf / Ultra
│   ├── VNet → GatewaySubnet
│   └── Public IP → Create
│
├── Step 2: Connect ER Gateway to Circuit
│   └── Connection type: ExpressRoute → Select circuit
│
├── Step 3: Create VPN Gateway (SECOND)
│   ├── Gateway type: VPN
│   ├── VPN type: Route-based
│   │   ⚠️ Must be Route-based (not Policy-based)
│   ├── SKU: VpnGw1 or higher
│   ├── VNet → same GatewaySubnet
│   └── Public IP → Create (different from ER gateway)
│
├── Step 4: Create S2S VPN Connection
│   ├── Local Network Gateway → on-prem device
│   ├── Shared key
│   └── Connection type: Site-to-site
│
└── Result
    ├── ExpressRoute = primary path (preferred by BGP)
    ├── VPN = backup (failover if ER goes down)
    └── ⚠️ Create ER gateway FIRST, then VPN gateway
```

---

### 22.6 Configure Global Reach

> **Portal:** `ExpressRoute Circuit → Settings → Global Reach`

```
Configure Global Reach
│
├── Prerequisites
│   ├── Two ExpressRoute circuits (different peering locations)
│   ├── At least one circuit with Premium SKU
│   ├── Private peering configured on both circuits
│   └── Supported regions (not available everywhere)
│
├── Step 1: Navigate to Circuit 1
│   └── ExpressRoute Circuit 1 → Settings → Global Reach
│
├── Step 2: + Add
│   ├── Peer circuit: Select Circuit 2
│   │   └── Or enter Circuit 2 Resource ID (if cross-sub)
│   │       + Authorization Key
│   ├── Address prefix: /29 subnet (for BGP peering between circuits)
│   │   ⚠️ Must not overlap with any existing address space
│   └── Add
│
└── Result
    ├── On-prem Site A ↔ ER Circuit 1 ↔ Microsoft backbone ↔
    │   ER Circuit 2 ↔ On-prem Site B
    ├── ⚠️ Traffic does NOT go over the internet
    └── ⚠️ Charged per GB transferred between circuits
```

---

### 22.7 Monitor ExpressRoute Circuit

> **Portal:** `ExpressRoute Circuit → Monitoring`

```
Monitor ExpressRoute
│
├── View Metrics
│   │   Portal: ExpressRoute Circuit → Monitoring → Metrics
│   ├── BitsInPerSecond / BitsOutPerSecond → throughput
│   ├── BGP Availability (%) → routing health
│   │   ⚠️ < 100% = BGP session issues
│   ├── ARP Availability (%) → Layer 2 connectivity
│   │   ⚠️ < 100% = physical/L2 link problem
│   ├── DroppedInBitsPerSecond / DroppedOutBitsPerSecond
│   └── Filter by: Peering Type (Private/Microsoft)
│
├── Gateway Metrics
│   │   Portal: ExpressRoute Gateway → Monitoring → Metrics
│   ├── Gateway BitsInPerSecond / BitsOutPerSecond
│   ├── Gateway CPU utilization
│   ├── Packets per second
│   └── Count of routes advertised / learned
│
├── Diagnostic Logs
│   │   Portal: ExpressRoute Circuit → Monitoring → Diagnostic settings
│   ├── + Add diagnostic setting
│   ├── Logs: BGP Route Table, ARP Table
│   ├── Destination: Log Analytics / Storage
│   └── Save
│
├── Configure Alerts
│   ├── Signal: BGP Availability < 99%
│   ├── Signal: BitsInPerSecond > threshold
│   ├── Action Group: Email / SMS
│   └── Create
│
└── Service Health
    ├── Portal: Monitor → Service Health → ExpressRoute
    └── Check for planned maintenance / outages
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
