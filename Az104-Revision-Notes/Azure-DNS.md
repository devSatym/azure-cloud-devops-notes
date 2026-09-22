<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure DNS — AZ-104 Revision Notes

---

## 1. What is Azure DNS?

- **Hosting service for DNS domains** — provides name resolution using Microsoft Azure infrastructure
- Two types: **Public DNS zones** (internet-facing) and **Private DNS zones** (VNet-internal)
- Uses Azure's global **anycast network** of name servers for high performance
- Does **NOT** support domain purchasing — buy domain from registrar, then host in Azure DNS
- 100% SLA for valid DNS responses

---

## 2. Key Components

| Component | Purpose |
|---|---|
| **DNS Zone** | Container for DNS records for a domain (e.g., contoso.com) |
| **Public DNS Zone** | Hosts records for internet name resolution |
| **Private DNS Zone** | Hosts records for VNet-internal name resolution (not internet-accessible) |
| **Record Set** | Group of records with same name and type (e.g., multiple A records for same hostname) |
| **Name Servers (NS)** | Azure-assigned authoritative name servers for your zone |
| **Virtual Network Link** | Connects a Private DNS Zone to a VNet |
| **Auto-registration** | Automatically creates DNS records for VMs in a linked VNet |

---

## 3. DNS Record Types

| Record Type | Purpose | Example |
|---|---|---|
| **A** | Maps hostname → **IPv4 address** | www → 10.0.0.1 |
| **AAAA** | Maps hostname → **IPv6 address** | www → 2001:db8::1 |
| **CNAME** | Alias → **another hostname** | www → contoso.azurewebsites.net |
| **MX** | Mail exchange server | @ → mail.contoso.com (priority 10) |
| **TXT** | Text record (SPF, verification, etc.) | @ → "v=spf1 include:..." |
| **NS** | Authoritative name servers for zone | @ → ns1-01.azure-dns.com |
| **SOA** | Start of Authority — zone metadata | Auto-created, one per zone |
| **SRV** | Service location (port + host) | _sip._tcp → sipserver:5060 |
| **PTR** | Reverse lookup (IP → hostname) | Used in reverse DNS zones |
| **CAA** | Certificate Authority Authorization | Specifies allowed CAs for domain |

> ⚠️ **EXAM TIP:** **CNAME** cannot be at the **zone apex** (root domain, e.g., `contoso.com`). At apex, use an **Alias record** instead. CNAME can only be used for subdomains (e.g., `www.contoso.com`).

> ⚠️ **EXAM TIP:** **SOA and NS records** are auto-created when zone is created. They **cannot be deleted** and are always at the zone apex (@).

---

## 4. Alias Records

- Special Azure DNS feature — **not a standard DNS record type**
- Points to an **Azure resource** instead of an IP address
- Supported on **A, AAAA, and CNAME** record types
- **Automatically updates** when the target resource IP changes (no stale records)

### Alias Record Targets

| Target Resource | Use Case |
|---|---|
| **Azure Public IP** | Track IP changes automatically |
| **Azure Traffic Manager profile** | Point apex domain to Traffic Manager |
| **Azure CDN endpoint** | Point domain to CDN |
| **Another DNS record set** (same zone) | Alias within the same zone |

### Portal Path — Create Alias Record
```
DNS Zone → + Record set →
Name, Type: A/AAAA/CNAME →
Alias record set: Yes →
Alias type: Azure resource / Zone record set →
Select target → OK
```

> ⚠️ **EXAM TIP:** Alias records solve the **zone apex CNAME problem**. You CANNOT use CNAME at apex (@), but you CAN use an **A record alias** pointing to Traffic Manager/Public IP/CDN. This is **heavily tested**.

> ⚠️ **EXAM TIP:** Alias records **auto-update** when target IP changes. Standard A records become **stale** if IP changes.

---

## 5. Public DNS Zones

- Host DNS records for **internet-facing** domain name resolution
- After creating zone → update domain registrar with Azure NS records
- **Globally distributed** across Azure's anycast DNS servers
- Support all standard DNS record types

### Portal Path — Create Public DNS Zone
```
Home → + Create a resource → DNS zone → Create →
Subscription, RG, Name (e.g., contoso.com) →
Type: Public → Create
```

### Portal Path — Add Record Set
```
DNS Zone → + Record set →
Name (e.g., www), Type (A/CNAME/MX/etc.),
TTL, Value → OK
```

### Delegate Domain to Azure DNS
1. Create DNS Zone in Azure → note the 4 NS records assigned
2. Go to **domain registrar** → update NS records to Azure's name servers
3. Delegation complete — Azure DNS now authoritative for the domain

> ⚠️ **EXAM TIP:** Azure DNS does NOT sell domains. Buy from registrar (App Service Domains, GoDaddy, etc.) then delegate NS records to Azure DNS. Azure DNS **only hosts** the zones.

> ⚠️ **EXAM TIP:** Each public DNS zone gets **4 name servers** (across different TLDs: .com, .net, .org, .info). All 4 should be configured at registrar.

---

## 6. Private DNS Zones

- Host DNS records for **VNet-internal** name resolution (not resolvable from internet)
- VMs in linked VNets resolve records using Azure's **168.63.129.16** DNS
- Supports **auto-registration** — automatically creates/removes A records for VMs
- Can use **any domain name** (e.g., contoso.internal, myapp.local)
- Cross-VNet resolution supported (link multiple VNets to same zone)

### Portal Path — Create Private DNS Zone
```
Home → + Create a resource → Private DNS zone → Create →
Subscription, RG, Name (e.g., contoso.internal) → Create
```

### Virtual Network Links

| Link Type | Auto-registration | Access |
|---|---|---|
| **Registration link** | ✅ Enabled — VMs auto-registered | VMs resolve + auto-register |
| **Resolution link** | ❌ Disabled | VMs resolve only (no auto-register) |

### Constraints
- One VNet can have **auto-registration** enabled with only **ONE private zone**
- One VNet can have **resolution links** to up to **1,000 private zones**
- One private zone can have up to **1,000 virtual network links**
- One private zone can have up to **100 auto-registration links**

### Portal Path — Add Virtual Network Link
```
Private DNS Zone → Settings → Virtual network links →
+ Add → Name, VNet: Select, Enable auto registration: Yes/No → OK
```

> ⚠️ **EXAM TIP:** **Auto-registration = ONE private zone per VNet only** (cannot auto-register in multiple zones). But a VNet can RESOLVE from multiple private zones.

> ⚠️ **EXAM TIP:** Auto-registration creates **A records** for VM names automatically. When VM is deleted → record is auto-removed.

> ⚠️ **EXAM TIP:** Private DNS zone names do NOT need to be real domains — you can use any name (e.g., `corp.internal`).

---

## 7. Public vs Private DNS Zone Comparison

| Feature | **Public DNS Zone** | **Private DNS Zone** |
|---|---|---|
| **Scope** | Internet (global) | VNet-internal only |
| **Resolution** | Anyone on internet | Only linked VNets |
| **Domain required** | Must own/register domain | Any name (no registration needed) |
| **Auto-registration** | ❌ | ✅ (VMs in linked VNets) |
| **NS delegation** | Yes (update registrar) | No (automatic via 168.63.129.16) |
| **Name servers** | Azure's public name servers | Azure internal DNS (168.63.129.16) |
| **Record types** | All standard types | All standard types |
| **Alias records** | ✅ | ❌ |
| **Access from on-prem** | Via internet DNS | Via DNS forwarder / Private Resolver |

> ⚠️ **EXAM TIP:** **Alias records = Public DNS zones only**. Private DNS zones do NOT support alias records.

---

## 8. Azure DNS Private Resolver

- Enables DNS resolution between **on-prem and Azure Private DNS Zones**
- Replaces need for custom DNS forwarder VMs
- Two components:
  | Component | Purpose | Subnet |
  |---|---|---|
  | **Inbound Endpoint** | On-prem → Azure DNS resolution | Dedicated subnet (min /28) |
  | **Outbound Endpoint** | Azure → On-prem DNS resolution | Dedicated subnet (min /28) |
  | **DNS Forwarding Ruleset** | Rules for outbound forwarding (domain → on-prem DNS IPs) |

### Portal Path — Create Private Resolver
```
Home → + Create a resource → DNS Private Resolver → Create →
Name, Region, VNet → Inbound endpoint: subnet → 
Outbound endpoint: subnet → Ruleset → Create
```

> ⚠️ **EXAM TIP:** Private Resolver inbound/outbound endpoints each need a **separate dedicated subnet** (/28 min). Cannot share subnets with other resources.

---

## 9. Record Set TTL (Time to Live)

- TTL = how long DNS clients/resolvers **cache** the record
- Default TTL: **3600 seconds** (1 hour)
- Range: **1 second to 2,147,483,647 seconds**
- **SOA record** has its own TTL for negative caching (default: 300 seconds)
- Lower TTL = more frequent lookups (useful during migrations)

> ⚠️ **EXAM TIP:** Default TTL = **3600 seconds (1 hour)**. SOA minimum TTL = **300 seconds** (negative caching). Before DNS migration → lower TTL first.

---

## 10. Wildcard Records

- Use `*` as record name to match any subdomain
- Example: `*.contoso.com` → returns IP for any undefined subdomain
- Supported types: **A, AAAA, CNAME, MX, TXT, SRV, CAA**
- Does NOT match the zone apex or explicit records

> ⚠️ **EXAM TIP:** Wildcard records match only when **no exact record exists**. Explicit records always take priority over wildcard.

---

## 11. Security & RBAC

### RBAC Roles

| Role | Permissions |
|---|---|
| **DNS Zone Contributor** | Full management of DNS zones and records (NOT zone deletion) |
| **Network Contributor** | Full network management including DNS |
| **Contributor** | Full access to everything |
| **Private DNS Zone Contributor** | Full management of private DNS zones and records |
| **Reader** | View-only |

| Action | Minimum Role |
|---|---|
| Create public DNS zone | DNS Zone Contributor |
| Add/modify/delete records | DNS Zone Contributor |
| Delete DNS zone | Contributor |
| Create private DNS zone | Private DNS Zone Contributor |
| Create VNet link | Private DNS Zone Contributor + Network Contributor (on VNet) |

### Resource Locks
- **CanNotDelete** lock → prevents zone deletion but allows record changes
- **ReadOnly** lock → prevents ALL changes (records frozen)
- Apply at zone level to protect from accidental deletion

### Portal Path — Add Lock
```
DNS Zone → Settings → Locks → + Add →
Name, Lock type: Delete / Read-only → OK
```

> ⚠️ **EXAM TIP:** **DNS Zone Contributor** can manage records but **cannot delete the zone itself**. To delete zone → need **Contributor**. This distinction is tested.

> ⚠️ **EXAM TIP:** Two separate roles: **DNS Zone Contributor** (public) vs **Private DNS Zone Contributor** (private). They are different roles.

---

## 12. Monitoring & Diagnostics

### Key Metrics

| Metric | Description |
|---|---|
| **Query Volume** | Number of DNS queries served per zone |
| **Record Set Count** | Number of record sets in zone |
| **Record Set Capacity Utilization** | % of 10,000 record set limit used |

### Portal Path — View Metrics
```
DNS Zone → Monitoring → Metrics →
Select: Query Volume / Record Set Count → Apply
```

### Portal Path — Alerts
```
DNS Zone → Monitoring → Alerts → + New alert rule →
Signal: Record Set Capacity Utilization > 80% →
Action Group → Create
```

### Activity Log
```
DNS Zone → Monitoring → Activity log →
Filter by operation: Create/Update/Delete record sets
```

> ⚠️ **EXAM TIP:** Monitor **Record Set Capacity Utilization** — alert when approaching 10,000 limit. No diagnostic logs for DNS queries (only metrics).

---

## 13. Pricing Key Points

| Component | Cost |
|---|---|
| **DNS Zone (Public)** | Per zone per month (~$0.50/month for first 25 zones) |
| **DNS Zone (Private)** | Per zone per month |
| **DNS Queries** | Per million queries ($0.40 per million for first billion) |
| **Azure DNS Private Resolver** | Per endpoint per hour + per DNS query |
| **Health checks** | Included (via Traffic Manager, not DNS itself) |
| **Record set changes** | Free (no charge for adding/modifying records) |

> ⚠️ **EXAM TIP:** First **25 public zones = cheapest rate**. Queries are charged per million. Private zones also have per-zone and per-query charges.

---

## 14. Limitations & Constraints

| Constraint | Limit |
|---|---|
| Public DNS zones per subscription | **250** (can increase) |
| Private DNS zones per subscription | **1,000** |
| Record sets per zone (public) | **10,000** |
| Record sets per zone (private) | **25,000** |
| Records per record set | **20** |
| VNet links per private zone | **1,000** |
| Auto-registration links per private zone | **100** |
| Auto-registration private zones per VNet | **1** |
| Resolution private zones per VNet | **1,000** |
| Private Resolver inbound endpoints | **5 per resolver** |
| Private Resolver outbound endpoints | **5 per resolver** |
| DNS forwarding rulesets | **10 per outbound endpoint** |

> ⚠️ **EXAM TIP:** Public zone = **10,000 record sets**. Private zone = **25,000 record sets**. Records per set = **20**.

---

## 15. CLI / PowerShell Commands

| Action | Command |
|---|---|
| Create public zone | `az network dns zone create -g <rg> -n contoso.com` |
| Create private zone | `az network private-dns zone create -g <rg> -n contoso.internal` |
| Add A record | `az network dns record-set a add-record -g <rg> -z contoso.com -n www -a 1.2.3.4` |
| Add CNAME | `az network dns record-set cname set-record -g <rg> -z contoso.com -n www -c app.azurewebsites.net` |
| Add MX record | `az network dns record-set mx add-record -g <rg> -z contoso.com -n @ -e mail.contoso.com -p 10` |
| List records | `az network dns record-set list -g <rg> -z contoso.com` |
| Delete record | `az network dns record-set a remove-record -g <rg> -z contoso.com -n www -a 1.2.3.4` |
| Create VNet link | `az network private-dns link vnet create -g <rg> -z contoso.internal -n mylink -v <vnet-id> -e true` |
| List NS records | `az network dns zone show -g <rg> -n contoso.com --query nameServers` |

> ⚠️ **EXAM TIP:** CLI for public DNS = `az network dns`. CLI for private DNS = `az network private-dns`. Different command groups!

---

## 16. Quick-Fire Exam Points ⚡

1. Azure DNS = **hosting service** for DNS zones — does NOT sell/register domains
2. **Public DNS zone** = internet resolution. **Private DNS zone** = VNet-internal resolution
3. Each public zone gets **4 Azure name servers** — all 4 must be set at registrar
4. **CNAME cannot be at zone apex (@)**. Use **Alias record** (A type) instead
5. **Alias records** = Public DNS only, auto-update when Azure resource IP changes
6. Alias targets: **Public IP, Traffic Manager, CDN, same-zone record set**
7. **SOA and NS records** are auto-created and **cannot be deleted**
8. Default TTL = **3600 seconds (1 hour)**. SOA min TTL = **300 seconds**
9. **Private DNS auto-registration**: VM A records created/removed automatically
10. **One VNet → ONE auto-registration private zone only** (but many resolution links)
11. Private zone VNet links: max **1,000 total**, max **100 auto-registration**
12. **DNS Zone Contributor** = manage records but **cannot delete the zone**
13. **Private DNS Zone Contributor** = separate role from DNS Zone Contributor
14. Record sets per public zone = **10,000**. Per private zone = **25,000**
15. Records per record set = **20**
16. Public zones per subscription = **250**. Private zones = **1,000**
17. **Wildcard records (*)** match unresolved subdomains, explicit records take priority
18. Azure internal DNS IP = **168.63.129.16** (used by VMs for DNS resolution)
19. **Azure DNS Private Resolver** = replaces custom DNS forwarder VMs for hybrid DNS
20. Private Resolver needs **separate subnets** for inbound (/28) and outbound (/28) endpoints
21. CLI: Public = `az network dns`, Private = `az network private-dns` — different groups
22. **Alias record on A/AAAA at apex → Traffic Manager** = exam favorite scenario
23. DNS query charges: per million queries. Zone charges: per zone per month
24. **ReadOnly lock** on zone = freezes ALL records. **CanNotDelete** = allows record changes
25. Private DNS zone names can be **anything** — no real domain needed (e.g., `my.internal`)

---

## 17. Step-by-Step Configuration Mind Maps 🗺️

---

### 17.1 Create Public DNS Zone & Delegate Domain

> **Portal:** `Home → + Create a resource → DNS zone`

```
Create Public DNS Zone & Delegate
│
├── Step 1: Create DNS Zone
│   │   Portal: Home → + Create a resource → DNS zone
│   ├── Subscription, Resource Group
│   ├── Name: e.g., contoso.com
│   │   ⚠️ Must match domain you own/registered
│   ├── Type: Public
│   └── Create
│   ⚠️ RBAC: DNS Zone Contributor or higher
│
├── Step 2: Note Azure Name Servers
│   │   Portal: DNS Zone → Overview → Name servers
│   ├── 4 name servers displayed, e.g.:
│   │   ├── ns1-01.azure-dns.com
│   │   ├── ns2-01.azure-dns.net
│   │   ├── ns3-01.azure-dns.org
│   │   └── ns4-01.azure-dns.info
│   └── ⚠️ All 4 must be configured at registrar
│
├── Step 3: Update Domain Registrar
│   ├── Log in to domain registrar (GoDaddy, Namecheap, etc.)
│   ├── Find NS record / DNS delegation settings
│   ├── Replace existing NS records with all 4 Azure NS records
│   └── Save
│   ⚠️ Propagation may take up to 48 hours
│   ⚠️ Azure DNS does NOT register domains — buy elsewhere
│
├── Step 4: Add DNS Records
│   │   Portal: DNS Zone → + Record set
│   ├── A record: www → 1.2.3.4 (TTL: 3600)
│   ├── CNAME: blog → myblog.azurewebsites.net
│   ├── MX: @ → mail.contoso.com (priority 10)
│   ├── TXT: @ → "v=spf1 include:spf.protection.outlook.com"
│   └── Alias: @ (A record) → Public IP / Traffic Manager
│       ⚠️ Alias = auto-updates when resource IP changes
│
└── Step 5: Verify
    ├── nslookup www.contoso.com
    ├── dig contoso.com
    └── Confirm records resolve correctly
```

---

### 17.2 Create Private DNS Zone with Auto-Registration

> **Portal:** `Home → + Create a resource → Private DNS zone`

```
Create Private DNS Zone with Auto-Registration
│
├── Step 1: Create Private DNS Zone
│   │   Portal: Home → + Create a resource → Private DNS zone
│   ├── Subscription, Resource Group
│   ├── Name: e.g., contoso.internal
│   │   ⚠️ Can be any name — no domain registration needed
│   └── Create
│   ⚠️ RBAC: Private DNS Zone Contributor
│
├── Step 2: Add Virtual Network Link (Registration)
│   │   Portal: Private DNS Zone → Virtual network links → + Add
│   ├── Link name: e.g., hub-vnet-link
│   ├── Virtual network: Select VNet
│   ├── Enable auto registration: ✅ Yes
│   │   ⚠️ Only ONE auto-registration zone per VNet
│   │   ⚠️ VMs in VNet get A records created automatically
│   └── OK
│
├── Step 3: Verify Auto-Registration
│   │   Portal: Private DNS Zone → Overview → Record sets
│   ├── VM A records appear automatically (VM name → private IP)
│   └── ⚠️ Records auto-removed when VM is deleted
│
├── Step 4: Add Resolution-Only Link (Other VNets)
│   │   Portal: Private DNS Zone → Virtual network links → + Add
│   ├── Link name: e.g., spoke-vnet-link
│   ├── Virtual network: Select spoke VNet
│   ├── Enable auto registration: ❌ No (resolution only)
│   └── OK
│   ⚠️ VMs in this VNet can resolve but won't auto-register
│
└── Limits
    ├── Max 1,000 VNet links per zone
    ├── Max 100 auto-registration links per zone
    └── Max 1 auto-registration zone per VNet
```

---

### 17.3 Create Alias Record at Zone Apex

> **Portal:** `DNS Zone → + Record set`

```
Create Alias Record (Zone Apex)
│
├── Scenario: Point root domain (contoso.com) to Azure resource
│   ⚠️ CNAME not allowed at apex (@)
│   ⚠️ Use Alias record instead
│
├── Step 1: Navigate to DNS Zone
│   └── DNS Zone → + Record set
│
├── Step 2: Configure Alias Record
│   ├── Name: @ (apex) or leave blank
│   ├── Type: A (or AAAA)
│   ├── Alias record set: Yes
│   ├── Alias type:
│   │   ├── Azure resource → select:
│   │   │   ├── Public IP address
│   │   │   ├── Traffic Manager profile
│   │   │   └── Azure CDN endpoint
│   │   └── Zone record set → select record in same zone
│   ├── Select resource from dropdown
│   └── OK
│
└── ⚠️ Alias record auto-tracks IP changes
    ⚠️ Only supported in PUBLIC DNS zones (not private)
    ⚠️ Traffic Manager alias at apex = most tested exam scenario
```

---

### 17.4 Configure Azure DNS Private Resolver

> **Portal:** `Home → + Create a resource → DNS Private Resolver`

```
Configure DNS Private Resolver
│
├── Prerequisites
│   ├── VNet exists with available subnets
│   ├── Two dedicated subnets (/28 min each)
│   │   ├── Inbound endpoint subnet
│   │   └── Outbound endpoint subnet
│   │   ⚠️ Cannot be shared with other resources
│   └── On-prem DNS server IPs known (for outbound forwarding)
│
├── Step 1: Create Private Resolver
│   │   Portal: Home → + Create a resource → DNS Private Resolver
│   ├── Subscription, RG, Name, Region
│   ├── Virtual network: Select VNet
│   └── Create
│
├── Step 2: Create Inbound Endpoint
│   │   (On-prem → Azure DNS resolution)
│   ├── Name
│   ├── Subnet: Select dedicated subnet (/28+)
│   ├── Private IP: Dynamic or Static
│   └── Create
│   ⚠️ On-prem DNS forwards queries TO this IP
│   ⚠️ Resolves Azure Private DNS Zones
│
├── Step 3: Create Outbound Endpoint
│   │   (Azure → On-prem DNS resolution)
│   ├── Name
│   ├── Subnet: Select different dedicated subnet (/28+)
│   └── Create
│
├── Step 4: Create DNS Forwarding Ruleset
│   │   Portal: Outbound endpoint → DNS forwarding rulesets → Create
│   ├── Name
│   ├── Link to outbound endpoint
│   ├── Link to VNet(s) that need the rules
│   └── Add rules:
│       ├── Domain name: e.g., onprem.local.
│       ├── Target DNS servers: On-prem DNS IPs (+ port 53)
│       └── State: Enabled
│   ⚠️ Domain name must end with dot (.)
│
└── Result
    ├── On-prem → Inbound EP IP → resolves Azure private zones
    ├── Azure VMs → Outbound EP → forwards to on-prem DNS
    └── ⚠️ Replaces custom DNS forwarder VMs
```

---

### 17.5 Monitor DNS Zone

> **Portal:** `DNS Zone → Monitoring`

```
Monitor DNS Zone
│
├── View Metrics
│   │   Portal: DNS Zone → Monitoring → Metrics
│   ├── Query Volume → queries per zone
│   ├── Record Set Count → current record sets
│   └── Record Set Capacity Utilization (%)
│       ⚠️ Alert if approaching 10,000 limit (public)
│
├── Configure Alerts
│   │   Portal: DNS Zone → Monitoring → Alerts
│   ├── + New alert rule
│   ├── Signal: Record Set Capacity Utilization > 80%
│   ├── Signal: Query Volume spikes (unusual traffic)
│   ├── Action Group: Email / SMS
│   └── Create
│
├── Activity Log
│   │   Portal: DNS Zone → Monitoring → Activity log
│   ├── Track: record set create/update/delete
│   ├── Track: zone modifications
│   └── Filter by operation and time range
│
└── Resource Locks (Protection)
    │   Portal: DNS Zone → Settings → Locks
    ├── CanNotDelete → prevents zone deletion
    └── ReadOnly → freezes all records (no changes)
        ⚠️ ReadOnly lock blocks ALL record modifications
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
