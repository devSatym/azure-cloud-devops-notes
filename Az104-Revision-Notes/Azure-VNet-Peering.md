<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Virtual Network Peering — AZ-104 Revision Notes

---

## 1. What is VNet Peering?

- Connects **two Azure VNets** so resources communicate via **private IP addresses**
- Traffic stays on the **Microsoft Azure backbone** — never traverses the public internet
- Provides **low-latency, high-bandwidth** connectivity between VNets
- **Non-transitive** by default — if A↔B and B↔C, A ≠ C
- Two peering links created per connection (one in each direction)
- Supported across **different subscriptions**, **different Azure AD tenants**, **different regions**

> ⚠️ **EXAM TIP:** VNet Peering is **non-transitive**. If VNet-A is peered with VNet-B, and VNet-B is peered with VNet-C, VNet-A **CANNOT** communicate with VNet-C unless you explicitly peer A↔C or use a hub with gateway transit/NVA.

---

## 2. Types of VNet Peering

| Feature | **Regional Peering** | **Global Peering** |
|---|---|---|
| **Scope** | Same Azure region | Different Azure regions |
| **Latency** | Azure backbone, lowest latency | Azure backbone, slightly higher |
| **Bandwidth** | No bandwidth cap | No bandwidth cap |
| **Basic Load Balancer** | ✅ Frontend accessible | ❌ Cannot access Basic LB frontend |
| **Standard Load Balancer** | ✅ | ✅ |
| **Gateway Transit** | ✅ Supported | ✅ Supported |
| **Data transfer cost** | Ingress + Egress (lower rate) | Ingress + Egress (higher rate) |
| **Use case** | Same-region multi-VNet | Cross-region connectivity |

> ⚠️ **EXAM TIP:** VMs behind a **Basic Load Balancer** are NOT accessible over **Global VNet Peering**. Upgrade to **Standard Load Balancer** for global peering scenarios. This is heavily tested.

> ⚠️ **EXAM TIP:** Regional peering = cheaper than global peering. Both use the Azure backbone (never internet).

---

## 3. Key Properties & Behavior

### 3.1 Address Space Rules
- Peered VNets **MUST NOT have overlapping address spaces**
- If address spaces overlap → peering creation will **fail**
- Address space can be **updated** on a peered VNet (address space sync) without deleting peering

### 3.2 Peering Link States

| State | Meaning |
|---|---|
| **Initiated** | Only one side's link created (other side not yet configured) |
| **Connected** | Both sides configured — peering is **active** |
| **Disconnected** | One side deleted their link — peering is **broken** |

> ⚠️ **EXAM TIP:** Both peering links must show **Connected** for traffic to flow. If one side shows **Initiated**, the peering is NOT yet functional.

> ⚠️ **EXAM TIP:** If you **delete** one side of the peering, the other side changes to **Disconnected** and the peering is broken. You must re-create both links.

### 3.3 Non-Transitivity
```
VNet-A ←→ VNet-B ←→ VNet-C

A can reach B ✅
B can reach C ✅
A can reach C ❌ (NOT transitive)
```

**Solutions for transitivity:**
1. **Direct peering** — peer A↔C explicitly
2. **Hub-spoke with Gateway Transit** — hub VNet has VPN Gateway, spokes use remote gateway
3. **Hub-spoke with NVA** — route through Network Virtual Appliance in hub + UDRs
4. **Azure Virtual WAN** — managed hub for full-mesh connectivity

---

## 4. Peering Configuration Settings

| Setting | Default | Description |
|---|---|---|
| **Allow virtual network access** | ✅ Enabled | Allow traffic between peered VNets |
| **Allow forwarded traffic** | ❌ Disabled | Accept traffic NOT originating from the peered VNet (forwarded by NVA/gateway) |
| **Allow gateway transit** | ❌ Disabled | Let the peered VNet use YOUR VPN/ExpressRoute gateway |
| **Use remote gateways** | ❌ Disabled | Use the peered VNet's gateway instead of having your own |

### Setting Details

#### Allow Virtual Network Access
- When **Enabled**: Resources in both VNets can communicate via private IPs
- When **Disabled**: NSG rule `AllowVNetInBound` does NOT include the peered VNet
- Typically always **Enabled** (default)

#### Allow Forwarded Traffic
- When **Enabled**: Accepts traffic forwarded by an NVA or gateway from the peer (traffic not originating in the peer VNet)
- When **Disabled**: Only traffic originating FROM the peered VNet is accepted
- **Enable on spoke VNets** in hub-spoke topology so they accept traffic forwarded by the hub NVA

#### Allow Gateway Transit
- Enable on the VNet that **HAS the gateway** (the hub)
- Lets peered VNets use your gateway for on-prem connectivity
- Only ONE VNet in the peering relationship can enable this

#### Use Remote Gateways
- Enable on the VNet that **does NOT have a gateway** (the spoke)
- Uses the peered VNet's gateway
- **Mutually exclusive** with "Allow gateway transit" on the SAME VNet
- VNet CANNOT have its own gateway AND use remote gateway simultaneously

> ⚠️ **EXAM TIP:** You **CANNOT** enable both `Allow gateway transit` AND `Use remote gateways` on the **same VNet**. Hub enables "Allow gateway transit" → Spoke enables "Use remote gateways". Never both on one VNet.

> ⚠️ **EXAM TIP:** `Allow forwarded traffic` must be enabled on the **receiving** VNet to accept NVA-forwarded traffic. Combined with UDRs, this enables hub-spoke NVA routing.

> ⚠️ **EXAM TIP:** A VNet that has **Use remote gateways** enabled **CANNOT** also have its own VPN/ExpressRoute gateway deployed.

---

## 5. Gateway Transit

- Allows a peered VNet to use another VNet's **VPN or ExpressRoute Gateway** for on-prem connectivity
- Avoids deploying a gateway in every VNet (cost savings)
- Hub-spoke pattern: **Hub** has the gateway, **Spokes** use it via peering

### Configuration

| VNet Role | Setting to Enable |
|---|---|
| **Hub** (has gateway) | `Allow gateway transit` ✅ |
| **Spoke** (no gateway) | `Use remote gateways` ✅ |

### Gateway Transit Requirements
- Hub VNet must have a **deployed, active VPN or ExpressRoute Gateway**
- Spoke VNet must NOT have its own gateway
- Works with both **Regional** and **Global** peering
- Spoke VNet gets routes propagated from the hub gateway (BGP routes, on-prem routes)

#### Portal Path — Enable Gateway Transit
```
HUB VNet → Peerings → Select peering →
Allow gateway transit: ✅ Enabled → Save
```

```
SPOKE VNet → Peerings → Select peering →
Use remote gateways: ✅ Enabled → Save
```

> ⚠️ **EXAM TIP:** Gateway Transit is a **top exam scenario**: "Spoke VNets need on-prem access through hub VPN Gateway." Answer: Enable `Allow gateway transit` on hub, `Use remote gateways` on spokes.

> ⚠️ **EXAM TIP:** Only **one VNet per peering pair** can enable `Allow gateway transit`. You cannot have gateways in both and enable transit both ways.

---

## 6. Hub-Spoke Topology

```
                    On-Premises
                        │
                   VPN / ExpressRoute
                        │
                ┌───────┴───────┐
                │   Hub VNet    │
                │  (Gateway +   │
                │   NVA/FW)     │
                └─┬─────┬─────┬─┘
                  │     │     │
          Peering │     │     │ Peering
                  │     │     │
            ┌─────┴┐ ┌─┴───┐ ┌┴─────┐
            │Spoke1│ │Spoke2│ │Spoke3│
            └──────┘ └─────┘ └──────┘
```

### Hub-Spoke Requirements

| VNet | Settings |
|---|---|
| **Hub** | Allow gateway transit ✅, Allow forwarded traffic ✅ |
| **Spoke** | Use remote gateways ✅, Allow forwarded traffic ✅ |

### Spoke-to-Spoke Traffic via Hub NVA
- Spokes are peered to hub only — not to each other (non-transitive)
- Spoke → Hub NVA → Other Spoke
- Requires:
  1. **UDRs** on spoke subnets pointing to hub NVA IP
  2. **IP forwarding** enabled on NVA NIC
  3. **Allow forwarded traffic** enabled on all peerings
  4. NVA must be configured to forward/route traffic

> ⚠️ **EXAM TIP:** Spoke-to-spoke via hub NVA requires: UDRs + IP forwarding on NVA + Allow forwarded traffic on peerings. Missing any one = traffic won't flow.

---

## 7. Cross-Subscription & Cross-Tenant Peering

### Cross-Subscription Peering
- Peering between VNets in **different subscriptions** — fully supported
- User must have **Network Contributor** (or equivalent) on **both** subscriptions
- Both subscriptions can be under the same or different Azure AD tenants

### Cross-Tenant Peering
- Peering between VNets in **different Azure AD tenants**
- Cannot be created through the **portal** — must use **CLI, PowerShell, or ARM template**
- Requires the remote VNet **resource ID** (cannot browse to select)
- Each admin must grant the other's user a role on their VNet (Network Contributor)

#### CLI — Cross-Subscription Peering
```bash
# From VNet1 side
az network vnet peering create \
  -g <rg1> -n VNet1-to-VNet2 --vnet-name VNet1 \
  --remote-vnet /subscriptions/<sub2-id>/resourceGroups/<rg2>/providers/Microsoft.Network/virtualNetworks/VNet2 \
  --allow-vnet-access

# From VNet2 side (different subscription)
az network vnet peering create \
  -g <rg2> -n VNet2-to-VNet1 --vnet-name VNet2 \
  --remote-vnet /subscriptions/<sub1-id>/resourceGroups/<rg1>/providers/Microsoft.Network/virtualNetworks/VNet1 \
  --allow-vnet-access
```

> ⚠️ **EXAM TIP:** Cross-tenant peering **CANNOT** be done via the Azure Portal. Use CLI, PowerShell, or ARM templates. You need the full **resource ID** of the remote VNet.

> ⚠️ **EXAM TIP:** For cross-subscription peering, the user must have **Network Contributor** role (or equivalent permissions) on **BOTH VNets** in both subscriptions.

---

## 8. VNet Peering vs VPN Gateway (VNet-to-VNet) Comparison

| Feature | **VNet Peering** | **VPN Gateway (VNet-to-VNet)** |
|---|---|---|
| **Connectivity** | Azure backbone (private) | IPsec/IKE encrypted tunnel |
| **Encryption** | Not encrypted by default | ✅ Encrypted (IPsec) |
| **Bandwidth** | Full bandwidth (no gateway bottleneck) | Limited by gateway SKU |
| **Latency** | Very low | Higher (gateway processing) |
| **Transitivity** | Non-transitive | Non-transitive |
| **Cross-region** | ✅ Global peering | ✅ Cross-region S2S |
| **Cross-subscription** | ✅ | ✅ |
| **Cross-tenant** | ✅ (CLI/PS only) | ✅ |
| **Max connections** | 500 peerings per VNet | Depends on gateway SKU |
| **Gateway required** | ❌ No | ✅ Yes (both sides) |
| **Deployment time** | Seconds | 30–45 minutes |
| **Cost** | Per GB data transfer | Per hour + per GB transfer |
| **Setup complexity** | Simple | More complex |
| **Use case** | Same-org multi-VNet | Encryption needed / on-prem integration |

> ⚠️ **EXAM TIP:** VNet Peering = **faster, cheaper, simpler**, no encryption by default. VPN Gateway VNet-to-VNet = **encrypted** but slower, costlier, requires gateway. If question asks about "encrypted VNet-to-VNet" → VPN Gateway. If "lowest latency" → Peering.

> ⚠️ **EXAM TIP:** VNet Peering traffic is **NOT encrypted** by default. Use **VNet encryption** (DTLS between VMs) or **VPN Gateway** for encryption.

---

## 9. Service Chaining with Peering

- Use **UDRs** to direct traffic from one peered VNet through an **NVA or VPN Gateway** in another VNet
- Common patterns:
  - Spoke → Hub Firewall/NVA → Internet
  - Spoke → Hub Firewall/NVA → Other Spoke
  - Spoke → Hub VPN Gateway → On-premises
- Requires correct UDRs + "Allow forwarded traffic" + IP forwarding on NVA

---

## 10. Peering and Private DNS Resolution

- VNet peering does NOT automatically share **custom DNS settings**
- Each VNet retains its own DNS configuration
- For name resolution across peered VNets:
  - Use **Azure Private DNS Zones** linked to both VNets
  - Or configure **custom DNS servers** accessible from both VNets
  - Or use **Azure DNS Private Resolver**

> ⚠️ **EXAM TIP:** VNet peering provides **IP connectivity** but NOT automatic **DNS resolution**. You must separately configure DNS (Private DNS Zone links or custom DNS) for cross-VNet name resolution.

---

## 11. Security & RBAC

### Required Roles

| Action | Required Role |
|---|---|
| Create peering (both VNets same subscription) | **Network Contributor** on both VNets |
| Create peering (cross-subscription) | **Network Contributor** on both VNets in both subscriptions |
| Create peering (cross-tenant) | **Network Contributor** on both VNets + invitation flow |
| Delete peering | **Network Contributor** on the VNet where you delete |
| Modify peering settings | **Network Contributor** on the VNet being modified |

### Granular Permission
- Minimum permission: `Microsoft.Network/virtualNetworks/virtualNetworkPeerings/write` on the local VNet + `Microsoft.Network/virtualNetworks/peer/action` on the remote VNet

### NSGs with Peering
- NSG rules continue to apply **independently** in each VNet
- Service tag `VirtualNetwork` in NSGs **includes peered VNet address ranges**
- To block traffic between peered VNets → add explicit NSG **Deny** rules (since `AllowVNetInBound` at 65000 allows it by default)

> ⚠️ **EXAM TIP:** After peering, the `VirtualNetwork` service tag **automatically includes** the peered VNet's address space. To restrict cross-VNet traffic → must add explicit NSG deny rules with **lower priority number** than 65000.

> ⚠️ **EXAM TIP:** Minimum permissions for peering: **`write` on local VNet peerings** + **`peer/action` on remote VNet**. Both sides need configuration.

---

## 12. Monitoring & Diagnostics

### Peering Status
```
Virtual Network → Peerings →
View Status column: Initiated / Connected / Disconnected
```

### Metrics
- No dedicated peering metrics — monitor via **Network Watcher** Connection Monitor
- Monitor cross-VNet connectivity with **Connection Monitor** (latency, packet loss)

### Troubleshooting

| Symptom | Likely Cause |
|---|---|
| Peering shows **Initiated** | Other side not yet configured |
| Peering shows **Disconnected** | One side deleted peering — recreate both |
| Cannot create peering | Address spaces overlap |
| Traffic blocked after peering | NSG deny rule or "Allow virtual network access" disabled |
| Cannot reach Basic LB VMs | Global peering + Basic LB → upgrade to Standard LB |
| On-prem can't reach spoke VNet | Gateway transit not configured |
| Spoke can't reach other spoke | No direct peering + no hub NVA/UDR routing |

### Network Watcher Tools
```
Network Watcher → Connection troubleshoot →
Source VM (VNet-A) → Destination VM (VNet-B) → Check
```

---

## 13. Pricing Key Points

| Component | Cost |
|---|---|
| **VNet Peering (creation)** | Free — no setup cost |
| **Regional peering data transfer** | **Ingress: ~$0.01/GB** + **Egress: ~$0.01/GB** |
| **Global peering data transfer** | **Higher** — varies by region pair (e.g., ~$0.035/GB) |
| **Gateway Transit** | No extra peering cost (gateway charges still apply) |

- Both **ingress and egress** are charged (both directions)
- Regional is significantly **cheaper** than global
- No per-peering monthly fee — only data transfer charges

> ⚠️ **EXAM TIP:** VNet Peering charges apply on **both sides** — ingress AND egress. Global peering is **more expensive** than regional peering.

---

## 14. Limitations & Constraints

| Constraint | Limit |
|---|---|
| Peerings per VNet | **500** |
| VNets peered transitively | ❌ Non-transitive |
| Overlapping address spaces | ❌ Not allowed |
| Peering link creation | Must be created on **both** sides |
| Basic LB over global peering | ❌ Not supported |
| Cross-tenant via portal | ❌ Not supported (CLI/PS/ARM only) |
| VNet with `Use remote gateways` having own gateway | ❌ Not allowed |
| Both VNets using `Allow gateway transit` | ❌ Only one can enable it |
| Address space update while peered | ✅ Supported (address space sync) |
| Re-peering after deletion | Must recreate on **both** sides |

> ⚠️ **EXAM TIP:** Max **500 peerings per VNet**. If you need more than 500 connections → consider Azure Virtual WAN or hub-spoke with gateway transit.

---

## 15. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| Create peering | `az network vnet peering create -g <rg> -n <name> --vnet-name <vnet1> --remote-vnet <vnet2-name-or-id> --allow-vnet-access` |
| List peerings | `az network vnet peering list -g <rg> --vnet-name <vnet>` |
| Show peering details | `az network vnet peering show -g <rg> --vnet-name <vnet> -n <peering-name>` |
| Update peering (forwarded traffic) | `az network vnet peering update -g <rg> --vnet-name <vnet> -n <peering-name> --set allowForwardedTraffic=true` |
| Enable gateway transit | `az network vnet peering update -g <rg> --vnet-name <hub-vnet> -n <peering-name> --set allowGatewayTransit=true` |
| Enable use remote gateways | `az network vnet peering update -g <rg> --vnet-name <spoke-vnet> -n <peering-name> --set useRemoteGateways=true` |
| Delete peering | `az network vnet peering delete -g <rg> --vnet-name <vnet> -n <peering-name>` |
| Cross-sub peering | Use full resource ID for `--remote-vnet`: `/subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.Network/virtualNetworks/<vnet>` |

### PowerShell

| Action | Command |
|---|---|
| Create peering | `Add-AzVirtualNetworkPeering -Name <name> -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $vnet2.Id` |
| Get peering | `Get-AzVirtualNetworkPeering -VirtualNetworkName <vnet> -ResourceGroupName <rg>` |
| Update settings | `$peering = Get-AzVirtualNetworkPeering ...; $peering.AllowForwardedTraffic = $true; Set-AzVirtualNetworkPeering -VirtualNetworkPeering $peering` |
| Remove peering | `Remove-AzVirtualNetworkPeering -Name <name> -VirtualNetworkName <vnet> -ResourceGroupName <rg>` |

> ⚠️ **EXAM TIP:** You must create peering from **BOTH sides** — one `az network vnet peering create` from VNet1 and another from VNet2. A single command only creates one direction.

---

## 16. Quick-Fire Exam Points ⚡

1. VNet Peering = private connectivity over **Azure backbone** — never internet
2. **Non-transitive** — A↔B + B↔C ≠ A↔C
3. Two types: **Regional** (same region) and **Global** (cross-region)
4. Both peering links must show **Connected** for traffic to flow
5. **Overlapping address spaces** prevent peering — not allowed
6. Address spaces can be **updated while peered** (address space sync)
7. Peered VNets can be in **different subscriptions** or **different tenants**
8. **Cross-tenant** peering **cannot** be done through Azure Portal — use CLI/PS/ARM
9. Max **500 peerings** per VNet
10. **Basic Load Balancer** VMs cannot be reached over **Global** peering → use Standard LB
11. `Allow virtual network access` = enable/disable IP connectivity (default: enabled)
12. `Allow forwarded traffic` = accept NVA/gateway-forwarded traffic (default: disabled)
13. `Allow gateway transit` = share your gateway with peered VNet (default: disabled)
14. `Use remote gateways` = use peer's gateway (default: disabled)
15. **Cannot** enable `Allow gateway transit` AND `Use remote gateways` on the **same VNet**
16. VNet with `Use remote gateways` cannot have its **own gateway** deployed
17. Gateway Transit: Hub = `Allow gateway transit`, Spoke = `Use remote gateways`
18. Spoke-to-spoke via hub NVA requires: **UDRs** + **IP forwarding** + **Allow forwarded traffic**
19. Peering does NOT provide automatic **DNS resolution** — configure separately
20. `VirtualNetwork` service tag in NSGs **includes peered VNet address ranges**
21. To block cross-peering traffic → add NSG **Deny** rule with priority < 65000
22. Peering is **NOT encrypted** by default — use VNet encryption or VPN Gateway if needed
23. Required role: **Network Contributor** on **both** VNets
24. Minimum permission: `virtualNetworkPeerings/write` (local) + `peer/action` (remote)
25. Deleting one side → other side becomes **Disconnected** — must recreate both
26. Peering creation is **instant** (seconds) — VPN Gateway takes 30–45 minutes
27. Cost: **per-GB ingress + egress** on both sides. Global > Regional pricing
28. No monthly peering fee — only data transfer charges
29. VNet Peering = **no bandwidth cap**. VPN Gateway = limited by SKU throughput
30. If question says "lowest latency cross-VNet" → **VNet Peering** (not VPN Gateway)

---

## 17. Step-by-Step Configuration Mind Maps 🗺️

---

### 17.1 Create Regional VNet Peering (Same Subscription)

> **Portal:** `Virtual Network → Peerings → + Add`

```
Create Regional VNet Peering
│
├── Prerequisites
│   ├── Both VNets exist in the SAME region
│   ├── Address spaces do NOT overlap
│   │   ⚠️ Peering fails if address spaces overlap
│   └── RBAC: Network Contributor on BOTH VNets
│
├── Step 1: Navigate to First VNet
│   └── Virtual Network (VNet1) → Peerings → + Add
│
├── Step 2: Configure This VNet (VNet1) Side
│   ├── Peering link name: e.g., VNet1-to-VNet2
│   ├── Traffic to remote virtual network: Allow (default)
│   ├── Traffic forwarded from remote virtual network: Block / Allow
│   │   ⚠️ Enable "Allow" if using hub NVA forwarding
│   ├── Virtual network gateway or Route Server:
│   │   ├── None (default)
│   │   ├── Use this virtual network's gateway → Allow gateway transit
│   │   └── Use the remote virtual network's gateway → Use remote gateways
│   │       ⚠️ Cannot enable both on the same VNet
│   └── ⚠️ This VNet must NOT have a gateway if using remote gateways
│
├── Step 3: Configure Remote VNet (VNet2) Side
│   ├── Peering link name: e.g., VNet2-to-VNet1
│   ├── Subscription: Same (or different for cross-sub)
│   ├── Virtual Network: Select VNet2
│   │   ⚠️ VNet2 must be in same region for regional peering
│   ├── Traffic to remote virtual network: Allow
│   ├── Traffic forwarded from remote virtual network: Block / Allow
│   └── Virtual network gateway or Route Server: None / Configure
│
├── Step 4: Add
│   └── Both links created simultaneously
│       ⚠️ Both must show "Connected" status
│
└── Step 5: Verify
    ├── VNet1 → Peerings → Status: Connected
    ├── VNet2 → Peerings → Status: Connected
    └── Test: Ping/RDP from VM in VNet1 to VM in VNet2 (private IP)
```

---

### 17.2 Create Global VNet Peering (Cross-Region)

> **Portal:** `Virtual Network → Peerings → + Add`

```
Create Global VNet Peering
│
├── Prerequisites
│   ├── Both VNets exist (in DIFFERENT regions)
│   ├── Address spaces do NOT overlap
│   ├── ⚠️ No Basic Load Balancers if cross-region access needed
│   │   ⚠️ Basic LB VMs unreachable over global peering
│   └── RBAC: Network Contributor on both VNets
│
├── Step 1: Navigate to VNet1
│   └── Virtual Network (VNet1, e.g., East US) → Peerings → + Add
│
├── Step 2: Configure This VNet Side
│   ├── Peering link name: e.g., EastUS-to-WestEU
│   ├── Traffic settings (same as regional peering)
│   └── Gateway transit settings (same as regional)
│
├── Step 3: Configure Remote VNet Side
│   ├── Subscription: Select (can be different)
│   ├── Virtual Network: Select VNet2 (e.g., West Europe)
│   │   ⚠️ Portal auto-detects cross-region = Global Peering
│   ├── Peering link name: e.g., WestEU-to-EastUS
│   └── Traffic and gateway settings
│
├── Step 4: Add
│
└── ⚠️ Notes
    ├── Global peering data transfer costs are HIGHER
    ├── Basic LB frontend IPs NOT accessible
    └── Gateway transit IS supported for global peering
```

---

### 17.3 Configure Hub-Spoke with Gateway Transit

> **Portal:** `Hub VNet → Peerings → + Add` (for each spoke)

```
Hub-Spoke with Gateway Transit
│
├── Prerequisites
│   ├── Hub VNet has a deployed VPN or ExpressRoute Gateway
│   │   ⚠️ Gateway must be ACTIVE before enabling transit
│   ├── Spoke VNets do NOT have their own gateways
│   │   ⚠️ Spoke cannot have a gateway AND use remote gateways
│   ├── No overlapping address spaces
│   └── RBAC: Network Contributor on hub + all spoke VNets
│
├── Step 1: Create Peering from Hub to Spoke
│   │   Hub VNet → Peerings → + Add
│   ├── This VNet (Hub):
│   │   ├── Peering link name: Hub-to-Spoke1
│   │   ├── Traffic to remote: Allow
│   │   ├── Traffic forwarded from remote: Allow
│   │   │   ⚠️ Enable this for NVA forwarding
│   │   └── Virtual network gateway: Use this virtual network's gateway
│   │       ⚠️ = "Allow gateway transit" ✅
│   │
│   └── Remote VNet (Spoke1):
│       ├── Peering link name: Spoke1-to-Hub
│       ├── Traffic to remote: Allow
│       ├── Traffic forwarded from remote: Allow
│       └── Virtual network gateway: Use the remote virtual network's gateway
│           ⚠️ = "Use remote gateways" ✅
│
├── Step 2: Repeat for Each Spoke
│   ├── Hub-to-Spoke2, Hub-to-Spoke3, etc.
│   └── Same gateway transit settings
│
├── Step 3: Verify
│   ├── All peerings show "Connected"
│   ├── Spoke VMs can reach on-prem via hub gateway
│   └── Check effective routes on spoke VM:
│       VM → Networking → Effective routes
│       → On-prem routes should appear (via VNetGateway)
│
└── ⚠️ Critical Notes
    ├── On-prem routes propagated to spokes via gateway BGP
    ├── Spoke-to-spoke still NOT possible (non-transitive)
    │   → Need UDR + NVA in hub for spoke-to-spoke
    └── Only ONE side enables gateway transit per peering
```

---

### 17.4 Configure Spoke-to-Spoke via Hub NVA

> **Portal:** `Route Tables + VNet Peering settings`

```
Spoke-to-Spoke via Hub NVA
│
├── Prerequisites
│   ├── Hub VNet with NVA (e.g., Azure Firewall or 3rd-party NVA)
│   ├── Hub peered to Spoke1 and Spoke2
│   ├── NVA has IP forwarding enabled
│   │   ⚠️ Without IP forwarding on NVA NIC, traffic is dropped
│   └── All peerings have "Allow forwarded traffic" enabled
│
├── Step 1: Enable IP Forwarding on NVA
│   │   Portal: NVA VM → Networking → NIC → IP configurations
│   └── IP forwarding: Enabled → Save
│       ⚠️ BOTH Azure NIC setting AND OS-level forwarding must be enabled
│
├── Step 2: Verify Peering Settings
│   ├── Hub ↔ Spoke1 peering:
│   │   ├── Hub side: Allow forwarded traffic ✅
│   │   └── Spoke1 side: Allow forwarded traffic ✅
│   └── Hub ↔ Spoke2 peering:
│       ├── Hub side: Allow forwarded traffic ✅
│       └── Spoke2 side: Allow forwarded traffic ✅
│
├── Step 3: Create UDR for Spoke1
│   │   Portal: Route tables → + Create
│   ├── Route: Spoke2 address space → Next hop: Virtual Appliance → NVA IP
│   │   Example: 10.2.0.0/16 → Virtual Appliance → 10.0.1.4 (NVA IP)
│   └── Associate to Spoke1 subnet
│
├── Step 4: Create UDR for Spoke2
│   │   Portal: Route tables → + Create
│   ├── Route: Spoke1 address space → Next hop: Virtual Appliance → NVA IP
│   │   Example: 10.1.0.0/16 → Virtual Appliance → 10.0.1.4 (NVA IP)
│   └── Associate to Spoke2 subnet
│
└── Step 5: Verify
    ├── Spoke1 VM → ping Spoke2 VM (should route through NVA)
    ├── Check VM effective routes: Next hop = Virtual Appliance
    └── Use Network Watcher → Next Hop to verify routing
        ⚠️ All 4 pieces needed: UDR + IP forwarding + Allow forwarded traffic + NVA config
```

---

### 17.5 Cross-Subscription Peering

> **Portal:** `Virtual Network → Peerings → + Add`

```
Cross-Subscription Peering
│
├── Prerequisites
│   ├── User has Network Contributor on VNet in Subscription A
│   ├── User has Network Contributor on VNet in Subscription B
│   │   ⚠️ Same user needs permissions on BOTH subscriptions
│   │   OR each admin creates their side
│   └── Address spaces do not overlap
│
├── Method 1: Portal (Same Tenant)
│   │   VNet1 (Sub A) → Peerings → + Add
│   ├── Remote VNet:
│   │   ├── Subscription: Select Subscription B
│   │   └── Virtual Network: Select VNet2
│   └── Configure settings → Add
│       ⚠️ Both links created automatically via portal
│
├── Method 2: CLI (Cross-Tenant)
│   │   ⚠️ Cross-tenant CANNOT use portal
│   ├── Admin A creates peering from VNet1:
│   │   az network vnet peering create \
│   │     -g <rg1> -n VNet1-to-VNet2 --vnet-name VNet1 \
│   │     --remote-vnet <full-resource-id-of-vnet2> \
│   │     --allow-vnet-access
│   ├── Admin B creates peering from VNet2:
│   │   az network vnet peering create \
│   │     -g <rg2> -n VNet2-to-VNet1 --vnet-name VNet2 \
│   │     --remote-vnet <full-resource-id-of-vnet1> \
│   │     --allow-vnet-access
│   └── ⚠️ Must use full resource IDs for remote VNet
│
└── Verify
    ├── Both peerings show "Connected"
    └── Test private IP connectivity between VMs
```

---

### 17.6 Modify Peering Settings

> **Portal:** `Virtual Network → Peerings → Select peering`

```
Modify Peering Settings
│
├── Portal: Virtual Network → Peerings → Click peering name
│
├── Modifiable Settings
│   ├── Traffic to remote virtual network: Allow / Block
│   ├── Traffic forwarded from remote virtual network: Allow / Block
│   └── Virtual network gateway or Route Server:
│       ├── None
│       ├── Use this virtual network's gateway (Allow gateway transit)
│       └── Use the remote virtual network's gateway (Use remote gateways)
│           ⚠️ Cannot change gateway transit if gateway doesn't exist
│
├── Save
│   └── Changes take effect immediately
│       ⚠️ Disabling "Allow virtual network access" = instant traffic disruption
│
└── Delete Peering
    ├── Virtual Network → Peerings → Select → Delete
    ├── ⚠️ Deleting one side → other side becomes "Disconnected"
    └── ⚠️ Must delete and recreate on BOTH sides to re-peer
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
