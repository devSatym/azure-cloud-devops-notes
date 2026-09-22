<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Key Vault — AZ-104 Revision Notes

---

## 1. What is Azure Key Vault?

- Cloud service to **securely store and manage** secrets, keys, and certificates
- Centralized secrets management — eliminates hard-coding credentials in applications
- FIPS 140-2 Level 2 (Standard) / Level 3 (Premium — HSM-backed) validated
- Access controlled via **Azure RBAC** or **Key Vault Access Policies**
- Supports **soft delete** and **purge protection** for recovery

---

## 2. Key Components / Object Types

| Object Type | Description | Examples |
|---|---|---|
| **Secrets** | Any sensitive string (max 25 KB) | Passwords, connection strings, API keys, SAS tokens |
| **Keys** | Cryptographic keys (RSA / EC) | Encryption keys, signing keys, CMK for Storage/Disk |
| **Certificates** | X.509 certificates + private key | SSL/TLS certificates, code signing certs |

### Object Identification
- Vault URL: `https://<vault-name>.vault.azure.net`
- Secret: `https://<vault-name>.vault.azure.net/secrets/<secret-name>/<version>`
- Key: `https://<vault-name>.vault.azure.net/keys/<key-name>/<version>`
- Certificate: `https://<vault-name>.vault.azure.net/certificates/<cert-name>/<version>`

> ⚠️ **EXAM TIP:** Vault name must be **globally unique**, 3–24 characters, alphanumeric + hyphens only, start with letter.

---

## 3. SKU Comparison: Standard vs Premium

| Feature | **Standard** | **Premium** |
|---|---|---|
| **Price** | Lower | Higher |
| **Secrets** | ✅ | ✅ |
| **Software-protected keys** | ✅ | ✅ |
| **HSM-protected keys** | ❌ | ✅ |
| **Certificates** | ✅ | ✅ |
| **FIPS 140-2** | Level 2 | **Level 3** |
| **Key types** | RSA 2048/3072/4096, EC | RSA 2048/3072/4096, EC, **HSM-backed** |
| **Managed HSM** | ❌ | Separate product (dedicated HSM pools) |

> ⚠️ **EXAM TIP:** **HSM-backed keys** = Premium tier ONLY. If exam asks about hardware-protected keys → Premium. Standard = software-protected only.

> ⚠️ **EXAM TIP:** You **CANNOT change** Standard → Premium or Premium → Standard after creation. Must create a new vault.

---

## 4. Permission Model: RBAC vs Access Policies

### 4.1 Azure RBAC (Recommended)

| Feature | Details |
|---|---|
| **Scope** | Management group, subscription, RG, vault, or **individual key/secret/cert** |
| **Model** | Uses Azure role-based access control |
| **Granularity** | Most granular — can scope to single secret |
| **Recommended** | ✅ Microsoft recommended |
| **Audit** | Azure AD activity logs |

### 4.2 Vault Access Policy (Legacy)

| Feature | Details |
|---|---|
| **Scope** | Vault-level ONLY — cannot scope to individual objects |
| **Model** | Per-vault policy for each identity |
| **Granularity** | All-or-nothing per object type (all keys, all secrets, all certs) |
| **Recommended** | ❌ Legacy — being replaced by RBAC |
| **Max policies** | **1,024** per vault |

### Comparison: RBAC vs Access Policy

| Feature | **Azure RBAC** | **Vault Access Policy** |
|---|---|---|
| **Permission scope** | Individual object level | Vault level only |
| **Inheritance** | ✅ Inherits from higher scopes | ❌ Per-vault only |
| **Central management** | ✅ Azure IAM | ❌ Per-vault config |
| **Condition-based access** | ✅ | ❌ |
| **Cross-vault** | ✅ Assign at RG/sub level | ❌ Must configure per vault |
| **Max** | Azure RBAC limits | 1,024 access policies per vault |

#### Portal Path — Set Permission Model
```
Key Vault → Settings → Access configuration →
Permission model: Azure role-based access control / Vault access policy → Save
```

> ⚠️ **EXAM TIP:** RBAC = **recommended**, granular to individual secrets/keys. Access Policy = legacy, vault-level only. Switching models takes effect **immediately** but existing assignments need migration.

> ⚠️ **EXAM TIP:** You can have ONLY **one permission model** active at a time — either RBAC or Access Policy, not both simultaneously.

---

## 5. RBAC Roles for Key Vault

| Role | Secrets | Keys | Certificates | Management |
|---|---|---|---|---|
| **Key Vault Administrator** | ✅ Full | ✅ Full | ✅ Full | ✅ (except purge) |
| **Key Vault Secrets Officer** | ✅ Full (CRUD) | ❌ | ❌ | ❌ |
| **Key Vault Secrets User** | ✅ Read only | ❌ | ❌ | ❌ |
| **Key Vault Crypto Officer** | ❌ | ✅ Full (CRUD) | ❌ | ❌ |
| **Key Vault Crypto User** | ❌ | ✅ Use (encrypt/decrypt/sign) | ❌ | ❌ |
| **Key Vault Certificates Officer** | ❌ | ❌ | ✅ Full (CRUD) | ❌ |
| **Key Vault Reader** | ✅ Read metadata | ✅ Read metadata | ✅ Read metadata | ✅ Read |
| **Key Vault Contributor** | ❌ No data access | ❌ No data access | ❌ No data access | ✅ Manage vault |

> ⚠️ **EXAM TIP:** **Key Vault Contributor** = manages the vault resource but has **NO data plane access** — cannot read/write secrets, keys, or certificates. Similar to Storage Account Contributor vs Blob Data Contributor.

> ⚠️ **EXAM TIP:** **Key Vault Reader** = reads metadata only (list names, properties) — **CANNOT read secret values or key material**.

> ⚠️ **EXAM TIP:** For an application to **read** secrets → assign **Key Vault Secrets User**. To **manage** secrets (create/update/delete) → assign **Key Vault Secrets Officer**.

---

## 6. Secrets

- Store any sensitive string value (max **25 KB**)
- Each secret has: name, value, content type (optional), activation/expiration dates, enabled/disabled state
- **Versioned** — each update creates a new version
- Latest version accessed without specifying version ID

### Operations

| Operation | Description |
|---|---|
| Set | Create or update (creates new version) |
| Get | Retrieve value (requires Get permission) |
| List | List secret names (not values) |
| Delete | Remove (soft delete if enabled) |
| Purge | Permanently remove (requires purge protection to be off or retention expired) |
| Backup/Restore | Export/import encrypted blob |

#### Portal Path — Create Secret
```
Key Vault → Objects → Secrets → + Generate/Import →
Upload options: Manual →
Name: my-secret →
Value: supersecretvalue →
Content type: text/plain (optional) →
Set activation / expiration date (optional) →
Enabled: Yes → Create
```

#### Portal Path — View/Update Secret
```
Key Vault → Secrets → Select secret →
Current version (click to see value) → Show Secret Value
```

> ⚠️ **EXAM TIP:** Secret **value** is only visible if you have **Get** permission on secrets. **List** permission only shows names, not values.

---

## 7. Keys

- Cryptographic keys for encryption, decryption, signing, verification, wrapping, unwrapping
- Key types: **RSA** (2048, 3072, 4096) and **EC** (P-256, P-384, P-521)
- Standard tier: **software-protected**. Premium tier: **HSM-protected** (hardware)
- Used as: Customer-managed keys (CMK) for Azure Storage, Disk Encryption, SQL TDE

### Key Operations

| Operation | Description |
|---|---|
| Create / Import | Generate new key or import existing |
| Encrypt / Decrypt | Encrypt/decrypt data with the key |
| Sign / Verify | Digital signatures |
| Wrap / Unwrap | Key wrapping (encrypt another key — envelope encryption) |
| Rotate | Manual or auto-rotation |

#### Portal Path — Create Key
```
Key Vault → Objects → Keys → + Generate/Import →
Options: Generate / Import →
Name: my-encryption-key →
Key type: RSA / EC / RSA-HSM / EC-HSM →
RSA key size: 2048 / 3072 / 4096 →
Set activation / expiration date (optional) →
Enabled: Yes → Create
```

> ⚠️ **EXAM TIP:** **RSA-HSM** and **EC-HSM** key types = Premium vault only. If you select HSM key type in Standard vault → error.

---

## 8. Key Rotation

- **Automatic key rotation** — Azure rotates keys on a schedule
- Configure rotation policy per key
- Supports notification via Event Grid **near-expiry** events

### Rotation Policy Options

| Setting | Description |
|---|---|
| **Time after creation** | Rotate X days/months after key creation |
| **Time before expiry** | Rotate X days/months before expiration |
| **Expiry time** | Key validity period |
| **Notification** | Event Grid event X days before expiry |

#### Portal Path — Configure Key Rotation
```
Key Vault → Keys → Select key → Rotation policy →
Rotation type: Automatically / Notify only →
Rotation time: e.g., 90 days after creation →
Notification: 30 days before expiry →
Save
```

#### CLI
```bash
az keyvault key rotation-policy update --vault-name <vault> -n <key> \
  --value '{
    "lifetimeActions": [{
      "trigger": {"timeAfterCreate": "P90D"},
      "action": {"type": "Rotate"}
    }],
    "attributes": {"expiryTime": "P1Y"}
  }'
```

> ⚠️ **EXAM TIP:** Key **auto-rotation** creates a new version of the key. Services using CMK (like Storage) that reference the key **without version** will automatically use the new version.

---

## 9. Certificates

- Store and manage X.509 certificates
- Can **generate self-signed** or request from integrated CAs (DigiCert, GlobalSign)
- Certificate = public cert + private key (stored as secret) + metadata
- Auto-renewal supported with integrated CAs

### Certificate Sources

| Source | Description |
|---|---|
| **Self-signed** | Generated by Key Vault |
| **Integrated CA** | DigiCert, GlobalSign — auto-issue and renew |
| **Non-integrated CA** | Generate CSR → submit to CA → import signed cert |
| **Import** | Upload existing PFX/PEM certificate |

#### Portal Path — Create/Import Certificate
```
Key Vault → Objects → Certificates → + Generate/Import →
Method: Generate / Import →
Certificate name: my-cert →
Type of CA: Self-signed / Certificate authority →
Subject: CN=example.com →
Validity period: 12 months →
Content type: PKCS #12 / PEM →
Lifetime action: Auto-renew at 80% / 30 days before expiry →
Create
```

### Certificate Lifecycle Actions

| Action | Trigger | Description |
|---|---|---|
| **Auto-renew** | X% of lifetime or X days before expiry | CA issues new cert |
| **Email contacts** | X% of lifetime or X days before expiry | Notify listed contacts |

#### Portal Path — Certificate Contacts (Notification)
```
Key Vault → Certificates → Certificate contacts →
+ Add → Email address → Add
```

> ⚠️ **EXAM TIP:** When a certificate is imported/created, Key Vault stores: (1) public cert as **Certificate**, (2) private key as a **Secret** (PFX/PEM), (3) metadata. Application retrieves the **secret** to get the full PFX.

---

## 10. Soft Delete & Purge Protection

### Soft Delete

| Feature | Details |
|---|---|
| **Default** | ✅ **Enabled** (mandatory since Feb 2025) |
| **Cannot disable** | Soft delete cannot be turned off once enabled |
| **Retention** | **7–90 days** (default: **90 days**) |
| **Behavior** | Deleted objects retained in deleted state for retention period |
| **Recovery** | Can recover (undelete) within retention window |
| **Purge** | Permanently delete if purge is allowed |

### Purge Protection

| Feature | Details |
|---|---|
| **Default** | ❌ Disabled (recommended to enable) |
| **Effect** | Prevents **permanent deletion** during retention period |
| **Cannot disable** | Once enabled, cannot be turned off |
| **Use case** | Required for CMK (Storage encryption, Disk encryption) |

### Soft Delete States

| State | Description |
|---|---|
| **Active** | Normal operational state |
| **Soft-deleted** | Marked for deletion, recoverable within retention period |
| **Purged** | Permanently deleted, unrecoverable |

#### Portal Path
```
Key Vault → Settings → Properties →
Soft-delete: Enabled (cannot disable) →
Purge protection: Enable (irreversible) →
Days to retain: 90 (7–90) →
Save
```

#### Recover/Purge Deleted Items
```
Key Vault → Secrets/Keys/Certificates →
Manage deleted secrets (top toolbar) →
Select item → Recover / Purge
```

> ⚠️ **EXAM TIP:** Soft delete is **MANDATORY** — cannot be disabled on any vault. Retention default = **90 days** (configurable 7–90).

> ⚠️ **EXAM TIP:** **Purge protection** = once enabled, CANNOT disable. Prevents permanent deletion during retention. **Required** for CMK scenarios (Storage, Disk encryption).

> ⚠️ **EXAM TIP:** To completely remove a soft-deleted vault: `az keyvault purge --name <vault>`. Requires **Key Vault Contributor** role + purge protection must be off.

---

## 11. Networking / Firewall

### Access Options

| Method | Description |
|---|---|
| **Public endpoint (all networks)** | Default — accessible from internet |
| **Public endpoint (selected networks)** | Restrict to specific VNets/IPs |
| **Private endpoint** | Private IP in your VNet |
| **Disable public access** | Only private endpoint access |

#### Portal Path
```
Key Vault → Settings → Networking →
Firewalls and virtual networks tab:
  Allow access from: All networks / Selected networks / Disabled →
  Virtual networks: + Add existing/new VNet →
  Firewall: Add client IP ranges →
  Exceptions: Allow trusted Microsoft services ✅ →
  Save

Private endpoint connections tab:
  + Private endpoint → Create
```

> ⚠️ **EXAM TIP:** **"Allow trusted Microsoft services to bypass this firewall"** — must be checked for Azure services (VMs with managed identity, App Service, Backup, etc.) to access Key Vault when firewall is active.

> ⚠️ **EXAM TIP:** Trusted services include: Azure VMs, App Service, Azure Backup, Storage (CMK), Disk Encryption, SQL, AKS.

---

## 12. Managed Identity Integration

- **Recommended** way for Azure resources to access Key Vault (no credentials in code)
- Managed Identity + RBAC role assignment = most secure pattern

### Pattern

| Step | Detail |
|---|---|
| 1. Enable managed identity | On the Azure resource (VM, App Service, Function, etc.) |
| 2. Grant access | Assign RBAC role (e.g., Key Vault Secrets User) at vault or secret scope |
| 3. Access in code | Use Azure SDK — identity auto-provides token, retrieves secret |

> ⚠️ **EXAM TIP:** **Managed Identity + Key Vault RBAC** = the exam's preferred pattern for secure secret access. No credentials stored anywhere.

---

## 13. Key Vault References (App Service / Functions)

- App Service and Functions can reference Key Vault secrets **directly** in app settings
- Format: `@Microsoft.KeyVault(SecretUri=https://<vault>.vault.azure.net/secrets/<name>/)`
- OR: `@Microsoft.KeyVault(VaultName=<vault>;SecretName=<name>;SecretVersion=<version>)`
- App automatically resolves the secret at runtime

### Requirements

| Requirement | Detail |
|---|---|
| Managed Identity | ✅ System or User-assigned on App Service |
| RBAC / Access Policy | Key Vault Secrets User role (or Get secret permission) |
| App Setting format | `@Microsoft.KeyVault(SecretUri=...)` |

#### Portal Path
```
App Service → Settings → Environment variables →
+ Add → Name: MY_SECRET →
Value: @Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/) →
Apply → Save → Confirm
```

> ⚠️ **EXAM TIP:** Key Vault reference in App Service requires: (1) **Managed identity** on App Service, (2) **Key Vault Secrets User** role (or Get permission), (3) correct `@Microsoft.KeyVault(...)` syntax.

> ⚠️ **EXAM TIP:** If version is omitted in SecretUri → App Service uses the **latest version** automatically.

---

## 14. Key Vault for Azure Services (CMK)

| Service | Key Vault Use | Notes |
|---|---|---|
| **Storage Account** | Customer-managed key (encryption at rest) | Requires purge protection |
| **Azure Disk Encryption** | BitLocker (Windows) / DM-Crypt (Linux) keys | Vault must be in same region |
| **Azure SQL TDE** | Transparent Data Encryption key | Server-level CMK |
| **Azure Backup** | Encryption of backup data | Can use CMK from Key Vault |
| **App Service** | TLS certificates | Import from Key Vault |

> ⚠️ **EXAM TIP:** CMK (customer-managed keys) in Key Vault requires: (1) **Purge protection** enabled, (2) **Soft delete** enabled (mandatory), (3) Vault must be in **same region** (for Disk Encryption).

---

## 15. Backup & Restore

| Feature | Details |
|---|---|
| **Backup scope** | Individual object (secret, key, or certificate) |
| **Backup format** | Encrypted blob (can only restore to same Azure AD tenant / geography) |
| **Restore target** | Same geography + same Azure AD tenant |
| **Cannot** | Backup entire vault at once (per-object only) |

#### CLI
```bash
# Backup a secret
az keyvault secret backup --vault-name <vault> -n <secret> -f secret.bak

# Restore a secret
az keyvault secret restore --vault-name <vault> -f secret.bak
```

> ⚠️ **EXAM TIP:** Key Vault backup can ONLY be restored to a vault in the **same Azure geography** (e.g., US, Europe) and **same Azure AD tenant**. Cannot restore cross-geography.

---

## 16. Logging & Monitoring

### Diagnostic Logging

| Log Category | Description |
|---|---|
| **AuditEvent** | All authenticated API requests (success + failure) |

#### Portal Path
```
Key Vault → Monitoring → Diagnostic settings →
+ Add diagnostic setting →
Log: AuditEvent ✅ →
Destination: Log Analytics workspace / Storage Account / Event Hub →
Save
```

### Key Metrics

| Metric | Description |
|---|---|
| **Service Api Hit** | Total API requests |
| **Service Api Latency** | Request processing time |
| **Service Api Result** | Success/failure count |
| **Overall Vault Saturation** | % of vault capacity used |
| **Overall Vault Availability** | Vault availability % |

#### Portal Path — Metrics
```
Key Vault → Monitoring → Metrics →
Metric: Service Api Hit / Saturation / Availability → Apply
```

### Alerts

#### Portal Path
```
Key Vault → Monitoring → Alerts → + New alert rule →
Signal: Service Api Hit (threshold), Certificate expiry, etc. →
Action group → Create
```

### Event Grid Integration

| Event | Description |
|---|---|
| **SecretNearExpiry** | Secret approaching expiration |
| **SecretExpired** | Secret has expired |
| **SecretNewVersionCreated** | New version of secret created |
| **KeyNearExpiry** | Key approaching expiration |
| **KeyExpired** | Key has expired |
| **KeyNewVersionCreated** | New version of key created |
| **CertificateNearExpiry** | Certificate approaching expiration |
| **CertificateExpired** | Certificate has expired |
| **CertificateNewVersionCreated** | New certificate version |

#### Portal Path
```
Key Vault → Events → + Event Subscription →
Event Types: SecretNearExpiry, CertificateExpired, etc. →
Endpoint: Logic App / Function / Webhook / Event Hub →
Create
```

> ⚠️ **EXAM TIP:** Use **Event Grid** events for near-expiry notifications — automate rotation or alerting. `SecretNearExpiry` fires **30 days** before expiration by default.

---

## 17. Pricing Key Points

| Component | Standard | Premium |
|---|---|---|
| **Secrets operations** | $0.03 / 10,000 transactions | $0.03 / 10,000 transactions |
| **Software keys** | $0.03 / 10,000 transactions | $0.03 / 10,000 transactions |
| **HSM keys** | ❌ N/A | $1 / key / month + $0.03 / 10K ops |
| **Certificate renewals** | $3 per renewal request | $3 per renewal request |
| **Key rotation** | Included in key operations | Included in key operations |
| **Soft-deleted vault** | No charge during retention | No charge during retention |

> ⚠️ **EXAM TIP:** HSM-backed keys incur a **per-key per-month** charge ($1/key/month) in addition to transaction charges. Software keys = only transaction charges.

---

## 18. Limitations & Constraints

| Constraint | Limit |
|---|---|
| **Vaults per subscription** | **No fixed limit** (service default varies) |
| **Secrets per vault** | **No fixed limit** (service capacity based) |
| **Secret max size** | **25 KB** |
| **Key size (RSA)** | 2048, 3072, 4096 bits |
| **Transaction rate** | **2,000 transactions / 10 seconds** per vault (Standard) |
| **HSM key transactions** | **1,000 / 10 seconds** per vault (Premium) |
| **Access policies per vault** | **1,024** |
| **Vault name length** | **3–24 characters** |
| **Vault name characters** | Alphanumeric + hyphens, start with letter |
| **Soft-delete retention** | **7–90 days** (default 90) |
| **Key rotation** | Auto-rotation available |
| **Backup restore** | Same geography + tenant only |

> ⚠️ **EXAM TIP:** Key Vault is **throttled** at **2,000 transactions per 10 seconds** per vault. If exceeded → HTTP 429 (throttling). Solution: cache secrets, use multiple vaults.

---

## 19. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| Create vault | `az keyvault create -g <rg> -n <vault> --location <region> --sku standard` |
| Create vault (Premium) | `az keyvault create -g <rg> -n <vault> --sku premium` |
| Enable purge protection | `az keyvault update -g <rg> -n <vault> --enable-purge-protection true` |
| Set access policy | `az keyvault set-policy -n <vault> --object-id <id> --secret-permissions get list set delete` |
| Create secret | `az keyvault secret set --vault-name <vault> -n <name> --value "myvalue"` |
| Get secret | `az keyvault secret show --vault-name <vault> -n <name>` |
| List secrets | `az keyvault secret list --vault-name <vault> -o table` |
| Delete secret | `az keyvault secret delete --vault-name <vault> -n <name>` |
| Recover deleted secret | `az keyvault secret recover --vault-name <vault> -n <name>` |
| Purge deleted secret | `az keyvault secret purge --vault-name <vault> -n <name>` |
| Create key | `az keyvault key create --vault-name <vault> -n <name> --kty RSA --size 2048` |
| Import certificate | `az keyvault certificate import --vault-name <vault> -n <name> -f cert.pfx --password <pw>` |
| List deleted vaults | `az keyvault list-deleted` |
| Purge deleted vault | `az keyvault purge --name <vault>` |
| Recover deleted vault | `az keyvault recover --name <vault>` |
| Backup secret | `az keyvault secret backup --vault-name <vault> -n <name> -f backup.blob` |
| Restore secret | `az keyvault secret restore --vault-name <vault> -f backup.blob` |

### PowerShell

| Action | Command |
|---|---|
| Create vault | `New-AzKeyVault -Name <vault> -ResourceGroupName <rg> -Location <region>` |
| Set secret | `Set-AzKeyVaultSecret -VaultName <vault> -Name <name> -SecretValue (ConvertTo-SecureString "value" -AsPlainText -Force)` |
| Get secret | `Get-AzKeyVaultSecret -VaultName <vault> -Name <name> -AsPlainText` |
| Get key | `Get-AzKeyVaultKey -VaultName <vault> -Name <name>` |
| Remove secret | `Remove-AzKeyVaultSecret -VaultName <vault> -Name <name>` |
| Undo removal | `Undo-AzKeyVaultSecretRemoval -VaultName <vault> -Name <name>` |
| Set access policy | `Set-AzKeyVaultAccessPolicy -VaultName <vault> -ObjectId <id> -PermissionsToSecrets get,list,set` |
| Enable purge protection | `Update-AzKeyVault -VaultName <vault> -EnablePurgeProtection` |

---

## 20. Quick-Fire Exam Points ⚡

1. Key Vault stores 3 object types: **Secrets** (strings ≤ 25 KB), **Keys** (crypto), **Certificates** (X.509)
2. Two SKUs: **Standard** (software keys) and **Premium** (HSM-backed keys, FIPS 140-2 Level 3)
3. **Cannot upgrade/downgrade** Standard ↔ Premium — must create new vault
4. **HSM-protected keys** = Premium tier ONLY
5. Two permission models: **Azure RBAC** (recommended, granular) vs **Vault Access Policy** (legacy, vault-level)
6. Only ONE permission model active at a time — cannot mix RBAC + access policies
7. **Key Vault Contributor** = manages vault resource but **NO data access** (cannot read secrets/keys)
8. **Key Vault Secrets User** = read secrets. **Key Vault Secrets Officer** = full secret CRUD
9. **Key Vault Reader** = reads metadata (names/properties) but **NOT secret values**
10. **Soft delete** = MANDATORY on all vaults, cannot disable. Default retention = **90 days** (7–90)
11. **Purge protection** = once enabled, CANNOT disable. Prevents permanent deletion during retention
12. Purge protection **required** for CMK scenarios (Storage, Disk Encryption)
13. **Managed Identity + Key Vault RBAC** = exam-preferred pattern for secure access
14. App Service Key Vault reference: `@Microsoft.KeyVault(SecretUri=...)` in app settings
15. Key Vault reference requires: Managed Identity + **Key Vault Secrets User** role
16. Certificate stored as: public cert (Certificate object) + private key (Secret object)
17. **Key auto-rotation** creates new version; services using **versionless URI** auto-pick up new version
18. Vault name: **globally unique**, **3–24 chars**, alphanumeric + hyphens, start with letter
19. Throttling: **2,000 transactions / 10 seconds** per vault → HTTP 429 if exceeded
20. Backup/restore: same **Azure geography + Azure AD tenant** only — cannot cross-geo restore
21. **Event Grid** integration: SecretNearExpiry, KeyExpired, CertificateNearExpiry events
22. Firewall: **"Allow trusted Microsoft services"** must be checked for Azure services (VMs, Backup, Storage CMK)
23. Disk Encryption: Key Vault must be in **same region** as VM
24. CMK for Storage: requires purge protection + soft delete enabled
25. Deleted vault retains its name during retention → cannot reuse name until purged/retention expires
26. `az keyvault list-deleted` shows soft-deleted vaults. `az keyvault recover` undeletes.
27. Secret **versions** — each update creates new version. Omit version in URI = latest version
28. Access policies max = **1,024** per vault
29. HSM keys = $1/key/month + transaction charges. Software keys = transaction charges only
30. Self-signed certs via Key Vault or integrated CAs (DigiCert, GlobalSign) with auto-renewal

---

## 21. Step-by-Step Configuration Mind Maps 🗺️

---

### 21.1 Create Key Vault

> **Portal:** `Home → Key vaults → + Create`

```
Create Key Vault
│
├── Portal: Home → Key vaults → + Create
│
├── Basics tab:
│   ├── Subscription: Select
│   ├── Resource Group: Select / Create new
│   ├── Key vault name: globally unique (3–24 chars, alphanumeric + hyphens)
│   │   ⚠️ Name globally unique. Start with letter. Can't reuse soft-deleted vault name
│   ├── Region: Select
│   │   ⚠️ Must be same region as VM if used for Disk Encryption
│   ├── Pricing tier: Standard / Premium
│   │   ├── Standard: Software-protected keys
│   │   └── Premium: HSM-protected keys
│   │   ⚠️ CANNOT change tier after creation
│   └── Soft-delete retention: 90 days (default, 7–90)
│       ⚠️ Soft delete is mandatory, cannot disable
│
├── Access configuration tab:
│   ├── Permission model:
│   │   ├── Azure role-based access control ✅ (recommended)
│   │   └── Vault access policy (legacy)
│   │   ⚠️ Only ONE model active at a time
│   └── Resource access checkboxes:
│       ├── Azure Virtual Machines for deployment: ✅ (VMs can retrieve certs)
│       ├── Azure Resource Manager for template deployment: ✅ (ARM can reference secrets)
│       └── Azure Disk Encryption for volume encryption: ✅ (required for VM disk encryption)
│
├── Networking tab:
│   ├── Connectivity method:
│   │   ├── Public endpoint (all networks) — default
│   │   ├── Public endpoint (selected virtual networks and IP addresses)
│   │   │   ├── + Add virtual network/subnet
│   │   │   ├── Firewall: Add IP addresses
│   │   │   └── ✅ Allow trusted Microsoft services to bypass firewall
│   │   │       ⚠️ Required for Azure services (VMs, Backup, Storage CMK)
│   │   └── Private endpoint
│   │       ├── + Create private endpoint
│   │       └── Disable public access
│   └── Next
│
├── Tags tab → optional
│
├── Review + create → Create
│
└── Post-creation:
    ├── Enable purge protection (if needed for CMK):
    │   Key Vault → Properties → Purge protection: Enable → Save
    │   ⚠️ Irreversible — cannot disable once enabled
    └── Assign RBAC roles:
        Key Vault → Access Control (IAM) → + Add role assignment
```

---

### 21.2 Store and Retrieve Secrets

> **Portal:** `Key Vault → Objects → Secrets → + Generate/Import`

```
Store and Retrieve Secrets
│
├── Prerequisites
│   ├── Key Vault exists
│   └── RBAC: Key Vault Secrets Officer (create/update/delete) or
│       Access Policy with Set + Get permissions
│
├── Create Secret (Portal)
│   │   Portal: Key Vault → Objects → Secrets → + Generate/Import
│   ├── Upload options: Manual
│   ├── Name: my-db-connection-string
│   │   ⚠️ Secret name: alphanumeric + hyphens (no spaces)
│   ├── Secret value: Server=tcp:...;Password=xxx
│   │   ⚠️ Max value size = 25 KB
│   ├── Content type: text/plain (optional, for documentation)
│   ├── Set activation date: (optional)
│   ├── Set expiration date: (optional)
│   │   ⚠️ Expiration = informational only — does NOT block access after expiry
│   │       Must use Event Grid + automation to enforce rotation
│   ├── Enabled: Yes
│   └── Create
│
├── Create Secret (CLI)
│   │   az keyvault secret set --vault-name myvault \
│   │     --name my-secret --value "supersecretvalue"
│   └── Returns: secret URI with version
│
├── Retrieve Secret (Portal)
│   │   Portal: Key Vault → Secrets → Select secret
│   ├── Current Version: click to view
│   ├── Show Secret Value button → reveals the value
│   └── RBAC required: Key Vault Secrets User (Get permission)
│
├── Retrieve Secret (CLI)
│   │   az keyvault secret show --vault-name myvault --name my-secret
│   │   # Value only:
│   │   az keyvault secret show --vault-name myvault --name my-secret --query value -o tsv
│
├── Update Secret (Creates New Version)
│   │   az keyvault secret set --vault-name myvault \
│   │     --name my-secret --value "newvalue"
│   └── ⚠️ Previous version remains accessible by version ID
│
├── Delete Secret
│   │   az keyvault secret delete --vault-name myvault --name my-secret
│   │   → Soft-deleted (recoverable within retention period)
│   │   Permanent: az keyvault secret purge --vault-name myvault --name my-secret
│   │   ⚠️ Purge blocked if purge protection is enabled
│   └── Recover: az keyvault secret recover --vault-name myvault --name my-secret
│
└── ⚠️ Notes
    ├── Expiration date = advisory only (does NOT auto-block access)
    ├── Each update = new version. URI without version = latest
    ├── List permission shows names only, NOT values
    ├── Soft-deleted secrets retained for configured retention
    └── Use Event Grid (SecretNearExpiry) to automate rotation
```

---

### 21.3 Configure Key Vault Access (RBAC Model)

> **Portal:** `Key Vault → Access Control (IAM) → + Add role assignment`

```
Configure Key Vault RBAC
│
├── Step 1: Ensure RBAC Model Active
│   │   Portal: Key Vault → Settings → Access configuration
│   ├── Permission model: Azure role-based access control
│   └── Save
│       ⚠️ Switching from Access Policy → RBAC: existing policies no longer enforced
│
├── Step 2: Assign Roles at Vault Level
│   │   Portal: Key Vault → Access Control (IAM) → + Add role assignment
│   │
│   ├── Common Scenarios:
│   │   │
│   │   ├── "App needs to READ secrets"
│   │   │   Role: Key Vault Secrets User
│   │   │   Scope: Vault or specific secret
│   │   │   Member: App's Managed Identity
│   │   │
│   │   ├── "Admin needs to MANAGE secrets"
│   │   │   Role: Key Vault Secrets Officer
│   │   │   Scope: Vault
│   │   │   Member: Admin user/group
│   │   │
│   │   ├── "App needs to USE keys (encrypt/decrypt)"
│   │   │   Role: Key Vault Crypto User
│   │   │   Scope: Vault or specific key
│   │   │   Member: App's Managed Identity
│   │   │
│   │   ├── "Admin needs to MANAGE everything"
│   │   │   Role: Key Vault Administrator
│   │   │   Scope: Vault
│   │   │   Member: Admin user/group
│   │   │
│   │   └── "DevOps needs to manage vault but NOT access data"
│   │       Role: Key Vault Contributor
│   │       Scope: Vault or RG
│   │       Member: DevOps group
│   │       ⚠️ Key Vault Contributor = NO data access
│   │
│   └── Steps:
│       ├── Role tab: Select role → Next
│       ├── Members tab: + Select members → Choose identity → Select → Next
│       ├── Conditions tab (optional): Add conditions
│       └── Review + assign
│
├── Step 3: Assign at Individual Object Level (Advanced)
│   │   Use Azure CLI for sub-vault scope:
│   │   az role assignment create --assignee <identity-id> \
│   │     --role "Key Vault Secrets User" \
│   │     --scope "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<vault>/secrets/<secret-name>"
│   └── ⚠️ Individual object scope = only with RBAC model, not access policies
│
└── ⚠️ Notes
    ├── RBAC allows scope to individual secret/key/cert
    ├── Access policies = vault-level only (cannot scope to individual objects)
    ├── Key Vault Contributor ≠ data access
    ├── Managed Identity + RBAC = best practice (no credentials)
    └── Role assignments may take up to 10 minutes to propagate
```

---

### 21.4 Configure Vault Access Policy (Legacy Model)

> **Portal:** `Key Vault → Access policies → + Add Access Policy`

```
Configure Vault Access Policy
│
├── Step 1: Ensure Access Policy Model Active
│   │   Portal: Key Vault → Settings → Access configuration
│   ├── Permission model: Vault access policy
│   └── Save
│
├── Step 2: Add Access Policy
│   │   Portal: Key Vault → Settings → Access policies → + Create
│   │
│   ├── Permissions tab:
│   │   ├── Key permissions: Get, List, Create, Delete, Encrypt, Decrypt,
│   │   │   Wrap, Unwrap, Sign, Verify, Import, Backup, Restore, Recover, Purge
│   │   ├── Secret permissions: Get, List, Set, Delete, Backup, Restore, Recover, Purge
│   │   └── Certificate permissions: Get, List, Create, Delete, Import, Update,
│   │       ManageContacts, GetIssuers, ListIssuers, SetIssuers, DeleteIssuers,
│   │       ManageIssuers, Recover, Purge, Backup, Restore
│   │   ⚠️ Select minimum required permissions (least privilege)
│   │
│   ├── Principal tab:
│   │   ├── Search for user, group, service principal, or managed identity
│   │   └── Select → Next
│   │
│   ├── Application tab (optional):
│   │   └── Compound identity (rare, skip for most scenarios)
│   │
│   └── Review + create → Create
│
├── ⚠️ Access policy takes effect immediately
│
├── Max Policies: 1,024 per vault
│
└── ⚠️ Notes
    ├── Access policies = vault-level (all secrets or none, all keys or none)
    ├── Cannot scope to individual secret/key/cert
    ├── Microsoft recommends migrating to RBAC
    ├── If switching to RBAC, existing access policies are IGNORED
    └── Both models cannot be active simultaneously
```

---

### 21.5 Configure Key Vault for Azure Disk Encryption

> **Portal:** `Key Vault → Access configuration + VM → Disks → Encryption`

```
Key Vault for Azure Disk Encryption (ADE)
│
├── Prerequisites
│   ├── Key Vault in SAME REGION as VM
│   │   ⚠️ Cross-region NOT supported for Disk Encryption
│   ├── Key Vault: Soft delete enabled (mandatory)
│   ├── Key Vault: Purge protection recommended
│   ├── Key Vault: "Azure Disk Encryption for volume encryption" = ✅
│   │   Portal: Key Vault → Access configuration → Resource access →
│   │   ✅ Azure Disk Encryption for volume encryption → Save
│   └── RBAC: VM Contributor + Key Vault permissions
│
├── Step 1: Create Key (KEK — Key Encryption Key) [Optional]
│   │   Portal: Key Vault → Keys → + Generate/Import
│   ├── Name: disk-encryption-kek
│   ├── Key type: RSA / RSA-HSM
│   ├── RSA key size: 2048+ (recommended 4096)
│   └── Create
│       ⚠️ KEK wraps the BitLocker/DM-Crypt key — adds extra security layer
│
├── Step 2: Enable Disk Encryption on VM
│   │   Portal: Virtual Machine → Disks → Additional settings →
│   │   Encryption settings:
│   ├── Disks to encrypt:
│   │   ├── OS disk only
│   │   ├── Data disks only
│   │   └── OS and data disks (recommended)
│   ├── Key Vault: Select vault (same region)
│   ├── Key (KEK): Select key (optional but recommended)
│   ├── Version: Latest / Specific version
│   └── Save
│
│   CLI:
│   az vm encryption enable -g <rg> --name <vm> \
│     --disk-encryption-keyvault <vault-name> \
│     --key-encryption-key <key-name> --volume-type All
│
├── Verify
│   │   az vm encryption show -g <rg> --name <vm>
│   └── Status: OsVolumeEncrypted: Encrypted, DataVolumesEncrypted: Encrypted
│
└── ⚠️ Exam Notes
    ├── Key Vault MUST be in same region as VM
    ├── Supported: Windows (BitLocker), Linux (DM-Crypt)
    ├── Premium SSD, Standard SSD, Standard HDD supported
    ├── Temp disks are NOT encrypted by ADE (use encryption at host)
    ├── Generation 2 VMs + ADE: supported
    └── KEK (Key Encryption Key) = optional but adds security layer
```

---

### 21.6 App Service Key Vault Reference

> **Portal:** `App Service → Environment variables → Add with @Microsoft.KeyVault(...)`

```
App Service Key Vault Reference
│
├── Prerequisites
│   ├── Key Vault exists with secrets stored
│   ├── App Service with Managed Identity enabled:
│   │   Portal: App Service → Settings → Identity →
│   │   System assigned: Status = On → Save → Yes
│   ├── Key Vault RBAC: Assign Key Vault Secrets User to App Service identity:
│   │   Key Vault → Access Control (IAM) → + Add role assignment →
│   │   Role: Key Vault Secrets User →
│   │   Members: App Service managed identity → Assign
│   └── ⚠️ If Key Vault has firewall: add App Service outbound IPs or use VNet integration
│
├── Step 1: Get Secret URI
│   │   Key Vault → Secrets → Select secret → Copy Secret Identifier
│   │   Example: https://myvault.vault.azure.net/secrets/MySecret/abc123def456
│   └── ⚠️ Omit version for auto-latest: https://myvault.vault.azure.net/secrets/MySecret/
│
├── Step 2: Create App Setting with Reference
│   │   Portal: App Service → Settings → Environment variables →
│   │   + Add
│   ├── Name: DB_CONNECTION_STRING
│   ├── Value: @Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/MySecret/)
│   │   OR: @Microsoft.KeyVault(VaultName=myvault;SecretName=MySecret)
│   ├── Deployment slot setting: ✅ (if slot-specific)
│   └── Apply → Save → Confirm
│
├── Step 3: Verify
│   ├── Portal: Environment variables → Check status icon:
│   │   ├── ✅ Green checkmark = resolved successfully
│   │   └── ❌ Red X = failed (check identity/RBAC/firewall)
│   └── App code reads env var normally — resolved at runtime
│
└── ⚠️ Notes
    ├── Requires: Managed Identity + Key Vault Secrets User role
    ├── Omit version = always latest secret value
    ├── Secret value refreshed on app restart or ~24 hours
    ├── Works for App Setting AND Connection String entries
    └── If Key Vault unreachable at startup → app may fail to start
```

---

### 21.7 Recover Deleted Key Vault / Objects

> **Portal:** `Key Vault → Secrets → Manage deleted secrets`

```
Recover Deleted Vault / Objects
│
├── Recover Deleted Objects (Secrets/Keys/Certs)
│   │
│   ├── Portal:
│   │   Key Vault → Objects → Secrets/Keys/Certificates →
│   │   Manage deleted secrets (toolbar button) →
│   │   Select item → Recover (or Purge)
│   │
│   ├── CLI:
│   │   # List deleted secrets
│   │   az keyvault secret list-deleted --vault-name <vault>
│   │   # Recover
│   │   az keyvault secret recover --vault-name <vault> -n <name>
│   │   # Purge (if purge protection allows)
│   │   az keyvault secret purge --vault-name <vault> -n <name>
│   │
│   └── ⚠️ Purge protection enabled → cannot purge until retention expires
│
├── Recover Deleted Vault
│   │
│   ├── CLI:
│   │   # List deleted vaults
│   │   az keyvault list-deleted
│   │   # Recover vault
│   │   az keyvault recover --name <vault-name>
│   │   # Purge vault (if allowed)
│   │   az keyvault purge --name <vault-name> --location <region>
│   │
│   └── PowerShell:
│       Get-AzKeyVault -InRemovedState
│       Undo-AzKeyVaultRemoval -VaultName <name> -ResourceGroupName <rg> -Location <region>
│
├── ⚠️ Key Points
│   ├── Soft-deleted vault name is RESERVED during retention → cannot reuse
│   ├── Recover restores vault with ALL its objects and access policies/RBAC
│   ├── Purge = permanent deletion (irreversible)
│   ├── Required RBAC: Key Vault Contributor (recover vault), relevant Officer role (recover objects)
│   └── Soft delete retention: 7–90 days (default 90)
│
└── ⚠️ Exam Scenarios
    ├── "Accidentally deleted vault" → Recover: az keyvault recover
    ├── "Need to reuse vault name" → Purge the soft-deleted vault first
    ├── "Cannot purge vault" → Purge protection enabled, wait for retention to expire
    └── "Lost access to secrets" → Check soft-deleted state, recover if within retention
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
