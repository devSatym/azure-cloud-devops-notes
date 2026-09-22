<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Kubernetes Service (AKS) — AZ-104 Revision Notes

---

## 1. What is Azure Kubernetes Service?

- **Managed Kubernetes** orchestration service for deploying, scaling, and managing containerized applications
- Azure manages the **control plane** (API server, etcd, scheduler, controller manager) — **free**
- You manage (and pay for) **worker nodes** (VMs that run your containers)
- Supports **Linux** and **Windows** node pools
- Integrates with ACR, Azure Monitor, Azure AD (Entra ID), Azure Policy, VNet

> ⚠️ **EXAM TIP:** AKS **control plane is FREE**. You only pay for the **worker node VMs**, associated storage, and networking. The control plane (API server, etcd) is fully managed by Azure.

---

## 2. Key Components

| Component | Description |
|---|---|
| **Cluster** | AKS resource containing control plane + node pools |
| **Control Plane** | Managed by Azure: API server, etcd, scheduler, controller |
| **Node Pool** | Group of VMs (nodes) with the same configuration |
| **System Node Pool** | Runs critical Kubernetes system pods (CoreDNS, metrics-server). Required |
| **User Node Pool** | Runs your application workloads. Optional but common |
| **Node** | Individual VM running kubelet and container runtime |
| **Pod** | Smallest deployable unit — one or more containers |
| **Deployment** | Declarative definition for pod replicas and updates |
| **Service** | Stable network endpoint to expose pods (ClusterIP, LoadBalancer, NodePort) |
| **Namespace** | Logical isolation within a cluster |
| **Ingress** | HTTP/HTTPS routing rules (Layer 7 load balancer) |
| **kubelet** | Agent on each node that manages pod lifecycle |
| **kubectl** | CLI tool to interact with Kubernetes API server |

---

## 3. AKS vs ACI vs App Service (Container Comparison)

| Feature | **AKS** | **ACI** | **App Service** |
|---|---|---|---|
| **Type** | Managed Kubernetes | Serverless containers | Managed PaaS |
| **Orchestration** | Full Kubernetes | None | Basic (plan-based) |
| **Scaling** | HPA, Cluster Autoscaler, KEDA | Manual / KEDA | Auto-scale (plan) |
| **Startup time** | Minutes (cluster) | Seconds | Seconds |
| **Best for** | Microservices, complex apps | Short tasks, burst | Web apps |
| **Control plane cost** | Free | N/A | N/A |
| **Compute cost** | Per node VM | Per-second CPU + RAM | Per App Service Plan |
| **Multi-container** | Pods (native) | Container groups (Linux only) | Single per app |
| **Networking** | Full VNet native | Public or VNet | VNet integration |
| **Windows** | ✅ (user node pool) | ✅ | ✅ |
| **Persistent storage** | Azure Disk, Azure Files (PV/PVC) | Azure Files only | Mounted storage |

> ⚠️ **EXAM TIP:** AKS = complex workloads with orchestration. ACI = simple, short-lived containers. If "microservices", "auto-healing", "rolling updates" → AKS. If "run one container quickly" → ACI.

---

## 4. Node Pools

### System Node Pool
- Runs Kubernetes **system pods** (CoreDNS, tunnelfront, konnectivity-agent, metrics-server)
- **At least one system pool** required per cluster
- Minimum recommended: **3 nodes** for production
- OS: Linux only (system pods require Linux)
- Taint: `CriticalAddonsOnly=true:NoSchedule` (optional, keeps app pods off system nodes)

### User Node Pool
- Runs **application workloads**
- Supports **Linux** and **Windows**
- Can have **multiple user pools** with different VM sizes
- Can scale independently (manual or auto-scale)
- Can have **0 nodes** (scale to zero)

| Feature | System Node Pool | User Node Pool |
|---|---|---|
| **Required** | ✅ Yes (at least 1) | ❌ Optional |
| **OS** | Linux only | Linux or Windows |
| **System pods** | ✅ Must run here | ❌ Can be excluded |
| **Scale to 0** | ❌ (min 1 node) | ✅ Yes |
| **Multiple per cluster** | ✅ | ✅ |
| **VM size change** | ❌ Cannot change (create new pool) | ❌ Cannot change (create new pool) |

> ⚠️ **EXAM TIP:** **System node pool** must have at least **1 node** running (cannot scale to 0). User node pools CAN scale to 0. System pool MUST be Linux.

> ⚠️ **EXAM TIP:** You **cannot resize** (change VM size of) an existing node pool. You must create a **new node pool** with the desired size and drain/delete the old one.

#### Portal Path — Add Node Pool
```
AKS Cluster → Settings → Node pools → + Add node pool →
Name, Mode (System/User), OS (Linux/Windows),
VM Size, Node count, Enable autoscale → Add
```

---

## 5. Scaling

### 5.1 Manual Scaling
- Set a fixed number of nodes in a node pool
- `az aks scale --node-count <n>`

### 5.2 Cluster Autoscaler
- Automatically adjusts **node count** based on pending pods
- Scales **out** when pods can't be scheduled (insufficient resources)
- Scales **in** when nodes are underutilized
- Configured per **node pool**
- Set **minimum** and **maximum** node count

#### Portal Path — Enable Cluster Autoscaler
```
AKS Cluster → Settings → Node pools → Select pool →
Scale method: Autoscale →
Min node count, Max node count → Apply
```

### 5.3 Horizontal Pod Autoscaler (HPA)
- Kubernetes-native — scales **pod replicas** based on CPU/memory metrics
- Configured via `kubectl autoscale deployment <name> --min=2 --max=10 --cpu-percent=50`
- Not directly configured in Azure Portal — use kubectl or YAML

### 5.4 ACI Virtual Nodes (Burst to ACI)
- Enables AKS to burst workloads to **ACI** when cluster is overloaded
- Uses **Virtual Kubelet** — ACI appears as a virtual node
- Requires: AKS with **Azure CNI** networking + ACI provider enabled
- **Linux pods only** for virtual nodes
- Near-instant scaling — no VM provisioning wait

| Scaling Type | What Scales | Scope |
|---|---|---|
| **Manual** | Nodes | Node pool |
| **Cluster Autoscaler** | Nodes | Node pool |
| **HPA** | Pod replicas | Deployment |
| **Virtual Nodes (ACI)** | Burst pods to ACI | Cluster |

> ⚠️ **EXAM TIP:** **Cluster Autoscaler** = scales **nodes**. **HPA** = scales **pods**. They work together: HPA scales pods → Cluster Autoscaler adds nodes if needed → pods get scheduled.

> ⚠️ **EXAM TIP:** **Virtual Nodes** (burst to ACI) require **Azure CNI** networking (not kubenet) and are **Linux only**.

---

## 6. Networking

### Network Models

| Feature | **Kubenet** | **Azure CNI** |
|---|---|---|
| **Pod IPs** | Pods get IPs from separate overlay | Pods get IPs from **VNet subnet** |
| **IP consumption** | Low (NAT for pods) | High (every pod gets a VNet IP) |
| **Network Policy** | Calico only | Azure NPM or Calico |
| **Performance** | Slightly lower (extra hop) | Native VNet performance |
| **VNet integration** | Via routing table (UDR) | Native — pods are directly in VNet |
| **Windows nodes** | ❌ Not supported | ✅ Supported |
| **Virtual Nodes (ACI)** | ❌ Not supported | ✅ Required |
| **Default** | ✅ Default for new clusters | Optional |
| **IP planning** | Simpler | Requires larger subnet |

> ⚠️ **EXAM TIP:** **Kubenet** = default, pods use overlay network (separate address space). **Azure CNI** = pods get VNet IPs directly (better integration but uses more IPs). Windows node pools and Virtual Nodes **require Azure CNI**.

> ⚠️ **EXAM TIP:** You **CANNOT change** the network model after cluster creation. Kubenet ↔ Azure CNI change requires cluster recreation.

### Kubernetes Service Types

| Type | Description |
|---|---|
| **ClusterIP** | Internal only — accessible within the cluster (default) |
| **NodePort** | Exposes on each node's IP at a static port |
| **LoadBalancer** | Creates an **Azure Load Balancer** (public or internal) |
| **ExternalName** | Maps to an external DNS name |

> ⚠️ **EXAM TIP:** `LoadBalancer` type service creates an **Azure Standard Load Balancer** by default. For internal-only → add annotation `service.beta.kubernetes.io/azure-load-balancer-internal: "true"`.

### Ingress Controllers
- Layer 7 (HTTP/HTTPS) routing — host-based, path-based routing
- Options: **NGINX Ingress Controller**, **Application Gateway Ingress Controller (AGIC)**
- AGIC integrates with Azure Application Gateway for WAF + SSL termination

---

## 7. Storage

### Storage Options

| Storage | Type | Access Modes | Persistence | Use Case |
|---|---|---|---|---|
| **Azure Disk** | Block | ReadWriteOnce (1 pod) | ✅ | Databases, stateful apps |
| **Azure Files** | File (SMB/NFS) | ReadWriteMany (multiple pods) | ✅ | Shared storage, config files |
| **emptyDir** | Temp | N/A | ❌ (pod lifetime) | Temporary scratch space |

### Kubernetes Storage Concepts

| Concept | Description |
|---|---|
| **PersistentVolume (PV)** | Cluster-level storage resource |
| **PersistentVolumeClaim (PVC)** | Request for storage by a pod |
| **StorageClass** | Defines the provisioner and parameters for dynamic PVs |

### Built-in Storage Classes

| StorageClass | Type | Description |
|---|---|---|
| **default** / **managed** | Azure Disk (Standard SSD LRS) | Default if none specified |
| **managed-premium** | Azure Disk (Premium SSD LRS) | High-performance |
| **azurefile** | Azure Files (Standard SMB) | Shared file storage |
| **azurefile-premium** | Azure Files (Premium SMB) | High-perf shared storage |

> ⚠️ **EXAM TIP:** **Azure Disk** = ReadWriteOnce (single pod). **Azure Files** = ReadWriteMany (multiple pods can access simultaneously). If exam says "shared storage across pods" → Azure Files.

> ⚠️ **EXAM TIP:** When a pod with an Azure Disk PVC moves to another **node**, the disk is automatically **detached and reattached**. This causes brief downtime.

---

## 8. Kubernetes Deployments & Updates

### Deployment Strategies

| Strategy | Description |
|---|---|
| **RollingUpdate** (default) | Gradually replaces old pods with new | 
| **Recreate** | Kills all old pods, then creates new |

### Key kubectl Commands

| Action | Command |
|---|---|
| Get cluster credentials | `az aks get-credentials -g <rg> -n <cluster>` |
| List pods | `kubectl get pods` |
| List services | `kubectl get services` |
| List deployments | `kubectl get deployments` |
| List nodes | `kubectl get nodes` |
| Apply YAML | `kubectl apply -f <file.yaml>` |
| Scale deployment | `kubectl scale deployment <name> --replicas=5` |
| Expose deployment | `kubectl expose deployment <name> --type=LoadBalancer --port=80 --target-port=8080` |
| View pod logs | `kubectl logs <pod-name>` |
| Exec into pod | `kubectl exec -it <pod-name> -- /bin/bash` |
| Delete resource | `kubectl delete -f <file.yaml>` |
| Set image (update) | `kubectl set image deployment/<name> <container>=<image>:<tag>` |
| Rollback | `kubectl rollout undo deployment/<name>` |
| Autoscale | `kubectl autoscale deployment <name> --min=2 --max=10 --cpu-percent=50` |
| Describe resource | `kubectl describe pod <pod-name>` |
| Get namespaces | `kubectl get namespaces` |

> ⚠️ **EXAM TIP:** `az aks get-credentials` downloads kubeconfig to `~/.kube/config` and merges it. This is required BEFORE running any `kubectl` commands against the cluster.

---

## 9. AKS Identity & Access

### Cluster Identity

| Identity Type | Description |
|---|---|
| **System-assigned Managed Identity** | Default — Azure creates identity for the cluster |
| **User-assigned Managed Identity** | Pre-created identity — used for specific permissions |
| **Service Principal** | Legacy — Azure AD app registration (not recommended for new clusters) |

### Azure AD (Entra ID) Integration
- **Azure AD RBAC for Kubernetes** — use Azure AD groups for Kubernetes access
- Map Azure AD users/groups to Kubernetes **ClusterRoles** and **Roles**
- Methods:
  - **Azure RBAC for Kubernetes Authorization** — use Azure role assignments directly
  - **Kubernetes RBAC** — use native Kubernetes RoleBindings with Azure AD
- **Local accounts** can be disabled for security (force Azure AD login)

### Kubernetes RBAC Roles

| Azure Built-in Role | Kubernetes Access |
|---|---|
| **Azure Kubernetes Service Cluster Admin Role** | Full cluster admin (`cluster-admin` ClusterRole) |
| **Azure Kubernetes Service Cluster User Role** | Read cluster credentials (basic access to run kubectl) |
| **Azure Kubernetes Service RBAC Admin** | Full admin within namespaces (not cluster-level) |
| **Azure Kubernetes Service RBAC Cluster Admin** | Full cluster-wide admin via Azure RBAC |
| **Azure Kubernetes Service RBAC Reader** | Read-only access to most resources |
| **Azure Kubernetes Service RBAC Writer** | Read/write to most resources (not roles/bindings) |

> ⚠️ **EXAM TIP:** **AKS Cluster Admin Role** = downloads admin kubeconfig (`cluster-admin`). **AKS Cluster User Role** = downloads user kubeconfig (limited). If Azure AD integrated, user must also authenticate via Azure AD.

> ⚠️ **EXAM TIP:** **Disabling local accounts** forces all access through **Azure AD** — improves security. Without local accounts, `az aks get-credentials --admin` is blocked.

#### Portal Path — Azure AD Integration
```
AKS Cluster → Settings → Cluster configuration →
Authentication and Authorization:
Azure AD authentication with Kubernetes RBAC /
Azure AD authentication with Azure RBAC → Save
```

---

## 10. Security Features

| Feature | Description |
|---|---|
| **Azure AD Integration** | Authentication via Azure AD (Entra ID) |
| **Kubernetes RBAC** | Authorization within the cluster (Roles, ClusterRoles) |
| **Azure RBAC for AKS** | Azure-native role assignments for Kubernetes access |
| **Network Policies** | Control pod-to-pod traffic (Calico / Azure NPM) |
| **Azure Policy for AKS** | Enforce governance (e.g., no privileged containers) |
| **Microsoft Defender for Containers** | Threat detection, vulnerability scanning |
| **Managed Identity** | No credentials for cluster operations |
| **Secrets Store CSI Driver** | Mount Key Vault secrets into pods |
| **Private Cluster** | API server only accessible via private endpoint |
| **Disable local accounts** | Force Azure AD authentication |

### Private Cluster
- API server gets a **private endpoint** — NO public IP
- Access only from within the VNet (or peered/VPN-connected networks)
- Use **Azure Private DNS Zone** for API server DNS resolution

#### Portal Path — Enable Private Cluster
```
AKS Cluster → Create → Networking →
Network access: Private → Save
```

> ⚠️ **EXAM TIP:** **Private cluster** = API server has no public IP. kubectl only works from within the VNet or connected networks. Use Azure Bastion or VPN to manage.

### Network Policies
- Control which pods can communicate with each other
- Analogous to NSGs but for **pod-to-pod** traffic
- **Azure NPM**: works with Azure CNI only
- **Calico**: works with both kubenet and Azure CNI

> ⚠️ **EXAM TIP:** Network Policies must be enabled at **cluster creation time** — cannot be added later. Choose Azure or Calico during creation.

---

## 11. AKS Cluster Upgrades

### Upgrade Types

| Type | What Updates |
|---|---|
| **Kubernetes version upgrade** | Control plane + node pool Kubernetes version |
| **Node image upgrade** | OS image on nodes (security patches) |

### Upgrade Process
- Control plane upgrades **first**, then node pools
- Can only upgrade to **next minor version** (e.g., 1.27 → 1.28, not 1.27 → 1.29)
- **Surge upgrade**: extra nodes added during upgrade to minimize disruption
- **Max surge** setting: how many extra nodes to add during rolling upgrade

### Auto-Upgrade Channels

| Channel | Description |
|---|---|
| **none** | No automatic upgrades (default) |
| **patch** | Auto-apply latest patch version |
| **stable** | Auto-upgrade to latest stable minor version |
| **rapid** | Auto-upgrade to latest supported version |
| **node-image** | Auto-upgrade node images only |

#### Portal Path — Upgrade
```
AKS Cluster → Settings → Cluster configuration →
Kubernetes version → Upgrade version → Select version → Save
```

#### CLI — Upgrade
```bash
# Check available versions
az aks get-upgrades -g <rg> -n <cluster> -o table

# Upgrade cluster
az aks upgrade -g <rg> -n <cluster> --kubernetes-version <version>

# Upgrade node image only
az aks nodepool upgrade -g <rg> --cluster-name <cluster> -n <nodepool> --node-image-only
```

> ⚠️ **EXAM TIP:** You can only upgrade **one minor version** at a time (1.27 → 1.28 → 1.29). Skipping versions is NOT allowed. Control plane must be upgraded **before** node pools.

> ⚠️ **EXAM TIP:** Default auto-upgrade channel = **none**. Clusters do NOT auto-upgrade by default.

---

## 12. Monitoring & Diagnostics

### Container Insights (Azure Monitor)
- Collects **performance metrics** and **container logs** from AKS
- Requires **Azure Monitor Agent** or **Log Analytics agent (OMS)**
- Provides: node/pod/container CPU, memory, network, disk metrics
- **Live logs** — real-time container stdout/stderr in portal

#### Portal Path — Enable Monitoring
```
AKS Cluster → Monitoring → Insights →
Enable Container Insights → Select Log Analytics workspace → Configure
```

### Key Metrics

| Metric | Description |
|---|---|
| Node CPU/Memory % | Resource utilization per node |
| Pod count | Running/pending/failed pods |
| Container restarts | Stability indicator |
| Disk usage | Node disk utilization |
| Network in/out | Traffic per node/pod |

### Diagnostic Tools

| Tool | Description |
|---|---|
| **Container Insights** | Full monitoring + logs |
| **kubectl logs** | Container stdout/stderr |
| **kubectl describe** | Resource details and events |
| **Diagnose and solve problems** | Portal-based troubleshooter |
| **AKS Diagnostics** | Cluster health checks |
| **Resource Health** | Cluster availability status |

#### Portal Path — Metrics
```
AKS Cluster → Monitoring → Metrics →
Metric namespace: Insights → Metric: select → Apply
```

---

## 13. Azure Container Registry (ACR) Integration

- AKS can pull images from ACR using **managed identity** (recommended)
- **Attach ACR** to AKS = automatic AcrPull role assignment

#### CLI — Attach ACR
```bash
# Attach ACR to AKS cluster (assigns AcrPull)
az aks update -g <rg> -n <cluster> --attach-acr <acr-name>

# Detach
az aks update -g <rg> -n <cluster> --detach-acr <acr-name>
```

> ⚠️ **EXAM TIP:** `az aks update --attach-acr` assigns the **AcrPull** role to the AKS managed identity on the ACR. This is the recommended way to integrate ACR with AKS.

---

## 14. Pricing Key Points

| Component | Cost |
|---|---|
| **Control plane** | **Free** (standard tier for SLA) |
| **Worker nodes** | Per-VM pricing (based on VM size) |
| **AKS Free tier** | No SLA, control plane free |
| **AKS Standard tier** | **99.95%** SLA (or 99.99% with AZ), monthly fee for control plane |
| **Node VMs** | Per-second compute |
| **Managed disks** | Per disk per month |
| **Load Balancer** | Standard LB per-rule + data processed |
| **Public IP** | Per-hour |
| **Egress data** | Outbound transfer charged |
| **Container Insights** | Log Analytics ingestion charges |

### AKS Pricing Tiers

| Tier | SLA | Control Plane Cost | Use Case |
|---|---|---|---|
| **Free** | None (best effort) | Free | Dev/test |
| **Standard** | 99.95% (99.99% with AZ) | ~$0.10/cluster/hour | Production |
| **Premium** | 99.95% / 99.99% | Higher | Mission-critical + LTS |

> ⚠️ **EXAM TIP:** AKS **Free tier** = no SLA. **Standard tier** = financial SLA (99.95%). If the exam asks about production AKS with SLA → Standard or Premium tier.

---

## 15. Limitations & Constraints

| Constraint | Limit |
|---|---|
| Max nodes per cluster | **5,000** |
| Max pods per node (kubenet) | **110** (default) |
| Max pods per node (Azure CNI) | **250** |
| Max node pools per cluster | **100** |
| Default node count | **3** (system pool) |
| System node pool minimum | **1 node** (cannot be 0) |
| Kubernetes version support | Latest + 2 previous minor versions |
| Namespaces | Unlimited |
| Services per cluster | No hard limit |
| Network model change | ❌ Cannot change after creation |
| Node pool VM resize | ❌ Must create new pool |
| Network Policy addition | ❌ Must be set at creation |
| Windows system node pool | ❌ Not allowed (Linux only) |

---

## 16. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| Create cluster | `az aks create -g <rg> -n <cluster> --node-count 3 --generate-ssh-keys --network-plugin azure` |
| Get credentials | `az aks get-credentials -g <rg> -n <cluster>` |
| Get admin credentials | `az aks get-credentials -g <rg> -n <cluster> --admin` |
| List clusters | `az aks list -o table` |
| Show cluster | `az aks show -g <rg> -n <cluster>` |
| Scale node pool | `az aks scale -g <rg> -n <cluster> --node-count 5 --nodepool-name <pool>` |
| Add node pool | `az aks nodepool add -g <rg> --cluster-name <cluster> -n <pool> --node-count 3 --node-vm-size Standard_D4s_v5` |
| Add Windows node pool | `az aks nodepool add -g <rg> --cluster-name <cluster> -n winpool --os-type Windows --node-count 2` |
| Delete node pool | `az aks nodepool delete -g <rg> --cluster-name <cluster> -n <pool>` |
| Enable autoscaler | `az aks nodepool update -g <rg> --cluster-name <cluster> -n <pool> --enable-cluster-autoscaler --min-count 1 --max-count 10` |
| Upgrade cluster | `az aks upgrade -g <rg> -n <cluster> --kubernetes-version <version>` |
| Get upgrade versions | `az aks get-upgrades -g <rg> -n <cluster> -o table` |
| Attach ACR | `az aks update -g <rg> -n <cluster> --attach-acr <acr>` |
| Start cluster | `az aks start -g <rg> -n <cluster>` |
| Stop cluster | `az aks stop -g <rg> -n <cluster>` |
| Delete cluster | `az aks delete -g <rg> -n <cluster>` |
| Enable monitoring | `az aks enable-addons -g <rg> -n <cluster> -a monitoring` |
| Enable virtual nodes | `az aks enable-addons -g <rg> -n <cluster> -a virtual-node --subnet-name <subnet>` |

### PowerShell

| Action | Command |
|---|---|
| Create cluster | `New-AzAksCluster -ResourceGroupName <rg> -Name <cluster> -NodeCount 3` |
| Get credentials | `Import-AzAksCredential -ResourceGroupName <rg> -Name <cluster>` |
| Scale | `Set-AzAksCluster -ResourceGroupName <rg> -Name <cluster> -NodeCount 5` |
| Remove cluster | `Remove-AzAksCluster -ResourceGroupName <rg> -Name <cluster>` |

> ⚠️ **EXAM TIP:** `az aks stop` = **stop the entire cluster** (no billing for node VMs). `az aks start` = restart. Stopped clusters keep configuration but deallocate all nodes.

---

## 17. Quick-Fire Exam Points ⚡

1. AKS = **managed Kubernetes** — Azure manages the control plane for free
2. You pay for **worker node VMs**, disks, networking — NOT the control plane (Free tier)
3. **Standard tier** required for financial SLA (**99.95%**, 99.99% with AZ)
4. **System node pool** = required (Linux only), runs system pods, min **1 node**
5. **User node pool** = optional, runs app workloads, supports Linux and Windows, can scale to **0**
6. **Cannot change VM size** of a node pool — must create a new pool
7. Max **5,000 nodes** per cluster, max **100 node pools**
8. Default max pods per node: **110** (kubenet), **250** (Azure CNI)
9. **Kubenet** = default network plugin. **Azure CNI** = pods get VNet IPs directly
10. **Cannot change** network model after cluster creation (kubenet ↔ CNI)
11. **Windows node pools** require **Azure CNI** — not supported with kubenet
12. **Virtual Nodes** (burst to ACI) require **Azure CNI** and are **Linux only**
13. **Network Policies** (Calico/Azure NPM) must be set at **cluster creation** — cannot add later
14. **Cluster Autoscaler** = scales **nodes**. **HPA** = scales **pods**. They work together
15. Kubernetes version upgrade: only **one minor version at a time** (1.27 → 1.28)
16. Control plane upgrades **before** node pools
17. Default auto-upgrade channel = **none** (no auto-upgrade)
18. `az aks get-credentials` = download kubeconfig (required before kubectl)
19. `az aks get-credentials --admin` = admin access (bypasses Azure AD)
20. **AKS Cluster Admin Role** = full admin. **Cluster User Role** = basic user
21. **Private cluster** = API server private endpoint, no public access
22. **Disabling local accounts** = forces Azure AD authentication
23. **Azure Disk** = ReadWriteOnce (single pod). **Azure Files** = ReadWriteMany (shared)
24. Default StorageClass = `managed` (Standard SSD LRS Azure Disk)
25. `az aks update --attach-acr` = assigns **AcrPull** role to AKS managed identity
26. **LoadBalancer** service type creates Azure Standard LB
27. Ingress = Layer 7 routing (NGINX / AGIC with Application Gateway)
28. **Container Insights** = Azure Monitor for AKS metrics + logs
29. `az aks stop` = deallocate entire cluster (saves cost). `az aks start` = resume
30. **Secrets Store CSI Driver** = mount Key Vault secrets as pod volumes
31. **Azure Policy for AKS** = enforce governance (e.g., no privileged pods, required labels)
32. `kubectl apply -f` = declarative deployment. `kubectl create` = imperative
33. `kubectl rollout undo` = rollback a deployment
34. **ClusterIP** = internal only (default service type). **LoadBalancer** = external
35. Kubernetes supports latest + **2 previous minor versions**

---

## 18. Step-by-Step Configuration Mind Maps 🗺️

---

### 18.1 Create AKS Cluster

> **Portal:** `Home → Kubernetes services → + Create → Kubernetes cluster`

```
Create AKS Cluster
│
├── Basics
│   ├── Subscription, Resource Group
│   ├── Cluster preset: Dev/Test / Standard / Production / Enterprise
│   ├── Cluster name
│   ├── Region
│   ├── Availability Zones: Zones 1, 2, 3 (recommended for production)
│   ├── AKS pricing tier:
│   │   ├── Free: No SLA (dev/test)
│   │   ├── Standard: 99.95% SLA (production)
│   │   └── Premium: 99.95% + LTS (mission-critical)
│   │   ⚠️ Free tier = no financial SLA
│   ├── Kubernetes version: Select (latest recommended)
│   │   ⚠️ Can only upgrade one minor version at a time
│   ├── Automatic upgrade: none / patch / stable / rapid / node-image
│   │   ⚠️ Default = none (no auto-upgrade)
│   └── Authentication and Authorization:
│       ├── Local accounts with Kubernetes RBAC
│       ├── Azure AD with Kubernetes RBAC
│       └── Azure AD with Azure RBAC (recommended)
│           ⚠️ Disable local accounts for security
│
├── Node Pools
│   ├── System node pool (default: agentpool):
│   │   ├── Node size: Standard_D2s_v5 (select)
│   │   ├── Scale method: Manual / Autoscale
│   │   │   ├── Manual: Node count (e.g., 3)
│   │   │   └── Autoscale: Min 1, Max 5
│   │   │       ⚠️ System pool min = 1 (cannot be 0)
│   │   ├── Max pods per node: 30–250 (default 110 kubenet / 30 Azure CNI)
│   │   └── OS: Linux (system pool = Linux only)
│   │
│   └── + Add user node pool (optional):
│       ├── Name, VM size, node count
│       ├── OS: Linux / Windows
│       │   ⚠️ Windows requires Azure CNI
│       └── Scale to 0: ✅ allowed for user pools
│
├── Networking
│   ├── Network configuration:
│   │   ├── Kubenet (default): Pods use overlay network
│   │   │   ⚠️ No Windows nodes, no Virtual Nodes
│   │   └── Azure CNI: Pods get VNet IPs
│   │       ⚠️ Requires larger subnet, more IP planning
│   │       ⚠️ CANNOT change after creation
│   ├── VNet: Select or create
│   ├── Subnet: Select (needs enough IPs for nodes + pods with CNI)
│   ├── DNS name prefix
│   ├── Network policy:
│   │   ├── None
│   │   ├── Azure (Azure NPM — Azure CNI only)
│   │   └── Calico (kubenet or Azure CNI)
│   │   ⚠️ CANNOT add network policy after creation
│   ├── Load Balancer: Standard (default)
│   └── Network access: Public / Private
│       ⚠️ Private = no public API server endpoint
│
├── Integrations
│   ├── Container registry: Select ACR (attaches AcrPull role)
│   ├── Azure Monitor: Enable Container Insights ✅
│   │   └── Log Analytics workspace: Select / Create
│   └── Azure Policy: Enable ✅
│
├── Advanced
│   ├── Infrastructure: Enable Secret Store CSI Driver
│   ├── Kubernetes API server: Public or Private
│   └── Managed identity: System / User-assigned
│
├── Tags → Review + Create
│
└── RBAC: Contributor or Owner on Resource Group
    ⚠️ Cluster creation takes 5–10 minutes
```

---

### 18.2 Add Node Pool

> **Portal:** `AKS Cluster → Settings → Node pools → + Add node pool`

```
Add Node Pool
│
├── Prerequisites
│   ├── AKS cluster exists
│   ├── For Windows pool: cluster must use Azure CNI
│   └── RBAC: Contributor on AKS cluster
│
├── Configuration
│   ├── Node pool name (lowercase, alphanumeric, max 12 chars for Windows)
│   ├── Mode:
│   │   ├── System: Runs system pods (Linux only)
│   │   └── User: Runs application pods (Linux or Windows)
│   ├── OS type: Linux / Windows
│   │   ⚠️ Windows = Azure CNI required, max 12-char pool name
│   ├── Availability zones: Select
│   ├── Node size: Select VM SKU
│   │   ⚠️ Cannot change after creation — must create new pool
│   ├── Scale method:
│   │   ├── Manual: Node count
│   │   └── Autoscale: Min / Max
│   │       ⚠️ User pool can scale to 0, System pool cannot
│   ├── Max pods per node: Set (default varies by network plugin)
│   ├── Node labels: Key-value pairs (for pod scheduling)
│   └── Node taints: Taint to limit scheduling (e.g., gpu=true:NoSchedule)
│
└── Add
    ⚠️ Adding a pool takes a few minutes
    ⚠️ Old pool can be drained and deleted if migrating
```

---

### 18.3 Configure Cluster Autoscaler

> **Portal:** `AKS Cluster → Node pools → Select pool → Scale method`

```
Configure Cluster Autoscaler
│
├── Portal: AKS Cluster → Settings → Node pools → Select pool
│
├── Scale method: Autoscale
│   ├── Minimum node count: e.g., 1
│   │   ⚠️ System pool: minimum = 1
│   │   ⚠️ User pool: minimum = 0 (scale to zero)
│   ├── Maximum node count: e.g., 10
│   │   ⚠️ Max 5,000 total nodes per cluster
│   └── Apply
│
├── How It Works:
│   ├── Pods pending (no resources) → autoscaler adds nodes
│   ├── Nodes underutilized for 10+ minutes → autoscaler removes nodes
│   └── Works with HPA: HPA scales pods → CA scales nodes
│
├── CLI:
│   az aks nodepool update -g <rg> --cluster-name <cluster> \
│     -n <pool> --enable-cluster-autoscaler \
│     --min-count 1 --max-count 10
│
├── Disable:
│   az aks nodepool update -g <rg> --cluster-name <cluster> \
│     -n <pool> --disable-cluster-autoscaler
│
└── ⚠️ Notes
    ├── Cluster autoscaler = NODE scaling (not pod scaling)
    ├── HPA = POD scaling (complements autoscaler)
    └── Both can work together for full auto-scaling
```

---

### 18.4 Deploy Application with kubectl

> **CLI:** `kubectl apply -f deployment.yaml`

```
Deploy Application to AKS
│
├── Prerequisites
│   ├── AKS cluster running
│   ├── kubectl installed locally
│   └── Credentials configured:
│       az aks get-credentials -g <rg> -n <cluster>
│       ⚠️ Required before any kubectl command
│
├── Step 1: Create Deployment YAML
│   │   deployment.yaml:
│   ├── apiVersion: apps/v1
│   ├── kind: Deployment
│   ├── metadata: name, labels
│   ├── spec:
│   │   ├── replicas: 3
│   │   ├── selector: matchLabels
│   │   ├── template:
│   │   │   ├── containers:
│   │   │   │   ├── name, image, ports
│   │   │   │   └── resources: requests / limits (CPU, memory)
│   │   │   └── nodeSelector (optional): target specific node pool
│   │   └── strategy: RollingUpdate (default) / Recreate
│
├── Step 2: Create Service YAML
│   │   service.yaml:
│   ├── kind: Service
│   ├── type: LoadBalancer
│   │   ⚠️ Creates Azure Standard Load Balancer with public IP
│   │   For internal: add annotation azure-load-balancer-internal: "true"
│   └── ports: port (external) → targetPort (container)
│
├── Step 3: Apply
│   ├── kubectl apply -f deployment.yaml
│   ├── kubectl apply -f service.yaml
│   └── kubectl get services → note EXTERNAL-IP
│       ⚠️ May take 1–2 minutes for IP assignment
│
├── Step 4: Verify
│   ├── kubectl get pods → all Running ✅
│   ├── kubectl get services → EXTERNAL-IP assigned
│   └── curl http://<EXTERNAL-IP> → app responds
│
└── Step 5: Scale / Update
    ├── Scale: kubectl scale deployment <name> --replicas=5
    ├── Update: kubectl set image deployment/<name> <container>=<image>:<new-tag>
    ├── Rollback: kubectl rollout undo deployment/<name>
    └── HPA: kubectl autoscale deployment <name> --min=2 --max=10 --cpu-percent=50
```

---

### 18.5 Upgrade AKS Cluster

> **Portal:** `AKS Cluster → Settings → Cluster configuration → Kubernetes version`

```
Upgrade AKS Cluster
│
├── Step 1: Check Available Upgrades
│   │   Portal: AKS Cluster → Cluster configuration → Kubernetes version
│   └── CLI: az aks get-upgrades -g <rg> -n <cluster> -o table
│       ⚠️ Only next minor version available (e.g., 1.27 → 1.28)
│       ⚠️ Cannot skip versions (no 1.27 → 1.29)
│
├── Step 2: Upgrade Control Plane
│   │   Portal: Select new version → Save
│   └── CLI: az aks upgrade -g <rg> -n <cluster> --kubernetes-version 1.28.0
│       ⚠️ Control plane upgrades FIRST
│       ⚠️ Brief API server unavailability during upgrade
│
├── Step 3: Upgrade Node Pools
│   │   Upgrade happens automatically with cluster upgrade
│   │   OR upgrade individual pools:
│   └── az aks nodepool upgrade -g <rg> --cluster-name <cluster> \
│         -n <pool> --kubernetes-version 1.28.0
│       ⚠️ Nodes upgraded one-by-one (rolling)
│       ⚠️ Pods cordoned and drained before node upgrade
│
├── Node Image Upgrade (OS patches)
│   └── az aks nodepool upgrade -g <rg> --cluster-name <cluster> \
│         -n <pool> --node-image-only
│       ⚠️ Just patches node OS, does not change K8s version
│
├── Configure Auto-Upgrade (Optional)
│   │   Portal: AKS Cluster → Cluster configuration → Automatic upgrade
│   ├── none: No auto-upgrade (default)
│   ├── patch: Auto-apply patches
│   ├── stable: Auto-upgrade to latest stable
│   ├── rapid: Auto-upgrade to latest
│   └── node-image: Auto-upgrade node images only
│
└── ⚠️ Notes
    ├── Always test upgrades in dev/staging first
    ├── Ensure workloads have Pod Disruption Budgets (PDB)
    ├── Latest + 2 previous minor versions supported
    └── Unsupported versions = no support from Microsoft
```

---

### 18.6 Integrate ACR with AKS

> **CLI:** `az aks update --attach-acr`

```
Integrate ACR with AKS
│
├── Prerequisites
│   ├── AKS cluster exists
│   ├── ACR exists (Basic/Standard/Premium)
│   └── RBAC: Owner/Contributor on both AKS and ACR
│
├── Method 1: Attach During Creation
│   └── az aks create -g <rg> -n <cluster> \
│         --attach-acr <acr-name> \
│         --node-count 3 --generate-ssh-keys
│
├── Method 2: Attach to Existing Cluster
│   └── az aks update -g <rg> -n <cluster> --attach-acr <acr-name>
│       ⚠️ Assigns AcrPull role to AKS managed identity on ACR
│       ⚠️ This is the recommended approach (no credentials)
│
├── Method 3: Portal
│   └── AKS Cluster → Settings → Integrations →
│       Container registry → Select ACR → Apply
│
├── Verify
│   ├── az aks check-acr -g <rg> -n <cluster> --acr <acr>.azurecr.io
│   └── Deploy pod with ACR image:
│       kubectl apply -f pod-with-acr-image.yaml
│
├── Detach
│   └── az aks update -g <rg> -n <cluster> --detach-acr <acr-name>
│
└── ⚠️ Notes
    ├── AcrPull = minimum role to pull images (read-only)
    ├── No secrets or passwords needed with managed identity
    └── Works across subscriptions (need Owner on both)
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
