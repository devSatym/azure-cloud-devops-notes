<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Bastion — AZ-104 Revision Notes

---

## 1. What is Azure Bastion?

- **Managed PaaS service** for secure **RDP/SSH** access to VMs **via the Azure Portal browser**
- Connects over **TLS (port 443)** — no public IP needed on VMs
- Eliminates exposure of RDP (3389) / SSH (22) ports to the public internet
- Deployed into a **dedicated subnet** named **AzureBastionSubnet**
- Provides secure connectivity **without VPN or jump box**

---

## 2. Key Components

| Component | Purpose |
|---|---|
| **Azure Bastion** | Managed service deployed in VNet — brokers RDP/SSH sessions |
| **AzureBastionSubnet** | Dedicated subnet (MUST be named `AzureBastionSubnet`, minimum /26) |
| **Public IP** | Standard SKU, Static — used by Bastion for inbound TLS (443) |
| **VM** | Target VM (Windows = RDP, Linux = SSH) — no public IP required |

---

## 3. SKU Comparison

| Feature | **Developer** | **Basic** | **Standard** | **Premium** |
|---|---|---|---|---|
| **Concurrent sessions** | Personal use | 25 (scale units × 2) | 50 (scale units × 2 per instance) | 50+ |
| **Host scaling (instances)** | ❌ (1 fixed) | ❌ (2 fixed) | ✅ (2–50 instances) | ✅ (2–50 instances) |
| **Connect to VMs in peered VNets** | ❌ | ✅ | ✅ | ✅ |
| **Connect via VM private IP** | ❌ | ❌ | ✅ | ✅ |
| **File upload/download** | ❌ | ❌ | ✅ | ✅ |
| **Native client (SSH/RDP client)** | ❌ | ❌ | ✅ | ✅ |
| **Shareable link** | ❌ | ❌ | ✅ | ✅ |
| **Kerberos authentication** | ❌ | ❌ | ✅ | ✅ |
| **Session recording** | ❌ | ❌ | ❌ | ✅ |
| **Private-only deployment** | ❌ | ❌ | ❌ | ✅ |
| **Dedicated subnet** | ❌ (uses VNet) | ✅ (AzureBastionSubnet) | ✅ | ✅ |
| **Availability Zones** | ❌ | ❌ | ❌ | ✅ |
| **Pricing** | Lowest | Per hour | Per hour + per instance | Per hour + per instance |

> ⚠️ **EXAM TIP:** **Basic** = no file transfer, no native client, no manual scaling, fixed 2 instances. **Standard** = file transfer, native client, shareable links, host scaling (2–50). **Developer** = personal use only, no dedicated subnet.

> ⚠️ **EXAM TIP:** Can upgrade **Developer → Basic → Standard → Premium** but **cannot downgrade**.

---

## 4. AzureBastionSubnet

- **MUST** be named exactly **`AzureBastionSubnet`** — no other name accepted
- Minimum size: **/26** (64 addresses) — for Basic/Standard/Premium
- Developer SKU: does NOT require this subnet
- **No UDRs** allowed on AzureBastionSubnet (breaks Bastion)
- **NSG** can be applied but requires specific rules (see Section 10)
- Must be in the **same VNet** as Bastion resource
- No other resources can be deployed in this subnet

### Portal Path — Create AzureBastionSubnet
```
Virtual Network → Settings → Subnets →
+ Subnet → Name: AzureBastionSubnet →
Address range: /26 or larger → Save
```

> ⚠️ **EXAM TIP:** Three mandatory subnet names to remember: `AzureBastionSubnet` (/26), `GatewaySubnet` (recommend /27), `AzureFirewallSubnet` (/26). Each serves a different purpose.

> ⚠️ **EXAM TIP:** **No UDRs** on AzureBastionSubnet. UDRs break Bastion. NSGs are allowed but need specific rules.

---

## 5. How Bastion Works

```
User (Browser/Native Client)
    │
    ├── HTTPS (TLS on port 443)
    │
    ▼
Azure Bastion (in AzureBastionSubnet)
    │
    ├── RDP (3389) or SSH (22) — private network only
    │
    ▼
Target VM (private IP only — no public IP needed)
```

- User connects via **Azure Portal → VM → Connect → Bastion**
- Connection is **HTML5-based** in the browser (no client software needed for Basic)
- Standard/Premium: also supports **native RDP/SSH client** via `az network bastion` CLI
- All traffic encrypted via TLS — never exposes RDP/SSH to internet

---

## 6. Connecting to VMs

### Via Azure Portal (All SKUs)
```
Virtual Machine → Connect → Bastion →
Enter Username + Password (or SSH Key) → Connect
```

- Opens **in-browser** RDP (Windows) or SSH (Linux) session
- Supports: **Password**, **SSH private key** (paste or file), **SSH key from Key Vault**

### Via Native Client (Standard/Premium Only)
```
az network bastion rdp --name <bastion> --resource-group <rg> --target-resource-id <vm-id>
az network bastion ssh --name <bastion> --resource-group <rg> --target-resource-id <vm-id> --auth-type ssh-key --ssh-key <path>
```

- Uses native **mstsc.exe** (RDP) or **ssh** client
- Supports **Azure AD/Entra ID login** for Linux VMs

### Via Private IP (Standard/Premium Only)
- Connect to any VM reachable from Bastion VNet using its **private IP** (not just Azure VMs)
- Includes on-prem VMs accessible via VPN/ExpressRoute
```
az network bastion rdp --name <bastion> --resource-group <rg> --target-ip-address <private-ip>
```

> ⚠️ **EXAM TIP:** Portal-based connection = all SKUs. **Native client** = Standard/Premium only. **Connect by IP** = Standard/Premium only.

---

## 7. VNet Peering Support

- Bastion in hub VNet can connect to VMs in **peered spoke VNets**
- Requirements: VNet peering established, Bastion in one VNet, VM in peered VNet
- **Basic, Standard, Premium** support peered VNets
- **Developer** does NOT support peered VNets

> ⚠️ **EXAM TIP:** Bastion supports **peered VNets** (Basic and above). You do NOT need a separate Bastion per VNet. Deploy once in hub → access VMs in all peered spokes.

---

## 8. File Transfer (Standard/Premium Only)

- Upload/download files between local machine and target VM
- **Upload**: Max **500 MB** per file
- **Download**: Max **2 GB** per file (Premium), max **500 MB** (Standard)
- Works via the **browser-based** session
- NOT available on Basic or Developer SKU

### How to Use
```
During active Bastion session in browser →
Upload: Use upload icon in toolbar → select local file
Download: Browse to file on VM → use download icon
```

> ⚠️ **EXAM TIP:** File transfer = **Standard/Premium only**. Upload max = 500 MB. Download max = 500 MB (Standard) / 2 GB (Premium).

---

## 9. Shareable Link (Standard/Premium Only)

- Generate a **link** that allows users to connect to a VM via Bastion **without Azure Portal access**
- User only needs the link + VM credentials (no Azure RBAC needed for VM)
- Bastion admin creates the link and shares it
- Link can be **revoked** at any time

### Portal Path — Create Shareable Link
```
Bastion → Settings → Shareable links → + Add →
Select VM(s) → Generate link → Copy & share
```

> ⚠️ **EXAM TIP:** Shareable links allow access **without Azure Portal login**. Users don't need Azure RBAC on the VM — just the link and VM credentials.

---

## 10. NSG on AzureBastionSubnet

### Required Inbound Rules

| Rule | Source | Destination | Port | Protocol |
|---|---|---|---|---|
| Allow internet to Bastion | Internet | AzureBastionSubnet | **443** | TCP |
| Allow Bastion control plane | GatewayManager | AzureBastionSubnet | **443** | TCP |
| Allow Azure LB health | AzureLoadBalancer | AzureBastionSubnet | **443** | TCP |
| Allow Bastion data plane | VirtualNetwork | AzureBastionSubnet | **8080, 5701** | Any |

### Required Outbound Rules

| Rule | Source | Destination | Port | Protocol |
|---|---|---|---|---|
| Allow to VMs (SSH) | AzureBastionSubnet | VirtualNetwork | **22** | TCP |
| Allow to VMs (RDP) | AzureBastionSubnet | VirtualNetwork | **3389** | TCP |
| Allow to Azure (diagnostics) | AzureBastionSubnet | AzureCloud | **443** | TCP |
| Allow Bastion data plane | AzureBastionSubnet | VirtualNetwork | **8080, 5701** | Any |

> ⚠️ **EXAM TIP:** Inbound **443 from Internet** + **443 from GatewayManager** = MANDATORY. Outbound **22 + 3389 to VirtualNetwork** = MANDATORY. If these are blocked, Bastion will NOT work.

> ⚠️ **EXAM TIP:** GatewayManager service tag is required for Bastion **AND** VPN/ER gateway management (specifically for v2 App Gateway it's ports 65200-65535, for Bastion it's 443).

---

## 11. Host Scaling (Standard/Premium Only)

- Scale Bastion capacity by adding **instances (scale units)**
- Range: **2–50 instances**
- Each instance supports approximately **20–25 concurrent RDP** or **40–50 concurrent SSH** sessions
- Basic SKU: fixed at **2 instances** (cannot scale)

### Portal Path — Configure Scaling
```
Bastion → Settings → Configuration →
Instance count: 2–50 (slider) → Apply
```

> ⚠️ **EXAM TIP:** Default = 2 instances. Max = 50 instances. Can only **scale up** (increasing takes ~10 min). Scaling down also supported.

---

## 12. Azure Bastion vs Traditional Jump Box vs P2S VPN

| Feature | **Azure Bastion** | **Jump Box VM** | **P2S VPN** |
|---|---|---|---|
| **Type** | Managed PaaS | IaaS VM you manage | VPN Gateway |
| **Public IP on target VM** | ❌ Not needed | ❌ Not needed | ❌ Not needed |
| **Port exposure** | Only 443 (TLS) | 3389/22 exposed on jump box | UDP 500/4500 or TCP 443 |
| **Maintenance** | ❌ None (Microsoft managed) | ✅ OS patching, hardening required | Gateway management |
| **Client software** | Browser (or native client Standard+) | RDP/SSH client | VPN client |
| **Cost** | Per hour + per instance | VM compute cost | Gateway hourly + egress |
| **Session recording** | ✅ (Premium) | ❌ | ❌ |
| **Zero Trust** | ✅ (no open ports) | ❌ (exposed RDP/SSH) | ✅ |

> ⚠️ **EXAM TIP:** If question says "secure RDP/SSH without exposing public IP or VPN" → **Azure Bastion**. If "connect from browser without installing anything" → **Azure Bastion**.

---

## 13. Security & RBAC

### RBAC Requirements for Connecting via Bastion

| Permission | Required On |
|---|---|
| **Reader** role | Bastion resource |
| **Reader** role | Target VM |
| **Reader** role | Target VM's NIC (with private IP) |
| **VM credentials** | Username + Password or SSH key |

- Standard Azure Reader on Bastion + VM + NIC = minimum to connect
- No special Bastion-specific RBAC role exists
- **Shareable links** bypass RBAC requirement (only need VM credentials)

### RBAC for Management

| Action | Minimum Role |
|---|---|
| Create / delete Bastion | Network Contributor (+ Contributor on subnet) |
| Modify Bastion config | Network Contributor |
| Create shareable links | Contributor on Bastion |
| View Bastion | Reader |

> ⚠️ **EXAM TIP:** To connect via Bastion, user needs **Reader** on Bastion + VM + NIC. No special "Bastion User" role exists — it's just Reader.

---

## 14. Monitoring & Diagnostics

### Diagnostic Logs

| Log | Content |
|---|---|
| **BastionAuditLogs** | Who connected, to which VM, when, session duration, protocol |

### Portal Path — Enable Diagnostics
```
Bastion → Monitoring → Diagnostic settings →
+ Add → Select: BastionAuditLogs →
Destination: Log Analytics / Storage / Event Hub → Save
```

### Key Metrics

| Metric | Description |
|---|---|
| **Session count** | Active Bastion sessions |
| **Memory utilization** | Bastion instance memory usage |
| **CPU utilization** | Bastion instance CPU usage |

### Portal Path — View Metrics
```
Bastion → Monitoring → Metrics →
Select: Session Count / CPU / Memory → Apply
```

### Portal Path — Alerts
```
Bastion → Monitoring → Alerts → + New alert rule →
Signal: Session count / CPU → Action Group → Create
```

> ⚠️ **EXAM TIP:** **BastionAuditLogs** = track who accessed which VM and when. Critical for compliance and security audits.

---

## 15. Pricing Key Points

| Component | Cost |
|---|---|
| **Developer SKU** | Free during preview / lowest cost |
| **Basic SKU** | Per-hour charge (2 fixed instances) |
| **Standard SKU** | Per-hour base + per additional instance-hour (2–50) |
| **Premium SKU** | Per-hour base + per additional instance-hour + premium features |
| **Outbound data** | Standard data transfer charges |
| **Public IP** | Standard Static Public IP charges |

### Cost Optimization
- **Developer SKU** for personal/dev use (cheapest)
- **Basic SKU** for small teams (fixed 2 instances)
- Cannot stop/deallocate Bastion — runs continuously once deployed
- **Delete Bastion** when not needed (recreate takes ~10 min)

> ⚠️ **EXAM TIP:** Bastion **cannot be stopped/deallocated** (unlike Azure Firewall). It runs 24/7. To save cost → **delete** and recreate when needed.

---

## 16. Limitations & Constraints

| Constraint | Limit |
|---|---|
| Subnet name | Must be **`AzureBastionSubnet`** |
| Subnet minimum size | **/26** (Basic/Standard/Premium) |
| Max instances (Standard/Premium) | **50** |
| Fixed instances (Basic) | **2** |
| Concurrent RDP per instance | ~**20–25** |
| Concurrent SSH per instance | ~**40–50** |
| File upload max | **500 MB** |
| File download max (Standard) | **500 MB** |
| File download max (Premium) | **2 GB** |
| UDR on AzureBastionSubnet | **Not allowed** |
| Public IP SKU | **Standard, Static** only |
| Peered VNet support | Basic+ (not Developer) |
| Native client support | Standard+ |
| Session recording | Premium only |

---

## 17. CLI / PowerShell Commands

| Action | Command |
|---|---|
| Create Bastion | `az network bastion create -g <rg> -n <name> --vnet-name <vnet> --public-ip-address <pip> --sku Standard` |
| Delete Bastion | `az network bastion delete -g <rg> -n <name>` |
| List Bastion | `az network bastion list -g <rg>` |
| RDP via native client | `az network bastion rdp -n <bastion> -g <rg> --target-resource-id <vm-id>` |
| SSH via native client | `az network bastion ssh -n <bastion> -g <rg> --target-resource-id <vm-id> --auth-type ssh-key --ssh-key <path>` |
| SSH by IP | `az network bastion ssh -n <bastion> -g <rg> --target-ip-address <ip> --auth-type password --username <user>` |
| Create tunnel | `az network bastion tunnel -n <bastion> -g <rg> --target-resource-id <vm-id> --resource-port 3389 --port 50001` |

> ⚠️ **EXAM TIP:** `az network bastion rdp/ssh` = native client connection (Standard+ only). `az network bastion tunnel` = creates a tunnel for custom ports.

---

## 18. Quick-Fire Exam Points ⚡

1. Azure Bastion = **managed PaaS** for secure **RDP/SSH via browser** (HTML5 over TLS 443)
2. **No public IP needed on VMs** — eliminates RDP/SSH port exposure to internet
3. Subnet MUST be named **`AzureBastionSubnet`**, minimum **/26**
4. **No UDRs** on AzureBastionSubnet — UDRs break Bastion
5. NSG on AzureBastionSubnet must allow: inbound **443 from Internet + GatewayManager**, outbound **22+3389 to VirtualNetwork**
6. **Developer** = personal use, no dedicated subnet, no peering, cheapest
7. **Basic** = 2 fixed instances, peered VNets, no file transfer, no native client
8. **Standard** = scaling (2–50), file transfer, native client, shareable links, connect by IP
9. **Premium** = session recording, private-only deployment, availability zones
10. Can upgrade **Developer → Basic → Standard → Premium** but **cannot downgrade**
11. **File transfer**: upload max 500 MB; download max 500 MB (Standard) / 2 GB (Premium)
12. **Shareable link** = access VM without Azure Portal login (Standard/Premium)
13. Bastion in **hub VNet** → connect to VMs in **peered spoke VNets** (Basic+)
14. **Native client** (`az network bastion rdp/ssh`) = Standard/Premium only
15. **Connect by private IP** = Standard/Premium — includes on-prem VMs via VPN/ER
16. RBAC to connect: **Reader** on Bastion + VM + NIC + VM credentials
17. No special "Bastion User" RBAC role — just **Reader**
18. Bastion **cannot be stopped/deallocated** — runs 24/7. Delete to save cost.
19. Public IP must be **Standard SKU, Static** assignment
20. Each instance: ~20–25 concurrent RDP or ~40–50 concurrent SSH sessions
21. **BastionAuditLogs** = who connected to which VM, when, duration (compliance)
22. Bastion vs Jump Box: Bastion = managed/no patching, Jump Box = IaaS/you manage
23. Bastion uses **port 443 (TLS)** from user to Bastion, then **3389/22** internally to VM
24. **Network Contributor** = minimum role to create/manage Bastion
25. Bastion deployed per-VNet (but serves peered VNets too)

---

## 19. Step-by-Step Configuration Mind Maps 🗺️

---

### 19.1 Create Azure Bastion

> **Portal:** `Home → + Create a resource → Bastion → Create`

```
Create Azure Bastion
│
├── Prerequisites
│   ├── VNet exists
│   ├── AzureBastionSubnet created (minimum /26)
│   │   ⚠️ MUST be named "AzureBastionSubnet"
│   │   ⚠️ No UDRs on this subnet
│   └── Standard Static Public IP (create during wizard or pre-create)
│
├── Step 1: Basics
│   ├── Subscription, Resource Group
│   ├── Name
│   ├── Region (must match VNet)
│   ├── Tier: Developer / Basic / Standard / Premium
│   │   ⚠️ Upgrade possible, downgrade NOT possible
│   ├── Virtual network: Select
│   ├── Subnet: AzureBastionSubnet (auto-selected)
│   │   └── If not exists → "Manage subnet configuration" → create it
│   └── Public IP address
│       ├── Create new or select existing
│       ├── SKU: Standard (required)
│       └── Assignment: Static (required)
│       ⚠️ Basic SKU Public IP NOT supported
│
├── Step 2: Advanced (Standard/Premium only)
│   ├── Instance count: 2–50 (Standard/Premium)
│   │   ⚠️ Default = 2. Min 2 for production.
│   ├── Copy/Paste: Enabled (default)
│   ├── Native client support: Enable (Standard+)
│   ├── Shareable link: Enable (Standard+)
│   ├── IP-based connection: Enable (Standard+)
│   ├── Kerberos: Enable (Standard+)
│   └── Session recording: Enable (Premium)
│
├── Step 3: Tags (Optional)
│
└── Step 4: Review + Create → Create
    ├── RBAC: Network Contributor
    └── ⚠️ Deployment takes ~5–10 minutes
```

---

### 19.2 Connect to VM via Bastion (Portal)

> **Portal:** `Virtual Machine → Connect → Bastion`

```
Connect to VM via Bastion (Browser)
│
├── Prerequisites
│   ├── Azure Bastion deployed in VM's VNet (or peered VNet)
│   ├── User has Reader role on: Bastion + VM + VM's NIC
│   └── VM is running
│
├── Step 1: Navigate to VM
│   └── Virtual Machine → Overview → Connect → Bastion
│       ⚠️ If Bastion not deployed → option to create one
│
├── Step 2: Enter Credentials
│   ├── Authentication Type:
│   │   ├── Password → Enter username + password
│   │   ├── SSH Private Key from Local File → Upload .pem file
│   │   ├── SSH Private Key → Paste key content
│   │   └── SSH Key from Azure Key Vault → Select vault + key
│   └── ⚠️ Linux VM = SSH (port 22), Windows VM = RDP (port 3389)
│
├── Step 3: Connection Settings (optional)
│   ├── Protocol: RDP / SSH
│   └── Port: Default 3389 (RDP) or 22 (SSH), can customize
│
├── Step 4: Connect
│   ├── Opens in new browser tab (HTML5)
│   ├── Full desktop (RDP) or terminal (SSH)
│   └── ⚠️ Pop-up blocker may prevent new tab — allow pop-ups
│
└── During Session
    ├── Clipboard (copy/paste): Enabled by default
    ├── File upload: Toolbar icon (Standard+ only, max 500 MB)
    ├── File download: Navigate to file → download (Standard+)
    └── Disconnect: Close browser tab or timeout
```

---

### 19.3 Connect via Native Client (Standard/Premium)

> **Tools:** Azure CLI

```
Connect via Native RDP/SSH Client
│
├── Prerequisites
│   ├── Azure Bastion (Standard or Premium SKU)
│   ├── Native client support: Enabled on Bastion
│   ├── Azure CLI installed
│   └── az login completed
│
├── RDP Connection (Windows VM)
│   │
│   ├── By Resource ID:
│   │   └── az network bastion rdp
│   │       --name <bastion-name>
│   │       --resource-group <rg>
│   │       --target-resource-id <vm-resource-id>
│   │   ⚠️ Opens native mstsc.exe (Remote Desktop)
│   │
│   └── By Private IP (Standard+):
│       └── az network bastion rdp
│           --name <bastion-name>
│           --resource-group <rg>
│           --target-ip-address <private-ip>
│
├── SSH Connection (Linux VM)
│   │
│   ├── With SSH Key:
│   │   └── az network bastion ssh
│   │       --name <bastion-name>
│   │       --resource-group <rg>
│   │       --target-resource-id <vm-resource-id>
│   │       --auth-type ssh-key
│   │       --ssh-key ~/.ssh/id_rsa
│   │
│   └── With Password:
│       └── az network bastion ssh
│           --name <bastion-name>
│           --resource-group <rg>
│           --target-resource-id <vm-resource-id>
│           --auth-type password
│           --username <user>
│
├── Tunnel (Custom Port Forwarding)
│   └── az network bastion tunnel
│       --name <bastion-name>
│       --resource-group <rg>
│       --target-resource-id <vm-resource-id>
│       --resource-port 3389
│       --port 50001
│   ⚠️ Forwards local port 50001 → VM port 3389 via Bastion
│   ⚠️ Connect local RDP client to localhost:50001
│
└── ⚠️ Native client = Standard/Premium SKU only
    ⚠️ Must enable "Native client support" in Bastion config
```

---

### 19.4 Configure NSG on AzureBastionSubnet

> **Portal:** `NSG → Settings → Inbound/Outbound security rules`

```
Configure NSG for AzureBastionSubnet
│
├── ⚠️ NSG is optional but if applied, these rules are MANDATORY
│
├── Inbound Rules (MUST ALLOW)
│   │
│   ├── Rule 1: Allow HTTPS from Internet
│   │   ├── Source: Internet
│   │   ├── Destination: AzureBastionSubnet
│   │   ├── Port: 443
│   │   ├── Protocol: TCP
│   │   ├── Action: Allow
│   │   └── Priority: e.g., 100
│   │
│   ├── Rule 2: Allow GatewayManager
│   │   ├── Source: GatewayManager
│   │   ├── Port: 443
│   │   ├── Protocol: TCP
│   │   ├── Action: Allow
│   │   └── Priority: 110
│   │   ⚠️ Required for Bastion control plane
│   │
│   ├── Rule 3: Allow Azure Load Balancer
│   │   ├── Source: AzureLoadBalancer
│   │   ├── Port: 443
│   │   └── Action: Allow
│   │
│   └── Rule 4: Allow Bastion Data Plane
│       ├── Source: VirtualNetwork
│       ├── Port: 8080, 5701
│       └── Action: Allow
│       ⚠️ Internal communication between Bastion instances
│
├── Outbound Rules (MUST ALLOW)
│   │
│   ├── Rule 1: Allow RDP to VMs
│   │   ├── Destination: VirtualNetwork
│   │   ├── Port: 3389
│   │   └── Action: Allow
│   │
│   ├── Rule 2: Allow SSH to VMs
│   │   ├── Destination: VirtualNetwork
│   │   ├── Port: 22
│   │   └── Action: Allow
│   │
│   ├── Rule 3: Allow to Azure Cloud
│   │   ├── Destination: AzureCloud
│   │   ├── Port: 443
│   │   └── Action: Allow
│   │   ⚠️ For diagnostics and metrics
│   │
│   └── Rule 4: Allow Bastion Data Plane
│       ├── Destination: VirtualNetwork
│       ├── Port: 8080, 5701
│       └── Action: Allow
│
└── ⚠️ If ANY required rule is missing/denied → Bastion FAILS
    ⚠️ No UDRs allowed on AzureBastionSubnet at all
```

---

### 19.5 Monitor Azure Bastion

> **Portal:** `Bastion → Monitoring`

```
Monitor Azure Bastion
│
├── Enable Diagnostic Logs
│   │   Portal: Bastion → Monitoring → Diagnostic settings
│   ├── + Add diagnostic setting
│   ├── Logs: BastionAuditLogs
│   │   ├── Records: user, target VM, timestamp, duration, protocol
│   │   └── ⚠️ Critical for compliance audits
│   ├── Destination: Log Analytics / Storage / Event Hub
│   └── Save
│
├── View Metrics
│   │   Portal: Bastion → Monitoring → Metrics
│   ├── Session Count → active connections
│   ├── CPU Utilization → per instance
│   ├── Memory Utilization → per instance
│   └── ⚠️ High CPU/Memory → consider scaling up instances
│
├── Configure Alerts
│   │   Portal: Bastion → Monitoring → Alerts
│   ├── + New alert rule
│   ├── Signal: Session Count > threshold
│   ├── Signal: CPU > 80%
│   ├── Action Group: Email / SMS
│   └── Create
│
└── Log Analytics Query (Audit)
    └── AzureDiagnostics
        | where Category == "BastionAuditLogs"
        | project TimeGenerated, UserName, TargetVMIPAddress,
                  Protocol, SessionDuration
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
