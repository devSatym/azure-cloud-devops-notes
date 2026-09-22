<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Policy — AZ-104 Revision Notes

---

## 1. What is Azure Policy?

- **Governance service** to enforce organizational standards and assess compliance at scale
- Evaluates Azure resources against **policy rules** → marks compliant or non-compliant
- Enforces rules on **resource properties** (not user permissions — that's RBAC)
- Works at **creation time** (prevent) and **existing resources** (audit/remediate)
- Evaluates ARM resource properties — **not data plane** (some exceptions: Key Vault, Kubernetes)
- Built on Azure Resource Manager (ARM)
- **Free** for Azure resources policy evaluation

> ⚠️ **EXAM TIP:** Azure Policy ≠ RBAC. **Policy** = what resources can look like (compliance). **RBAC** = who can do what (access). They are complementary. Policy can DENY a resource creation even if the user has Owner role.

---

## 2. Key Components

| Component | Description |
|---|---|
| **Policy Definition** | Individual rule (JSON) defining what to evaluate and what effect to apply |
| **Initiative Definition** | Group of related policy definitions (policy set) |
| **Assignment** | Binding a policy/initiative to a scope (MG / Sub / RG / Resource) |
| **Parameters** | Variables in a policy to make it reusable |
| **Scope** | Where the policy is assigned (Management Group → Subscription → Resource Group) |
| **Exclusions** | Resources or scopes excluded from the assignment |
| **Exemption** | Temporarily or permanently exempt a resource from policy (Waiver or Mitigated) |
| **Compliance** | Assessment result — compliant, non-compliant, exempt, unknown |
| **Remediation** | Fix non-compliant resources (for DeployIfNotExists / Modify effects) |

---

## 3. Policy Effects

| Effect | When Evaluated | What Happens | Creates Resources? |
|---|---|---|---|
| **Deny** | Create/Update request | Blocks the request | ❌ |
| **Audit** | Create/Update + existing | Logs as non-compliant, does NOT block | ❌ |
| **AuditIfNotExists** | After resource creation | Audits if related resource doesn't exist | ❌ |
| **DeployIfNotExists (DINE)** | After resource creation | Deploys related resource if missing | ✅ (via remediation task) |
| **Modify** | Create/Update | Adds/changes/removes tags or properties | ❌ (modifies in-place) |
| **Append** | Create/Update | Adds properties to resource (legacy — use Modify) | ❌ |
| **Disabled** | Never | Policy is not evaluated | ❌ |
| **DenyAction** | Delete/Action | Blocks specific actions (e.g., prevent deletion) | ❌ |
| **Manual** | N/A | User manually attests compliance | ❌ |

### Effect Evaluation Order
```
Disabled → Append → Modify → Deny → DenyAction → Audit → AuditIfNotExists → DeployIfNotExists
```

> ⚠️ **EXAM TIP:** **Deny** = blocks resource creation/update. **Audit** = allows but marks non-compliant. **DeployIfNotExists** = auto-deploy missing resources (requires **remediation task** + **Managed Identity**). **Modify** = change tags/properties during create/update. Evaluation order matters — **Deny** is checked BEFORE Audit.

---

## 4. Policy Definition Structure (JSON)

```json
{
  "properties": {
    "displayName": "Allowed locations",
    "policyType": "BuiltIn",
    "mode": "Indexed",
    "description": "Restrict allowed locations for resources",
    "parameters": {
      "allowedLocations": {
        "type": "Array",
        "metadata": { "displayName": "Allowed locations" }
      }
    },
    "policyRule": {
      "if": {
        "not": {
          "field": "location",
          "in": "[parameters('allowedLocations')]"
        }
      },
      "then": {
        "effect": "deny"
      }
    }
  }
}
```

### Policy Modes

| Mode | Evaluates | Example |
|---|---|---|
| **All** | All resource types including resource groups and tags | Tag policies |
| **Indexed** | Only resource types that support tags and location | Location restriction policies |
| **Resource Provider modes** | Specific providers (e.g., `Microsoft.Kubernetes.Data`, `Microsoft.KeyVault.Data`) | K8s, Key Vault data plane |

> ⚠️ **EXAM TIP:** **Mode: All** = evaluates resource groups too (needed for tag policies on RGs). **Mode: Indexed** = skips resource types that don't support tags/location. For tag enforcement on RGs, use **All** mode.

---

## 5. Built-in Policies — Most Exam-Relevant

| Policy Name | Effect | Purpose |
|---|---|---|
| **Allowed locations** | Deny | Restrict which Azure regions resources can be created in |
| **Allowed resource types** | Deny | Restrict which resource types can be created |
| **Not allowed resource types** | Deny | Block specific resource types |
| **Allowed virtual machine size SKUs** | Deny | Restrict VM sizes |
| **Require a tag and its value on resources** | Deny | Enforce mandatory tags |
| **Inherit a tag from the resource group** | Modify | Auto-apply RG's tag to child resources |
| **Audit VMs that do not use managed disks** | Audit | Flag VMs using unmanaged disks |
| **Deploy Log Analytics agent for Windows VMs** | DINE | Auto-deploy agent on VMs |
| **Configure Azure Backup on VMs** | DINE | Auto-enable backup |
| **Storage accounts should restrict network access** | Audit | Flag storage without network rules |
| **Key Vault should use RBAC** | Audit | Flag Key Vaults not using RBAC model |
| **Audit Windows VMs without minimum password age** | AuditIfNotExists | Guest config audit |

> ⚠️ **EXAM TIP:** **"Allowed locations"** is NOT automatically assigned — you must assign it. Know the difference between "Allowed resource types" (whitelist) and "Not allowed resource types" (blacklist).

---

## 6. Initiatives (Policy Sets)

- **Group of related policy definitions** assigned together
- Simplifies management — assign one initiative instead of many individual policies
- Can include **built-in and custom** policies
- **Parameters** can be shared across policies in the initiative
- Initiatives assigned at same scopes as individual policies

### Built-in Initiatives (Exam-Relevant)

| Initiative | Purpose |
|---|---|
| **Azure Security Benchmark** | Comprehensive security baseline |
| **Enable Azure Monitor for VMs** | Deploy AMA + DCR for VM monitoring |
| **Enable Azure Monitor for VMSS** | Deploy AMA for VMSS |
| **ISO 27001** | ISO compliance baseline |
| **NIST SP 800-53** | NIST compliance |
| **CIS Microsoft Azure Foundations Benchmark** | CIS hardening |

### Portal Path — Create Initiative
```
Policy → Authoring → Definitions → + Initiative definition →
Name, Description, Category →
Add policy definitions (select from built-in/custom) →
Configure parameters (map initiative params to policy params) →
Review + Create
```

> ⚠️ **EXAM TIP:** Initiatives = grouping mechanism. Easier to track compliance for a **group of policies** than individual ones. Microsoft Defender for Cloud uses **Azure Security Benchmark** initiative automatically.

---

## 7. Policy Assignments

### Assigning a Policy or Initiative

| Setting | Description |
|---|---|
| **Scope** | Management Group / Subscription / Resource Group |
| **Exclusions** | Specific child scopes to exclude from the assignment |
| **Policy / Initiative** | Which policy or initiative to assign |
| **Assignment name** | Auto-populated; can customize |
| **Enforcement mode** | Enabled (enforce) / Disabled (audit-only, no enforcement) |
| **Parameters** | Values for policy parameters (e.g., allowed locations list) |
| **Remediation** | For DINE/Modify: create remediation task + managed identity |
| **Non-compliance message** | Custom message shown when resource is non-compliant |

### Portal Path — Assign Policy
```
Policy → Authoring → Assignments → + Assign policy →
Scope: Select MG/Sub/RG → Exclusions (optional) →
Select Policy definition →
Parameters: Fill in required values →
Remediation: Create remediation task (DINE/Modify) + Managed Identity →
Non-compliance messages (optional) →
Review + Create
```

### Portal Path — Assign Initiative
```
Policy → Authoring → Assignments → + Assign initiative →
(Same flow as policy assignment)
```

### Enforcement Mode

| Mode | Behavior |
|---|---|
| **Enabled** (Default) | Policy is fully enforced (Deny blocks, DINE deploys, etc.) |
| **Disabled** (DoNotEnforce) | Compliance is evaluated but effects are NOT enforced (audit-only) |

> ⚠️ **EXAM TIP:** **Enforcement mode: Disabled** = compliance shows results but Deny won't block, DINE won't deploy. Useful for testing. **Exclusions** = specific scopes skipped. Assignments **inherit downward** like RBAC.

---

## 8. Policy Evaluation

### When Does Evaluation Happen?

| Trigger | Timing |
|---|---|
| **New/updated resource** | Real-time at ARM request (Deny blocks immediately) |
| **New policy assignment** | Within ~30 minutes for existing resources |
| **Updated policy definition** | Within ~30 minutes |
| **Standard compliance cycle** | Every **24 hours** (full re-evaluation) |
| **On-demand scan** | Triggered manually via REST API or CLI |

### On-Demand Evaluation

```bash
# Trigger on-demand evaluation (CLI)
az policy state trigger-scan --resource-group <RG>

# Trigger at subscription level
az policy state trigger-scan
```

```powershell
# PowerShell
Start-AzPolicyComplianceScan -ResourceGroupName <RG>

# Subscription-level
Start-AzPolicyComplianceScan
```

> ⚠️ **EXAM TIP:** Existing resources are evaluated within **~30 minutes** of assignment. Full compliance re-scan = every **24 hours**. **Deny** effect works in **real-time** at creation. On-demand scan is async.

---

## 9. Remediation

- Applies to **DeployIfNotExists (DINE)** and **Modify** effects only
- Fixes existing non-compliant resources to make them compliant
- Requires a **Managed Identity** (system or user-assigned) with permissions to deploy/modify
- Remediation can run on existing resources or be auto-triggered for new resources
- **Remediation tasks** track progress and can be re-run

### Remediation Details

| Feature | Description |
|---|---|
| **Managed Identity** | Auto-created (system) or selected (user); needs permissions at target scope |
| **Permissions** | MI gets roles defined in the policy definition's `roleDefinitionIds` |
| **Re-evaluate compliance** | Option to re-evaluate before remediation |
| **Parallel deployments** | Control concurrency (default: parallel) |
| **Failure threshold** | Stop remediation if failure % exceeds threshold |
| **Scope** | Can remediate subset of resources or all |

### Portal Path — Create Remediation Task
```
Policy → Compliance → Select non-compliant policy →
Create Remediation Task →
Select scope → Managed Identity (System/User-assigned) → Remediate
```

### Portal Path — View Remediation Tasks
```
Policy → Remediation → Remediation tasks → View status and progress
```

> ⚠️ **EXAM TIP:** **DINE and Modify** are the only effects that support remediation. The Managed Identity gets roles from the policy definition automatically. If MI doesn't have right permissions → remediation fails. Remediation runs on **existing** non-compliant resources.

---

## 10. Policy Exemptions

- **Exempt** a resource or scope from policy evaluation
- Two types:

| Type | Description | Duration |
|---|---|---|
| **Waiver** | Resource doesn't need to comply (accepted risk) | Optional expiry date |
| **Mitigated** | Compliance met through alternative means | Optional expiry date |

- Exemptions can have an **expiry date** — auto-expire when date passes
- Exempt resources show as **"Exempt"** in compliance (not compliant or non-compliant)

### Portal Path — Create Exemption
```
Policy → Compliance → Select policy → Select resource →
Create Exemption → Category (Waiver / Mitigated) →
Name, Description, Expiry date (optional) → Create
```

> ⚠️ **EXAM TIP:** Exemption ≠ Exclusion. **Exclusion** = scope excluded at assignment time. **Exemption** = resource exempted after assignment (more granular). Exemptions can **expire**.

---

## 11. Tagging Policies — Common Scenarios

| Scenario | Policy | Effect |
|---|---|---|
| **Require a tag on resources** | "Require a tag on resources" | Deny |
| **Require a tag with specific value** | "Require a tag and its value on resources" | Deny |
| **Inherit tag from RG** | "Inherit a tag from the resource group" | Modify |
| **Inherit tag from subscription** | "Inherit a tag from the subscription" | Modify |
| **Add/replace tag on resources** | "Add a tag to resources" | Modify |
| **Append tag if missing** | "Append a tag and its value to resources" | Append (legacy) |
| **Require tag on RGs** | "Require a tag on resource groups" | Deny |

### Tag Policy — Key Points
- Use **Mode: All** for tag policies affecting resource groups
- **Inherit** policies use **Modify** effect (need remediation task for existing)
- Tags are **NOT inherited** from RG to resources by default — need policy for this

> ⚠️ **EXAM TIP:** Tags are **NOT automatically inherited** from RG to resources. Use "Inherit a tag from the resource group" policy (Modify effect). To enforce tags on RGs themselves, use "Require a tag on resource groups" with **Mode: All**.

---

## 12. Azure Policy vs RBAC vs Resource Locks

| Feature | Azure Policy | Azure RBAC | Resource Locks |
|---|---|---|---|
| **Purpose** | Enforce resource configurations | Control who can do what | Prevent modification/deletion |
| **Scope** | MG → Sub → RG | MG → Sub → RG → Resource | Sub → RG → Resource |
| **Target** | Resources (properties) | Users (actions) | Resources (operations) |
| **Example** | "VMs must use managed disks" | "User X can create VMs" | "Cannot delete RG" |
| **Can block Owner?** | ✅ Yes (Deny effect) | N/A (permits actions) | ✅ Yes (applies to all) |
| **Compliance view** | ✅ Yes | ❌ No | ❌ No |
| **Remediation** | ✅ (DINE/Modify) | ❌ No | ❌ No |
| **Cost** | Free (Azure resources) | Free | Free |

> ⚠️ **EXAM TIP:** Policy **Deny** can stop even an **Owner** from creating a non-compliant resource. RBAC can't enforce resource configuration. Resource locks can't enforce compliance. All three are free and complementary.

---

## 13. Guest Configuration (Machine Configuration)

- Audit or enforce settings **inside VMs** (guest OS level)
- Audits: password policies, installed software, OS settings, certificates, etc.
- Requires **Azure Policy Guest Configuration extension** on VMs
- Requires **System-assigned Managed Identity** on VMs
- Works on Azure VMs, Arc-enabled servers, and VMSS

### Guest Configuration Modes

| Mode | Description | Effect |
|---|---|---|
| **Audit** | Report on OS settings (read-only) | AuditIfNotExists |
| **Apply and monitor** | Enforce OS settings and report | DeployIfNotExists (DINE) |
| **Apply and autocorrect** | Enforce and continuously correct drift | DINE |

### Prerequisites
- VM extension: `AzurePolicyforWindows` / `AzurePolicyforLinux`
- System-assigned Managed Identity on VM
- Outbound connectivity (port 443)

### Portal Path — Assign Guest Configuration Policy
```
Policy → Definitions → Category: Guest Configuration →
Select policy (e.g., "Audit Windows VMs with passwords less than 14 chars") →
Assign → Select scope → Parameters → Remediation (MI) → Create
```

> ⚠️ **EXAM TIP:** Guest Config = audits **inside the VM** (OS settings). Requires **extension + system MI** on the VM. Regular Azure Policy only evaluates ARM properties. Guest Config uses AuditIfNotExists / DINE effects.

---

## 14. Security & RBAC for Policy

### Required Roles

| Action | Required Role |
|---|---|
| View policies and compliance | **Policy Reader** (or Reader) |
| Create/edit policy definitions | **Policy Contributor** (or Contributor/Owner) |
| Assign policies | **Policy Contributor** (needs write on scope) |
| Create initiatives | **Policy Contributor** |
| Create remediation tasks | Contributor on scope + managed identity permissions |
| Create exemptions | **Policy Contributor** (or specific exemption permission) |
| View Activity Log (policy events) | Reader |

### Built-in Policy Roles

| Role | Permissions |
|---|---|
| **Policy Reader** | Read policy definitions, assignments, compliance data |
| **Policy Contributor** | Read + create/edit/delete policy definitions, assignments, exemptions |
| **Resource Policy Contributor** | Create/modify policies + assign + create remediation tasks |

> ⚠️ **EXAM TIP:** **Resource Policy Contributor** is the recommended role for policy management. Includes policy + remediation permissions. **Contributor** role at subscription level also has policy permissions.

---

## 15. Monitoring & Compliance

### Compliance Dashboard

| Metric | Description |
|---|---|
| **Overall compliance** | % of resources compliant across all assigned policies |
| **Per-policy compliance** | % compliant for each policy/initiative |
| **Non-compliant resources** | List of resources failing each policy |
| **Compliance state** | Compliant / Non-compliant / Exempt / Unknown / Not started |

### Portal Path — View Compliance
```
Policy → Compliance → View overall compliance %
→ Select policy/initiative → View non-compliant resources
→ Select resource → View compliance details
```

### Policy Events in Activity Log
```
Azure Monitor → Activity Log → Filter: Operation = "Microsoft.Authorization/policyAssignments/write"
→ See policy assignment events
```

### Policy State Change Alerts
```
Azure Monitor → Alerts → + Create → Alert Rule →
Signal: "Policy State Change" →
Configure condition → Action Group → Create
```

### Export Compliance Data
```
Policy → Compliance → Export to CSV
OR
Use Azure Resource Graph: PolicyResources table
```

### Azure Resource Graph Queries
```kusto
// Count non-compliant resources
PolicyResources
| where type == "microsoft.policyinsights/policystates"
| where properties.complianceState == "NonCompliant"
| summarize count() by tostring(properties.policyDefinitionName)
```

> ⚠️ **EXAM TIP:** Compliance evaluation = every **24 hours** automatic cycle. On-demand = `az policy state trigger-scan`. Use **Azure Resource Graph** to query policy compliance at scale.

---

## 16. CLI & PowerShell Commands

### Azure CLI

```bash
# List all policy definitions
az policy definition list --output table

# Show specific policy definition
az policy definition show --name <policy-name>

# Create custom policy definition
az policy definition create --name "custom-policy" \
  --display-name "Custom Policy" \
  --description "My custom policy" \
  --rules policy-rules.json \
  --params policy-params.json \
  --mode Indexed

# Delete policy definition
az policy definition delete --name "custom-policy"

# Assign a policy
az policy assignment create \
  --name "restrict-locations" \
  --policy "e56962a6-4747-49cd-b67b-bf8b01975c4c" \
  --scope "/subscriptions/{sub-id}" \
  --params '{"listOfAllowedLocations": {"value": ["eastus", "westus"]}}'

# Assign with enforcement disabled (audit-only)
az policy assignment create \
  --name "audit-only-test" \
  --policy <policy-id> \
  --scope "/subscriptions/{sub-id}" \
  --enforcement-mode DoNotEnforce

# List policy assignments
az policy assignment list --scope "/subscriptions/{sub-id}"

# Delete policy assignment
az policy assignment delete --name "restrict-locations"

# Trigger on-demand evaluation
az policy state trigger-scan --resource-group <RG>

# View compliance summary
az policy state summarize --subscription <sub-id>

# List non-compliant resources
az policy state list --subscription <sub-id> \
  --filter "complianceState eq 'NonCompliant'"

# Create initiative definition
az policy set-definition create --name "my-initiative" \
  --definitions initiative-defs.json \
  --params initiative-params.json

# Assign initiative
az policy assignment create --name "my-initiative-assign" \
  --policy-set-definition "my-initiative" \
  --scope "/subscriptions/{sub-id}"

# Create remediation task
az policy remediation create \
  --name "fix-noncompliant" \
  --policy-assignment "restrict-locations" \
  --resource-group <RG>

# Create exemption
az policy exemption create \
  --name "temp-exemption" \
  --policy-assignment "restrict-locations" \
  --exemption-category Waiver \
  --scope "/subscriptions/{sub-id}/resourceGroups/{rg-name}" \
  --expires-on "2026-06-30T00:00:00Z"
```

### Azure PowerShell

```powershell
# List policy definitions
Get-AzPolicyDefinition

# Create custom policy
$definition = New-AzPolicyDefinition -Name "custom-policy" `
  -DisplayName "Custom Policy" -Policy "policy-rules.json" `
  -Parameter "policy-params.json" -Mode Indexed

# Assign policy
New-AzPolicyAssignment -Name "restrict-locations" `
  -PolicyDefinition $definition `
  -Scope "/subscriptions/{sub-id}" `
  -listOfAllowedLocations @("eastus", "westus")

# Assign with DoNotEnforce
New-AzPolicyAssignment -Name "audit-only" `
  -PolicyDefinition $definition `
  -Scope "/subscriptions/{sub-id}" `
  -EnforcementMode DoNotEnforce

# Remove assignment
Remove-AzPolicyAssignment -Name "restrict-locations"

# Trigger compliance scan
Start-AzPolicyComplianceScan -ResourceGroupName <RG>

# Get compliance summary
Get-AzPolicyStateSummary -SubscriptionId <sub-id>

# List non-compliant resources
Get-AzPolicyState -SubscriptionId <sub-id> `
  -Filter "complianceState eq 'NonCompliant'"

# Create remediation
Start-AzPolicyRemediation -Name "fix-it" `
  -PolicyAssignmentId <assignment-id> `
  -ResourceGroupName <RG>
```

> ⚠️ **EXAM TIP:** `--enforcement-mode DoNotEnforce` = audit-only assignment. `az policy state trigger-scan` = on-demand. Initiative = `az policy set-definition`. Know the difference between `policy definition` and `policy set-definition` (initiative).

---

## 17. Pricing Key Points

| Item | Cost |
|---|---|
| **Azure Policy (Azure resources)** | **Free** |
| **Policy compliance evaluation** | Free |
| **Built-in policies and initiatives** | Free |
| **Custom policies** | Free |
| **Guest Configuration (audit)** | Free (Azure VMs) |
| **Guest Configuration (apply/enforce)** | Charged per server/month (non-Azure / Arc) |
| **Azure Policy for Kubernetes** | Free (Azure AKS built-in), may have compute costs |
| **Regulatory compliance (Defender)** | Depends on Defender plan |

> ⚠️ **EXAM TIP:** Azure Policy for Azure resources = **completely free**. Guest Configuration **audit** = free for Azure VMs. Guest Config **enforcement** on non-Azure/Arc servers = per-server charge.

---

## 18. Limits

| Limit | Value |
|---|---|
| Policy definitions per subscription | **500** |
| Initiative definitions per subscription | **200** |
| Policy definitions per initiative | **100** |
| Parameters per policy | **20** |
| Assignments per scope | **Unlimited** (but 100 initiatives recommended) |
| Policy definition max size | 1 MB |
| Exemptions per policy assignment | **1,000** |
| Policy rules max conditions | 4,096 characters |

> ⚠️ **EXAM TIP:** Max **500 policy definitions** and **200 initiative definitions** per subscription. Max **100 policies** per initiative. Max **20 parameters** per policy.

---

## 19. Quick-Fire Exam Points ⚡

1. **Azure Policy** = governance service to enforce resource compliance; works on **ARM resource properties**
2. Policy ≠ RBAC: Policy = what resources look like; RBAC = who can do what
3. **Deny** = blocks non-compliant resource creation/update in **real-time**
4. **Audit** = allows resource but marks **non-compliant** (no blocking)
5. **DeployIfNotExists (DINE)** = auto-deploys related resource; needs **remediation task + Managed Identity**
6. **Modify** = changes properties/tags during create/update; needs **MI** for existing resources
7. **Disabled** = policy not evaluated at all
8. **DenyAction** = blocks specific actions like delete
9. Effect evaluation order: **Disabled → Append → Modify → Deny → DenyAction → Audit → AuditIfNotExists → DINE**
10. **Deny runs BEFORE Audit** — if Deny blocks, Audit never evaluates
11. Policy assigned at higher scope **inherits downward** (MG → Sub → RG)
12. **Enforcement mode: Disabled** = compliance is tracked but effects NOT enforced (test mode)
13. Policy compliance re-evaluated every **24 hours** automatically
14. New assignment → existing resources evaluated within **~30 minutes**
15. **On-demand scan**: `az policy state trigger-scan`
16. **Initiative** = group of policies; max **100 policies** per initiative
17. Max **500 policy definitions** per subscription; **200 initiatives** per subscription
18. **Exclusions** = at assignment time; **Exemptions** = after assignment (Waiver or Mitigated)
19. Exemptions can have an **expiry date**
20. **Remediation** only for DINE and Modify effects; needs Managed Identity
21. MI receives roles from `roleDefinitionIds` in the policy definition
22. **Mode: All** = evaluates resource groups (needed for tag policies on RGs)
23. **Mode: Indexed** = skips resource types without tags/location support
24. Tags are **NOT auto-inherited** from RG → use "Inherit tag" policy (Modify)
25. "Allowed locations" policy is **NOT assigned by default** — must be assigned
26. **Allowed locations** restricts resource deployment regions
27. **Allowed resource types** = whitelist; **Not allowed resource types** = blacklist
28. **Guest Configuration** = audits inside VM (OS settings); needs extension + system MI
29. **Policy Deny** can block even an **Owner** from creating non-compliant resources
30. **Resource Policy Contributor** = recommended role for policy management
31. Azure Policy for Azure resources = **completely free**
32. **Custom policy** = JSON with if/then rule + effect
33. Parameters make policies reusable across scopes
34. **Non-compliance message** = custom message shown when blocked
35. Azure Resource Graph → `PolicyResources` table for compliance queries at scale
36. Azure Security Benchmark initiative auto-assigned by Microsoft Defender for Cloud
37. `az policy set-definition` = CLI for initiatives (policy sets)
38. `--enforcement-mode DoNotEnforce` = audit-only (test before enforce)
39. Remediation tasks can be re-run and tracked in the portal
40. Policy works at **management plane (ARM)** — not data plane (with few exceptions like K8s, Key Vault)

---

## 20. Step-by-Step Configuration Mind Maps 🗺️

---

### 20.1 Assign a Built-in Policy

> **Portal:** `Policy → Authoring → Assignments → + Assign policy`

```
Assign Built-in Policy
│
├── Step 1: Scope
│   ├── Select Management Group / Subscription / Resource Group
│   └── Exclusions (optional) — exclude specific child scopes/resources
│       ⚠️ Exclusions are fixed at assignment time
│
├── Step 2: Basics
│   ├── Click "..." next to Policy definition
│   ├── Search: e.g., "Allowed locations"
│   ├── Select the built-in policy → Select
│   ├── Assignment name (auto-populated, can customize)
│   ├── Description (optional)
│   └── Policy enforcement: Enabled (default) / Disabled (audit-only)
│       ⚠️ Disabled = tests compliance without enforcing (DoNotEnforce)
│
├── Step 3: Parameters
│   └── Fill in required values (e.g., list of allowed locations)
│       ⚠️ Parameters make policies reusable
│
├── Step 4: Remediation (only for DINE / Modify effects)
│   ├── Create a remediation task: Yes/No
│   ├── Managed Identity Type: System-assigned / User-assigned
│   ├── MI Location: Select region
│   └── ⚠️ MI auto-receives roles from policy's roleDefinitionIds
│
├── Step 5: Non-compliance messages (optional)
│   └── Custom message shown when resource fails compliance
│
├── Step 6: Review + Create
│
├── Required Role: Resource Policy Contributor / Contributor / Owner
│
└── ⚠️ Key Points
    ├── Existing resources evaluated within ~30 minutes
    ├── Deny effect blocks in real-time at ARM request
    ├── Assignment inherits to all child scopes
    └── Full compliance re-scan every 24 hours
```

---

### 20.2 Create a Custom Policy Definition

> **Portal:** `Policy → Authoring → Definitions → + Policy definition`

```
Create Custom Policy Definition
│
├── Step 1: Navigate
│   └── Policy → Definitions → + Policy definition
│
├── Step 2: Basics
│   ├── Definition location: Select Subscription or Management Group
│   │   ⚠️ MG-level = usable across subscriptions in that MG
│   ├── Name
│   ├── Description
│   └── Category: Existing (Tags, Compute, etc.) or Create new
│
├── Step 3: Policy Rule (JSON)
│   ├── Mode: All / Indexed
│   │   ⚠️ All = includes RGs; Indexed = resource types with tags/location only
│   │
│   ├── If block (condition)
│   │   ├── field: resource property to evaluate
│   │   ├── Operators: equals, notEquals, contains, in, notIn, like, exists, etc.
│   │   ├── Logical: allOf (AND), anyOf (OR), not
│   │   └── value: expected value or parameter reference
│   │
│   ├── Then block (effect)
│   │   └── effect: Deny / Audit / DeployIfNotExists / Modify / AuditIfNotExists / Disabled
│   │
│   └── Parameters (optional)
│       └── Define reusable variables (type, defaultValue, allowedValues)
│
├── Step 4: Save
│
├── Required Role: Policy Contributor / Owner on definition location
│
└── ⚠️ Key Points
    ├── Max 500 custom policy definitions per subscription
    ├── Definition size max 1 MB
    ├── Test with enforcement disabled before enforcing
    └── Can also create via CLI: az policy definition create --rules file.json
```

---

### 20.3 Create and Assign an Initiative

> **Portal:** `Policy → Authoring → Definitions → + Initiative definition`

```
Create and Assign Initiative
│
├── Part A: Create Initiative Definition
│   │
│   ├── Step 1: Policy → Definitions → + Initiative definition
│   │
│   ├── Step 2: Basics
│   │   ├── Definition location (Subscription / MG)
│   │   ├── Name, Description, Category
│   │   └── Version
│   │
│   ├── Step 3: Policies
│   │   ├── + Add policy definition(s)
│   │   ├── Search and select policies (built-in + custom)
│   │   └── ⚠️ Max 100 policies per initiative
│   │
│   ├── Step 4: Initiative Parameters
│   │   └── Define params that map to individual policy params
│   │       (share parameters across policies)
│   │
│   ├── Step 5: Policy Parameters
│   │   └── Map initiative params to each policy's params
│   │
│   ├── Step 6: Review + Create
│   │
│   └── Required Role: Policy Contributor
│
├── Part B: Assign Initiative
│   │
│   ├── Step 1: Policy → Assignments → + Assign initiative
│   │
│   ├── Step 2: Scope + Exclusions
│   │
│   ├── Step 3: Select initiative definition
│   │
│   ├── Step 4: Parameters (fill in initiative-level params)
│   │
│   ├── Step 5: Remediation (for DINE/Modify policies in initiative)
│   │
│   ├── Step 6: Review + Create
│   │
│   └── Required Role: Resource Policy Contributor
│
└── ⚠️ Max 200 initiative definitions per subscription
```

---

### 20.4 Create Remediation Task for Existing Resources

> **Portal:** `Policy → Compliance → Select policy → Create Remediation Task`

```
Create Remediation Task
│   ⚠️ Only for DeployIfNotExists (DINE) and Modify effects
│
├── Step 1: Navigate
│   └── Policy → Compliance → Select non-compliant policy assignment
│
├── Step 2: Click "Create Remediation Task"
│
├── Step 3: Configure
│   ├── Policy to remediate (auto-selected)
│   ├── Scope: All resources / Selected scope
│   ├── Locations filter (optional)
│   ├── Re-evaluate compliance before remediating: Yes / No
│   └── Managed Identity
│       ├── Type: System-assigned (auto-created) / User-assigned (select existing)
│       ├── MI Location (region)
│       └── ⚠️ MI auto-gets roles from policy's roleDefinitionIds
│
├── Step 4: Remediate
│
├── Step 5: Monitor progress
│   └── Policy → Remediation → Remediation tasks → View status
│
├── Required Role: Contributor on scope + sufficient MI permissions
│
└── ⚠️ Key Points
    ├── Remediation deploys/modifies resources to make them compliant
    ├── If MI lacks permissions → remediation fails
    ├── Can remediate all or subset of non-compliant resources
    ├── Track failed tasks and re-run as needed
    └── Only works for DINE and Modify — NOT for Deny or Audit
```

---

### 20.5 Create a Policy Exemption

> **Portal:** `Policy → Compliance → Select policy → Select resource → Create Exemption`

```
Create Policy Exemption
│
├── Step 1: Navigate
│   └── Policy → Compliance → Select policy assignment →
│       Select non-compliant resource → Exempt resource
│
├── Step 2: Configure Exemption
│   ├── Name
│   ├── Description
│   ├── Category
│   │   ├── Waiver — risk is accepted, no alternative compliance
│   │   └── Mitigated — compliance met through alternative means
│   └── Expiration date (optional)
│       ⚠️ Exemption auto-expires on this date
│
├── Step 3: Create
│
├── Required Role: Resource Policy Contributor / Owner
│
└── ⚠️ Key Points
    ├── Exemption ≠ Exclusion (exclusion = at assignment, exemption = after)
    ├── Exempt resources show "Exempt" status (not Compliant or Non-compliant)
    ├── Max 1,000 exemptions per policy assignment
    ├── Can set on individual resources or scopes
    └── Waiver = accepted risk; Mitigated = alternative compliance
```

---

### 20.6 Enforce Tags Using Policy

> **Portal:** `Policy → Assignments → + Assign policy`

```
Enforce Tags — Common Scenarios
│
├── Scenario 1: Require tag on ALL resources (Deny)
│   ├── Policy: "Require a tag on resources"
│   ├── Effect: Deny
│   ├── Parameter: Tag name (e.g., "Environment")
│   ├── Mode: Indexed
│   └── ⚠️ Blocks resource creation if tag is missing
│
├── Scenario 2: Require tag on Resource Groups (Deny)
│   ├── Policy: "Require a tag on resource groups"
│   ├── Effect: Deny
│   ├── Parameter: Tag name
│   ├── Mode: All ← ⚠️ Must be "All" for RGs
│   └── ⚠️ Blocks RG creation if tag is missing
│
├── Scenario 3: Inherit tag from RG to resources (Modify)
│   ├── Policy: "Inherit a tag from the resource group"
│   ├── Effect: Modify
│   ├── Parameter: Tag name
│   ├── Remediation: Create task + MI
│   │   ⚠️ Needs remediation for EXISTING resources
│   └── New resources auto-get tag at creation
│
├── Scenario 4: Audit resources missing tag (Audit)
│   ├── Policy: "Require a tag on resources"
│   ├── Effect: Change to Audit (via parameter or custom policy)
│   └── Reports non-compliant but does NOT block
│
└── ⚠️ Key Points
    ├── Tags NOT auto-inherited from RG — needs policy
    ├── Mode: All → includes RGs; Mode: Indexed → excludes RGs
    ├── Modify (inherit) on existing resources → create remediation task
    └── Can combine: Deny tag on RG + Inherit to resources
```

---

### 20.7 Assign Guest Configuration Policy

> **Portal:** `Policy → Definitions → Category: Guest Configuration`

```
Assign Guest Configuration Policy
│
├── Prerequisites
│   ├── Target VMs need:
│   │   ├── System-assigned Managed Identity enabled
│   │   └── Guest Configuration extension installed
│   │       ├── Windows: AzurePolicyforWindows
│   │       └── Linux: AzurePolicyforLinux
│   └── Outbound connectivity on port 443
│       ⚠️ Use initiative "Deploy prerequisites..." to auto-install MI + extension
│
├── Step 1: Policy → Definitions → Filter: Category = Guest Configuration
│
├── Step 2: Select policy
│   └── e.g., "Audit Windows VMs that have passwords less than minimum age"
│
├── Step 3: Assign
│   ├── Scope: Select MG / Sub / RG
│   ├── Parameters: Configure thresholds
│   ├── Remediation: MI for DINE policies (to install extension/guest config)
│   └── Review + Create
│
├── Step 4: View compliance
│   └── Policy → Compliance → View guest config audit results
│
├── Required Role: Resource Policy Contributor + VM Contributor
│
└── ⚠️ Key Points
    ├── Guest Config audits INSIDE the VM (OS settings)
    ├── Regular policy only audits ARM properties
    ├── Audit mode = AuditIfNotExists; Enforce = DINE
    └── Use prerequisite initiative to auto-deploy extension + MI
```

---

### 20.8 View and Analyze Policy Compliance

> **Portal:** `Policy → Compliance`

```
View and Analyze Compliance
│
├── Step 1: Navigate
│   └── Policy → Compliance
│
├── Step 2: Overview
│   ├── Overall compliance % across all assignments
│   ├── Per-policy/initiative compliance %
│   └── Filter by: Scope (MG/Sub/RG), Compliance state, Assignment
│
├── Step 3: Drill down
│   ├── Select policy → View non-compliant resources
│   ├── Select resource → View compliance details (which rule failed)
│   └── View remediation tasks status
│
├── Step 4: Export
│   ├── Export to CSV from compliance blade
│   └── Or use Azure Resource Graph:
│       PolicyResources | where type == "microsoft.policyinsights/policystates"
│       | where properties.complianceState == "NonCompliant"
│
├── Step 5: Create Alert (optional)
│   └── Azure Monitor → Alerts → Signal: Policy State Change
│       → Configure condition → Action Group → Create
│
├── Required Role: Policy Reader / Reader
│
└── ⚠️ Key Points
    ├── Full re-scan every 24 hours
    ├── New assignments = ~30 minutes for existing resources
    ├── On-demand: az policy state trigger-scan
    └── Compliance states: Compliant / Non-compliant / Exempt / Unknown
```

---

## 21. Scope Inheritance & Assignment Behavior

### Inheritance Rules

| Rule | Description |
|---|---|
| **Downward inheritance** | Policy assigned at MG → applies to all child Subs, RGs, resources |
| **No upward inheritance** | Policy assigned at RG does NOT affect parent Sub or MG |
| **Multiple assignments** | A resource can be evaluated by MULTIPLE policies from different scopes |
| **Most restrictive wins** | If multiple Deny policies apply, ALL must be satisfied |
| **Exclusions** | Specific child scopes excluded at assignment time (not individual resources) |
| **Exemptions** | Individual resources or scopes exempted AFTER assignment |

### Scope Hierarchy

```
Root Management Group
  └── Management Group (MG)
        └── Subscription (Sub)
              └── Resource Group (RG)
                    └── Resource
```

- Policy assigned at **Root MG** → enforced across **entire Azure tenant**
- Each subscription belongs to exactly **one Management Group**
- MGs can be nested up to **6 levels deep** (excluding root)
- Policy definition **location** determines where it can be assigned (MG-level = across subs in that MG)

### Important Behavior

- When **multiple policies** of a resource conflict:
  - **All Deny policies** must pass — if ANY Deny blocks, resource is blocked
  - **Audit** effects are additive — each audit generates its own compliance record
  - **DINE/Modify** effects: if multiple apply, only **one remediation** per resource property
- **Moved resources**: when a resource is moved to a new RG/Sub, it gets re-evaluated against new scope policies
- **Deleted assignments**: compliance data is retained for **~24 hours** after deletion, then removed

> ⚠️ **EXAM TIP:** Policy assignments **inherit downward only**. A resource can be evaluated by MULTIPLE policies. If ANY Deny policy fails, the resource creation is blocked. Moving a resource triggers re-evaluation against the new scope's policies.

---

## 22. Policy Conditions & Operators

### Condition Operators

| Operator | Type | Description |
|---|---|---|
| **equals** / **notEquals** | String/Number | Exact match comparison |
| **like** / **notLike** | String | Wildcard pattern matching (`*`, `?`) |
| **match** / **notMatch** | String | Pattern matching (`.` = any char, `#` = digit) |
| **matchInsensitively** / **notMatchInsensitively** | String | Case-insensitive match |
| **contains** / **notContains** | String/Array | Substring or array member check |
| **in** / **notIn** | Array | Value exists in array |
| **containsKey** / **notContainsKey** | Object | Key exists in object |
| **less** / **lessOrEquals** | Number/Date | Numeric or date comparison |
| **greater** / **greaterOrEquals** | Number/Date | Numeric or date comparison |
| **exists** | Boolean | Checks if a field exists (`true`/`false`) |

### Logical Operators

```json
{
  "allOf": [ ... ],    // AND — ALL conditions must be true
  "anyOf": [ ... ],    // OR — ANY condition can be true
  "not": { ... }       // NOT — negates the condition
}
```

### Field Function

```json
{
  "field": "Microsoft.Compute/virtualMachines/storageProfile.osDisk.osType",
  "equals": "Windows"
}
```

- **field()** = evaluates **ARM resource properties**
- Aliases map ARM properties to policy-friendly names (e.g., `Microsoft.Compute/virtualMachines/imageOffer`)
- Use `az provider show --namespace Microsoft.Compute --expand "resourceTypes/aliases"` to list aliases

### Count Expressions

```json
{
  "count": {
    "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
    "where": {
      "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].access",
      "equals": "Allow"
    }
  },
  "greater": 0
}
```

- **Field count** = count array members matching a condition
- **Value count** = count items in a parameter array matching a condition
- Used for complex scenarios like "deny NSGs with inbound rules allowing all traffic"

### Value Function

```json
{
  "value": "[concat('tags[', parameters('tagName'), ']')]",
  "exists": "false"
}
```

- **value()** = evaluates expressions, template functions, and parameters
- Can combine with **template functions**: `concat()`, `if()`, `substring()`, `resourceGroup()`, etc.

> ⚠️ **EXAM TIP:** **field()** evaluates resource properties. **value()** evaluates expressions and parameters. **Aliases** map ARM paths to usable names. **Count** expressions evaluate array members. Use `allOf` for AND, `anyOf` for OR logic. `exists: true/false` checks if a property is defined.

---

## 23. Azure Policy for Kubernetes

### Overview

| Feature | Description |
|---|---|
| **What** | Extend Azure Policy to Kubernetes clusters (AKS & Arc-enabled) |
| **How** | Uses **Azure Policy Add-on for AKS** (built on Gatekeeper/OPA) |
| **Mode** | `Microsoft.Kubernetes.Data` (resource provider mode) |
| **Scope** | Evaluates pods, containers, namespaces, and other K8s resources |
| **Cost** | Free for AKS built-in policies |

### Key Policies for Kubernetes

| Policy | Effect | Purpose |
|---|---|---|
| Do not allow privileged containers | Deny/Audit | Block containers running as privileged |
| Enforce HTTPS ingress | Deny/Audit | Require HTTPS on ingress controllers |
| Enforce internal load balancers | Deny | Block public load balancers |
| Enforce resource limits (CPU/Memory) | Deny/Audit | Require resource limits on containers |
| Allowed container images (registries) | Deny | Restrict container image sources |
| Do not allow container privilege escalation | Deny/Audit | Block `allowPrivilegeEscalation: true` |
| Enforce labels on pods/namespaces | Deny/Audit | Require specific labels |

### Enable Azure Policy Add-on for AKS

```bash
# Enable on existing AKS cluster
az aks enable-addons --addons azure-policy --name <cluster> --resource-group <rg>

# Verify add-on is running
kubectl get pods -n kube-system | grep azure-policy

# Check gatekeeper pods
kubectl get pods -n gatekeeper-system
```

### How It Works

```
Azure Policy Assignment
       ↓
Azure Policy Add-on (syncs policies to cluster every ~15 minutes)
       ↓
Gatekeeper (OPA — Open Policy Agent)
       ↓
Admission Controller (validates/mutates requests)
       ↓
Kubernetes API Server
```

- Policies sync to cluster as **constraint templates** and **constraints**
- Evaluation happens at **admission time** (when pods/resources are created)
- Existing non-compliant resources are **audited** (not auto-deleted)
- Compliance results appear in **Azure Policy compliance dashboard**

> ⚠️ **EXAM TIP:** Azure Policy for Kubernetes uses **Gatekeeper (OPA)** under the hood. Policies sync every **~15 minutes**. Mode is `Microsoft.Kubernetes.Data`. The add-on must be **explicitly enabled** on AKS clusters. Existing non-compliant pods are only audited, not deleted.

---

## 24. Regulatory Compliance & Microsoft Defender for Cloud

### Integration with Defender for Cloud

| Feature | Description |
|---|---|
| **Security Policy** | Azure Policy initiative assigned via Defender for Cloud |
| **Azure Security Benchmark** | Default initiative auto-assigned when Defender is enabled |
| **Regulatory compliance** | Map policies to compliance frameworks (ISO, NIST, PCI-DSS, CIS, SOC) |
| **Recommendations** | Defender surfaces policy non-compliance as security recommendations |
| **Secure Score** | Non-compliant policies reduce the overall secure score |

### Regulatory Compliance Dashboard

```
Defender for Cloud → Regulatory compliance →
Select standard (e.g., ISO 27001, NIST 800-53) →
View compliance controls → Drill into assessments →
See Azure Policy compliance results for each control
```

### Key Compliance Initiatives

| Standard | Initiative | Purpose |
|---|---|---|
| **Azure Security Benchmark (ASB)** | Auto-assigned by Defender | Microsoft's recommended security baseline |
| **CIS Microsoft Azure Foundations** | Assign manually | CIS hardening benchmark |
| **ISO 27001:2013** | Assign manually | Information security management |
| **NIST SP 800-53 Rev 5** | Assign manually | US government security controls |
| **PCI DSS v4** | Assign manually | Payment card industry data security |
| **SOC 2 Type 2** | Assign manually | Service organization controls |

### Adding Compliance Standards

```
Defender for Cloud → Environment settings → Select subscription →
Security policies → Add more standards →
Select compliance standard → Assign
```

### How Policies Map to Compliance

```
Compliance Standard (e.g., ISO 27001)
  └── Control Domains (e.g., A.9 Access Control)
        └── Controls (e.g., A.9.1.2 Access to networks)
              └── Azure Policy assignments (e.g., "NSG should restrict inbound")
                    └── Compliance state per resource
```

> ⚠️ **EXAM TIP:** **Azure Security Benchmark** is auto-assigned by Defender for Cloud to all subscriptions. Other standards (ISO, NIST, CIS) must be **manually assigned**. Compliance results from Azure Policy feed into Defender's **Regulatory compliance** dashboard and affect the **Secure Score**. Defender for Cloud uses Azure Policy initiatives under the hood.

---

## 25. Policy as Code (DevOps Integration)

### Overview

- Manage Azure Policy definitions and assignments **as code** in source control
- Use CI/CD pipelines (GitHub Actions / Azure DevOps) to deploy policies
- Benefits: version control, peer review, testing, audit trail

### Workflow

```
Developer writes/edits policy JSON
       ↓
Commit to Git repository (feature branch)
       ↓
Pull Request → Code review → Approve
       ↓
CI/CD Pipeline triggers
       ↓
Deploy policy definition to Azure (ARM/Bicep/Terraform)
       ↓
Assign policy to target scopes
       ↓
Monitor compliance
```

### Tools for Policy as Code

| Tool | Usage |
|---|---|
| **Azure DevOps Pipeline** | Deploy policies via ARM/Bicep templates |
| **GitHub Actions** | `azure/policy-compliance-scan` action |
| **Bicep** | `Microsoft.Authorization/policyDefinitions` resource |
| **Terraform** | `azurerm_policy_definition` / `azurerm_policy_assignment` |
| **ARM Templates** | Deploy policy definitions and assignments as JSON |
| **Azure CLI / PowerShell** | Script-based deployment in pipelines |

### Export Existing Policies

```bash
# Export all policy definitions from a subscription
az policy definition list --subscription <sub-id> --output json > policies.json

# Export policy assignments
az policy assignment list --scope "/subscriptions/{sub-id}" --output json > assignments.json
```

### Bicep Example — Custom Policy Definition

```bicep
resource policyDef 'Microsoft.Authorization/policyDefinitions@2021-06-01' = {
  name: 'deny-public-ip'
  properties: {
    displayName: 'Deny Public IP addresses'
    policyType: 'Custom'
    mode: 'Indexed'
    policyRule: {
      if: {
        field: 'type'
        equals: 'Microsoft.Network/publicIPAddresses'
      }
      then: {
        effect: 'deny'
      }
    }
  }
}
```

> ⚠️ **EXAM TIP:** Policy as Code enables managing policies via **source control and CI/CD**. Export policies with `az policy definition list`. Policy definitions can be deployed using **ARM templates, Bicep, or Terraform**. GitHub has a specific `azure/policy-compliance-scan` action.

---

## 26. Additional Exam Scenarios & Tricky Points ⚡

### Scenario-Based Questions

| Scenario | Correct Approach |
|---|---|
| **Restrict VM creation to specific regions** | Assign \"Allowed locations\" policy with Deny effect |
| **See which VMs use unmanaged disks (no blocking)** | Assign \"Audit VMs that do not use managed disks\" (Audit effect) |
| **Auto-enable backup on new VMs** | Assign \"Configure Azure Backup on VMs\" (DINE) + remediation task |
| **Enforce tags on resources created in a RG** | \"Require a tag on resources\" (Deny) at RG scope |
| **Auto-copy tags from RG to resources** | \"Inherit a tag from the resource group\" (Modify) + remediation for existing |
| **Block specific VM sizes** | \"Allowed virtual machine size SKUs\" (Deny) |
| **Test a policy without enforcing** | Assign with **enforcement mode: Disabled (DoNotEnforce)** |
| **Temporarily allow a non-compliant resource** | Create an **Exemption** (Waiver or Mitigated) with expiry date |
| **Audit OS settings inside Windows VMs** | Guest Configuration policy (AuditIfNotExists) + extension + MI |
| **Ensure storage accounts deny public access** | Assign \"Storage accounts should restrict network access\" (Audit or Modify) |
| **Prevent resource deletion** | Use **DenyAction** effect or **Resource Locks** (CanNotDelete) |
| **Deploy Log Analytics agent automatically** | DINE policy + remediation task + Managed Identity |
| **Policy for ALL subscriptions in an org** | Assign at **Root Management Group** scope |

### Common Mistakes & Tricky Points

1. **Policy vs Lock for preventing deletion**: **DenyAction** policy prevents deletion based on conditions; **Resource Locks** prevent deletion unconditionally. DenyAction is newer and more flexible.
2. **Modify vs Append**: **Modify** is the modern approach (supports tags, properties, MI-backed). **Append** is legacy — only adds properties, cannot modify or remove. Exam may test this difference.
3. **DINE without remediation task**: DINE only works on **new resources** automatically. For **existing** non-compliant resources, you MUST create a remediation task manually.
4. **Enforcement mode Disabled vs Disabled effect**: **Enforcement mode: Disabled** = policy evaluates but doesn't enforce (all effects become audit-like). **Effect: Disabled** = policy is not evaluated at all.
5. **Initiative parameter mapping**: Initiative parameters must be explicitly mapped to each policy's parameters. Unmapped parameters use default values or must be set at assignment time.
6. **Policy definition location**: A policy defined at **subscription level** cannot be assigned to other subscriptions. Define at **MG level** for cross-subscription use.
7. **Managed Identity for Modify**: The **Modify** effect uses a Managed Identity only for **remediation** of existing resources. For new resources, Modify applies inline without MI.
8. **Multiple policy assignments**: If 2 Deny policies both target the same resource property with DIFFERENT allowed values, and no value satisfies BOTH → **resource creation is impossible**.
9. **"Allowed locations" vs Resource Group location**: The \"Allowed locations\" policy restricts **resource** deployment regions but does **NOT** restrict RG deployment regions by default. Use \"Allowed locations for resource groups\" for that.
10. **Compliance state "Unknown"**: Appears when a resource hasn't been evaluated yet (e.g., just created, or policy just assigned). Not the same as Non-compliant.

### Comparison: Policy Exemption vs Exclusion vs Enforcement Disabled

| Feature | Exclusion | Exemption | Enforcement Disabled |
|---|---|---|---|
| **When set** | At assignment time | After assignment | At assignment time |
| **Granularity** | Scope (RG/Sub) | Resource or scope | Entire assignment |
| **Compliance shown** | Not evaluated | Shown as "Exempt" | Evaluated but not enforced |
| **Expiry** | No (permanent until removed) | Yes (optional expiry date) | No |
| **Use case** | Skip entire child scope | Temporary/permanent exception per resource | Test/dry-run a policy |

### Comparison: DeployIfNotExists (DINE) vs Modify

| Feature | DeployIfNotExists | Modify |
|---|---|---|
| **When evaluated** | After resource creation (async) | During create/update (sync) |
| **Creates new resources** | ✅ Yes | ❌ No (modifies in-place) |
| **Example** | Deploy diagnostic settings, backup policies | Add/change/remove tags, properties |
| **MI required for** | Always (new + existing remediation) | Existing resources only (remediation) |
| **Remediation for existing** | ✅ Yes (manual task) | ✅ Yes (manual task) |
| **Real-time for new** | Slightly delayed (post-deployment) | Inline (same request) |

> ⚠️ **EXAM TIP:** DINE runs **after** resource creation and can **create new related resources**. Modify runs **during** create/update and only **changes properties in-place**. Both need remediation tasks for existing resources. Know the difference between Exclusion/Exemption/Enforcement-Disabled — this is frequently tested.


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
