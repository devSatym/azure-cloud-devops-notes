<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---



Current coverage is aligned with the major skill areas Microsoft uses for Azure Administrator, Azure Solutions Architect, and Azure DevOps Engineer roles, plus the Azure Well-Architected Framework. ([Microsoft Learn][1])

## Topic Priority

### 🔴 M IMP — Must Know

1. Azure Fundamentals, Architecture & Resource Hierarchy
2. Microsoft Entra ID, Authentication, Authorization & Azure RBAC
3. Azure Virtual Networking
4. Azure Virtual Machines, VMSS & Compute
5. Azure Storage
6. Azure Load Balancing & Application Delivery
7. AKS — Azure Kubernetes Service
8. Azure DevOps & GitHub Actions CI/CD
9. Terraform on Azure
10. Azure Monitor, Log Analytics, Application Insights & KQL
11. Azure Security, Key Vault & Defender for Cloud
12. High Availability, Backup, Disaster Recovery & Business Continuity
13. Azure Troubleshooting & Production Scenarios

### 🟠 IMP — Important

14. Azure Container Registry & Container Security
15. App Service, Azure Functions & Azure Container Apps
16. Private Endpoints, Private Link & Private Networking
17. Hybrid Connectivity — VPN, ExpressRoute & Azure Bastion
18. Azure Governance — Policy, Management Groups, Locks & Tags
19. Managed Identities, Service Principals & Workload Identity
20. Cost Management, FinOps & Resource Optimization
21. Azure DNS, Traffic Manager, Front Door & Application Gateway
22. Azure Databases & Managed Data Services
23. Messaging — Service Bus, Event Grid & Event Hubs
24. Git, Branching & Source-Control Strategy
25. DevSecOps & Software Supply-Chain Security

### 🟡 GOOD — Good to Know

26. Azure CLI, PowerShell & Cloud Shell
27. ARM Templates & Bicep
28. Azure Automation, Update Manager & Operations
29. Azure Landing Zones, Cloud Adoption Framework & Azure Arc
30. Azure Well-Architected Framework, SRE & Architecture Design
31. Azure Migration & Modernization

---

# 1. 🔴 M IMP — Azure Fundamentals, Architecture & Resource Hierarchy

### Easy

**T01-E01.** What is Microsoft Azure?

**T01-E02.** What is cloud computing?

**T01-E03.** What is the difference between public cloud, private cloud, and hybrid cloud?

**T01-E04.** What is IaaS, PaaS, and SaaS?

**T01-E05.** What is an Azure region?

**T01-E06.** What is an Azure Availability Zone?

**T01-E07.** What is an Azure subscription?

**T01-E08.** What is an Azure resource group?

### Medium

**T01-M01.** What is the Azure resource hierarchy?

**T01-M02.** What is the relationship between a tenant, management group, subscription, resource group, and resource?

**T01-M03.** Can an Azure resource belong to multiple resource groups?

**T01-M04.** Can resources inside one resource group exist in different Azure regions?

**T01-M05.** Can you move an Azure resource from one resource group to another?

**T01-M06.** Can you move Azure resources between subscriptions?

**T01-M07.** What happens when you delete an Azure resource group?

**T01-M08.** What is Azure Resource Manager?

**T01-M09.** What is a control plane versus a data plane in Azure?

**T01-M10.** What are Azure resource providers?

### Hard

**T01-H01.** How would you design the Azure subscription hierarchy for a large enterprise?

**T01-H02.** When would you create separate subscriptions rather than separate resource groups?

**T01-H03.** How would you separate development, staging, and production Azure environments?

**T01-H04.** How would you design Azure resources for multiple business units while maintaining centralized governance?

**T01-H05.** What happens internally when you deploy a resource through ARM?

**T01-H06.** How do Azure resource-provider registration and subscription permissions affect resource deployment?

---

# 2. 🔴 M IMP — Microsoft Entra ID, Authentication, Authorization & RBAC

### Easy

**T02-E01.** What is Microsoft Entra ID?

**T02-E02.** What was Azure Active Directory renamed to?

**T02-E03.** What is an Entra tenant?

**T02-E04.** What is the difference between authentication and authorization?

**T02-E05.** What is Azure RBAC?

**T02-E06.** What is an Azure role assignment?

**T02-E07.** What are Owner, Contributor, and Reader roles?

**T02-E08.** What is the difference between an Azure subscription and an Entra tenant?

### Medium

**T02-M01.** What are the components of an Azure RBAC role assignment?

**T02-M02.** What is an Azure RBAC scope?

**T02-M03.** At which scopes can RBAC roles be assigned?

**T02-M04.** How is RBAC inheritance applied?

**T02-M05.** What is the difference between an Azure RBAC role and a Microsoft Entra directory role?

**T02-M06.** What is the difference between Contributor and Owner?

**T02-M07.** What is a custom Azure RBAC role?

**T02-M08.** What is Conditional Access?

**T02-M09.** What is MFA?

**T02-M10.** What is Privileged Identity Management?

**T02-M11.** What is the difference between eligible and active PIM assignments?

### Hard

**T02-H01.** A user has Reader at subscription level and Contributor at resource-group level; what can the user do?

**T02-H02.** How would you implement least-privilege access for a DevOps team?

**T02-H03.** How would you grant a CI/CD pipeline permission to deploy only into one resource group?

**T02-H04.** How would you troubleshoot a user receiving `AuthorizationFailed` despite having an RBAC role?

**T02-H05.** How do deny assignments differ from Azure RBAC role assignments?

**T02-H06.** How would you design privileged administration using PIM instead of permanent Owner assignments?

**T02-H07.** How would you give developers access to Kubernetes resources without making them Azure subscription Contributors?

---

# 3. 🔴 M IMP — Azure Virtual Networking

### Easy

**T03-E01.** What is an Azure Virtual Network?

**T03-E02.** What is a subnet?

**T03-E03.** What is CIDR notation?

**T03-E04.** What is a private IP address?

**T03-E05.** What is a public IP address?

**T03-E06.** What is an NSG?

**T03-E07.** What is a route table?

**T03-E08.** What is VNet peering?

### Medium

**T03-M01.** How does traffic flow between subnets in the same VNet?

**T03-M02.** What is the difference between inbound and outbound NSG rules?

**T03-M03.** How are NSG rules prioritized?

**T03-M04.** What happens when two NSG rules match the same traffic?

**T03-M05.** Can an NSG be attached to both a subnet and a network interface?

**T03-M06.** What happens when subnet NSG and NIC NSG rules conflict?

**T03-M07.** What is an Application Security Group?

**T03-M08.** What is a User Defined Route?

**T03-M09.** What is the Azure system route table?

**T03-M10.** What is the purpose of `0.0.0.0/0`?

**T03-M11.** What is Azure NAT Gateway?

**T03-M12.** How does Azure assign outbound connectivity?

**T03-M13.** What is IP forwarding?

### Hard

**T03-H01.** How would you troubleshoot two Azure VMs that cannot communicate?

**T03-H02.** How would you troubleshoot asymmetric routing in Azure?

**T03-H03.** What happens if two peered VNets have overlapping CIDR ranges?

**T03-H04.** Is VNet peering transitive?

**T03-H05.** How would you implement hub-and-spoke networking?

**T03-H06.** How do you route spoke-to-spoke traffic through Azure Firewall?

**T03-H07.** What is forced tunneling?

**T03-H08.** How would you inspect the effective routes of an Azure VM?

**T03-H09.** How would you inspect effective NSG rules?

**T03-H10.** How would you troubleshoot intermittent SNAT port exhaustion?

---

# 4. 🔴 M IMP — Azure Virtual Machines, VMSS & Compute

### Easy

**T04-E01.** What is an Azure Virtual Machine?

**T04-E02.** What is an Azure VM size?

**T04-E03.** What is an Azure managed disk?

**T04-E04.** What are OS disks and data disks?

**T04-E05.** What is an Azure Availability Set?

**T04-E06.** What is an Availability Zone?

**T04-E07.** What is a Virtual Machine Scale Set?

### Medium

**T04-M01.** What is the difference between Availability Sets and Availability Zones?

**T04-M02.** What are fault domains and update domains?

**T04-M03.** What is the difference between Standard HDD, Standard SSD, Premium SSD, and Ultra Disk?

**T04-M04.** What is an ephemeral OS disk?

**T04-M05.** What is a VM extension?

**T04-M06.** What is cloud-init?

**T04-M07.** How do you resize an Azure VM?

**T04-M08.** What happens when an Azure VM is stopped versus deallocated?

**T04-M09.** Are you charged for a stopped Azure VM?

**T04-M10.** What is Accelerated Networking?

**T04-M11.** What are Spot VMs?

**T04-M12.** What are Azure Dedicated Hosts?

### Hard

**T04-H01.** A VM is running but cannot be accessed over SSH; how would you troubleshoot it?

**T04-H02.** How would you recover a Linux VM after an incorrect `/etc/fstab` configuration prevents booting?

**T04-H03.** How would you design a highly available web application using VM Scale Sets?

**T04-H04.** How does VMSS autoscaling work?

**T04-H05.** How would you perform rolling upgrades on a VM Scale Set?

**T04-H06.** How do you choose between VMs, VMSS, App Service, Container Apps, and AKS?

**T04-H07.** How would you reduce VM costs without sacrificing production availability?

---

# 5. 🔴 M IMP — Azure Storage

### Easy

**T05-E01.** What is an Azure Storage Account?

**T05-E02.** What is Blob Storage?

**T05-E03.** What is Azure Files?

**T05-E04.** What is Azure Queue Storage?

**T05-E05.** What is Azure Table Storage?

**T05-E06.** What is the difference between Blob Storage and Azure Files?

**T05-E07.** What are hot, cool, cold, and archive storage tiers?

### Medium

**T05-M01.** What is LRS?

**T05-M02.** What is ZRS?

**T05-M03.** What is GRS?

**T05-M04.** What is GZRS?

**T05-M05.** What is RA-GRS?

**T05-M06.** When would you choose ZRS instead of GRS?

**T05-M07.** What is a SAS token?

**T05-M08.** What is the difference between account SAS, service SAS, and user-delegation SAS?

**T05-M09.** What are storage account access keys?

**T05-M10.** How can Microsoft Entra authentication be used with Azure Storage?

**T05-M11.** What are lifecycle-management policies?

**T05-M12.** What is Blob versioning?

**T05-M13.** What is Blob soft delete?

**T05-M14.** What is immutable Blob Storage?

### Hard

**T05-H01.** How would you prevent a storage account from being accessible from the public internet?

**T05-H02.** How would an application access Blob Storage without storing credentials?

**T05-H03.** What happens during a storage-account regional outage with GRS?

**T05-H04.** How would you protect critical blobs from accidental or malicious deletion?

**T05-H05.** How would you securely allow a customer to upload one file directly to Blob Storage?

**T05-H06.** How would you troubleshoot an Azure VM that cannot mount an Azure Files share?

---

# 6. 🔴 M IMP — Azure Load Balancing & Application Delivery

### Easy

**T06-E01.** What is Azure Load Balancer?

**T06-E02.** What is Azure Application Gateway?

**T06-E03.** What is Azure Front Door?

**T06-E04.** What is Azure Traffic Manager?

**T06-E05.** What is the difference between Layer 4 and Layer 7 load balancing?

**T06-E06.** What is a health probe?

### Medium

**T06-M01.** What is the difference between public and internal Azure Load Balancers?

**T06-M02.** What is the difference between Azure Load Balancer and Application Gateway?

**T06-M03.** When should you use Application Gateway instead of Load Balancer?

**T06-M04.** What is Application Gateway WAF?

**T06-M05.** What is SSL/TLS termination?

**T06-M06.** What is path-based routing?

**T06-M07.** What is host-based routing?

**T06-M08.** What are backend pools?

**T06-M09.** What are load-balancing rules?

**T06-M10.** What are inbound NAT rules?

### Hard

**T06-H01.** When would you use Front Door + Application Gateway together?

**T06-H02.** How would you expose a globally distributed application using Azure services?

**T06-H03.** How would you design regional failover for a public web application?

**T06-H04.** An Application Gateway returns HTTP 502 errors; how would you troubleshoot it?

**T06-H05.** How would you implement end-to-end TLS between clients, Application Gateway, and backend services?

**T06-H06.** How would you prevent direct public access to backend services behind Front Door?

---

# 7. 🔴 M IMP — AKS

### Easy

**T07-E01.** What is Azure Kubernetes Service?

**T07-E02.** Which Kubernetes components are managed by Microsoft in AKS?

**T07-E03.** What is an AKS node pool?

**T07-E04.** What is the difference between system and user node pools?

**T07-E05.** What is an AKS cluster identity?

**T07-E06.** What is Azure CNI?

**T07-E07.** What is a Kubernetes Service of type `LoadBalancer` in AKS?

**T07-E08.** What is AKS Workload Identity?

### Medium

**T07-M01.** How does AKS networking work?

**T07-M02.** What is Azure CNI Overlay?

**T07-M03.** How do pods communicate with Azure resources?

**T07-M04.** How does a Kubernetes LoadBalancer Service integrate with Azure?

**T07-M05.** What is the AKS cluster autoscaler?

**T07-M06.** What is Horizontal Pod Autoscaler?

**T07-M07.** What is the difference between HPA and cluster autoscaler?

**T07-M08.** What is a private AKS cluster?

**T07-M09.** How does Entra ID integrate with AKS authentication?

**T07-M10.** How can Azure RBAC be integrated with AKS authorization?

**T07-M11.** How do you pull private images from ACR into AKS?

**T07-M12.** How do you store secrets securely for AKS workloads?

**T07-M13.** How does the Key Vault CSI driver work with AKS?

**T07-M14.** How do AKS upgrades work?

### Hard

**T07-H01.** An AKS pod remains in `Pending`; how would you troubleshoot it?

**T07-H02.** A pod is in `CrashLoopBackOff`; what would you investigate?

**T07-H03.** AKS pods cannot reach the internet; how would you troubleshoot the issue?

**T07-H04.** AKS pods cannot access an Azure database through a private endpoint; what would you check?

**T07-H05.** How would you design a production-grade private AKS cluster?

**T07-H06.** How would you implement zero-secret Azure authentication for Kubernetes workloads?

**T07-H07.** How would you upgrade a production AKS cluster with minimal downtime?

**T07-H08.** How would you protect application availability while draining AKS nodes?

**T07-H09.** How would you separate system workloads and application workloads?

**T07-H10.** How would you troubleshoot an AKS service that works internally but is inaccessible externally?

**T07-H11.** How would you troubleshoot an AKS DNS-resolution failure?

**T07-H12.** How would you handle a sudden traffic spike that requires both pod and node scaling?

---

# 8. 🔴 M IMP — Azure DevOps & GitHub Actions CI/CD

### Easy

**T08-E01.** What is CI?

**T08-E02.** What is Continuous Delivery?

**T08-E03.** What is Continuous Deployment?

**T08-E04.** What is Azure DevOps?

**T08-E05.** What are Azure Pipelines?

**T08-E06.** What is GitHub Actions?

**T08-E07.** What is a pipeline agent?

**T08-E08.** What is a GitHub Actions runner?

### Medium

**T08-M01.** What is the difference between Microsoft-hosted and self-hosted agents?

**T08-M02.** What is an Azure DevOps service connection?

**T08-M03.** What is an Azure DevOps environment?

**T08-M04.** How are secrets stored in Azure Pipelines?

**T08-M05.** What are pipeline variables?

**T08-M06.** What are variable groups?

**T08-M07.** What is a multi-stage YAML pipeline?

**T08-M08.** What are pipeline templates?

**T08-M09.** How do pipeline artifacts differ from container images?

**T08-M10.** How can a GitHub Actions workflow authenticate to Azure?

**T08-M11.** Why is OIDC federation preferred over client secrets?

**T08-M12.** How would you trigger different pipelines for different branches?

**T08-M13.** How can approvals be added before a production deployment?

### Hard

**T08-H01.** Design a CI/CD pipeline for an application deployed to AKS.

**T08-H02.** How would you deploy from GitHub Actions to Azure without storing an Azure client secret?

**T08-H03.** How would you design reusable pipelines for 30 microservices?

**T08-H04.** How would you prevent an unreviewed pull request from reaching production?

**T08-H05.** How would you implement build-once-deploy-many across dev, staging, and production?

**T08-H06.** How would you roll back a failed deployment automatically?

**T08-H07.** How would you troubleshoot a pipeline that works with a personal account but fails using a service connection?

**T08-H08.** How would you secure a self-hosted Azure DevOps agent?

**T08-H09.** How would you avoid giving CI/CD Contributor access to an entire subscription?

---

# 9. 🔴 M IMP — Terraform on Azure

### Easy

**T09-E01.** What is Terraform?

**T09-E02.** What is Infrastructure as Code?

**T09-E03.** What is the AzureRM provider?

**T09-E04.** What does `terraform init` do?

**T09-E05.** What does `terraform plan` do?

**T09-E06.** What does `terraform apply` do?

**T09-E07.** What does `terraform destroy` do?

**T09-E08.** What is Terraform state?

### Medium

**T09-M01.** Why should Terraform state be stored remotely?

**T09-M02.** How can Azure Storage be used as a Terraform backend?

**T09-M03.** How does Terraform state locking work with Azure Storage?

**T09-M04.** What is a Terraform module?

**T09-M05.** What is the difference between variables and locals?

**T09-M06.** What are Terraform outputs?

**T09-M07.** What is a Terraform data source?

**T09-M08.** What is `for_each`?

**T09-M09.** What is `count`?

**T09-M10.** When would you use `for_each` rather than `count`?

**T09-M11.** What is `depends_on`?

**T09-M12.** What is the Terraform lifecycle block?

**T09-M13.** What does `prevent_destroy` do?

**T09-M14.** What does `create_before_destroy` do?

**T09-M15.** What is Terraform drift?

### Hard

**T09-H01.** How do you import an existing Azure resource into Terraform?

**T09-H02.** What would you do if Terraform state were accidentally deleted?

**T09-H03.** What would you do if two pipelines tried to run Terraform simultaneously?

**T09-H04.** How would you organize Terraform for dev, staging, and production?

**T09-H05.** When should Terraform workspaces be used versus separate state files?

**T09-H06.** How would you authenticate Terraform to Azure using workload identity federation rather than a client secret?

**T09-H07.** How would you handle a resource that someone manually modified in the Azure portal?

**T09-H08.** How would you safely rename a Terraform resource without recreating it?

**T09-H09.** How would you safely refactor resources between Terraform modules?

**T09-H10.** How would you design reusable Terraform modules for multiple Azure subscriptions?

**T09-H11.** How would you prevent secrets from leaking into Terraform state?

**T09-H12.** What is the AzAPI provider and when might you use it alongside AzureRM?

---

# 10. 🔴 M IMP — Azure Monitor, Log Analytics, Application Insights & KQL

### Easy

**T10-E01.** What is Azure Monitor?

**T10-E02.** What are Azure metrics?

**T10-E03.** What are Azure logs?

**T10-E04.** What is a Log Analytics workspace?

**T10-E05.** What is Application Insights?

**T10-E06.** What is KQL?

**T10-E07.** What is an Azure Monitor alert?

**T10-E08.** What is an Action Group?

### Medium

**T10-M01.** What is the difference between metrics and logs?

**T10-M02.** What is the Azure Activity Log?

**T10-M03.** What is the difference between Activity Logs and resource logs?

**T10-M04.** What are diagnostic settings?

**T10-M05.** Where can diagnostic logs be sent?

**T10-M06.** What is the Azure Monitor Agent?

**T10-M07.** What is a Data Collection Rule?

**T10-M08.** What is Container Insights?

**T10-M09.** What is a metric alert?

**T10-M10.** What is a log-search alert?

**T10-M11.** What are Application Insights availability tests?

**T10-M12.** How are Application Insights traces, requests, dependencies, and exceptions related?

### Hard

**T10-H01.** How would you monitor an AKS-based application end to end?

**T10-H02.** A VM has high CPU usage every night; how would you investigate it using Azure Monitor?

**T10-H03.** How would you detect repeated pod restarts using KQL?

**T10-H04.** How would you find HTTP 500 errors using Application Insights?

**T10-H05.** How would you correlate a slow API request with its downstream dependency?

**T10-H06.** How would you design alerts that avoid excessive alert noise?

**T10-H07.** How would you route different Azure alerts to different operations teams?

**T10-H08.** What would you investigate if logs suddenly stopped appearing in Log Analytics?

### KQL

**T10-K01.** What does the KQL `where` operator do?

**T10-K02.** What does `project` do?

**T10-K03.** What does `summarize` do?

**T10-K04.** What does `extend` do?

**T10-K05.** What does `order by` do?

**T10-K06.** What is `bin()` used for?

**T10-K07.** How would you count events grouped by resource?

**T10-K08.** How would you calculate the number of failures during the last hour?

---

# 11. 🔴 M IMP — Azure Security, Key Vault & Defender for Cloud

### Easy

**T11-E01.** What is Azure Key Vault?

**T11-E02.** What can be stored inside Key Vault?

**T11-E03.** What is the difference between a secret, key, and certificate?

**T11-E04.** What is Microsoft Defender for Cloud?

**T11-E05.** What is the shared-responsibility model?

**T11-E06.** What is encryption at rest?

**T11-E07.** What is encryption in transit?

### Medium

**T11-M01.** How can an Azure application access Key Vault securely?

**T11-M02.** What is Key Vault RBAC?

**T11-M03.** What is Key Vault soft delete?

**T11-M04.** What is purge protection?

**T11-M05.** Why should secrets not be stored in pipeline YAML?

**T11-M06.** What is a customer-managed encryption key?

**T11-M07.** What is Microsoft Defender for Servers?

**T11-M08.** What is Microsoft Defender for Containers?

**T11-M09.** What is the difference between Defender for Cloud and Microsoft Sentinel?

**T11-M10.** What is Secure Score?

### Hard

**T11-H01.** How would you design secret management for applications running on AKS?

**T11-H02.** How would you rotate a database password without redeploying every application manually?

**T11-H03.** How would you ensure that Key Vault is accessible only over private networking?

**T11-H04.** How would you secure administrative access to production Azure subscriptions?

**T11-H05.** How would you protect an Azure environment against accidentally exposed credentials?

**T11-H06.** How would you investigate a suspected compromise of an Azure service principal?

**T11-H07.** How would you design defense in depth for an internet-facing Azure application?

---

# 12. 🔴 M IMP — High Availability, Backup, DR & Business Continuity

### Easy

**T12-E01.** What is high availability?

**T12-E02.** What is disaster recovery?

**T12-E03.** What is RTO?

**T12-E04.** What is RPO?

**T12-E05.** What is Azure Backup?

**T12-E06.** What is Azure Site Recovery?

**T12-E07.** What is a Recovery Services vault?

### Medium

**T12-M01.** What is the difference between backup and disaster recovery?

**T12-M02.** What is the difference between Availability Zones and multi-region deployment?

**T12-M03.** What is zone redundancy?

**T12-M04.** What is geo-redundancy?

**T12-M05.** How does Azure Site Recovery work for Azure VMs?

**T12-M06.** What is a recovery point?

**T12-M07.** What is backup retention?

**T12-M08.** What is a DR failover?

**T12-M09.** What is failback?

**T12-M10.** What is a DR drill?

### Hard

**T12-H01.** Design a solution with an RTO of 15 minutes and an RPO of 5 minutes.

**T12-H02.** How would you design a web application to survive an entire Azure region outage?

**T12-H03.** How would you test disaster recovery without affecting production?

**T12-H04.** How would DNS affect your RTO during regional failover?

**T12-H05.** What parts of an AKS application require backup?

**T12-H06.** How would you protect critical data from ransomware?

**T12-H07.** What tradeoffs exist between active-active and active-passive architectures?

**T12-H08.** How would you prove that your disaster-recovery plan actually meets its required RTO and RPO?

---

# 13. 🔴 M IMP — Azure Troubleshooting & Production Scenarios

### Easy/Medium

**T13-01.** A VM cannot access the internet; what would you check?

**T13-02.** You cannot SSH into an Azure VM; how would you troubleshoot it?

**T13-03.** An application cannot connect to an Azure SQL database; what would you investigate?

**T13-04.** A user gets `403 Forbidden` while accessing Blob Storage; what would you check?

**T13-05.** Terraform receives `AuthorizationFailed`; what would you investigate?

**T13-06.** A GitHub Actions workflow cannot log in to Azure using OIDC; what would you check?

**T13-07.** A deployment worked yesterday but fails today; how would you approach the problem?

**T13-08.** A VM is running out of disk space; how would you investigate and fix it?

**T13-09.** CPU usage suddenly reaches 100%; how would you investigate it?

**T13-10.** Azure resources are healthy but users report that the application is unavailable; what would you check?

### Hard

**T13-11.** Application latency increased after deployment even though CPU and memory are normal; how would you investigate it?

**T13-12.** An AKS application experiences intermittent 503 errors; how would you troubleshoot it?

**T13-13.** Pods can reach public internet endpoints but not an Azure private endpoint; what would you investigate?

**T13-14.** Users in one geographic region experience high latency while others do not; how would you troubleshoot it?

**T13-15.** An application works from a VM but not from AKS in the same VNet; what would you investigate?

**T13-16.** After adding a UDR, several services become unreachable; how would you identify the problem?

**T13-17.** A private endpoint resolves to a public IP address; what is likely wrong?

**T13-18.** An Azure VM can resolve DNS names but cannot establish HTTPS connections; what would you check?

**T13-19.** A storage account works from a developer laptop but not from production; how would you troubleshoot it?

**T13-20.** A production deployment caused errors for only 20% of users; what deployment or infrastructure issues would you investigate?

**T13-21.** Logs show no errors but users report failed requests; how would you proceed?

**T13-22.** An NSG appears correct but connectivity still fails; what else would you inspect?

**T13-23.** How would you determine whether an outage is caused by your application, Azure networking, DNS, or an Azure platform issue?

**T13-24.** What Azure tools would you use during a production networking incident?

**T13-25.** What would you examine before restarting a production service during an incident?

---

# 14. 🟠 IMP — Azure Container Registry & Container Security

### Easy

**T14-E01.** What is Azure Container Registry?

**T14-E02.** What is a container registry?

**T14-E03.** What is a Docker image tag?

**T14-E04.** What is an image digest?

**T14-E05.** How does AKS pull images from ACR?

### Medium

**T14-M01.** What are ACR SKUs?

**T14-M02.** How can ACR be made private?

**T14-M03.** What is ACR authentication?

**T14-M04.** Why should production deployments use immutable image identifiers?

**T14-M05.** Why is deploying `latest` considered risky?

**T14-M06.** What is ACR geo-replication?

**T14-M07.** What are ACR Tasks?

### Hard

**T14-H01.** How would you secure the flow from source code to ACR to AKS?

**T14-H02.** How would you ensure that only trusted container images can run in AKS?

**T14-H03.** How would you scan images for vulnerabilities before deployment?

**T14-H04.** What is an SBOM?

**T14-H05.** What is container-image signing?

**T14-H06.** What is software provenance?

---

# 15. 🟠 IMP — App Service, Functions & Container Apps

### Easy

**T15-E01.** What is Azure App Service?

**T15-E02.** What is an App Service Plan?

**T15-E03.** What is Azure Functions?

**T15-E04.** What is serverless computing?

**T15-E05.** What is Azure Container Apps?

### Medium

**T15-M01.** What are App Service deployment slots?

**T15-M02.** How does App Service autoscaling work?

**T15-M03.** How can App Service access Key Vault without credentials?

**T15-M04.** What is VNet Integration for App Service?

**T15-M05.** What is a private endpoint for App Service?

**T15-M06.** What are Azure Functions triggers and bindings?

**T15-M07.** What is the difference between Consumption and Premium Azure Functions hosting?

**T15-M08.** How does Container Apps scaling work?

**T15-M09.** When would you choose Container Apps instead of AKS?

### Hard

**T15-H01.** When should you use App Service versus AKS?

**T15-H02.** When should you use Azure Functions instead of Container Apps?

**T15-H03.** How would you implement zero-downtime deployment with App Service slots?

**T15-H04.** How would you privately connect App Service to Azure SQL?

**T15-H05.** How would you troubleshoot an Azure Function experiencing cold-start latency?

---

# 16. 🟠 IMP — Private Endpoints, Private Link & Private Networking

### Easy

**T16-E01.** What is Azure Private Link?

**T16-E02.** What is a private endpoint?

**T16-E03.** What is a service endpoint?

**T16-E04.** What is a private DNS zone?

### Medium

**T16-M01.** What is the difference between a service endpoint and private endpoint?

**T16-M02.** Does a private endpoint receive a private IP address?

**T16-M03.** Why is DNS important for Private Link?

**T16-M04.** What happens to the public endpoint when a private endpoint is created?

**T16-M05.** Can public network access be disabled independently?

**T16-M06.** How do private DNS zones work across peered VNets?

### Hard

**T16-H01.** Why might a VM resolve an Azure PaaS service to its public address despite a private endpoint existing?

**T16-H02.** How would on-premises clients resolve Azure private endpoints?

**T16-H03.** How would you provide private access to Azure PaaS services from multiple spoke networks?

**T16-H04.** How would you design centralized private DNS in hub-and-spoke architecture?

**T16-H05.** How would you troubleshoot a private endpoint showing approved but remaining unreachable?

---

# 17. 🟠 IMP — VPN, ExpressRoute, Bastion & Hybrid Networking

### Easy

**T17-E01.** What is Azure VPN Gateway?

**T17-E02.** What is a site-to-site VPN?

**T17-E03.** What is a point-to-site VPN?

**T17-E04.** What is Azure ExpressRoute?

**T17-E05.** What is Azure Bastion?

### Medium

**T17-M01.** What is the difference between VPN Gateway and ExpressRoute?

**T17-M02.** When would you use site-to-site versus point-to-site VPN?

**T17-M03.** What is BGP?

**T17-M04.** Why is BGP useful with Azure networking?

**T17-M05.** Can VPN Gateway and ExpressRoute coexist?

**T17-M06.** Why is Azure Bastion preferable to exposing SSH/RDP publicly?

### Hard

**T17-H01.** How would you design redundant connectivity between an on-premises data center and Azure?

**T17-H02.** How would you route traffic between on-premises networks and multiple Azure spokes?

**T17-H03.** How would you troubleshoot an Azure VPN tunnel that shows connected but carries no traffic?

**T17-H04.** What would you investigate if ExpressRoute connectivity worked for some prefixes but not others?

**T17-H05.** How would you provide secure administrative access to hundreds of private Azure VMs?

---

# 18. 🟠 IMP — Azure Governance

### Easy

**T18-E01.** What are Azure management groups?

**T18-E02.** What is Azure Policy?

**T18-E03.** What is an Azure resource lock?

**T18-E04.** What are Azure tags?

### Medium

**T18-M01.** What is the difference between Azure Policy and Azure RBAC?

**T18-M02.** What is a policy definition?

**T18-M03.** What is a policy initiative?

**T18-M04.** What is a policy assignment?

**T18-M05.** What is a policy exemption?

**T18-M06.** What are common Azure Policy effects?

**T18-M07.** What does the `Deny` effect do?

**T18-M08.** What does `Audit` do?

**T18-M09.** What is `DeployIfNotExists`?

**T18-M10.** What is a remediation task?

### Hard

**T18-H01.** How would you enforce mandatory tags across an enterprise?

**T18-H02.** How would you prevent creation of public IP addresses in production?

**T18-H03.** How would you restrict deployments to approved Azure regions?

**T18-H04.** How would you prevent accidental deletion of critical production resources?

**T18-H05.** How would you apply common governance controls across hundreds of subscriptions?

**T18-H06.** How would you identify existing resources that violate a newly introduced Azure Policy?

---

# 19. 🟠 IMP — Managed Identities, Service Principals & Workload Identity

### Easy

**T19-E01.** What is a service principal?

**T19-E02.** What is a managed identity?

**T19-E03.** What is a system-assigned managed identity?

**T19-E04.** What is a user-assigned managed identity?

**T19-E05.** What is workload identity federation?

### Medium

**T19-M01.** What is the difference between a managed identity and service principal?

**T19-M02.** When would you use a user-assigned managed identity?

**T19-M03.** What happens to a system-assigned identity when its Azure resource is deleted?

**T19-M04.** Why are managed identities preferred over client secrets?

**T19-M05.** What is an Entra application registration?

**T19-M06.** What is the relationship between an application registration and service principal?

**T19-M07.** How does GitHub Actions OIDC authentication to Azure work?

### Hard

**T19-H01.** How would you eliminate long-lived Azure credentials from CI/CD?

**T19-H02.** How would a pod running in AKS access Key Vault without a Kubernetes Secret containing Azure credentials?

**T19-H03.** How would you grant one workload access to one storage account without allowing access to others?

**T19-H04.** How do federated identity credentials reduce secret-management risks?

**T19-H05.** How would you troubleshoot a managed identity receiving HTTP 403 from Key Vault?

---

# 20. 🟠 IMP — Azure Cost Management & FinOps

### Easy

**T20-E01.** What is Azure Cost Management?

**T20-E02.** What is an Azure budget?

**T20-E03.** What are Azure reservations?

**T20-E04.** What is Azure Savings Plan for Compute?

**T20-E05.** What are Spot VMs?

### Medium

**T20-M01.** How can tags help with Azure cost allocation?

**T20-M02.** What is Azure Advisor?

**T20-M03.** How can Azure Advisor reduce cost?

**T20-M04.** What is rightsizing?

**T20-M05.** What is autoscaling from a cost perspective?

**T20-M06.** What kinds of Azure data-transfer costs should you consider?

**T20-M07.** Why can Log Analytics become expensive?

### Hard

**T20-H01.** An Azure bill suddenly increases by 40%; how would you investigate it?

**T20-H02.** How would you reduce the cost of a nonproduction AKS cluster?

**T20-H03.** How would you reduce Log Analytics costs without losing critical operational data?

**T20-H04.** When would you choose reservations instead of autoscaling or Spot capacity?

**T20-H05.** How would you implement showback or chargeback for multiple teams sharing Azure?

**T20-H06.** How would you balance reliability and cost for a production application?

---

# 21. 🟠 IMP — DNS, Traffic Manager, Front Door & Application Gateway

### Easy

**T21-E01.** What is Azure DNS?

**T21-E02.** What is an A record?

**T21-E03.** What is a CNAME record?

**T21-E04.** What is DNS TTL?

**T21-E05.** What is Traffic Manager?

### Medium

**T21-M01.** What Traffic Manager routing methods are available?

**T21-M02.** What is priority routing?

**T21-M03.** What is weighted routing?

**T21-M04.** What is performance routing?

**T21-M05.** What is geographic routing?

**T21-M06.** How does Traffic Manager differ from Front Door?

**T21-M07.** Why can DNS-based failover be slower than proxy-based failover?

### Hard

**T21-H01.** How would you design global routing for users in India, Europe, and North America?

**T21-H02.** How would you migrate traffic between two Azure regions without downtime?

**T21-H03.** How could DNS caching affect a disaster-recovery failover?

**T21-H04.** How would you troubleshoot a domain that resolves correctly externally but incorrectly inside an Azure VNet?

**T21-H05.** When would Front Door be preferable to Traffic Manager?

---

# 22. 🟠 IMP — Azure Databases & Managed Data Services

### Easy

**T22-E01.** What is Azure SQL Database?

**T22-E02.** What is Azure SQL Managed Instance?

**T22-E03.** What is Azure Database for PostgreSQL?

**T22-E04.** What is Azure Cosmos DB?

**T22-E05.** What is Azure Cache for Redis?

### Medium

**T22-M01.** What is the difference between Azure SQL Database and SQL Managed Instance?

**T22-M02.** How can Azure SQL be accessed privately?

**T22-M03.** How can an application authenticate to Azure SQL using Entra ID?

**T22-M04.** What is database connection pooling?

**T22-M05.** What are read replicas?

**T22-M06.** What is geo-replication?

**T22-M07.** What is Cosmos DB partitioning?

**T22-M08.** What is a Cosmos DB partition key?

### Hard

**T22-H01.** How would you design database high availability across Azure regions?

**T22-H02.** What would you investigate if database CPU is low but application requests are slow?

**T22-H03.** How would you troubleshoot connection exhaustion between AKS and a managed database?

**T22-H04.** How would you migrate an application database with minimal downtime?

**T22-H05.** When would you choose Cosmos DB over Azure SQL?

---

# 23. 🟠 IMP — Service Bus, Event Grid & Event Hubs

### Easy

**T23-E01.** What is Azure Service Bus?

**T23-E02.** What is Azure Event Grid?

**T23-E03.** What is Azure Event Hubs?

**T23-E04.** What is a queue?

**T23-E05.** What is a topic?

### Medium

**T23-M01.** What is the difference between Service Bus Queue and Service Bus Topic?

**T23-M02.** What is a dead-letter queue?

**T23-M03.** What is message lock duration?

**T23-M04.** What is duplicate detection?

**T23-M05.** What are Service Bus sessions?

**T23-M06.** How is Event Grid different from Service Bus?

**T23-M07.** How is Event Hubs different from Service Bus?

**T23-M08.** What are Event Hubs partitions?

**T23-M09.** What is a consumer group?

### Hard

**T23-H01.** Which service would you use for processing millions of telemetry events per second?

**T23-H02.** Which service would you use for reliable business-command processing?

**T23-H03.** How would you design an asynchronous microservice workflow with retries and dead-letter handling?

**T23-H04.** How would you design idempotent consumers?

**T23-H05.** How would you troubleshoot continuously increasing queue depth?

---

# 24. 🟠 IMP — Git, Branching & Source Control

### Easy

**T24-E01.** What is Git?

**T24-E02.** What is a commit?

**T24-E03.** What is a branch?

**T24-E04.** What is a pull request?

**T24-E05.** What is `git merge`?

**T24-E06.** What is `git rebase`?

### Medium

**T24-M01.** What is the difference between merge and rebase?

**T24-M02.** What is a merge conflict?

**T24-M03.** What is `git cherry-pick`?

**T24-M04.** What is `git revert`?

**T24-M05.** What is the difference between `git revert` and `git reset`?

**T24-M06.** What is GitFlow?

**T24-M07.** What is trunk-based development?

**T24-M08.** What are branch-protection rules?

### Hard

**T24-H01.** What branching strategy would you choose for a team practicing continuous delivery?

**T24-H02.** How would you recover from a bad commit already merged into production?

**T24-H03.** How would you prevent direct pushes to `main`?

**T24-H04.** How would you require CI checks before a pull request can be merged?

**T24-H05.** Why are long-lived feature branches problematic in DevOps environments?

---

# 25. 🟠 IMP — DevSecOps & Software Supply-Chain Security

### Easy

**T25-E01.** What is DevSecOps?

**T25-E02.** What is shift-left security?

**T25-E03.** What is SAST?

**T25-E04.** What is DAST?

**T25-E05.** What is dependency scanning?

**T25-E06.** What is container-image scanning?

**T25-E07.** What is secret scanning?

### Medium

**T25-M01.** Where should security scanning occur in a CI/CD pipeline?

**T25-M02.** What is an SBOM?

**T25-M03.** What is image signing?

**T25-M04.** What is artifact attestation?

**T25-M05.** What is build provenance?

**T25-M06.** Why should CI pipelines use short-lived credentials?

**T25-M07.** What is policy as code?

**T25-M08.** How can Terraform configurations be security scanned?

### Hard

**T25-H01.** How would you design a secure software-supply-chain pipeline targeting AKS?

**T25-H02.** How would you prevent unsigned images from running in production?

**T25-H03.** What would you do if a critical vulnerability were found in an image already running in production?

**T25-H04.** How would you secure third-party GitHub Actions or Azure Pipeline tasks?

**T25-H05.** How would you prevent CI/CD compromise from becoming an Azure subscription compromise?

**T25-H06.** How would you implement least privilege from GitHub Actions through ACR to AKS?

---

# 26. 🟡 GOOD — Azure CLI, PowerShell & Cloud Shell

### Easy

**T26-E01.** What is Azure CLI?

**T26-E02.** What is Azure PowerShell?

**T26-E03.** What is Azure Cloud Shell?

**T26-E04.** How do you authenticate Azure CLI?

**T26-E05.** How do you check the currently selected Azure subscription?

### Medium

**T26-M01.** How do you change the active Azure subscription using CLI?

**T26-M02.** How do you list resource groups using Azure CLI?

**T26-M03.** How do you query JSON results from Azure CLI?

**T26-M04.** How would you automate creating Azure resources using CLI scripts?

**T26-M05.** When would you choose Azure CLI over Terraform?

### Hard

**T26-H01.** How would you write an idempotent Azure CLI deployment script?

**T26-H02.** How would you authenticate an automation script without storing user credentials?

**T26-H03.** How would you troubleshoot a script that works locally but fails from a CI agent?

**T26-H04.** How would you automate operations across multiple Azure subscriptions?

---

# 27. 🟡 GOOD — ARM Templates & Bicep

### Easy

**T27-E01.** What is an ARM template?

**T27-E02.** What is Bicep?

**T27-E03.** Why was Bicep introduced?

**T27-E04.** Is Bicep declarative or imperative?

### Medium

**T27-M01.** What are Bicep parameters?

**T27-M02.** What are Bicep variables?

**T27-M03.** What are Bicep outputs?

**T27-M04.** What are Bicep modules?

**T27-M05.** What is incremental deployment?

**T27-M06.** What is a deployment scope?

**T27-M07.** How does Bicep differ from Terraform?

### Hard

**T27-H01.** When would you choose Bicep instead of Terraform?

**T27-H02.** How would you create reusable infrastructure using Bicep modules?

**T27-H03.** How would you safely manage dev and production with the same Bicep code?

**T27-H04.** What advantages does Bicep gain from being Azure-native?

---

# 28. 🟡 GOOD — Azure Automation, Update Manager & Operations

### Easy

**T28-E01.** What is Azure Automation?

**T28-E02.** What is an Automation Account?

**T28-E03.** What is an Azure Automation runbook?

**T28-E04.** What is Azure Update Manager?

### Medium

**T28-M01.** What types of tasks are suitable for Azure Automation?

**T28-M02.** How can an Automation Account authenticate securely to Azure?

**T28-M03.** How would you schedule an automation runbook?

**T28-M04.** How can Azure Update Manager help manage VM patching?

**T28-M05.** How would you coordinate patching for production VMs?

### Hard

**T28-H01.** How would you automate stopping nonproduction VMs overnight?

**T28-H02.** How would you design a safe production patch-management process?

**T28-H03.** How would you ensure patching does not take all application instances offline simultaneously?

**T28-H04.** How would you automatically remediate a recurring Azure configuration issue?

---

# 29. 🟡 GOOD — Azure Landing Zones, Cloud Adoption Framework & Azure Arc

### Easy

**T29-E01.** What is an Azure landing zone?

**T29-E02.** What is the Azure Cloud Adoption Framework?

**T29-E03.** What is Azure Arc?

**T29-E04.** What is a platform landing zone?

**T29-E05.** What is an application landing zone?

### Medium

**T29-M01.** Why are management groups important in enterprise landing zones?

**T29-M02.** What responsibilities should a platform team centralize?

**T29-M03.** How would you separate platform resources from workload resources?

**T29-M04.** What is a hub-and-spoke landing-zone network?

**T29-M05.** What problems does Azure Arc solve?

### Hard

**T29-H01.** How would you design an Azure foundation for 100 application teams?

**T29-H02.** How would you enforce centralized governance while allowing development teams autonomy?

**T29-H03.** How would you onboard a new subscription into an enterprise Azure environment?

**T29-H04.** How would you manage both Azure and on-premises resources using a common governance model?

---

# 30. 🟡 GOOD — Azure Well-Architected Framework, SRE & Architecture

Microsoft's current Well-Architected Framework centers on Reliability, Security, Cost Optimization, Operational Excellence, and Performance Efficiency. ([Microsoft Learn][2])

### Easy

**T30-E01.** What is the Azure Well-Architected Framework?

**T30-E02.** What are the five pillars of the Azure Well-Architected Framework?

**T30-E03.** What is reliability?

**T30-E04.** What is operational excellence?

**T30-E05.** What is performance efficiency?

**T30-E06.** What is cost optimization?

**T30-E07.** What is security?

**T30-E08.** What is an SLA?

**T30-E09.** What is an SLI?

**T30-E10.** What is an SLO?

### Medium

**T30-M01.** What is an error budget?

**T30-M02.** What is the difference between availability and reliability?

**T30-M03.** What is observability?

**T30-M04.** What are logs, metrics, and traces?

**T30-M05.** What is toil in SRE?

**T30-M06.** What is MTTR?

**T30-M07.** What is MTBF?

**T30-M08.** What is a blameless postmortem?

**T30-M09.** What is graceful degradation?

**T30-M10.** What is fault isolation?

### Hard

**T30-H01.** How would you calculate availability when several dependent services have different SLAs?

**T30-H02.** How would you decide whether a workload needs multi-zone or multi-region architecture?

**T30-H03.** How would you balance reliability against cost?

**T30-H04.** How would you establish SLIs and SLOs for an API?

**T30-H05.** What should happen when a team's error budget is exhausted?

**T30-H06.** How would you design an Azure workload to degrade gracefully when one dependency fails?

**T30-H07.** How would you reduce operational toil in an Azure environment?

**T30-H08.** What would a good production post-incident review contain?

---

# 31. 🟡 GOOD — Azure Migration & Modernization

### Easy

**T31-E01.** What is cloud migration?

**T31-E02.** What is Azure Migrate?

**T31-E03.** What is lift-and-shift?

**T31-E04.** What is rehosting?

**T31-E05.** What is refactoring?

### Medium

**T31-M01.** What are common cloud-migration strategies?

**T31-M02.** How would you assess an on-premises workload before moving it to Azure?

**T31-M03.** How do dependencies affect migration planning?

**T31-M04.** How would you choose an Azure VM size for an existing on-premises server?

**T31-M05.** When should an application be rehosted versus modernized?

**T31-M06.** How can databases be migrated to Azure?

### Hard

**T31-H01.** How would you migrate a critical application with minimal downtime?

**T31-H02.** How would you migrate hundreds of interdependent VMs to Azure?

**T31-H03.** How would you validate that a migrated application performs correctly before cutover?

**T31-H04.** What would your rollback strategy be if a production migration failed?

**T31-H05.** How would you modernize a VM-based application into Azure PaaS or container services?

---

# 32. 🔴 M IMP — Real Azure Architecture Scenarios

### Easy/Medium

**T32-01.** Design Azure infrastructure for a three-tier web application.

**T32-02.** How would you deploy a public frontend, private backend, and private database?

**T32-03.** How would you secure communication between those tiers?

**T32-04.** How would you design separate dev, staging, and production environments?

**T32-05.** How would you give developers access to dev while restricting production?

**T32-06.** How would you centralize logs from all environments?

**T32-07.** How would you configure alerts for production?

**T32-08.** How would you manage application secrets?

**T32-09.** How would you implement automatic scaling?

**T32-10.** How would you protect the application from common web attacks?

### Hard

**T32-11.** Design an Azure architecture for an internet-facing application serving millions of users.

**T32-12.** Design a highly available AKS platform across Availability Zones.

**T32-13.** Design an AKS platform where the Kubernetes API, nodes, database, registry, and Key Vault have no public exposure.

**T32-14.** Design a multi-region Azure application capable of surviving a complete region failure.

**T32-15.** Design networking for 20 application teams using hub-and-spoke architecture.

**T32-16.** Design CI/CD for 50 microservices deployed to AKS.

**T32-17.** Design a GitOps deployment architecture using GitHub Actions, ACR, AKS, and Argo CD.

**T32-18.** Design Azure authentication for GitHub Actions without storing client secrets.

**T32-19.** Design secretless authentication from AKS workloads to Azure services.

**T32-20.** Design centralized monitoring for VMs, AKS, PaaS applications, and databases.

**T32-21.** Design disaster recovery for an application with a five-minute RPO.

**T32-22.** Design an Azure architecture that meets PCI-DSS-style isolation requirements.

**T32-23.** Design infrastructure that allows outbound internet traffic only through a centralized firewall.

**T32-24.** Design a platform where developers cannot manually change production infrastructure.

**T32-25.** Design a Terraform workflow for several teams and multiple Azure subscriptions.

---

# 33. 🔴 M IMP — DevOps Production & Behavioral Scenario Questions

### Deployment Scenarios

**T33-01.** Your production deployment fails halfway through; what do you do?

**T33-02.** Your new version causes 5xx errors immediately after release; what do you do?

**T33-03.** A release works in staging but fails in production; how would you investigate it?

**T33-04.** How would you minimize deployment downtime?

**T33-05.** What is a rolling deployment?

**T33-06.** What is a blue-green deployment?

**T33-07.** What is a canary deployment?

**T33-08.** When would you choose canary over blue-green?

**T33-09.** How would you perform an automatic rollback?

**T33-10.** What metrics would determine whether a deployment should continue or roll back?

### Infrastructure Scenarios

**T33-11.** Someone manually changes an Azure resource managed by Terraform; what do you do?

**T33-12.** Terraform wants to recreate a production resource unexpectedly; what do you do before applying?

**T33-13.** Terraform state becomes locked; how would you investigate it?

**T33-14.** A pipeline accidentally destroyed a production resource; how would you prevent this from happening again?

**T33-15.** How would you ensure infrastructure changes receive peer review?

### Security Scenarios

**T33-16.** A service-principal secret has been committed to Git; what are your immediate actions?

**T33-17.** A developer requests Owner access to production because their deployment is failing; what would you do?

**T33-18.** Your application secret expires and production goes down; how would you redesign the system to prevent recurrence?

**T33-19.** A container scan identifies a critical CVE one hour before a scheduled deployment; what would you do?

**T33-20.** You discover that an Azure Storage Account containing sensitive data is publicly accessible; what do you do?

### Incident Scenarios

**T33-21.** What are your first steps when receiving a production outage alert?

**T33-22.** How do you determine incident severity?

**T33-23.** When should you roll back versus troubleshoot forward?

**T33-24.** How would you communicate during a major production incident?

**T33-25.** What information should be captured during an incident?

**T33-26.** What should be included in a postmortem?

**T33-27.** How would you prevent the same incident from recurring?

**T33-28.** How do you distinguish symptoms from root causes?

**T33-29.** What is the difference between remediation and root-cause correction?

**T33-30.** How would you prioritize reliability work against feature development?

---

# 34. 🔴 M IMP — Cross-Service Comparison Questions

**T34-01.** Availability Set vs Availability Zone?

**T34-02.** Azure Load Balancer vs Application Gateway?

**T34-03.** Application Gateway vs Front Door?

**T34-04.** Front Door vs Traffic Manager?

**T34-05.** NSG vs Azure Firewall?

**T34-06.** Azure Firewall vs WAF?

**T34-07.** Service Endpoint vs Private Endpoint?

**T34-08.** VPN Gateway vs ExpressRoute?

**T34-09.** Azure Bastion vs public SSH/RDP?

**T34-10.** VNet peering vs VPN?

**T34-11.** NAT Gateway vs Load Balancer outbound connectivity?

**T34-12.** System-assigned vs user-assigned managed identity?

**T34-13.** Managed identity vs service principal?

**T34-14.** Azure RBAC vs Entra directory roles?

**T34-15.** Azure Policy vs RBAC?

**T34-16.** Azure Policy vs resource locks?

**T34-17.** Contributor vs Owner?

**T34-18.** Resource group vs subscription?

**T34-19.** Management group vs subscription?

**T34-20.** LRS vs ZRS?

**T34-21.** GRS vs GZRS?

**T34-22.** Blob Storage vs Azure Files?

**T34-23.** Managed disk vs Azure Files?

**T34-24.** App Service vs VM?

**T34-25.** App Service vs Container Apps?

**T34-26.** Container Apps vs AKS?

**T34-27.** AKS vs VM Scale Sets?

**T34-28.** Azure Functions vs Container Apps?

**T34-29.** ACR image tag vs image digest?

**T34-30.** Azure Monitor metrics vs Log Analytics logs?

**T34-31.** Activity Log vs resource logs?

**T34-32.** Application Insights vs Log Analytics?

**T34-33.** Azure Backup vs Site Recovery?

**T34-34.** RTO vs RPO?

**T34-35.** Active-active vs active-passive?

**T34-36.** Horizontal scaling vs vertical scaling?

**T34-37.** HPA vs AKS cluster autoscaler?

**T34-38.** Terraform vs Bicep?

**T34-39.** Terraform `count` vs `for_each`?

**T34-40.** Local Terraform state vs remote state?

**T34-41.** Azure DevOps Pipelines vs GitHub Actions?

**T34-42.** Microsoft-hosted vs self-hosted pipeline agents?

**T34-43.** Continuous Delivery vs Continuous Deployment?

**T34-44.** Rolling deployment vs blue-green deployment?

**T34-45.** Blue-green vs canary deployment?

**T34-46.** SAST vs DAST?

**T34-47.** Authentication vs authorization?

**T34-48.** Encryption at rest vs encryption in transit?

**T34-49.** Service Bus vs Event Grid?

**T34-50.** Service Bus vs Event Hubs?

**T34-51.** Azure SQL Database vs SQL Managed Instance?

**T34-52.** Azure SQL vs Cosmos DB?

**T34-53.** IaaS vs PaaS?

**T34-54.** High availability vs disaster recovery?

**T34-55.** SLA vs SLO?

**T34-56.** Monitoring vs observability?

**T34-57.** Logs vs metrics vs traces?

**T34-58.** Azure Advisor vs Azure Monitor?

**T34-59.** Azure Defender for Cloud vs Microsoft Sentinel?

**T34-60.** Azure Firewall vs NAT Gateway?

[1]: https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104?utm_source=chatgpt.com "Study guide for Exam AZ-104: Microsoft Azure Administrator | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/azure/well-architected/workloads?utm_source=chatgpt.com "Azure Well-Architected Framework workloads - Microsoft Azure Well-Architected Framework | Microsoft Learn"


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
