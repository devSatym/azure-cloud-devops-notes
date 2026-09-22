<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure VPN Gateway — AZ-104 Revision Notes

---

## 1. What is Azure VPN Gateway?

- Sends **encrypted traffic** between Azure VNet and on-premises / other VNets over the **public internet**
- Uses **IPsec/IKE** tunnels (Site-to-Site, VNet-to-VNet) or **SSTP/OpenVPN/IKEv2** (Point-to-Site)
- Deployed into a **dedicated subnet** named **GatewaySubnet**
- Each VNet can have **only ONE VPN Gateway**
- Can coexist with **one ExpressRoute Gateway** in the same VNet

---

## 2. Key Components

| Component | Purpose |
|---|---|
| **VPN Gateway** | Virtual network gateway of type "VPN" — terminates IPsec tunnels |
| **GatewaySubnet** | Dedicated subnet (MUST be named `GatewaySubnet`) for gateway VMs |
| **Public IP Address** | Assigned to gateway — on-prem devices connect to this IP |
| **Local Network Gateway** | Represents the on-prem VPN device (its public IP + on-prem address spaces) |
| **Connection** | Links VPN Gateway to Local Network Gateway (S2S) or another VPN Gateway (V2V) |
| **VPN Client** | Software on user devices for P2S connections |

---

## 3. VPN Types

| Type | Description |
|---|---|
| **Route-based** | Uses routing table to direct traffic into tunnels. Supports **P2S, S2S, multi-site, VNet-to-VNet, coexistence with ExpressRoute**. **Most common / recommended.** |
| **Policy-based** | Uses IPsec policies (traffic selectors) to encrypt traffic. **Only Basic SKU.** Supports **1 S2S tunnel only.** No P2S. |

> ⚠️ **EXAM TIP:** **Route-based = almost always the answer.** Policy-based = Basic SKU only, max 1 tunnel, no P2S, no VNet-to-VNet. If question mentions multiple tunnels → Route-based.

---

## 4. Connection Types

### 4.1 Site-to-Site (S2S)

- Azure VNet ↔ On-premises network over **IPsec/IKE VPN tunnel**
- Requires **on-prem VPN device** with public IP
- Uses **Local Network Gateway** to represent on-prem
- Supports **IKEv1 and IKEv2**
- Can have **multiple S2S connections** (route-based, non-Basic SKU)

### 4.2 Point-to-Site (P2S)

- **Individual client devices** → Azure VNet (remote access VPN)
- No on-prem VPN device needed
- Protocols: **SSTP** (Windows only), **OpenVPN** (all platforms), **IKEv2** (Windows/macOS)
- Auth: **Certificate-based**, **Azure AD (Entra ID)**, **RADIUS**

| Protocol | Platform | Port |
|---|---|---|
| **SSTP** | Windows only | TCP 443 |
| **OpenVPN** | Windows, macOS, Linux, iOS, Android | TCP 443 or UDP 1194 |
| **IKEv2** | Windows, macOS | UDP 500, 4500 |

### 4.3 VNet-to-VNet (V2V)

- Azure VNet ↔ Azure VNet over **IPsec/IKE tunnel** through Azure backbone
- Can be cross-region, cross-subscription
- Each VNet needs its own VPN Gateway
- Alternative to **VNet Peering** (peering = no gateway, lower latency, no encryption)

> ⚠️ **EXAM TIP:** **VNet-to-VNet VPN** = encrypted, uses gateways, higher latency. **VNet Peering** = not encrypted (but private), no gateways, lower latency, higher bandwidth. Peering is preferred unless encryption over backbone is required.

---

## 5. SKU Comparison

| SKU | S2S Tunnels | P2S Connections | Throughput | Active-Active | BGP | AZ Support |
|---|---|---|---|---|---|---|
| **Basic** | Max **10** | Max **128** | 100 Mbps | ❌ | ❌ | ❌ |
| **VpnGw1** | Max **30** | Max **250** | 650 Mbps | ✅ | ✅ | ❌ |
| **VpnGw2** | Max **30** | Max **500** | 1 Gbps | ✅ | ✅ | ❌ |
| **VpnGw3** | Max **30** | Max **1,000** | 1.25 Gbps | ✅ | ✅ | ❌ |
| **VpnGw4** | Max **100** | Max **5,000** | 5 Gbps | ✅ | ✅ | ❌ |
| **VpnGw5** | Max **100** | Max **10,000** | 10 Gbps | ✅ | ✅ | ❌ |
| **VpnGw1AZ** | Max **30** | Max **250** | 650 Mbps | ✅ | ✅ | ✅ |
| **VpnGw2AZ** | Max **30** | Max **500** | 1 Gbps | ✅ | ✅ | ✅ |
| **VpnGw3AZ** | Max **30** | Max **1,000** | 1.25 Gbps | ✅ | ✅ | ✅ |
| **VpnGw4AZ** | Max **100** | Max **5,000** | 5 Gbps | ✅ | ✅ | ✅ |
| **VpnGw5AZ** | Max **100** | Max **10,000** | 10 Gbps | ✅ | ✅ | ✅ |

> ⚠️ **EXAM TIP:** **Basic SKU** = no BGP, no Active-Active, no resize (must delete & recreate), policy-based supported, max 10 S2S & 128 P2S. Cannot upgrade Basic → other SKUs in-place.

> ⚠️ **EXAM TIP:** **AZ SKUs** (VpnGw1AZ–VpnGw5AZ) = zone-redundant. Use **Standard SKU Public IP** with AZ gateways. Non-AZ SKUs use Basic Public IP.

> ⚠️ **EXAM TIP:** Can resize within same generation (e.g., VpnGw1→VpnGw3) but **cannot resize between AZ and non-AZ** or to/from Basic.

---

## 6. GatewaySubnet

- **MUST** be named exactly **`GatewaySubnet`** — no other name accepted
- Dedicated to gateway resources only (no VMs, NSGs are NOT recommended)
- Recommended size: **/27 or larger** (supports coexistence with ExpressRoute)
- Minimum: /29 (8 IPs) but NOT recommended
- Contains Azure-managed gateway VMs (not visible to you)

### Portal Path — Create GatewaySubnet
```
Virtual Network → Settings → Subnets →
+ Gateway subnet → Address range (e.g., /27) → Save
```

> ⚠️ **EXAM TIP:** Subnet MUST be named `GatewaySubnet` (case-sensitive). This is different from Application Gateway which can use ANY subnet name. **Do NOT apply NSGs** to the GatewaySubnet.

---

## 7. Site-to-Site (S2S) VPN — Detailed

### Requirements
- On-prem VPN device with **public IP** (not behind NAT for policy-based)
- Compatible VPN device (see Microsoft docs)
- Non-overlapping address spaces between VNet and on-prem
- **Shared key (PSK)** — must match on both sides

### Local Network Gateway
- Represents on-prem VPN device in Azure
- Contains: **On-prem VPN device public IP** + **on-prem address prefixes**
- If on-prem address spaces change → update Local Network Gateway

### Portal Path — Create Local Network Gateway
```
Home → + Create a resource → Local Network Gateway → Create →
Name, IP Address (on-prem VPN device), Address Space(s) (on-prem CIDRs),
Subscription, RG, Region → Create
```

### Portal Path — Create S2S Connection
```
VPN Gateway → Settings → Connections → + Add →
Name, Connection type: Site-to-site (IPsec),
Local Network Gateway: Select,
Shared Key (PSK): Enter → OK
```

> ⚠️ **EXAM TIP:** **Shared key must match** on Azure and on-prem device. If connection fails → verify shared key, on-prem device public IP, and address spaces.

---

## 8. Point-to-Site (P2S) VPN — Detailed

### Authentication Methods

| Method | Details |
|---|---|
| **Azure Certificate** | Root CA cert uploaded to Azure; client certs generated from root. No additional infra needed. |
| **Azure AD (Entra ID)** | Users auth with Entra ID credentials. **OpenVPN protocol only.** |
| **RADIUS** | Uses existing RADIUS server for auth. Supports MFA. |

### Address Pool
- Define **client address pool** (e.g., 172.16.0.0/24) — VPN clients get IPs from this pool
- Must NOT overlap with VNet or on-prem address spaces

### Portal Path — Configure P2S
```
VPN Gateway → Settings → Point-to-site configuration →
Address pool: Enter CIDR →
Tunnel type: OpenVPN / SSTP / IKEv2 →
Authentication type: Azure certificate / Azure AD / RADIUS →
Root certificate: Upload (.cer) →
Save → Download VPN client
```

> ⚠️ **EXAM TIP:** **Azure AD auth = OpenVPN only** (cannot use SSTP or IKEv2). Certificate auth works with all tunnel types. RADIUS = requires RADIUS server.

> ⚠️ **EXAM TIP:** P2S address pool must NOT overlap with VNet address space. Clients receive IPs from this pool.

---

## 9. Active-Active VPN Gateway

- Gateway gets **TWO public IPs** (two gateway instances)
- Both tunnels active simultaneously — **both handle traffic**
- On-prem device must establish **two tunnels** (one to each Azure public IP)
- Provides **higher availability** than Active-Standby
- NOT supported on **Basic SKU**
- Default = **Active-Standby** (one instance active, one standby)

### Portal Path — Enable Active-Active
```
VPN Gateway → Settings → Configuration →
Active-active mode: Enabled →
Second public IP address: Create/Select → Save
```

> ⚠️ **EXAM TIP:** **Active-Active** = 2 public IPs, 2 tunnels, both active. **Active-Standby** (default) = 1 public IP, standby takes over on failure (~10–15 sec failover). Active-Active requires VpnGw1 or higher.

---

## 10. BGP (Border Gateway Protocol)

- Dynamic routing protocol used with VPN Gateway
- Automatically exchanges routes between Azure and on-prem
- Eliminates need for static routes / UDRs for VPN traffic
- Azure VPN Gateway uses **ASN 65515** by default (configurable)
- NOT supported on **Basic SKU**
- Required for **transit routing** and **multi-site** scenarios

### Portal Path — Configure BGP
```
VPN Gateway → Settings → Configuration →
Configure BGP: Enabled →
ASN: 65515 (default, changeable) →
BGP peer IP address: auto-assigned from GatewaySubnet → Save
```

### On Connection (enable BGP)
```
VPN Gateway → Connections → Select connection →
Enable BGP: Yes → Save
```

> ⚠️ **EXAM TIP:** Default Azure VPN Gateway ASN = **65515**. On-prem ASN must be **different** from Azure ASN. BGP = dynamic routing (no manual route maintenance).

---

## 11. VPN Gateway vs ExpressRoute

| Feature | **VPN Gateway** | **ExpressRoute** |
|---|---|---|
| **Connection** | Over public internet (encrypted) | Private dedicated connection (not over internet) |
| **Protocol** | IPsec/IKE | BGP peering via connectivity provider |
| **Encryption** | ✅ IPsec (built-in) | ❌ Not encrypted by default (can add MACsec/VPN) |
| **Bandwidth** | Up to 10 Gbps (VpnGw5) | Up to 100 Gbps |
| **Latency** | Variable (internet-dependent) | Predictable, low latency |
| **Cost** | Lower | Higher |
| **Redundancy** | Active-Active | Built-in redundant circuits |
| **Use case** | Dev/test, small workloads, backup connectivity | Production, high-bandwidth, mission-critical |
| **Coexistence** | ✅ Can coexist in same VNet | ✅ Can coexist in same VNet |
| **SLA** | 99.9% (Active-Standby), 99.95% (Active-Active) | 99.95% |

> ⚠️ **EXAM TIP:** If question says "private connection, not over internet" → **ExpressRoute**. If "encrypted over internet" → **VPN Gateway**. They can **coexist** in the same VNet (requires separate gateways).

---

## 12. VPN Gateway vs VNet Peering (VNet-to-VNet)

| Feature | **VNet-to-VNet VPN** | **VNet Peering** |
|---|---|---|
| **Encryption** | ✅ IPsec | ❌ (private Azure backbone) |
| **Gateway required** | ✅ Both VNets need VPN GW | ❌ No gateways |
| **Bandwidth** | Limited by SKU | Azure backbone bandwidth |
| **Latency** | Higher | Lower |
| **Transitivity** | ❌ Not transitive (without BGP/NVA) | ❌ Not transitive |
| **Cross-region** | ✅ | ✅ (Global peering) |
| **Cross-subscription** | ✅ | ✅ |
| **Cost** | Gateway hourly + egress | Ingress + egress data transfer |

---

## 13. Forced Tunneling

- Routes **ALL internet-bound traffic** from Azure VMs back through on-prem via S2S VPN
- On-prem inspects/filters internet traffic (compliance requirement)
- Configured via **UDR** with default route `0.0.0.0/0` → VPN Gateway
- OR via **BGP** advertising `0.0.0.0/0` from on-prem

### Portal Path — Configure via UDR
```
Route Table → Routes → + Add →
Route name, Address prefix: 0.0.0.0/0,
Next hop type: Virtual network gateway → Add
```

> ⚠️ **EXAM TIP:** Forced tunneling = all internet traffic → on-prem. Configured via **UDR (0.0.0.0/0 → Virtual network gateway)** or **BGP default route advertisement**. Can break Azure PaaS services that need direct internet.

---

## 14. IPsec/IKE Custom Policy

- Override default IPsec/IKE parameters per connection
- Configure: IKE encryption, integrity, DH group, IPsec encryption, integrity, PFS group, SA lifetime
- Applied per **connection** (not per gateway)

### Portal Path
```
VPN Gateway → Connections → Select connection →
Settings → Configuration → IPsec/IKE policy:
Custom → Set parameters → Save
```

> ⚠️ **EXAM TIP:** Custom IPsec/IKE policies are per-connection. Both sides must match. Default uses Microsoft-recommended parameters.

---

## 15. Security & RBAC

### RBAC Roles

| Role | Permissions |
|---|---|
| **Network Contributor** | Full management of VPN Gateway, connections, LNG |
| **Contributor** | Full access to all resources |
| **Reader** | View-only |

| Action | Minimum Role |
|---|---|
| Create VPN Gateway | Network Contributor |
| Create/modify connections | Network Contributor |
| Create Local Network Gateway | Network Contributor |
| Configure P2S | Network Contributor |
| View gateway status | Reader |

---

## 16. Monitoring & Diagnostics

### Key Metrics

| Metric | Description |
|---|---|
| **Tunnel Bandwidth** | Throughput per tunnel (bytes/sec) |
| **Tunnel Egress/Ingress Bytes** | Data sent/received per tunnel |
| **Tunnel Egress/Ingress Packets** | Packets sent/received |
| **Tunnel NAT Allocations** | NAT allocation count |
| **Gateway P2S Bandwidth** | P2S throughput |
| **P2S Connection Count** | Active P2S connections |
| **BGP Peer Status** | Up/Down per BGP peer |
| **BGP Routes Advertised/Learned** | Route counts |

### Portal Path — View Metrics
```
VPN Gateway → Monitoring → Metrics →
Select metric → Filter by tunnel / connection → Apply
```

### Portal Path — Diagnostic Logs
```
VPN Gateway → Monitoring → Diagnostic settings →
+ Add → Select: GatewayDiagnosticLog, TunnelDiagnosticLog,
RouteDiagnosticLog, IKEDiagnosticLog, P2SDiagnosticLog →
Destination: Log Analytics / Storage → Save
```

### Portal Path — Connection Troubleshoot
```
VPN Gateway → Help → VPN Troubleshoot →
Select Connection / Gateway → Start troubleshooting
```

---

## 17. Pricing Key Points

| Component | Cost |
|---|---|
| **Gateway hourly** | Charged per hour gateway exists (even if no connections) |
| **S2S/V2V tunnels** | First 10 tunnels free (VpnGw1+), then per-tunnel |
| **P2S connections** | Included in gateway cost (up to SKU limit) |
| **Data egress** | Standard outbound data transfer charges |
| **Basic SKU** | Cheapest hourly rate |
| **AZ SKUs** | Higher rate than non-AZ equivalents |

> ⚠️ **EXAM TIP:** Gateway is charged **per hour even when idle** (no connections). To save cost in dev/test → **delete the gateway** (takes ~30–45 min to recreate). Unlike App Gateway, VPN Gateway cannot be stopped/deallocated.

---

## 18. Limitations & Constraints

| Constraint | Limit |
|---|---|
| VPN Gateways per VNet | **1** |
| Max S2S tunnels (Basic) | **10** |
| Max S2S tunnels (VpnGw1–3) | **30** |
| Max S2S tunnels (VpnGw4–5) | **100** |
| Max P2S connections (Basic) | **128** |
| Max P2S connections (VpnGw5) | **10,000** |
| GatewaySubnet name | **Must be `GatewaySubnet`** |
| GatewaySubnet recommended size | **/27 or larger** |
| Gateway deployment time | **30–45 minutes** |
| Active-Active | Not on Basic SKU |
| BGP | Not on Basic SKU |
| Policy-based VPN | Basic SKU only |

---

## 19. Quick-Fire Exam Points ⚡

1. VPN Gateway = **encrypted IPsec/IKE tunnel over the public internet**
2. Subnet MUST be named **`GatewaySubnet`** (case-sensitive, no exceptions)
3. GatewaySubnet recommended size = **/27 or larger**; do NOT attach NSGs
4. Only **ONE VPN Gateway per VNet** (can coexist with 1 ExpressRoute Gateway)
5. **Route-based = default/recommended**; Policy-based = Basic SKU only, 1 tunnel, no P2S
6. **Basic SKU**: no BGP, no Active-Active, no resize, max 10 S2S / 128 P2S
7. Cannot upgrade **Basic → VpnGw1** in-place (must delete & recreate)
8. Can resize within generation: VpnGw1↔VpnGw2↔VpnGw3; but NOT Basic ↔ VpnGwX
9. **AZ SKUs** require **Standard Public IP**; non-AZ SKUs use Basic Public IP
10. **Active-Standby** (default) = 1 public IP, ~10–15 sec failover
11. **Active-Active** = 2 public IPs, 2 tunnels both active, better HA, VpnGw1+
12. **Local Network Gateway** = represents on-prem device (its public IP + address prefixes)
13. **Shared Key (PSK)** must match on Azure and on-prem VPN device
14. **P2S protocols**: SSTP (Windows/TCP 443), IKEv2 (Win/Mac/UDP 500,4500), OpenVPN (all/TCP 443)
15. **Azure AD auth for P2S = OpenVPN protocol only** (most tested!)
16. P2S client address pool must **NOT overlap** with VNet or on-prem address spaces
17. **BGP default ASN = 65515** (Azure side); on-prem must use different ASN
18. Forced tunneling = **0.0.0.0/0 → Virtual network gateway** via UDR or BGP
19. VPN Gateway vs ExpressRoute: VPN = encrypted/internet, ER = private/dedicated/not encrypted
20. VNet-to-VNet VPN = encrypted with gateways; VNet Peering = no encryption, no gateways, faster
21. Gateway takes **30–45 minutes** to deploy — plan accordingly
22. Gateway is **charged per hour even idle** — cannot stop/deallocate like App Gateway
23. Custom IPsec/IKE policy = per-connection, not per-gateway
24. **Network Contributor** = minimum RBAC role to create/manage VPN Gateway
25. Connection status: **Connected**, **NotConnected**, **Unknown**
26. **Multi-site VPN** = multiple S2S connections to different on-prem sites (route-based, non-Basic)
27. Max throughput: Basic=100 Mbps, VpnGw1=650 Mbps, VpnGw5=10 Gbps
28. Diagnostic logs: Gateway, Tunnel, Route, IKE, P2S — send to Log Analytics for troubleshooting
29. First **10 S2S tunnels included** in gateway cost (VpnGw1+), additional tunnels extra
30. **Certificate auth for P2S**: upload root CA cert (.cer) to Azure, generate client certs from it

---

## 20. Step-by-Step Configuration Mind Maps 🗺️

---

### 20.1 Create VPN Gateway

> **Portal:** `Home → + Create a resource → Virtual Network Gateway`

```
Create VPN Gateway
│
├── Prerequisites
│   ├── VNet with GatewaySubnet already created
│   │   ⚠️ Subnet MUST be named "GatewaySubnet"
│   │   ⚠️ Recommended /27 or larger
│   └── Public IP address (create during wizard)
│
├── Step 1: Basics
│   ├── Subscription, Resource Group
│   ├── Name
│   ├── Region (must match VNet region)
│   ├── Gateway type: VPN (not ExpressRoute)
│   ├── VPN type: Route-based (recommended) / Policy-based
│   │   ⚠️ Policy-based = Basic SKU only, 1 tunnel, no P2S
│   ├── SKU: VpnGw1 / VpnGw2 / VpnGw3 / VpnGw1AZ / etc.
│   │   ⚠️ Basic SKU = no BGP, no Active-Active, cannot resize
│   │   ⚠️ AZ SKUs require Standard Public IP
│   ├── Generation: Generation1 / Generation2
│   ├── Virtual network: Select VNet
│   │   └── GatewaySubnet: Auto-selected
│   ├── Public IP address
│   │   ├── Create new or select existing
│   │   ├── Name
│   │   └── SKU: Basic (non-AZ) / Standard (AZ SKUs)
│   ├── Active-active mode: Disabled (default) / Enabled
│   │   └── If Enabled: Second Public IP required
│   │       ⚠️ Not available on Basic SKU
│   ├── Configure BGP: No (default) / Yes
│   │   └── If Yes: ASN (default 65515)
│   │       ⚠️ Not available on Basic SKU
│   └── ⚠️ Deployment takes 30–45 minutes
│
├── Step 2: Tags (Optional)
│
└── Step 3: Review + Create → Create
    └── RBAC: Network Contributor or higher
```

---

### 20.2 Create Site-to-Site (S2S) VPN Connection

> **Portal:** `VPN Gateway → Settings → Connections`

```
Create S2S VPN Connection
│
├── Prerequisites
│   ├── VPN Gateway deployed and running
│   ├── On-prem VPN device with public IP
│   ├── On-prem network address spaces known
│   └── Non-overlapping address spaces
│
├── Step 1: Create Local Network Gateway
│   │   Portal: Home → + Create a resource → Local Network Gateway
│   ├── Name
│   ├── Endpoint: IP address / FQDN
│   ├── IP address: On-prem VPN device public IP
│   ├── Address Space(s): On-prem CIDRs (e.g., 10.0.0.0/16)
│   │   ⚠️ Must match actual on-prem ranges
│   │   ⚠️ If ranges change, update LNG
│   ├── BGP settings (optional): Enable + ASN + BGP peer IP
│   ├── Subscription, RG, Region
│   └── Create
│
├── Step 2: Create Connection
│   │   Portal: VPN Gateway → Settings → Connections → + Add
│   ├── Name
│   ├── Connection type: Site-to-site (IPsec)
│   ├── Virtual network gateway: Auto-selected
│   ├── Local network gateway: Select created LNG
│   ├── Shared key (PSK): Enter key
│   │   ⚠️ MUST match on-prem VPN device config
│   ├── IKE Protocol: IKEv2 (recommended) / IKEv1
│   ├── Enable BGP: Yes/No
│   └── OK → Connection created
│
├── Step 3: Configure On-prem VPN Device
│   ├── Azure VPN Gateway public IP (from gateway overview)
│   ├── Same shared key (PSK)
│   ├── On-prem address spaces
│   └── IPsec/IKE parameters
│       ⚠️ Download config script: Connection → Download configuration
│
└── Step 4: Verify Connection
    ├── Portal: VPN Gateway → Connections → Status = "Connected" ✓
    └── ⚠️ If "NotConnected": check PSK, on-prem IP, NSG, firewall
```

---

### 20.3 Configure Point-to-Site (P2S) VPN

> **Portal:** `VPN Gateway → Settings → Point-to-site configuration`

```
Configure P2S VPN
│
├── Prerequisites
│   ├── VPN Gateway (VpnGw1 or higher for full features)
│   ├── For Certificate auth: Root CA cert + Client certs
│   ├── For Azure AD: Entra ID tenant + App registration
│   └── For RADIUS: RADIUS server accessible from gateway
│
├── Step 1: Configure P2S
│   │   Portal: VPN Gateway → Settings → Point-to-site configuration
│   │
│   ├── Address pool: e.g., 172.16.0.0/24
│   │   ⚠️ Must NOT overlap with VNet or on-prem
│   │
│   ├── Tunnel type:
│   │   ├── SSTP (SSL) → Windows only, TCP 443
│   │   ├── IKEv2 → Windows, macOS, UDP 500/4500
│   │   ├── OpenVPN (SSL) → All platforms, TCP 443
│   │   └── IKEv2 + OpenVPN → both protocols available
│   │   ⚠️ Azure AD auth = OpenVPN ONLY
│   │
│   ├── Authentication type:
│   │   ├── Azure certificate
│   │   │   ├── Root certificate: Paste base64 .cer content
│   │   │   └── Revoked certificates: Add thumbprints to revoke
│   │   ├── Azure Active Directory (Entra ID)
│   │   │   ├── Tenant ID
│   │   │   ├── Audience (Azure VPN App ID)
│   │   │   └── Issuer URL
│   │   │   ⚠️ OpenVPN protocol REQUIRED
│   │   └── RADIUS server
│   │       ├── Server IP
│   │       ├── Server secret
│   │       └── ⚠️ Requires network connectivity to RADIUS server
│   │
│   └── Save
│
├── Step 2: Download VPN Client
│   │   Portal: Point-to-site configuration → Download VPN client
│   ├── Extract zip → install on client device
│   └── Connect using configured auth method
│
└── Step 3: Verify
    ├── VPN Gateway → Point-to-site configuration → Allocated IPs
    └── Monitor: P2S Connection Count metric
```

---

### 20.4 Create VNet-to-VNet VPN Connection

> **Portal:** `VPN Gateway → Connections → + Add`

```
VNet-to-VNet VPN
│
├── Prerequisites
│   ├── VPN Gateway in EACH VNet (both must exist)
│   ├── Non-overlapping address spaces
│   └── Both gateways must be Route-based
│
├── Step 1: Create Connection on Gateway 1
│   │   Portal: VPN Gateway 1 → Connections → + Add
│   ├── Name: VNet1-to-VNet2
│   ├── Connection type: VNet-to-VNet
│   ├── Second virtual network gateway: Select Gateway 2
│   ├── Shared key (PSK): Enter key
│   │   ⚠️ Same PSK for both directions
│   ├── Enable BGP: Yes/No
│   └── OK
│
├── Step 2: Create Connection on Gateway 2 (reverse)
│   │   Portal: VPN Gateway 2 → Connections → + Add
│   ├── Name: VNet2-to-VNet1
│   ├── Connection type: VNet-to-VNet
│   ├── Second virtual network gateway: Select Gateway 1
│   ├── Shared key: Same as Step 1
│   └── OK
│   ⚠️ Need connections from BOTH sides
│   ⚠️ Can be different subscriptions / regions
│
└── Step 3: Verify
    └── Both connections show Status: "Connected"
```

---

### 20.5 Enable Active-Active Mode

> **Portal:** `VPN Gateway → Settings → Configuration`

```
Enable Active-Active VPN Gateway
│
├── Prerequisites
│   ├── SKU: VpnGw1 or higher (NOT Basic)
│   └── Second Public IP address
│
├── Step 1: Navigate
│   └── VPN Gateway → Settings → Configuration
│
├── Step 2: Enable Active-Active
│   ├── Active-active mode: Enabled
│   ├── Second public IP address:
│   │   ├── Create new Standard/Basic Public IP
│   │   └── Or select existing
│   └── Save
│   ⚠️ Gateway restarts during change (~30 min)
│
├── Step 3: Update On-prem VPN Device
│   ├── Create 2 tunnels (one to each Azure public IP)
│   └── Both tunnels carry traffic simultaneously
│
└── Result
    ├── 2 active gateway instances
    ├── 2 public IPs
    ├── 2 tunnels per S2S connection
    └── ⚠️ SLA improves from 99.9% → 99.95%
```

---

### 20.6 Configure Forced Tunneling

> **Portal:** `Route Table → Routes`

```
Configure Forced Tunneling
│
├── Option A: Using UDR (Route Table)
│   │
│   ├── Step 1: Create Route Table
│   │   └── Home → + Create → Route Table → Name, Region → Create
│   │
│   ├── Step 2: Add Default Route
│   │   │   Portal: Route Table → Routes → + Add
│   │   ├── Route name: ForcedTunnel
│   │   ├── Destination: IP Addresses
│   │   ├── CIDR: 0.0.0.0/0
│   │   ├── Next hop type: Virtual network gateway
│   │   └── Add
│   │   ⚠️ All internet traffic → on-prem via VPN
│   │
│   └── Step 3: Associate to Subnet(s)
│       │   Portal: Route Table → Subnets → + Associate
│       ├── Select VNet → Select subnet
│       └── ⚠️ Do NOT associate with GatewaySubnet
│
├── Option B: Using BGP
│   ├── Advertise 0.0.0.0/0 from on-prem BGP speaker
│   └── Azure VMs will route internet traffic via VPN gateway
│
└── ⚠️ Forced tunneling can break Azure PaaS services
    ⚠️ Use Service Endpoints or Private Endpoints for PaaS access
```

---

### 20.7 Monitor VPN Gateway

> **Portal:** `VPN Gateway → Monitoring`

```
Monitor VPN Gateway
│
├── View Metrics
│   │   Portal: VPN Gateway → Monitoring → Metrics
│   ├── Tunnel Bandwidth → throughput per tunnel
│   ├── Tunnel Egress/Ingress Bytes → data volume
│   ├── P2S Connection Count → active P2S clients
│   ├── BGP Peer Status → Up/Down
│   └── Gateway P2S Bandwidth → P2S throughput
│
├── Diagnostic Logs
│   │   Portal: VPN Gateway → Monitoring → Diagnostic settings
│   ├── + Add diagnostic setting
│   ├── Logs:
│   │   ├── GatewayDiagnosticLog → gateway events
│   │   ├── TunnelDiagnosticLog → tunnel connect/disconnect
│   │   ├── RouteDiagnosticLog → route changes
│   │   ├── IKEDiagnosticLog → IKE negotiation events
│   │   └── P2SDiagnosticLog → P2S connection events
│   ├── Destination: Log Analytics / Storage
│   └── Save
│
├── Connection Troubleshoot
│   │   Portal: VPN Gateway → Help → VPN Troubleshoot
│   ├── Select gateway or connection
│   ├── Storage account (for logs)
│   └── Start troubleshooting
│   ⚠️ Checks: config, connectivity, health, throughput
│
└── Alerts
    │   Portal: VPN Gateway → Monitoring → Alerts
    ├── Signal: Tunnel Bandwidth, P2S Count, BGP status
    ├── Action Group: Email/SMS
    └── Create
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
