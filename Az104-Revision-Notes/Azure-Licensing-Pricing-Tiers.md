<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Licensing & Pricing Tiers — AZ-104 Revision Notes

---

## 1. What is Azure Licensing?

- Azure uses a **pay-as-you-go** model for most resources + **license-based** for specific services
- Key licensing areas for AZ-104: **Entra ID**, **Windows Server**, **SQL Server**, **Azure Hybrid Benefit**, **Resource SKUs/Tiers**
- Licensing determines which **features** are available and at what **cost**
- Many Azure services have **Free** and **Paid** tiers with different feature sets

> ⚠️ **EXAM TIP:** AZ-104 heavily tests which features require which license tier — especially **Entra ID Free vs P1 vs P2** and resource **SKU differences** (Basic vs Standard vs Premium).

---

## 2. Microsoft Entra ID (Azure AD) Licensing

### Entra ID Editions

| Feature | Free | Office 365 Apps | P1 | P2 |
|---|---|---|---|---|
| **User/Group management** | ✅ | ✅ | ✅ | ✅ |
| **SSO (10 app limit)** | ✅ (10 apps) | ✅ (Unlimited) | ✅ (Unlimited) | ✅ (Unlimited) |
| **B2B collaboration** | ✅ | ✅ | ✅ | ✅ |
| **Self-service password change** (cloud) | ✅ | ✅ | ✅ | ✅ |
| **MFA** (Security Defaults) | ✅ | ✅ | ✅ | ✅ |
| **Conditional Access** | ❌ | ❌ | ✅ | ✅ |
| **Self-service password reset (SSPR)** | ❌ | ✅ (cloud only) | ✅ | ✅ |
| **SSPR with on-prem writeback** | ❌ | ❌ | ✅ | ✅ |
| **Hybrid identity (Azure AD Connect)** | ❌ | ❌ | ✅ | ✅ |
| **Dynamic groups** | ❌ | ❌ | ✅ | ✅ |
| **Group-based license assignment** | ❌ | ❌ | ✅ | ✅ |
| **Application Proxy** | ❌ | ❌ | ✅ | ✅ |
| **Custom Entra ID roles** | ❌ | ❌ | ✅ | ✅ |
| **Privileged Identity Management (PIM)** | ❌ | ❌ | ❌ | ✅ |
| **Access Reviews** | ❌ | ❌ | ❌ | ✅ |
| **Identity Protection** | ❌ | ❌ | ❌ | ✅ |
| **Entitlement Management** | ❌ | ❌ | ❌ | ✅ |
| **Risk-based Conditional Access** | ❌ | ❌ | ❌ | ✅ |
| **Azure AD Connect Health** | ❌ | ❌ | ✅ | ✅ |
| **Terms of use** | ❌ | ❌ | ✅ | ✅ |

### Key License Distinctions (Most Tested)

| Feature | Requires |
|---|---|
| **Conditional Access** | P1 |
| **SSPR with on-prem writeback** | P1 |
| **Dynamic Groups** | P1 |
| **Application Proxy** | P1 |
| **PIM** | P2 |
| **Access Reviews** | P2 |
| **Identity Protection** | P2 |
| **Risk-based Conditional Access** | P2 |
| **Entitlement Management** | P2 |

> ⚠️ **EXAM TIP:** **P1** = Conditional Access, Dynamic Groups, SSPR writeback, Application Proxy. **P2** = PIM, Access Reviews, Identity Protection, Risk-based CA. If the exam says "just-in-time access" → P2 (PIM). If "auto-assign users to groups based on attributes" → P1 (Dynamic Groups).

### Portal Path — View/Assign Licenses

```
Microsoft Entra ID → Licenses → All products →
  Select license → Assign → + Add users/groups → Assign
```

### Portal Path — Check License Features

```
Microsoft Entra ID → Overview → License section (shows current edition)
```

---

## 3. MFA Licensing

| MFA Method | License Required |
|---|---|
| **Security Defaults** (block legacy auth, require all MFA) | Free (all editions) |
| **Per-user MFA** (enable/disable per user) | Free (all editions) |
| **Conditional Access-based MFA** (policy-driven, more granular) | **P1** |
| **Risk-based MFA** (auto-triggered by sign-in risk) | **P2** |
| **MFA Server** (on-premises, deprecated) | Legacy — not on exam |

> ⚠️ **EXAM TIP:** Basic MFA = **Free** (Security Defaults). Conditional Access MFA (choose WHO/WHEN/WHERE) = **P1**. Risk-based MFA (auto-triggers on risky sign-ins) = **P2**. Security Defaults and Conditional Access are **mutually exclusive** — cannot use both.

---

## 4. Azure Hybrid Benefit (AHB)

### What is Azure Hybrid Benefit?

- Bring existing **on-premises licenses** to Azure for **discounted pricing**
- Applies to **Windows Server** and **SQL Server** with Software Assurance (SA)
- Savings: up to **85%** vs PAYG pricing (combined with Reserved Instances)

### Windows Server AHB

| Item | Details |
|---|---|
| **License type** | Windows Server Standard or Datacenter with SA |
| **Benefit** | Use Windows VM in Azure without paying for the OS license |
| **Standard (16-core)** | Covers up to **2 VMs** with ≤8 vCPUs each |
| **Datacenter (16-core)** | Covers **unlimited VMs** on dedicated hardware or 2 VMs on shared |
| **Where** | Azure VMs, VMSS, Azure Dedicated Host |
| **Savings** | Up to **40%** on Windows VM costs |

### SQL Server AHB

| Item | Details |
|---|---|
| **License type** | SQL Server Enterprise or Standard with SA |
| **Benefit** | Use SQL on Azure without paying license cost |
| **Applies to** | Azure SQL Database, SQL Managed Instance, SQL on Azure VMs |
| **Enterprise** | Can convert to SQL DB Business Critical or MI Business Critical |
| **Standard** | Can convert to SQL DB General Purpose or MI General Purpose |
| **Savings** | Up to **55%** on SQL costs |

### Portal Path — Enable AHB on VM

```
Virtual Machine → Configuration →
  Licensing → Azure Hybrid Benefit →
  "Already have a Windows Server license?": Yes → Save
```

### Portal Path — Enable AHB on SQL

```
SQL Database → Compute + storage →
  "Already have a SQL Server license?": Yes (Apply Azure Hybrid Benefit) → Apply
```

### CLI — Enable AHB on VM

```bash
# Create VM with AHB
az vm create --name myVM --resource-group myRG \
  --image Win2022Datacenter --size Standard_D4s_v3 \
  --license-type Windows_Server

# Update existing VM
az vm update --name myVM --resource-group myRG \
  --set licenseType=Windows_Server

# Check current license type
az vm show --name myVM --resource-group myRG --query licenseType
```

> ⚠️ **EXAM TIP:** Azure Hybrid Benefit for Windows = `--license-type Windows_Server`. For SQL = `--license-type` on SQL resource. Must have **Software Assurance** or equivalent subscription. Savings = up to 40% Windows, 55% SQL. Can be **combined** with Reserved Instances for up to 85% total savings.

---

## 5. VM Licensing & Pricing Models

### VM Pricing Options

| Pricing Model | Discount | Commitment | Cancel? | Best For |
|---|---|---|---|---|
| **Pay-As-You-Go** | 0% (baseline) | None | Anytime | Variable/unknown workloads |
| **Reserved Instance (RI)** | Up to **72%** | 1 or 3 years | Yes ($50K cap, 12% fee) | Stable, predictable workloads |
| **Savings Plan** | Up to **65%** | 1 or 3 years | ❌ No | Variable compute across services |
| **Spot VMs** | Up to **90%** | None (evictable) | N/A (evicted) | Batch, interruptible workloads |
| **Dev/Test pricing** | ~**55%** on VMs | Dev/Test sub | Anytime | Non-production environments |
| **Azure Hybrid Benefit** | Up to **40%** (Windows) | SA license owned | N/A | Existing on-prem licenses |

### Spot VM Details

| Feature | Details |
|---|---|
| **Discount** | Up to 90% vs PAYG pricing |
| **Eviction policy** | **Deallocate** (default) or **Delete** |
| **Eviction type** | Capacity-based or price-based |
| **Max price** | Set max you'll pay; evicted if market exceeds it (-1 = PAYG max) |
| **SLA** | ❌ No SLA — can be evicted anytime |
| **Use cases** | Batch processing, CI/CD, dev/test, stateless workloads |
| **NOT for** | Production, stateful apps, databases |

### Portal Path — Create Spot VM

```
Virtual Machines → + Create →
  Basics → Size → Check "Run with Azure Spot discount" →
  Eviction type: Capacity only / Price or capacity →
  Eviction policy: Stop/Deallocate / Delete →
  Max price ($/hr or -1 for PAYG rate) → Continue
```

> ⚠️ **EXAM TIP:** Spot VMs = up to **90% savings** but **NO SLA** and can be **evicted anytime**. Eviction policy: Deallocate (keep disk) or Delete (remove everything). Max price = -1 means pay up to PAYG rate (lowest eviction risk). Never use Spot for production.

---

## 6. Load Balancer SKU Licensing

| Feature | Basic (Free) | Standard (Paid) |
|---|---|---|
| **Price** | **FREE** | Per-rule + per-GB processed |
| **Backend pool size** | Up to 300 VMs | Up to **1,000** VMs |
| **Backend pool type** | Single availability set or VMSS | Any VMs in single VNet |
| **Health probes** | TCP, HTTP | TCP, HTTP, **HTTPS** |
| **Availability Zones** | ❌ | ✅ Zone-redundant + zonal |
| **SLA** | ❌ None | ✅ **99.99%** |
| **Diagnostics** | ❌ Limited | ✅ Azure Monitor, logs |
| **HA Ports** | ❌ | ✅ |
| **Outbound rules** | ❌ | ✅ |
| **Secure by default** | ❌ Open (NSG optional) | ✅ **Closed** (NSG required) |
| **Global LB** | ❌ | ✅ (cross-region) |
| **Move support** | ✅ | ❌ Cannot move cross-sub |

> ⚠️ **EXAM TIP:** Basic LB = **FREE**, no SLA, no zone support, open by default. Standard LB = **paid**, 99.99% SLA, zone-redundant, **closed by default** (must add NSG). Standard requires **Standard Public IP**. Basic is being **retired** — new deployments should use Standard. Backend: Basic = 1 availability set; Standard = any VMs in VNet.

---

## 7. Public IP SKU Licensing

| Feature | Basic | Standard |
|---|---|---|
| **Price** | **FREE** (static); Dynamic = free | Per-hour charge |
| **Allocation** | Dynamic or Static | **Static only** |
| **Availability Zone** | ❌ | ✅ Zone-redundant |
| **Routing preference** | ❌ | ✅ Internet or Microsoft |
| **SLA** | ❌ | ✅ 99.99% |
| **Secure by default** | ❌ Open | ✅ **Closed** (NSG required) |
| **Compatible with** | Basic LB | Standard LB |
| **Global tier** | ❌ | ✅ |

> ⚠️ **EXAM TIP:** Standard PIP = **static only**, zone-redundant, closed by default. Basic PIP = dynamic or static, no zones. Standard LB **requires** Standard PIP. Cannot mix Basic PIP with Standard LB.

---

## 8. VPN Gateway SKU Licensing

| SKU | S2S Tunnels | P2S Connections | Throughput | AZ Support | Price |
|---|---|---|---|---|---|
| **Basic** | 10 | 128 | 100 Mbps | ❌ | Lowest |
| **VpnGw1** | 30 | 250 | 650 Mbps | ❌ | $ |
| **VpnGw1AZ** | 30 | 250 | 650 Mbps | ✅ | $$ |
| **VpnGw2** | 30 | 500 | 1 Gbps | ❌ | $$ |
| **VpnGw2AZ** | 30 | 500 | 1 Gbps | ✅ | $$$ |
| **VpnGw3** | 30 | 1,000 | 1.25 Gbps | ❌ | $$$ |
| **VpnGw3AZ** | 30 | 1,000 | 1.25 Gbps | ✅ | $$$$ |
| **VpnGw4** | 100 | 5,000 | 5 Gbps | ❌ | $$$$ |
| **VpnGw5** | 100 | 10,000 | 10 Gbps | ❌ | $$$$$ |

### Key VPN Gateway SKU Points

- **Basic** SKU = legacy; does **NOT** support IKEv2, RADIUS, OpenVPN, or AZ
- Basic to non-Basic upgrade requires **gateway re-creation** (delete + create)
- VpnGw1-5 can upgrade **in-place** (no delete needed, but brief downtime)
- **AZ** suffix = availability zone support = higher resiliency

> ⚠️ **EXAM TIP:** Basic VPN GW = no IKEv2, no OpenVPN, no zone support, no RADIUS auth. Cannot **upgrade** Basic to VpnGw — must **delete and recreate**. VpnGw1→2→3 can upgrade in-place. Basic is being deprecated for new deployments.

---

## 9. Application Gateway SKU Licensing

| Feature | Standard v2 | WAF v2 |
|---|---|---|
| **Layer** | Layer 7 LB | Layer 7 LB + WAF |
| **Autoscaling** | ✅ | ✅ |
| **Zone redundancy** | ✅ | ✅ |
| **WAF** | ❌ | ✅ (OWASP rules) |
| **Static VIP** | ✅ | ✅ |
| **Min instances** | 0 (scale to zero) | 0 |
| **Pricing** | Fixed + capacity units | Fixed + capacity units (higher) |
| **Header rewrite** | ✅ | ✅ |
| **Custom health probes** | ✅ | ✅ |

### v1 vs v2 (Legacy v1 still on exam)

| Feature | v1 Standard/WAF | v2 Standard/WAF |
|---|---|---|
| **Autoscaling** | ❌ (manual) | ✅ |
| **Zone redundancy** | ❌ | ✅ |
| **Static VIP** | ❌ (dynamic) | ✅ (static) |
| **Performance** | Lower | Higher |
| **Key Vault integration** | ❌ | ✅ |
| **Pricing model** | Instance-based | Capacity unit-based |

> ⚠️ **EXAM TIP:** v2 = autoscaling + zone-redundant + static VIP. v1 = manual scaling, no zones, dynamic VIP. **WAF** tier adds web application firewall (OWASP). v1 → v2 migration is **not in-place** — must create new and migrate.

---

## 10. Storage Account Tier Licensing

### Performance Tiers

| Feature | Standard (HDD) | Premium (SSD) |
|---|---|---|
| **Backing storage** | HDD | SSD |
| **Account types** | GPv2, GPv1, BlobStorage | BlockBlobStorage, FileStorage, GPv2 |
| **Redundancy options** | LRS, ZRS, GRS, RA-GRS, GZRS, RA-GZRS | LRS, ZRS |
| **Services** | Blob, File, Queue, Table | Block Blobs OR Files (depends on type) |
| **Use case** | General purpose / Archive | Low-latency, high IOPS |
| **Price** | Lower | Higher |

### Access Tiers (Blob)

| Tier | Storage Cost | Access Cost | Min Duration | Use Case |
|---|---|---|---|---|
| **Hot** | Highest | Lowest | None | Frequently accessed data |
| **Cool** | Lower | Higher | **30 days** | Infrequently accessed |
| **Cold** | Even lower | Even higher | **90 days** | Rarely accessed |
| **Archive** | Lowest | Highest | **180 days** | Long-term compliance, rarely read |

### Archive Tier — Special Rules

| Rule | Details |
|---|---|
| **Offline tier** | Data is offline — cannot read directly |
| **Rehydrate** | Must rehydrate to Hot or Cool before reading |
| **Rehydration time** | **Standard** = up to 15 hours; **High priority** = <1 hour |
| **Min retention** | 180 days — early delete = charged for remaining days |
| **Set at** | Blob level only (not account level) |
| **Compatible redundancy** | LRS, GRS, RA-GRS only |

### Redundancy Tiers & Pricing

| Redundancy | Copies | Regions | Durability | Cost |
|---|---|---|---|---|
| **LRS** | 3 copies, 1 datacenter | 1 | 99.999999999% (11 9's) | Lowest |
| **ZRS** | 3 copies, 3 zones | 1 | 99.9999999999% (12 9's) | $ |
| **GRS** | 6 copies (3+3) | 2 | 99.99999999999999% (16 9's) | $$ |
| **RA-GRS** | 6 copies, read access secondary | 2 | 16 9's | $$$ |
| **GZRS** | 6 copies (3 zones + 3 secondary) | 2 | 16 9's | $$$$ |
| **RA-GZRS** | 6 copies, read access secondary | 2 | 16 9's | Highest |

> ⚠️ **EXAM TIP:** Archive = **offline**, must rehydrate (standard: 15 hrs, high priority: <1 hr). Min retention: Hot = none, Cool = **30 days**, Cold = **90 days**, Archive = **180 days**. Early delete = charged for remaining days. Premium = **LRS and ZRS only** (no GRS).

---

## 11. Azure Backup Vault Licensing

| Feature | Details |
|---|---|
| **Azure VM Backup** | Per-protected-instance pricing (size-based tiers) |
| **Azure Files Backup** | Per-protected-instance + storage consumed |
| **SQL in VM Backup** | Per-protected-instance (database size) |
| **MARS Agent** | Per-protected-instance (on-prem to Azure) |
| **Soft Delete** | Enabled by default; 14 extra days retention = **no extra charge** |
| **Vault type** | Recovery Services Vault or Backup Vault |

### Backup Pricing Tiers (VM)

| VM Size (Protected Data) | Monthly Cost (approximate) |
|---|---|
| ≤ 50 GB | Lowest tier |
| 50–500 GB | Mid tier |
| > 500 GB | Higher tiers (incremental) |

> ⚠️ **EXAM TIP:** Backup pricing = **per protected instance** + storage consumed. Soft delete = 14 days additional retention at **no cost**. Recovery Services Vault is **free to create** — you pay for protected instances and storage.

---

## 12. Azure Monitor & Log Analytics Licensing

| Feature | Free Tier | Pay-As-You-Go |
|---|---|---|
| **Data ingestion** | **5 GB/month** free per workspace | Per-GB pricing after free tier |
| **Data retention** | **31 days** free | Up to **730 days** (2 years) paid |
| **Basic Logs** | N/A | Lower ingestion cost, limited query |
| **Archived Logs** | N/A | Lowest cost, must restore before query |
| **Commitment Tiers** | N/A | 100, 200, 300, 400, 500+ GB/day (discounted) |
| **Alerts** | Free (limited) | Per-signal, per-condition evaluated |
| **Application Insights** | **5 GB/month free** | Per-GB after |

### Data Retention Tiers

| Tier | Retention | Query | Cost |
|---|---|---|---|
| **Analytics (Interactive)** | 31 days free, up to 730 days | Full KQL query | Highest |
| **Basic Logs** | 8 days | Limited query (basic search) | Lower ingestion |
| **Archive** | Up to 12 years | Must restore first (takes minutes) | Lowest |

### Alert Pricing

| Alert Type | Cost |
|---|---|
| **Metric alerts** | Per-signal/month evaluated |
| **Log alerts** | Per-signal evaluated + frequency |
| **Activity log alerts** | **FREE** |
| **Service Health alerts** | **FREE** |
| **Smart detection (App Insights)** | **FREE** |

> ⚠️ **EXAM TIP:** Log Analytics = **5 GB/month free**, **31 days** free retention. Activity Log alerts and Service Health alerts = **FREE**. Archive = lowest cost but must restore before querying. Commitment tiers give discounts for high-volume ingestion.

---

## 13. Azure Key Vault Licensing

| Feature | Standard | Premium |
|---|---|---|
| **Secrets** | ✅ | ✅ |
| **Keys (software)** | ✅ | ✅ |
| **Keys (HSM-protected)** | ❌ | ✅ |
| **Certificates** | ✅ | ✅ |
| **Pricing** | Per-operation (low) | Per-operation (higher for HSM) |
| **HSM type** | N/A | FIPS 140-2 Level 2 |
| **Managed HSM** | N/A | Separate service (FIPS 140-2 Level 3) |

> ⚠️ **EXAM TIP:** Key Vault **Standard** = software-protected keys. Key Vault **Premium** = HSM-protected keys (hardware). Secrets and certificates available in both tiers. Managed HSM = separate, higher-security service.

---

## 14. Azure App Service Plan Licensing

| Tier | Compute | Custom Domain | SSL | Slots | Auto-scale | VNet Integration | Price |
|---|---|---|---|---|---|---|---|
| **Free (F1)** | Shared | ❌ | ❌ | ❌ | ❌ | ❌ | Free |
| **Shared (D1)** | Shared | ✅ | ❌ | ❌ | ❌ | ❌ | $ |
| **Basic (B1-B3)** | Dedicated | ✅ | ✅ | ❌ | ❌ | ❌ | $$ |
| **Standard (S1-S3)** | Dedicated | ✅ | ✅ | ✅ (5) | ✅ | ✅ (regional) | $$$ |
| **Premium v3 (P1v3-P3v3)** | Dedicated | ✅ | ✅ | ✅ (20) | ✅ | ✅ | $$$$ |
| **Isolated v2 (I1v2-I3v2)** | ASE | ✅ | ✅ | ✅ (20) | ✅ | ✅ (full) | $$$$$ |

### Key Tier Differences

| Feature | Minimum Tier Required |
|---|---|
| Custom domain | **Shared (D1)** |
| SSL/TLS bindings | **Basic (B1)** |
| Deployment slots | **Standard (S1)** |
| Auto-scale (horizontal) | **Standard (S1)** |
| VNet integration | **Standard (S1)** — regional |
| Always On | **Basic (B1)** |
| Azure Hybrid Benefit | **Basic** and above |
| Private Endpoints | **Standard** and above |
| Azure App Service Environment (full isolation) | **Isolated v2** |

> ⚠️ **EXAM TIP:** Deployment slots = **Standard (S1)** minimum. Auto-scale = **Standard (S1)** minimum. SSL = **Basic (B1)** minimum. Custom domains = **Shared (D1)** minimum. Free and Shared = shared infrastructure, no SLA. Isolated = ASE (fully isolated network). This is HEAVILY tested.

---

## 15. Azure DNS Licensing

| Feature | Cost |
|---|---|
| **DNS Zone** | Per-zone per month ($0.50/zone/month first 25) |
| **DNS Queries** | Per-million queries |
| **Private DNS Zone** | Per-zone per month |
| **Traffic Manager** | Per-profile + per-million queries + health checks |

---

## 16. ExpressRoute Licensing

| SKU | Features | Price Level |
|---|---|---|
| **Local** | Access to 1-2 Azure regions near peering location | Lowest (unlimited data) |
| **Standard** | Access to all regions in same geopolitical area | $$ (metered or unlimited) |
| **Premium** | Access to all regions globally + more routes | $$$ (metered or unlimited) |

| Feature | Standard | Premium |
|---|---|---|
| **VNet links** | 10 | 100 |
| **Routes (BGP)** | 4,000 | 10,000 |
| **Global reach** | ❌ | ✅ |
| **Microsoft 365 access** | Limited | ✅ |

> ⚠️ **EXAM TIP:** ExpressRoute **Local** = cheapest, unlimited data, limited to nearby regions. **Standard** = same geopolitical area. **Premium** = global reach + more VNet links (100 vs 10) + more BGP routes (10K vs 4K).

---

## 17. Free vs Paid — Master Reference Table

| Service | Free Component | Paid Component |
|---|---|---|
| **Entra ID** | Free edition (basic user mgmt) | P1/P2 for advanced features |
| **RBAC** | ✅ Completely free | — |
| **Azure Policy** | ✅ Free for Azure resources | Charge for Arc-connected |
| **Management Groups** | ✅ Free | — |
| **Resource Tags** | ✅ Free | — |
| **Cost Management** | ✅ Free | AWS connector = paid |
| **Budgets & Alerts** | ✅ Free | — |
| **Azure Advisor** | ✅ Free | — |
| **Activity Log** | ✅ Free (90-day retention) | Export to Log Analytics = paid |
| **Service Health** | ✅ Free | — |
| **Resource Health** | ✅ Free | — |
| **Azure Monitor metrics** | ✅ Free (platform metrics) | Custom metrics = paid |
| **Log Analytics** | 5 GB/month free | Per-GB after + retention charges |
| **Metric Alerts** | — | Per-signal/month |
| **Activity Log Alerts** | ✅ Free | — |
| **Load Balancer (Basic)** | ✅ Free | — |
| **Load Balancer (Standard)** | — | Per-rule + data processed |
| **Public IP (Basic Dynamic)** | ✅ Free | — |
| **Public IP (Standard/Static)** | — | Per-hour |
| **VPN Gateway** | — | Per-hour + egress data |
| **Application Gateway** | — | Fixed + capacity units |
| **Azure Bastion** | — | Per-hour |
| **Recovery Services Vault** | ✅ Free to create | Per-instance + storage |
| **Key Vault** | — | Per-operation |
| **Network Watcher** | ✅ Free (limited) | Per-check, flow logs = storage |
| **NSG** | ✅ Free | — |
| **UDR/Route Table** | ✅ Free | — |
| **Azure Backup** | — | Per-instance + storage |
| **Azure Site Recovery** | — | Per-instance/month |

> ⚠️ **EXAM TIP:** Completely FREE: RBAC, Policy, Tags, MGs, Budgets, Advisor, Service Health, Activity Log Alerts, NSGs, Route Tables, Basic LB, Basic Dynamic PIP. Know which services are free vs paid — exam tests this frequently.

---

## 18. Quick-Fire Exam Points ⚡

1. **Entra ID Free** = basic user/group mgmt, SSO (10 apps), Security Defaults MFA
2. **Entra ID P1** = Conditional Access, Dynamic Groups, SSPR writeback, Application Proxy
3. **Entra ID P2** = PIM, Access Reviews, Identity Protection, Risk-based CA, Entitlement Mgmt
4. **Conditional Access** requires **P1** minimum
5. **PIM** (just-in-time access) requires **P2**
6. **Access Reviews** require **P2**
7. **Risk-based Conditional Access** requires **P2** (Identity Protection)
8. **Security Defaults** = free MFA for all users; mutually exclusive with Conditional Access
9. **Azure Hybrid Benefit** (Windows) = up to **40%** savings; requires SA
10. **Azure Hybrid Benefit** (SQL) = up to **55%** savings; requires SA
11. AHB + Reserved Instances = up to **85%** combined savings
12. **Spot VMs** = up to **90%** savings, **no SLA**, evictable anytime
13. Spot eviction policy: **Deallocate** (default) or **Delete**
14. **Reserved Instances** = up to **72%** savings, 1/3 year term, can exchange/cancel
15. **Savings Plans** = up to **65%** savings, 1/3 year, **cannot** exchange/cancel
16. Discount order: **Reservations → Savings Plans → PAYG**
17. **Basic LB** = free, no SLA, no zones, open by default, max 300 backend
18. **Standard LB** = paid, 99.99% SLA, zone-redundant, **closed** by default, max 1,000 backend
19. Standard LB requires **Standard Public IP** — cannot mix Basic/Standard
20. **Basic PIP** = dynamic or static, no zones, open by default
21. **Standard PIP** = static only, zone-redundant, **closed** by default
22. **VPN Gateway Basic** SKU = no IKEv2, no OpenVPN, no zones, no RADIUS — legacy
23. Basic → VpnGw upgrade = **must delete and recreate** gateway
24. VpnGw1 → VpnGw2 → VpnGw3 = can upgrade **in-place**
25. **App Gateway v2** = autoscaling + zone-redundant + static VIP
26. **App Gateway v1** = manual scaling, no zones, dynamic VIP
27. **WAF tier** adds OWASP web application firewall rules
28. **Storage Hot** = highest storage cost, lowest access cost
29. **Storage Archive** = lowest storage cost, offline, must **rehydrate** (15 hrs standard)
30. Archive min retention = **180 days**; Cool = **30 days**; Cold = **90 days**
31. **Premium Storage** = LRS and ZRS only (no GRS)
32. **App Service Free/Shared** = shared compute, no SLA, no SSL
33. **App Service Basic** = dedicated compute, SSL, Always On
34. **App Service Standard** = deployment slots, auto-scale, VNet integration
35. Deployment slots minimum = **Standard (S1)**
36. Auto-scale minimum = **Standard (S1)**
37. App Service Isolated = **ASE** = full network isolation
38. **Key Vault Standard** = software keys; **Premium** = HSM-protected keys
39. **ExpressRoute Local** = cheapest, limited regions, unlimited data
40. **ExpressRoute Premium** = global reach, 100 VNet links, 10K BGP routes
41. **Log Analytics** = 5 GB/month free, 31 days free retention
42. **Activity Log alerts** = FREE; **Metric alerts** = per-signal charge
43. **Azure Backup** = per-instance + storage; vault creation = free
44. **Soft delete** (backup) = 14 extra days at **no charge**
45. RBAC, Policy, Tags, MGs, Budgets, Advisor, NSGs, Route Tables = **all FREE**
46. **Dev/Test** subscription pricing = ~55% savings on VMs (no Windows license charge)
47. **Dynamic Groups** require **P1** license
48. **Application Proxy** requires **P1** license
49. Recovery Services Vault free to create — pay for protected instances
50. **Standard LB, Standard PIP** = closed by default (must add NSG to allow traffic)

---

## 19. Step-by-Step Configuration Mind Maps 🗺️

---

### 19.1 Assign Entra ID Licenses

> **Portal:** `Microsoft Entra ID → Licenses`

```
Assign Entra ID Licenses
│
├── Step 1: Navigate
│   └── Microsoft Entra ID → Licenses → All products
│
├── Step 2: Select License
│   ├── Microsoft Entra ID P1
│   ├── Microsoft Entra ID P2
│   ├── EMS E3 / E5 (includes P1/P2)
│   └── Microsoft 365 E3 / E5 (includes P1/P2)
│
├── Step 3: Assign
│   ├── + Assign → + Add users and groups
│   ├── Select users or groups
│   │   └── ⚠️ Group-based licensing requires P1+
│   ├── Assignment options: toggle individual services on/off
│   └── Assign
│
├── Step 4: Verify
│   └── Entra ID → Users → Select user → Licenses → View assigned
│
├── Required Role: License Administrator or Global Administrator
│
└── ⚠️ Key Points
    ├── Group-based licensing = P1 feature
    ├── License required per USER (not per device)
    ├── P2 needed for PIM, Access Reviews, Identity Protection
    ├── Some features (MFA Security Defaults) work without any license
    └── Check "Licensed users" report for compliance
```

---

### 19.2 Enable Azure Hybrid Benefit on VM

> **Portal:** `Virtual Machine → Configuration → Licensing`

```
Enable Azure Hybrid Benefit
│
├── Prerequisites
│   ├── Valid Windows Server license with Software Assurance
│   ├── Standard (16-core) = 2 VMs with ≤8 vCPUs each
│   └── Datacenter (16-core) = unlimited VMs on dedicated host
│
├── For New VM:
│   ├── VM → Create → Basics → ... →
│   │   Licensing section → "Would you like to use an existing
│   │   Windows Server license?" → ✅ Yes → Create
│   └── CLI: az vm create --license-type Windows_Server
│
├── For Existing VM:
│   ├── VM → Configuration → Licensing →
│   │   "Azure Hybrid Benefit" → Select "I confirm..." → Save
│   └── CLI: az vm update --set licenseType=Windows_Server
│
├── For SQL:
│   ├── SQL Database → Compute + storage →
│   │   "Already have a SQL Server license?" → Yes → Apply
│   └── SQL MI → Compute + storage → Azure Hybrid Benefit → Apply
│
├── Required Role: Contributor+ on the VM/SQL resource
│
└── ⚠️ Key Points
    ├── Windows AHB savings = up to 40%
    ├── SQL AHB savings = up to 55%
    ├── Can combine with Reserved Instances (up to 85%)
    ├── Must have Software Assurance or equivalent
    ├── Self-attestation — Azure trusts you have the license
    └── Can toggle on/off anytime (no lock-in)
```

---

### 19.3 Choose Storage Access Tier

> **Portal:** `Storage Account → Blob → Change tier`

```
Choose Storage Access Tier
│
├── Account-Level Default Tier
│   ├── Storage Account → Configuration →
│   │   Default access tier: Hot / Cool / Cold → Save
│   └── ⚠️ This sets the DEFAULT for new blobs (can override per-blob)
│
├── Blob-Level Tier Change
│   ├── Storage Account → Containers → Select blob →
│   │   Change tier → Hot / Cool / Cold / Archive → Save
│   └── ⚠️ Archive = blob goes OFFLINE immediately
│
├── Rehydrate from Archive
│   ├── Blob → Change tier → Hot or Cool →
│   │   Priority: Standard (up to 15 hrs) / High (<1 hr)
│   └── ⚠️ High priority = higher cost
│
├── Lifecycle Management (automated tiering)
│   ├── Storage Account → Data management → Lifecycle management →
│   │   + Add rule → Name →
│   │   Conditions:
│   │     ├── Days since modification (e.g., >30 → Cool)
│   │     ├── Days since modification (e.g., >90 → Cold)
│   │     ├── Days since modification (e.g., >180 → Archive)
│   │     └── Days since modification (e.g., >365 → Delete)
│   └── Filters: blob prefix, blob type → Add
│
├── Required Role: Storage Account Contributor / Storage Blob Data Contributor
│
└── ⚠️ Key Points
    ├── Early deletion charges apply (30/90/180 day minimums)
    ├── Archive = offline — must rehydrate before read
    ├── Lifecycle rules = automated cost optimization
    ├── Premium = NO access tiers (always hot-like performance)
    └── Account-level default ≠ blob-level (blob overrides account)
```

---

### 19.4 Choose App Service Plan Tier

> **Portal:** `App Service Plan → Scale up (App Service plan)`

```
Choose App Service Plan Tier
│
├── Step 1: Navigate
│   └── App Service Plan → Scale up (App Service plan)
│
├── Step 2: Select Category
│   ├── Dev/Test: Free (F1) / Shared (D1) / Basic (B1)
│   │   └── For development, testing, non-production
│   ├── Production: Standard (S1-S3) / Premium v3 (P1v3-P3v3)
│   │   └── For production workloads
│   └── Isolated: Isolated v2 (I1v2-I3v2)
│       └── For ASE, full network isolation
│
├── Step 3: Select Size
│   ├── View CPU, Memory, Storage per size
│   ├── View included features per tier
│   └── Select → Apply
│
├── Scale Out (Horizontal — separate)
│   └── App Service Plan → Scale out →
│       Manual: Set instance count
│       Auto: Rules based on metrics (Standard+)
│
├── Required Role: Contributor+ on App Service Plan
│
└── ⚠️ Key Points
    ├── Free/Shared = NO SLA, shared compute
    ├── Basic = dedicated, SSL, Always On, NO slots
    ├── Standard = slots (5), auto-scale, VNet integration
    ├── Premium v3 = slots (20), better hardware
    ├── Isolated v2 = ASE, full isolation
    ├── Scale UP = change tier (vertical)
    ├── Scale OUT = add instances (horizontal)
    └── Scaling up/down = no downtime
```


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
