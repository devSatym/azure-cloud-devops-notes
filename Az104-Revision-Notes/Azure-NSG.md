<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Network Security Groups (NSGs) — AZ-104 Revision Notes

---

## 1. What is an NSG?

- **Stateful** network traffic filter for Azure VNet resources
- Contains **inbound** and **outbound** security rules to **allow or deny** traffic
- Can be associated with **subnets** and/or **NICs** (Network Interfaces)
- Evaluated by **priority** — lowest number = highest priority = evaluated first
- **Free** — no cost for NSGs themselves
- Operates at **Layer 3 & Layer 4** (IP, port, protocol) — NOT Layer 7
- One NSG can be associated with **multiple** subnets and NICs
- One subnet or NIC can have only **one** NSG associated at a time

> ⚠️ **EXAM TIP:** NSGs are **stateful** — if inbound traffic is allowed, the return outbound traffic is **automatically allowed** (and vice versa). You do NOT need a separate outbound rule for return traffic.

> ⚠️ **EXAM TIP:** NSG is **NOT a firewall** — it's a packet filter at L3/L4. Azure Firewall operates at L3-L7 and supports FQDN filtering, threat intelligence, etc.

---

## 2. Key Components

| Component | Description |
|---|---|
| **Inbound Security Rules** | Filter traffic coming INTO the resource |
| **Outbound Security Rules** | Filter traffic going OUT of the resource |
| **Default Rules** | Pre-created rules (cannot be deleted, can be overridden) |
| **Service Tags** | Named groups of IP ranges for Azure services |
| **Application Security Groups (ASGs)** | Logical grouping of VMs for use in NSG rules |
| **Augmented Rules** | Rules using service tags, ASGs, multiple IPs/ports |

---

## 3. Default Rules (Cannot Be Deleted)

### Inbound Default Rules

| Priority | Name | Source | Destination | Port | Protocol | Action |
|---|---|---|---|---|---|---|
| **65000** | AllowVnetInBound | VirtualNetwork | VirtualNetwork | Any | Any | **Allow** |
| **65001** | AllowAzureLoadBalancerInBound | AzureLoadBalancer | Any | Any | Any | **Allow** |
| **65500** | DenyAllInBound | Any | Any | Any | Any | **Deny** |

### Outbound Default Rules

| Priority | Name | Source | Destination | Port | Protocol | Action |
|---|---|---|---|---|---|---|
| **65000** | AllowVnetOutBound | VirtualNetwork | VirtualNetwork | Any | Any | **Allow** |
| **65001** | AllowInternetOutBound | VirtualNetwork | Internet | Any | Any | **Allow** |
| **65500** | DenyAllOutBound | Any | Any | Any | Any | **Deny** |

> ⚠️ **EXAM TIP:** Default rules use priorities **65000, 65001, 65500**. You CANNOT delete them. You CAN override them with custom rules using a **lower priority number** (100–4096).

> ⚠️ **EXAM TIP:** By default, **outbound internet is ALLOWED** (AllowInternetOutBound at 65001). By default, **inbound from internet is DENIED** (DenyAllInBound at 65500). This is heavily tested.

> ⚠️ **EXAM TIP:** `VirtualNetwork` service tag includes: VNet address space + peered VNets + VPN-connected networks + on-prem (via gateway). It's **broader than just the VNet CIDR**.

---

## 4. NSG Rule Properties

| Property | Values / Range |
|---|---|
| **Name** | Unique within the NSG |
| **Priority** | **100 – 4096** (lower = higher priority) |
| **Source** | Any, IP/CIDR, Service Tag, or ASG |
| **Source port range** | Single port, range (e.g., 1024-65535), or `*` |
| **Destination** | Any, IP/CIDR, Service Tag, or ASG |
| **Destination port range** | Single port, range, or `*` |
| **Protocol** | TCP, UDP, ICMP, ESP, AH, or Any (`*`) |
| **Action** | **Allow** or **Deny** |
| **Direction** | Inbound or Outbound |

### Augmented Security Rules
- Allow **multiple** IP addresses, ranges, ports in a **single rule**
- Use **Service Tags** and **ASGs** as source/destination
- Reduce number of rules needed — simplifies management
- Example: Allow ports 80, 443, 8080 in ONE rule instead of three

> ⚠️ **EXAM TIP:** Priority range is **100 to 4096**. You CANNOT use priorities below 100 or above 4096 (65000+ is reserved for defaults). Best practice: increment by 10 or 100 for flexibility.

---

## 5. NSG Rule Evaluation Order

### Inbound Traffic (to resource)
```
Internet/Source → Subnet NSG (evaluated FIRST) → NIC NSG (evaluated SECOND) → VM
```

### Outbound Traffic (from resource)
```
VM → NIC NSG (evaluated FIRST) → Subnet NSG (evaluated SECOND) → Internet/Destination
```

### Rule Processing Within an NSG
1. Rules evaluated by **priority** (lowest number first)
2. **First match wins** — once a rule matches, processing stops
3. If no custom rule matches → **default rules** apply
4. If no rule matches at all → traffic is **denied** (DenyAll at 65500)

> ⚠️ **EXAM TIP:** **Inbound** = Subnet NSG first, then NIC NSG. **Outbound** = NIC NSG first, then Subnet NSG. Traffic must be **allowed by BOTH** NSGs if both are applied. If either denies, traffic is denied.

> ⚠️ **EXAM TIP:** If an NSG is applied to **subnet only** (no NIC NSG), then only subnet NSG is evaluated. If applied to **NIC only**, only NIC NSG is evaluated.

---

## 6. NSG Association

### Where NSGs Can Be Applied

| Association Level | Scope | Best For |
|---|---|---|
| **Subnet** | All resources in the subnet | Broad policies (e.g., block all SSH) |
| **NIC** | Specific VM NIC only | Granular per-VM rules |
| **Both** | NSG on subnet AND NIC | Layered defense |

### Association Rules
- One NSG → **multiple** subnets and NICs (many-to-one)
- One subnet → **one** NSG only
- One NIC → **one** NSG only
- NSG and resource must be in the **same region**
- NSG and resource can be in **different resource groups** (same subscription)

#### Portal Path — Associate NSG to Subnet
```
NSG → Settings → Subnets → + Associate →
Select VNet → Select Subnet → OK
```

#### Portal Path — Associate NSG to NIC
```
NSG → Settings → Network interfaces → + Associate →
Select NIC → OK
```

**Alternative (from VNet):**
```
Virtual Network → Subnets → Select Subnet →
Network security group dropdown → Select NSG → Save
```

**Alternative (from VM):**
```
VM → Networking → Network interface → Select NIC →
Network security group → Select NSG → Save
```

> ⚠️ **EXAM TIP:** NSG must be in the **same region** as the subnet/NIC. You CANNOT associate a US East NSG with a West Europe subnet.

> ⚠️ **EXAM TIP:** You can **dissociate** an NSG by selecting "None" from the dropdown. This removes all filtering (default Azure behavior applies).

---

## 7. Service Tags

- Pre-defined, Microsoft-managed **groups of IP address ranges**
- Automatically updated — you don't manage the IPs
- Cannot create custom service tags

### Key Service Tags for AZ-104

| Service Tag | Represents |
|---|---|
| **VirtualNetwork** | VNet address space + peered VNets + on-prem (via gateway) + VNet service endpoints |
| **AzureLoadBalancer** | Azure Load Balancer health probe IP (168.63.129.16) |
| **Internet** | All public IP space (outside VNet) |
| **Storage** | Azure Storage service IPs |
| **Storage.RegionName** | Storage IPs in specific region (e.g., `Storage.EastUS`) |
| **Sql** | Azure SQL Database service IPs |
| **AzureActiveDirectory** | Azure AD / Entra ID service IPs |
| **AzureMonitor** | Azure Monitor, Log Analytics, Application Insights |
| **GatewayManager** | Azure VPN Gateway management traffic |
| **AzureBackup** | Azure Backup service |
| **AzureKeyVault** | Azure Key Vault service |
| **AzureCloud** | All Azure datacenter IPs |
| **AzureCloud.RegionName** | Azure IPs in specific region |

> ⚠️ **EXAM TIP:** `VirtualNetwork` tag is NOT just the VNet CIDR — it includes peered VNets, on-prem connected via gateway, and VNet service endpoint addresses. This catches many people off guard.

> ⚠️ **EXAM TIP:** Service tags can be used as **source or destination** in NSG rules. They simplify rules and auto-update when Azure changes IP ranges.

---

## 8. Application Security Groups (ASGs)

- **Logical grouping** of VMs/NICs for NSG rule references
- Use ASGs as source/destination in NSG rules instead of IP addresses
- Simplifies security management for multi-tier applications (e.g., WebServers ASG, DBServers ASG)

### Key Constraints

| Constraint | Limit |
|---|---|
| All NICs in an ASG | Must be in the **same VNet** |
| NICs per ASG | Multiple (no hard limit on ASG membership) |
| ASGs per NIC | A NIC can be in **multiple ASGs** |
| Source + Dest ASGs in one rule | Both must be in the **same VNet** (cannot mix VNets) |
| ASG and NSG | Must be in the **same region** |
| Mixing ASG and IP in same rule | **Cannot** combine ASG with IP addresses in the same rule source/destination |

#### Portal Path — Create ASG
```
Home → Application security groups → + Create →
Subscription, Resource Group, Name, Region → Create
```

#### Portal Path — Add NIC to ASG
```
VM → Networking → Application security groups →
Configure → Select ASG(s) → Save
```

#### Portal Path — Use ASG in NSG Rule
```
NSG → Inbound/Outbound security rules → + Add →
Source: Application security group → Select ASG →
Destination: Application security group → Select ASG →
Port, Protocol, Action, Priority → Add
```

> ⚠️ **EXAM TIP:** All NICs in an ASG must be in the **same VNet**. You CANNOT add NICs from different VNets to the same ASG. This is a frequently tested constraint.

> ⚠️ **EXAM TIP:** You **cannot mix** an ASG with an IP address in the same source or destination field of a rule. Use one OR the other.

> ⚠️ **EXAM TIP:** Common exam scenario: "Three-tier app with Web, App, DB tiers" → create 3 ASGs, then create NSG rules using ASGs for clean traffic flow control.

---

## 9. NSG vs Azure Firewall Comparison

| Feature | **NSG** | **Azure Firewall** |
|---|---|---|
| **Layer** | L3/L4 (IP, port, protocol) | L3–L7 (including FQDN, URL) |
| **Scope** | Subnet / NIC level | Centralized, VNet-level |
| **FQDN filtering** | ❌ | ✅ |
| **Threat intelligence** | ❌ | ✅ |
| **TLS inspection** | ❌ | ✅ (Premium) |
| **NAT rules (DNAT)** | ❌ | ✅ |
| **Stateful** | ✅ | ✅ |
| **Cost** | **Free** | **Paid** (~$1.25/hr+) |
| **Logging** | NSG Flow Logs | Diagnostic logs + Log Analytics |
| **Centralized management** | Per-NSG | Azure Firewall Manager / Policy |
| **Best for** | Micro-segmentation within VNet | Centralized perimeter security |

> ⚠️ **EXAM TIP:** NSG = **free, L3/L4, per-subnet/NIC**. Azure Firewall = **paid, L3–L7, centralized**. If the question says "filter by FQDN" or "threat intelligence" → Azure Firewall. If "allow/deny by port and IP" → NSG.

---

## 10. Effective Security Rules

- Shows the **combined, evaluated** result of all NSG rules applied to a NIC
- Merges rules from both **subnet NSG** and **NIC NSG**
- Shows which rules actually apply and in what order
- Essential for troubleshooting "why can't I connect" scenarios

#### Portal Path — View Effective Security Rules
```
VM → Networking → Effective security rules tab
```

**Or via Network Watcher:**
```
Network Watcher → Effective security rules →
Select VM → View effective rules
```

> ⚠️ **EXAM TIP:** Use **Effective Security Rules** to see the COMBINED view of all NSG rules. Use **IP Flow Verify** (Network Watcher) to test if a SPECIFIC packet is allowed/denied.

---

## 11. NSG Flow Logs

- Log **all IP flows** through an NSG (allow + deny decisions)
- Stored in **Azure Storage Account** (JSON format)
- Two versions:

| Feature | Version 1 | Version 2 |
|---|---|---|
| Basic 5-tuple + action | ✅ | ✅ |
| Bytes & packets per flow | ❌ | ✅ |
| Flow state (Begin, Continue, End) | ❌ | ✅ |
| Throughput info | ❌ | ✅ |
| Recommended | Legacy | ✅ Always use V2 |

- Retention: **0–365 days** (0 = keep forever)
- Can be analyzed with **Traffic Analytics** (requires Log Analytics workspace)

#### Portal Path — Enable NSG Flow Logs
```
Network Watcher → NSG flow logs → + Create →
Select NSG → Storage account (same region) →
Retention: 0–365 days → Version: 2 →
Traffic Analytics: Yes/No →
Log Analytics workspace → Create
```

> ⚠️ **EXAM TIP:** NSG Flow Logs stored in **Storage Account** (NOT Log Analytics). Traffic Analytics reads from storage and sends insights to Log Analytics.

> ⚠️ **EXAM TIP:** Storage account must be in the **same region** as the NSG. Cross-region flow log storage is NOT supported.

> ⚠️ **EXAM TIP:** To use Traffic Analytics, you need BOTH a Storage Account AND a Log Analytics workspace. Processing interval: 10 min or 60 min.

---

## 12. Security & RBAC

### Key RBAC Roles

| Role | NSG Permissions |
|---|---|
| **Network Contributor** | Full CRUD on NSGs, rules, associations |
| **Owner** | Full access + role assignment |
| **Contributor** | Full access minus role assignment |
| **Reader** | View-only |
| **Security Admin** | View and update NSGs (via Security Center) |

### Required Permissions (Granular)

| Action | Permission |
|---|---|
| Create NSG | `Microsoft.Network/networkSecurityGroups/write` |
| Create/modify rules | `Microsoft.Network/networkSecurityGroups/securityRules/write` |
| Associate to subnet | `Microsoft.Network/virtualNetworks/subnets/write` + NSG permissions |
| Associate to NIC | `Microsoft.Network/networkInterfaces/write` + NSG permissions |
| View effective rules | `Microsoft.Network/networkInterfaces/effectiveNetworkSecurityGroups/action` |
| Enable flow logs | `Microsoft.Network/networkWatchers/configureFlowLog/action` |

### Resource Locks on NSGs
- **CanNotDelete** — prevents NSG deletion, rules can still be modified
- **ReadOnly** — prevents ALL changes (rules frozen, no association changes)

#### Portal Path — Add Lock
```
NSG → Settings → Locks → + Add →
Name, Lock type: Delete / Read-only → OK
```

> ⚠️ **EXAM TIP:** To associate an NSG to a subnet, you need permissions on **BOTH** the NSG (`networkSecurityGroups/write`) AND the subnet (`virtualNetworks/subnets/write`).

---

## 13. Common Exam Scenarios & Rules

### Allow RDP (Remote Desktop)

| Property | Value |
|---|---|
| Direction | Inbound |
| Source | Your IP or Any |
| Destination | VirtualNetwork (or VM IP) |
| Port | **3389** |
| Protocol | TCP |
| Action | Allow |

### Allow SSH

| Property | Value |
|---|---|
| Direction | Inbound |
| Source | Your IP or Any |
| Destination | VirtualNetwork |
| Port | **22** |
| Protocol | TCP |
| Action | Allow |

### Allow HTTP/HTTPS

| Property | Value |
|---|---|
| Direction | Inbound |
| Source | Internet (or Any) |
| Destination | ASG: WebServers |
| Port | **80, 443** |
| Protocol | TCP |
| Action | Allow |

### Deny All Inbound (Explicit)

| Property | Value |
|---|---|
| Priority | 4096 (or high number) |
| Source | Any |
| Destination | Any |
| Port | * |
| Protocol | Any |
| Action | Deny |

> ⚠️ **EXAM TIP:** Common ports: RDP = **3389/TCP**, SSH = **22/TCP**, HTTP = **80/TCP**, HTTPS = **443/TCP**, DNS = **53/TCP+UDP**, SMB = **445/TCP**, SQL Server = **1433/TCP**.

---

## 14. Monitoring & Alerts

### NSG Diagnostic Logging
```
NSG → Monitoring → Diagnostic settings → + Add →
Send to: Log Analytics / Storage / Event Hub →
Categories: NetworkSecurityGroupEvent, NetworkSecurityGroupRuleCounter →
Save
```

### Key Monitoring Tools

| Tool | Purpose |
|---|---|
| **NSG Flow Logs** | Log all allow/deny decisions per flow |
| **Traffic Analytics** | Visual insights from flow logs |
| **IP Flow Verify** | Test specific packet allow/deny |
| **Effective Security Rules** | View merged ruleset on a NIC |
| **NSG Diagnostics** | Detailed evaluation of rules for a traffic flow |
| **Activity Log** | Track NSG create/update/delete operations |

#### Portal Path — Activity Log
```
NSG → Monitoring → Activity log →
Filter by: Operation (Create/Update/Delete NSG rules)
```

---

## 15. Pricing Key Points

| Component | Cost |
|---|---|
| **NSG** | **Free** |
| **NSG Rules** | **Free** (unlimited rules up to limit) |
| **NSG Flow Logs** | Per GB stored (Storage Account costs) |
| **Traffic Analytics** | Per GB processed + Log Analytics ingestion |
| **ASGs** | **Free** |

> ⚠️ **EXAM TIP:** NSGs and ASGs are completely **free**. Costs only come from flow log storage and Traffic Analytics.

---

## 16. Limitations & Constraints

| Constraint | Limit |
|---|---|
| NSGs per subscription | **5,000** |
| Rules per NSG | **1,000** |
| NSGs per NIC | **1** |
| NSGs per subnet | **1** |
| Source/dest IPs per rule | **4,000** (augmented rules) |
| Ports per rule | **Multiple** (augmented rules) |
| Priority range | **100 – 4096** |
| Default rule priorities | **65000, 65001, 65500** |
| ASGs per subscription | **3,000** |
| ASG associations per NIC | **Multiple** |

> ⚠️ **EXAM TIP:** **5,000 NSGs** per subscription. **1,000 rules** per NSG. These are subscription-level limits.

---

## 17. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| Create NSG | `az network nsg create -g <rg> -n <name>` |
| Create inbound rule | `az network nsg rule create -g <rg> --nsg-name <nsg> -n AllowHTTP --priority 100 --destination-port-ranges 80 --access Allow --protocol Tcp --direction Inbound` |
| Create rule (multiple ports) | `az network nsg rule create -g <rg> --nsg-name <nsg> -n AllowWeb --priority 110 --destination-port-ranges 80 443 8080 --access Allow --protocol Tcp --direction Inbound` |
| Associate to subnet | `az network vnet subnet update -g <rg> --vnet-name <vnet> -n <subnet> --network-security-group <nsg>` |
| Associate to NIC | `az network nic update -g <rg> -n <nic> --network-security-group <nsg>` |
| Remove NSG from subnet | `az network vnet subnet update -g <rg> --vnet-name <vnet> -n <subnet> --network-security-group ""` |
| List NSGs | `az network nsg list -g <rg>` |
| List rules | `az network nsg rule list -g <rg> --nsg-name <nsg>` |
| Delete rule | `az network nsg rule delete -g <rg> --nsg-name <nsg> -n <rule-name>` |
| Show effective rules | `az network nic list-effective-nsg -g <rg> -n <nic>` |

### PowerShell

| Action | Command |
|---|---|
| Create NSG | `New-AzNetworkSecurityGroup -Name <name> -ResourceGroupName <rg> -Location <loc>` |
| Create rule | `$rule = New-AzNetworkSecurityRuleConfig -Name AllowHTTP -Protocol Tcp -Direction Inbound -Priority 100 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 80 -Access Allow` |
| Create NSG with rule | `New-AzNetworkSecurityGroup -Name <nsg> -ResourceGroupName <rg> -Location <loc> -SecurityRules $rule` |
| Add rule to existing NSG | `$nsg = Get-AzNetworkSecurityGroup -Name <nsg> -ResourceGroupName <rg>` then `Add-AzNetworkSecurityRuleConfig ... \| Set-AzNetworkSecurityGroup` |
| Associate to subnet | `Set-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name <subnet> -AddressPrefix <cidr> -NetworkSecurityGroup $nsg` then `$vnet \| Set-AzVirtualNetwork` |
| View effective rules | `Get-AzEffectiveNetworkSecurityGroup -NetworkInterfaceName <nic> -ResourceGroupName <rg>` |

> ⚠️ **EXAM TIP:** To **remove** an NSG association via CLI, set `--network-security-group ""` (empty string). In PowerShell, set `-NetworkSecurityGroup $null`.

---

## 18. Quick-Fire Exam Points ⚡

1. NSG = **stateful** L3/L4 packet filter — return traffic auto-allowed
2. NSG is **free** — no cost for NSGs, rules, or ASGs
3. Applied at **subnet** and/or **NIC** level — NOT at VNet level
4. One subnet/NIC → **one NSG only**. One NSG → **many subnets/NICs**
5. **Inbound evaluation**: Subnet NSG first → NIC NSG second
6. **Outbound evaluation**: NIC NSG first → Subnet NSG second
7. Traffic must be **allowed by BOTH** NSGs if both are applied
8. Priority range: **100–4096**. Lower number = higher priority
9. **First match wins** — processing stops after first matching rule
10. Default rules at priorities **65000, 65001, 65500** — CANNOT be deleted
11. **AllowVNetInBound** (65000) — allows all intra-VNet traffic by default
12. **AllowInternetOutBound** (65001) — all outbound internet allowed by default
13. **DenyAllInBound** (65500) — blocks all inbound not explicitly allowed
14. Service tag `VirtualNetwork` includes VNet + peered + VPN + on-prem addresses
15. Service tag `AzureLoadBalancer` = source IP **168.63.129.16** (health probes)
16. **ASG** = logical VM group — all NICs must be in the **same VNet**
17. Cannot mix ASG and IP addresses in the **same rule field**
18. A NIC can belong to **multiple ASGs**
19. NSG and resource must be in the **same region**
20. NSG and resource CAN be in **different resource groups** (same subscription)
21. **Augmented rules** = multiple IPs/ports/service tags in a single rule
22. **NSG Flow Logs** → stored in Storage Account (same region) as JSON
23. Always use Flow Log **Version 2** (includes bytes, packets, flow state)
24. **Traffic Analytics** requires Storage Account + Log Analytics workspace
25. **Effective Security Rules** = merged view of subnet + NIC NSGs
26. **IP Flow Verify** (Network Watcher) = test if specific packet is allowed/denied
27. NSGs per subscription = **5,000**. Rules per NSG = **1,000**
28. ASGs per subscription = **3,000**
29. NSG does NOT support **FQDN** filtering — use Azure Firewall for that
30. **ReadOnly lock** on NSG freezes ALL rules. **CanNotDelete** allows rule changes
31. To associate NSG to subnet, need permissions on **both** NSG and subnet
32. Common ports: RDP=**3389**, SSH=**22**, HTTP=**80**, HTTPS=**443**, SQL=**1433**
33. Standard Public IPs are **secure by default** — require NSG rule to allow inbound
34. Flow log retention: **0–365 days** (0 = keep forever)
35. NSG does NOT apply to **Private Endpoints** by default — must enable network policies on subnet

---

## 19. Step-by-Step Configuration Mind Maps 🗺️

---

### 19.1 Create NSG and Add Rules

> **Portal:** `Network Security Groups → + Create`

```
Create NSG and Add Rules
│
├── Step 1: Create NSG
│   │   Portal: Home → Network Security Groups → + Create
│   ├── Subscription
│   ├── Resource Group
│   ├── Name (unique within RG)
│   ├── Region
│   │   ⚠️ Must match region of subnets/NICs you want to associate
│   └── Review + Create → Create
│   ⚠️ RBAC: Network Contributor or higher
│
├── Step 2: Add Inbound Rule
│   │   Portal: NSG → Inbound security rules → + Add
│   ├── Source: Any / IP addresses / Service tag / ASG
│   │   ├── If IP: Enter CIDR (e.g., 10.0.0.0/24)
│   │   ├── If Service Tag: Select (e.g., Internet, AzureLoadBalancer)
│   │   └── If ASG: Select existing ASG
│   │       ⚠️ Cannot mix ASG with IP in same field
│   ├── Source port ranges: * (usually *)
│   ├── Destination: Any / IP addresses / Service tag / ASG
│   ├── Destination port ranges: Single (80) / Range (1024-65535) / Multiple (80,443,8080)
│   ├── Protocol: TCP / UDP / ICMP / Any
│   ├── Action: Allow / Deny
│   ├── Priority: 100–4096
│   │   ⚠️ Lower number = evaluated first. Leave gaps (100, 200, 300)
│   └── Name → Add
│
├── Step 3: Add Outbound Rule (if needed)
│   │   Portal: NSG → Outbound security rules → + Add
│   └── Same fields as inbound
│       ⚠️ Default outbound allows internet (65001) — only add if restricting
│
└── Step 4: Verify default rules
    └── NSG → Inbound/Outbound rules → View default rules tab
        ⚠️ Default rules CANNOT be deleted — override with higher priority
```

---

### 19.2 Associate NSG to Subnet

> **Portal:** `NSG → Subnets → + Associate`

```
Associate NSG to Subnet
│
├── Prerequisites
│   ├── NSG exists in same region as VNet
│   ├── VNet and subnet exist
│   └── RBAC: Network Contributor on both NSG and VNet
│       ⚠️ Need permissions on BOTH resources
│
├── Method 1: From NSG
│   │   Portal: NSG → Settings → Subnets → + Associate
│   ├── Virtual network: Select VNet
│   ├── Subnet: Select subnet
│   └── OK
│       ⚠️ Any existing NSG on that subnet is REPLACED
│       ⚠️ Applies immediately — no downtime, but traffic rules change instantly
│
├── Method 2: From VNet
│   │   Portal: Virtual Network → Subnets → Select subnet
│   ├── Network security group: Select NSG from dropdown
│   └── Save
│
└── Verify
    ├── NSG → Subnets → See associated subnets
    └── VNet → Subnet → See NSG field populated
```

---

### 19.3 Associate NSG to NIC

> **Portal:** `NSG → Network interfaces → + Associate`

```
Associate NSG to NIC
│
├── Prerequisites
│   ├── NSG exists in same region as NIC
│   └── RBAC: Network Contributor
│
├── Method 1: From NSG
│   │   Portal: NSG → Settings → Network interfaces → + Associate
│   ├── Select NIC from list
│   └── OK
│
├── Method 2: From VM
│   │   Portal: VM → Networking → Network interface → Click NIC
│   ├── NIC → Network security group
│   ├── Select NSG OR None (to remove)
│   └── Save
│
└── ⚠️ If BOTH subnet NSG and NIC NSG exist:
    ├── Inbound: Subnet first → NIC second
    ├── Outbound: NIC first → Subnet second
    └── Traffic must pass BOTH to be allowed
```

---

### 19.4 Create and Use Application Security Groups

> **Portal:** `Application security groups → + Create`

```
Create and Use ASGs
│
├── Step 1: Create ASGs
│   │   Portal: Home → Application security groups → + Create
│   ├── Subscription, Resource Group
│   ├── Name (e.g., WebServers, DBServers)
│   ├── Region
│   │   ⚠️ Must match region of VMs/NICs
│   └── Create
│   ⚠️ Create one ASG per application tier
│
├── Step 2: Add VMs to ASGs
│   │   Portal: VM → Networking → Application security groups
│   ├── Configure the application security groups
│   ├── Select ASG(s) to join
│   │   ⚠️ All NICs in an ASG must be in the SAME VNet
│   │   ⚠️ A NIC can be in MULTIPLE ASGs
│   └── Save
│
├── Step 3: Create NSG Rules Using ASGs
│   │   Portal: NSG → Inbound security rules → + Add
│   ├── Example 1: Allow Web traffic
│   │   ├── Source: Service tag → Internet
│   │   ├── Destination: ASG → WebServers
│   │   ├── Port: 80, 443
│   │   ├── Protocol: TCP
│   │   ├── Action: Allow
│   │   └── Priority: 100
│   │
│   ├── Example 2: Web → App tier
│   │   ├── Source: ASG → WebServers
│   │   ├── Destination: ASG → AppServers
│   │   ├── Port: 8080
│   │   ├── Action: Allow
│   │   └── Priority: 200
│   │
│   └── Example 3: App → DB tier
│       ├── Source: ASG → AppServers
│       ├── Destination: ASG → DBServers
│       ├── Port: 1433
│       ├── Action: Allow
│       └── Priority: 300
│
└── ⚠️ No need to update rules when VMs are added/removed — just update ASG membership
```

---

### 19.5 Enable NSG Flow Logs

> **Portal:** `Network Watcher → NSG flow logs → + Create`

```
Enable NSG Flow Logs
│
├── Prerequisites
│   ├── Network Watcher enabled in NSG's region
│   ├── NSG exists
│   ├── Storage account in SAME region as NSG
│   │   ⚠️ Cross-region storage NOT supported
│   └── Microsoft.Insights provider registered
│       └── az provider register --namespace Microsoft.Insights
│
├── Step 1: Create Flow Log
│   │   Portal: Network Watcher → NSG flow logs → + Create
│   ├── Select NSG
│   └── Flow log name
│
├── Step 2: Storage Configuration
│   ├── Storage account: Select (same region)
│   └── Retention (days): 0–365
│       ├── 0 = keep forever
│       └── ⚠️ Longer retention = more storage cost
│
├── Step 3: Flow Log Version
│   └── Version 2 ✅ (always select V2)
│       ⚠️ V2 adds bytes, packets, flow state
│
├── Step 4: Traffic Analytics (Optional)
│   ├── Enable: Yes
│   ├── Processing interval: 10 min / 60 min
│   │   ⚠️ 10 min = faster insights, higher cost
│   └── Log Analytics workspace: Select
│
└── Step 5: Create
    ⚠️ RBAC: Network Contributor +
    Microsoft.Network/networkWatchers/configureFlowLog/action
```

---

### 19.6 Troubleshoot NSG with Network Watcher

> **Portal:** `Network Watcher → IP flow verify / Effective security rules`

```
Troubleshoot NSG Issues
│
├── Scenario: VM cannot receive RDP/SSH/HTTP traffic
│
├── Tool 1: IP Flow Verify (test specific packet)
│   │   Portal: Network Watcher → IP flow verify
│   ├── Select VM
│   ├── Direction: Inbound
│   ├── Protocol: TCP
│   ├── Local IP: VM private IP
│   ├── Local port: 3389 (RDP) / 22 (SSH) / 80 (HTTP)
│   ├── Remote IP: Source IP
│   ├── Remote port: Any (e.g., 12345)
│   └── Result: Allowed / Denied + WHICH RULE caused it
│       ⚠️ Checks both subnet and NIC NSG rules
│
├── Tool 2: Effective Security Rules (view all rules)
│   │   Portal: VM → Networking → Effective security rules
│   └── Shows merged view of subnet + NIC NSG rules
│       ⚠️ Look for conflicting Deny rules with higher priority
│
├── Tool 3: NSG Diagnostics (detailed evaluation)
│   │   Portal: Network Watcher → NSG diagnostics
│   ├── Select VM, direction, protocol, source, destination
│   └── Shows ALL evaluated NSG rules and their effect
│
└── Common Fixes
    ├── Add Allow rule with LOWER priority number
    ├── Check if NSG is associated (not just created)
    ├── Check if BOTH subnet and NIC NSGs allow the traffic
    └── Verify Standard Public IP has NSG rule (secure by default)
        ⚠️ Standard PIPs require explicit NSG Allow rule
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
