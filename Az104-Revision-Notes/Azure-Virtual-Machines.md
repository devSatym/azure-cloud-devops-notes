<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Virtual Machines — AZ-104 Revision Notes

---

## 1. What is an Azure VM?

- **IaaS compute resource** — full control over OS, runtime, applications
- Runs **Windows** or **Linux** guest operating systems
- Deployed into a **VNet subnet** with a **NIC**
- Requires a **resource group**, **region**, **VM size**, and **OS image**
- Billed **per-second** (compute) when running; stopped-deallocated = no compute charges
- Supports **Availability Sets**, **Availability Zones**, and **VMSS** for high availability

> ⚠️ **EXAM TIP:** A **stopped (deallocated)** VM incurs NO compute charges but you still pay for **disks** and **public IPs (Standard SKU)**. A VM that is only "stopped" from inside the OS (not deallocated) **still incurs compute charges**.

---

## 2. VM Size Families

| Series | Optimized For | Use Case |
|---|---|---|
| **B** | Burstable | Dev/test, low-traffic web servers |
| **D / Dv2-v5** | General purpose | Most workloads, balanced CPU/memory |
| **E / Ev2-v5** | Memory optimized | Databases, in-memory caching |
| **F / Fv2** | Compute optimized | Batch processing, gaming, analytics |
| **L** | Storage optimized | Big data, SQL, NoSQL, data warehousing |
| **M** | Memory optimized (large) | Very large in-memory workloads (SAP HANA) |
| **N** | GPU | Machine learning, graphics rendering, video |
| **H** | HPC | High-performance computing |

### VM Size Naming Convention
```
Example: Standard_D4s_v5
         │        ││ │
         │        ││ └── Version
         │        │└──── "s" = Premium SSD support
         │        └───── 4 vCPUs
         └────────────── Family (D = General purpose)
```

> ⚠️ **EXAM TIP:** The **"s"** in VM size (e.g., D4**s**_v5) means the VM supports **Premium SSD** storage. Without "s", only Standard disks.

> ⚠️ **EXAM TIP:** **B-series** VMs accumulate CPU credits during low usage and burst above baseline during peaks. Other series have fixed CPU allocation.

---

## 3. VM States

| State | Compute Billing | Disk Billing |
|---|---|---|
| **Running** | ✅ Charged | ✅ Charged |
| **Stopped (from OS)** | ✅ Charged | ✅ Charged |
| **Stopped (Deallocated)** | ❌ Not charged | ✅ Charged |
| **Deleted** | ❌ | Depends on delete settings |

> ⚠️ **EXAM TIP:** "Stopped" from inside the OS ≠ "Deallocated." Only **Stop (Deallocate)** from Azure Portal/CLI stops compute billing. OS-level shutdown still charges compute.

---

## 4. Azure Managed Disks

### Disk Types

| Disk Type | IOPS (Max) | Throughput | Use Case |
|---|---|---|---|
| **Ultra Disk** | 160,000 | 4,000 MB/s | Mission-critical (SAP, SQL, top-tier) |
| **Premium SSD v2** | 80,000 | 1,200 MB/s | Production, customizable IOPS/throughput |
| **Premium SSD (P)** | 20,000 | 900 MB/s | Production workloads |
| **Standard SSD (E)** | 6,000 | 750 MB/s | Web servers, dev/test |
| **Standard HDD (S)** | 2,000 | 500 MB/s | Backup, non-critical, infrequent access |

### Disk Roles

| Role | Description | Required? |
|---|---|---|
| **OS Disk** | Contains the operating system | ✅ Yes (always 1) |
| **Data Disk** | Additional storage for applications/data | Optional (0 or more) |
| **Temporary Disk** | Local, ephemeral (D: Windows / /dev/sdb Linux) | Most VMs (not all) |

> ⚠️ **EXAM TIP:** **Temporary disk** data is **lost** on VM deallocation/maintenance. NEVER store important data on temp disk. It is local SSD, not a managed disk.

> ⚠️ **EXAM TIP:** Number of data disks depends on **VM size**. E.g., Standard_D2s_v5 = 4 data disks, Standard_D16s_v5 = 32 data disks.

### Disk Encryption Options

| Option | Description |
|---|---|
| **SSE with PMK** | Server-side encryption with platform-managed keys (default, automatic) |
| **SSE with CMK** | Server-side encryption with customer-managed keys (Key Vault) |
| **Azure Disk Encryption (ADE)** | OS-level encryption — BitLocker (Windows) / DM-Crypt (Linux) |
| **Encryption at Host** | Encrypts temp disk and disk caches on the host |
| **Confidential Disk Encryption** | VM confidential computing with encrypted OS disk |

> ⚠️ **EXAM TIP:** **SSE (PMK)** is enabled **by default** on ALL managed disks — no action needed. **ADE** uses BitLocker/DM-Crypt and requires **Key Vault** with access policy enabled for disk encryption.

#### Portal Path — Disk Encryption
```
VM → Disks → Select disk → Encryption type →
SSE with PMK / SSE with CMK / Encryption at host
```

#### Portal Path — Azure Disk Encryption
```
VM → Disks → Additional settings →
Encryption → Azure Disk Encryption → Enable →
Key Vault → Select/Create → Save
```

---

## 5. Availability Options

### 5.1 Availability Sets

- Logical grouping of VMs within a datacenter
- **Fault Domains (FD):** Separate physical racks (power + network) — max **3** FDs
- **Update Domains (UD):** Groups rebooted together during maintenance — max **20** UDs (default **5**)
- SLA: **99.95%** (with 2+ VMs)
- **Free** — no extra cost for the Availability Set

| Setting | Range | Default |
|---|---|---|
| Fault Domains | 1–3 | **2** |
| Update Domains | 1–20 | **5** |

> ⚠️ **EXAM TIP:** Availability Set = **99.95% SLA**. Max **3 FDs**, max **20 UDs** (default 5). VMs must be added to Availability Set **at creation time** — cannot add later.

> ⚠️ **EXAM TIP:** You **CANNOT** move an existing VM into an Availability Set. You must create a new VM and specify the Availability Set during creation.

### 5.2 Availability Zones

- Physically separate datacenters within a region (independent power, cooling, network)
- Each region with zones has **minimum 3 zones** (Zone 1, 2, 3)
- SLA: **99.99%** (with 2+ VMs across 2+ zones)
- Protection against **entire datacenter failure**

| Feature | Availability Set | Availability Zone |
|---|---|---|
| **Protection** | Rack-level (FD/UD) | Datacenter-level |
| **SLA** | 99.95% | 99.99% |
| **Scope** | Single datacenter | Multiple datacenters in a region |
| **Fault Domains** | Up to 3 | Each zone = 1 FD |
| **Cost** | Free | Free (but cross-zone data transfer charged) |
| **Can add VM later** | ❌ Must be at creation | ❌ Must be at creation |

> ⚠️ **EXAM TIP:** **Availability Zones** = higher SLA (99.99%) than Sets (99.95%). Zones protect against datacenter failure; Sets protect against rack failure.

> ⚠️ **EXAM TIP:** Single VM with **Premium SSD** for all disks = **99.9% SLA** (no Availability Set/Zone needed).

### 5.3 Single VM SLA

| Configuration | SLA |
|---|---|
| Single VM + **Premium SSD** (all disks) | **99.9%** |
| Single VM + **Ultra Disk** (all disks) | **99.9%** |
| Availability Set (2+ VMs) | **99.95%** |
| Availability Zones (2+ VMs, 2+ zones) | **99.99%** |

---

## 6. VM Scale Sets (VMSS)

- Deploy and manage a **group of identical VMs**
- Supports **auto-scaling** (scale out/in) based on metrics or schedule
- Two orchestration modes: **Uniform** and **Flexible**

### Orchestration Modes

| Feature | Uniform | Flexible |
|---|---|---|
| **VM model** | Identical (same template) | Mix of VM sizes/configs |
| **Availability** | 5 FDs (default) | Up to 5 FDs spread |
| **Scaling** | Auto-scale supported | Auto-scale supported |
| **Mix VM sizes** | ❌ | ✅ |
| **Availability Zones** | ✅ | ✅ |
| **Load Balancer** | ✅ | ✅ |
| **Recommended** | Legacy | ✅ Preferred for new |

### Auto-Scale Settings

| Setting | Description |
|---|---|
| **Minimum instances** | Floor — never scale below this |
| **Maximum instances** | Ceiling — never scale above this |
| **Default instances** | Starting count when no metrics available |
| **Scale-out rule** | Add instances when metric exceeds threshold |
| **Scale-in rule** | Remove instances when metric drops below threshold |
| **Cool-down period** | Wait time after scaling before evaluating again (default: **5 minutes**) |

### Scale Metrics (Common)

| Metric | Typical Threshold |
|---|---|
| **CPU %** | Scale out > 75%, Scale in < 25% |
| **Memory %** | Scale out > 80% |
| **Disk queue depth** | Scale out > 5 |
| **Network In/Out** | Custom thresholds |
| **Schedule-based** | Time-of-day or day-of-week |

#### Portal Path — Create VMSS
```
Home → Virtual machine scale sets → + Create →
Orchestration mode: Uniform/Flexible →
Image, Size, Instance count →
Scaling: Manual / Custom autoscale →
Rules → Create
```

> ⚠️ **EXAM TIP:** Auto-scale **cool-down** default = **5 minutes**. This prevents "flapping" — scaling in/out too rapidly. If exam asks "VMs keep scaling up and down" → increase cool-down period.

> ⚠️ **EXAM TIP:** VMSS can scale to **1,000 instances** (with custom image) or **600** (with marketplace image) in Uniform mode.

---

## 7. VM Extensions

- **Post-deployment configuration** and automation on VMs
- Installed on VMs to add functionality
- Run scripts, install software, configure VMs

### Key Extensions for AZ-104

| Extension | Purpose |
|---|---|
| **Custom Script Extension** | Run scripts (PowerShell/Bash) on VM after deployment |
| **DSC Extension** | Desired State Configuration (Windows) |
| **Azure Disk Encryption** | Enable BitLocker (Win) / DM-Crypt (Linux) |
| **Network Watcher Agent** | Packet capture, connection monitor |
| **Log Analytics Agent (MMA)** | Send logs to Log Analytics workspace (legacy) |
| **Azure Monitor Agent (AMA)** | Send logs/metrics to Azure Monitor (current) |
| **BGInfo** | Display VM info on desktop (Windows) |
| **Diagnostics Extension** | Boot diagnostics, guest-level metrics |

### Custom Script Extension

- **Windows:** Downloads and executes PowerShell scripts
- **Linux:** Downloads and executes shell scripts
- Script source: **Storage Account**, **GitHub URL**, or **inline**
- Timeout: **90 minutes** for script execution
- Runs **once** — for ongoing config use DSC
- Only **one** Custom Script Extension per VM at a time

#### Portal Path — Add Extension
```
VM → Extensions + applications → + Add →
Select extension → Configure → Create
```

#### CLI — Custom Script Extension
```bash
az vm extension set --vm-name <vm> -g <rg> \
  --name CustomScriptExtension \
  --publisher Microsoft.Compute \
  --settings '{"fileUris":["https://url/script.ps1"],"commandToExecute":"powershell -ExecutionPolicy Unrestricted -File script.ps1"}'
```

> ⚠️ **EXAM TIP:** Custom Script Extension has a **90-minute timeout**. Only **one** Custom Script Extension per VM. If you need to run a new script, remove the old extension first.

---

## 8. VM Networking

### NIC Configuration
- Every VM must have **at least 1 NIC**
- Number of NICs depends on **VM size** (e.g., D2 = 2 NICs, D4 = 4 NICs)
- Each NIC connects to a **subnet** in a VNet
- NIC has **private IP** (required) and optional **public IP**
- **Accelerated Networking** — SR-IOV for lower latency (supported on most sizes)

### IP Configuration

| Type | Allocation | Notes |
|---|---|---|
| **Private IP - Dynamic** | DHCP from subnet (default) | Same IP on restart if not deallocated |
| **Private IP - Static** | You specify IP | Must be in subnet range |
| **Public IP - Dynamic** | Assigned when VM starts | Released on deallocation |
| **Public IP - Static** | Fixed, always assigned | Persists through deallocation |

> ⚠️ **EXAM TIP:** Dynamic private IP stays the **same** across stop/start cycles — only changes if VM is **deleted and recreated**. But dynamic **public** IP changes every time VM is deallocated.

> ⚠️ **EXAM TIP:** Standard Public IP = **secure by default** — requires NSG to allow inbound traffic. Basic Public IP = open by default (retiring).

#### Portal Path — Configure NIC
```
VM → Networking → Network interface → Click NIC →
IP configurations → Select → Static/Dynamic → Save
```

---

## 9. VM Connection Methods

| Method | Port | Protocol | Best For |
|---|---|---|---|
| **RDP** | **3389** | TCP | Windows VMs |
| **SSH** | **22** | TCP | Linux VMs |
| **Azure Bastion** | **443** | TLS | Secure access without public IP |
| **Serial Console** | N/A | Direct | Troubleshoot boot/network issues |
| **Run Command** | N/A | Agent | Execute scripts without connecting |

### Azure Bastion
- Secure RDP/SSH over **TLS (port 443)** from Azure Portal
- VM does NOT need a **public IP**
- Requires **AzureBastionSubnet** (minimum **/26**)
- SKUs: Basic, Standard, Premium

### Just-In-Time (JIT) VM Access
- Opens management ports (RDP/SSH) **only when needed**, for a limited time
- Requires **Microsoft Defender for Servers** (Plan 1 or 2)
- Creates temporary NSG rules to allow access
- Default time: configurable (max **24 hours**)
- Reduces attack surface by keeping ports closed

#### Portal Path — JIT Access
```
VM → Connect → Select → Request access →
(or) Microsoft Defender for Cloud → Just in time VM access
```

> ⚠️ **EXAM TIP:** JIT requires **Microsoft Defender for Servers**. It creates temporary **NSG allow rules** that auto-expire. If JIT is not available, check if Defender is enabled.

---

## 10. Boot Diagnostics

- Captures **screenshot** and **serial console output** during VM boot
- Helps troubleshoot boot failures
- Stored in **managed storage account** (default) or custom storage account
- **Enabled by default** on new VMs

#### Portal Path
```
VM → Help → Boot diagnostics → Screenshot / Serial log
```

```
VM → Settings → Boot diagnostics →
Enable with managed storage account / custom storage account
```

---

## 11. VM Redeploy & Reapply

| Action | What It Does |
|---|---|
| **Restart** | Restarts VM on same host |
| **Redeploy** | Moves VM to a **new host** (new hardware, same config). Temp disk data lost |
| **Reapply** | Re-applies VM configuration (provisions again on same host) |

#### Portal Path
```
VM → Help → Redeploy + Reapply → Redeploy / Reapply
```

> ⚠️ **EXAM TIP:** **Redeploy** = new host (use when VM is unresponsive). Temp disk data is **lost**. Data disks are preserved.

---

## 12. VM Backup & Disaster Recovery

### Azure Backup for VMs
- **Application-consistent** snapshots (Windows) / **file-consistent** (Linux)
- Stored in **Recovery Services Vault**
- Backup frequency: Every **4 hours** to once daily
- Retention: 1 day to **9,999 days** (~27 years)
- Supports both **managed** and **unmanaged** disks
- Instant Restore: restore from **snapshot tier** (1–5 days retention)

#### Portal Path
```
VM → Backup → Recovery Services vault → Select/Create →
Backup policy → Select/Create → Enable backup
```

### Azure Site Recovery (ASR)
- **Disaster recovery** — replicate VMs to another region
- RPO: as low as **30 seconds**
- Supports: Azure-to-Azure, VMware-to-Azure, Hyper-V-to-Azure
- Uses **Recovery Services Vault** in the target region

#### Portal Path
```
VM → Disaster recovery →
Target region → Replication settings → Enable replication
```

---

## 13. Security & RBAC

### Key RBAC Roles

| Role | Permissions |
|---|---|
| **Virtual Machine Contributor** | Full VM management, CANNOT manage VNet or storage |
| **Virtual Machine Administrator Login** | View + login as admin (RDP admin / sudo) |
| **Virtual Machine User Login** | View + login as regular user |
| **Owner / Contributor** | Full access |
| **Reader** | View-only |

### VM Login Roles (Azure AD / Entra ID Authentication)

| Role | Access Level |
|---|---|
| **VM Administrator Login** | Login with **admin** privileges |
| **VM User Login** | Login with **standard user** privileges |

> ⚠️ **EXAM TIP:** **Virtual Machine Contributor** can manage VMs but **CANNOT** manage the VNet or Storage accounts. It also **cannot assign roles** (need Owner for that).

> ⚠️ **EXAM TIP:** For **Azure AD login** to VMs, assign **VM Administrator Login** or **VM User Login** role, AND the VM must have the AADLoginForWindows/Linux extension.

---

## 14. Update Management

- **Azure Update Manager** (replacement for legacy Update Management in Automation Account)
- Assess, schedule, and deploy OS patches for Windows and Linux VMs
- No Log Analytics agent required (uses Azure VM Agent)
- Supports on-demand or scheduled patching

### Patch Orchestration Options

| Mode | Description |
|---|---|
| **Azure-orchestrated** | Azure manages patching schedule |
| **AutomaticByPlatform** | Azure auto-patches based on availability |
| **AutomaticByOS** | OS handles updates (Windows Update) |
| **Manual** | Admin applies patches manually |

#### Portal Path
```
VM → Updates → Assess now / Schedule updates
```

```
Azure Update Manager → Machines →
Select VMs → Schedule updates → Maintenance configuration
```

---

## 15. Monitoring & Alerts

### Key Metrics

| Metric | Description |
|---|---|
| **Percentage CPU** | CPU utilization |
| **Available Memory Bytes** | Free RAM (requires guest agent) |
| **Disk Read/Write Operations/Sec** | IOPS |
| **Disk Read/Write Bytes** | Throughput |
| **Network In/Out Total** | Network traffic |
| **OS Disk Queue Depth** | Pending IO requests |

#### Portal Path — Metrics
```
VM → Monitoring → Metrics →
Metric: Percentage CPU → Add filter → Apply
```

#### Portal Path — Alerts
```
VM → Monitoring → Alerts → + Create alert rule →
Signal: Percentage CPU → Condition: > 80% → 
Action Group → Create
```

### VM Insights
- Requires **Azure Monitor Agent** + **Data Collection Rule**
- Provides performance trends, dependency mapping
- Shows top processes, network connections

#### Portal Path
```
VM → Monitoring → Insights → Enable →
Select Log Analytics workspace → Enable
```

---

## 16. Pricing Key Points

| Component | Billing Model |
|---|---|
| **VM Compute** | Per-second, based on VM size (only when running) |
| **Managed Disks** | Per-disk per-month (even when VM is stopped) |
| **Standard Public IP** | Per-hour (even when VM is stopped) |
| **Data Transfer** | Outbound egress charged; inbound free |
| **Snapshots** | Per GB per month |
| **Bastion** | Per hour + data transfer |
| **VMSS** | No extra cost (pay for underlying VMs) |

### Cost Saving Options

| Option | Savings | Commitment |
|---|---|---|
| **Reserved Instances** | Up to **72%** off | 1 or 3 years |
| **Azure Spot VMs** | Up to **90%** off | Can be evicted anytime |
| **Azure Hybrid Benefit** | Save on OS license | Bring existing Windows Server / SQL license |
| **Dev/Test pricing** | Reduced rates | Dev/Test subscription |
| **Auto-shutdown** | Stop VMs off-hours | Schedule-based |

> ⚠️ **EXAM TIP:** **Spot VMs** can be **evicted** at any time when Azure needs capacity. Eviction policy: **Stop-Deallocate** (default) or **Delete**. Not for production.

> ⚠️ **EXAM TIP:** **Reserved Instances** = 1 or 3 year commitment for up to 72% savings. Can be exchanged or cancelled (with fees). Payment: upfront, monthly, or no upfront.

> ⚠️ **EXAM TIP:** **Azure Hybrid Benefit** = bring your existing Windows Server license with Software Assurance to save on VM OS costs. Also works for SQL Server.

#### Portal Path — Auto-Shutdown
```
VM → Operations → Auto-shutdown →
Enable: Yes → Time → Time zone → Notification → Save
```

#### Portal Path — Spot VM
```
Create VM → Basics → Azure Spot instance: ✅ →
Eviction type: Capacity only / Price or capacity →
Eviction policy: Stop-Deallocate / Delete →
Max price: Set or -1 (pay-as-you-go)
```

---

## 17. Limitations & Constraints

| Constraint | Limit |
|---|---|
| VMs per subscription per region | **25,000** |
| VMs per Availability Set | **200** |
| VMSS max instances (Uniform) | **1,000** (custom image) / **600** (marketplace) |
| VMSS max instances (Flexible) | **1,000** |
| Data disks per VM | **Depends on VM size** (4 to 64) |
| NICs per VM | **Depends on VM size** (1 to 8) |
| Availability Set FDs | **Max 3** |
| Availability Set UDs | **Max 20** (default 5) |
| VM name max length | **64 characters** (Linux), **15 characters** (Windows hostname) |
| OS disk max size | **4 TiB** (4,095 GiB) |
| Data disk max size | **32 TiB** (Premium SSD v2 / Ultra) |

---

## 18. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| Create VM | `az vm create -g <rg> -n <name> --image Ubuntu2204 --size Standard_D2s_v5 --admin-username azureuser --generate-ssh-keys` |
| List VMs | `az vm list -g <rg> -o table` |
| Start VM | `az vm start -g <rg> -n <name>` |
| Stop (deallocate) | `az vm deallocate -g <rg> -n <name>` |
| Restart | `az vm restart -g <rg> -n <name>` |
| Delete | `az vm delete -g <rg> -n <name>` |
| Resize | `az vm resize -g <rg> -n <name> --size Standard_D4s_v5` |
| List sizes | `az vm list-sizes -l eastus` |
| List available sizes for VM | `az vm list-vm-resize-options -g <rg> -n <name>` |
| Add data disk | `az vm disk attach -g <rg> --vm-name <name> --name <disk> --new --size-gb 128 --sku Premium_LRS` |
| Run command | `az vm run-command invoke -g <rg> -n <name> --command-id RunPowerShellScript --scripts "Get-Service"` |
| Open port | `az vm open-port -g <rg> -n <name> --port 80` |
| Show Ip | `az vm list-ip-addresses -g <rg> -n <name>` |

### PowerShell

| Action | Command |
|---|---|
| Create VM | `New-AzVM -ResourceGroupName <rg> -Name <name> -Location eastus -Image Ubuntu2204 -Size Standard_D2s_v5` |
| Start | `Start-AzVM -ResourceGroupName <rg> -Name <name>` |
| Stop (dealloc) | `Stop-AzVM -ResourceGroupName <rg> -Name <name>` |
| Restart | `Restart-AzVM -ResourceGroupName <rg> -Name <name>` |
| Resize | `$vm = Get-AzVM ...; $vm.HardwareProfile.VmSize = "Standard_D4s_v5"; Update-AzVM -VM $vm ...` |
| Run command | `Invoke-AzVMRunCommand -ResourceGroupName <rg> -VMName <name> -CommandId RunPowerShellScript -ScriptString "Get-Service"` |

> ⚠️ **EXAM TIP:** `az vm deallocate` = stop + deallocate (stops billing). `az vm stop` = OS shutdown only (billing continues). Always use **deallocate** to save costs.

---

## 19. Quick-Fire Exam Points ⚡

1. VM is **IaaS** — full OS control, billed **per-second** when running
2. **Stopped (Deallocated)** = no compute cost. **Stopped (OS-level)** = STILL billed
3. Disks and Standard Public IPs are billed **even when VM is deallocated**
4. **B-series** = burstable (accumulates CPU credits). All others = fixed CPU
5. **"s" in VM size** (e.g., D4**s**_v5) = supports **Premium SSD**
6. Azure reserves **5 IPs per subnet** (first 4 + last)
7. **Temp disk** data lost on deallocation/maintenance — never store critical data
8. Max data disks depends on **VM size** — not configurable separately
9. **SSE with PMK** = default disk encryption on ALL managed disks (automatic)
10. **ADE** = OS-level encryption (BitLocker/DM-Crypt), requires **Key Vault**
11. **Availability Set**: max 3 FDs, max 20 UDs (default 5), SLA = **99.95%**
12. **Availability Zones**: 3+ zones per region, SLA = **99.99%**
13. Single VM + Premium SSD = **99.9%** SLA
14. VM must be in Availability Set/Zone **at creation time** — cannot add later
15. **Custom Script Extension**: 90-min timeout, one per VM, runs once
16. Only **one Custom Script Extension** per VM at a time
17. VMSS Uniform max instances: **1,000** (custom) / **600** (marketplace)
18. Auto-scale cool-down default: **5 minutes**
19. **Spot VM** = up to 90% savings but can be **evicted anytime**
20. **Reserved Instance** = 1 or 3 year, up to **72%** savings
21. **Azure Hybrid Benefit** = bring Windows Server/SQL license with SA
22. **Redeploy** = move VM to new host (temp disk lost, data disks preserved)
23. VM Connection: RDP = **3389**, SSH = **22**, Bastion = **443**
24. **JIT Access** requires **Defender for Servers** — creates temporary NSG rules
25. **VM Contributor** can manage VMs but NOT VNet or storage
26. **VM Admin Login** / **VM User Login** = Azure AD authentication to VM
27. VMs per subscription per region: **25,000**
28. VMs per Availability Set: **200**
29. OS disk max: **4 TiB**. Data disk max: **32 TiB**
30. Dynamic Private IP stays same across restarts — changes only if VM is recreated
31. Dynamic Public IP **changes** on every deallocation — use Static to preserve
32. `az vm deallocate` = stop billing. `az vm stop` = still billing
33. Boot diagnostics enabled **by default** — helps troubleshoot boot failures
34. Number of NICs per VM depends on **VM size** (1 to 8)
35. **Accelerated Networking** = SR-IOV for lower latency, supported on most sizes

---

## 20. Step-by-Step Configuration Mind Maps 🗺️

---

### 20.1 Create a Virtual Machine

> **Portal:** `Home → Virtual Machines → + Create → Azure virtual machine`

```
Create Virtual Machine
│
├── Basics
│   ├── Subscription
│   ├── Resource Group (select/create)
│   ├── VM Name (⚠️ Windows hostname max 15 chars)
│   ├── Region (⚠️ determines available sizes and features)
│   ├── Availability options:
│   │   ├── No infrastructure redundancy (single VM)
│   │   ├── Availability zone: Select Zone 1/2/3
│   │   │   ⚠️ 99.99% SLA with 2+ VMs across 2+ zones
│   │   ├── Availability set: Select/Create
│   │   │   ⚠️ 99.95% SLA | Max 3 FDs, 20 UDs
│   │   │   ⚠️ Cannot change after creation
│   │   └── Virtual machine scale set
│   ├── Security type: Standard / Trusted Launch / Confidential
│   ├── Image: Select OS (Windows/Linux)
│   ├── VM Architecture: x64 / Arm64
│   ├── Size: Select VM size
│   │   ⚠️ "s" suffix = Premium SSD support
│   ├── Azure Spot instance: Yes/No
│   │   ⚠️ Can be evicted — not for production
│   ├── Administrator account:
│   │   ├── Windows: Username + Password
│   │   └── Linux: Username + SSH key / Password
│   └── Inbound port rules:
│       ├── Public inbound ports: None / Allow (SSH/RDP/HTTP/HTTPS)
│       └── ⚠️ Opening ports creates NSG rules — restrict in production
│
├── Disks
│   ├── OS disk type: Premium SSD / Standard SSD / Standard HDD
│   ├── OS disk size: Default or custom
│   ├── Encryption type: PMK / CMK / Encryption at host
│   ├── Delete with VM: Yes/No
│   │   ⚠️ If No, disk persists after VM deletion (cost continues)
│   └── Data disks: + Create and attach / Attach existing
│       ├── Name, Size, Disk type, LUN
│       └── Delete with VM: Yes/No
│
├── Networking
│   ├── Virtual Network: Select/Create
│   ├── Subnet: Select
│   ├── Public IP: Create new / None
│   │   ├── SKU: Standard (⚠️ secure by default — needs NSG)
│   │   └── Assignment: Static / Dynamic
│   ├── NIC NSG: None / Basic / Advanced
│   ├── Accelerated networking: Enabled (if supported)
│   ├── Load balancing: None / LB / App Gateway
│   └── Delete NIC on VM delete: Yes/No
│
├── Management
│   ├── Azure AD login: Enable (requires extension)
│   ├── Auto-shutdown: Enable → Time → Notification
│   ├── Backup: Enable → Recovery Services Vault → Policy
│   ├── Guest OS updates: Patch orchestration mode
│   └── Boot diagnostics: Enable (default) / Disable
│
├── Advanced
│   ├── Extensions: Add (Custom Script, DSC, etc.)
│   ├── Custom data / Cloud-init (Linux)
│   ├── Proximity placement group
│   ├── Dedicated host
│   └── VM generation: Gen 1 / Gen 2
│
├── Tags
│
└── Review + Create
    └── RBAC: Virtual Machine Contributor or Contributor
```

---

### 20.2 Resize a Virtual Machine

> **Portal:** `VM → Size`

```
Resize VM
│
├── Prerequisites
│   ├── VM can be running or stopped
│   │   ⚠️ Some sizes require VM to be STOPPED (deallocated)
│   └── Target size must be available in the VM's region/hardware cluster
│
├── Step 1: Check Available Sizes
│   │   Portal: VM → Availability + scaling → Size
│   └── Shows available sizes (grayed out = not available on current hardware)
│       ⚠️ If desired size not available:
│       ├── Stop (deallocate) VM → more sizes become available
│       └── If still not available → redeploy to different hardware
│
├── Step 2: Select New Size
│   ├── Select target size from list
│   └── Resize
│       ⚠️ VM will RESTART during resize (brief downtime)
│       ⚠️ Dynamic public IP may change
│
├── CLI: az vm resize -g <rg> -n <vm> --size Standard_D4s_v5
│
└── ⚠️ Availability Set constraint:
    └── All VMs in an Availability Set must support the same size family
        Cannot mix sizes across different hardware generations in a set
```

---

### 20.3 Attach Data Disk

> **Portal:** `VM → Disks → + Create and attach a new disk`

```
Attach Data Disk
│
├── Method 1: Create New Disk
│   │   Portal: VM → Settings → Disks → Data disks → + Create and attach new disk
│   ├── Name
│   ├── Storage type: Premium SSD / Standard SSD / Standard HDD / Ultra
│   ├── Size (GiB): e.g., 128, 256, 512, 1024
│   ├── Encryption: PMK / CMK
│   ├── LUN: Auto-assigned or manual (0, 1, 2, ...)
│   └── Save
│       ⚠️ Max data disks depends on VM size
│
├── Method 2: Attach Existing Disk
│   │   Portal: VM → Disks → + Attach existing disk
│   ├── Select existing managed disk
│   │   ⚠️ Disk must be in same region and NOT attached to another VM
│   └── Save
│
├── Step 2: Initialize in OS
│   ├── Windows: Disk Management → Initialize → Create Volume
│   └── Linux: lsblk → fdisk/parted → mkfs → mount → fstab
│       ⚠️ Disk shows as raw — must format and mount in OS
│
└── CLI: az vm disk attach -g <rg> --vm-name <vm> --name <disk> --new --size-gb 128 --sku Premium_LRS
```

---

### 20.4 Configure Auto-Shutdown

> **Portal:** `VM → Operations → Auto-shutdown`

```
Configure Auto-Shutdown
│
├── Portal: VM → Operations → Auto-shutdown
│
├── Enable: Yes
├── Scheduled shutdown time: e.g., 19:00 (7 PM)
├── Time zone: Select
├── Notification:
│   ├── Email: Enter email address
│   ├── Webhook URL (optional)
│   └── Minutes before shutdown to send notification: 15/30/60
│
└── Save
    ⚠️ Auto-shutdown DEALLOCATES — stops compute billing
    ⚠️ No auto-START — must manually start or use Automation Runbook
    ⚠️ Notification sent before shutdown so user can cancel if needed
```

---

### 20.5 Configure VMSS Auto-Scale

> **Portal:** `VMSS → Scaling`

```
Configure VMSS Auto-Scale
│
├── Portal: VMSS → Availability + scaling → Scaling
│
├── Manual Scale
│   └── Instance count: Set fixed number
│
├── Custom Autoscale
│   ├── Default condition:
│   │   ├── Scale mode: Scale based on metric / Scale to specific instance count
│   │   ├── Instance limits:
│   │   │   ├── Minimum: e.g., 2
│   │   │   ├── Maximum: e.g., 10
│   │   │   └── Default: e.g., 2
│   │   │       ⚠️ Default used when metrics unavailable
│   │   │
│   │   ├── Scale-out rule: + Add
│   │   │   ├── Metric: Percentage CPU
│   │   │   ├── Operator: Greater than
│   │   │   ├── Threshold: 75
│   │   │   ├── Duration: 10 minutes
│   │   │   ├── Action: Increase count by 1
│   │   │   └── Cool down: 5 minutes (⚠️ default)
│   │   │
│   │   └── Scale-in rule: + Add
│   │       ├── Metric: Percentage CPU
│   │       ├── Operator: Less than
│   │       ├── Threshold: 25
│   │       ├── Duration: 10 minutes
│   │       ├── Action: Decrease count by 1
│   │       └── Cool down: 5 minutes
│   │
│   └── Schedule-based condition (optional):
│       ├── Start/end date+time
│       ├── Repeat: specific days
│       └── Instance limits for scheduled period
│
└── Save
    ⚠️ Always create BOTH scale-out and scale-in rules
    ⚠️ Without scale-in, instances grow but never shrink
```

---

### 20.6 Enable Azure Disk Encryption (ADE)

> **Portal:** `VM → Disks → Additional settings`

```
Enable Azure Disk Encryption
│
├── Prerequisites
│   ├── Key Vault exists in same region as VM
│   ├── Key Vault Access Policy: Azure Disk Encryption for volume encryption ✅
│   │   ⚠️ OR Key Vault RBAC with appropriate role
│   ├── VM is running
│   └── VM size supports ADE (most do, except Basic A-series)
│
├── Step 1: Configure Key Vault
│   │   Portal: Key Vault → Settings → Access configuration
│   ├── Azure Disk Encryption for volume encryption: ✅ Enabled
│   └── Save
│       ⚠️ MUST enable this checkbox or ADE will fail
│
├── Step 2: Enable ADE on VM
│   │   Portal: VM → Disks → Additional settings → Encryption
│   ├── Disks to encrypt: OS only / Data only / OS and data
│   ├── Key Vault: Select
│   ├── Key (optional): Select or auto-generate
│   └── Save
│
├── CLI:
│   az vm encryption enable -g <rg> -n <vm> \
│     --disk-encryption-keyvault <keyvault-name> \
│     --volume-type All
│
└── ⚠️ Notes
    ├── Windows: Uses BitLocker
    ├── Linux: Uses DM-Crypt
    ├── Encrypts OS + Data disks (or selected)
    └── Cannot be disabled on Linux OS disk once enabled
```

---

### 20.7 Configure JIT VM Access

> **Portal:** `VM → Connect` or `Defender for Cloud → Just in time VM access`

```
Configure JIT VM Access
│
├── Prerequisites
│   ├── Microsoft Defender for Servers enabled (Plan 1 or 2)
│   │   ⚠️ JIT requires Defender — will not appear without it
│   └── NSG or Azure Firewall on the VM
│
├── Step 1: Enable JIT on VM
│   │   Portal: Microsoft Defender for Cloud →
│   │   Workload protections → Just in time VM access
│   ├── Not configured tab → Select VM → Enable JIT
│   └── Or: VM → Connect → JIT → Configure JIT
│
├── Step 2: Configure Ports
│   ├── Port: 3389 (RDP) / 22 (SSH) / Custom
│   ├── Protocol: TCP / UDP / Any
│   ├── Allowed source IPs: Any / Specific IPs / My IP
│   ├── Max request time: 1–24 hours
│   │   ⚠️ Default max = 3 hours for RDP
│   └── Save
│
├── Step 3: Request Access
│   │   Portal: VM → Connect → Request access
│   ├── Select port(s)
│   ├── Source IP: My IP / Custom
│   ├── Time range: 1–24 hours
│   └── Open ports
│       ⚠️ Creates temporary NSG Allow rule
│       ⚠️ Rule auto-expires after time limit
│
└── RBAC: Reader + Microsoft.Security/locations/jitNetworkAccessPolicies/*/
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
