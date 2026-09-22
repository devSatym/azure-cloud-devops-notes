<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure RBAC (Role-Based Access Control) — AZ-104 Revision Notes

---

## 1. What is Azure RBAC?

- **Authorization system** built on Azure Resource Manager (ARM) to manage access to Azure resources
- Controls **who** can do **what** on **which** Azure resources
- Uses **role assignments** = Security Principal + Role Definition + Scope
- **Additive model** — effective permissions = union of all assigned roles
- Evaluated on **every** Azure Resource Manager request
- **Not the same** as Entra ID (Azure AD) roles — RBAC = Azure resources, Entra roles = directory objects

> ⚠️ **EXAM TIP:** Azure RBAC ≠ Entra ID roles. RBAC manages Azure **resources** (VMs, Storage, Networks). Entra ID roles manage **directory** objects (users, groups, apps). They are separate systems.

---

## 2. Key Components

| Component | Description |
|---|---|
| **Security Principal** | WHO — User, Group, Service Principal, Managed Identity |
| **Role Definition** | WHAT — Collection of permissions (Actions / NotActions / DataActions / NotDataActions) |
| **Scope** | WHERE — Management Group, Subscription, Resource Group, Resource |
| **Role Assignment** | Binding of Principal + Role + Scope |
| **Deny Assignment** | Explicitly blocks actions (created by Azure Blueprints / managed apps, not directly by users) |

---

## 3. Scope Hierarchy

```
Management Group
  └── Subscription
       └── Resource Group
            └── Resource
```

- Roles assigned at a **higher scope** are **inherited** by lower scopes
- Management Group → Subscription → Resource Group → Resource
- A role at Subscription level = applies to ALL Resource Groups and Resources in that subscription
- A role at Resource Group level = applies to ALL Resources in that RG

> ⚠️ **EXAM TIP:** Role assignments are **inherited downward**. An assignment at Management Group level flows to ALL subscriptions, RGs, and resources below. You **cannot block** inheritance (but deny assignments can override).

---

## 4. Built-in Roles

### Fundamental Roles (Most Tested)

| Role | Permissions | Assign Roles? |
|---|---|---|
| **Owner** | Full access to ALL resources + CAN assign roles to others | ✅ Yes |
| **Contributor** | Full access to ALL resources, CANNOT assign roles | ❌ No |
| **Reader** | View ALL resources, CANNOT modify anything | ❌ No |
| **User Access Administrator** | Manage user access (assign roles), NO resource management | ✅ Yes |

### Key Differences — Owner vs Contributor

| Feature | Owner | Contributor |
|---|---|---|
| Create/modify/delete resources | ✅ | ✅ |
| Manage RBAC role assignments | ✅ | ❌ |
| Manage Blueprints assignments | ✅ | ❌ |
| Manage policies | ✅ (assign) | ❌ (cannot assign) |
| View cost / billing | ✅ | ✅ |
| Full resource management | ✅ | ✅ |

> ⚠️ **EXAM TIP:** **Contributor** can do everything EXCEPT manage role assignments. If a question says "grant full access but cannot assign roles to others" → answer = **Contributor**. Only **Owner** and **User Access Administrator** can assign roles.

### Common Service-Specific Roles (Exam-Relevant)

| Role | Scope | Permissions |
|---|---|---|
| **Virtual Machine Contributor** | VMs | Manage VMs, not VNet or Storage they connect to |
| **Virtual Machine Administrator Login** | VMs | Login with admin privileges (RDP/SSH) |
| **Virtual Machine User Login** | VMs | Login with user privileges |
| **Network Contributor** | Networking | Manage networks, not access to them |
| **Storage Account Contributor** | Storage | Manage storage accounts, not data inside |
| **Storage Blob Data Owner** | Storage | Full access to blob containers and data + RBAC |
| **Storage Blob Data Contributor** | Storage | Read, write, delete blob containers and data |
| **Storage Blob Data Reader** | Storage | Read blob containers and data |
| **Storage Queue Data Contributor** | Storage | Read, write, delete queues and messages |
| **Storage Queue Data Reader** | Storage | Read queues and messages |
| **Key Vault Administrator** | Key Vault | Full access to Key Vault (RBAC model) |
| **Key Vault Secrets User** | Key Vault | Read secret contents |
| **Monitoring Reader** | Monitoring | Read all monitoring data |
| **Monitoring Contributor** | Monitoring | Read + manage monitoring settings, alerts |
| **Log Analytics Reader** | Log Analytics | Read log data |
| **Log Analytics Contributor** | Log Analytics | Read + configure Log Analytics |
| **Backup Operator** | Backup | Manage backup except vault creation/access |
| **Backup Contributor** | Backup | Manage backup including vault creation |
| **Security Reader** | Security | Read security data and policies |
| **Security Admin** | Security | View + update security policies and alerts |
| **Cost Management Reader** | Billing | View cost data |
| **Cost Management Contributor** | Billing | View + manage cost data |
| **DNS Zone Contributor** | DNS | Manage DNS zones and records |

### Storage Data Plane Roles

| Role | Data Access | Management Access |
|---|---|---|
| **Storage Account Contributor** | ❌ No data access | ✅ Manage account (keys, settings) |
| **Storage Blob Data Owner** | ✅ Full (read/write/delete + RBAC) | ❌ |
| **Storage Blob Data Contributor** | ✅ Read/Write/Delete blobs | ❌ |
| **Storage Blob Data Reader** | ✅ Read-only blobs | ❌ |
| **Reader and Data Access** | ✅ Read account keys → data access | ✅ Read management |

> ⚠️ **EXAM TIP:** **Storage Account Contributor** manages the storage account but does NOT give access to the data (blobs, queues, tables). For data access, use **Storage Blob Data** roles. This is a very common exam question.

---

## 5. Role Definitions — Structure

```json
{
  "Name": "Custom Role Name",
  "Description": "What the role does",
  "Actions": ["Microsoft.Compute/virtualMachines/read"],
  "NotActions": ["Microsoft.Compute/virtualMachines/delete"],
  "DataActions": ["Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"],
  "NotDataActions": [],
  "AssignableScopes": ["/subscriptions/{sub-id}"]
}
```

### Permission Types

| Permission | Description | Example |
|---|---|---|
| **Actions** | Management plane operations ALLOWED | `Microsoft.Compute/virtualMachines/*` |
| **NotActions** | Management operations EXCLUDED from Actions | `Microsoft.Authorization/roleAssignments/write` |
| **DataActions** | Data plane operations ALLOWED | `Microsoft.Storage/.../blobs/read` |
| **NotDataActions** | Data operations EXCLUDED from DataActions | `Microsoft.Storage/.../blobs/delete` |

### How Permissions Are Calculated
```
Effective Permissions = Actions - NotActions
Effective Data Permissions = DataActions - NotDataActions
```

- **NotActions** is a subtraction from Actions (NOT a deny)
- If a different role grants the same action, **it will still be permitted** (additive model)
- **Wildcard**: `*` = all actions; `*/read` = all read actions on all resource types

> ⚠️ **EXAM TIP:** **NotActions ≠ Deny**. NotActions only subtracts from Actions within the SAME role definition. If another assigned role grants the same permission, the user STILL has it (additive). True deny = **Deny Assignments** only.

---

## 6. Custom Roles

- Create roles with **specific permissions** not available in built-in roles
- Requires **Microsoft.Authorization/roleDefinitions/write** permission (Owner or User Access Administrator)
- **Assignable Scopes** = where the custom role can be used (Management Group, Subscription, or Resource Group)
- Limits:
  - **5,000 custom roles** per tenant
  - **2,000 role assignments** per subscription (increased from previous limits)
  - Custom role NOT assignable at Resource level scope for AssignableScopes

### Portal Path — Create Custom Role
```
Subscription (or RG) → Access control (IAM) → + Add → Add custom role →
Start from scratch / Clone a role / Start from JSON →
Name, Description →
Permissions: + Add permissions → Select provider → Check permissions →
Exclude permissions (NotActions) →
Assignable scopes: Add scopes (subscriptions/RGs) →
JSON (review) → Review + Create
```

### CLI — Create Custom Role
```bash
# Create from JSON file
az role definition create --role-definition custom-role.json

# List custom roles
az role definition list --custom-role-only --output table

# Update custom role
az role definition update --role-definition updated-role.json

# Delete custom role
az role definition delete --name "Custom Role Name"
```

### PowerShell — Create Custom Role
```powershell
# Get existing role as template
$role = Get-AzRoleDefinition "Reader"
$role.Id = $null
$role.Name = "Custom VM Operator"
$role.Description = "Can start, restart, and monitor VMs"
$role.Actions.Clear()
$role.Actions.Add("Microsoft.Compute/virtualMachines/start/action")
$role.Actions.Add("Microsoft.Compute/virtualMachines/restart/action")
$role.Actions.Add("Microsoft.Compute/virtualMachines/read")
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/{sub-id}")
New-AzRoleDefinition -Role $role

# List custom roles
Get-AzRoleDefinition -Custom

# Delete custom role
Remove-AzRoleDefinition -Name "Custom VM Operator"
```

> ⚠️ **EXAM TIP:** Custom role limit = **5,000 per tenant**. Max role assignments = **2,000 per subscription**. AssignableScopes define WHERE the role can be assigned (not who). Custom roles need **Owner** or **User Access Administrator** to create.

---

## 7. Role Assignments

### How to Assign a Role

| Step | Description |
|---|---|
| 1. **Scope** | Choose where: Management Group / Subscription / Resource Group / Resource |
| 2. **Role** | Choose which: Built-in or custom role |
| 3. **Principal** | Choose who: User, Group, Service Principal, Managed Identity |
| 4. **Condition** (Optional) | ABAC conditions for data actions (e.g., blob index tags) |

### Portal Path — Assign Role
```
Resource / Resource Group / Subscription → Access control (IAM) →
+ Add → Add role assignment →
Role tab: Select role →
Members tab: Select members (User / Group / Service Principal / Managed Identity) →
Conditions tab (optional): Add condition →
Review + assign
```

### Portal Path — View Role Assignments
```
Resource / RG / Subscription → Access control (IAM) →
Role assignments tab → View all current assignments
```

### Portal Path — Check Access
```
Resource / RG / Subscription → Access control (IAM) →
Check access tab → Search for user/group/SP →
View effective permissions for that principal
```

### Role Assignment Limits

| Limit | Value |
|---|---|
| Role assignments per subscription | **4,000** |
| Role assignments per management group | **500** |
| Custom roles per tenant | **5,000** |
| Role definition max description | 2,048 characters |
| Custom role max conditions | Varies by resource type |

> ⚠️ **EXAM TIP:** Max role assignments = **4,000 per subscription** (was 2,000; increased). Use **groups** to reduce assignment count — assign role to group, not individual users. One group assignment = 1 count regardless of members.

---

## 8. Deny Assignments

- **Explicitly block** specific actions even if a role grants them
- **Takes precedence** over role assignments (deny > allow)
- **Cannot be created directly by users** — only by Azure Blueprints and Azure Managed Apps
- Evaluation order: **Deny Assignment → Role Assignment**
- If deny exists for an action → user cannot perform it even if role allows it

### Deny vs NotActions

| Feature | Deny Assignment | NotActions |
|---|---|---|
| **Effect** | Blocks action regardless of other roles | Subtracts from Actions within same role |
| **Scope** | Can block at any scope | Only within the role definition |
| **Overridable** | ❌ No (deny always wins) | ✅ Yes (another role can grant same action) |
| **Created by** | Azure Blueprints / Managed Apps | Role definition author |
| **User-creatable** | ❌ No | ✅ Yes |

> ⚠️ **EXAM TIP:** Users **cannot directly create** deny assignments. Only Azure Blueprints and Managed Applications can. Deny **always beats** allow. NotActions ≠ Deny.

---

## 9. Azure RBAC vs Entra ID Roles vs Azure Policy

| Feature | Azure RBAC | Entra ID Roles | Azure Policy |
|---|---|---|---|
| **Purpose** | Manage access to Azure resources | Manage Entra ID directory | Enforce resource compliance |
| **Scope** | Management Group → Resource | Entra ID tenant / AU | Management Group → Resource |
| **Example** | VM Contributor, Reader | Global Admin, User Admin | "VMs must use managed disks" |
| **Assigned at** | IAM (Access Control) | Entra ID → Roles | Policy → Assignments |
| **Who is affected** | Users, groups, SPs, MIs | Users, groups, apps | Resources (not users) |
| **Effect** | Allow/Deny actions | Allow directory operations | Audit/Deny/Append/Modify/DeployIfNotExists |
| **Custom** | ✅ Custom roles | ✅ Custom roles (P1) | ✅ Custom policies |

> ⚠️ **EXAM TIP:** RBAC = who can do what. Policy = what is allowed to exist. RBAC controls **user actions**. Policy controls **resource configurations**. They complement each other — RBAC doesn't restrict resource properties.

---

## 10. Conditions & ABAC (Attribute-Based Access Control)

- Add **conditions** to role assignments for **fine-grained** access control
- Currently supports **Azure Storage** data actions (blobs)
- Conditions use attributes: blob index tags, container name, blob path, etc.
- Example: "User can read blobs only where tag Project=Alpha"
- Role assignment + condition = ABAC (Attribute-Based Access Control)

### Condition Example
```
Allow Storage Blob Data Reader
WHERE
  @Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<$key_case_sensitive$>] StringEquals 'Alpha'
```

### Portal Path — Add Condition
```
Resource → Access control (IAM) → + Add → Add role assignment →
Role → Members → Conditions tab →
+ Add condition → Select attribute → Define expression → Save →
Review + assign
```

> ⚠️ **EXAM TIP:** ABAC conditions currently work primarily with **Azure Storage blob data roles**. Conditions are optional and provide more granular control within a role assignment.

---

## 11. Managed Identities & RBAC

### Types

| Type | Description | Lifecycle |
|---|---|---|
| **System-assigned** | Tied to one resource; auto-created and deleted with resource | Same as resource |
| **User-assigned** | Standalone Azure resource; can be shared across multiple resources | Independent |

- Managed Identities authenticate to Azure services **without storing credentials**
- Assign **Azure RBAC roles** to managed identities just like any other principal
- Common use: VM with system MI → assign Storage Blob Data Reader → VM reads blobs without keys

### Portal Path — Assign Role to Managed Identity
```
Resource (e.g., Storage Account) → Access control (IAM) →
+ Add → Add role assignment → Select role →
Members → + Select members → Managed identity → Select MI → Assign
```

### Portal Path — Enable System MI on VM
```
Virtual Machine → Settings → Identity → System assigned → Status: On → Save
```

> ⚠️ **EXAM TIP:** Managed Identity = service principal automatically managed. **System-assigned** = 1:1 with resource, deleted when resource deleted. **User-assigned** = standalone, reusable, persists independently.

---

## 12. Elevate Access — Global Admin to Azure RBAC

- **Global Administrator** in Entra ID can elevate to **User Access Administrator** at root scope (`/`)
- Grants ability to manage RBAC across ALL Azure subscriptions in the tenant
- Must be **explicitly enabled** — not automatic
- Should be **disabled after use** (least privilege)

### Portal Path
```
Microsoft Entra ID → Properties →
Access management for Azure resources → Toggle to Yes → Save
```

### Effect
- Gives Global Admin **User Access Administrator** role at root (`/`) scope
- Can then assign Owner or any role on any subscription

> ⚠️ **EXAM TIP:** This is a **one-way emergency action** — Global Admin elevates to manage ALL subscriptions. Must manually toggle on. Applies at tenant root scope. **Turn off after use**.

---

## 13. Resource Locks & RBAC Interaction

| Lock Type | Effect | Who Can Delete Lock |
|---|---|---|
| **ReadOnly** | Resources can be read but NOT modified or deleted | Owner or User Access Administrator |
| **CanNotDelete (Delete)** | Resources can be modified but NOT deleted | Owner or User Access Administrator |

- Locks apply to **ALL users** regardless of RBAC role (even Owner)
- Must remove the lock first to perform the locked action
- Locks are **inherited** down from RG to resources
- To manage locks: need `Microsoft.Authorization/locks/*` permission (Owner / User Access Administrator)

### Portal Path — Create Lock
```
Resource / RG / Subscription → Settings → Locks →
+ Add → Lock name → Lock type (Read-only / Delete) → Notes → OK
```

> ⚠️ **EXAM TIP:** Resource locks apply to **ALL users** including Owner. A **ReadOnly** lock on a storage account prevents listing keys (which is a POST operation). Even Owner must remove the lock first. Only Owner and User Access Administrator can manage locks.

---

## 14. Classic Administrators (Legacy)

| Role | Description | Status |
|---|---|---|
| **Account Administrator** | Billing owner of subscription (1 per subscription) | Legacy |
| **Service Administrator** | Full access to subscription (= Owner at sub level) | Legacy |
| **Co-Administrator** | Full access to subscription (= Owner at sub level except IAM) | Legacy |

- **Max co-administrators** = 200 per subscription
- Classic roles predated Azure RBAC — use RBAC instead
- Account Admin can be checked in subscription properties
- Co-admins added via: `Subscription → Access control (IAM) → Classic administrators`

> ⚠️ **EXAM TIP:** Classic admins are **legacy**. 1 Account Admin + 1 Service Admin + up to 200 Co-Admins per subscription. **New deployments should use RBAC only**. Questions may test classic vs RBAC differences.

---

## 15. CLI & PowerShell Commands

### Azure CLI

```bash
# List all role assignments at subscription scope
az role assignment list --subscription <sub-id> --output table

# List role assignments for a specific user
az role assignment list --assignee user@contoso.com --output table

# Assign a role
az role assignment create \
  --assignee user@contoso.com \
  --role "Contributor" \
  --scope "/subscriptions/{sub-id}/resourceGroups/{rg-name}"

# Assign role to group
az role assignment create \
  --assignee-object-id <group-object-id> \
  --assignee-principal-type Group \
  --role "Reader" \
  --scope "/subscriptions/{sub-id}"

# Remove a role assignment
az role assignment delete \
  --assignee user@contoso.com \
  --role "Contributor" \
  --scope "/subscriptions/{sub-id}/resourceGroups/{rg-name}"

# List all built-in role definitions
az role definition list --output table

# Show specific role definition
az role definition list --name "Contributor" --output json

# Create custom role from JSON
az role definition create --role-definition custom-role.json

# Delete custom role
az role definition delete --name "Custom Role Name"

# List custom roles only
az role definition list --custom-role-only --output table
```

### Azure PowerShell

```powershell
# List all role assignments
Get-AzRoleAssignment -Scope "/subscriptions/{sub-id}"

# List role assignments for a user
Get-AzRoleAssignment -SignInName user@contoso.com

# Assign a role
New-AzRoleAssignment -SignInName user@contoso.com `
  -RoleDefinitionName "Contributor" `
  -ResourceGroupName "myRG"

# Assign role to group
New-AzRoleAssignment -ObjectId <group-object-id> `
  -RoleDefinitionName "Reader" `
  -Scope "/subscriptions/{sub-id}"

# Remove role assignment
Remove-AzRoleAssignment -SignInName user@contoso.com `
  -RoleDefinitionName "Contributor" `
  -ResourceGroupName "myRG"

# List built-in role definitions
Get-AzRoleDefinition | Where-Object { $_.IsCustom -eq $false }

# Get specific role
Get-AzRoleDefinition -Name "Contributor"

# Create custom role
New-AzRoleDefinition -InputFile custom-role.json

# Delete custom role
Remove-AzRoleDefinition -Name "Custom Role Name"
```

> ⚠️ **EXAM TIP:** `--scope` uses full resource ID path. `--assignee` takes UPN for users, object ID for groups/SPs. Use `--assignee-principal-type` when assigning to groups to avoid ambiguity.

---

## 16. Monitoring Role Assignments

### Activity Log
- All role assignment changes logged in **Activity Log**
- Category: **Administrative**
- Operations: `Microsoft.Authorization/roleAssignments/write`, `Microsoft.Authorization/roleAssignments/delete`

### Portal Path — View RBAC Changes
```
Azure Monitor → Activity Log → Filter:
  Operation: "Create role assignment" / "Delete role assignment"
  → View details (who, when, what role, what scope)
```

### Export to Log Analytics
```
Azure Monitor → Activity Log → Export Activity Logs →
+ Add Diagnostic Setting → Administrative category →
Send to Log Analytics → Save
```

### Access Reviews (P2)
- Periodically review role assignments
- `Entra ID → Identity Governance → Access Reviews → + New` → Select Azure resource roles

> ⚠️ **EXAM TIP:** All RBAC changes are in the **Activity Log**. Export to Log Analytics for retention beyond 90 days. Use **Access Reviews (P2)** to periodically review role assignments.

---

## 17. Best Practices for RBAC

| Practice | Reason |
|---|---|
| **Assign roles to Groups**, not users | Reduces assignment count; easier management |
| **Use built-in roles** first | Custom only when built-in doesn't fit |
| **Least privilege** | Assign minimum permissions needed |
| **Use Reader** as default | Only elevate when necessary |
| **Avoid Owner** at subscription level | Too broad; use specific roles |
| **Use Managed Identities** | No credentials to manage for apps/services |
| **Use PIM** for privileged roles | Just-in-time access (P2) |
| **Use Conditions (ABAC)** | Fine-grained data access on storage |
| **Regular access reviews** | Remove stale assignments |
| **Resource locks** for critical resources | Prevent accidental deletion |

---

## 18. Pricing Key Points

| Item | Cost |
|---|---|
| **Azure RBAC** | **FREE** — no additional cost |
| **Built-in roles** | Free |
| **Custom roles** | Free |
| **Role assignments** | Free |
| **Resource locks** | Free |
| **PIM** (just-in-time RBAC) | **P2 license** required |
| **Access Reviews** | **P2 license** required |
| **Conditional Access + RBAC** | **P1 license** for CA |

> ⚠️ **EXAM TIP:** RBAC itself is **completely free**. No charge for roles, assignments, or custom roles. PIM and Access Reviews (which enhance RBAC) require P2.

---

## 19. Quick-Fire Exam Points ⚡

1. **RBAC** = authorization system for Azure resources, built on ARM
2. Role Assignment = **Security Principal + Role Definition + Scope**
3. **Additive model** — permissions are unioned across all role assignments
4. Scope hierarchy: **Management Group → Subscription → Resource Group → Resource**
5. Roles **inherit downward** — assigned at MG → applies to all subscriptions/RG/resources below
6. **Owner** = full access + assign roles; **Contributor** = full access - assign roles
7. **User Access Administrator** = manage role assignments only, no resource management
8. Only **Owner** and **User Access Administrator** can assign roles
9. **Contributor CANNOT** assign roles — most common exam question
10. **NotActions ≠ Deny** — only subtracts from Actions within same role; other roles can still grant
11. **Deny Assignments** = true deny, created by Blueprints/Managed Apps only, NOT user-creatable
12. **Deny > Allow** — deny assignments always override role assignments
13. Max role assignments per subscription = **4,000**
14. Max role assignments per management group = **500**
15. Max custom roles per tenant = **5,000**
16. **Storage Account Contributor** = manage account, NO data access
17. **Storage Blob Data Reader/Contributor/Owner** = data plane access to blobs
18. Custom role created via JSON with Actions, NotActions, DataActions, NotDataActions, AssignableScopes
19. **AssignableScopes** = where role can be used (not who gets it)
20. **Resource locks** apply to ALL users including Owner — must remove lock first
21. **ReadOnly lock** on storage = prevents listing keys (POST operation)
22. Only Owner / User Access Administrator can manage locks
23. Classic admins: Account Admin (1) + Service Admin (1) + Co-Admins (up to 200) — **legacy**
24. **Global Admin** can elevate to User Access Administrator at root scope via Entra ID Properties
25. **Managed Identity** = service principal with auto-managed credentials; assign RBAC roles normally
26. **System-assigned MI** = 1:1 with resource, deleted with resource
27. **User-assigned MI** = standalone resource, reusable across resources
28. ABAC conditions work primarily with **Azure Storage** blob data roles
29. Azure RBAC is **completely free** — no cost for roles, assignments, custom roles
30. **PIM** (P2) = just-in-time role activation for RBAC roles
31. Activity Log records all role assignment changes — export for >90 days retention
32. Assign roles to **Groups** to reduce assignment count and simplify management
33. `az role assignment create --assignee --role --scope` = CLI syntax for assignment
34. `New-AzRoleAssignment -SignInName -RoleDefinitionName -Scope` = PowerShell syntax
35. **Wildcard** `*` = all operations; `*/read` = read on all resource types
36. **Check access** = `IAM → Check access → Search for user → View effective permissions`
37. Co-Admins managed via: `Subscription → IAM → Classic administrators`
38. **Reader** can view everything but change nothing — safe default role
39. Effective permissions = union of all assigned roles at all scopes (additive)
40. Locks are **inherited** from Resource Group to all Resources within

---

## 20. Step-by-Step Configuration Mind Maps 🗺️

---

### 20.1 Assign a Built-in Role

> **Portal:** `Resource/RG/Subscription → Access control (IAM) → + Add → Add role assignment`

```
Assign Built-in Role
│
├── Step 1: Navigate to Scope
│   ├── Management Group / Subscription / Resource Group / Resource
│   └── → Access control (IAM)
│
├── Step 2: + Add → Add role assignment
│
├── Step 3: Role tab
│   ├── Search or browse roles
│   ├── Categories: Privileged administrator roles / Job function roles
│   ├── Select role (e.g., Contributor, Reader, VM Contributor)
│   └── Next
│
├── Step 4: Members tab
│   ├── Assign access to: User, group, or service principal / Managed identity
│   ├── + Select members → Search for user/group/SP/MI
│   ├── Select → Next
│   └── ⚠️ Assigning to a GROUP is best practice (reduces assignment count)
│
├── Step 5: Conditions tab (optional — only for data roles)
│   └── Add condition for ABAC (Azure Storage data roles only)
│
├── Step 6: Review + assign
│
├── Required Role: Owner or User Access Administrator at the scope
│
└── ⚠️ Key Points
    ├── Assignment inherits downward (RG → all resources in it)
    ├── Max 4,000 assignments per subscription
    ├── Permissions are additive across all assignments
    └── Assignment takes effect in ~1-2 minutes (may take up to 10 min)
```

---

### 20.2 Create a Custom Role

> **Portal:** `Subscription → Access control (IAM) → + Add → Add custom role`

```
Create Custom Role
│
├── Step 1: Navigate to scope (Subscription or RG)
│   └── → Access control (IAM) → + Add → Add custom role
│
├── Step 2: Basics
│   ├── Custom role name (unique within tenant)
│   ├── Description
│   └── Baseline: Start from scratch / Clone a role / Start from JSON
│       ⚠️ "Clone a role" = fastest for exam scenarios
│
├── Step 3: Permissions
│   ├── + Add permissions
│   │   ├── Search by resource provider (e.g., Microsoft.Compute)
│   │   ├── Select specific actions (read, write, delete, */action)
│   │   └── Check individual permissions
│   │
│   └── Exclude permissions (NotActions)
│       └── Remove specific actions from the granted set
│
├── Step 4: Assignable scopes
│   ├── + Add assignable scopes
│   ├── Select Management Groups / Subscriptions / Resource Groups
│   └── ⚠️ This defines WHERE the role CAN be assigned (not who gets it)
│
├── Step 5: JSON tab
│   └── Review/edit raw JSON definition
│
├── Step 6: Review + Create
│
├── Required Role: Owner or User Access Administrator
│   (needs Microsoft.Authorization/roleDefinitions/write)
│
└── ⚠️ Key Points
    ├── Max 5,000 custom roles per tenant
    ├── AssignableScopes cannot be Resource-level (only MG, Sub, RG)
    ├── Custom role changes take up to 10 minutes to propagate
    └── Can also create via CLI: az role definition create --role-definition file.json
```

---

### 20.3 Remove a Role Assignment

> **Portal:** `Resource/RG/Subscription → Access control (IAM) → Role assignments`

```
Remove Role Assignment
│
├── Step 1: Navigate to scope where role was assigned
│   └── → Access control (IAM) → Role assignments tab
│
├── Step 2: Find the assignment
│   ├── Filter by user / group / role
│   └── Identify the correct assignment
│
├── Step 3: Check the assignment → Remove
│   └── Confirm removal
│
├── Required Role: Owner or User Access Administrator at the scope
│
└── ⚠️ Key Points
    ├── Inherited assignments CANNOT be removed at child scope
    │   → Must remove at the scope where originally assigned
    ├── Removing a role is immediate (but may take a few minutes to propagate)
    └── CLI: az role assignment delete --assignee --role --scope
```

---

### 20.4 Check Effective Access for a User

> **Portal:** `Resource/RG/Subscription → Access control (IAM) → Check access`

```
Check Effective Access
│
├── Step 1: Navigate to desired scope
│   └── → Access control (IAM) → Check access tab
│
├── Step 2: Search for the user/group/service principal/managed identity
│   └── Select the principal
│
├── Step 3: View results
│   ├── Shows all role assignments (direct + inherited)
│   ├── Shows effective permissions (union of all roles)
│   ├── Shows deny assignments (if any)
│   └── Shows scope of each assignment
│
├── Required Role: Reader (to view IAM) — any role works
│
└── ⚠️ Key Points
    ├── This shows inherited assignments from higher scopes
    ├── Effective = additive union of all assigned roles
    └── Deny assignments override everything
```

---

### 20.5 Create a Resource Lock

> **Portal:** `Resource/RG/Subscription → Settings → Locks`

```
Create Resource Lock
│
├── Step 1: Navigate to Resource / Resource Group / Subscription
│   └── → Settings → Locks
│
├── Step 2: + Add
│
├── Step 3: Configure Lock
│   ├── Lock name
│   ├── Lock type:
│   │   ├── Read-only → Cannot modify OR delete
│   │   │   ⚠️ ReadOnly on Storage = prevents listing keys (POST)
│   │   └── Delete → Can modify but CANNOT delete
│   └── Notes (optional)
│
├── Step 4: OK
│
├── Required Role: Owner or User Access Administrator
│   (needs Microsoft.Authorization/locks/write)
│
└── ⚠️ Key Points
    ├── Locks apply to ALL users regardless of role (even Owner)
    ├── Must remove lock before performing locked action
    ├── Locks INHERIT from RG to all resources inside
    ├── Lock on RG → all resources inside protected
    └── To delete a locked resource: remove lock → then delete
```

---

### 20.6 Elevate Global Admin to Azure RBAC

> **Portal:** `Microsoft Entra ID → Properties`

```
Elevate Global Admin Access
│
├── Prerequisites
│   ├── Must be Global Administrator in Entra ID
│   └── Must have MFA enabled (recommended)
│
├── Step 1: Navigate
│   └── Microsoft Entra ID → Properties
│
├── Step 2: Find "Access management for Azure resources"
│   └── Toggle to YES → Save
│
├── Step 3: Assign roles at subscription/MG level
│   └── Now has User Access Administrator at root (/) scope
│       → Can assign Owner/Contributor etc. on any subscription
│
├── Step 4: ⚠️ Disable after use
│   └── Toggle back to NO → Save
│
├── Required Role: Global Administrator (Entra ID)
│
└── ⚠️ Key Points
    ├── Grants User Access Administrator at root scope (/)
    ├── Root scope (/) = ALL management groups + subscriptions
    ├── This is a TEMPORARY emergency action
    ├── Must manually disable after completing the task
    └── Only Global Admins can do this (no other role)
```

---

### 20.7 Assign Role to Managed Identity

> **Portal:** `Target Resource → Access control (IAM) → + Add`

```
Assign Role to Managed Identity
│
├── Step 1: Enable Managed Identity (if not already)
│   ├── System-assigned: Resource → Identity → System assigned → On → Save
│   └── User-assigned: Search "Managed Identities" → + Create → Name, RG, Region → Create
│
├── Step 2: Navigate to target resource (e.g., Storage Account)
│   └── → Access control (IAM) → + Add → Add role assignment
│
├── Step 3: Select Role
│   └── e.g., Storage Blob Data Reader, Key Vault Secrets User
│
├── Step 4: Members
│   ├── Assign access to: Managed identity
│   ├── + Select members
│   ├── Filter by subscription → Select identity type
│   │   ├── System-assigned managed identity (select resource)
│   │   └── User-assigned managed identity (select MI resource)
│   └── Select → Next
│
├── Step 5: Review + assign
│
├── Required Role: Owner or User Access Administrator on target resource
│
└── ⚠️ Key Points
    ├── System-assigned MI deleted when resource is deleted
    ├── User-assigned MI persists independently
    ├── MI = service principal (no credentials to manage)
    └── Common pattern: VM MI → Storage Blob Data Reader → read blobs without keys
```

---

### 20.8 View and Manage Classic Administrators

> **Portal:** `Subscription → Access control (IAM) → Classic administrators`

```
View/Manage Classic Administrators
│
├── Step 1: Navigate to Subscription
│   └── → Access control (IAM) → Classic administrators tab
│
├── Step 2: View current classic admins
│   ├── Account Administrator (1 per subscription — billing)
│   ├── Service Administrator (1 per subscription — full access)
│   └── Co-Administrators (up to 200)
│
├── Step 3: Add Co-Administrator
│   └── + Add → Add co-administrator → Select user → Add
│
├── Step 4: Remove Co-Administrator
│   └── Select → Remove
│
├── Required Role: Service Administrator or Owner
│
└── ⚠️ Key Points
    ├── Classic admins = LEGACY (use RBAC instead)
    ├── Service Admin ≈ Owner at subscription level
    ├── Co-Admin ≈ Owner at subscription level (but cannot change Service Admin)
    ├── Account Admin = billing only (cannot manage resources by default)
    └── Max 200 Co-Administrators per subscription
```

---

## 21. RBAC Permission Evaluation Order

When a user makes a request through Azure Resource Manager, permissions are evaluated in a specific order:

### Evaluation Flow

```
User Request → ARM
│
├── Step 1: Collect ALL role assignments at ALL scopes
│   └── Subscription + RG + Resource + Management Group
│
├── Step 2: Check DENY assignments
│   ├── If deny matches → ACCESS DENIED (stop here)
│   └── If no deny match → continue
│
├── Step 3: Check ALLOW (role assignments)
│   ├── Union all Actions + DataActions from all assigned roles
│   ├── Subtract NotActions + NotDataActions from their respective roles
│   ├── If any remaining permission matches → ACCESS GRANTED
│   └── If no match → ACCESS DENIED
│
└── Result: Allow or Deny
```

### Key Evaluation Rules

| Rule | Description |
|---|---|
| **Deny beats Allow** | Deny assignments always win, even if a role explicitly grants the action |
| **Additive across roles** | All role assignments are **unioned** — permissions from multiple roles combine |
| **Scope inheritance** | Roles assigned at higher scopes apply at lower scopes automatically |
| **Most permissive wins** | If one role grants Read and another grants Write at same scope, user has both |
| **Management plane vs Data plane** | Actions/NotActions evaluated separately from DataActions/NotDataActions |

> ⚠️ **EXAM TIP:** Evaluation order = **Deny → Allow**. First check deny assignments, then check role assignments. If BOTH a deny and an allow exist for the same action, **deny wins**. If no deny and no allow → **implicit deny** (default deny).

---

## 22. Privileged Identity Management (PIM) for Azure RBAC

- **PIM** = Azure AD service for just-in-time (JIT) privileged access management
- Requires **Entra ID P2** (or EMS E5) license
- Reduces standing admin access — users **activate** roles when needed
- Supports both **Entra ID roles** and **Azure resource RBAC roles**

### Eligible vs Active Assignments

| Assignment Type | Description | Standing Access? |
|---|---|---|
| **Active** | Role is always ON — traditional assignment | ✅ Yes |
| **Eligible** | Role must be **activated** before use — JIT access | ❌ No (must activate) |
| **Time-bound Active** | Active role with an expiry date | ✅ Until expiry |
| **Time-bound Eligible** | Eligible role with an expiry date | ❌ Until expiry |

### PIM Activation Flow

```
User has Eligible Assignment
│
├── Step 1: User requests activation via PIM portal
│   └── Entra ID → Identity Governance → PIM → My roles → Activate
│
├── Step 2: Provide justification + duration
│   ├── Reason for activation (text)
│   ├── Duration (within max allowed — e.g., 1-8 hours)
│   └── Ticket number (if configured)
│
├── Step 3: Approval (if required)
│   ├── If approval workflow configured → Approver must approve
│   └── If no approval required → Auto-activated
│
├── Step 4: MFA verification (if configured)
│
├── Step 5: Role is activated for the specified duration
│   └── User now has the role permissions
│
└── Step 6: Role automatically deactivated after duration expires
```

### PIM Settings per Role

| Setting | Options | Purpose |
|---|---|---|
| **Activation max duration** | 0.5 – 24 hours | How long the role stays active |
| **Require MFA** | On / Off | Force MFA on activation |
| **Require justification** | On / Off | Require text reason |
| **Require ticket info** | On / Off | Link to change request |
| **Require approval** | On / Off | Route to approver before activation |
| **Approvers** | Select users/groups | Who can approve activation requests |
| **Assignment expiration** | Permanent / Time-bound | Auto-expire eligible/active assignments |
| **Notification** | Email to admins/assignees | Alert on assignment or activation |

### Azure Resource Roles in PIM

- PIM can manage RBAC roles at **Management Group**, **Subscription**, **Resource Group**, and **Resource** scope
- Same eligible/active model as Entra ID roles
- Activate via: `PIM → Azure resources → Select resource → My roles → Activate`

> ⚠️ **EXAM TIP:** PIM requires **P2 license**. Key concepts: **Eligible** = must activate (JIT), **Active** = always on. If a question mentions "just-in-time access" or "time-limited admin access" → answer = **PIM**. PIM applies to BOTH Entra ID roles AND Azure RBAC roles.

---

## 23. Time-Bound (Expiring) Role Assignments

- Azure now supports **start and end date** on role assignments directly (without PIM)
- Available in the portal under **Conditions** tab when creating assignments
- Assignment types: **Eligible** (PIM) or **Active** with optional expiry

### Assignment Duration Options

| Option | Behavior |
|---|---|
| **Permanent (no expiry)** | Role stays until manually removed |
| **Time-bound** | Role automatically removed at the end date/time |
| **Start date** | Role becomes effective at a specific date |
| **End date** | Role expires and is automatically removed |

### Portal Path — Time-Bound Assignment

```
Resource/RG/Subscription → Access control (IAM) → + Add → Add role assignment →
Role → Members → Settings tab →
Assignment type: Active / Eligible →
Set expiration: Date + Time →
Review + assign
```

### CLI — Time-Bound Assignment

```bash
# Create a time-bound role assignment with start and end time
az role assignment create \
  --assignee user@contoso.com \
  --role "Contributor" \
  --scope "/subscriptions/{sub-id}" \
  --start-date "2025-01-01T00:00:00Z" \
  --expiration-date "2025-06-30T23:59:59Z"
```

> ⚠️ **EXAM TIP:** Time-bound assignments can be made **without PIM** in recent Azure updates. Use for contractors, temporary project access, or audit compliance. The assignment is auto-removed at expiry — no manual intervention needed.

---

## 24. Subscription Transfer & RBAC Impact

### What Happens When a Subscription is Transferred

| Item | Behavior on Transfer |
|---|---|
| **RBAC role assignments** | ❌ **ALL deleted** — must be reassigned in new tenant |
| **Custom role definitions** | ❌ **Deleted** — must be recreated in new tenant |
| **Managed identities (system-assigned)** | ⚠️ **Disabled** — must be re-enabled and roles reassigned |
| **Managed identities (user-assigned)** | ⚠️ **Must be reassigned** in new tenant |
| **Resource locks** | ✅ **Preserved** — locks move with the subscription |
| **Resources** | ✅ **Preserved** — all resources remain intact |
| **Azure Blueprints** | ❌ **Removed** |
| **Azure Policy assignments** | ❌ **Removed** |
| **Classic administrators** | ❌ **Removed** |
| **Azure DevOps service connections** | ❌ **Broken** — must reconfigure |

### Transfer Between Management Groups (Same Tenant)

| Item | Behavior |
|---|---|
| **RBAC role assignments** | ✅ **Preserved** (same tenant) |
| **Custom roles** | ✅ **Preserved** (same tenant) |
| **Inherited roles from old MG** | ❌ **Lost** — new MG inheritance takes effect |

> ⚠️ **EXAM TIP:** Subscription transfer **between tenants** = ALL RBAC assignments deleted. Transfer **between management groups** (same tenant) = RBAC preserved but inheritance changes. This is a very commonly tested scenario. Plan for re-assignment of all roles post-transfer.

---

## 25. Guest Users (B2B) & RBAC

- **External users** (B2B guests from partner tenants) can be assigned Azure RBAC roles
- Guest users appear in your Entra ID as **guest** user type
- Same RBAC system applies — assign roles at any scope
- Guest limitations can be configured via **External Collaboration Settings**

### Guest User RBAC Considerations

| Consideration | Details |
|---|---|
| **Role assignment** | Same as internal users — any built-in or custom role |
| **Scope** | Can be assigned at any scope (MG, Sub, RG, Resource) |
| **Default permissions** | Guests have **limited** Entra ID directory access by default |
| **MFA requirement** | Can enforce MFA via Conditional Access in your tenant |
| **Access reviews** | Use Access Reviews (P2) to periodically review guest access |
| **Invitation** | Users must accept invitation before role works |
| **Cross-tenant data access** | Governed by cross-tenant access settings + RBAC |

### Portal Path — Invite Guest & Assign Role

```
Step 1: Invite guest user
  Microsoft Entra ID → Users → + New user → Invite external user →
  Email, Display name → Invite

Step 2: Assign RBAC role to guest
  Resource/RG/Subscription → Access control (IAM) →
  + Add → Add role assignment →
  Select role → Members → Select guest user → Review + assign
```

### External Collaboration Settings

```
Microsoft Entra ID → External Identities → External collaboration settings →
Guest user access restrictions:
  ├── Guest users have same access as members
  ├── Guest users have limited access to properties/memberships (default)
  └── Guest user access is restricted to properties of own directory objects
```

> ⚠️ **EXAM TIP:** Guest users CAN be assigned Azure RBAC roles the same way as internal users. Guest default = limited Entra ID directory access, but full Azure resource access via RBAC. Use **Conditional Access** to require MFA for guests.

---

## 26. Role Assignments via ARM Templates & Bicep

### ARM Template — Role Assignment

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "type": "string",
      "metadata": { "description": "Object ID of the user, group, or service principal" }
    },
    "roleDefinitionId": {
      "type": "string",
      "metadata": { "description": "Role Definition ID (GUID)" }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Authorization/roleAssignments",
      "apiVersion": "2022-04-01",
      "name": "[guid(resourceGroup().id, parameters('principalId'), parameters('roleDefinitionId'))]",
      "properties": {
        "roleDefinitionId": "[subscriptionResourceId('Microsoft.Authorization/roleDefinitions', parameters('roleDefinitionId'))]",
        "principalId": "[parameters('principalId')]",
        "principalType": "ServicePrincipal"
      }
    }
  ]
}
```

### Bicep — Role Assignment

```bicep
@description('Object ID of the principal')
param principalId string

@description('Role Definition ID GUID - e.g., b24988ac-6180-42a0-ab88-20f7382dd24c for Contributor')
param roleDefinitionId string

resource roleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(resourceGroup().id, principalId, roleDefinitionId)
  properties: {
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', roleDefinitionId)
    principalId: principalId
    principalType: 'ServicePrincipal'
  }
}
```

### Common Built-in Role Definition GUIDs

| Role | GUID |
|---|---|
| **Owner** | `8e3af657-a8ff-443c-a75c-2fe8c4bcb635` |
| **Contributor** | `b24988ac-6180-42a0-ab88-20f7382dd24c` |
| **Reader** | `acdd72a7-3385-48ef-bd42-f606fba81ae7` |
| **User Access Administrator** | `18d7d88d-d35e-4fb5-a5c3-7773c20a72d9` |
| **Storage Blob Data Contributor** | `ba92f5b4-2d11-453d-a403-e96b0029c9fe` |
| **Storage Blob Data Reader** | `2a2b9908-6ea1-4ae2-8e65-a410df84e7d1` |
| **Key Vault Secrets User** | `4633458b-17de-408a-b874-0445c86b69e6` |

### Key Points for ARM/Bicep

- **Name** must be a GUID — use `guid()` function for deterministic generation
- **principalType** — specify `User`, `Group`, `ServicePrincipal`, or `ForeignGroup` to avoid lookup delays
- **Idempotent** — deploying same assignment twice = no error (no duplicate created)
- Scope = resource group by default for RG-level deployments; specify `scope` for resource-level

> ⚠️ **EXAM TIP:** ARM/Bicep role assignments require a **GUID as the name** — use `guid()` for reproducible deployments. Always specify `principalType` to avoid deployment delays from Entra ID lookups.

---

## 27. Management Group RBAC — Key Details

### How RBAC Works with Management Groups

```
Root Management Group (/)
├── MG: IT Department
│   ├── Sub: Production
│   └── Sub: Development
└── MG: Finance
    └── Sub: Finance-Prod
```

| Feature | Details |
|---|---|
| **Root Management Group** | Every tenant has one; cannot be deleted or moved |
| **Role at root MG** | Applies to ALL MGs, subscriptions, RGs, and resources below |
| **Max depth** | 6 levels of MGs below root (total 7 including root) |
| **Max MGs per tenant** | 10,000 |
| **Default access at root** | Global Admin can elevate (Section 12), no other user has default access |
| **Role assignments at MG** | Max **500** per management group |

### Important Behaviors

- Roles assigned at a MG apply to **all** child subscriptions and their resources
- You **cannot** assign a role at a child scope that grants **more** than what's available at the parent (RBAC is additive, but deny still blocks)
- Moving a subscription between MGs = **loses inherited roles** from the old MG, **gains inherited roles** from the new MG
- Direct role assignments on the subscription itself are **preserved** during MG moves

### Portal Path — Assign Role at Management Group

```
Management Groups → Select MG → Access control (IAM) →
+ Add → Add role assignment → Select role → Select members → Assign
```

> ⚠️ **EXAM TIP:** Management Group role assignments are limited to **500** per MG. Roles at root MG apply to the **entire** tenant. Moving subscriptions between MGs changes **inherited** roles but keeps **direct** assignments. Max MG depth = **6 levels** below root.

---

## 28. Troubleshooting RBAC — Common Issues

| Symptom | Likely Cause | Fix |
|---|---|---|
| User can't see a resource | No Reader (or above) role at that scope | Assign Reader at RG/Subscription |
| User sees resource but cannot modify | Has Reader but not Contributor | Assign Contributor or specific role |
| User can't assign roles to others | Has Contributor, not Owner | Assign Owner or User Access Administrator |
| Role assigned but user sees "Forbidden" | Deny assignment overriding allow | Check deny assignments (Blueprints?) |
| Custom role not visible for assignment | Not in AssignableScopes | Update AssignableScopes to include target scope |
| Role assignment not taking effect | Propagation delay (up to 10 min) | Wait; or clear browser cache / re-login |
| Error: "No more role assignments can be created" | Hit 4,000 per subscription limit | Use groups to reduce assignment count |
| Storage data access denied | Has Storage Account Contributor but not Data role | Assign Storage Blob Data Contributor/Reader |
| Guest user can't access resources | Invitation not accepted / no RBAC role | Ensure invite is accepted + role assigned |
| Lock prevents modification | ReadOnly lock on resource/RG | Remove lock first, modify, re-apply lock |

### Diagnostic Steps

```
1. Check access:
   Resource → IAM → Check access → Search user → View effective permissions

2. Check deny assignments:
   Resource → IAM → Deny assignments tab

3. Check Activity Log:
   Azure Monitor → Activity Log → Filter: Authorization category

4. Check resource locks:
   Resource → Settings → Locks

5. Check group membership:
   Entra ID → Users → Select user → Groups → Verify group memberships
```

> ⚠️ **EXAM TIP:** Most "access denied" issues in exam scenarios are: (1) wrong role (Contributor vs Owner), (2) wrong scope, (3) storage management vs data plane confusion, or (4) resource lock blocking operations.

---

## 29. Additional Quick-Fire Exam Points ⚡

41. **Evaluation order**: Deny → Allow → Implicit Deny (default deny if nothing matches)
42. **PIM** = Eligible assignments, activation, time-bound, requires P2
43. **Eligible assignment** = user must activate the role; **Active** = always on
44. Subscription **transfer between tenants** = ALL RBAC assignments deleted
45. Subscription **transfer between MGs** (same tenant) = direct assignments kept, inheritance changes
46. **Guest users** (B2B) can be assigned RBAC roles the same way as internal users
47. **ARM/Bicep** role assignment name must be a **GUID** — use `guid()` function
48. Always specify **principalType** in ARM/Bicep to avoid Entra ID lookup delays
49. **Root Management Group** = cannot be deleted or moved; every tenant has exactly one
50. Max MG depth = **6 levels** below root (7 total including root)
51. Max management groups per tenant = **10,000**
52. Management Group role assignments limited to **500** per MG
53. Moving subscription between MGs: **inherited** roles change, **direct** roles preserved
54. **ReadOnly lock** on a storage account **prevents listing access keys** (key list = POST operation)
55. Storage Account Contributor + Storage Blob Data Reader = manage account settings + read blob data
56. **Conditional Access** works **alongside** RBAC — CA controls authentication, RBAC controls authorization
57. **Service Principal** = identity used by apps/automation; assign RBAC roles the same as users
58. Role assignment changes recorded in **Activity Log** under `Microsoft.Authorization` resource provider
59. Custom roles can use wildcards: `Microsoft.Compute/*` = all actions on compute resources
60. **notAction on `*/read`** with action `*` = full access except all read operations (unusual pattern)


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
