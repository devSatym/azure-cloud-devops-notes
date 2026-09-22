<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Pricing Factors — AZ-104 Revision Notes

---

## 1. What Determines Azure Pricing?

- Azure pricing is based on a **consumption model** — pay for what you use
- Key universal pricing factors: **Resource type**, **Region**, **SKU/Tier**, **Consumption (time/data/operations)**
- Some resources are **always free** (RBAC, NSGs, Tags, Policy, Activity Log alerts)
- Pricing can be **reduced** via: Reservations, Savings Plans, Hybrid Benefit, Spot, Dev/Test

### Universal Pricing Factors (Apply to Most Resources)

| Factor | Impact |
|---|---|
| **Region** | Prices vary by region (US East often cheapest; Brazil/Australia often highest) |
| **SKU / Tier** | Basic < Standard < Premium = increasing cost |
| **Consumption model** | Per-hour, per-GB, per-operation, per-unit, per-instance |
| **Data transfer (egress)** | Inbound = FREE; Outbound (egress) = charged |
| **Redundancy** | LRS < ZRS < GRS < RA-GRS < GZRS < RA-GZRS |
| **Reserved capacity** | Prepay 1/3 years = discounts up to 72% |
| **License** | BYOL (Hybrid Benefit) vs included license |

> ⚠️ **EXAM TIP:** **Inbound data transfer** = always **FREE**. **Outbound** (egress) = charged. Data between services in the **same region** = often free or cheap. **Cross-region** data transfer = always charged. This is tested frequently.

---

## 2. Data Transfer Pricing (Applies to ALL Resources)

| Transfer Type | Cost |
|---|---|
| **Inbound (ingress)** to Azure | **FREE** — always |
| **Outbound (egress)** from Azure to internet | **Charged** per GB (tiered pricing) |
| **Between Azure regions** | **Charged** per GB |
| **Within same region, same VNet** | **FREE** |
| **Within same region, VNet peering** | Charged (small per-GB fee) |
| **Cross-region VNet peering** | Charged (higher per-GB fee) |
| **Availability Zone transfer** (same region, cross-zone) | **Charged** (small per-GB) |
| **VPN Gateway data transfer** | Charged per GB egress |
| **ExpressRoute data transfer** | Metered plan = charged; Unlimited plan = included |

### Egress Pricing Tiers (approximate)

| Monthly Egress | Per-GB Cost |
|---|---|
| First 5 GB | **FREE** |
| 5 GB – 10 TB | ~$0.087/GB |
| 10 TB – 50 TB | ~$0.083/GB |
| 50 TB – 150 TB | ~$0.07/GB |
| 150 TB+ | ~$0.05/GB |

> ⚠️ **EXAM TIP:** First **5 GB/month** egress = FREE. Ingress = always FREE. Cross-zone traffic within same region = charged (small). VNet peering in same region = charged per GB. These data transfer costs add up and are tested.

---

## 3. Compute — Virtual Machines

### Pricing Factors

| Factor | How It Affects Price |
|---|---|
| **VM Size (Series/SKU)** | B-series (burstable, cheapest) → D/E/F/M series (progressively more) |
| **vCPUs + RAM** | More cores + memory = higher cost |
| **Region** | Same VM size costs differently per region |
| **OS** | Linux = cheaper (no OS license); Windows = OS license included |
| **Disk type** | Standard HDD < Standard SSD < Premium SSD < Ultra Disk |
| **Run time** | Per-second billing (minimum 1 minute) |
| **State** | Running = charged; **Stopped (deallocated)** = NOT charged for compute |
| **Public IP** | Static PIP = per-hour charge; Dynamic = free when allocated |
| **Outbound data** | Charged per GB |
| **Hybrid Benefit** | BYOL Windows/SQL = up to 40-55% savings |
| **Reserved Instance** | 1/3 year = up to 72% savings |
| **Spot pricing** | Up to 90% discount, evictable |
| **Dev/Test pricing** | ~55% savings (no Windows license charge) |

### VM Pricing Comparison

| Pricing Model | Discount | Commitment | Cancel | Use Case |
|---|---|---|---|---|
| **PAYG** | 0% | None | Anytime | Unknown workloads |
| **RI 1-year** | ~35-40% | 1 year | $50K cap, 12% fee | Steady workloads |
| **RI 3-year** | ~55-72% | 3 years | $50K cap, 12% fee | Long-term stable |
| **Savings Plan** | Up to 65% | 1/3 year | ❌ Cannot cancel | Variable compute |
| **Spot** | Up to 90% | None | Evictable | Batch, interruptible |
| **Dev/Test** | ~55% | Dev/Test sub | Anytime | Non-production |
| **AHB (Windows)** | ~40% | SA license | Toggle anytime | Have on-prem licenses |

### What You're Charged For (Running VM)

```
VM Cost Breakdown
│
├── Compute (per-second, billed hourly)
│   ├── vCPU + Memory allocation
│   └── OS License (Windows) or free (Linux)
│
├── Storage (always charged, even when VM stopped)
│   ├── OS Disk (managed disk type)
│   ├── Data Disks (managed disk type + size)
│   └── Temporary disk (included, no charge)
│
├── Networking
│   ├── Public IP (Standard = per-hour; Basic Dynamic = free)
│   ├── Outbound data transfer (egress)
│   └── VNet peering traffic (if applicable)
│
└── Optional Add-ons
    ├── Azure Backup (per-instance + storage)
    ├── Azure Site Recovery (per-instance)
    ├── Azure Monitor Agent (Log Analytics ingestion)
    └── Extensions (some free, some paid)
```

> ⚠️ **EXAM TIP:** **Stopped (deallocated)** VM = NO compute charge, but **disk still charged**. **Stopped (not deallocated)** = STILL charged for compute! Must **deallocate** to stop billing. Temporary/ephemeral disk = free but data lost on deallocation.

---

## 4. Compute — VM Scale Sets (VMSS)

### Pricing Factors

| Factor | Details |
|---|---|
| **VM size** | Same pricing as individual VMs |
| **Number of instances** | More instances = higher cost (linear) |
| **Orchestration mode** | Uniform (standard) / Flexible |
| **Auto-scale** | Cost varies with instance count changes |
| **Spot instances in VMSS** | Up to 90% savings per instance |
| **VMSS itself** | **FREE** — you pay for VMs inside it |

> ⚠️ **EXAM TIP:** VMSS service = **FREE**. You pay for the VMs within it. Auto-scale adds/removes instances = cost changes dynamically. Spot VMSS instances can reduce costs dramatically for fault-tolerant workloads.

---

## 5. Networking — Load Balancer

| Factor | Basic (FREE) | Standard (Paid) |
|---|---|---|
| **Base charge** | FREE | Per-rule per-hour |
| **Data processed** | FREE | Per-GB processed |
| **Rules** | Up to 250 | Up to 1,000+ |
| **Health probes** | FREE | Included |
| **Outbound rules** | N/A | Per-rule |

### Standard LB Pricing Components

```
Standard LB Cost = (Rules × $/rule/hour) + (Data processed × $/GB)
```

> ⚠️ **EXAM TIP:** Basic LB = completely **FREE**. Standard LB = per-rule + per-GB. If exam asks about cost-free load balancing → Basic LB (but no SLA, no zones).

---

## 6. Networking — Application Gateway

### Pricing Factors

| Factor | v1 | v2 |
|---|---|---|
| **Base charge** | Per-instance per-hour | **Fixed hourly cost** |
| **Data processed** | Per-GB | **Capacity Units** (CU) |
| **WAF** | Additional per-instance | Additional per-CU |
| **Instances** | Manual (you set count) | Auto-scaled |

### v2 Capacity Units (CU)

Each CU includes:
- **2,500 persistent connections**
- **2.22 Mbps throughput**
- **1 compute unit** (TLS, URL rewrites, etc.)

You pay for the **higher of**: configured minimum OR actual consumption.

> ⚠️ **EXAM TIP:** App Gateway v2 pricing = **fixed hourly + capacity units consumed**. v1 = per-instance. WAF tier = higher cost than Standard. AutoScale in v2 = can scale to zero (0 minimum instances = only fixed cost).

---

## 7. Networking — VPN Gateway

### Pricing Factors

| Factor | Details |
|---|---|
| **SKU** | Basic < VpnGw1 < VpnGw2 < VpnGw3 < VpnGw4 < VpnGw5 |
| **Hourly charge** | Charged even when no tunnel is active |
| **S2S tunnels** | Included in SKU (10 for Basic; 30 for GW1-3; 100 for GW4-5) |
| **P2S connections** | Included in SKU allowance |
| **Outbound data** | Charged per GB |
| **AZ suffix** | Higher cost than non-AZ version |
| **Active-Active** | 2 gateway instances = ~2× cost |

### Cost Formula

```
VPN Gateway Cost = (Hours running × $/hour for SKU) + (Egress data × $/GB)
```

> ⚠️ **EXAM TIP:** VPN Gateway charges **per-hour even when no tunnels are active**. You pay for the gateway existing. Active-Active mode = roughly 2× cost because two instances. Data transfer out = additional charges. Basic SKU = cheapest but fewest features.

---

## 8. Networking — ExpressRoute

### Pricing Factors

| Factor | Details |
|---|---|
| **SKU** | Local < Standard < Premium |
| **Bandwidth** | 50 Mbps to 10 Gbps |
| **Data plan** | **Metered** (per-GB egress) or **Unlimited** (flat rate) |
| **Port fee** | Monthly charge per port |
| **Add-ons** | Global Reach, Premium add-on |

### ExpressRoute SKU Pricing

| SKU | Data Pricing | VNet Links | Best For |
|---|---|---|---|
| **Local** | **FREE unlimited** data (to nearby regions) | 10 | Cost-sensitive, nearby regions |
| **Standard (Metered)** | Per-GB egress | 10 | Variable traffic |
| **Standard (Unlimited)** | Fixed monthly flat rate | 10 | High/predictable traffic |
| **Premium (Metered)** | Per-GB egress (higher cap) | 100 | Global reach |
| **Premium (Unlimited)** | Fixed monthly flat rate (higher) | 100 | Global, high volume |

> ⚠️ **EXAM TIP:** ExpressRoute **Local** = cheapest — **unlimited free data** to nearby regions. Standard/Premium Metered = charged per GB. Unlimited = flat rate regardless of data volume. Premium adds Global Reach + more VNet links (100 vs 10).

---

## 9. Networking — Azure Bastion

### Pricing Factors

| SKU | Pricing Model | Scale Units |
|---|---|---|
| **Basic** | Per-hour (fixed for 2 scale units) | Cannot adjust |
| **Standard** | Per-hour + per-scale-unit | 2–50 scale units |

| Factor | Details |
|---|---|
| **Hourly charge** | Charged when deployed (always-on) |
| **Outbound data** | Charged per GB |
| **Scale units** | More units = higher cost but more concurrent sessions |
| **RDP/SSH sessions** | Included in hourly cost |

> ⚠️ **EXAM TIP:** Bastion = **always-on, always charging** while deployed. There is NO free tier. To stop costs, you must **delete** the Bastion resource. Each scale unit supports ~20-25 concurrent RDP / 40-50 SSH sessions.

---

## 10. Networking — Azure Firewall

### Pricing Factors

| SKU | Deployment Charge | Data Processing |
|---|---|---|
| **Standard** | Per-hour (fixed) | Per-GB processed |
| **Premium** | Per-hour (higher) | Per-GB processed (higher) |
| **Basic** | Per-hour (lowest) | Per-GB processed |

| Factor | Details |
|---|---|
| **Hourly charge** | 24×7 — charged even with zero traffic |
| **Data processed** | Per-GB through the firewall |
| **Premium features** | TLS inspection, IDPS, URL filtering = higher rate |
| **Firewall Manager** | Additional charges for policies |

> ⚠️ **EXAM TIP:** Azure Firewall = expensive — charges **per-hour** (always-on) + per-GB of data. In Dev/Test, consider deleting and recreating to save costs. Standard vs Premium = Premium adds TLS inspection, IDPS at higher cost.

---

## 11. Networking — Other Resources

| Resource | Pricing Factors | Free? |
|---|---|---|
| **NSG** | **FREE** | ✅ |
| **Route Table (UDR)** | **FREE** | ✅ |
| **VNet** | **FREE** (no charge for VNet itself) | ✅ |
| **Subnet** | **FREE** | ✅ |
| **VNet Peering** | Per-GB data transfer (in + out) | ❌ |
| **NAT Gateway** | Per-hour + per-GB processed | ❌ |
| **DDoS Protection Standard** | ~$2,944/month fixed + per-GB overage | ❌ |
| **Network Watcher** | Free (limited); NSG flow logs = storage charges | Partial |
| **Traffic Manager** | Per-profile/month + per-million DNS queries + health checks | ❌ |
| **Azure DNS** | Per-zone/month + per-million queries | ❌ |
| **Private DNS Zone** | Per-zone/month + per-million queries | ❌ |
| **Private Endpoint** | Per-hour + per-GB data processed | ❌ |
| **Public IP (Basic Dynamic)** | **FREE** | ✅ |
| **Public IP (Standard/Static)** | Per-hour | ❌ |

> ⚠️ **EXAM TIP:** VNet, Subnet, NSG, Route Tables = **FREE**. VNet peering = charged per-GB BOTH directions. DDoS Protection Standard = very expensive (~$2,944/month). Private Endpoints = per-hour + per-GB. Know what's free vs paid.

---

## 12. Storage Accounts

### Pricing Factors

| Factor | How It Affects Price |
|---|---|
| **Performance tier** | Standard (HDD) < Premium (SSD) |
| **Access tier** | Hot > Cool > Cold > Archive (storage cost) |
| **Redundancy** | LRS < ZRS < GRS < RA-GRS < GZRS < RA-GZRS |
| **Capacity (GB stored)** | Per-GB per-month |
| **Operations** | Per 10,000 read/write/list operations |
| **Data retrieval** | Per-GB retrieved (Cool, Cold, Archive) |
| **Data transfer (egress)** | Per-GB outbound |
| **Blob snapshots** | Charged for incremental changes |
| **Soft delete** | Charged for retained deleted data |
| **Region** | Varies by Azure region |

### Access Tier Cost Comparison

| Tier | Storage $/GB | Read Ops (per 10K) | Data Retrieval $/GB | Min Retention |
|---|---|---|---|---|
| **Hot** | Highest | Lowest | FREE | None |
| **Cool** | ~50% less | Higher | Charged | 30 days |
| **Cold** | ~68% less | Higher | Charged | 90 days |
| **Archive** | ~90% less | Highest | Highest | 180 days |

### Storage Pricing Formula

```
Storage Cost =
  (Capacity × $/GB for tier/redundancy)
  + (Write ops × $/10K ops)
  + (Read ops × $/10K ops)
  + (Data retrieval × $/GB — Cool/Cold/Archive only)
  + (Egress data × $/GB)
  + (Snapshots/versions × incremental $/GB)
```

### Managed Disk Pricing

| Disk Type | Price Level | IOPS | Throughput |
|---|---|---|---|
| **Standard HDD** | Lowest | Up to 500 | Up to 60 MB/s |
| **Standard SSD** | $ | Up to 6,000 | Up to 750 MB/s |
| **Premium SSD** | $$ | Up to 20,000 | Up to 900 MB/s |
| **Premium SSD v2** | $$$ | Up to 80,000 | Up to 1,200 MB/s |
| **Ultra Disk** | $$$$ | Up to 160,000 | Up to 4,000 MB/s |

- Disks charged by **provisioned size** (not used capacity)
- **Snapshots** = charged for actual data stored (incremental)
- **Unattached disks** still incur charges — delete if unused

> ⚠️ **EXAM TIP:** Storage = billed by **capacity + operations + retrieval + egress**. Hot = high storage cost, low access cost. Archive = low storage, high access cost. Disks = charged for **provisioned size** (even if you only use 10GB of a 128GB disk). Unattached disks = still cost money. Early delete from Cool/Cold/Archive = charged for remaining minimum days.

---

## 13. Azure SQL Database

### Pricing Factors

| Factor | Details |
|---|---|
| **Purchasing model** | **DTU** (bundled) or **vCore** (granular) |
| **Tier** | Basic < Standard < Premium (DTU); GP < BC < Hyperscale (vCore) |
| **Compute** | Provisioned (per-hour) or **Serverless** (per-second, auto-pause) |
| **Storage** | Per-GB per-month (included amount varies by tier) |
| **Backup storage** | First (1× DB size) storage free; excess = per-GB |
| **Redundancy** | Locally redundant < zone-redundant < geo-redundant |
| **License** | Included or BYOL (Azure Hybrid Benefit) |
| **Region** | Varies per region |

### DTU vs vCore Comparison

| Feature | DTU Model | vCore Model |
|---|---|---|
| **Bundle** | CPU + IO + Memory bundled | Each configurable separately |
| **Tiers** | Basic, Standard (S0-S12), Premium (P1-P15) | General Purpose, Business Critical, Hyperscale |
| **Scaling** | Fixed levels (S0, S1, S2, etc.) | Flexible vCPU count |
| **AHB** | ❌ Not available | ✅ Available |
| **Reserved capacity** | ❌ | ✅ (1/3 year) |
| **Serverless** | ❌ | ✅ (GP tier only) |
| **Best for** | Simple, predictable workloads | Flexible, complex workloads |

### Serverless Pricing

| Factor | Details |
|---|---|
| **Compute** | Per-second billing based on vCores used |
| **Auto-pause** | After configurable idle period → paused (no compute charge) |
| **Auto-resume** | First connection resumes the database |
| **Min/Max vCores** | Configure range (e.g., 0.5–4 vCores) |
| **Storage** | Always charged (even when paused) |

> ⚠️ **EXAM TIP:** DTU = bundled, simple pricing, no AHB. vCore = flexible, AHB available, Serverless option. **Serverless** = per-second billing + auto-pause (only GP tier). When paused, **storage still charged**. Exam may ask which model supports AHB → vCore only.

---

## 14. Azure App Service

### Pricing Factors

| Factor | Details |
|---|---|
| **Tier** | Free < Shared < Basic < Standard < Premium < Isolated |
| **Instance size** | Small / Medium / Large within each tier |
| **Instance count** | Number of instances (auto-scale or manual) |
| **OS** | Linux = cheaper; Windows = more expensive |
| **Region** | Varies by region |
| **Custom domain** | Requires Shared+ (free has restrictions) |
| **SSL** | Requires Basic+ |
| **Slots** | Included in Standard+ (no extra charge per slot) |

### Key Pricing Points

| Tier | Charged For | Notes |
|---|---|---|
| **Free** | Nothing (60 min CPU/day limit) | 1 GB storage, no SLA |
| **Shared** | Per-app compute minutes | No SLA |
| **Basic** | Per-instance per-hour (dedicated) | SLA, Always On |
| **Standard** | Per-instance per-hour | Slots, Auto-scale included |
| **Premium** | Per-instance per-hour (higher) | Better hardware, more slots |
| **Isolated** | Per-instance + ASE stamp fee | ASE = additional flat fee |

> ⚠️ **EXAM TIP:** App Service Plan = you pay for the **Plan** (not per app). Multiple apps on same plan = no extra App Service charge. Auto-scale adding instances = cost increases. **Free** tier = 60 CPU-minutes/day. **Isolated** = Plan cost + ASE hosting environment stamp fee (expensive).

---

## 15. Azure Monitor & Log Analytics

### Pricing Factors

| Component | Pricing |
|---|---|
| **Platform metrics** | **FREE** |
| **Custom metrics** | Per-million samples ingested |
| **Log ingestion** | Per-GB ingested (5 GB/month free) |
| **Log retention** | First 31 days free; then per-GB/month |
| **Basic logs** | Lower ingestion rate; limited query |
| **Archive** | Lowest retention rate; must restore to query |
| **Queries** | Cluster-based = per-GB scanned; PAYG = included |
| **Alerts (metric)** | Per-signal monitored per-month |
| **Alerts (log)** | Per-signal + frequency evaluated |
| **Activity Log alerts** | **FREE** |
| **Service Health alerts** | **FREE** |
| **Application Insights** | Per-GB ingested (5 GB/month free) |
| **Diagnostic settings** | Log Analytics ingestion charges apply |

### Commitment Tier Discounts

| Daily Ingestion | Discount vs PAYG |
|---|---|
| 100 GB/day | Lowest discount |
| 200 GB/day | More |
| 300 GB/day | More |
| 500 GB/day | More |
| 1,000+ GB/day | Highest discount |

> ⚠️ **EXAM TIP:** Platform metrics = **FREE**. Activity Log and Service Health alerts = **FREE**. Log Analytics = **5 GB/month free**, 31 days free retention. Diagnostic settings sending to Log Analytics = you pay ingestion charges. Commitment tiers reduce per-GB cost for high volume.

---

## 16. Azure Backup & Site Recovery

### Azure Backup Pricing

| Factor | Details |
|---|---|
| **Protected instance fee** | Based on size of data being backed up (tiered) |
| **Storage consumed** | Per-GB per-month for backup data |
| **Vault type** | Recovery Services Vault or Backup Vault = free to create |
| **Redundancy** | LRS < GRS (for vault storage) |
| **Soft delete** | 14 extra days = no charge |

| Protected Data Size | Instance Fee Tier |
|---|---|
| ≤ 50 GB | Lowest |
| 50–500 GB | Mid |
| 500 GB – 1 TB | Higher |
| 1 TB+ | Per-500 GB increments |

### Azure Site Recovery Pricing

| Factor | Details |
|---|---|
| **Per-instance** | Per protected VM per month |
| **First 31 days** | FREE for each new instance |
| **Storage** | Charged for replicated data storage |
| **Network** | Egress charges for replication traffic |
| **Test failover** | Charged for compute/storage during test |

> ⚠️ **EXAM TIP:** Backup vault = **free to create**; pay per-instance + storage. ASR = per-instance/month, first **31 days free**. Soft delete = 14 days retention at no extra charge. Backup storage redundancy (LRS vs GRS) affects cost.

---

## 17. Azure Key Vault

### Pricing Factors

| Factor | Standard | Premium |
|---|---|---|
| **Secrets operations** | $0.03 per 10K ops | $0.03 per 10K ops |
| **Keys (software)** | $0.03 per 10K ops | $0.03 per 10K ops |
| **Keys (HSM)** | N/A | $1 per key/month + ops |
| **Certificates** | $0.03 per renewal | $0.03 per renewal |
| **Certificate operations** | $0.03 per 10K ops | $0.03 per 10K ops |
| **Managed HSM** | N/A | Per-HSM-pool per hour |

> ⚠️ **EXAM TIP:** Key Vault = per-operation pricing (very cheap). HSM-protected keys = **Premium** tier only + per-key monthly charge. Secrets and certs = same price in both tiers.

---

## 18. Completely FREE Azure Services (AZ-104 Scope)

| Service | Details |
|---|---|
| **RBAC** | Role assignments = free |
| **Azure Policy** | Free for Azure resources (Arc = paid) |
| **Management Groups** | Free |
| **Resource Tags** | Free |
| **Resource Groups** | Free |
| **NSG** | Free |
| **Route Tables (UDR)** | Free |
| **VNet** | Free (VNet itself, not peering data) |
| **Subnets** | Free |
| **Cost Management** | Free for Azure |
| **Budgets** | Free |
| **Azure Advisor** | Free |
| **Activity Log** | Free (90 days retention) |
| **Activity Log Alerts** | Free |
| **Service Health** | Free |
| **Resource Health** | Free |
| **Basic Load Balancer** | Free |
| **Basic Dynamic Public IP** | Free |
| **Pricing Calculator** | Free (external tool) |
| **TCO Calculator** | Free (external tool) |
| **Azure AD Free edition** | Free (basic features) |
| **VMSS orchestration** | Free (pay for VMs only) |
| **Platform metrics** | Free |

> ⚠️ **EXAM TIP:** Memorize this list. If an exam question asks "which resource incurs NO additional charges," look for: RBAC, NSG, Route Tables, VNet, Tags, Policy, Basic LB, Activity Log alerts, Service Health.

---

## 19. Quick-Fire Exam Points ⚡

1. **Ingress** (data in) = always **FREE**; **Egress** (data out) = charged
2. First **5 GB/month** egress = FREE
3. Data transfer within **same VNet, same region** = FREE
4. **VNet peering** = charged per-GB in BOTH directions
5. **Cross-region** data transfer = always charged (both directions)
6. Azure region affects pricing — US East typically cheapest
7. **VM compute** = per-second billing (min 1 minute)
8. **Stopped (deallocated)** VM = NO compute charge; **disk still charged**
9. **Stopped (not deallocated)** VM = STILL charged for compute!
10. Managed Disks charged by **provisioned size**, not used capacity
11. **Unattached disks** still cost money — delete if unused
12. Temporary/ephemeral disk = included with VM, no extra charge
13. **Basic LB** = FREE; **Standard LB** = per-rule + per-GB
14. **NSG, UDR, VNet, Subnet** = all FREE
15. **VPN Gateway** = per-hour EVEN with zero tunnels + egress data
16. **Azure Bastion** = always-on, per-hour when deployed
17. **Azure Firewall** = per-hour (always-on) + per-GB processed
18. **DDoS Protection Standard** = ~$2,944/month flat + overage
19. **Public IP Basic Dynamic** = FREE; Standard/Static = per-hour
20. Storage cost = **capacity + operations + retrieval (Cool/Archive) + egress**
21. Storage **Hot** = high storage cost, low access cost
22. Storage **Archive** = low storage cost, high access + retrieval cost
23. Archive early deletion = charged for remaining days of 180-day minimum
24. Premium Storage = **LRS and ZRS only** (no GRS option)
25. Storage redundancy cost: **LRS < ZRS < GRS < RA-GRS < GZRS < RA-GZRS**
26. **SQL DTU** model = no AHB, no Serverless; **vCore** = AHB + Serverless
27. SQL Serverless = per-second, auto-pause; storage always charged when paused
28. SQL backup storage = first (1× DB size) FREE; excess charged
29. **App Service Plan** = pay for the plan, not per app
30. App Service **Free** tier = 60 CPU-minutes/day limit
31. App Service **Isolated** = Plan cost + ASE stamp fee
32. App Service auto-scale adding instances = cost increases
33. **Log Analytics** = 5 GB/month free; 31 days free retention
34. **Activity Log alerts** = FREE; **Metric alerts** = per-signal charge
35. Platform metrics = **FREE**; Custom metrics = per-million samples
36. **Diagnostic settings** → Log Analytics = you pay ingestion charges
37. **Azure Backup** = per-instance fee + storage; vault creation = FREE
38. Backup soft delete = 14 days at no charge
39. **ASR** = per-instance/month; first 31 days FREE
40. **Key Vault** = per-operation (very cheap); HSM keys = Premium only
41. **Reserved Instance** = up to 72% savings; can exchange/cancel ($50K)
42. **Savings Plan** = up to 65% savings; CANNOT cancel/exchange
43. **Spot VM** = up to 90% savings; NO SLA, evictable
44. **Azure Hybrid Benefit** = up to 40% Windows / 55% SQL savings
45. AHB + RI combined = up to **85%** savings
46. **ExpressRoute Local** = free unlimited data to nearby regions
47. Commitment tiers for Log Analytics = discounts for 100+ GB/day
48. **Cost Management, Budgets, Advisor** = all FREE services
49. **Cross-zone** traffic (same region) = small per-GB charge
50. Private Endpoints = per-hour + per-GB — not free

---

## 20. Step-by-Step Configuration Mind Maps 🗺️

---

### 20.1 Estimate Costs with Azure Pricing Calculator

> **URL:** `pricing.azure.com`

```
Estimate Costs (Pricing Calculator)
│
├── Step 1: Navigate
│   └── https://azure.microsoft.com/pricing/calculator/
│
├── Step 2: Add Products
│   ├── Search or browse by category
│   ├── Click product to add to estimate
│   │   ├── Virtual Machines
│   │   ├── Storage Accounts
│   │   ├── SQL Database
│   │   ├── App Service
│   │   ├── Bandwidth (egress)
│   │   └── Any Azure service
│   └── Configure each product:
│       ├── Region
│       ├── SKU / Tier / Size
│       ├── Hours per month (730 = 24×7)
│       ├── OS (Linux / Windows)
│       ├── Licensing (PAYG / AHB / RI / Savings Plan)
│       ├── Redundancy (LRS / GRS / etc.)
│       └── Data transfer estimates
│
├── Step 3: Review Estimate
│   ├── Monthly estimated cost (per product + total)
│   ├── Adjust quantities and configurations
│   └── Compare options (PAYG vs RI vs Spot)
│
├── Step 4: Export / Share
│   ├── Export as Excel
│   └── Share via link
│
└── ⚠️ Key Points
    ├── External website — NOT in Azure Portal
    ├── Estimates only — not actual billing
    ├── Does NOT include egress by default — add manually
    ├── Does NOT account for discounts unless you select them
    └── TCO Calculator is SEPARATE (for on-prem comparison)
```

---

### 20.2 Optimize Costs with Azure Advisor

> **Portal:** `Azure Advisor → Cost`

```
Optimize Costs with Advisor
│
├── Step 1: Navigate
│   └── Azure Advisor → Cost tab
│
├── Step 2: Review Recommendations
│   ├── Right-size or shutdown underutilized VMs
│   │   └── Based on CPU/memory utilization over 7+ days
│   ├── Delete unattached managed disks
│   │   └── Disks not attached to any VM
│   ├── Delete unused public IP addresses
│   │   └── Static PIPs with no association
│   ├── Purchase reserved instances
│   │   └── Based on VM usage patterns
│   ├── Purchase savings plans
│   ├── Reconfigure idle VPN gateways
│   └── Delete unused ExpressRoute circuits
│
├── Step 3: View Impact
│   ├── Impact: High / Medium / Low
│   ├── Estimated annual savings ($)
│   └── Affected resources
│
├── Step 4: Take Action
│   ├── Quick Fix (auto-remediate for some)
│   ├── Navigate to resource → make change
│   └── Dismiss / Postpone
│
├── Required Role: Reader (view) / Contributor (act)
│
└── ⚠️ Key Points
    ├── Advisor = FREE
    ├── Refreshes periodically, not real-time
    ├── Also covers: Security, Reliability, Performance
    └── Set up Advisor Alerts for new cost recommendations
```

---

### 20.3 Set Up Cost Allocation with Tags

> **Portal:** `Resource → Tags` + `Azure Policy`

```
Set Up Cost Allocation with Tags
│
├── Step 1: Define Tagging Strategy
│   ├── Decide key tag names:
│   │   ├── CostCenter (e.g., CC-1001)
│   │   ├── Department (e.g., Engineering)
│   │   ├── Project (e.g., ProjectAlpha)
│   │   ├── Environment (e.g., Prod / Dev / Test)
│   │   └── Owner (e.g., john@company.com)
│   └── Document naming conventions
│
├── Step 2: Apply Tags Manually
│   └── Resource → Tags → Add Name:Value → Save
│
├── Step 3: Enforce Tags via Azure Policy
│   ├── Azure Policy → Definitions →
│   │   ├── "Require a tag and its value on resources" (Deny)
│   │   ├── "Inherit a tag from the resource group" (Modify)
│   │   └── "Append a tag and its value to resources" (Append)
│   ├── Assign policy → Scope → Parameters (tag name/value)
│   └── ⚠️ Deny = blocks resource creation without tag
│
├── Step 4: View Costs by Tag
│   └── Cost Management → Cost analysis →
│       Group by → Tag → Select tag key →
│       View cost breakdown by tag value
│
├── Required Role
│   ├── Tag Contributor (to apply tags)
│   ├── Policy Contributor (to assign policies)
│   └── Cost Management Reader (to view analysis)
│
└── ⚠️ Key Points
    ├── Tags do NOT inherit from RG → use Policy (Modify)
    ├── Max 50 tags per resource
    ├── Tags are FREE
    ├── Policy enforcement = FREE for Azure resources
    └── Without tags, cost allocation is very difficult
```

---

### 20.4 Configure VM for Lowest Cost

> **Portal:** `Virtual Machines → Create`

```
Configure VM for Lowest Cost
│
├── Step 1: Choose Region
│   └── Select cheapest region that meets latency requirements
│       └── ⚠️ US East / US West 2 typically cheapest
│
├── Step 2: Choose OS
│   └── Linux = cheaper than Windows (no OS license)
│
├── Step 3: Choose Size
│   ├── B-series (burstable) = cheapest for light workloads
│   ├── Consider right-sizing based on actual needs
│   └── Check Advisor for resize recommendations
│
├── Step 4: Apply Discounts
│   ├── Spot VM: Check "Run with Azure Spot discount"
│   │   ├── Eviction policy: Deallocate / Delete
│   │   └── ⚠️ No SLA — for non-critical workloads only
│   ├── Azure Hybrid Benefit: "Already have a Windows license?"
│   │   └── Saves up to 40% on Windows VMs
│   ├── Reserved Instance: Purchase separately
│   │   └── Saves up to 72% for 1/3 years
│   └── Dev/Test subscription: Use for non-production
│       └── Saves ~55% (no Windows license)
│
├── Step 5: Choose Disk
│   ├── Standard HDD = cheapest (non-critical)
│   ├── Standard SSD = balanced
│   ├── Premium SSD = production workloads
│   └── ⚠️ Delete disk option on VM deletion to avoid orphaned costs
│
├── Step 6: Minimize Networking Costs
│   ├── Avoid Standard Static Public IP if not needed
│   ├── Use Basic Dynamic PIP (free) or no PIP (use Bastion)
│   └── Keep traffic within same region/VNet (free)
│
├── Step 7: Auto-shutdown
│   ├── VM → Auto-shutdown → Enable →
│   │   Set time → Notify before shutdown → Save
│   └── ⚠️ Auto-shutdown = deallocates (stops compute charges)
│
└── ⚠️ Cost Optimization Summary
    ├── Spot = up to 90% (interruptible)
    ├── RI = up to 72% (1/3 year commit)
    ├── AHB = up to 40% Windows / 55% SQL
    ├── Savings Plan = up to 65% (flexible)
    ├── Dev/Test = ~55% (non-production)
    ├── Deallocate when not in use
    ├── Right-size based on Advisor
    └── Delete unattached disks and unused PIPs
```


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
