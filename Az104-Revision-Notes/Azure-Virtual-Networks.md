<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Virtual Networks (VNet) — AZ-104 Revision Notes

---

## 1. What is Azure VNet?

- **Logically isolated network** in Azure for deploying and managing resources
- Scoped to a **single region** and a **single subscription**
- Enables Azure resources to communicate with each other, the internet, and on-premises networks
- Provides **isolation, segmentation, filtering, and routing** of network traffic
- Free of charge — no cost for the VNet itself
- Foundation of Azure networking — almost every resource connects to a VNet
- Supports **IPv4** and **dual-stack (IPv4 + IPv6)**
- Can have **multiple address spaces** added
- Supports **VNet encryption** (encrypt traffic between VMs in same VNet)

> ⚠️ EXAM TIP: A VNet is **region-specific** — it CANNOT span multiple regions. Use **VNet Peering** to connect VNets across regions.

---

## 2. Key Components / Types

| Component | Description |
|---|---|
| **Address Space** | CIDR block (e.g., 10.0.0.0/16) defining IP range |
| **Subnets** | Subdivisions of address space for resource grouping |
| **NSG** | Network Security Groups for traffic filtering |
| **Route Tables (UDR)** | User-defined routes to control traffic flow |
| **VNet Peering** | Connect VNets (same or cross-region) |
| **Service Endpoints** | Extend VNet identity to Azure services |
| **Private Endpoints** | Private IP for Azure PaaS services inside VNet |
| **NAT Gateway** | Outbound internet connectivity for subnets |
| **Bastion** | Secure RDP/SSH without public IP |
| **VPN Gateway** | Encrypted tunnel to on-prem / other VNets |
| **DNS Settings** | Custom or Azure-provided DNS |

---

## 3. Address Space

- Uses **private IP ranges** (RFC 1918): `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`
- Also supports non-RFC 1918 ranges (but NOT `224.0.0.0/4` multicast, `255.255.255.255/32`, `127.0.0.0/8`, `169.254.0.0/16`, `168.63.129.16/32`)
- Multiple address spaces can be added to a single VNet
- Address spaces **must NOT overlap** with other connected VNets or on-prem networks
- **Can add/remove** address spaces after VNet creation (no downtime if no subnets in the range)
- Smallest subnet supported: **/29** (3 usable IPs)
- Largest subnet supported: **/2** (but **/8** is practical max for RFC 1918)
- Smallest VNet: **/29**
- **Cannot** use: `224.0.0.0/4` (multicast), `255.255.255.255/32`, `127.0.0.0/8` (loopback), `169.254.0.0/16` (link-local), `168.63.129.16/32` (internal DNS)

> ⚠️ EXAM TIP: Azure **reserves 5 IPs** in each subnet:
> | Reserved IP | Purpose |
> |---|---|
> | x.x.x.**0** | Network address |
> | x.x.x.**1** | Default gateway |
> | x.x.x.**2** | DNS mapping |
> | x.x.x.**3** | DNS mapping |
> | x.x.x.**last** | Broadcast |
>
> Example: /24 subnet = 256 - 5 = **251 usable IPs**
> Example: /29 subnet = 8 - 5 = **3 usable IPs**

**Portal Path:** `Virtual Networks → + Create → Basics → IP Addresses tab`

---

## 4. Subnets

- A **range within the VNet address space**
- Resources are deployed into subnets
- Each subnet must have a **unique, non-overlapping** CIDR range within the VNet
- Subnet size **can be changed** after creation (if no conflicting resources)
- **Cannot** be named: `GatewaySubnet`, `AzureFirewallSubnet`, `AzureBastionSubnet`, `RouteServerSubnet` for general use — these are **reserved names**

### Special / Reserved Subnets

| Subnet Name | Purpose | Required Size |
|---|---|---|
| **GatewaySubnet** | VPN / ExpressRoute Gateway | /27 or larger recommended |
| **AzureBastionSubnet** | Azure Bastion | /26 or larger |
| **AzureFirewallSubnet** | Azure Firewall | /26 |
| **AzureFirewallManagementSubnet** | Firewall forced tunneling | /26 |
| **RouteServerSubnet** | Azure Route Server | /27 |

> ⚠️ EXAM TIP: **GatewaySubnet** MUST be named exactly `GatewaySubnet` — no other name works for VPN/ExpressRoute gateways.

### Subnet Delegation

- Assigns a subnet for **exclusive use** by a specific Azure service
- Only **one delegation** per subnet
- Examples: `Microsoft.Web/serverFarms` (App Service), `Microsoft.Sql/managedInstances`, `Microsoft.ContainerInstance/containerGroups`

**Portal Path:** `Virtual Network → Subnets → + Subnet`
**Portal Path (Delegation):** `Virtual Network → Subnets → Select subnet → Subnet delegation dropdown`

---

## 5. VNet Peering

### Types

| Feature | Regional (VNet Peering) | Global (Global VNet Peering) |
|---|---|---|
| **Scope** | Same region | Cross-region |
| **Latency** | Azure backbone, low latency | Azure backbone, slightly higher |
| **Cost** | Ingress + Egress charges | Higher than regional |
| **Gateway Transit** | ✅ Supported | ✅ Supported |
| **Basic LB access** | ✅ Yes | ❌ No (Standard LB only) |

### Key Properties

- **Non-transitive** — if VNet A ↔ B and B ↔ C, A ≠ C (unless explicitly peered or using hub-spoke with gateway transit)
- **Two peering links** created (one in each VNet direction)
- Both links must be `Connected` for peering to work
- Address spaces **must NOT overlap**
- Can peer VNets across **different subscriptions** and **different Azure AD tenants**
- Resources in peered VNets can communicate via **private IPs**
- Peering is **not transitive** by default

### Peering Settings

| Setting | Description |
|---|---|
| **Allow virtual network access** | Enable communication between VNets |
| **Allow forwarded traffic** | Accept traffic not originating from peered VNet |
| **Allow gateway transit** | Let peered VNet use your VPN gateway |
| **Use remote gateways** | Use the other VNet's gateway (mutually exclusive with "Allow gateway transit") |

> ⚠️ EXAM TIP: You **cannot** enable both `Allow gateway transit` and `Use remote gateways` on the **same VNet**. One VNet has the gateway (enables transit), the other uses it (uses remote).

> ⚠️ EXAM TIP: Address space changes — you can now update address space on a peered VNet **without deleting** the peering (feature: address space sync).

**Portal Path:** `Virtual Network → Peerings → + Add`

---

## 6. DNS in VNets

### Options

| Option | Description |
|---|---|
| **Azure-provided DNS** | Default, automatic, no config needed. Resolves VMs in same VNet only |
| **Custom DNS** | Point to your own DNS server IPs |
| **Azure DNS Private Zones** | Name resolution for VNets using custom domain names |

### Azure DNS Private Zones

- Provides DNS resolution **within and across** VNets
- Domain zone (e.g., `contoso.internal`) hosted in Azure
- **VNet Links** connect Private DNS Zone to VNets
  - **Auto-registration**: VMs in linked VNet auto-register DNS records (A records)
  - Max **1 registration VNet** link per Private DNS Zone per VNet
  - A VNet can be linked as **registration** to only **1** Private DNS Zone
  - A VNet can be linked as **resolution** to up to **1000** Private DNS Zones
- Max 25,000 record sets per zone

> ⚠️ EXAM TIP: A VNet can have **auto-registration** enabled for only **1** Private DNS Zone, but can link for **resolution** to many.

**Portal Path (VNet DNS):** `Virtual Network → DNS servers → Custom / Default`
**Portal Path (Private DNS Zone):** `Private DNS zones → + Create → Virtual network links → + Add`

---

## 7. Network Security Groups (NSGs)

- **Stateful** packet filter — if inbound allowed, return traffic is auto-allowed
- Applied to **Subnet** or **NIC** level (or both)
- Contains **inbound** and **outbound** security rules
- Evaluated by **priority** (lowest number = highest priority)
- Priority range: **100 – 4096**
- **Default rules** cannot be deleted but can be overridden with higher priority

### Default Rules (Cannot Delete)

| Rule | Direction | Priority | Action |
|---|---|---|---|
| AllowVNetInBound | Inbound | 65000 | Allow |
| AllowAzureLoadBalancerInBound | Inbound | 65001 | Allow |
| DenyAllInBound | Inbound | 65500 | Deny |
| AllowVNetOutBound | Outbound | 65000 | Allow |
| AllowInternetOutBound | Outbound | 65001 | Allow |
| DenyAllOutBound | Outbound | 65500 | Deny |

### NSG Rule Properties

| Property | Description |
|---|---|
| **Priority** | 100-4096, lower = evaluated first |
| **Source** | IP, CIDR, Service Tag, or ASG |
| **Source Port** | Single, range, or `*` |
| **Destination** | IP, CIDR, Service Tag, or ASG |
| **Destination Port** | Single, range, or `*` |
| **Protocol** | TCP, UDP, ICMP, ESP, AH, or Any |
| **Action** | Allow or Deny |

### Service Tags (Common)

| Tag | Scope |
|---|---|
| `VirtualNetwork` | VNet + peered VNets + VPN-connected |
| `AzureLoadBalancer` | Azure LB health probes |
| `Internet` | All public IPs |
| `Storage` | Azure Storage |
| `Sql` | Azure SQL |
| `AzureActiveDirectory` | Azure AD |
| `AzureMonitor` | Monitor, Log Analytics, App Insights |

### Application Security Groups (ASGs)

- Group VMs logically (e.g., WebServers, DBServers)
- Use ASGs as source/destination in NSG rules instead of IPs
- All NICs in an ASG must be in the **same VNet**
- A NIC can belong to **multiple ASGs**
- Simplifies rule management at scale

> ⚠️ EXAM TIP: When NSG is applied to **both** subnet and NIC — **inbound**: subnet NSG evaluated first, then NIC NSG. **Outbound**: NIC NSG first, then subnet NSG.

> ⚠️ EXAM TIP: NSGs are **stateful** — if you allow inbound traffic, return outbound traffic is automatically allowed.

**Portal Path (Create):** `Network Security Groups → + Create`
**Portal Path (Rules):** `NSG → Inbound/Outbound security rules → + Add`
**Portal Path (Associate):** `NSG → Subnets → + Associate` or `NSG → Network interfaces → + Associate`
**Portal Path (ASG):** `Application Security Groups → + Create`

---

## 8. User-Defined Routes (UDR) / Route Tables

### System Routes (Default)

| Destination | Next Hop |
|---|---|
| VNet address space | Virtual network |
| 0.0.0.0/0 | Internet |
| 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 | None (dropped) |

### Custom Route Next Hop Types

| Next Hop Type | Description |
|---|---|
| **Virtual appliance** | NVA IP (e.g., firewall) — requires IP forwarding enabled |
| **Virtual network gateway** | VPN/ExpressRoute gateway |
| **Virtual network** | Override default system route |
| **Internet** | Route to internet |
| **None** | Drop/blackhole traffic |

### Key Points

- Route table is **associated with a subnet** (not NIC)
- One route table per subnet; one route table can be associated with **multiple subnets**
- Routes are evaluated using **longest prefix match**
- If same prefix: UDR > BGP > System routes
- **0.0.0.0/0** override — commonly used to force traffic through NVA/firewall
- **BGP route propagation** can be disabled per route table

> ⚠️ EXAM TIP: To route traffic through an NVA, you need **both**: a UDR pointing to the NVA IP **AND** IP forwarding enabled on the NVA NIC.

> ⚠️ EXAM TIP: Route priority order: **UDR > BGP > System Routes**

**Portal Path:** `Route tables → + Create → Routes → + Add`
**Portal Path (Associate):** `Route table → Subnets → + Associate`
**Portal Path (BGP Propagation):** `Route table → Configuration → Propagate gateway routes → Yes/No`

---

## 9. Service Endpoints

- Extends VNet identity to Azure PaaS services
- Traffic stays on **Azure backbone** (not over internet)
- **Free** — no additional cost
- Service endpoint adds an **optimal route** in route table automatically
- Applied at the **subnet** level
- Must also configure PaaS resource firewall to allow the VNet/subnet

### Supported Services (Key)

| Service | Endpoint Name |
|---|---|
| Azure Storage | `Microsoft.Storage` |
| Azure SQL Database | `Microsoft.Sql` |
| Azure Key Vault | `Microsoft.KeyVault` |
| Azure Cosmos DB | `Microsoft.AzureCosmosDB` |
| Azure Service Bus | `Microsoft.ServiceBus` |
| Azure Event Hubs | `Microsoft.EventHub` |

### Service Endpoints vs Private Endpoints

| Feature | Service Endpoint | Private Endpoint |
|---|---|---|
| **Access type** | Public IP of service (optimized route) | Private IP inside your VNet |
| **Traffic path** | Azure backbone | Azure backbone |
| **Cost** | Free | Charged (per hour + data) |
| **On-prem access** | ❌ No (VNet only) | ✅ Yes (via VPN/ER) |
| **Cross-region** | Limited | ✅ Yes |
| **DNS** | No change needed | Requires Private DNS Zone |
| **Data exfiltration protection** | Limited | ✅ Full (specific resource) |
| **Configuration** | Subnet + PaaS firewall | NIC in subnet + DNS |

> ⚠️ EXAM TIP: **Service Endpoints** provide access from a subnet to an **entire service** (e.g., all of Azure Storage). **Private Endpoints** provide access to a **specific resource** (e.g., one storage account).

> ⚠️ EXAM TIP: Service Endpoints do **NOT** provide access from on-premises. For on-prem access to PaaS over private network, use **Private Endpoints**.

**Portal Path:** `Virtual Network → Subnets → Select subnet → Service Endpoints → Add`
**Portal Path (PaaS side):** `Storage Account → Networking → Firewalls and virtual networks → Add existing VNet`

---

## 10. Private Endpoints

- A **network interface** with a private IP from your VNet
- Connects **privately** to a specific Azure PaaS resource (e.g., a specific Storage Account, SQL DB)
- Traffic does **NOT** leave the Microsoft network
- Requires a **Private DNS Zone** for name resolution (auto-integration available)
- Works across **regions** and **subscriptions**
- Supports **Network Policies** (NSGs, UDRs) on Private Endpoints (must be enabled)

### Common Private Link Resources

| Service | Group ID |
|---|---|
| Storage - Blob | `blob` |
| Storage - File | `file` |
| SQL Database | `sqlServer` |
| Key Vault | `vault` |
| Cosmos DB | `Sql` |
| App Service | `sites` |

> ⚠️ EXAM TIP: By default, **NSGs and UDRs do NOT apply** to Private Endpoints. You must explicitly **enable network policies** on the subnet: `PrivateEndpointNetworkPolicies = Enabled`.

**Portal Path:** `Private Link Center → Private endpoints → + Create`
**Portal Path (DNS):** `Private endpoint → DNS configuration`
**Portal Path (Enable Network Policies):** `Virtual Network → Subnets → Select subnet → Private endpoint network policies → Enabled`

---

## 11. Azure Bastion

- Provides **secure RDP/SSH** to VMs directly from Azure Portal over **TLS (port 443)**
- **No public IP** required on the VM
- Protection against **port scanning** and **zero-day exploits**
- Deployed into a dedicated **AzureBastionSubnet** (/26 or larger)

### SKU Comparison

| Feature | Basic | Standard | Premium |
|---|---|---|---|
| **Connect to target VMs in same VNet** | ✅ | ✅ | ✅ |
| **Connect to peered VNets** | ❌ | ✅ | ✅ |
| **Host scaling (instances)** | 2 (fixed) | 2–50 | 2–50 |
| **Upload/download files** | ❌ | ✅ | ✅ |
| **Shareable link** | ❌ | ✅ | ✅ |
| **Connect to Linux SSH (native client)** | ❌ | ✅ | ✅ |
| **Kerberos auth** | ❌ | ❌ | ✅ |
| **Session recording** | ❌ | ❌ | ✅ |
| **Private-only deployment** | ❌ | ❌ | ✅ |

> ⚠️ EXAM TIP: Bastion subnet must be named **exactly** `AzureBastionSubnet` and be at least **/26**.

> ⚠️ EXAM TIP: **Basic SKU** cannot connect to VMs in **peered VNets** — need **Standard or higher**.

**Portal Path:** `Bastions → + Create` or `VM → Connect → Bastion`

---

## 12. NAT Gateway

- Provides **outbound-only** internet connectivity for subnets
- Uses a **static public IP** (up to 16 Public IPs or Public IP Prefixes)
- **Zone-resilient** (Standard SKU)
- Replaces default outbound connectivity on the subnet
- SNAT port exhaustion protection — up to **64,512 SNAT ports** per public IP
- **Takes precedence** over Load Balancer outbound rules and VM public IPs for outbound
- No inbound-initiated connections

> ⚠️ EXAM TIP: NAT Gateway **supersedes** all other outbound configurations (LB outbound rules, instance-level public IPs) for subnets it is attached to.

**Portal Path:** `NAT Gateways → + Create → Associate with Subnet`

---

## 13. IP Addressing

### Public IP SKUs

| Feature | Basic | Standard |
|---|---|---|
| **Allocation** | Static or Dynamic | Static only |
| **Availability Zones** | ❌ | ✅ Zone-redundant / Zonal |
| **Routing preference** | Internet only | Internet or Microsoft Network |
| **Security** | Open by default | Secure by default (need NSG to allow) |
| **Load Balancer** | Basic LB | Standard LB |
| **Redundancy** | No | Zone-redundant |
| **Retirement** | Sept 30, 2025 | Current |

> ⚠️ EXAM TIP: **Basic Public IPs are retiring** — use **Standard** for all new deployments. Standard Public IPs are **secure by default** (no inbound until NSG allows).

### Private IP Allocation

| Method | Description |
|---|---|
| **Dynamic** | Azure assigns next available IP from subnet (default) |
| **Static** | You specify a fixed IP from the subnet range |

**Portal Path:** `Public IP addresses → + Create`
**Portal Path (VM NIC):** `VM → Networking → Network interface → IP configurations`

---

## 14. VPN Gateway

### Types

| Feature | Policy-Based | Route-Based |
|---|---|---|
| **IKE version** | IKEv1 only | IKEv1 & IKEv2 |
| **S2S connections** | 1 only | Multiple (depends on SKU) |
| **P2S** | ❌ | ✅ |
| **VNet-to-VNet** | ❌ | ✅ |
| **Coexist with ExpressRoute** | ❌ | ✅ |
| **BGP** | ❌ | ✅ |
| **Recommended** | Legacy | ✅ Yes |

### Gateway SKUs (Route-Based)

| SKU | S2S Tunnels | P2S Connections | Throughput |
|---|---|---|---|
| **Basic** | 10 | 128 | 100 Mbps |
| **VpnGw1** | 30 | 250 | 650 Mbps |
| **VpnGw2** | 30 | 500 | 1 Gbps |
| **VpnGw3** | 30 | 1000 | 1.25 Gbps |
| **VpnGw4** | 100 | 5000 | 5 Gbps |
| **VpnGw5** | 100 | 10000 | 10 Gbps |

- AZ SKUs available: VpnGw1**AZ** through VpnGw5**AZ**
- Gateway deployment can take **30-45 minutes**
- Requires a **GatewaySubnet** (/27 or larger recommended)

### P2S Authentication Methods

| Method | OS Support |
|---|---|
| **Azure Certificate** | Windows, macOS, Linux |
| **RADIUS** | All (via RADIUS server) |
| **Azure AD (Entra ID)** | Windows 10+ (OpenVPN only) |

> ⚠️ EXAM TIP: You CANNOT resize between **Basic** and **VpnGw** SKUs — requires redeployment. You CAN resize within VpnGw1-5 family.

> ⚠️ EXAM TIP: VPN Gateway requires a **dedicated subnet** named `GatewaySubnet`. Only **one** VPN Gateway per VNet.

**Portal Path:** `Virtual Network Gateways → + Create`

---

## 15. ExpressRoute (Basics for AZ-104)

- **Private, dedicated** connection to Azure via connectivity provider
- Does **NOT** go over public internet
- Higher reliability, lower latency, faster speeds than VPN
- Supports bandwidths from 50 Mbps to 10 Gbps (100 Gbps with ExpressRoute Direct)
- **Peering types**: Azure Private (VNets), Microsoft (Microsoft 365, Dynamics)
- Coexists with VPN Gateway (failover scenario)

---

## 15.5 VNet Encryption

- Encrypts traffic **between VMs** within the same VNet or peered VNets
- Uses **DTLS encryption** on Azure host networking infrastructure
- Requires **Accelerated Networking** enabled on all VMs
- Supported on **General-purpose and memory-optimized** VM sizes (D/Dv2/Dv3/E/Ev3/F series and above)
- Works with VNet Peering (regional and global)
- **No performance impact** — encryption handled by the host

### Portal Path — Enable VNet Encryption
```
Virtual Network → Overview → Properties → Encryption: Enabled
```

> ⚠️ **EXAM TIP:** VNet encryption requires **Accelerated Networking** on ALL VMs in the VNet. VMs without it will have traffic **dropped**, not just unencrypted.

> ⚠️ **EXAM TIP:** VNet encryption encrypts traffic between VMs — it does NOT encrypt traffic to PaaS services or to the internet.

---

## 16. Network Watcher

- Regional service for **monitoring, diagnosing, and logging** network issues
- Auto-enabled when you create/update a VNet in a region

### Key Tools

| Tool | Purpose |
|---|---|
| **IP flow verify** | Check if traffic to/from VM is allowed or denied (identifies NSG rule) |
| **Next hop** | Shows next hop for a packet from a VM |
| **Connection troubleshoot** | Test connectivity between source and destination |
| **NSG flow logs** | Log traffic flowing through NSGs (stored in Storage Account) |
| **Topology** | Visual map of VNet resources |
| **Packet capture** | Capture packets on a VM NIC |
| **Connection monitor** | Continuous monitoring of connectivity |
| **VPN troubleshoot** | Diagnose VPN gateway issues |
| **NSG diagnostics** | Check effective security rules |

> ⚠️ EXAM TIP: **IP Flow Verify** = tells you which NSG rule is allowing/blocking traffic. **Next Hop** = tells you where the packet goes next (useful for verifying UDR).

**Portal Path:** `Network Watcher → IP flow verify / Next hop / Topology / etc.`
**Portal Path (NSG Flow Logs):** `Network Watcher → NSG flow logs → + Create`
**Portal Path (Packet Capture):** `Network Watcher → Packet capture → + Add`

---

## 17. Security & RBAC

### Key RBAC Roles

| Role | Permissions |
|---|---|
| **Network Contributor** | Full manage VNets, NSGs, Route Tables, Gateways, etc. (no access to resources inside) |
| **Owner** | Full access + assign roles |
| **Contributor** | Full access minus role assignment |
| **Reader** | View only |

### Azure DDoS Protection

| Feature | DDoS Network Protection | DDoS IP Protection |
|---|---|---|
| **Scope** | VNet level (all resources) | Per Public IP |
| **Cost** | ~$2,944/month + overage | ~$199/month per IP |
| **Metrics/Alerts** | ✅ | ✅ |
| **Rapid Response** | ✅ | ❌ |
| **Cost Protection** | ✅ | ❌ |
| **WAF discount** | ✅ | ❌ |

**Portal Path (DDoS):** `DDoS protection plans → + Create` then `Virtual Network → DDoS protection → Enable`

> ⚠️ **EXAM TIP:** DDoS **Network** Protection = VNet-level, expensive (~$2,944/month) but covers up to **100 public IPs** and includes **cost protection** (reimbursement for scale-out costs during attack). DDoS **IP** Protection = per-IP, cheaper (~$199/IP/month), no cost protection.

> ⚠️ **EXAM TIP:** DDoS Infrastructure Protection (basic) is **automatically enabled** for all Azure resources at no cost. You only pay for Network/IP tiers.

---

## 17.5 Limitations & Constraints

| Constraint | Limit |
|---|---|
| VNets per subscription per region | **1,000** |
| Subnets per VNet | **3,000** |
| VNet peerings per VNet | **500** |
| Private IP addresses per VNet | **65,536** |
| Public IP addresses per subscription | **Varies by type** |
| NSGs per subscription | **5,000** |
| NSG rules per NSG | **1,000** |
| Route tables per subscription | **200** |
| Routes per route table | **400** |
| DNS servers per VNet | **25** |
| Private endpoints per VNet | **1,000** |
| Service endpoints per subnet | **Multiple services** |
| NICs per VM | **Depends on VM size** |
| Address spaces per VNet | **500** |
| Reserved IPs per subnet | **5** (first 4 + last) |

> ⚠️ **EXAM TIP:** Key limits to memorize: **500 peerings per VNet**, **3,000 subnets per VNet**, **1,000 NSG rules per NSG**, **5 reserved IPs per subnet**.

---

## 18. Monitoring & Alerts

- **VNet flow logs** — log traffic metadata (Network Watcher)
- **NSG flow logs** — track traffic per NSG rule
- **Azure Monitor** — Metrics for VNet Gateway (tunnel bandwidth, packets, etc.)
- **Connection Monitor** — continuous reachability tests
- **Traffic Analytics** — visual analytics on NSG flow logs (requires Log Analytics workspace)
- **Diagnostic Logs** — VPN Gateway, Bastion, Application Gateway diagnostics

**Portal Path (Traffic Analytics):** `Network Watcher → Traffic Analytics`
**Portal Path (VNet Flow Logs):** `Network Watcher → Flow logs → + Create`

---

## 19. Pricing Key Points

| Item | Cost |
|---|---|
| **VNet** | Free |
| **VNet Peering** | Ingress + Egress per GB (global > regional) |
| **VPN Gateway** | Per hour + egress data |
| **Public IP (Standard)** | Per hour |
| **Bastion** | Per hour + data transfer |
| **Private Endpoint** | Per hour + data processed |
| **NAT Gateway** | Per hour + data processed |
| **DDoS Network Protection** | ~$2,944/month |
| **Service Endpoints** | Free |
| **NSG** | Free |
| **Route Tables** | Free |

---

## 20. CLI / PowerShell Commands (Exam-Relevant)

### Azure CLI

```bash
# Create VNet
az network vnet create -g MyRG -n MyVNet --address-prefix 10.0.0.0/16 --subnet-name MySubnet --subnet-prefix 10.0.0.0/24

# Create Subnet
az network vnet subnet create -g MyRG --vnet-name MyVNet -n MySubnet2 --address-prefix 10.0.1.0/24

# Create NSG
az network nsg create -g MyRG -n MyNSG

# Create NSG Rule
az network nsg rule create -g MyRG --nsg-name MyNSG -n AllowHTTP --priority 100 --destination-port-ranges 80 --access Allow --protocol Tcp --direction Inbound

# Associate NSG to Subnet
az network vnet subnet update -g MyRG --vnet-name MyVNet -n MySubnet --network-security-group MyNSG

# Create VNet Peering
az network vnet peering create -g MyRG -n Peer1to2 --vnet-name VNet1 --remote-vnet VNet2 --allow-vnet-access

# Create Route Table
az network route-table create -g MyRG -n MyRouteTable

# Add Route
az network route-table route create -g MyRG --route-table-name MyRouteTable -n ToFirewall --address-prefix 0.0.0.0/0 --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.2.4

# Add Service Endpoint
az network vnet subnet update -g MyRG --vnet-name MyVNet -n MySubnet --service-endpoints Microsoft.Storage

# Create Public IP
az network public-ip create -g MyRG -n MyPublicIP --sku Standard --allocation-method Static
```

### PowerShell

```powershell
# Create VNet
New-AzVirtualNetwork -Name MyVNet -ResourceGroupName MyRG -Location eastus -AddressPrefix 10.0.0.0/16

# Add Subnet
Add-AzVirtualNetworkSubnetConfig -Name MySubnet -VirtualNetwork $vnet -AddressPrefix 10.0.0.0/24
$vnet | Set-AzVirtualNetwork

# Create NSG
New-AzNetworkSecurityGroup -Name MyNSG -ResourceGroupName MyRG -Location eastus

# Create NSG Rule
$rule = New-AzNetworkSecurityRuleConfig -Name AllowHTTP -Protocol Tcp -Direction Inbound -Priority 100 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 80 -Access Allow

# Create Public IP
New-AzPublicIpAddress -Name MyPublicIP -ResourceGroupName MyRG -Location eastus -Sku Standard -AllocationMethod Static
```

---

## 21. Quick-Fire Exam Points ⚡

1. VNet is **region-specific** — cannot span regions
2. Azure reserves **5 IPs** per subnet (first 4 + last)
3. Smallest subnet = **/29** (3 usable IPs)
4. NSG is **stateful** — return traffic auto-allowed
5. NSG evaluation: **Inbound** = Subnet NSG → NIC NSG | **Outbound** = NIC NSG → Subnet NSG
6. NSG default rules priority starts at **65000** — cannot be deleted
7. NSG rule priority range: **100 – 4096**
8. Peering is **non-transitive** — A↔B, B↔C does NOT mean A↔C
9. Peered VNet address spaces **must NOT overlap**
10. VNet Peering: both links must show **Connected** status
11. **Basic LB** VMs cannot be accessed over **global** peering — use **Standard LB**
12. UDR priority: **UDR > BGP > System routes**
13. NVA routing needs **UDR + IP forwarding enabled** on NVA NIC
14. Service Endpoints = free, subnet-level, entire service access
15. Private Endpoints = charged, specific resource, private IP in VNet
16. Service Endpoints do **NOT** work from on-premises
17. Private Endpoints: enable **network policies** on subnet for NSG/UDR to work
18. GatewaySubnet must be named **exactly** `GatewaySubnet`
19. Bastion subnet = `AzureBastionSubnet`, minimum **/26**
20. Basic Bastion cannot connect to **peered VNet** VMs
21. Standard Public IPs are **secure by default** — need NSG to allow inbound
22. **Basic Public IPs are retiring** (Sept 2025)
23. VPN Gateway: cannot resize between **Basic** and **VpnGw** families
24. Only **one** VPN Gateway per VNet
25. Gateway deployment takes **30-45 minutes**
26. P2S with **Azure AD auth** = Windows 10+ with OpenVPN only
27. NAT Gateway **overrides** all other outbound connectivity on its subnet
28. NAT Gateway supports up to **16 Public IPs**
29. IP Flow Verify = which **NSG rule** blocks/allows traffic
30. Next Hop = where packet is **routed** (verifies UDR)
31. VNet = **Free**, Peering = **per GB**, NSG = **Free**, UDR = **Free**
32. DNS: VNet can auto-register to only **1** Private DNS Zone
33. A VNet can link for **resolution** to up to **1000** Private DNS Zones
34. ASG NICs must be in the **same VNet**
35. Subnet delegation = **one service** per subnet
36. Max **500 peerings per VNet**, max **3,000 subnets per VNet**
37. Max **1,000 NSG rules per NSG**, max **5,000 NSGs per subscription**
38. VNet encryption requires **Accelerated Networking** on ALL VMs — without it traffic is **dropped**
39. DDoS Infrastructure Protection = **free, automatic**. DDoS Network/IP = **paid tiers**
40. DDoS Network Protection covers up to **100 public IPs** per plan + **cost protection**
41. IPv6 is **dual-stack only** — cannot have IPv6-only VNet
42. VNets per subscription per region = **1,000**
43. **Cannot move** a VNet with a VPN Gateway to a different subscription without deleting the gateway first
44. Route tables per subscription = **200**, routes per table = **400**
45. VNet encryption = **DTLS** encryption between VMs, not to PaaS/internet

---

## 22. Step-by-Step Configuration Mind Maps 🗺️

### 22.1 Create a Virtual Network

```
Azure Portal → Virtual Networks → + Create
│
├── Basics
│   ├── Subscription (select)
│   ├── Resource Group (select/create)
│   ├── Name (unique within RG)
│   └── Region (⚠️ VNet is region-specific, cannot change later)
│
├── IP Addresses
│   ├── IPv4 Address Space (e.g., 10.0.0.0/16)
│   │   └── ⚠️ Must NOT overlap with connected VNets/on-prem
│   ├── + Add subnet
│   │   ├── Subnet name
│   │   ├── Subnet address range (e.g., 10.0.0.0/24)
│   │   ├── NAT Gateway (optional)
│   │   ├── Service Endpoints (optional)
│   │   └── Subnet Delegation (optional)
│   └── + Add IPv6 (optional)
│
├── Security
│   ├── Azure Bastion (Enable/Disable)
│   ├── Azure Firewall (Enable/Disable)
│   └── Azure DDoS Protection (Enable/Disable)
│
├── Tags
│
└── Review + Create
    └── RBAC: Network Contributor or higher
```

### 22.2 Create and Associate NSG

```
Azure Portal → Network Security Groups → + Create
│
├── Basics
│   ├── Subscription
│   ├── Resource Group
│   ├── Name
│   └── Region (⚠️ Must match VNet/Subnet region)
│
├── After Creation → Add Rules
│   ├── NSG → Inbound security rules → + Add
│   │   ├── Source: Any / IP / Service Tag / ASG
│   │   ├── Source port ranges: * or specific
│   │   ├── Destination: Any / IP / Service Tag / ASG
│   │   ├── Destination port ranges: e.g., 80, 443, 3389
│   │   ├── Protocol: TCP / UDP / ICMP / Any
│   │   ├── Action: Allow / Deny
│   │   ├── Priority: 100-4096 (⚠️ lower = higher priority)
│   │   └── Name
│   └── NSG → Outbound security rules → + Add (same fields)
│
├── Associate to Subnet
│   └── NSG → Subnets → + Associate → Select VNet → Select Subnet
│
└── Associate to NIC
    └── NSG → Network interfaces → + Associate → Select NIC
    └── RBAC: Network Contributor
    └── ⚠️ Inbound: Subnet NSG → NIC NSG | Outbound: NIC NSG → Subnet NSG
```

### 22.3 Configure VNet Peering

```
Azure Portal → Virtual Network → Peerings → + Add
│
├── This virtual network
│   ├── Peering link name (e.g., VNet1-to-VNet2)
│   ├── Allow traffic to remote VNet: Enabled
│   ├── Allow traffic forwarded from remote VNet: Enabled/Disabled
│   └── Allow gateway transit: Enabled (if this VNet has gateway)
│
├── Remote virtual network
│   ├── Peering link name (e.g., VNet2-to-VNet1)
│   ├── Subscription (can be different)
│   ├── Virtual Network (select target — ⚠️ no overlapping address spaces)
│   ├── Allow traffic to remote VNet: Enabled
│   ├── Allow traffic forwarded from remote VNet: Enabled/Disabled
│   └── Use remote gateway: Enabled (if using other VNet's gateway)
│       └── ⚠️ Cannot enable both "Allow gateway transit" AND "Use remote gateways" on same VNet
│
├── Prerequisites
│   ├── Both VNets must exist
│   ├── Non-overlapping address spaces
│   └── RBAC: Network Contributor on both VNets
│
└── ⚠️ Both peering links must show "Connected" status
```

### 22.4 Create User-Defined Route (UDR)

```
Azure Portal → Route tables → + Create
│
├── Basics
│   ├── Subscription
│   ├── Resource Group
│   ├── Name
│   ├── Region (⚠️ Must match subnet region)
│   └── Propagate gateway routes: Yes/No
│       └── ⚠️ Set to No to prevent BGP route propagation
│
├── After Creation → Add Routes
│   └── Route table → Routes → + Add
│       ├── Route name
│       ├── Destination type: IP Addresses / Service Tag
│       ├── Destination IP / CIDR (e.g., 0.0.0.0/0)
│       ├── Next hop type:
│       │   ├── Virtual appliance (⚠️ requires IP forwarding on NVA NIC)
│       │   ├── Virtual network gateway
│       │   ├── Virtual network
│       │   ├── Internet
│       │   └── None (drop traffic)
│       └── Next hop address (if Virtual appliance — NVA IP)
│
├── Associate to Subnet
│   └── Route table → Subnets → + Associate → Select VNet → Select Subnet
│
└── RBAC: Network Contributor
```

### 22.5 Configure Service Endpoints

```
Azure Portal → Virtual Network → Subnets → Select Subnet
│
├── Service Endpoints → + Add
│   ├── Select Service (e.g., Microsoft.Storage, Microsoft.Sql)
│   └── Save
│
├── Then Configure PaaS Resource Firewall
│   └── e.g., Storage Account → Networking → Firewalls and virtual networks
│       ├── Allow access from: Selected networks
│       ├── + Add existing virtual network
│       │   ├── Select VNet
│       │   └── Select Subnet (with endpoint enabled)
│       └── Save
│
├── Prerequisites
│   ├── VNet and subnet must exist
│   └── RBAC: Network Contributor (VNet) + Contributor on PaaS resource
│
└── ⚠️ Service Endpoint does NOT provide on-prem access
```

### 22.6 Create Private Endpoint

```
Azure Portal → Private Link Center → Private endpoints → + Create
│
├── Basics
│   ├── Subscription, Resource Group
│   ├── Name
│   └── Region
│
├── Resource
│   ├── Connection method: Connect to resource in my directory / by resource ID
│   ├── Subscription
│   ├── Resource type (e.g., Microsoft.Storage/storageAccounts)
│   ├── Resource (select specific account)
│   └── Target sub-resource (e.g., blob, file, sqlServer)
│
├── Virtual Network
│   ├── VNet (select)
│   ├── Subnet (select)
│   ├── Private IP configuration: Dynamic / Static
│   └── ⚠️ Enable network policies on subnet for NSG/UDR support
│
├── DNS
│   ├── Integrate with private DNS zone: Yes (recommended)
│   └── Private DNS Zone: auto-created (e.g., privatelink.blob.core.windows.net)
│
├── Tags → Review + Create
│
└── RBAC: Network Contributor + appropriate role on target resource
```

### 22.7 Deploy Azure Bastion

```
Azure Portal → Bastions → + Create
│
├── Basics
│   ├── Subscription, Resource Group
│   ├── Name
│   ├── Region (⚠️ must match VNet region)
│   ├── Tier: Basic / Standard / Premium
│   │   └── ⚠️ Basic cannot connect to peered VNets
│   └── Virtual Network (must have AzureBastionSubnet /26+)
│       └── ⚠️ Subnet name MUST be "AzureBastionSubnet"
│
├── Instance count (Standard/Premium: 2-50)
│
├── Public IP
│   └── Create new Standard SKU Public IP
│
├── Tags → Review + Create
│
├── Connect to VM
│   └── VM → Connect → Bastion → Enter credentials → Connect
│
└── RBAC: On Bastion = Reader | On VM = Reader + VM Login role
```

### 22.8 Configure NAT Gateway

```
Azure Portal → NAT Gateways → + Create
│
├── Basics
│   ├── Subscription, Resource Group
│   ├── Name
│   ├── Region
│   └── Idle timeout: 4-120 minutes (default 4)
│
├── Outbound IP
│   ├── Public IP addresses (select/create — up to 16)
│   └── Public IP prefixes (optional — up to 16)
│
├── Subnet
│   └── Select VNet → Select Subnet(s)
│       └── ⚠️ NAT Gateway overrides LB outbound rules & instance-level PIPs
│
├── Tags → Review + Create
│
└── RBAC: Network Contributor
```

### 22.9 Configure DNS (Private DNS Zone)

```
Azure Portal → Private DNS zones → + Create
│
├── Basics
│   ├── Subscription, Resource Group
│   └── Name (e.g., contoso.internal)
│
├── After Creation → Link to VNet
│   └── Private DNS zone → Virtual network links → + Add
│       ├── Link name
│       ├── Subscription
│       ├── Virtual Network (select)
│       └── Enable auto registration: Yes/No
│           └── ⚠️ Auto-registration: VNet can register to only 1 Private DNS Zone
│           └── ⚠️ A VNet can link for resolution to up to 1000 zones
│
├── Add Records (optional)
│   └── Private DNS zone → + Record set
│       ├── Name, Type (A, CNAME, etc.), TTL
│       └── IP Address
│
└── RBAC: Private DNS Zone Contributor
```

### 22.10 Create VPN Gateway (Site-to-Site)

```
Azure Portal → Virtual Network Gateways → + Create
│
├── Basics
│   ├── Subscription, Resource Group
│   ├── Name
│   ├── Region
│   ├── Gateway type: VPN
│   ├── SKU: VpnGw1 – VpnGw5 (⚠️ Basic cannot resize to VpnGw)
│   ├── Generation: Gen1 / Gen2
│   ├── Virtual Network (must have GatewaySubnet)
│   │   └── ⚠️ Only ONE VPN Gateway per VNet
│   ├── Public IP (create/select Standard SKU)
│   └── Enable active-active: Yes/No
│       └── Active-active requires 2 public IPs
│
├── ⚠️ Deployment takes 30-45 minutes
│
├── After Deployment → Create S2S Connection
│   ├── Create Local Network Gateway
│   │   └── Local Network Gateways → + Create
│   │       ├── On-prem VPN device public IP
│   │       └── On-prem address spaces
│   │
│   └── Create Connection
│       └── VPN Gateway → Connections → + Add
│           ├── Connection type: Site-to-site (IPsec)
│           ├── Local Network Gateway (select)
│           ├── Shared Key (PSK)
│           └── IKE Protocol: IKEv1 / IKEv2
│
└── RBAC: Network Contributor
    └── Prerequisites: GatewaySubnet created, Public IP
```

### 22.11 Enable DDoS Protection on VNet

```
Azure Portal → DDoS protection plans → + Create
│
├── Create DDoS Protection Plan
│   ├── Subscription, Resource Group
│   ├── Name
│   └── Region
│       ⚠️ Plan covers up to 100 public IPs
│       ⚠️ Cost: ~$2,944/month (Network Protection)
│
├── Associate with VNet
│   └── Virtual Network → DDoS protection → Enable
│       ├── DDoS protection plan: Select plan
│       └── Save
│
└── RBAC: Network Contributor
    ⚠️ DDoS Infrastructure Protection is FREE and automatic
    ⚠️ Network Protection adds: metrics, alerts, cost protection, rapid response
```

### 22.12 Enable VNet Encryption

```
Azure Portal → Virtual Network → Properties
│
├── Prerequisites
│   ├── All VMs must have Accelerated Networking enabled
│   │   ⚠️ VMs without Accelerated Networking = traffic DROPPED
│   ├── Supported VM sizes only (D/E/F series and above)
│   └── Region must support VNet encryption
│
├── Enable Encryption
│   ├── Virtual Network → Overview → Properties
│   ├── Encryption: Enabled
│   └── Enforcement:
│       ├── Allow unencrypted (default — drops only if both sides support)
│       └── Drop unencrypted (strict — all traffic must be encrypted)
│           ⚠️ Strict mode = VMs without Accelerated Networking lose connectivity
│
└── RBAC: Network Contributor
```

---

*End of Azure Virtual Networks (VNet) — AZ-104 Revision Notes*


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
