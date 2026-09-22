<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Container Instances (ACI) — AZ-104 Revision Notes

---

## 1. What is Azure Container Instances?

- **Fastest and simplest** way to run a container in Azure — no VM management
- **Serverless containers** — no infrastructure to provision or manage
- Runs **Linux** and **Windows** containers
- **Per-second billing** — pay only for what you use (vCPU + memory + GPU)
- Ideal for: burst workloads, batch jobs, CI/CD build agents, event-driven tasks, quick demos
- NOT for long-running production workloads needing orchestration → use **AKS** instead

> ⚠️ **EXAM TIP:** ACI = **simplest container hosting** (no orchestration). AKS = full **Kubernetes orchestration**. If the question mentions "no infrastructure," "quick deployment," or "short-lived tasks" → ACI.

---

## 2. Key Components

| Component | Description |
|---|---|
| **Container Group** | Collection of containers scheduled on the same host (share lifecycle, network, storage) |
| **Container** | Individual container instance (image, CPU, memory) |
| **Container Image** | Docker image from ACR, Docker Hub, or any registry |
| **Restart Policy** | Controls when containers restart (Always / OnFailure / Never) |
| **OS Type** | Linux or Windows |
| **Volumes** | Persistent storage (Azure Files, emptyDir, secret, gitRepo) |

---

## 3. Container Groups

- Top-level resource in ACI — analogous to a **pod** in Kubernetes
- Containers in a group share:
  - **Lifecycle** (start/stop together)
  - **Network** (same IP address, shared port namespace)
  - **Storage volumes**
  - **Host machine** (same underlying compute)
- Multi-container groups currently supported only on **Linux**
- Windows supports **single container per group only**

### Multi-Container Group Example
```
Container Group (Public IP: 52.x.x.x)
├── Container 1: Web app (port 80)
├── Container 2: Sidecar logger
└── Shared volume: Azure Files
```

> ⚠️ **EXAM TIP:** **Multi-container groups** are Linux only. Windows container groups support only **one container**. This is frequently tested.

> ⚠️ **EXAM TIP:** Containers in a group share the **same IP address** and **port namespace**. Container 1 uses port 80, Container 2 must use a different port (e.g., 8080).

---

## 4. ACI vs AKS vs App Service (Container Comparison)

| Feature | **ACI** | **AKS** | **App Service (Containers)** |
|---|---|---|---|
| **Type** | Serverless containers | Managed Kubernetes | Managed PaaS |
| **Orchestration** | None | Full Kubernetes | Basic (App Service Plan) |
| **Scaling** | Manual / KEDA | Auto-scale (HPA, cluster) | Auto-scale (plan-based) |
| **Setup complexity** | Very simple | Complex | Medium |
| **Best for** | Short-lived tasks, burst | Microservices, full orchestration | Web apps in containers |
| **Startup time** | Seconds | Minutes (cluster) | Seconds |
| **Billing** | Per-second (CPU + RAM) | Per-node VM | Per App Service Plan |
| **Multi-container** | Container groups | Pods | Single container per app |
| **Persistent storage** | Azure Files, limited | Full (PV, PVC) | Mounted storage |
| **Networking** | Public IP / VNet | VNet native | VNet integration |
| **Windows containers** | ✅ | ✅ | ✅ |
| **GPU support** | ✅ (Linux) | ✅ | ❌ |

> ⚠️ **EXAM TIP:** ACI = **no orchestration**, **per-second billing**, **fast startup**. AKS = **Kubernetes orchestration**, **per-node billing**. If "simple one-off job" → ACI. If "microservices with auto-scaling" → AKS.

---

## 5. Restart Policies

| Policy | Behavior | Use Case |
|---|---|---|
| **Always** | Restart container whenever it stops (default) | Long-running services (web servers) |
| **OnFailure** | Restart only if container exits with non-zero code | Batch jobs with retry on failure |
| **Never** | Never restart — run once and stop | One-time tasks, data processing |

> ⚠️ **EXAM TIP:** Default restart policy = **Always**. For batch/one-time jobs, use **Never** or **OnFailure**. If restart policy is "Never," the container is billed until the group is deleted.

---

## 6. Networking

### Public Networking
- Container group gets a **public IP address** (optional)
- Exposed ports defined at container group level
- FQDN: optional DNS name label → `<label>.<region>.azurecontainer.io`
- No built-in load balancer between container groups

### Private Networking (VNet Deployment)
- Deploy ACI into a **VNet subnet**
- Container gets a **private IP** — no public exposure
- Requires a **dedicated subnet** (delegated to `Microsoft.ContainerInstance/containerGroups`)
- Subnet delegation required — no other resources allowed in the subnet
- Enables connectivity to VNet resources (VMs, databases, etc.)
- Can combine with NSGs and UDRs for traffic control

| Networking Mode | IP | Access | Use Case |
|---|---|---|---|
| **Public** | Public IP + optional FQDN | Internet-accessible | Public APIs, demos |
| **Private (VNet)** | Private IP in VNet | Internal only | Backend processing, secure workloads |
| **None** | No IP | No network access | Offline computation |

#### Portal Path — Network Configuration
```
Container Instances → Create → Networking tab →
Networking type: Public / Private / None →
DNS name label (public) / VNet + Subnet (private)
```

> ⚠️ **EXAM TIP:** VNet deployment requires a **dedicated, delegated subnet** (`Microsoft.ContainerInstance/containerGroups`). This is the same pattern as App Service VNet Integration — dedicated subnet concept.

> ⚠️ **EXAM TIP:** ACI in VNet cannot have a **public IP** — it is either public OR private, not both. Use Application Gateway or Azure Front Door for public-facing private ACI.

---

## 7. Storage & Volumes

| Volume Type | Description | Persistence |
|---|---|---|
| **Azure Files** | SMB share mounted into container | ✅ Persistent across restarts |
| **emptyDir** | Temporary directory shared between containers in a group | ❌ Lost when group stops |
| **secret** | Mount secrets as files (in-memory tmpfs) | ❌ In-memory only |
| **gitRepo** | Clone a Git repo at startup as a volume | ❌ Cloned at creation |

### Azure Files Volume Mount
- Requires: **Storage Account** + **File Share**
- Storage Account credentials passed at deployment
- Supports both **SMB** and **NFS** (Linux only for NFS)
- Multiple containers in a group can mount the same share

> ⚠️ **EXAM TIP:** **Azure Files** is the ONLY persistent volume option for ACI. emptyDir, secret, and gitRepo are ephemeral. If exam asks "persistent storage for containers in ACI" → Azure Files.

> ⚠️ **EXAM TIP:** To mount Azure Files, you must provide the **Storage Account name** and **access key** at deployment time.

---

## 8. Container Registry Integration (ACR)

- ACI can pull images from **Azure Container Registry (ACR)**, Docker Hub, or any private registry
- For ACR:
  - Use **admin account** credentials (not recommended for production)
  - Use **service principal** with AcrPull role
  - Use **managed identity** (recommended)

### Authentication Methods

| Method | Security Level | Setup |
|---|---|---|
| **ACR Admin Account** | Low (shared credentials) | Enable in ACR → Use username/password |
| **Service Principal** | Medium | Create SP with AcrPull → pass client ID + secret |
| **Managed Identity** | High (no credentials) | Assign managed identity with AcrPull role |

#### Portal Path — Configure Image Source
```
Container Instances → Create → Basics →
Image source: Azure Container Registry / Docker Hub / Other →
Registry, Image, Tag → Configure
```

> ⚠️ **EXAM TIP:** For secure image pulls, use **managed identity** or **service principal** with **AcrPull** role. ACR admin account is for dev/test only.

---

## 9. Environment Variables & Secrets

- Pass configuration to containers via **environment variables**
- **Secure values**: marked as secure — not shown in portal/CLI output (hidden)
- Similar to Kubernetes Secrets for individual values

#### Portal Path
```
Container Instances → Create → Advanced →
Environment variables → Add →
Key, Value, Secure: Yes/No
```

#### CLI
```bash
az container create ... \
  --environment-variables 'ENV1'='value1' \
  --secure-environment-variables 'DB_PASSWORD'='secret123'
```

> ⚠️ **EXAM TIP:** **Secure environment variables** are write-only — they are NOT visible in the Azure Portal, CLI output, or logs after creation. Regular environment variables ARE visible.

---

## 10. Container Commands & Exec

- Override the container's default **command line** at creation
- Execute commands in a running container using `az container exec`
- View container logs with `az container logs`

#### CLI — Exec into Container
```bash
# Interactive shell
az container exec -g <rg> -n <name> --exec-command "/bin/bash"

# Specific container in multi-container group
az container exec -g <rg> -n <name> --container-name <container> --exec-command "/bin/sh"
```

#### CLI — View Logs
```bash
az container logs -g <rg> -n <name>
az container logs -g <rg> -n <name> --container-name <container>
```

---

## 11. Resource Allocation

### CPU & Memory

| Resource | Minimum | Maximum (Linux) | Maximum (Windows) |
|---|---|---|---|
| **vCPU** | 0.1 (1 core = 1.0) | 4 per container | 4 per container |
| **Memory (GB)** | 0.1 | 16 GB per container | 16 GB per container |
| **GPU** | 0 | Available (Linux only) | ❌ |

### Container Group Limits

| Resource | Limit |
|---|---|
| vCPU per group | **4** (standard regions) |
| Memory per group | **16 GB** (standard regions) |
| Containers per group | **60** |
| Volumes per group | **20** |
| Ports per group | **25** |
| Container group max size (image) | Varies by region |

> ⚠️ **EXAM TIP:** Default ACI limits are **4 vCPUs** and **16 GB RAM** per container group. For more resources, request a quota increase or use AKS.

> ⚠️ **EXAM TIP:** GPU is available for ACI but **Linux only** — Windows does NOT support GPU.

---

## 12. YAML & ARM Template Deployment

### Deployment Methods

| Method | Multi-Container Groups | Common Use |
|---|---|---|
| **Azure CLI** | ❌ Single container only | Quick deployments |
| **YAML file** | ✅ (recommended) | Multi-container groups |
| **ARM template** | ✅ | Infrastructure as code, additional resources |
| **Azure Portal** | ❌ Single container only | Manual creation |

> ⚠️ **EXAM TIP:** Azure CLI and Portal only create **single-container groups**. For **multi-container groups**, use **YAML file** or **ARM template**.

### CLI with YAML
```bash
az container create -g <rg> --file deploy.yaml
```

### YAML Example
```yaml
apiVersion: '2021-10-01'
name: my-container-group
location: eastus
properties:
  containers:
  - name: web
    properties:
      image: nginx
      ports:
      - port: 80
      resources:
        requests:
          cpu: 1.0
          memoryInGb: 1.5
  - name: sidecar
    properties:
      image: busybox
      resources:
        requests:
          cpu: 0.5
          memoryInGb: 0.5
  osType: Linux
  restartPolicy: Always
  ipAddress:
    type: Public
    ports:
    - port: 80
```

---

## 13. Security & RBAC

### RBAC Roles

| Role | Permissions |
|---|---|
| **Contributor** | Full ACI management |
| **Reader** | View container groups |
| **Owner** | Full access + role assignment |
| **AcrPull** | Pull images from ACR (assign to managed identity/SP) |

### Security Features

| Feature | Description |
|---|---|
| **Managed Identity** | System or user-assigned identity for accessing Azure resources |
| **Secure Environment Variables** | Hidden values (not displayed in portal/CLI) |
| **VNet Deployment** | Private networking — no public exposure |
| **Azure Private DNS** | Name resolution for VNet-deployed containers |
| **Confidential Containers** | Hardware-based TEE (Trusted Execution Environment) — preview |

#### Portal Path — Managed Identity
```
Container Instances → Create → Advanced →
Managed identity: System assigned / User assigned
```

> ⚠️ **EXAM TIP:** ACI supports **managed identity** (system and user-assigned) for secure Azure resource access. Use it to pull images from ACR without storing credentials.

---

## 14. Container Group Lifecycle

| State | Description | Billing |
|---|---|---|
| **Pending** | Pulling image, allocating resources | ❌ |
| **Running** | All containers running | ✅ Billed |
| **Succeeded** | All containers exited successfully (code 0) | ✅ Billed until deleted |
| **Failed** | One or more containers failed | ✅ Billed until deleted |
| **Stopped** | Manually stopped | ❌ Not billed |

### Start / Stop / Delete

| Action | Effect |
|---|---|
| **Stop** | Stops all containers — deallocates resources — no billing |
| **Start** | Restarts stopped group (same config, new resources) |
| **Delete** | Removes the resource completely |
| **Restart** | Restarts containers in the group |

#### Portal Path
```
Container Instance → Overview → Stop / Start / Restart / Delete
```

> ⚠️ **EXAM TIP:** Containers in **Succeeded** or **Failed** state are **still billed** until you **delete** or **stop** the container group. Just exiting is not enough to stop billing.

---

## 15. Monitoring & Diagnostics

### Metrics

| Metric | Description |
|---|---|
| **CPU Usage** | CPU utilization (millicores) |
| **Memory Usage** | Memory in bytes |
| **Network Bytes Received/Sent** | Network traffic |

#### Portal Path — Metrics
```
Container Instance → Monitoring → Metrics →
Metric: CPU Usage / Memory Usage → Apply
```

### Logs & Diagnostics

| Tool | Description | Path |
|---|---|---|
| **Container Logs** | stdout/stderr output | Containers → Logs |
| **Events** | Creation, start, stop events | Containers → Events |
| **Diagnostic Settings** | Send logs to Log Analytics / Event Hubs | Diagnostic settings → Add |
| **Azure Monitor** | Alerts on metrics | Alerts → + Alert rule |

#### Portal Path — View Logs
```
Container Instance → Containers → Select container → Logs
```

#### Portal Path — View Events
```
Container Instance → Containers → Select container → Events
```

---

## 16. Pricing Key Points

| Component | Billing |
|---|---|
| **vCPU** | Per-second (per vCPU allocated) |
| **Memory** | Per-second (per GB allocated) |
| **GPU** | Per-second (per GPU) |
| **Windows containers** | Higher rate than Linux |
| **Networking** | Standard data transfer charges |
| **Stopped group** | ❌ No compute charges |
| **Succeeded/Failed (not deleted)** | ✅ Still charged |

- **Per-second** billing from group start to stop/delete
- **No minimum charge** per container
- **Windows** containers cost more than Linux
- Billed for **allocated** resources (not actual usage)

> ⚠️ **EXAM TIP:** ACI bills for **allocated** resources, not actual consumption. If you allocate 4 vCPUs but only use 1, you pay for 4. Choose resource allocations carefully.

> ⚠️ **EXAM TIP:** **Windows** containers are **more expensive** than Linux containers on ACI. Prefer Linux when possible for cost savings.

---

## 17. Limitations & Constraints

| Constraint | Limit |
|---|---|
| vCPU per container group | **4** (standard) |
| Memory per container group | **16 GB** (standard) |
| Containers per group | **60** |
| Volumes per group | **20** |
| Ports per group IP | **25** |
| Multi-container groups | **Linux only** |
| GPU | **Linux only** |
| Container OS | Linux or Windows (not mixed) |
| VNet + Public IP | **Cannot combine** — either public or private |
| Windows multi-container | ❌ Not supported |
| Persistent storage | **Azure Files only** |
| Auto-scaling | ❌ Not built-in (use KEDA with AKS virtual nodes) |
| Custom domains / SSL | ❌ Not native (use App Gateway / Front Door) |
| Container group deployment (CLI/Portal) | Single container only; YAML/ARM for multi |

---

## 18. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| Create container | `az container create -g <rg> -n <name> --image nginx --cpu 1 --memory 1.5 --ports 80 --dns-name-label <label> --os-type Linux` |
| Create from ACR | `az container create -g <rg> -n <name> --image <acr>.azurecr.io/<image>:<tag> --registry-login-server <acr>.azurecr.io --registry-username <user> --registry-password <pw>` |
| Create from YAML | `az container create -g <rg> --file deploy.yaml` |
| List containers | `az container list -g <rg> -o table` |
| Show details | `az container show -g <rg> -n <name>` |
| View logs | `az container logs -g <rg> -n <name>` |
| Exec into container | `az container exec -g <rg> -n <name> --exec-command "/bin/bash"` |
| Start | `az container start -g <rg> -n <name>` |
| Stop | `az container stop -g <rg> -n <name>` |
| Restart | `az container restart -g <rg> -n <name>` |
| Delete | `az container delete -g <rg> -n <name>` |
| Attach (stream output) | `az container attach -g <rg> -n <name>` |
| Export to YAML | `az container export -g <rg> -n <name> --file exported.yaml` |

### PowerShell

| Action | Command |
|---|---|
| Create | `New-AzContainerGroup -ResourceGroupName <rg> -Name <name> -Image nginx -OsType Linux -Cpu 1 -MemoryInGB 1.5 -Port 80 -DnsNameLabel <label>` |
| Get | `Get-AzContainerGroup -ResourceGroupName <rg> -Name <name>` |
| Get logs | `Get-AzContainerInstanceLog -ResourceGroupName <rg> -ContainerGroupName <name>` |
| Remove | `Remove-AzContainerGroup -ResourceGroupName <rg> -Name <name>` |

---

## 19. Quick-Fire Exam Points ⚡

1. ACI = **serverless containers** — fastest/simplest way to run containers in Azure
2. **No VM provisioning**, no Kubernetes cluster — fully managed
3. Supports **Linux** and **Windows** containers
4. **Per-second billing** based on allocated vCPU + memory + GPU
5. Container **Group** = top-level resource (shares network, storage, lifecycle)
6. **Multi-container groups** = **Linux only**. Windows = single container per group
7. Containers in a group share the **same IP address** and **port namespace**
8. Default restart policy = **Always**. Use **Never** for one-time jobs
9. Max vCPU per group = **4** (standard). Max memory = **16 GB**
10. Max containers per group = **60**
11. GPU support = **Linux only**
12. Persistent storage = **Azure Files only** (emptyDir, secret, gitRepo are ephemeral)
13. Mount Azure Files requires **Storage Account name + key** at deployment
14. **VNet deployment** = private IP, dedicated delegated subnet
15. Cannot have **public IP + VNet** simultaneously — one or the other
16. FQDN format: `<label>.<region>.azurecontainer.io`
17. Portal and CLI create **single-container groups** only. YAML/ARM for multi-container
18. **Secure environment variables** = hidden after creation (write-only)
19. Image sources: **ACR, Docker Hub, any registry**
20. For ACR: use **managed identity** (recommended) or service principal with **AcrPull**
21. ACR admin account = dev/test only (shared credentials, not recommended)
22. `az container exec` = interactive shell into running container
23. `az container logs` = view stdout/stderr
24. Containers in **Succeeded/Failed** state are **still billed** until stopped or deleted
25. **Stopped** container group = no billing
26. **Windows** containers cost **more** than Linux
27. Billed for **allocated** resources, not actual usage
28. ACI vs AKS: ACI = simple/fast/no orchestration. AKS = full Kubernetes
29. No built-in **auto-scaling** — use KEDA/AKS virtual nodes for scaling
30. No native **custom domains or SSL** — use App Gateway or Front Door

---

## 20. Step-by-Step Configuration Mind Maps 🗺️

---

### 20.1 Create Container Instance (Single Container)

> **Portal:** `Home → Container instances → + Create`

```
Create Container Instance
│
├── Basics
│   ├── Subscription, Resource Group
│   ├── Container name (lowercase, alphanumeric, hyphens)
│   ├── Region
│   ├── Availability zones (optional)
│   ├── SKU: Standard / Confidential
│   ├── Image source:
│   │   ├── Azure Container Registry → Select registry, image, tag
│   │   │   ⚠️ ACR must have admin/SP/MI access configured
│   │   ├── Docker Hub → Image name: e.g., nginx
│   │   └── Other registry → Server URL, username, password
│   ├── OS type: Linux / Windows
│   │   ⚠️ Windows = single container only, higher cost
│   └── Size:
│       ├── vCPU: 1 (default, max 4 per group)
│       └── Memory (GB): 1.5 (default, max 16 per group)
│
├── Networking
│   ├── Networking type:
│   │   ├── Public: Public IP + optional FQDN
│   │   │   ├── DNS name label: <label> → <label>.<region>.azurecontainer.io
│   │   │   └── Ports: 80/TCP (add more as needed)
│   │   ├── Private: Deploy into VNet
│   │   │   ├── VNet: Select
│   │   │   ├── Subnet: Select (delegated to Microsoft.ContainerInstance)
│   │   │   │   ⚠️ Dedicated subnet required — no other resources
│   │   │   └── Ports: Configure
│   │   └── None: No network access
│   │       ⚠️ Cannot change between Public/Private after creation
│   └── ⚠️ Cannot combine Public IP + VNet deployment
│
├── Advanced
│   ├── Restart policy: Always / OnFailure / Never
│   │   ⚠️ Default = Always. Use Never for one-time tasks
│   ├── Environment variables:
│   │   ├── Key + Value (visible in portal)
│   │   └── Key + Secure Value (hidden after creation)
│   ├── Command override: Custom entrypoint (optional)
│   └── Managed identity:
│       ├── System assigned: On/Off
│       └── User assigned: + Add
│
├── Tags → Review + Create
│
└── RBAC: Contributor on the Resource Group
```

---

### 20.2 Create Multi-Container Group (YAML)

> **CLI:** `az container create -g <rg> --file deploy.yaml`

```
Create Multi-Container Group
│
├── Prerequisites
│   ├── OS type: Linux ONLY
│   │   ⚠️ Multi-container groups NOT supported on Windows
│   ├── YAML file prepared with container definitions
│   └── RBAC: Contributor on the Resource Group
│
├── Step 1: Create YAML File
│   ├── apiVersion: '2021-10-01'
│   ├── location: eastus
│   ├── Container 1 (primary):
│   │   ├── name, image, ports, CPU, memory
│   │   └── Volume mounts (optional)
│   ├── Container 2 (sidecar):
│   │   ├── name, image, CPU, memory
│   │   └── Volume mounts (optional)
│   ├── Volumes (shared):
│   │   ├── Azure Files: share name, storage account, key
│   │   ├── emptyDir: {} (temporary shared directory)
│   │   └── secret: key-value pairs (in-memory)
│   ├── OS type: Linux
│   ├── Restart policy: Always / OnFailure / Never
│   └── IP address: Public (ports) / Private (VNet)
│
├── Step 2: Deploy
│   └── az container create -g <rg> --file deploy.yaml
│
├── Step 3: Verify
│   ├── az container show -g <rg> -n <group-name> -o table
│   ├── az container logs -g <rg> -n <group-name> --container-name <container1>
│   └── az container logs -g <rg> -n <group-name> --container-name <container2>
│
└── ⚠️ Notes
    ├── All containers share the same IP and port space
    ├── Max 60 containers per group
    ├── Total group: max 4 vCPU, 16 GB RAM
    └── Portal/CLI cannot create multi-container — YAML or ARM only
```

---

### 20.3 Mount Azure Files Volume

> **CLI:** `az container create ... --azure-file-volume-*`

```
Mount Azure Files Volume
│
├── Prerequisites
│   ├── Storage Account exists
│   ├── File share created in the Storage Account
│   ├── Storage Account access key available
│   └── ACI and Storage Account in same region (recommended)
│
├── Step 1: Create Storage Account + File Share
│   │   Portal: Storage Accounts → + Create → Create
│   │   Portal: Storage Account → File shares → + File share
│   ├── Share name: e.g., aci-share
│   └── Quota: Set size
│
├── Step 2: Get Storage Account Key
│   │   Portal: Storage Account → Access keys → Copy key1
│   └── Or CLI: az storage account keys list -g <rg> -n <sa>
│
├── Step 3: Create Container with Volume Mount
│   ├── CLI (single container):
│   │   az container create -g <rg> -n <name> \
│   │     --image nginx \
│   │     --azure-file-volume-account-name <storage-account> \
│   │     --azure-file-volume-account-key <key> \
│   │     --azure-file-volume-share-name aci-share \
│   │     --azure-file-volume-mount-path /mnt/data
│   │
│   └── YAML (multi-container):
│       volumes:
│       - name: filesharevolume
│         azureFile:
│           shareName: aci-share
│           storageAccountName: <account>
│           storageAccountKey: <key>
│       containers:
│       - name: app
│         volumeMounts:
│         - name: filesharevolume
│           mountPath: /mnt/data
│
├── Step 4: Verify
│   ├── az container exec -g <rg> -n <name> --exec-command "ls /mnt/data"
│   └── Upload a file to share → verify visible in container
│
└── ⚠️ Notes
    ├── Azure Files = ONLY persistent volume for ACI
    ├── emptyDir = lost on stop/restart
    ├── Storage Account key must be provided at creation
    └── Multiple containers can mount the same share
```

---

### 20.4 Deploy ACI in VNet (Private)

> **Portal:** `Container instances → Create → Networking → Private`

```
Deploy ACI to VNet
│
├── Prerequisites
│   ├── VNet exists in same region as ACI
│   ├── Dedicated subnet available (or create new)
│   │   ⚠️ Subnet must be delegated to Microsoft.ContainerInstance/containerGroups
│   │   ⚠️ No other resources can be in this subnet
│   ├── Minimum subnet size: /29 (recommended /24 for growth)
│   └── RBAC: Contributor + Network Contributor
│
├── Step 1: Prepare Subnet
│   │   Portal: VNet → Subnets → + Subnet (or select existing)
│   ├── Name: aci-subnet
│   ├── Address range: e.g., 10.0.1.0/24
│   ├── Subnet delegation: Microsoft.ContainerInstance/containerGroups
│   └── Save
│       ⚠️ Delegation locks subnet to ACI — cannot deploy other resources
│
├── Step 2: Create Container Instance
│   │   Portal: Container instances → + Create
│   ├── Basics: Name, Image, OS, Size (as normal)
│   └── Networking:
│       ├── Networking type: Private
│       ├── Virtual Network: Select VNet
│       ├── Subnet: Select aci-subnet (delegated)
│       ├── Ports: Configure
│       └── DNS name label: Not available (private)
│           ⚠️ No public IP — accessible only within VNet
│
├── Step 3: Configure DNS (Optional)
│   ├── Use Azure Private DNS zone for name resolution
│   └── Link private DNS zone to VNet
│
├── Step 4: Verify
│   ├── From VM in same VNet: curl <container-private-ip>:80
│   └── az container show → ipAddress.ip (private IP)
│
├── CLI:
│   az container create -g <rg> -n <name> \
│     --image nginx --os-type Linux \
│     --vnet <vnet-name> --subnet aci-subnet \
│     --ports 80
│
└── ⚠️ Notes
    ├── Cannot combine VNet with public IP
    ├── For public access → put App Gateway / Front Door in front
    ├── NSGs on subnet control inbound/outbound traffic
    └── Windows container groups supported in VNet (single container only)
```

---

### 20.5 Pull Image from ACR with Managed Identity

> **CLI:** `az container create ... --acr-identity`

```
ACI with ACR + Managed Identity
│
├── Prerequisites
│   ├── Azure Container Registry (ACR) exists with pushed image
│   ├── User-assigned managed identity created
│   │   (or use system-assigned — assigned after creation)
│   └── RBAC: Contributor on RG, ACR admin not needed
│
├── Step 1: Create User-Assigned Managed Identity
│   │   Portal: Managed Identities → + Create → Name, Region → Create
│   └── Copy the identity's resource ID and client ID
│
├── Step 2: Assign AcrPull Role to Identity
│   │   Portal: ACR → Access Control (IAM) → + Add role assignment
│   ├── Role: AcrPull
│   ├── Assign to: Managed Identity → Select the identity
│   └── Save
│       ⚠️ AcrPull is the minimum role needed to pull images
│
├── Step 3: Create ACI with Managed Identity
│   └── CLI:
│       az container create -g <rg> -n <name> \
│         --image <acr>.azurecr.io/myapp:latest \
│         --acr-identity <identity-resource-id> \
│         --assign-identity <identity-resource-id> \
│         --cpu 1 --memory 1.5 --ports 80
│
├── Step 4: Verify
│   ├── az container show -g <rg> -n <name> → check running state
│   └── az container logs -g <rg> -n <name>
│
└── ⚠️ Notes
    ├── Managed identity = no credentials stored anywhere
    ├── AcrPull = minimum role for image pulling
    ├── System-assigned MI cannot be used for initial pull (chicken-and-egg)
    │   → Use user-assigned MI for ACR pull
    └── ACR admin account is an alternative but less secure
```

---

### 20.6 Configure Container Restart Policy & Environment Variables

> **Portal:** `Container instances → Create → Advanced`

```
Configure Restart & Environment Variables
│
├── Portal: Container instances → + Create → Advanced tab
│
├── Restart Policy
│   ├── Always (default): Container restarts on any exit
│   │   Use for: Web servers, long-running services
│   ├── OnFailure: Restart only on non-zero exit code
│   │   Use for: Batch jobs with retry
│   └── Never: Run once, never restart
│       Use for: One-time tasks, migrations, data processing
│       ⚠️ Container stays in Succeeded/Failed state — still billed until deleted
│
├── Environment Variables
│   ├── + Add variable
│   │   ├── Name: APP_ENV
│   │   ├── Value: production
│   │   └── Secure: No (visible in portal)
│   ├── + Add secure variable
│   │   ├── Name: DB_PASSWORD
│   │   ├── Value: ••••••••
│   │   └── Secure: Yes (hidden after creation)
│   │       ⚠️ Cannot be viewed after deployment — write-only
│   └── ⚠️ Environment variables cannot be updated — must recreate group
│
├── Command Override (Optional)
│   └── Override default CMD/ENTRYPOINT
│       Example: ["/bin/sh", "-c", "echo hello && sleep 3600"]
│
└── CLI:
    az container create -g <rg> -n <name> --image nginx \
      --restart-policy OnFailure \
      --environment-variables 'APP_ENV'='production' \
      --secure-environment-variables 'DB_PASSWORD'='secret123' \
      --command-line "/bin/sh -c 'echo hello'"
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
