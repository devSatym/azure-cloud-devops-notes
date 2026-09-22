<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure App Service — AZ-104 Revision Notes

---

## 1. What is Azure App Service?

- **Fully managed PaaS** for hosting web applications, REST APIs, and mobile backends
- Supports: **.NET, .NET Core, Java, Node.js, PHP, Python, Ruby, custom containers**
- Built-in **auto-scale, load balancing, CI/CD, deployment slots, custom domains, SSL**
- Runs on **App Service Plans** (defines compute resources)
- Supports **Windows** and **Linux** hosting
- Global scale with **high availability** (SLA up to 99.95%)

> ⚠️ **EXAM TIP:** App Service is **PaaS** — you manage the app and configuration. Azure manages the OS, patching, infrastructure. If question says "no infrastructure management" → App Service.

---

## 2. Key Components

| Component | Description |
|---|---|
| **App Service Plan** | Defines compute resources (region, VM size, instance count) |
| **Web App** | The application hosted in the plan |
| **Deployment Slots** | Separate environments (staging, production) for swapping |
| **Custom Domain** | Map your own domain name to the web app |
| **TLS/SSL Certificates** | HTTPS encryption for custom domains |
| **Hybrid Connections** | Connect to on-prem resources without VPN |
| **VNet Integration** | Connect outbound from app to resources in a VNet |
| **Private Endpoints** | Inbound private access from a VNet |
| **WebJobs** | Background tasks running in the same App Service Plan |
| **Application Settings** | Environment variables and connection strings |
| **Managed Identity** | Azure AD identity for the app (no credentials) |

---

## 3. App Service Plan Tiers (SKUs)

| Tier | Plans | Auto-scale | Slots | VNet Integration | Custom Domain/SSL | Backups | Use Case |
|---|---|---|---|---|---|---|---|
| **Free (F1)** | Shared | ❌ | ❌ | ❌ | ❌ | ❌ | Testing |
| **Shared (D1)** | Shared | ❌ | ❌ | ❌ | Custom domain ✅, no SSL | ❌ | Dev |
| **Basic (B1-B3)** | Dedicated | ❌ Manual only | ❌ | ❌ | ✅ | ❌ | Dev/test |
| **Standard (S1-S3)** | Dedicated | ✅ Up to 10 | ✅ 5 slots | ✅ | ✅ | ✅ | Production |
| **Premium v2/v3 (P1v2-P3v3)** | Dedicated | ✅ Up to 30 | ✅ 20 slots | ✅ | ✅ | ✅ | High-perf production |
| **Isolated v2 (I1v2-I6v2)** | **ASE (dedicated VNet)** | ✅ Up to 100 | ✅ 20 slots | ✅ Built-in | ✅ | ✅ | Enterprise, compliance |

### Key Plan Limits

| Feature | Free | Shared | Basic | Standard | Premium v3 | Isolated v2 |
|---|---|---|---|---|---|---|
| **Max instances** | — | — | 3 | 10 | 30 | 100 |
| **Disk space** | 1 GB | 1 GB | 10 GB | 50 GB | 250 GB | 1 TB |
| **Custom domains** | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **SSL** | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| **Deployment slots** | ❌ | ❌ | ❌ | 5 | 20 | 20 |
| **Daily backups** | ❌ | ❌ | ❌ | ✅ (10/day) | ✅ (50/day) | ✅ (50/day) |
| **Auto-scale** | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| **99.95% SLA** | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| **VNet Integration** | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ (native) |
| **Always On** | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |

> ⚠️ **EXAM TIP:** **Deployment slots** start at **Standard** tier. Free/Shared/Basic have **NO slots**. Standard = 5 slots, Premium/Isolated = 20 slots.

> ⚠️ **EXAM TIP:** **Auto-scale** starts at **Standard** tier. Basic supports only **manual** scaling (fixed instance count).

> ⚠️ **EXAM TIP:** **Free and Shared** tiers run on **shared** VMs (multi-tenant). Basic and above run on **dedicated** VMs.

> ⚠️ **EXAM TIP:** **Always On** keeps the app loaded even with no traffic. Available from **Basic** tier and above. Default is **OFF** — must enable for production apps and WebJobs.

---

## 4. App Service Plans — Key Concepts

- One plan → **multiple web apps** (apps share the plan's resources)
- Plan defines: **Region, VM Size, OS (Windows/Linux), Instance count**
- **Cannot mix Windows and Linux** apps in the same plan (same RG in some regions)
- Scaling the plan = scaling ALL apps in that plan
- Apps can be moved between plans (same region + OS required)

#### Portal Path — Create App Service Plan
```
Home → App Service plans → + Create →
Subscription, RG, Name, OS (Windows/Linux),
Region, Pricing tier → Create
```

> ⚠️ **EXAM TIP:** You **cannot** mix Windows and Linux web apps in the **same App Service Plan**. They require separate plans.

> ⚠️ **EXAM TIP:** When you **scale up** (change tier), ALL apps in the plan are affected. When you **scale out** (add instances), ALL apps run on all instances.

---

## 5. Deployment Slots

- Separate **live environments** with their own URLs, settings, and config
- **Production slot** = default (always exists)
- Additional slots: staging, testing, dev, etc.
- **Swap**: exchange content and config between slots (zero-downtime deployment)
- Each slot has its own URL: `<appname>-<slotname>.azurewebsites.net`

### Slot Settings — What Swaps vs What Stays

| Swaps WITH the slot | Stays WITH the slot (sticky) |
|---|---|
| App code / content | Slot-specific app settings (marked as "slot setting") |
| App settings (NOT marked sticky) | Slot-specific connection strings (marked as "slot setting") |
| Connection strings (NOT marked sticky) | Custom domain bindings |
| Handler mappings | TLS/SSL certificates and bindings |
| Public certificates | Scale settings (instance count) |
| WebJobs content | Diagnostic settings |
| Hybrid connections | CORS |
| Service endpoints | Virtual network integration |
| Azure CDN | Managed identities |
| Always On setting | |
| General settings (framework, bitness) | |

### Swap Types

| Type | Description |
|---|---|
| **Swap** | Direct swap — staging becomes production instantly |
| **Swap with preview** | Multi-phase swap — apply config first, then complete swap |
| **Auto swap** | Automatically swap when code is deployed to a slot |

#### Portal Path — Create Deployment Slot
```
App Service → Deployment → Deployment slots → + Add slot →
Name → Clone settings from: (None / Production / other slot) → Add
```

#### Portal Path — Swap Slots
```
App Service → Deployment → Deployment slots → Swap →
Source: staging → Target: production →
Preview changes → Start Swap
```

#### Portal Path — Mark Setting as Slot-Specific
```
App Service → Settings → Configuration →
Application settings → Edit setting →
Deployment slot setting: ✅ → OK → Save
```

> ⚠️ **EXAM TIP:** Settings marked as **"Deployment slot setting"** are **sticky** — they stay with the slot and do NOT swap. Use this for per-environment connection strings (e.g., staging DB vs production DB).

> ⚠️ **EXAM TIP:** **Custom domains** and **TLS bindings** are sticky — they stay with the slot. After swapping, the production URL still points to the same domain.

> ⚠️ **EXAM TIP:** **Auto swap** is available on **Windows** only. NOT supported on **Linux**.

---

## 6. Custom Domains

- Map your own domain (e.g., `www.contoso.com`) to the app
- Requires **Basic tier or higher** (Custom domain: Shared+, SSL: Basic+)
- Validation: add a **CNAME** or **TXT** record at your DNS provider

### DNS Record Types

| Record | Use Case |
|---|---|
| **CNAME** | Map subdomain (e.g., `www.contoso.com` → `app.azurewebsites.net`) |
| **A record** | Map root domain (e.g., `contoso.com` → app's IP address) |
| **TXT** | Domain verification (`asuid.<domain>` → verification ID) |

#### Portal Path — Add Custom Domain
```
App Service → Settings → Custom domains → + Add custom domain →
Domain: enter domain name → Validate →
Add DNS records (CNAME/A + TXT at registrar) →
Validate → Add
```

> ⚠️ **EXAM TIP:** For **root/apex domain** (e.g., `contoso.com`), use an **A record** (+ TXT for verification). For **subdomain** (e.g., `www.contoso.com`), use a **CNAME**.

---

## 7. TLS/SSL Certificates

### Certificate Types

| Type | Description |
|---|---|
| **App Service Managed Certificate** | Free, auto-renewed, for custom domains. SNI SSL only |
| **App Service Certificate** | Purchased via Azure, stored in Key Vault |
| **Bring your own certificate** | Upload PFX/PEM from external CA |
| **Key Vault certificate** | Import from Azure Key Vault |

### SSL Binding Types

| Binding | Description |
|---|---|
| **SNI SSL** | Server Name Indication — multiple domains on same IP (modern standard) |
| **IP SSL** | Dedicated IP for the certificate (legacy, requires static IP) |

#### Portal Path — Add TLS Certificate
```
App Service → Settings → Certificates →
+ Add certificate → Source: Managed / Upload / Key Vault →
Configure binding → Custom Domains → Add binding → TLS/SSL type
```

> ⚠️ **EXAM TIP:** **App Service Managed Certificate** is **free**, but only supports **SNI SSL**, no wildcard (only standard domains), and no export. For wildcard or IP SSL → purchase or bring your own cert.

> ⚠️ **EXAM TIP:** To enforce **HTTPS only**: `App Service → Settings → Configuration → General settings → HTTPS Only: On`. This redirects HTTP to HTTPS.

---

## 8. Application Settings & Configuration

### Application Settings
- Environment variables for the app (key-value pairs)
- Override values in `appsettings.json` / `web.config` at runtime
- **Not stored in code** — configured in the portal/CLI
- Can be marked as **slot-specific** (sticky to that slot)

### Connection Strings
- Similar to app settings but specifically for database connections
- Types: SQLServer, MySQL, PostgreSQL, Custom
- Also support slot-specific marking

#### Portal Path — Configure Settings
```
App Service → Settings → Configuration →
Application settings tab → + New application setting →
Name, Value, Deployment slot setting ✅/❌ → OK → Save
```

```
App Service → Settings → Configuration →
Connection strings tab → + New connection string →
Name, Value, Type, Slot setting → OK → Save
```

### General Settings

| Setting | Portal Path |
|---|---|
| Stack (runtime) | Configuration → General settings → Stack |
| Platform (32/64-bit) | Configuration → General settings → Platform |
| Always On | Configuration → General settings → Always On |
| HTTPS Only | Configuration → General settings → HTTPS Only |
| Min TLS version | Configuration → General settings → Minimum TLS version |
| ARR Affinity | Configuration → General settings → ARR Affinity |
| HTTP version | Configuration → General settings → HTTP version |

> ⚠️ **EXAM TIP:** **ARR Affinity** = session affinity (sticky sessions). Enable for stateful apps. Disable for stateless apps to improve load distribution.

> ⚠️ **EXAM TIP:** **Always On** must be **enabled** for WebJobs that run continuously or on a schedule. Without it, the app unloads after idle timeout.

---

## 9. Networking Features

### Inbound Features

| Feature | Description | Tier Required |
|---|---|---|
| **App-assigned address** | Dedicated inbound IP (IP SSL) | Basic+ |
| **Access restrictions** | Allow/deny by IP, VNet, service tag | All tiers |
| **Private Endpoints** | Private IP in VNet for inbound access | Standard+ |
| **Service Endpoints** | Restrict access from specific subnets | All tiers |

### Outbound Features

| Feature | Description | Tier Required |
|---|---|---|
| **VNet Integration** | Route outbound traffic through a VNet | Standard v2+ |
| **Hybrid Connections** | Connect to on-prem resources (no VPN) | Standard+ |
| **NAT Gateway** | Static outbound IP via NAT Gateway (via VNet Integration) | Standard+ |

### VNet Integration
- **Regional VNet Integration**: app connects to a VNet in the **same region**
- App can access resources in the VNet (VMs, Private Endpoints, etc.)
- Requires a **dedicated subnet** (delegated to `Microsoft.Web/serverFarms`)
- Minimum subnet size: **/28** (recommended **/26**)
- Does NOT put the app IN the VNet — only outbound traffic routed through VNet

#### Portal Path — VNet Integration
```
App Service → Settings → Networking →
Outbound traffic → VNet integration → + Add VNet →
VNet → Subnet (dedicated, delegated) → Connect
```

### Access Restrictions
- Allow/deny inbound traffic by **IP address, IP range, VNet/subnet, service tag, service endpoint**
- Priority-based rules (lowest number = highest priority)
- **Default**: All traffic allowed (if no rules)
- Also supports rules for the **SCM/Kudu site** separately

#### Portal Path — Access Restrictions
```
App Service → Settings → Networking →
Inbound traffic → Access restriction →
+ Add rule → Source: IP / VNet / Service Tag →
Priority, Action (Allow/Deny) → Add
```

> ⚠️ **EXAM TIP:** VNet Integration = **outbound** only. Private Endpoints = **inbound** only. They serve opposite directions.

> ⚠️ **EXAM TIP:** VNet Integration requires a **dedicated, delegated subnet**. The subnet must be delegated to `Microsoft.Web/serverFarms`. Other resources cannot be in this subnet.

> ⚠️ **EXAM TIP:** Access Restrictions have a **separate rule set** for the **SCM (Kudu) site**. By default, SCM uses the same rules as the main site, but you can configure them independently.

---

## 10. Deployment Methods

| Method | Description |
|---|---|
| **Git (Local)** | Push from local Git repo to App Service |
| **GitHub Actions** | CI/CD from GitHub |
| **Azure DevOps** | CI/CD pipeline |
| **ZIP Deploy** | Upload ZIP file via CLI/API |
| **FTP/FTPS** | Upload files via FTP |
| **Visual Studio** | Publish directly |
| **Azure CLI** | `az webapp deploy` |
| **ARM Template** | Infrastructure as Code |
| **Docker Container** | Deploy container image (Linux) |
| **Kudu (SCM)** | Build server at `<appname>.scm.azurewebsites.net` |

#### Portal Path — Deployment Center
```
App Service → Deployment → Deployment center →
Source: GitHub / Azure Repos / Local Git / FTP →
Configure → Save
```

> ⚠️ **EXAM TIP:** **Kudu** (SCM) is the deployment engine at `https://<appname>.scm.azurewebsites.net`. It provides console, process explorer, log streaming, and deployment management.

---

## 11. Backup & Restore

- Backup the app + configuration + database to a **Storage Account**
- Requires **Standard tier or higher**
- Supports: app settings, file content, connected databases (SQL, MySQL)
- Max backup size: **10 GB** (app + database)
- Database backup max: **4 GB**

### Backup Types

| Type | Description |
|---|---|
| **Scheduled** | Automatic at defined intervals |
| **Manual** | On-demand backup |

#### Portal Path — Configure Backup
```
App Service → Settings → Backups →
Configure → Storage account + Container →
Schedule: On/Off → Frequency → Retention →
Include database: Yes → Connection string → Save
```

> ⚠️ **EXAM TIP:** App Service backup requires **Standard tier+** and a **Storage Account**. Max **10 GB** total (app + DB). Database max = **4 GB**.

> ⚠️ **EXAM TIP:** **Partial backups** are supported — you can exclude specific folders/files from backup.

---

## 12. Scaling

### Scale Up (Vertical)
- Change the **App Service Plan tier** (more CPU, RAM, features)
- Example: B1 → S1 → P1v3
- Affects ALL apps in the plan

### Scale Out (Horizontal)
- Add more **instances** (VMs) running the app
- Manual or auto-scale (Standard+)
- Load balanced automatically

### Auto-Scale Rules
- Based on **metrics** (CPU%, Memory%, HTTP Queue) or **schedule**
- Configure: minimum, maximum, and default instance count
- Cool-down period prevents rapid scaling changes

#### Portal Path — Scale Up
```
App Service Plan → Settings → Scale up (App Service plan) →
Select tier → Apply
```

#### Portal Path — Scale Out
```
App Service Plan → Settings → Scale out (App Service plan) →
Manual scale: Instance count → Save
OR
Custom autoscale → Rules → + Add rule →
Metric, Threshold, Action → Save
```

> ⚠️ **EXAM TIP:** Scale **Up** = change tier (bigger VM). Scale **Out** = more instances. Auto-scale is only available at **Standard** tier and above.

---

## 13. Authentication & Authorization

- **Built-in authentication** (Easy Auth) — no code changes required
- Supports: Azure AD (Entra ID), Microsoft, Google, Facebook, Twitter/X, OpenID Connect
- Can enforce authentication for **all requests** or allow anonymous access

#### Portal Path
```
App Service → Settings → Authentication →
+ Add identity provider →
Provider: Microsoft / Google / Facebook / etc. →
Configure → Save
```

> ⚠️ **EXAM TIP:** App Service **Authentication (Easy Auth)** runs as a **middleware** in the platform — no code changes needed. It can be configured to require authentication or allow unauthenticated access.

---

## 14. Managed Identity

| Type | Description |
|---|---|
| **System-assigned** | Tied to the app — deleted when app is deleted |
| **User-assigned** | Standalone identity — can be shared across multiple apps |

#### Portal Path
```
App Service → Settings → Identity →
System assigned: Status → On → Save
OR
User assigned → + Add → Select identity → Add
```

> ⚠️ **EXAM TIP:** Use **Managed Identity** to access Azure resources (Key Vault, Storage, SQL) **without storing credentials** in app settings. System-assigned is simpler; user-assigned is reusable.

---

## 15. Security & RBAC

### Key Roles

| Role | Permissions |
|---|---|
| **Website Contributor** | Manage web apps (NOT App Service Plans) |
| **Web Plan Contributor** | Manage App Service Plans (NOT web apps) |
| **Contributor** | Full access to both |
| **Reader** | View-only |

> ⚠️ **EXAM TIP:** **Website Contributor** manages the web app. **Web Plan Contributor** manages the plan. They are **separate** roles — this distinction is tested.

---

## 16. Monitoring & Diagnostics

### App Service Logs

| Log Type | Windows | Linux |
|---|---|---|
| **Application Logging** | ✅ (Filesystem / Blob) | ✅ (Filesystem only) |
| **Web Server Logging** | ✅ | ❌ |
| **Detailed Error Messages** | ✅ | ❌ |
| **Failed Request Tracing** | ✅ | ❌ |
| **Deployment Logging** | ✅ | ✅ |

#### Portal Path — Enable Logs
```
App Service → Monitoring → App Service logs →
Application Logging: On → Level: Error/Warning/Info/Verbose →
Web server logging: File System / Storage →
Save
```

### Monitoring Tools

| Tool | Description |
|---|---|
| **Metrics** | CPU%, Memory%, Requests, Response Time, HTTP errors |
| **Log Stream** | Real-time log tail in portal |
| **Diagnose and solve problems** | AI-assisted troubleshooting |
| **Application Insights** | Deep APM (application performance monitoring) |
| **Health Check** | Periodic ping to app path — remove unhealthy instances |

#### Portal Path — Health Check
```
App Service → Monitoring → Health check →
Enable: Yes → Path: /healthz → Save
```

> ⚠️ **EXAM TIP:** **Health Check** pings a path at regular intervals. If instance fails health check multiple times, it's **removed from load balancer rotation** (not deleted). Requires **2+ instances** to be useful.

---

## 17. Pricing Key Points

| Tier | Billing |
|---|---|
| **Free (F1)** | Free (limited: 60 CPU-min/day) |
| **Shared (D1)** | Per-app compute charges |
| **Basic+** | Per App Service Plan (regardless of # apps) |
| **Isolated** | Per ASE + per plan |

- Charged for the **App Service Plan**, not per app
- Multiple apps on one plan = share cost
- Stopped apps in a paid plan **still incur charges** (plan is running)
- Deployment slots count as **separate instances** in terms of resources

> ⚠️ **EXAM TIP:** You pay for the **App Service Plan**, not individual apps. A stopped app in a Standard plan **still costs money** because the plan's VMs are still running. To stop billing → delete the plan or move to Free tier.

> ⚠️ **EXAM TIP:** Deployment slots **consume resources** of the plan. If you have 5 slots + production, all 6 share the plan's instances.

---

## 18. Limitations & Constraints

| Constraint | Limit |
|---|---|
| Apps per App Service Plan | **Unlimited** (but resources are shared) |
| App Service Plans per RG | **100** |
| Deployment slots (Standard) | **5** |
| Deployment slots (Premium/Isolated) | **20** |
| Custom domains per app | **500** |
| SSL certificates per app | **Varies by tier** |
| Instances (Standard) | **10** |
| Instances (Premium v3) | **30** |
| Instances (Isolated v2) | **100** |
| Backup max size | **10 GB** |
| Database backup max | **4 GB** |
| App name | `<name>.azurewebsites.net` — globally unique |

---

## 19. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| Create App Service Plan | `az appservice plan create -g <rg> -n <plan> --sku S1 --is-linux` |
| Create Web App | `az webapp create -g <rg> -p <plan> -n <name> --runtime "DOTNETCORE:8.0"` |
| List web apps | `az webapp list -g <rg> -o table` |
| Deploy ZIP | `az webapp deploy -g <rg> -n <name> --src-path app.zip --type zip` |
| Start/Stop/Restart | `az webapp start/stop/restart -g <rg> -n <name>` |
| Create slot | `az webapp deployment slot create -g <rg> -n <name> --slot staging` |
| Swap slots | `az webapp deployment slot swap -g <rg> -n <name> --slot staging --target-slot production` |
| Set app setting | `az webapp config appsettings set -g <rg> -n <name> --settings KEY=VALUE` |
| Set slot-sticky setting | `az webapp config appsettings set -g <rg> -n <name> --slot-settings KEY=VALUE` |
| Configure custom domain | `az webapp config hostname add -g <rg> --webapp-name <name> --hostname www.contoso.com` |
| Enable managed identity | `az webapp identity assign -g <rg> -n <name>` |
| Scale out | `az appservice plan update -g <rg> -n <plan> --number-of-workers 3` |
| Scale up | `az appservice plan update -g <rg> -n <plan> --sku P1v3` |
| Configure backup | `az webapp config backup create -g <rg> --webapp-name <name> --container-url <sas-url>` |
| View log stream | `az webapp log tail -g <rg> -n <name>` |

### PowerShell

| Action | Command |
|---|---|
| Create plan | `New-AzAppServicePlan -ResourceGroupName <rg> -Name <plan> -Location eastus -Tier Standard -NumberofWorkers 1 -WorkerSize Small` |
| Create web app | `New-AzWebApp -ResourceGroupName <rg> -Name <name> -AppServicePlan <plan>` |
| Create slot | `New-AzWebAppSlot -ResourceGroupName <rg> -Name <name> -Slot staging` |
| Swap slots | `Switch-AzWebAppSlot -ResourceGroupName <rg> -Name <name> -SourceSlotName staging -DestinationSlotName production` |

---

## 20. Quick-Fire Exam Points ⚡

1. App Service = **PaaS** — managed hosting for web apps, APIs, mobile backends
2. App Service Plan defines compute — one plan can host **multiple apps**
3. **Cannot mix Windows and Linux** apps in the same plan
4. **Free/Shared** = shared VMs. **Basic+** = dedicated VMs
5. **Deployment slots** start at **Standard** tier (5 slots). Premium/Isolated = 20 slots
6. **Auto-scale** available from **Standard** tier. Basic = manual scale only
7. **Always On** available from **Basic** tier — keeps app loaded. Default is **OFF**
8. Slot swap = **zero-downtime deployment** — staging ↔ production
9. Settings marked **"Deployment slot setting"** are **sticky** (stay with the slot)
10. **Custom domains** and **TLS bindings** are sticky to the slot — don't swap
11. **App settings** and **connection strings** swap UNLESS marked as slot-specific
12. **Auto swap** supported on **Windows only** — not Linux
13. Root domain → **A record**. Subdomain → **CNAME**. Both need **TXT** for verification
14. **Managed Certificate** = free, SNI only, no wildcard, no export
15. **VNet Integration** = outbound only to VNet. **Private Endpoint** = inbound only
16. VNet Integration requires **dedicated subnet** delegated to `Microsoft.Web/serverFarms`
17. Access Restrictions = IP/VNet/Service Tag allow/deny rules for inbound
18. SCM (Kudu) site has **separate** access restriction rules
19. Backup requires **Standard+** tier + Storage Account. Max **10 GB** (app + DB 4 GB)
20. **Health Check** removes unhealthy instances from LB — needs 2+ instances
21. **Website Contributor** = manage apps. **Web Plan Contributor** = manage plans. Different roles
22. **Managed Identity** = access Azure resources without credentials (System or User-assigned)
23. App Service **Easy Auth** = built-in authentication middleware, no code changes
24. Stopped app in paid plan **still costs money** — plan VMs are still running
25. Deployment slots **consume plan resources** — each slot uses plan capacity
26. App name must be **globally unique** (`<name>.azurewebsites.net`)
27. Kudu (SCM) site: `https://<appname>.scm.azurewebsites.net`
28. Max instances: Standard = **10**, Premium v3 = **30**, Isolated v2 = **100**
29. **ARR Affinity** = sticky sessions. Disable for stateless apps
30. HTTPS Only setting: redirects all HTTP to HTTPS
31. **Hybrid Connections** = connect to on-prem without VPN (relay-based, TCP)
32. **Isolated tier** (ASE) = app runs in your own **dedicated VNet** — highest isolation
33. App Service Plan per RG limit = **100**
34. Custom domains per app = **500**
35. `az webapp deployment slot swap` = CLI command for slot swapping

---

## 21. Step-by-Step Configuration Mind Maps 🗺️

---

### 21.1 Create Web App

> **Portal:** `Home → App Services → + Create → Web App`

```
Create Web App
│
├── Basics
│   ├── Subscription, Resource Group
│   ├── Name (globally unique → <name>.azurewebsites.net)
│   ├── Publish: Code / Docker Container / Static Web App
│   ├── Runtime stack: .NET 8 / Node 20 / Python 3.12 / Java 17 / PHP 8 / Ruby
│   ├── Operating System: Windows / Linux
│   │   ⚠️ Cannot mix Windows/Linux in same plan
│   ├── Region
│   └── App Service Plan:
│       ├── Select existing / Create new
│       ├── SKU: Free F1 / Basic B1 / Standard S1 / Premium P1v3
│       │   ⚠️ Free/Shared = shared VMs, no slots, no auto-scale
│       │   ⚠️ Standard+ = slots, auto-scale, VNet integration
│       └── ⚠️ All apps in plan share resources
│
├── Deployment
│   ├── Continuous deployment: Enable/Disable
│   └── GitHub Actions: Configure (optional)
│
├── Networking
│   ├── Enable public access: On/Off
│   ├── VNet Integration: Enable (Standard+)
│   └── Private Endpoint: Enable (Standard+)
│
├── Monitoring
│   ├── Application Insights: Enable → Create/Select
│   └── ⚠️ Recommended for production monitoring
│
├── Tags → Review + Create
│
└── RBAC: Website Contributor or Contributor
```

---

### 21.2 Configure Deployment Slots & Swap

> **Portal:** `App Service → Deployment → Deployment slots`

```
Configure Deployment Slots & Swap
│
├── Prerequisites
│   ├── App Service Plan: Standard tier or higher
│   │   ⚠️ Free/Shared/Basic = NO deployment slots
│   └── RBAC: Website Contributor or Contributor
│
├── Step 1: Create Slot
│   │   Portal: App Service → Deployment → Deployment slots → + Add slot
│   ├── Name: e.g., staging
│   │   URL: <appname>-staging.azurewebsites.net
│   ├── Clone settings from:
│   │   ├── Do not clone (empty slot)
│   │   └── Production / other slot (copies config)
│   └── Add
│
├── Step 2: Deploy to Slot
│   ├── Deploy code to the staging slot (Git, ZIP, FTP)
│   ├── Test at staging URL
│   └── Verify new version works correctly
│
├── Step 3: Mark Slot-Specific Settings
│   │   Portal: App Service (staging) → Configuration
│   ├── App settings → Edit → Deployment slot setting: ✅
│   │   ⚠️ Sticky settings = stay with slot, NOT swapped
│   │   Common: DB connection strings, environment name
│   └── Save
│
├── Step 4: Swap
│   │   Portal: App Service → Deployment → Deployment slots → Swap
│   ├── Source: staging
│   ├── Target: production
│   ├── Preview config changes
│   ├── Start Swap
│   │   ⚠️ Zero-downtime — users see no interruption
│   │   ⚠️ Swap can be reversed by swapping again
│   └── Swap completes in seconds
│
└── Auto Swap (Optional, Windows only)
    │   Portal: Staging slot → Configuration → General settings
    ├── Auto swap enabled: On
    ├── Auto swap deployment slot: production
    └── Save
        ⚠️ Auto swap NOT supported on Linux
```

---

### 21.3 Configure Custom Domain + SSL

> **Portal:** `App Service → Settings → Custom domains`

```
Configure Custom Domain + SSL
│
├── Prerequisites
│   ├── App Service Plan: Basic tier or higher (for SSL)
│   │   Shared tier supports custom domain but NO SSL
│   ├── Domain registered with a DNS registrar
│   └── RBAC: Website Contributor
│
├── Step 1: Get Verification ID
│   │   Portal: App Service → Custom domains → Custom domain verification ID
│   └── Copy the verification ID
│
├── Step 2: Configure DNS Records (at registrar)
│   ├── For SUBDOMAIN (e.g., www.contoso.com):
│   │   ├── CNAME: www → <appname>.azurewebsites.net
│   │   └── TXT: asuid.www → <verification-ID>
│   │
│   └── For ROOT DOMAIN (e.g., contoso.com):
│       ├── A record: @ → <app's IP address>
│       │   (find IP in Custom domains page)
│       └── TXT: asuid → <verification-ID>
│           ⚠️ Root domain must use A record (CNAME not at apex)
│
├── Step 3: Add Domain in Portal
│   │   Portal: App Service → Custom domains → + Add custom domain
│   ├── Enter domain name
│   ├── Validate → should show green checkmarks
│   └── Add
│
├── Step 4: Add SSL Certificate
│   │   Portal: App Service → Certificates → + Add
│   ├── Option 1: App Service Managed Certificate (free)
│   │   ⚠️ SNI only, no wildcard, no export
│   ├── Option 2: Upload PFX certificate
│   ├── Option 3: Import from Key Vault
│   └── Select → Create/Import
│
├── Step 5: Create SSL Binding
│   │   Portal: App Service → Custom domains → Select domain → Add binding
│   ├── Certificate: Select
│   ├── TLS/SSL type: SNI SSL (default) / IP SSL
│   └── Add
│
└── Step 6: Enforce HTTPS
    │   Portal: App Service → Configuration → General settings
    └── HTTPS Only: On → Save
        ⚠️ Redirects all HTTP → HTTPS
```

---

### 21.4 Configure VNet Integration

> **Portal:** `App Service → Settings → Networking`

```
Configure VNet Integration (Outbound)
│
├── Prerequisites
│   ├── App Service Plan: Standard v2 or higher
│   ├── VNet exists in same region as App Service
│   ├── Dedicated subnet available (delegated to Microsoft.Web/serverFarms)
│   │   ⚠️ Subnet must be EMPTY — no other resources
│   │   ⚠️ Minimum size: /28 (recommended /26)
│   └── RBAC: Website Contributor + Network Contributor
│
├── Step 1: Delegate Subnet
│   │   Portal: Virtual Network → Subnets → Select/Create subnet
│   ├── Subnet delegation: Microsoft.Web/serverFarms
│   └── Save
│
├── Step 2: Connect App to VNet
│   │   Portal: App Service → Settings → Networking
│   ├── Outbound traffic → VNet integration → + Add VNet
│   ├── VNet: Select
│   ├── Subnet: Select delegated subnet
│   └── Connect
│
├── Step 3: Configure Route All (Optional)
│   │   Portal: App Service → Networking → VNet integration → Select
│   └── Route All: Enabled
│       ⚠️ Routes ALL outbound traffic through VNet (including internet)
│       Without Route All, only RFC 1918 traffic goes through VNet
│
└── Result
    ├── App can access VNet resources (VMs, Private Endpoints, etc.)
    ├── App can access on-prem via VNet gateway
    ├── App's outbound IP = VNet range (if Route All enabled)
    └── ⚠️ Inbound traffic NOT affected — use Access Restrictions or Private Endpoints for inbound
```

---

### 21.5 Configure Auto-Scale

> **Portal:** `App Service Plan → Scale out`

```
Configure Auto-Scale
│
├── Prerequisites
│   ├── App Service Plan: Standard tier or higher
│   │   ⚠️ Basic = manual scale only
│   └── RBAC: Website Contributor or Contributor
│
├── Portal: App Service Plan → Settings → Scale out (App Service plan)
│
├── Option 1: Manual Scale
│   ├── Instance count: Set number (e.g., 3)
│   └── Save
│       ⚠️ All apps in plan run on all instances
│
├── Option 2: Custom Autoscale
│   ├── Default condition:
│   │   ├── Instance limits:
│   │   │   ├── Minimum: 2
│   │   │   ├── Maximum: 10 (Standard max) / 30 (Premium max)
│   │   │   └── Default: 2
│   │   │
│   │   ├── Scale-out rule: + Add
│   │   │   ├── Metric source: App Service Plan
│   │   │   ├── Metric: CPU Percentage
│   │   │   ├── Operator: Greater than
│   │   │   ├── Threshold: 70%
│   │   │   ├── Duration: 10 minutes
│   │   │   ├── Action: Increase count by 1
│   │   │   └── Cool down: 5 minutes
│   │   │
│   │   └── Scale-in rule: + Add
│   │       ├── Metric: CPU Percentage
│   │       ├── Threshold: Less than 30%
│   │       └── Action: Decrease count by 1
│   │
│   └── Schedule-based (optional): specific days/times
│
└── Save
    ⚠️ Always pair scale-out with scale-in rule
    ⚠️ Without scale-in, instances grow but never shrink
    ⚠️ Scaling affects ALL apps in the plan
```

---

### 21.6 Configure App Service Backup

> **Portal:** `App Service → Settings → Backups`

```
Configure App Service Backup
│
├── Prerequisites
│   ├── App Service Plan: Standard tier or higher
│   │   ⚠️ Free/Shared/Basic = NO backups
│   ├── Storage Account with Blob container
│   ├── SAS URL with write permissions to the container
│   └── RBAC: Website Contributor or Contributor
│
├── Step 1: Create Storage Container
│   │   Portal: Storage Account → Containers → + Container
│   └── Name: e.g., appbackups → Create
│
├── Step 2: Configure Backup
│   │   Portal: App Service → Settings → Backups → Configure
│   ├── Storage account: Select
│   ├── Container: Select
│   │
│   ├── Scheduled backup: On
│   │   ├── Frequency: Every X hours/days
│   │   ├── Start time
│   │   ├── Retention (days): e.g., 30
│   │   └── Keep at least one backup: Yes
│   │
│   ├── Include database: Yes (optional)
│   │   ├── Connection string type: SQLAzure / MySQL
│   │   ├── Connection string: Enter
│   │   └── ⚠️ Database backup max = 4 GB
│   │
│   └── Save
│       ⚠️ Total backup max = 10 GB (app + DB)
│
└── Manual Backup
    └── App Service → Backups → Backup now
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
