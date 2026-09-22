<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Resource Manager (ARM) & Bicep Templates — AZ-104 Revision Notes

---

## 1. What is Azure Resource Manager (ARM)?

- **Management layer** for all Azure operations — every Azure API call goes through ARM
- Provides **consistent management** via Portal, CLI, PowerShell, SDKs, REST API
- Handles **authentication, authorization, request routing** for all Azure operations
- Enables **Infrastructure as Code (IaC)** via ARM templates (JSON) and Bicep (DSL)
- Supports: **resource groups, tagging, locking, RBAC, policy** — all managed through ARM

### ARM Key Capabilities
- **Declarative deployments** — define WHAT you want, ARM figures out HOW
- **Idempotent** — deploying the same template produces the same result
- **Dependency management** — auto-resolves resource dependencies
- **Parallel deployment** — deploys independent resources simultaneously
- **Rollback** — supports rollback on failure
- **Preview (What-If)** — preview changes before deploying

---

## 2. Key Components

| Component | Description |
|---|---|
| **ARM Template** | JSON file defining resources to deploy (Infrastructure as Code) |
| **Bicep** | Domain-specific language (DSL) that compiles to ARM JSON (simpler syntax) |
| **Template Spec** | Store ARM/Bicep templates as Azure resources for sharing and versioning |
| **Deployment** | Execution of a template — creates/updates resources |
| **Resource Group** | Logical container for resources (deployment scope) |
| **Resource Provider** | Namespace for resource types (e.g., `Microsoft.Compute`, `Microsoft.Network`) |
| **Resource Type** | Specific resource (e.g., `Microsoft.Compute/virtualMachines`) |
| **API Version** | Version of the resource provider API used |

---

## 3. ARM Template Structure (JSON)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": { },
  "variables": { },
  "functions": [ ],
  "resources": [ ],
  "outputs": { }
}
```

### Template Sections

| Section | Required | Description |
|---|---|---|
| **$schema** | ✅ | URL to the JSON schema file (defines template version) |
| **contentVersion** | ✅ | Version of the template (user-defined, e.g., "1.0.0.0") |
| **parameters** | ❌ | Input values provided at deployment time |
| **variables** | ❌ | Reusable values constructed from parameters/expressions |
| **functions** | ❌ | User-defined functions for use in the template |
| **resources** | ✅ | Azure resources to deploy (the core section) |
| **outputs** | ❌ | Values returned after deployment |

> ⚠️ **EXAM TIP:** Only **$schema**, **contentVersion**, and **resources** are REQUIRED in an ARM template. Parameters, variables, functions, and outputs are optional.

---

## 4. Parameters

- Input values supplied at deployment time (runtime)
- Allow template reuse across environments (dev/staging/prod)
- Support **types**: `string`, `int`, `bool`, `object`, `array`, `secureString`, `secureObject`

### Parameter Properties

| Property | Description |
|---|---|
| **type** | Data type (required) |
| **defaultValue** | Used if no value provided at deployment |
| **allowedValues** | List of valid values (enum) |
| **minValue / maxValue** | Range for int |
| **minLength / maxLength** | Length constraints for string/array |
| **description** | Metadata description |

### Parameter Example
```json
"parameters": {
  "vmSize": {
    "type": "string",
    "defaultValue": "Standard_D2s_v5",
    "allowedValues": ["Standard_D2s_v5", "Standard_D4s_v5"],
    "metadata": { "description": "Size of the VM" }
  },
  "adminPassword": {
    "type": "secureString",
    "metadata": { "description": "Admin password" }
  }
}
```

> ⚠️ **EXAM TIP:** Use **secureString** for passwords and secrets — values are NOT logged in deployment history or outputs. Never use plain `string` for passwords.

> ⚠️ **EXAM TIP:** **allowedValues** restricts input to a defined list. If a user provides an invalid value → deployment fails at validation.

---

## 5. Variables

- Constructed values within the template — not supplied at deployment
- Simplify complex expressions
- Defined once, referenced multiple times

```json
"variables": {
  "vnetName": "[concat(parameters('prefix'), '-vnet')]",
  "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), 'default')]"
}
```

> ⚠️ **EXAM TIP:** **Parameters** = values from the USER at deployment. **Variables** = values COMPUTED inside the template from parameters or functions.

---

## 6. Functions (Built-in Template Functions)

### Commonly Tested Functions

| Category | Function | Description |
|---|---|---|
| **String** | `concat()` | Concatenate strings |
| **String** | `toLower()`, `toUpper()` | Case conversion |
| **String** | `substring()` | Extract portion of string |
| **String** | `uniqueString()` | Deterministic hash (13 chars) — unique per scope |
| **String** | `format()` | Format string with placeholders |
| **Resource** | `resourceId()` | Get the resource ID of a resource |
| **Resource** | `reference()` | Get runtime properties of a deployed resource |
| **Resource** | `resourceGroup()` | Get current RG properties (name, location, id) |
| **Resource** | `subscription()` | Get current subscription properties |
| **Array** | `length()` | Array/string length |
| **Numeric** | `add()`, `sub()`, `mul()`, `div()` | Arithmetic |
| **Logical** | `if()` | Conditional expression |
| **Logical** | `equals()` | Equality comparison |
| **Deployment** | `deployment()` | Info about current deployment |
| **Date** | `utcNow()` | Current UTC timestamp |

> ⚠️ **EXAM TIP:** `uniqueString()` generates a **deterministic 13-character hash** based on input. Same input = same output. Use `uniqueString(resourceGroup().id)` for unique names per RG.

> ⚠️ **EXAM TIP:** `resourceGroup().location` returns the location of the resource group — commonly used to deploy resources in the same region as the RG.

---

## 7. Resources Section

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2023-01-01",
    "name": "[parameters('storageAccountName')]",
    "location": "[resourceGroup().location]",
    "sku": { "name": "Standard_LRS" },
    "kind": "StorageV2",
    "properties": { }
  }
]
```

### Resource Properties

| Property | Required | Description |
|---|---|---|
| **type** | ✅ | Resource type (e.g., `Microsoft.Compute/virtualMachines`) |
| **apiVersion** | ✅ | API version (e.g., `2023-01-01`) |
| **name** | ✅ | Resource name |
| **location** | ✅ (most) | Azure region |
| **sku** | Varies | Pricing tier / performance level |
| **kind** | Varies | Resource sub-type |
| **properties** | ✅ | Resource-specific configuration |
| **dependsOn** | ❌ | Explicit dependencies |
| **tags** | ❌ | Key-value metadata pairs |
| **copy** | ❌ | Create multiple instances (loop) |
| **condition** | ❌ | Deploy only if condition is true |

---

## 8. Dependencies

### Implicit Dependencies
- ARM auto-detects when you use `reference()` or `resourceId()` to refer to another resource
- No need for explicit `dependsOn`

### Explicit Dependencies
- Use `dependsOn` array when ARM cannot infer the dependency

```json
"dependsOn": [
  "[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))]"
]
```

> ⚠️ **EXAM TIP:** Use `dependsOn` only when ARM **cannot** detect the dependency automatically. Over-using `dependsOn` creates unnecessary serialization and slows deployments.

---

## 9. Outputs

- Return values after deployment (e.g., IP addresses, resource IDs, connection strings)
- Displayed in deployment results and accessible via CLI/PowerShell

```json
"outputs": {
  "storageEndpoint": {
    "type": "string",
    "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
  }
}
```

> ⚠️ **EXAM TIP:** **Never** output `secureString` values — they would be exposed in deployment history. Use Key Vault references instead.

---

## 10. Deployment Modes

| Mode | Behavior | Risk |
|---|---|---|
| **Incremental** (default) | Adds/updates resources in template. **Leaves existing resources untouched** | Low — safe |
| **Complete** | Deletes resources NOT in the template. Only resources in template exist after deploy | ⚠️ HIGH — can delete resources |

> ⚠️ **EXAM TIP:** Default mode = **Incremental**. **Complete mode DELETES** resources in the RG that are NOT in the template. ALWAYS use Incremental unless you intentionally want to remove extra resources.

> ⚠️ **EXAM TIP:** If asked "deploy template without affecting existing resources" → **Incremental** mode. If "ensure ONLY template resources exist in RG" → **Complete** mode.

---

## 11. Deployment Scopes

| Scope | Schema | CLI Command |
|---|---|---|
| **Resource Group** | `deploymentTemplate.json` | `az deployment group create` |
| **Subscription** | `subscriptionDeploymentTemplate.json` | `az deployment sub create` |
| **Management Group** | `managementGroupDeploymentTemplate.json` | `az deployment mg create` |
| **Tenant** | `tenantDeploymentTemplate.json` | `az deployment tenant create` |

> ⚠️ **EXAM TIP:** Most deployments target a **Resource Group**. Subscription-level deployments create RGs and policies. Management Group and Tenant scopes are for governance.

---

## 12. Bicep Language

- **Domain-Specific Language (DSL)** that compiles to ARM JSON
- **Cleaner, shorter syntax** than raw JSON — no square bracket expressions
- **First-class Azure tooling** — VS Code extension with IntelliSense
- **Transpile**: Bicep → ARM JSON (using `az bicep build`)
- **Decompile**: ARM JSON → Bicep (using `az bicep decompile`)
- All ARM template capabilities are available in Bicep

### Bicep vs ARM JSON Comparison

| Feature | ARM JSON | Bicep |
|---|---|---|
| **Syntax** | Verbose JSON | Concise DSL |
| **File extension** | `.json` | `.bicep` |
| **Comments** | ❌ Not supported | ✅ `//` and `/* */` |
| **Parameters** | JSON object | `param vmSize string = 'Standard_D2s_v5'` |
| **Variables** | JSON object | `var vnetName = '${prefix}-vnet'` |
| **String interpolation** | `concat()` function | `'${variable}-suffix'` native |
| **Modules** | Linked/Nested templates | `module` keyword |
| **Dependencies** | Manual `dependsOn` | Auto-detected (symbolic references) |
| **Resource references** | `resourceId()`, `reference()` | `resource.properties.xxx` syntax |
| **Intellisense** | Limited | ✅ Full VS Code support |
| **Compiled output** | Direct JSON | Compiles to ARM JSON |

> ⚠️ **EXAM TIP:** Bicep compiles to **ARM JSON** — there is no separate Bicep runtime. Azure still deploys ARM JSON. Bicep is just a **better authoring experience**.

---

## 13. Bicep Syntax

### Parameter
```bicep
@description('VM size')
@allowed(['Standard_D2s_v5', 'Standard_D4s_v5'])
param vmSize string = 'Standard_D2s_v5'

@secure()
param adminPassword string
```

### Variable
```bicep
var vnetName = '${prefix}-vnet'
var location = resourceGroup().location
```

### Resource
```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageName
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
  properties: {}
}
```

### Output
```bicep
output blobEndpoint string = storageAccount.properties.primaryEndpoints.blob
```

### Module
```bicep
module networkModule './modules/network.bicep' = {
  name: 'networkDeploy'
  params: {
    vnetName: vnetName
    location: location
  }
}
```

### Conditional Deployment
```bicep
resource publicIp 'Microsoft.Network/publicIPAddresses@2023-04-01' = if (deployPublicIp) {
  name: 'myPublicIp'
  location: location
  ...
}
```

### Loop (Copy)
```bicep
resource storageAccounts 'Microsoft.Storage/storageAccounts@2023-01-01' = [for i in range(0, 3): {
  name: '${prefix}storage${i}'
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}]
```

> ⚠️ **EXAM TIP:** Bicep uses `@secure()` decorator for secrets (equivalent to `secureString` in JSON). `@allowed()` = `allowedValues`. `@description()` = metadata description.

> ⚠️ **EXAM TIP:** Bicep **auto-detects dependencies** via symbolic references. You rarely need explicit `dependsOn`. If you reference `storageAccount.id`, Bicep knows the storage must exist first.

---

## 14. ARM Template Features

### 14.1 Linked Templates
- Reference **external templates** via URL
- Child templates stored in a **Storage Account** (with SAS token) or public URL
- Parent template orchestrates deployment of linked templates

### 14.2 Nested Templates
- Define child templates **inline** within the parent template
- No external URL needed
- Expression evaluation scope: `inner` or `outer`

| Feature | Linked Templates | Nested Templates |
|---|---|---|
| **Location** | External URL (Storage Account) | Inline in parent |
| **Reusability** | ✅ High (shared across templates) | ❌ Low (embedded) |
| **Parameters** | Passed from parent | Inner/Outer scope |
| **Access** | Needs URL + SAS token | No external access |

### 14.3 Template Specs
- Store ARM templates as **Azure resources** (`Microsoft.Resources/templateSpecs`)
- Version-controlled — multiple versions per spec
- Share via RBAC — no storage account needed
- Deploy via portal, CLI, or PowerShell

#### Portal Path — Template Specs
```
Home → Template specs → + Create template spec →
Name, Version, Upload template → Create
```

> ⚠️ **EXAM TIP:** **Template Specs** are the recommended way to share and version ARM templates — stored as Azure resources, secured with RBAC, no SAS tokens needed.

### 14.4 What-If Deployment
- Preview changes **before** deploying
- Shows: Create, Delete, Modify, No Change, Ignore
- Use for validation WITHOUT making actual changes

#### CLI — What-If
```bash
az deployment group what-if -g <rg> --template-file template.json --parameters @params.json
```

> ⚠️ **EXAM TIP:** **What-If** shows predicted changes without deploying. Use to validate Complete mode deployments to see what would be DELETED.

### 14.5 Copy (Loops)
- Create **multiple instances** of a resource using `copy` element
- Supports: resources, properties, variables, outputs
- `copyIndex()` function returns current iteration index

```json
"copy": {
  "name": "storageCopy",
  "count": 3,
  "mode": "serial"  // or "parallel" (default)
}
```

| Mode | Description |
|---|---|
| **parallel** (default) | All copies deploy simultaneously |
| **serial** | Deploy one at a time (with optional batchSize) |

### 14.6 Conditions
- Deploy a resource only when a condition is true
- `"condition": "[equals(parameters('env'), 'production')]"`

---

## 15. Deployment Commands

### Azure CLI

| Action | Command |
|---|---|
| Deploy to RG | `az deployment group create -g <rg> --template-file main.bicep --parameters @params.json` |
| Deploy to RG (ARM JSON) | `az deployment group create -g <rg> --template-file template.json` |
| Deploy to subscription | `az deployment sub create -l <location> --template-file sub.bicep` |
| What-If | `az deployment group what-if -g <rg> --template-file main.bicep` |
| Validate template | `az deployment group validate -g <rg> --template-file main.bicep` |
| List deployments | `az deployment group list -g <rg> -o table` |
| Show deployment | `az deployment group show -g <rg> -n <deployment-name>` |
| Delete deployment history | `az deployment group delete -g <rg> -n <deployment-name>` |
| Export template (from RG) | `az group export -g <rg>` |
| Export template (from resource) | `az resource show ... --include-response-body` |
| Build Bicep → JSON | `az bicep build --file main.bicep` |
| Decompile JSON → Bicep | `az bicep decompile --file template.json` |
| Install Bicep CLI | `az bicep install` |
| Upgrade Bicep CLI | `az bicep upgrade` |
| Create Template Spec | `az ts create -g <rg> -n <name> --version 1.0 --template-file template.json` |
| Deploy Template Spec | `az ts show -g <rg> -n <name> --version 1.0 --query id` then `az deployment group create -g <rg> --template-spec <id>` |

### PowerShell

| Action | Command |
|---|---|
| Deploy to RG | `New-AzResourceGroupDeployment -ResourceGroupName <rg> -TemplateFile main.bicep` |
| Deploy with parameters | `New-AzResourceGroupDeployment -ResourceGroupName <rg> -TemplateFile main.bicep -TemplateParameterFile params.json` |
| What-If | `New-AzResourceGroupDeployment -ResourceGroupName <rg> -TemplateFile main.bicep -WhatIf` |
| Validate | `Test-AzResourceGroupDeployment -ResourceGroupName <rg> -TemplateFile main.bicep` |
| Export RG template | `Export-AzResourceGroup -ResourceGroupName <rg>` |

---

## 16. Export Templates

### From Existing Resources
- Export current resource configuration as ARM JSON template
- Useful for reverse engineering existing resources into IaC

| Method | Description |
|---|---|
| **From Resource Group** | Exports all resources in RG |
| **From individual resource** | Exports single resource config |
| **From deployment history** | Exports the exact template used for a deployment |

#### Portal Path — Export Template
```
Resource Group → Automation → Export template → Download / Deploy

Resource → Automation → Export template → Download

Resource Group → Deployments → Select deployment → Template → Download
```

> ⚠️ **EXAM TIP:** **Exported templates** are a starting point — they often need cleanup. Not all resource properties export cleanly. Always review before redeploying.

> ⚠️ **EXAM TIP:** Export from **deployment history** gives the exact template used. Export from **resource** gives the current state (which may have drifted from original template).

---

## 17. Security & RBAC

### Deployment Roles

| Role | Permissions |
|---|---|
| **Contributor** | Deploy templates, manage resources (cannot assign roles) |
| **Owner** | Full access including role assignments |
| **Reader** | View deployments and templates |
| **Template Spec Contributor** | Create and manage Template Specs |
| **Template Spec Reader** | Read and deploy Template Specs |

### Security Best Practices

| Practice | Description |
|---|---|
| **secureString / @secure()** | Protect passwords — not logged or visible |
| **Key Vault references** | Reference secrets from Key Vault in parameters |
| **Template Specs** | Share templates securely via RBAC |
| **What-If** | Validate before deploying |
| **Azure Policy** | Enforce standards on deployed resources |
| **Resource Locks** | Prevent accidental deletion/modification |

### Key Vault Reference in Parameters
```json
{
  "adminPassword": {
    "reference": {
      "keyVault": {
        "id": "/subscriptions/.../resourceGroups/.../providers/Microsoft.KeyVault/vaults/myVault"
      },
      "secretName": "vmAdminPassword"
    }
  }
}
```

> ⚠️ **EXAM TIP:** Key Vault references in ARM parameters require the Key Vault to have **"Azure Resource Manager for template deployment"** enabled (`enabledForTemplateDeployment: true`).

---

## 18. Monitoring & Deployment History

### Deployment History
- Each RG stores the **last 800 deployments** (auto-managed)
- Old deployments auto-deleted when approaching limit
- View deployment status, duration, errors, template used

#### Portal Path — Deployment History
```
Resource Group → Settings → Deployments →
View list of deployments → Click for details →
Overview / Inputs / Outputs / Template
```

### Deployment States

| State | Description |
|---|---|
| **Succeeded** | All resources deployed successfully |
| **Failed** | One or more resources failed |
| **Canceled** | Deployment was canceled |
| **Running** | Deployment in progress |

> ⚠️ **EXAM TIP:** Max **800 deployments** per RG in history. Azure auto-deletes oldest when near limit. You can also manually delete deployment history entries.

---

## 19. Pricing Key Points

| Component | Cost |
|---|---|
| **ARM service** | **Free** — no charge for ARM itself |
| **Template deployments** | **Free** — no charge for deploying templates |
| **Template Specs** | **Free** — no storage charges |
| **Resources deployed** | Charged per resource pricing |
| **Bicep** | **Free** — open-source tooling |

> ⚠️ **EXAM TIP:** ARM, Bicep, and Template Specs are all **FREE**. You only pay for the **resources** that are deployed.

---

## 20. ARM vs Bicep vs Terraform Comparison

| Feature | ARM JSON | Bicep | Terraform |
|---|---|---|---|
| **Language** | JSON | DSL (Bicep) | HCL |
| **Provider** | Azure only | Azure only | Multi-cloud |
| **State file** | No (Azure tracks state) | No (Azure tracks state) | Yes (required) |
| **Compilation** | Direct | Compiles to ARM JSON | Direct to API |
| **Modules** | Linked/Nested templates | `module` keyword | `module` blocks |
| **Comments** | ❌ | ✅ | ✅ |
| **IntelliSense** | Limited | ✅ Full | ✅ |
| **Managed by** | Microsoft | Microsoft | HashiCorp |

> ⚠️ **EXAM TIP:** Both ARM and Bicep **do NOT use state files** — Azure itself is the state. Terraform uses a separate state file. This is a key distinction.

---

## 21. Limitations & Constraints

| Constraint | Limit |
|---|---|
| Template file size | **4 MB** (after expansion) |
| Parameter file size | **4 MB** |
| Parameters per template | **256** |
| Variables per template | **256** |
| Resources per template | **800** |
| Outputs per template | **64** |
| Deployment history per RG | **800** |
| Template expression length | **24,576 characters** |
| Copy iterations (loop) | **800** |
| Nested template depth | **5** levels |
| Deployment name max length | **64 characters** |

> ⚠️ **EXAM TIP:** Max template size = **4 MB**. Max resources per template = **800**. Max parameters/variables = **256** each. Max outputs = **64**.

---

## 22. Quick-Fire Exam Points ⚡

1. ARM = **management layer** for ALL Azure operations — every request goes through ARM
2. ARM templates = **JSON** files for declarative Infrastructure as Code
3. Bicep = **DSL** that compiles to ARM JSON — cleaner syntax, same capabilities
4. Required template sections: **$schema**, **contentVersion**, **resources** (3 required)
5. **Parameters** = values from user at deployment. **Variables** = computed inside template
6. Use **secureString** / `@secure()` for passwords — never logged in deployment history
7. **allowedValues** restricts parameter input to a defined list
8. `uniqueString()` = deterministic **13-char hash** — same input = same output
9. `resourceGroup().location` = deploy resource in same region as RG
10. `reference()` creates **implicit dependency** — no need for explicit `dependsOn`
11. Deployment mode default = **Incremental** (adds/updates, leaves existing untouched)
12. **Complete mode DELETES** resources not in the template — use with caution
13. **What-If** = preview changes before deploying (no actual changes made)
14. Deployment scopes: **Resource Group** (most common), Subscription, Management Group, Tenant
15. **Linked templates** = external URL (need SAS token). **Nested** = inline in parent
16. **Template Specs** = store templates as Azure resources with RBAC and versioning (recommended)
17. **Copy/loop**: max **800** iterations. Mode: parallel (default) or serial
18. Bicep auto-detects **dependencies** via symbolic references — rarely need `dependsOn`
19. Bicep uses `@secure()`, `@allowed()`, `@description()` **decorators** for parameters
20. `az bicep build` = compile Bicep to JSON. `az bicep decompile` = JSON to Bicep
21. Key Vault references require Key Vault `enabledForTemplateDeployment: true`
22. Exported templates need **cleanup** — not always production-ready
23. Export from **deployment history** = original template. Export from **resource** = current state
24. Max template size = **4 MB**. Max resources = **800**. Max parameters/variables = **256**
25. Deployment history max = **800** per RG (auto-managed)
26. ARM, Bicep, and Template Specs are all **FREE** — only deployed resources cost money
27. ARM/Bicep do **NOT use state files** (unlike Terraform) — Azure tracks state
28. Deployments are **idempotent** — same template = same result
29. Bicep file extension = `.bicep`. ARM template = `.json`
30. Bicep supports **comments** (`//`, `/* */`). ARM JSON does **NOT** support comments

---

## 23. Step-by-Step Configuration Mind Maps 🗺️

---

### 23.1 Deploy ARM Template (JSON)

> **Portal:** `Home → Deploy a custom template`

```
Deploy ARM Template
│
├── Method 1: Azure Portal
│   │   Home → Deploy a custom template
│   ├── Template source:
│   │   ├── Build your own template in the editor
│   │   │   └── Paste/edit JSON → Save
│   │   ├── Load a file → Upload .json file
│   │   ├── Select a common template (quickstart)
│   │   └── Template Spec → Select existing spec
│   ├── Parameters:
│   │   ├── Subscription
│   │   ├── Resource Group: Select / Create new
│   │   └── Fill in template parameters
│   ├── Review + Create
│   └── RBAC: Contributor on target RG
│
├── Method 2: Azure CLI
│   │   az deployment group create \
│   │     -g <rg> \
│   │     --template-file template.json \
│   │     --parameters @parameters.json
│   ├── Inline parameters:
│   │   --parameters vmSize="Standard_D2s_v5" adminUser="azureuser"
│   ├── ⚠️ Default mode = Incremental
│   └── For Complete mode: --mode Complete
│       ⚠️ DELETES resources not in template
│
├── Method 3: PowerShell
│   └── New-AzResourceGroupDeployment \
│         -ResourceGroupName <rg> \
│         -TemplateFile template.json \
│         -TemplateParameterFile parameters.json
│
├── Validate Before Deploy
│   ├── CLI: az deployment group validate -g <rg> --template-file template.json
│   ├── What-If: az deployment group what-if -g <rg> --template-file template.json
│   └── PS: Test-AzResourceGroupDeployment -ResourceGroupName <rg> -TemplateFile template.json
│
└── ⚠️ Notes
    ├── Template max size: 4 MB
    ├── Deployment is idempotent — rerunning updates (not duplicates)
    ├── Failed deployments stay in history
    └── Check errors: RG → Deployments → Failed deployment → Error details
```

---

### 23.2 Deploy Bicep Template

> **CLI:** `az deployment group create --template-file main.bicep`

```
Deploy Bicep Template
│
├── Prerequisites
│   ├── Azure CLI installed (Bicep integrated since CLI 2.20+)
│   ├── OR install Bicep: az bicep install
│   ├── VS Code + Bicep extension (recommended for authoring)
│   └── RBAC: Contributor on target scope
│
├── Step 1: Create Bicep File
│   │   main.bicep:
│   ├── param location string = resourceGroup().location
│   ├── param storageName string
│   ├── @secure()
│   │   param adminPassword string
│   ├── var uniqueName = '${storageName}${uniqueString(resourceGroup().id)}'
│   ├── resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
│   │     name: uniqueName
│   │     location: location
│   │     sku: { name: 'Standard_LRS' }
│   │     kind: 'StorageV2'
│   │   }
│   └── output endpoint string = storageAccount.properties.primaryEndpoints.blob
│
├── Step 2: (Optional) Build to JSON for Review
│   └── az bicep build --file main.bicep
│       → Generates main.json (ARM JSON equivalent)
│
├── Step 3: Validate
│   └── az deployment group validate -g <rg> --template-file main.bicep
│
├── Step 4: What-If Preview
│   └── az deployment group what-if -g <rg> --template-file main.bicep \
│         --parameters storageName="mystore"
│       ⚠️ Shows Create/Modify/Delete/NoChange — review before deploying
│
├── Step 5: Deploy
│   ├── CLI:
│   │   az deployment group create -g <rg> \
│   │     --template-file main.bicep \
│   │     --parameters storageName="mystore"
│   ├── PowerShell:
│   │   New-AzResourceGroupDeployment -ResourceGroupName <rg> \
│   │     -TemplateFile main.bicep -storageName "mystore"
│   └── Portal: Build your own template → paste compiled JSON
│
└── ⚠️ Notes
    ├── Bicep compiles to ARM JSON — Azure never sees Bicep directly
    ├── Dependencies auto-resolved from symbolic references
    ├── Decompile existing JSON: az bicep decompile --file template.json
    └── Bicep supports modules for reusability
```

---

### 23.3 Create and Deploy Template Specs

> **Portal:** `Home → Template specs → + Create template spec`

```
Create and Deploy Template Specs
│
├── Step 1: Create Template Spec
│   │
│   ├── Portal:
│   │   │   Home → Template specs → + Create template spec
│   │   ├── Subscription, Resource Group
│   │   ├── Name: e.g., "vm-template"
│   │   ├── Version: e.g., "1.0"
│   │   ├── Description
│   │   ├── Upload template: Browse → Select .json file
│   │   │   ⚠️ Must be valid ARM JSON (not Bicep directly)
│   │   │   Tip: Build Bicep to JSON first: az bicep build --file main.bicep
│   │   └── Create
│   │
│   └── CLI:
│       az ts create -g <rg> -n vm-template \
│         --version "1.0" --template-file template.json \
│         --description "Standard VM deployment"
│
├── Step 2: Share Via RBAC
│   │   Portal: Template spec → Access Control (IAM) → + Add role
│   ├── Template Spec Reader: Can deploy but not modify
│   ├── Template Spec Contributor: Can create/modify specs
│   └── ⚠️ No SAS tokens needed — uses Azure RBAC
│
├── Step 3: Deploy Template Spec
│   │
│   ├── Portal:
│   │   Template specs → Select spec → Select version →
│   │   Deploy → Fill parameters → Deploy
│   │
│   ├── CLI:
│   │   # Get template spec ID
│   │   id=$(az ts show -g <rg> -n vm-template --version "1.0" --query id -o tsv)
│   │   # Deploy
│   │   az deployment group create -g <target-rg> --template-spec $id
│   │
│   └── PowerShell:
│       $id = (Get-AzTemplateSpec -ResourceGroupName <rg> -Name vm-template -Version "1.0").Versions.Id
│       New-AzResourceGroupDeployment -TemplateSpecId $id -ResourceGroupName <target-rg>
│
├── Step 4: Version Management
│   ├── Create new version: az ts create -g <rg> -n vm-template --version "2.0" --template-file updated.json
│   ├── List versions: az ts list -g <rg>
│   └── Users can deploy specific version
│
└── ⚠️ Notes
    ├── Template Specs = FREE (no storage charges)
    ├── Stored as Azure resources, discoverable via portal
    ├── Version-controlled — can have multiple versions
    └── Recommended over linked templates + SAS tokens
```

---

### 23.4 Export Template from Existing Resources

> **Portal:** `Resource / Resource Group → Automation → Export template`

```
Export Template from Existing Resources
│
├── Method 1: Export from Resource Group
│   │   Portal: Resource Group → Automation → Export template
│   ├── Shows all resources in the RG as ARM JSON
│   ├── Select resources to include/exclude
│   ├── Options:
│   │   ├── Include parameters: ✅
│   │   └── Include variables: Choose
│   ├── Download (ZIP with template.json + parameters.json)
│   └── Deploy (deploy directly from export)
│       ⚠️ Exported template may need cleanup — not all properties export cleanly
│
├── Method 2: Export from Single Resource
│   │   Portal: Resource → Automation → Export template
│   ├── Shows the current state of the resource as ARM JSON
│   └── Download or Add to library
│       ⚠️ Current state may differ from original deployment template
│
├── Method 3: Export from Deployment History
│   │   Portal: Resource Group → Settings → Deployments →
│   │   Select deployment → Template
│   ├── Shows the EXACT template used for that deployment
│   ├── Download
│   └── ⚠️ Best source — exact template + parameters used
│
├── Method 4: CLI
│   ├── Export RG: az group export -g <rg> > exported.json
│   ├── Export resource: az resource show --ids <resource-id>
│   └── Export deployment: az deployment group export -g <rg> -n <deployment-name>
│
├── Method 5: Decompile to Bicep
│   └── az bicep decompile --file exported.json
│       ⚠️ Produces .bicep file — may need manual fixes
│
└── ⚠️ Notes
    ├── Not all properties export correctly (some read-only, some omitted)
    ├── Always review and test exported templates before reusing
    ├── Deployment history export = most reliable
    └── Decompiled Bicep may require manual cleanup
```

---

### 23.5 Use What-If for Deployment Validation

> **CLI:** `az deployment group what-if`

```
What-If Deployment Validation
│
├── Purpose: Preview changes WITHOUT actually deploying
│   ├── Shows what will be: Created, Modified, Deleted, NoChange
│   └── Essential for validating Complete mode deployments
│
├── CLI:
│   az deployment group what-if \
│     -g <rg> \
│     --template-file main.bicep \
│     --parameters @params.json
│
├── PowerShell:
│   New-AzResourceGroupDeployment \
│     -ResourceGroupName <rg> \
│     -TemplateFile main.bicep \
│     -WhatIf
│
├── Output Shows:
│   ├── 🟢 Create: New resources to be created
│   ├── 🟡 Modify: Properties that will change
│   ├── 🔴 Delete: Resources that will be removed (Complete mode)
│   ├── ⚪ NoChange: Resources unchanged
│   └── 🔵 Ignore: Not tracked by template
│
├── Key Scenarios:
│   ├── Before first deployment → see what will be created
│   ├── Before update → see what changes
│   ├── Before Complete mode → see what will be DELETED
│   │   ⚠️ Critical — always What-If before Complete mode
│   └── CI/CD pipeline validation step
│
└── ⚠️ Notes
    ├── What-If does NOT make any changes
    ├── Some resource types may show false positives
    ├── Use --mode Complete to preview Complete mode behavior
    └── Combine with validate: az deployment group validate
```

---

### 23.6 Deploy with Key Vault Secret Reference

> **Template:** Parameter file references Key Vault secret

```
Deploy with Key Vault Reference
│
├── Prerequisites
│   ├── Azure Key Vault exists
│   ├── Secret stored in Key Vault (e.g., "vmAdminPassword")
│   ├── Key Vault has template deployment enabled:
│   │   ⚠️ enabledForTemplateDeployment = true (REQUIRED)
│   │   Portal: Key Vault → Settings → Access configuration →
│   │     Azure Resource Manager for template deployment: ✅
│   ├── Deploying user has access to read secrets
│   └── RBAC: Contributor on RG + Key Vault Secrets User on vault
│
├── Step 1: Template (main.json / main.bicep)
│   ├── Define parameter as secureString:
│   │   JSON: "adminPassword": { "type": "secureString" }
│   │   Bicep: @secure() param adminPassword string
│   └── Reference in resource configuration
│
├── Step 2: Parameter File (parameters.json)
│   ├── {
│   │   "adminPassword": {
│   │     "reference": {
│   │       "keyVault": {
│   │         "id": "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<vault>"
│   │       },
│   │       "secretName": "vmAdminPassword"
│   │     }
│   │   }
│   │ }
│   └── ⚠️ Key Vault reference is ONLY in the parameter FILE — not in the template
│
├── Step 3: Deploy
│   └── az deployment group create -g <rg> \
│         --template-file main.bicep \
│         --parameters @parameters.json
│       ⚠️ Secret never exposed in deployment logs or history
│
└── ⚠️ Notes
    ├── enabledForTemplateDeployment MUST be true on KV
    ├── Secret is retrieved at deployment time (not stored in template)
    ├── Deployer needs read access to Key Vault secrets
    └── Most secure way to pass secrets in ARM/Bicep deployments
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
