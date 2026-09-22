<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Application Gateway — AZ-104 Revision Notes

---

## 1. What is Azure Application Gateway?

- **Layer 7 (HTTP/HTTPS) web traffic load balancer** for Azure web applications
- Operates at **Application Layer (OSI Layer 7)** — understands HTTP headers, URLs, cookies, host names
- Enables **URL path-based routing**, **multi-site hosting**, **SSL/TLS termination**, **cookie-based session affinity**, **WAF**, **WebSocket**, **HTTP/2**
- Regional service — deployed within a single Azure region
- Fully managed PaaS — no infrastructure to maintain
- Deployed into a **dedicated subnet** within a VNet

---

## 2. Key Components

| Component | Purpose |
|---|---|
| **Frontend IP** | Public IP, Private IP, or both — receives client requests |
| **Listener** | Listens on a specific port/protocol/host; routes incoming requests |
| **Rule (Request Routing Rule)** | Maps a listener to a backend pool + HTTP settings |
| **Backend Pool** | Group of backend targets (VMs, VMSS, App Services, IPs, FQDNs) |
| **HTTP Settings (Backend Settings)** | Port, protocol, cookie-based affinity, connection draining, custom probe, host override, timeout |
| **Health Probe** | Checks backend health; unhealthy targets removed from rotation |
| **WAF Policy** | Web Application Firewall rules for Layer 7 protection (WAF_v2 SKU) |
| **Rewrite Rules** | Modify HTTP request/response headers and URL (v2 SKU) |
| **Redirect Configuration** | Redirect traffic to another listener, external URL, or HTTP→HTTPS |
| **SSL/TLS Certificate** | Used for HTTPS listeners and backend re-encryption |
| **URL Path Map** | Defines path-based routing rules (e.g., /images/* → pool-images) |

---

## 3. SKU Comparison — Standard v1 vs Standard v2 vs WAF v1 vs WAF v2

| Feature | **Standard v1** | **Standard v2** | **WAF v1** | **WAF v2** |
|---|---|---|---|---|
| **Autoscaling** | ❌ (fixed instance count) | ✅ | ❌ | ✅ |
| **Zone redundancy** | ❌ | ✅ | ❌ | ✅ |
| **Static VIP** | ❌ (dynamic) | ✅ (static) | ❌ (dynamic) | ✅ (static) |
| **WAF** | ❌ | ❌ | ✅ | ✅ |
| **Header rewrite** | ❌ | ✅ | ❌ | ✅ |
| **URL rewrite** | ❌ | ✅ | ❌ | ✅ |
| **Key Vault integration** | ❌ | ✅ | ❌ | ✅ |
| **AKS Ingress** | ❌ | ✅ (AGIC) | ❌ | ✅ (AGIC) |
| **Private Link** | ❌ | ✅ | ❌ | ✅ |
| **Mutual Auth (mTLS)** | ❌ | ✅ | ❌ | ✅ |
| **Pricing** | Fixed instances | Capacity units | Fixed instances | Capacity units |
| **Performance** | Lower | Higher | Lower | Higher |

> ⚠️ **EXAM TIP:** **v1 SKUs are deprecated — new deployments should always use v2.** v2 = autoscaling, zone redundancy, static VIP, header/URL rewrite, Key Vault integration.

> ⚠️ **EXAM TIP:** **WAF_v2** = most common exam answer when question mentions "web app protection + L7 load balancing". Standard v2 = no WAF.

---

## 4. v2 SKU Autoscaling

| Setting | Details |
|---|---|
| **Minimum instance count** | 0–125 (default: 0 for v2, set ≥ 2 for production) |
| **Maximum instance count** | 0–125 |
| **Manual mode** | Fix instance count (no autoscaling) |

- Scales based on: traffic load, CPU, current compute units, active connections
- **Minimum 2 instances** recommended for production (HA / SLA)

### Portal Path — Configure Autoscaling
```
Application Gateway → Settings → Configuration →
Tier: Standard V2 / WAF V2 →
Enable autoscaling: Yes →
Min instance count, Max instance count → Save
```

> ⚠️ **EXAM TIP:** Autoscaling with min 0 instances → gateway can scale to 0 during zero traffic (saves cost but increases cold-start latency). Set min ≥ 2 for production SLA.

---

## 5. Listeners

### Listener Types

| Type | Description | Use Case |
|---|---|---|
| **Basic** | Listens on a single domain (or all) on a port | Single-site hosting |
| **Multi-site** | Listens on a **specific hostname** (host header) on a port | Multiple websites on same gateway |

### Protocol Support
- **HTTP** (port 80 default)
- **HTTPS** (port 443 default) — requires SSL certificate
- **HTTP/2** — supported (client → gateway only)
- **WebSocket** — supported natively on all SKUs

### Multi-site Hosting
- Multiple **hostnames** on the same public IP / port
- Each hostname gets its own listener → own routing rule → own backend pool
- Supports **wildcard hostnames** (e.g., `*.contoso.com`) — v2 only
- Max **100+ listeners** per gateway (v2)

### Portal Path — Create Listener
```
Application Gateway → Settings → Listeners →
+ Add listener → Name, Frontend IP (Public/Private),
Port, Protocol (HTTP/HTTPS),
Listener type (Basic/Multi-site) →
If Multi-site: Enter Hostname(s) →
If HTTPS: Select/Upload SSL Certificate → Add
```

> ⚠️ **EXAM TIP:** **Multi-site listener** = routes based on **Host header**. Different from **path-based routing** (which uses URL path). Both can be combined.

> ⚠️ **EXAM TIP:** **Wildcard hostnames** supported only on **v2 SKU** and up to **5 wildcards per listener**.

---

## 6. Routing Rules (Request Routing Rules)

### Rule Types

| Type | Routing Logic |
|---|---|
| **Basic** | All requests from a listener → single backend pool |
| **Path-based** | Routes based on URL path patterns (e.g., `/images/*` → pool-A, `/api/*` → pool-B) |

### Path-based Routing
- Define **URL Path Map** with path rules
- Each path rule: URL pattern → specific backend pool + HTTP settings
- **Default backend pool** for unmatched paths (required)
- Patterns: `/images/*`, `/video/*`, `/api/v1/*`, etc.

### Portal Path — Create Routing Rule
```
Application Gateway → Settings → Rules →
+ Request routing rule → Name, Priority →
Listener tab: Select listener →
Backend targets tab:
  Target type: Backend pool
  Backend target: Select pool
  Backend settings: Select HTTP settings →
  (For path-based: + Add path-based rule → Add multiple paths) →
Add
```

> ⚠️ **EXAM TIP:** **Path-based routing** is a key differentiator from Azure Load Balancer (L4). If exam says "route `/images/*` to one pool and `/api/*` to another" → **Application Gateway**.

> ⚠️ **EXAM TIP:** Rules have a **Priority** value (lower number = higher priority). Priority is **required** in v2.

---

## 7. Backend Pool

### Supported Backend Targets

| Target Type | Examples |
|---|---|
| **Virtual Machines** | Azure VMs (by NIC) |
| **VMSS** | Virtual Machine Scale Sets |
| **App Services** | Azure App Service / Web Apps |
| **IP Addresses** | Any reachable IP (including on-prem via VPN/ER) |
| **FQDN** | Fully qualified domain names |

- Backend pool can mix target types (e.g., VMs + App Services)
- **Empty backend pool** → returns **502 Bad Gateway**

### Portal Path — Configure Backend Pool
```
Application Gateway → Settings → Backend pools →
+ Add → Name →
Target type: IP/FQDN, VM, VMSS, App Service →
Add targets → Add
```

> ⚠️ **EXAM TIP:** Application Gateway backend can include **App Services**, **on-prem servers** (via FQDN/IP with VPN/ER), and VMs. Azure Load Balancer backend = VMs/VMSS in VNet only.

> ⚠️ **EXAM TIP:** When using App Service as backend, you must **override hostname** in HTTP settings with the App Service FQDN — otherwise App Service returns 404.

---

## 8. Backend Settings (HTTP Settings)

| Setting | Details |
|---|---|
| **Backend protocol** | HTTP or HTTPS (for re-encryption / end-to-end TLS) |
| **Backend port** | Port backend listens on (e.g., 80, 443, 8080) |
| **Cookie-based affinity** | Enable/Disable sticky sessions (uses `ApplicationGatewayAffinity` cookie) |
| **Affinity cookie name** | Custom name (v2 only) |
| **Connection draining** | Gracefully remove backend from pool (drain timeout: 1–3600 sec) |
| **Request timeout** | 1–86400 seconds (default: **20 seconds**) |
| **Override backend path** | Rewrite URL path sent to backend |
| **Override hostname** | Use specific hostname or pick from backend target |
| **Custom probe** | Associate a custom health probe |
| **Trusted root certificate** | Required when backend uses HTTPS with self-signed cert |

### Portal Path — Configure Backend Settings
```
Application Gateway → Settings → Backend settings →
+ Add → Name, Protocol (HTTP/HTTPS), Port,
Cookie-based affinity, Connection draining,
Request timeout, Override hostname, Custom probe → Add
```

> ⚠️ **EXAM TIP:** **Request timeout default = 20 seconds**. If backend takes longer → 502 error. Increase timeout for long-running requests.

> ⚠️ **EXAM TIP:** **Connection draining** = ensures existing connections complete before removing a backend instance (important during updates/scaling).

---

## 9. Health Probes

### Default Probe vs Custom Probe

| Feature | Default Probe | Custom Probe |
|---|---|---|
| **URL path** | `/` | Any custom path (e.g., `/health`) |
| **Interval** | 30 seconds | 1–86400 seconds |
| **Timeout** | 30 seconds | 1–86400 seconds |
| **Unhealthy threshold** | 3 | 1–20 |
| **Protocol** | Matches HTTP settings | HTTP / HTTPS |
| **Host** | From HTTP settings | Custom or from HTTP settings |
| **Port** | From HTTP settings | 1–65535 or from HTTP settings |
| **Status code match** | 200–399 | Custom range (e.g., 200-399, 200-204) |

### Custom Probe Settings

| Setting | Default | Range |
|---|---|---|
| **Interval** | 30 seconds | 1–86400 sec |
| **Timeout** | 30 seconds | 1–86400 sec |
| **Unhealthy threshold** | 3 failures | 1–20 |
| **Healthy status codes** | 200–399 | Custom (e.g., 200-204) |
| **Probe matching (body)** | None | Match body text (up to 4 KB) |

### Portal Path — Create Custom Health Probe
```
Application Gateway → Settings → Health probes →
+ Add → Name, Protocol (HTTP/HTTPS), Host, Port, Path,
Interval, Timeout, Unhealthy threshold →
HTTP status code match (e.g., 200-399) →
Body match (optional) → Test → Add
```

> ⚠️ **EXAM TIP:** Default probe accepts **200–399** as healthy. Custom probe can narrow this. Body matching = checks if response body contains a specific string.

> ⚠️ **EXAM TIP:** Default probe is auto-created ONLY when no custom probes are configured. Once you add custom probes, associate them via HTTP settings.

---

## 10. SSL/TLS Termination & End-to-End TLS

### SSL Offloading (Termination at Gateway)
- Client → **HTTPS** → Application Gateway → **HTTP** → Backend
- Gateway terminates SSL, sends unencrypted traffic to backend
- Reduces CPU load on backend servers

### End-to-End TLS (Re-encryption)
- Client → **HTTPS** → Application Gateway → **HTTPS** → Backend
- Gateway terminates SSL, re-encrypts to backend
- Required for compliance (data encrypted in transit everywhere)
- Need: **Trusted root certificate** of backend's CA in HTTP settings (for self-signed) or backend cert from well-known CA

### SSL Policy
- Controls **TLS protocol versions** and **cipher suites**
- Three types:
  | Policy Type | Description |
  |---|---|
  | **Predefined** | Microsoft-managed policies (e.g., AppGwSslPolicy20220101) |
  | **Custom** | Choose specific TLS versions + cipher suites |
  | **CustomV2** | More granular control (v2 only) |
- Minimum TLS version: **TLS 1.0, 1.1, 1.2, or 1.3**

### Portal Path — Upload SSL Certificate
```
Application Gateway → Settings → Listeners →
Select HTTPS listener → Certificate: Upload PFX / Select from Key Vault →
Save
```

### Portal Path — Configure SSL Policy
```
Application Gateway → Settings → SSL settings →
Policy type: Predefined / Custom →
Min TLS version → Select cipher suites → Save
```

> ⚠️ **EXAM TIP:** For **end-to-end TLS**: frontend listener = HTTPS + backend settings = HTTPS. Backend cert must be trusted by the gateway (upload root cert or use well-known CA).

> ⚠️ **EXAM TIP:** **Key Vault integration** (v2 only) — store SSL certs in Key Vault, gateway auto-renews. Gateway needs **managed identity** with `GET` secret permission on Key Vault.

---

## 11. Web Application Firewall (WAF)

### Overview
- **WAF = Layer 7 firewall** protecting against web exploits (SQL injection, XSS, etc.)
- Requires **WAF v1 or WAF v2** SKU
- Based on **OWASP Core Rule Sets (CRS)**: 3.2, 3.1, 3.0, 2.2.9
- Can also use **Microsoft Bot Manager rule set**

### WAF Modes

| Mode | Behavior |
|---|---|
| **Detection** | Logs threats only — does NOT block traffic |
| **Prevention** | Blocks requests matching rules (returns 403 Forbidden) |

### WAF Policy Components

| Component | Purpose |
|---|---|
| **Managed Rules** | OWASP CRS rules (auto-updated by Microsoft) |
| **Custom Rules** | User-defined match conditions + actions (allow, block, log, rate-limit) |
| **Exclusions** | Exclude specific headers, cookies, or query strings from WAF inspection |
| **Per-site / Per-URI policy** | Apply different WAF policies to different listeners or paths |

### Portal Path — Configure WAF Policy
```
Home → + Create a resource → Search "Web Application Firewall" →
Create WAF Policy →
Policy for: Application Gateway →
Basics: Subscription, RG, Name, Region →
Managed Rules: Select CRS version (3.2 recommended) →
Custom Rules: + Add custom rule →
Association: Select Application Gateway / Listener →
Create
```

### Portal Path — Change WAF Mode
```
WAF Policy → Settings → Policy settings →
Mode: Detection / Prevention → Save
```

> ⚠️ **EXAM TIP:** **Detection mode = logs only**, **Prevention mode = blocks**. Start with Detection to tune rules, then switch to Prevention for production.

> ⚠️ **EXAM TIP:** WAF policy can be associated at **gateway level** (global) or **per-listener/per-path** level (granular). Per-site policies = v2 only.

---

## 12. URL Rewrite & Header Rewrite (v2 Only)

### Header Rewrite
- Modify **request headers** (to backend) and **response headers** (to client)
- Add, remove, or update any HTTP header
- Use cases: Add `X-Forwarded-For`, remove `Server` header, add security headers (HSTS, CSP)

### URL Rewrite
- Modify **URL path** and/or **query string** before routing to backend
- Does NOT change the path visible to the client (internal rewrite)

### Rewrite Rule Set
- Contains one or more **rewrite rules**
- Each rule: **Conditions** (optional, match on headers/server variables) + **Actions** (set/delete header, rewrite URL)
- Associated to a **routing rule** (not global)

### Portal Path — Create Rewrite Rule
```
Application Gateway → Settings → Rewrites →
+ Rewrite set → Name →
+ Add rewrite rule → Name, Sequence →
Conditions: Add condition (optional) →
  Variable: e.g., http_req_host, uri_path
  Pattern: regex or string
Actions:
  Action type: Set / Delete
  Header type: Request / Response
  Header name: e.g., X-Forwarded-For
  Header value: {var_add_x_forwarded_for_proxy} →
Add → Associate to routing rule → Create
```

> ⚠️ **EXAM TIP:** Rewrite rules are **v2 only**. Common exam scenario: add `X-Forwarded-For` header for backend to identify client IP, or strip `Server` header from responses.

---

## 13. Redirection

### Redirect Types

| Type | HTTP Code | Use Case |
|---|---|---|
| **Permanent** | 301 | Permanent URL change, SEO preservation |
| **Found** | 302 | Temporary redirect |
| **See Other** | 303 | Redirect POST → GET |
| **Temporary** | 307 | Temporary, preserves HTTP method |

### Redirect Targets
- **Listener** — redirect to another listener on the same gateway (e.g., HTTP → HTTPS)
- **External site** — redirect to any external URL

### Portal Path — Configure HTTP→HTTPS Redirect
```
Application Gateway → Settings → Rules →
Select HTTP rule (port 80) → Backend targets →
Target type: Redirection →
Redirect type: Permanent (301) →
Redirect target: Listener (select HTTPS listener) →
Save
```

> ⚠️ **EXAM TIP:** **HTTP→HTTPS redirect** is a very common exam scenario. Create two listeners (HTTP + HTTPS), routing rule on HTTP listener → redirect to HTTPS listener (301).

---

## 14. Application Gateway Subnet

- Requires a **dedicated subnet** (no other resources except other App Gateways)
- Subnet name: any (recommended: `AppGatewaySubnet` — but NOT mandatory unlike VPN GW)
- Recommended subnet size: **/24** (256 addresses)
- Minimum subnet size: depends on instances + private frontend IPs + reserved addresses

### Subnet Sizing
| Component | IPs Used |
|---|---|
| Per instance | 1 private IP each |
| Private frontend IP | 1 each |
| Azure reserved | 5 per subnet |

### Rules
- **No NSG restrictions on AzureLoadBalancer tag** — must allow `65200–65535` inbound for v2 (management traffic)
- **No UDRs** that override default route (0.0.0.0/0) — breaks gateway health
- Can coexist with **other Application Gateways** in the same subnet

### Portal Path — Select Subnet during Creation
```
Create Application Gateway → Basics → Virtual Network →
Select VNet → Select Subnet (dedicated) → Next
```

> ⚠️ **EXAM TIP:** Must allow **inbound ports 65200–65535** on NSG for v2 (infrastructure communication). Source: `GatewayManager` service tag. Blocking this = gateway goes unhealthy.

> ⚠️ **EXAM TIP:** Subnet must be **dedicated** — only App Gateways in it. Unlike VPN Gateway, the subnet name does NOT have to be `AppGatewaySubnet` (no fixed name requirement).

---

## 15. Session Affinity (Cookie-Based)

- Application Gateway uses **cookie-based affinity** (not IP-based like Azure LB)
- Gateway injects `ApplicationGatewayAffinity` cookie in the response
- Subsequent requests with this cookie → routed to the **same backend instance**
- Enabled per **HTTP Settings / Backend Settings**
- Custom cookie name supported (v2)

### Portal Path
```
Application Gateway → Settings → Backend settings →
Select or Add → Cookie based affinity: Enable → Save
```

> ⚠️ **EXAM TIP:** App Gateway = **cookie-based** affinity. Azure LB = **IP-based** (2/3/5-tuple) affinity. If exam says "sticky sessions using cookies" → App Gateway.

---

## 16. Connection Draining

- Gracefully removes backend pool members during planned updates
- Existing connections allowed to complete during the **drain timeout** period
- New connections routed to remaining healthy backends
- Timeout: **1–3600 seconds**
- Enabled per HTTP Settings / Backend Settings

### Portal Path
```
Application Gateway → Settings → Backend settings →
Select → Connection draining: Enable →
Drain timeout (seconds) → Save
```

---

## 17. Autoscaling & Performance (v2)

### Capacity Units (CU)
- v2 scales in **Capacity Units** — each CU provides:
  | Resource | Per CU |
  |---|---|
  | **Compute Units** | ~2,500 concurrent connections |
  | **Persistent connections** | ~2,500 |
  | **Throughput** | ~2.22 Mbps |

- Gateway auto-provisions CUs based on load (in autoscale mode)
- Max CUs depends on instance count × CUs per instance

### Performance Limits

| Metric | v1 | v2 |
|---|---|---|
| Max instances | Medium: up to 10, Large: up to 32 | 125 (autoscale) |
| Connections per instance | Varies by size | ~2,500 per CU |
| Throughput | ~200 Mbps (Large) | Scales with CUs |
| WAF throughput | Lower (V1) | Higher (V2) |

---

## 18. Private Frontend IP (Internal App Gateway)

- Application Gateway can have **Private frontend IP** (internal-only, no public IP)
- Also called **Internal Load Balancer (ILB) Application Gateway**
- Used for **internal-only web apps** not exposed to the internet
- Can have **both** Public + Private frontend IPs simultaneously
- v2 supports private-only deployment (no public IP required)

### Portal Path
```
Create Application Gateway → Frontends →
Frontend IP address type: Private (or Both) →
Select subnet → Assign Private IP (Static/Dynamic)
```

> ⚠️ **EXAM TIP:** v2 can be deployed with **private IP only** (no public IP at all). v1 always required a public IP even for internal scenarios.

---

## 19. Azure Application Gateway vs Azure Load Balancer

| Feature | **Application Gateway** | **Azure Load Balancer** |
|---|---|---|
| **OSI Layer** | Layer 7 (HTTP/HTTPS) | Layer 4 (TCP/UDP) |
| **Protocol** | HTTP, HTTPS, HTTP/2, WebSocket | TCP, UDP |
| **Routing** | URL path-based, host-based | Port-based (no content awareness) |
| **SSL Offloading** | ✅ | ❌ |
| **WAF** | ✅ (WAF SKU) | ❌ |
| **Cookie affinity** | ✅ | ❌ (IP-based only) |
| **URL Rewrite** | ✅ (v2) | ❌ |
| **Health Probes** | HTTP / HTTPS (+ body match) | TCP / HTTP / HTTPS |
| **Scope** | Regional | Regional (+ cross-region) |
| **Backend types** | VMs, VMSS, App Services, IPs, FQDNs | VMs, VMSS, IPs |
| **Autoscaling** | ✅ (v2) | ❌ (no concept of instances) |
| **Pricing** | Fixed + Capacity Units | Per rule + data processed |

> ⚠️ **EXAM TIP:** HTTP/HTTPS workloads needing L7 features → **App Gateway**. TCP/UDP or non-HTTP → **Load Balancer**. Need both? Chain them: App Gateway → internal LB → backend.

---

## 20. Azure Application Gateway vs Azure Front Door

| Feature | **Application Gateway** | **Azure Front Door** |
|---|---|---|
| **Scope** | Regional | Global |
| **Protocol** | HTTP/HTTPS | HTTP/HTTPS |
| **SSL Offloading** | ✅ | ✅ |
| **WAF** | ✅ | ✅ |
| **URL routing** | ✅ | ✅ |
| **Caching** | ❌ | ✅ (CDN built-in) |
| **Anycast IP** | ❌ | ✅ |
| **Backend** | Same region resources | Global (any Azure region, on-prem) |
| **Best for** | Regional L7 load balancing | Global L7 load balancing + CDN + WAF |

> ⚠️ **EXAM TIP:** Regional L7 = **App Gateway**. Global L7 with CDN = **Front Door**. If question says "users across multiple geographies" → Front Door.

---

## 21. Security & RBAC

### NSG Rules for Application Gateway Subnet

| Rule | Direction | Source | Destination | Ports | Action |
|---|---|---|---|---|---|
| Allow client traffic | Inbound | Internet / Any | App GW Subnet | 80, 443 | Allow |
| Allow management (v2) | Inbound | `GatewayManager` | App GW Subnet | **65200–65535** | Allow |
| Allow Azure infra | Inbound | `AzureLoadBalancer` | App GW Subnet | Any | Allow |
| Allow health probes | Inbound | `AzureLoadBalancer` | App GW Subnet | Probe port | Allow |

> ⚠️ **EXAM TIP:** `GatewayManager` service tag + ports **65200–65535** = **MANDATORY** for v2. Without this → App Gateway health = unhealthy / deployment fails.

### SSL Certificate Management
- **Key Vault integration (v2)**: Store PFX certs in Azure Key Vault
  - App Gateway uses **User-assigned managed identity** to access Key Vault
  - Key Vault access policy: `GET` on secrets
  - Auto-renewal: gateway checks Key Vault every **4 hours** for renewed cert
- **Direct upload**: Upload PFX file with password during listener config

### RBAC Roles

| Role | Permissions |
|---|---|
| **Network Contributor** | Full management of App Gateway and networking |
| **Contributor** | Full access to all resources |
| **Reader** | View-only |
| **Owner** | Full access + role assignment |
| **Web Application Firewall Policy Contributor** | Manage WAF policies |

### Key RBAC Permissions
| Action | Minimum Role |
|---|---|
| Create / Delete Application Gateway | Network Contributor (+ Contributor on subnet) |
| Modify rules, listeners, pools | Network Contributor |
| Create / modify WAF Policy | Contributor |
| View configuration | Reader |

---

## 22. Monitoring & Diagnostics

### Key Metrics

| Metric | Description |
|---|---|
| **Healthy Host Count** | Number of healthy backends per pool |
| **Unhealthy Host Count** | Number of unhealthy backends per pool |
| **Total Requests** | Count of successful requests served |
| **Failed Requests** | Count of requests that returned errors |
| **Response Status** | HTTP response status code distribution (2xx, 3xx, 4xx, 5xx) |
| **Throughput** | Bytes per second served |
| **Current Connections** | Active connections established |
| **Current Capacity Units** | CUs consumed (v2 — useful for scaling decisions) |
| **Compute Units** | CPU consumed |
| **Backend Response Status** | Response codes from backends |
| **Backend Connect Time** | Time to establish connection to backend |
| **Backend First Byte Response Time** | Time from connection to first byte from backend |
| **Backend Last Byte Response Time** | Time from connection to last byte from backend |

### Diagnostic Logs

| Log Category | Content |
|---|---|
| **Access Log** | All requests (client IP, URL, response code, latency) |
| **Performance Log** | Per-backend instance performance (requests served, throughput, healthy count) |
| **Firewall Log** | WAF detection/prevention events (matched rules, blocked requests) |

### Portal Path — View Metrics
```
Application Gateway → Monitoring → Metrics →
Select Metric (e.g., Healthy Host Count) →
Filter by backend pool → Apply
```

### Portal Path — Enable Diagnostic Logs
```
Application Gateway → Monitoring → Diagnostic settings →
+ Add diagnostic setting → Select categories:
  ApplicationGatewayAccessLog, ApplicationGatewayPerformanceLog,
  ApplicationGatewayFirewallLog →
Destination: Log Analytics / Storage / Event Hub → Save
```

### Portal Path — Configure Alerts
```
Application Gateway → Monitoring → Alerts →
+ New alert rule → Signal: Unhealthy Host Count > 0 →
Action Group: Email/SMS/Webhook → Create
```

> ⚠️ **EXAM TIP:** **Unhealthy Host Count** and **Healthy Host Count** are the most important backend health metrics. Monitor **Response Status** for 502/504 errors.

> ⚠️ **EXAM TIP:** **502 Bad Gateway** = backend unreachable, probe failing, timeout too short, empty backend pool, or NSG blocking backend. This is a very commonly tested troubleshooting scenario.

---

## 23. Common 502 Error Causes & Troubleshooting

| Cause | Solution |
|---|---|
| Backend pool is empty | Add backend targets |
| All backends unhealthy (probe failing) | Fix health probe path / port / status code |
| NSG blocking traffic on backend | Allow App GW subnet as source |
| Backend timeout exceeded | Increase request timeout in HTTP settings |
| Backend not listening on configured port | Verify backend app port matches HTTP settings |
| Custom probe misconfigured | Check path, host, port, expected status |

---

## 24. Pricing Key Points

| Component | Cost |
|---|---|
| **v1: Fixed capacity** | Charged per gateway-hour (based on size: Small/Medium/Large) |
| **v2: Capacity Units** | Fixed cost (per gateway-hour) + consumption cost (per CU-hour) |
| **Data processed** | Per GB (first 10 GB/month free on some tiers) |
| **WAF** | Higher per-hour rate than Standard |
| **Outbound data transfer** | Standard Azure egress charges |

### Cost Optimization
- v2 autoscaling: set **min instances = 0** for dev/test (no cost when idle)
- Use **Standard v2** if WAF not needed (cheaper than WAF v2)
- Monitor **Current Capacity Units** metric to right-size

> ⚠️ **EXAM TIP:** v2 = **fixed cost + variable CU cost**. v1 = **fixed cost per instance**. v2 can be more cost-effective due to autoscaling.

---

## 25. Limitations & Constraints

| Constraint | Limit |
|---|---|
| Max instances per gateway (v2) | **125** |
| Max listeners per gateway | **100** (v2) |
| Max routing rules | **100** (v2) |
| Max backend pools | **100** |
| Max backends per pool | **1,200** |
| Max HTTP settings | **100** |
| Max health probes | **100** |
| Max rewrite rule sets | **100** |
| Max rewrite rules per set | **100** |
| Max URL path map rules | **100** |
| Idle timeout | **4 minutes** (fixed, not configurable) |
| Request timeout | Default **20 sec**, max **86400 sec** (24 hrs) |
| Max request header size | **32 KB** (v1), **16 KB** (v2 default, configurable to 64 KB) |
| Max request body size (WAF) | **128 KB** (v1), **2 MB** (v2 CRS 3.2+), configurable |
| Max URL size | **32 KB** |
| Max SSL certificates | **100** |
| Subnet requirement | **Dedicated subnet** |
| NSG required ports (v2) | **65200–65535** inbound from `GatewayManager` |

> ⚠️ **EXAM TIP:** Max **125 instances (v2)**, **100 listeners**, **100 rules**. Idle timeout = **4 min (fixed)**. Request timeout default = **20 sec**.

---

## 26. CLI / PowerShell Commands (Exam-Relevant)

### Azure CLI

| Action | Command |
|---|---|
| Create App Gateway | `az network application-gateway create -g <rg> -n <name> --sku Standard_v2 --capacity 2 --vnet-name <vnet> --subnet <subnet> --public-ip-address <pip> --http-settings-port 80 --http-settings-protocol Http --frontend-port 80 --routing-rule-type Basic` |
| List App Gateways | `az network application-gateway list -g <rg>` |
| Show App Gateway | `az network application-gateway show -g <rg> -n <name>` |
| Add backend pool | `az network application-gateway address-pool create -g <rg> --gateway-name <gw> -n <pool> --servers <ip1> <ip2>` |
| Add health probe | `az network application-gateway probe create -g <rg> --gateway-name <gw> -n <probe> --protocol Http --path /health --interval 30 --timeout 30 --threshold 3` |
| Add HTTP settings | `az network application-gateway http-settings create -g <rg> --gateway-name <gw> -n <settings> --port 80 --protocol Http --cookie-based-affinity Disabled --timeout 20` |
| Create WAF policy | `az network application-gateway waf-policy create -g <rg> -n <policy>` |
| Stop App Gateway | `az network application-gateway stop -g <rg> -n <name>` |
| Start App Gateway | `az network application-gateway start -g <rg> -n <name>` |

### PowerShell

| Action | Command |
|---|---|
| Create App Gateway | `New-AzApplicationGateway -ResourceGroupName <rg> -Name <name> -Sku Standard_v2 ...` |
| Get App Gateway | `Get-AzApplicationGateway -ResourceGroupName <rg> -Name <name>` |
| Stop (deallocate) | `Stop-AzApplicationGateway -ApplicationGateway <gw>` |
| Start | `Start-AzApplicationGateway -ApplicationGateway <gw>` |

> ⚠️ **EXAM TIP:** `az network application-gateway stop` **deallocates** the gateway (no charges). Use for cost savings in non-production. `start` re-provisions it (new IP if not static).

---

## 27. Quick-Fire Exam Points ⚡

1. Application Gateway = **Layer 7 (HTTP/HTTPS)** load balancer — understands URLs, headers, cookies
2. Azure Load Balancer = **Layer 4 (TCP/UDP)** — cannot route by URL or host
3. **v2 SKU** = autoscaling, zone redundancy, static VIP, header/URL rewrite, Key Vault integration
4. **v1 SKU is deprecated** — always use v2 for new deployments
5. **WAF v2** = web application firewall with OWASP CRS rules; **Detection** (log) vs **Prevention** (block)
6. Must deploy in a **dedicated subnet** — no other resources (except other App GWs)
7. v2 NSG requirement: allow **inbound ports 65200–65535** from **GatewayManager** service tag
8. **Listeners**: Basic (single-site) vs Multi-site (hostname-based)
9. **Path-based routing** = route `/images/*` vs `/api/*` to different backend pools
10. **Cookie-based affinity** (not IP-based like Azure LB) — uses `ApplicationGatewayAffinity` cookie
11. **Default request timeout = 20 seconds** — increase for long-running APIs
12. **Default idle timeout = 4 minutes** (fixed, not configurable)
13. **Default health probe** = checks `/` path, accepts **200–399**, interval 30 sec, threshold 3
14. **502 Bad Gateway** = empty backend pool, all backends unhealthy, NSG blocking, timeout too short
15. **SSL offloading** = terminate TLS at gateway → HTTP to backend (reduces backend CPU)
16. **End-to-end TLS** = HTTPS listener + HTTPS backend setting (re-encryption)
17. **Key Vault integration (v2)** = store certs in Key Vault, auto-renew every 4 hours; needs managed identity
18. **App Service as backend** → must **override hostname** in HTTP settings with App Service FQDN
19. **HTTP→HTTPS redirect**: Create HTTP listener → routing rule → redirect (301) to HTTPS listener
20. **Rewrite rules (v2 only)** = modify request/response headers, URL path/query string
21. **Connection draining** = drain timeout 1–3600 seconds; graceful removal during updates
22. **Autoscaling v2**: min 0–125, max 0–125 instances. Min ≥ 2 for production SLA
23. **Private frontend IP** = internal-only App Gateway (ILB mode); v2 supports private-IP-only (no public)
24. Backend pool supports: **VMs, VMSS, App Services, IPs, FQDNs** (more flexible than Azure LB)
25. **WAF Detection mode** first → tune exclusions → switch to **Prevention mode** for production
26. WAF max request body: **128 KB (v1)**, **2 MB (v2 CRS 3.2+)**
27. `Stop-AzApplicationGateway` / `az .. stop` = **deallocates** (no charges) — v2 only
28. Max: **125 instances, 100 listeners, 100 rules, 100 backend pools, 1200 backends per pool**
29. **Regional** service — for global L7 use **Azure Front Door**
30. **Network Contributor** role = minimum to create/manage App Gateway
31. Diagnostic logs: **Access Log, Performance Log, Firewall Log (WAF)**
32. Multi-site hosting: different hostnames → different listeners → different backends (same gateway/IP)
33. Path-based and host-based routing can be **combined** for complex scenarios
34. v2 **static VIP** — IP does not change during gateway lifecycle (v1 = dynamic)
35. Wildcard hostnames (e.g., `*.contoso.com`) = **v2 only**, max 5 wildcards per listener

---

## 28. Step-by-Step Configuration Mind Maps 🗺️

---

### 28.1 Create Application Gateway (Standard V2)

> **Portal:** `Home → + Create a resource → Search "Application Gateway" → Create`

```
Create Application Gateway (Standard V2)
│
├── Step 1: Basics
│   ├── Subscription
│   ├── Resource Group
│   ├── Name
│   ├── Region
│   ├── Tier: Standard V2 / WAF V2
│   ├── Enable autoscaling: Yes / No
│   │   ├── If Yes: Min instances (0–125), Max instances (0–125)
│   │   │   ⚠️ Min ≥ 2 for production HA/SLA
│   │   └── If No: Instance count (manual)
│   ├── Availability Zone: Zone 1, 2, 3 (select one or more)
│   │   ⚠️ Zone selection is set at creation — cannot change later
│   ├── HTTP/2: Enabled / Disabled
│   ├── Virtual Network: Select or Create VNet
│   └── Subnet: Select dedicated subnet
│       ⚠️ Must be DEDICATED (no other resources)
│       ⚠️ Recommended /24 size
│       ⚠️ NSG must allow 65200-65535 from GatewayManager
│
├── Step 2: Frontends
│   ├── Frontend IP address type: Public / Private / Both
│   ├── Public IP: Create new or select existing
│   │   ├── SKU: Standard (for v2)
│   │   └── Assignment: Static
│   │       ⚠️ v2 = Static VIP (does not change)
│   └── Private IP (optional): Subnet, Static/Dynamic
│
├── Step 3: Backends
│   ├── + Add Backend Pool
│   ├── Name
│   ├── Add target without targets: Yes/No
│   │   ⚠️ Empty pool → 502 Bad Gateway
│   └── Target type: VM, VMSS, App Service, IP, FQDN
│       ⚠️ Can mix types in single pool
│
├── Step 4: Configuration (Routing Rules)
│   ├── + Add a routing rule
│   ├── Rule name, Priority (lower = higher)
│   │
│   ├── Listener tab
│   │   ├── Listener name
│   │   ├── Frontend IP: Public / Private
│   │   ├── Port: 80, 443, custom
│   │   ├── Protocol: HTTP / HTTPS
│   │   │   └── If HTTPS: Select certificate
│   │   │       ├── Upload PFX
│   │   │       └── Choose from Key Vault (v2, needs managed identity)
│   │   ├── Listener type: Basic / Multi-site
│   │   │   └── If Multi-site: Enter hostname(s)
│   │   └── Error page URL: Custom 403/502 pages (optional)
│   │
│   └── Backend targets tab
│       ├── Target type: Backend pool / Redirection
│       │   ├── If Backend pool:
│       │   │   ├── Select backend pool
│       │   │   ├── Backend settings: + Add new
│       │   │   │   ├── Name, Protocol, Port
│       │   │   │   ├── Cookie affinity, Connection draining
│       │   │   │   ├── Request timeout (default 20 sec)
│       │   │   │   ├── Override hostname (for App Service)
│       │   │   │   └── Custom probe (optional)
│       │   │   └── Path-based routing (optional)
│       │   │       ├── + Add path rule
│       │   │       ├── Path: e.g., /images/*
│       │   │       ├── Backend pool + Backend settings
│       │   │       └── ⚠️ Default pool = unmatched paths
│       │   └── If Redirection:
│       │       ├── Redirect type: 301/302/303/307
│       │       └── Target: Listener / External URL
│       └── Add
│
├── Step 5: Tags (Optional)
│
└── Step 6: Review + Create → Create
    ├── RBAC: Network Contributor or higher
    └── ⚠️ Deployment takes 5–10 minutes
```

---

### 28.2 Configure SSL/TLS Termination (HTTPS Listener)

> **Portal:** `Application Gateway → Settings → Listeners`

```
Configure SSL Termination
│
├── Prerequisites
│   ├── SSL/TLS certificate in PFX format or in Azure Key Vault
│   ├── If Key Vault: User-assigned managed identity with GET secret permission
│   └── v2 SKU recommended
│
├── Step 1: Navigate
│   └── Application Gateway → Settings → Listeners
│
├── Step 2: Add HTTPS Listener
│   ├── + Add listener
│   ├── Name
│   ├── Frontend IP: Public or Private
│   ├── Port: 443
│   ├── Protocol: HTTPS
│   ├── Choose a certificate:
│   │   ├── Option A: Upload PFX
│   │   │   ├── Upload .pfx file
│   │   │   └── Enter certificate password
│   │   └── Option B: Choose from Key Vault (v2 only)
│   │       ├── Select managed identity
│   │       ├── Select Key Vault
│   │       └── Select certificate
│   │       ⚠️ Gateway auto-checks Key Vault every 4 hours
│   ├── Listener type: Basic / Multi-site
│   └── Add
│
├── Step 3: Create Backend Settings (HTTP to backend)
│   ├── Protocol: HTTP (SSL offloading) or HTTPS (end-to-end)
│   ├── Port: 80 (offloading) or 443 (end-to-end)
│   │   ⚠️ If HTTPS → upload Trusted Root Certificate of backend CA
│   └── Save
│
├── Step 4: Create Routing Rule
│   ├── Listener: Select HTTPS listener
│   ├── Backend pool + Backend settings
│   └── Add
│
└── Step 5: (Optional) Add HTTP→HTTPS Redirect
    ├── Create HTTP listener (port 80)
    ├── Create routing rule for HTTP listener
    ├── Target type: Redirection
    ├── Redirect type: Permanent (301)
    └── Target: HTTPS listener
    ⚠️ Most common exam scenario for App Gateway
```

---

### 28.3 Configure End-to-End TLS

> **Portal:** `Application Gateway → Settings → Backend settings`

```
Configure End-to-End TLS (Re-encryption)
│
├── Prerequisites
│   ├── HTTPS listener with SSL certificate (frontend)
│   ├── Backend server running HTTPS
│   └── Backend CA root certificate (if self-signed)
│
├── Step 1: Frontend — HTTPS Listener
│   └── Already configured (see 28.2)
│
├── Step 2: Backend Settings — HTTPS
│   ├── Application Gateway → Settings → Backend settings
│   ├── + Add backend setting
│   ├── Protocol: HTTPS
│   ├── Port: 443 (or backend HTTPS port)
│   ├── Use well known CA certificate: Yes / No
│   │   ├── If Yes (backend uses public CA) → no extra cert needed
│   │   └── If No (self-signed) → Upload Trusted Root Certificate
│   │       ├── + Add certificate
│   │       ├── Name
│   │       └── Upload CER file (root CA cert)
│   │       ⚠️ Must be root CA cert, NOT backend server cert
│   ├── Override hostname: Backend target / Specific hostname
│   │   ⚠️ Required for App Service backends
│   └── Save
│
├── Step 3: Routing Rule
│   ├── Link HTTPS listener → HTTPS backend settings → backend pool
│   └── Save
│
└── ⚠️ End-to-end TLS = data encrypted everywhere (client → GW → backend)
    ⚠️ More CPU on backend vs SSL offloading
```

---

### 28.4 Configure WAF Policy

> **Portal:** `Home → + Create a resource → Web Application Firewall policy`

```
Configure WAF Policy
│
├── Prerequisites
│   ├── Application Gateway with WAF V2 SKU
│   └── RBAC: Contributor or WAF Policy Contributor
│
├── Step 1: Create WAF Policy
│   ├── Home → + Create a resource → Web Application Firewall
│   ├── Policy for: Application Gateway
│   ├── Subscription, Resource Group
│   ├── Policy Name
│   └── Region (must match App Gateway region)
│
├── Step 2: Policy Settings
│   ├── Mode: Detection (log only) / Prevention (block)
│   │   ⚠️ Start with Detection → tune → switch to Prevention
│   ├── Max request body size: 128 KB (v1) / configurable (v2)
│   └── File upload limit
│
├── Step 3: Managed Rules
│   ├── Select CRS Version: 3.2 (recommended) / 3.1 / 3.0
│   ├── Rule groups enabled by default
│   ├── Can disable individual rules per group
│   └── Microsoft Bot Manager rule set (optional)
│
├── Step 4: Custom Rules
│   ├── + Add custom rule
│   ├── Name, Priority (lower = higher)
│   ├── Match conditions:
│   │   ├── Match variable: IP address, Geo-location, URI, Header, etc.
│   │   ├── Operator: Contains, Equals, GeoMatch, etc.
│   │   └── Match values
│   └── Action: Allow / Block / Log / Rate limit
│       ⚠️ Custom rules evaluated BEFORE managed rules
│
├── Step 5: Exclusions
│   ├── + Add exclusion
│   ├── Applies to: Request header / Cookie / Query string / Body
│   ├── Selector: Specific header/cookie name
│   └── ⚠️ Reduces false positives for known-safe parameters
│
├── Step 6: Association
│   ├── Associate to: Application Gateway (global)
│   │   └── Or specific Listener / Path (per-site policy, v2 only)
│   └── Save
│
└── Step 7: Review + Create → Create
```

---

### 28.5 Configure Path-Based Routing

> **Portal:** `Application Gateway → Settings → Rules`

```
Configure Path-Based Routing
│
├── Prerequisites
│   ├── At least 2 backend pools (e.g., pool-images, pool-api)
│   ├── Backend settings for each pool
│   └── Listener configured
│
├── Step 1: Navigate
│   └── Application Gateway → Settings → Rules
│
├── Step 2: + Add Request Routing Rule
│   ├── Rule name
│   ├── Priority: Enter number (lower = higher)
│   │   ⚠️ Priority is REQUIRED in v2
│   └── Continue
│
├── Step 3: Listener Tab
│   ├── Select existing listener or create new
│   └── Continue to Backend targets
│
├── Step 4: Backend Targets Tab
│   ├── Target type: Backend pool
│   ├── Default backend pool: Select (handles unmatched paths)
│   │   ⚠️ Default pool is REQUIRED for path-based routing
│   ├── Default backend settings: Select
│   │
│   ├── + Add Path-based Rule
│   │   ├── Path: /images/* → Backend: pool-images
│   │   ├── Backend settings: Select
│   │   └── Add
│   │
│   ├── + Add Path-based Rule
│   │   ├── Path: /api/* → Backend: pool-api
│   │   ├── Backend settings: Select
│   │   └── Add
│   │
│   └── More paths as needed
│       ⚠️ Path patterns: exact match, wildcard (*)
│       ⚠️ Paths are case-sensitive
│
└── Step 5: Add → Rule created
    ⚠️ Unmatched paths → default backend pool
```

---

### 28.6 Configure HTTP to HTTPS Redirect

> **Portal:** `Application Gateway → Settings → Rules`

```
Configure HTTP→HTTPS Redirect
│
├── Prerequisites
│   ├── HTTPS listener on port 443 (with SSL cert)
│   └── HTTP listener on port 80
│
├── Step 1: Ensure HTTPS Listener Exists
│   └── Application Gateway → Listeners → Verify HTTPS (443) listener
│
├── Step 2: Create HTTP Listener (if not exists)
│   ├── + Add listener
│   ├── Name: e.g., http-listener
│   ├── Port: 80
│   ├── Protocol: HTTP
│   └── Add
│
├── Step 3: Create Routing Rule for HTTP Listener
│   ├── Application Gateway → Rules → + Add
│   ├── Rule name, Priority
│   ├── Listener: Select HTTP listener (port 80)
│   ├── Backend targets tab:
│   │   ├── Target type: Redirection
│   │   ├── Redirection type: Permanent (301)
│   │   │   ⚠️ 301 = SEO-friendly, cached by browsers
│   │   ├── Redirection target: Listener
│   │   ├── Target listener: Select HTTPS listener (port 443)
│   │   ├── Include path: ✅ Yes
│   │   └── Include query string: ✅ Yes
│   └── Add
│
└── Result: All HTTP requests → 301 redirect → HTTPS
    ⚠️ Very common exam scenario — know the exact steps
```

---

### 28.7 Configure Multi-Site Hosting

> **Portal:** `Application Gateway → Settings → Listeners`

```
Configure Multi-Site Hosting
│
├── Scenario: www.contoso.com → pool-A, www.fabrikam.com → pool-B
│   Both on same Application Gateway / same public IP
│
├── Step 1: Create Backend Pools
│   ├── Pool A: pool-contoso (VMs serving contoso.com)
│   └── Pool B: pool-fabrikam (VMs serving fabrikam.com)
│
├── Step 2: Create Backend Settings
│   ├── HTTP settings for each pool (port, timeout, etc.)
│   └── Override hostname if needed
│
├── Step 3: Create Multi-site Listeners
│   │
│   ├── Listener 1:
│   │   ├── Name: listener-contoso
│   │   ├── Frontend IP: Public
│   │   ├── Port: 443 (HTTPS) + SSL cert for contoso.com
│   │   ├── Listener type: Multi-site
│   │   └── Hostname: www.contoso.com
│   │
│   └── Listener 2:
│       ├── Name: listener-fabrikam
│       ├── Frontend IP: Public (same IP)
│       ├── Port: 443 (HTTPS) + SSL cert for fabrikam.com
│       ├── Listener type: Multi-site
│       └── Hostname: www.fabrikam.com
│
├── Step 4: Create Routing Rules
│   ├── Rule 1: listener-contoso → pool-contoso
│   └── Rule 2: listener-fabrikam → pool-fabrikam
│
└── Step 5: DNS Configuration
    ├── contoso.com → CNAME → App GW public IP DNS name
    └── fabrikam.com → CNAME → App GW public IP DNS name
    ⚠️ Single public IP serves multiple sites via Host header
```

---

### 28.8 Configure Header / URL Rewrite (v2 Only)

> **Portal:** `Application Gateway → Settings → Rewrites`

```
Configure Header / URL Rewrite
│
├── Step 1: Navigate
│   └── Application Gateway → Settings → Rewrites
│   ⚠️ v2 SKU only
│
├── Step 2: + Rewrite Set
│   ├── Name
│   └── Associate to routing rule(s)
│
├── Step 3: + Add Rewrite Rule
│   ├── Rule name
│   ├── Sequence: evaluation order (lower = first)
│   │
│   ├── Conditions (optional — when to apply)
│   │   ├── Check if header/variable: matches pattern
│   │   ├── Variable: http_req_Host, uri_path, query_string, etc.
│   │   ├── Pattern: regex or literal
│   │   └── Case insensitive: Yes/No
│   │
│   └── Actions
│       ├── Request Header Rewrite
│       │   ├── Action: Set / Delete
│       │   ├── Header name: e.g., X-Forwarded-For
│       │   └── Value: {var_add_x_forwarded_for_proxy}
│       │
│       ├── Response Header Rewrite
│       │   ├── Action: Set / Delete
│       │   ├── Header name: e.g., Server
│       │   └── Value: (empty to remove)
│       │
│       └── URL Rewrite
│           ├── URL path: new path (e.g., /newpath)
│           ├── Query string: new query string
│           └── Re-evaluate path map: Yes/No
│               ⚠️ Yes = re-evaluate against path rules after rewrite
│
└── Step 4: Save
    ⚠️ Common scenarios: Add X-Forwarded-For, remove Server header,
       add HSTS (Strict-Transport-Security), rewrite URL paths
```

---

### 28.9 Configure Health Probes (Custom)

> **Portal:** `Application Gateway → Settings → Health probes`

```
Configure Custom Health Probe
│
├── Step 1: Navigate
│   └── Application Gateway → Settings → Health probes
│
├── Step 2: + Add
│
├── Step 3: Configure Probe
│   ├── Name
│   ├── Protocol: HTTP / HTTPS
│   │   ⚠️ HTTPS probe validates backend certificate
│   ├── Host: hostname or "Pick hostname from backend settings"
│   │   ⚠️ For App Service → use "Pick from backend settings"
│   ├── Port: 80, 443, or custom
│   │   └── Or "Pick port from backend settings"
│   ├── Path: e.g., /health, /api/ping
│   │   ⚠️ Must return HTTP 200–399 (or custom code range)
│   ├── Interval: 30 sec (default), 1–86400 sec
│   ├── Timeout: 30 sec (default), 1–86400 sec
│   │   ⚠️ Timeout must be < Interval
│   ├── Unhealthy threshold: 3 (default), 1–20
│   ├── Use probe matching conditions: Yes/No
│   │   ├── Status code match: 200-399 (default)
│   │   │   └── Custom: e.g., 200-204, 200, 301
│   │   └── Body match: string to search in response body
│   │       ⚠️ Matches first 4 KB of response body only
│   └── Backend settings: Associate with HTTP settings
│
├── Step 4: Test → Verify probe result
│
└── Step 5: Add
    ⚠️ Default probe auto-created only when NO custom probes exist
    ⚠️ Custom probe must be explicitly linked in Backend Settings
```

---

### 28.10 Monitor Application Gateway

> **Portal:** `Application Gateway → Monitoring`

```
Monitor Application Gateway
│
├── View Metrics
│   │   Portal: Application Gateway → Monitoring → Metrics
│   │
│   ├── Backend Health
│   │   ├── Healthy Host Count → per backend pool
│   │   ├── Unhealthy Host Count → per backend pool
│   │   └── ⚠️ Unhealthy > 0 = investigate probe failures
│   │
│   ├── Performance
│   │   ├── Total Requests
│   │   ├── Failed Requests
│   │   ├── Response Status → filter by 2xx, 4xx, 5xx
│   │   │   ⚠️ Spike in 502 → backend unreachable
│   │   ├── Throughput (bytes/sec)
│   │   ├── Current Connections
│   │   └── Current Capacity Units (v2)
│   │
│   └── Backend Timing
│       ├── Backend Connect Time
│       ├── Backend First Byte Response Time
│       └── Backend Last Byte Response Time
│
├── Diagnostic Logs
│   │   Portal: Application Gateway → Monitoring → Diagnostic settings
│   │
│   ├── + Add diagnostic setting
│   ├── Log categories:
│   │   ├── ApplicationGatewayAccessLog → all requests
│   │   ├── ApplicationGatewayPerformanceLog → per-backend stats
│   │   └── ApplicationGatewayFirewallLog → WAF events
│   │       ⚠️ Firewall log only available on WAF SKU
│   ├── Destination: Log Analytics / Storage / Event Hub
│   └── Save
│
├── Configure Alerts
│   │   Portal: Application Gateway → Monitoring → Alerts
│   ├── + New alert rule
│   ├── Signal: Unhealthy Host Count / Response Status / Failed Requests
│   ├── Condition: e.g., Unhealthy Host Count > 0
│   ├── Action Group: Email / SMS / Webhook
│   └── Create
│
├── WAF Logs (WAF SKU)
│   │   Portal: WAF Policy → Monitoring → Diagnostic settings
│   ├── Firewall Log → shows matched rules, blocked requests
│   ├── Filter by: Rule ID, Action (Blocked/Detected), Client IP
│   └── ⚠️ Essential for tuning WAF exclusions
│
└── Connection Troubleshooting
    │   Portal: Application Gateway → Help → Connection troubleshoot
    └── Test connectivity to backend instances
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
