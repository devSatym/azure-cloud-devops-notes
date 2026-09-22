<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Firewall — AZ-104 Revision Notes

---

## 1. What is Azure Firewall?

- **Cloud-native, managed network security service** — protects Azure VNet resources
- **Stateful firewall as a service** — built-in HA, unrestricted cloud scalability
- Works at **Layer 3–7** (network + application level filtering)
- Centrally create, enforce, and log **network and application** connectivity policies
- Uses a **static public IP** for outbound traffic (source NAT)
- Fully integrated with **Azure Monitor** for logging and analytics
- Deployed into a **dedicated subnet** named **AzureFirewallSubnet**

---

## 2. Key Components

| Component | Purpose |
|---|---|
| **Azure Firewall** | The firewall resource itself — deployed in a hub VNet |
| **AzureFirewallSubnet** | Dedicated subnet (MUST be named `AzureFirewallSubnet`, minimum /26) |
| **Firewall Policy** | Container for rule collections (NAT, Network, Application) — can be hierarchical |
| **Public IP** | Static public IP for inbound DNAT and outbound SNAT |
| **Rule Collection** | Group of rules of the same type with a priority and action |
| **Rule Collection Group** | Group of rule collections (in Firewall Policy) |
| **Threat Intelligence** | Built-in feed to alert/deny traffic from/to known malicious IPs/domains |
| **IDPS** | Intrusion Detection and Prevention System (Premium SKU) |
| **TLS Inspection** | Decrypt/inspect outbound HTTPS traffic (Premium SKU) |

---

## 3. SKU Comparison

| Feature | **Basic** | **Standard** | **Premium** |
|---|---|---|---|
| **Throughput** | 250 Mbps | 30 Gbps | 100 Gbps |
| **Network rules** | ✅ | ✅ | ✅ |
| **Application rules (FQDN filtering)** | ✅ | ✅ | ✅ |
| **NAT (DNAT) rules** | ✅ | ✅ | ✅ |
| **Threat Intelligence** | Alert only | Alert & Deny | Alert & Deny |
| **IDPS** | ❌ | ❌ | ✅ |
| **TLS Inspection** | ❌ | ❌ | ✅ |
| **URL Filtering** | ❌ | ✅ (FQDN-based) | ✅ (full URL path) |
| **Web Categories** | ❌ | ✅ (FQDN-based) | ✅ (full URL) |
| **DNS Proxy** | ❌ | ✅ | ✅ |
| **Custom DNS** | ❌ | ✅ | ✅ |
| **FQDN in Network Rules** | ❌ | ✅ | ✅ |
| **Forced Tunneling** | ❌ | ✅ | ✅ |
| **Multiple Public IPs** | ❌ | ✅ (up to 250) | ✅ (up to 250) |
| **Availability Zones** | ❌ | ✅ | ✅ |
| **Firewall Policy** | ✅ | ✅ | ✅ |
| **Certifications** | ❌ | ✅ | ✅ |
| **Best For** | SMB / low throughput | Most production workloads | High-security / regulated |

> ⚠️ **EXAM TIP:** **Standard** = most common exam answer. **Premium** = IDPS + TLS Inspection. **Basic** = limited features, 250 Mbps, no zones, single public IP, threat intel alert-only.

> ⚠️ **EXAM TIP:** **IDPS and TLS Inspection = Premium only**. This is heavily tested.

---

## 4. Rule Types & Processing Order

### Rule Types

| Rule Type | Layer | Purpose | Example |
|---|---|---|---|
| **NAT Rules (DNAT)** | L3/L4 | Translate inbound traffic to private IPs (port forwarding) | Internet:3389 → VM:3389 |
| **Network Rules** | L3/L4 | Allow/deny based on IP, port, protocol | Allow 10.0.0.0/24 → Any:443 TCP |
| **Application Rules** | L7 | Allow/deny based on FQDN, URL, web category | Allow *.microsoft.com:443 |

### Processing Order (CRITICAL)

```
1. NAT Rules (DNAT) — processed FIRST
2. Network Rules — processed SECOND
3. Application Rules — processed LAST
```

- Within each type: **lower priority number = higher priority = processed first**
- If a **Network Rule** matches → traffic is allowed/denied; Application Rules NOT evaluated
- **Deny by default** — if no rule matches, traffic is blocked

> ⚠️ **EXAM TIP:** Processing order: **NAT → Network → Application**. This is one of the MOST tested concepts. Network rules are evaluated BEFORE application rules. If a network rule matches, application rules are SKIPPED.

> ⚠️ **EXAM TIP:** **Default behavior = DENY ALL**. Must explicitly create rules to allow traffic.

---

## 5. NAT Rules (DNAT)

- **Destination NAT** — translates inbound traffic from public IP to private backend IP
- Used for publishing internal services to the internet (e.g., RDP, web server)
- Can translate **destination IP AND port**

### Portal Path — Create NAT Rule
```
Firewall Policy → Settings → DNAT rules →
+ Add a rule collection → Name, Priority, Action (Dnat) →
+ Add rule → Name, Source (IP/range), Protocol (TCP/UDP),
Destination (Firewall Public IP), Destination Port,
Translated address (backend private IP), Translated port → Add
```

> ⚠️ **EXAM TIP:** DNAT translates the **destination** (public → private). SNAT translates the **source** (private → public, automatic for outbound). DNAT rules action is always **Dnat** (no Allow/Deny).

---

## 6. Network Rules

- Filter traffic based on **source IP, destination IP, port, protocol** (L3/L4)
- Protocols: TCP, UDP, ICMP, Any
- Can use **IP addresses, IP groups, service tags, FQDNs** (Standard/Premium)
- Action: **Allow** or **Deny**

### Portal Path — Create Network Rule
```
Firewall Policy → Settings → Network rules →
+ Add a rule collection → Name, Priority, Action (Allow/Deny) →
+ Add rule → Name, Source, Protocol, Destination Ports,
Destination (IP/FQDN/Service Tag) → Add
```

> ⚠️ **EXAM TIP:** Network rules can use **Service Tags** (e.g., `AzureCloud`, `Storage`, `Sql`) as destination — no need to manage IP ranges manually.

---

## 7. Application Rules

- Filter **outbound** HTTP/HTTPS/MSSQL traffic based on **FQDNs, URLs, web categories**
- Action: **Allow** or **Deny**
- Supports wildcard FQDNs (e.g., `*.google.com`)
- **FQDN Tags** — predefined groups (e.g., `WindowsUpdate`, `AzureBackup`, `AppServiceEnvironment`)
- Can only be used for **outbound and east-west** traffic (NOT inbound)

### Portal Path — Create Application Rule
```
Firewall Policy → Settings → Application rules →
+ Add a rule collection → Name, Priority, Action (Allow/Deny) →
+ Add rule → Name, Source, Protocol:Port (http:80, https:443),
Target FQDNs / FQDN Tags / Web Categories → Add
```

> ⚠️ **EXAM TIP:** **FQDN Tags** are managed by Microsoft and auto-updated (e.g., `WindowsUpdate` includes all Windows Update URLs). You cannot create custom FQDN tags.

> ⚠️ **EXAM TIP:** Application rules work for **outbound** traffic only. For **inbound** → use DNAT rules.

---

## 8. Firewall Policy (Recommended)

- **Recommended** way to manage Azure Firewall rules (replaces classic rules)
- Can be **shared across multiple firewalls**
- Supports **hierarchical policies** (parent → child inheritance)
  - **Base policy** (parent): org-wide rules
  - **Child policy**: workload-specific rules that inherit parent rules
- Contains: Rule Collection Groups → Rule Collections → Rules
- Created as a **standalone resource** (separate from firewall)

### Hierarchy & Inheritance

| Level | Scope |
|---|---|
| **Base (Parent) Policy** | Organization/global rules (e.g., block malicious IPs) |
| **Child Policy** | Team/app-specific rules (inherits parent rules) |

- Child rules are evaluated **AFTER** parent rules by default
- Higher-priority child rules can override parent (within same rule type)

### Portal Path — Create Firewall Policy
```
Home → + Create a resource → Firewall Policy → Create →
Name, Region, Subscription, RG →
Parent policy (optional): Select base policy →
DNS settings, TLS inspection, IDPS (Premium) → Create
```

> ⚠️ **EXAM TIP:** Firewall Policy = **standalone resource**, can be reused across firewalls and regions. Classic rules (on firewall directly) cannot be shared.

---

## 9. Threat Intelligence

- Filters traffic based on **Microsoft Threat Intelligence feed**
- Known malicious IPs, FQDNs, and URLs
- Modes:
  | Mode | Behavior |
  |---|---|
  | **Off** | Disabled |
  | **Alert only** | Log threats but don't block (Basic SKU default) |
  | **Alert and deny** | Log AND block malicious traffic (Standard/Premium default) |

### Portal Path
```
Firewall Policy → Settings → Threat intelligence →
Mode: Off / Alert only / Alert and deny →
Allowlist IPs / FQDNs (false positive exceptions) → Save
```

> ⚠️ **EXAM TIP:** Threat intelligence is **built-in and free** (no extra cost). Default mode: **Alert and deny** (Standard/Premium), **Alert only** (Basic).

---

## 10. DNS Settings

### DNS Proxy
- Firewall acts as **DNS proxy** for client VMs
- VMs point to firewall private IP as DNS server
- Firewall resolves via configured upstream DNS or Azure DNS
- **Required** for FQDN filtering in network rules

### Custom DNS
- Configure custom DNS servers instead of Azure DNS
- Useful when using private DNS or on-prem DNS

### Portal Path
```
Firewall Policy → Settings → DNS →
DNS Proxy: Enabled →
DNS servers: Default (Azure DNS) / Custom (enter IPs) → Save
```

> ⚠️ **EXAM TIP:** **DNS Proxy must be enabled** for FQDN filtering in network rules to work. Without it, firewall cannot resolve FQDNs in network rules.

---

## 11. AzureFirewallSubnet

- **MUST** be named **`AzureFirewallSubnet`** — no other name
- Minimum size: **/26** (64 addresses)
- Recommended: /26
- No NSGs should be applied
- For forced tunneling: also need **AzureFirewallManagementSubnet** (/26)

> ⚠️ **EXAM TIP:** `AzureFirewallSubnet` = mandatory name, minimum **/26**. Compare: `GatewaySubnet` (VPN/ER) = minimum /29 recommend /27. `AzureFirewallManagementSubnet` = needed ONLY for forced tunneling.

---

## 12. Forced Tunneling

- Route firewall's **management traffic** differently from user traffic
- Requires a **separate management subnet**: `AzureFirewallManagementSubnet` (/26)
- Separate **management public IP** (in addition to data public IP)
- User traffic can be tunneled to on-prem via UDR (0.0.0.0/0 → on-prem)
- Management traffic always goes direct to Azure (never tunneled)
- **Standard & Premium only** (NOT Basic)

### Portal Path
```
Create Firewall → Firewall Management → Enable Forced Tunneling →
Management subnet: AzureFirewallManagementSubnet →
Management public IP: Select/Create → Create
```

> ⚠️ **EXAM TIP:** Forced tunneling = two subnets (AzureFirewallSubnet + AzureFirewallManagementSubnet) + two public IPs. Management traffic always goes DIRECT to Azure.

---

## 13. Multiple Public IPs

- Standard/Premium supports up to **250 public IPs**
- More public IPs = more **SNAT ports** (each IP provides ~2,496 ports)
- Helps prevent **SNAT port exhaustion** for outbound connections
- Each public IP can be used for different DNAT rules
- Basic SKU: **single public IP only**

### Portal Path
```
Azure Firewall → Settings → Public IP configuration →
+ Add public IP address → Select/Create → Save
```

> ⚠️ **EXAM TIP:** Each public IP adds **~2,496 SNAT ports**. Max **250 public IPs** per firewall (Standard/Premium). SNAT exhaustion → add more public IPs.

---

## 14. IP Groups

- Reusable object containing **IP addresses, ranges, or subnets**
- Use in rules instead of typing IPs repeatedly
- Can be used across **multiple firewalls and policies**
- Max **5,000 IP addresses/prefixes** per IP Group
- Max **100 IP Groups** per firewall

### Portal Path
```
Home → + Create a resource → IP Group → Create →
Name, Region, RG → IP addresses (add CIDRs) → Create
```

> ⚠️ **EXAM TIP:** IP Groups = reusable, shared across policies. Simplifies rule management when same IP sets used in multiple rules.

---

## 15. Azure Firewall vs NSG

| Feature | **Azure Firewall** | **NSG** |
|---|---|---|
| **Type** | Managed stateful firewall service | Stateful network filter |
| **Layer** | L3–L7 | L3–L4 only |
| **Scope** | Centralized (hub VNet) | Per-subnet or per-NIC |
| **FQDN filtering** | ✅ | ❌ |
| **Application rules** | ✅ (L7 HTTP/HTTPS) | ❌ |
| **Threat intelligence** | ✅ | ❌ |
| **NAT (DNAT)** | ✅ | ❌ |
| **IDPS / TLS inspection** | ✅ (Premium) | ❌ |
| **Logging** | Azure Monitor / Log Analytics | NSG Flow Logs |
| **Cost** | Paid (per hour + data) | Free |
| **Use case** | Centralized network security | Micro-segmentation per subnet/NIC |

> ⚠️ **EXAM TIP:** NSG = **free, per-subnet/NIC, L3-L4 only**. Azure Firewall = **paid, centralized, L3-L7, FQDN filtering, threat intel**. They are **complementary** — use both together.

---

## 16. Azure Firewall vs Azure Firewall Manager

| Feature | Description |
|---|---|
| **Azure Firewall Manager** | Central management for multiple Azure Firewalls across VNets and regions |
| **Secured Virtual Hub** | Azure Firewall deployed inside a Virtual WAN hub (managed by Firewall Manager) |
| **Hub VNet** | Azure Firewall deployed in a regular hub VNet (manage directly or via Manager) |
| **Security Partner Providers** | Route internet traffic through 3rd-party SECaaS (e.g., Zscaler, Check Point) |

> ⚠️ **EXAM TIP:** **Firewall Manager** = for managing firewalls at scale across subscriptions. **Secured virtual hub** = firewall inside Virtual WAN.

---

## 17. Availability Zones (Standard/Premium)

- Azure Firewall can be deployed across **multiple Availability Zones**
- **Zone-redundant** (default for Standard/Premium) — spans all zones for HA
- Can also be pinned to a **specific zone**
- Selected at **deployment time** — cannot change after
- SLA: **99.95% (single zone)**, **99.99% (zone-redundant)**

### Portal Path
```
Create Firewall → Availability zone:
Zone-redundant (1, 2, 3) / Specific zone / None
```

> ⚠️ **EXAM TIP:** Zone-redundant Azure Firewall SLA = **99.99%**. Single zone or no zone = **99.95%**.

---

## 18. Hub-and-Spoke Architecture

- Azure Firewall typically deployed in **hub VNet**
- Spoke VNets connect to hub via **VNet Peering**
- UDRs on spoke subnets route traffic through firewall: `0.0.0.0/0 → Firewall private IP`
- Firewall inspects **all traffic**: spoke-to-spoke, spoke-to-internet, spoke-to-on-prem

```
Spoke VNet 1 ──┐
                ├──► Hub VNet (Azure Firewall) ──► Internet / On-prem
Spoke VNet 2 ──┘
```

### UDR on Spoke Subnets
```
Route Table → Routes → + Add →
Name: ToFirewall, Prefix: 0.0.0.0/0,
Next hop type: Virtual appliance,
Next hop address: Firewall private IP (e.g., 10.0.1.4) → Add
```

> ⚠️ **EXAM TIP:** Next hop for UDR to Azure Firewall = **Virtual appliance** (NOT Virtual network gateway). Enter the **firewall's private IP address**.

---

## 19. Security & RBAC

### RBAC Roles

| Role | Permissions |
|---|---|
| **Network Contributor** | Full management of firewall and rules |
| **Contributor** | Full access to all resources |
| **Security Admin** | Manage firewall policies and security resources |
| **Reader** | View-only |

| Action | Minimum Role |
|---|---|
| Create / delete firewall | Network Contributor |
| Create / modify policy | Network Contributor |
| Create / modify rules | Network Contributor |
| View firewall config | Reader |

---

## 20. Monitoring & Diagnostics

### Diagnostic Logs

| Log Category | Content |
|---|---|
| **AzureFirewallApplicationRule** | Application rule hits (allowed/denied FQDNs) |
| **AzureFirewallNetworkRule** | Network rule hits (allowed/denied IPs/ports) |
| **AzureFirewallDnsProxy** | DNS proxy requests |
| **AzureFirewallThreatIntel** | Threat intelligence matches |
| **AZFWNatRule** | NAT rule processing |
| **AZFWIdpsSignature** | IDPS signature matches (Premium) |

### Portal Path — Enable Diagnostics
```
Azure Firewall → Monitoring → Diagnostic settings →
+ Add → Select logs: ApplicationRule, NetworkRule, ThreatIntel →
Destination: Log Analytics / Storage / Event Hub → Save
```

### Key Metrics

| Metric | Description |
|---|---|
| **Data processed** | Total data through firewall (bytes) |
| **Throughput** | Current data throughput |
| **Application rules hit count** | Application rule matches |
| **Network rules hit count** | Network rule matches |
| **SNAT port utilization** | % of SNAT ports in use |
| **Firewall health state** | Overall health (%) |

### Portal Path — Configure Alerts
```
Azure Firewall → Monitoring → Alerts → + New alert rule →
Signal: Firewall health state / SNAT port utilization →
Condition: e.g., health < 100% → Action Group → Create
```

### Azure Firewall Workbook
```
Azure Firewall → Monitoring → Workbooks →
Azure Firewall Workbook (built-in) → View
```

> ⚠️ **EXAM TIP:** **SNAT port utilization** > 80% = risk of exhaustion. Solution: add more public IPs. **Firewall health state** < 100% = investigate immediately.

---

## 21. Pricing Key Points

| Component | Cost |
|---|---|
| **Firewall deployment (per hour)** | Charged per hour the firewall exists |
| **Data processed (per GB)** | Charged per GB processed through firewall |
| **Basic SKU** | Lower hourly rate |
| **Standard SKU** | Medium hourly rate |
| **Premium SKU** | Highest hourly rate |
| **Firewall Policy** | Free (Standard), per-policy charge (Premium) |
| **Public IPs** | Standard Public IP charges |

### Cost Optimization
- **Stop/Deallocate** firewall during off-hours → `Stop-AzFirewall` (no charges when deallocated)
- No data processing charges when stopped
- Retains configuration — just loses public IP allocation (unless static)

> ⚠️ **EXAM TIP:** Azure Firewall CAN be **stopped/deallocated** (unlike VPN Gateway / ER Gateway). Use `Stop-AzFirewall` / `Start-AzFirewall` to save costs. Firewall charges per hour + per GB.

---

## 22. CLI / PowerShell Commands

| Action | Command |
|---|---|
| Create firewall | `az network firewall create -g <rg> -n <name> --vnet-name <vnet> --sku AZFW_VNet --tier Standard` |
| Stop (deallocate) | `az network firewall update -g <rg> -n <name> --no-wait` / PowerShell: `Stop-AzFirewall` |
| Start | PowerShell: `Start-AzFirewall -Name <name> -ResourceGroupName <rg> -VirtualNetwork <vnet> -PublicIPAddress <pip>` |
| Get firewall | `az network firewall show -g <rg> -n <name>` |
| Create policy | `az network firewall policy create -g <rg> -n <policy> --sku Standard` |
| Add network rule | `az network firewall policy rule-collection-group collection add-filter-collection --policy-name <policy> -g <rg> --rcg-name <rcg> --name <coll> --collection-priority 100 --action Allow --rule-type NetworkRule ...` |

---

## 23. Limitations & Constraints

| Constraint | Limit |
|---|---|
| AzureFirewallSubnet minimum size | **/26** |
| AzureFirewallSubnet name | Must be **`AzureFirewallSubnet`** |
| Max public IPs | **250** (Standard/Premium), **1** (Basic) |
| Max DNAT rules | **250** |
| Max network rules | **10,000** |
| Max application rules | **10,000** |
| Max IP Groups per firewall | **100** |
| Max IPs per IP Group | **5,000** |
| SNAT ports per public IP | **~2,496** |
| Throughput (Basic) | **250 Mbps** |
| Throughput (Standard) | **30 Gbps** |
| Throughput (Premium) | **100 Gbps** |

---

## 24. Quick-Fire Exam Points ⚡

1. Azure Firewall = **managed, stateful, L3–L7 firewall** as a service
2. Deployed in **AzureFirewallSubnet** (mandatory name, minimum **/26**)
3. Rule processing order: **NAT → Network → Application** (most tested point!)
4. **Default = deny all** — must create explicit allow rules
5. **Standard** = FQDN filtering, threat intel, DNS proxy, multi-IP, zones
6. **Premium** = Standard + **IDPS + TLS Inspection + full URL filtering**
7. **Basic** = 250 Mbps, single public IP, no zones, no IDPS, threat intel alert-only
8. **FQDN Tags** = Microsoft-managed sets (e.g., WindowsUpdate) — used in Application rules
9. **Service Tags** = used in Network rules for Azure service destinations
10. Application rules = **outbound only** (L7 FQDN/URL). Inbound = use **DNAT rules**
11. NAT (DNAT) rules: translate inbound public IP:port → private IP:port
12. **Network rules** evaluated BEFORE application rules — if network rule matches, app rules skipped
13. **DNS Proxy must be enabled** for FQDN filtering in network rules
14. **Threat intelligence**: Alert & deny (Standard/Premium default), Alert only (Basic default)
15. Firewall can be **stopped/deallocated** to save costs: `Stop-AzFirewall` / `Start-AzFirewall`
16. UDR to route through firewall: 0.0.0.0/0 → **Virtual appliance** → firewall private IP
17. Up to **250 public IPs** (Standard/Premium), each adds ~2,496 SNAT ports
18. **Forced tunneling** needs **AzureFirewallManagementSubnet** (/26) + management public IP
19. **Firewall Policy** = standalone, shareable, supports parent-child hierarchy
20. SLA: **99.99% (zone-redundant)**, **99.95% (single/no zone)**
21. Azure Firewall vs NSG: Firewall = centralized L3-L7; NSG = per-subnet L3-L4 (free)
22. Hub-and-spoke: Firewall in hub, UDRs on spokes route all traffic through firewall
23. SNAT port utilization > 80% → add more public IPs to avoid exhaustion
24. Max 250 DNAT rules, 10,000 network rules, 10,000 application rules
25. Hierarchical policies: parent (org-wide) → child (team-specific), child inherits parent rules
26. **Network Contributor** = minimum RBAC role for firewall management
27. Diagnostic logs: ApplicationRule, NetworkRule, DnsProxy, ThreatIntel → send to Log Analytics
28. AZ Firewall CAN coexist with NSGs — they are **complementary** (defense in depth)
29. IP Groups: max 100 per firewall, max 5,000 IPs per group
30. Zone selection is at **deployment time** — cannot change later

---

## 25. Step-by-Step Configuration Mind Maps 🗺️

---

### 25.1 Create Azure Firewall (Standard)

> **Portal:** `Home → + Create a resource → Firewall → Create`

```
Create Azure Firewall (Standard)
│
├── Step 1: Basics
│   ├── Subscription, Resource Group
│   ├── Name
│   ├── Region
│   ├── Availability zone: Zone 1,2,3 (zone-redundant) / specific / none
│   │   ⚠️ Cannot change after deployment
│   │   ⚠️ Zone-redundant = 99.99% SLA
│   ├── Firewall SKU: Standard
│   ├── Firewall management: Use Firewall Policy (recommended) / Classic
│   ├── Firewall policy: Create new or select existing
│   │   ⚠️ Policy can be shared across firewalls
│   ├── Virtual network: Create new or select existing
│   │   └── Must have AzureFirewallSubnet (/26 min)
│   ├── Public IP address: Create new (Standard SKU, Static)
│   └── Forced tunneling: Disabled (default)
│       ⚠️ If enabled: needs AzureFirewallManagementSubnet + mgmt public IP
│
├── Step 2: Tags (Optional)
│
└── Step 3: Review + Create → Create
    ├── RBAC: Network Contributor
    └── ⚠️ Deployment takes ~5–10 minutes
```

---

### 25.2 Configure Firewall Policy with Rules

> **Portal:** `Firewall Policy → Settings`

```
Configure Firewall Policy
│
├── Step 1: Create / Select Firewall Policy
│   └── Firewall Policy → Overview
│
├── Step 2: Add Rule Collection Group
│   │   Portal: Firewall Policy → Settings → Rule Collection Groups →
│   │           + Add a rule collection group
│   ├── Name
│   ├── Priority: e.g., 100 (lower = processed first)
│   └── Add
│
├── Step 3: Add NAT Rule Collection (DNAT)
│   │   Portal: Rule Collection Group → + Add a rule collection
│   ├── Name, Priority, Rule collection type: DNAT
│   ├── + Add rule:
│   │   ├── Name
│   │   ├── Source: IP / IP Group (e.g., * for any)
│   │   ├── Protocol: TCP / UDP
│   │   ├── Destination: Firewall Public IP
│   │   ├── Destination Port: e.g., 3389
│   │   ├── Translated address: Backend VM private IP
│   │   └── Translated port: e.g., 3389
│   └── Add
│   ⚠️ DNAT = inbound traffic. Processed FIRST.
│
├── Step 4: Add Network Rule Collection
│   ├── Rule collection type: Network, Action: Allow/Deny
│   ├── + Add rule:
│   │   ├── Source: IP / IP Group / Service Tag
│   │   ├── Protocol: TCP / UDP / ICMP / Any
│   │   ├── Destination: IP / FQDN / Service Tag
│   │   │   ⚠️ FQDN in network rules requires DNS Proxy enabled
│   │   └── Destination Ports: e.g., 443, 80, 1433
│   └── Add
│   ⚠️ Processed SECOND (after DNAT)
│
├── Step 5: Add Application Rule Collection
│   ├── Rule collection type: Application, Action: Allow/Deny
│   ├── + Add rule:
│   │   ├── Source: IP / IP Group
│   │   ├── Protocol: http:80, https:443, mssql:1433
│   │   ├── Destination type: FQDN / FQDN Tag / Web Category
│   │   └── Destination: e.g., *.microsoft.com / WindowsUpdate
│   └── Add
│   ⚠️ Processed LAST. Outbound only.
│
└── ⚠️ Rule priority within collection: lower number = higher priority
    ⚠️ Remember: NAT → Network → Application
```

---

### 25.3 Configure Hub-and-Spoke with Firewall

> **Portal:** Multiple resources (VNets, Peering, UDR, Firewall)

```
Hub-and-Spoke with Azure Firewall
│
├── Step 1: Create Hub VNet with AzureFirewallSubnet
│   ├── Hub VNet: e.g., 10.0.0.0/16
│   └── AzureFirewallSubnet: e.g., 10.0.1.0/26
│       ⚠️ Must be named AzureFirewallSubnet, min /26
│
├── Step 2: Deploy Azure Firewall in Hub
│   └── (See Mind Map 25.1)
│   └── Note firewall Private IP (e.g., 10.0.1.4)
│
├── Step 3: Create Spoke VNets
│   ├── Spoke 1: e.g., 10.1.0.0/16
│   └── Spoke 2: e.g., 10.2.0.0/16
│
├── Step 4: Peer Hub ↔ Spokes
│   │   Portal: Hub VNet → Peerings → + Add
│   ├── Hub → Spoke 1 (enable gateway transit if using VPN/ER)
│   ├── Hub → Spoke 2
│   ├── Spoke 1 → Hub (allow forwarded traffic)
│   └── Spoke 2 → Hub (allow forwarded traffic)
│   ⚠️ Both sides of peering must be configured
│
├── Step 5: Create UDR for Spoke Subnets
│   │   Portal: Route Table → Routes → + Add
│   ├── Route: 0.0.0.0/0 → Virtual appliance → 10.0.1.4 (FW IP)
│   │   ⚠️ Next hop = "Virtual appliance" (NOT virtual network gateway)
│   ├── Associate Route Table to Spoke 1 workload subnet
│   └── Associate Route Table to Spoke 2 workload subnet
│
├── Step 6: Create Firewall Rules
│   ├── Network rules: Allow spoke-to-spoke traffic
│   ├── Network rules: Allow spoke-to-internet (if needed)
│   └── Application rules: Allow specific FQDNs
│
└── Result
    ├── All spoke traffic → firewall → inspected → forwarded/denied
    └── ⚠️ Spoke-to-spoke traffic goes through firewall (not direct)
```

---

### 25.4 Stop/Start Azure Firewall (Cost Optimization)

> **Tools:** PowerShell / CLI only (no portal button)

```
Stop / Start Azure Firewall
│
├── Stop (Deallocate) — Save costs
│   │
│   ├── PowerShell:
│   │   ├── $fw = Get-AzFirewall -Name <name> -ResourceGroupName <rg>
│   │   ├── $fw.Deallocate()
│   │   └── Set-AzFirewall -AzureFirewall $fw
│   │
│   └── Result:
│       ├── Firewall stops processing traffic
│       ├── No hourly or data charges
│       ├── Public IP released (if dynamic)
│       └── Configuration retained
│       ⚠️ No portal button — PowerShell/CLI only
│
├── Start (Allocate)
│   │
│   ├── PowerShell:
│   │   ├── $fw = Get-AzFirewall -Name <name> -ResourceGroupName <rg>
│   │   ├── $pip = Get-AzPublicIpAddress -Name <pip> -ResourceGroupName <rg>
│   │   ├── $vnet = Get-AzVirtualNetwork -Name <vnet> -ResourceGroupName <rg>
│   │   ├── $fw.Allocate($vnet, $pip)
│   │   └── Set-AzFirewall -AzureFirewall $fw
│   │
│   └── Result:
│       ├── Firewall starts processing traffic
│       ├── Public IP re-assigned
│       └── ⚠️ May get different public IP if not using static
│
└── ⚠️ Use for dev/test environments to save costs
    ⚠️ UDRs still exist — traffic drops while firewall is stopped
```

---

### 25.5 Monitor Azure Firewall

> **Portal:** `Azure Firewall → Monitoring`

```
Monitor Azure Firewall
│
├── Enable Diagnostic Logs
│   │   Portal: Azure Firewall → Monitoring → Diagnostic settings
│   ├── + Add diagnostic setting
│   ├── Logs:
│   │   ├── AzureFirewallApplicationRule → FQDN matches
│   │   ├── AzureFirewallNetworkRule → IP/port matches
│   │   ├── AzureFirewallDnsProxy → DNS queries
│   │   └── AzureFirewallThreatIntel → threat matches
│   ├── Destination: Log Analytics (recommended) / Storage
│   └── Save
│
├── View Metrics
│   │   Portal: Azure Firewall → Monitoring → Metrics
│   ├── Firewall health state (%) → overall health
│   ├── Data processed (bytes) → throughput
│   ├── SNAT port utilization (%) → outbound health
│   │   ⚠️ > 80% = risk of port exhaustion
│   ├── Application rules hit count
│   ├── Network rules hit count
│   └── Throughput (bits/sec)
│
├── Azure Firewall Workbook
│   │   Portal: Azure Firewall → Monitoring → Workbooks
│   ├── Select "Azure Firewall Workbook"
│   └── Visual dashboard: rule hits, threats, data processed
│
├── Configure Alerts
│   │   Portal: Azure Firewall → Monitoring → Alerts
│   ├── + New alert rule
│   ├── Signal: SNAT port utilization > 80%
│   ├── Signal: Firewall health state < 100%
│   ├── Action Group: Email / SMS
│   └── Create
│
└── Log Analytics Queries
    ├── Application rule logs:
    │   AzureDiagnostics | where Category == "AzureFirewallApplicationRule"
    ├── Network rule logs:
    │   AzureDiagnostics | where Category == "AzureFirewallNetworkRule"
    └── Threat intel logs:
        AzureDiagnostics | where Category == "AzureFirewallThreatIntel"
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
