<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Disk Encryption — AZ-104 Revision Notes

---

## 1. What is Azure Disk Encryption?

- Encrypts **OS and data disks** of Azure VMs at rest
- Uses **BitLocker** (Windows) and **DM-Crypt** (Linux) — industry-standard technologies
- Keys stored in **Azure Key Vault** (same region as VM)
- Different from **Server-Side Encryption (SSE)** which is platform-level encryption on managed disks
- Meets regulatory and compliance requirements (FIPS 140-2)

---

## 2. Encryption Types — Full Comparison

| Feature | **Azure Disk Encryption (ADE)** | **Server-Side Encryption (SSE)** | **Encryption at Host** |
|---|---|---|---|
| **Technology** | BitLocker (Win) / DM-Crypt (Linux) | Azure Storage infrastructure | Host-level encryption |
| **Scope** | OS + Data disks | OS + Data disks | OS + Data + **Temp + Cache** disks |
| **Default** | ❌ Must enable manually | ✅ Always ON (PMK default) | ❌ Must enable manually |
| **Key management** | Key Vault (CMK) | PMK (default) or CMK (Key Vault / DES) | PMK or CMK (via DES) |
| **Temp disk encrypted** | ❌ No | ❌ No | ✅ **Yes** |
| **Disk cache encrypted** | ❌ No | ❌ No | ✅ **Yes** |
| **Guest OS sees** | Encrypted volume (BitLocker/LUKS) | Transparent | Transparent |
| **Performance impact** | ⚠️ Some (guest CPU used) | None (storage infra) | Minimal |
| **VM size restriction** | Not on Basic-tier VMs | None | Must support feature |

> ⚠️ **EXAM TIP:** **SSE (Server-Side Encryption)** is ALWAYS ON for all managed disks — you cannot disable it. Uses **Platform-Managed Keys (PMK)** by default. ADE is an **additional** layer on top of SSE.

> ⚠️ **EXAM TIP:** **Temp disks** and **disk caches** are NOT encrypted by ADE or SSE. Only **Encryption at Host** encrypts temp/cache disks. This is a key exam differentiator.

> ⚠️ **EXAM TIP:** ADE and Encryption at Host **cannot be combined** on the same VM.

---

## 3. Azure Disk Encryption (ADE) — Deep Dive

### 3.1 How It Works

```
VM Disk → BitLocker/DM-Crypt encrypts → Key stored in Key Vault
           ↓
Optional: KEK (Key Encryption Key) wraps the BitLocker/DM-Crypt key
```

| Component | Description |
|---|---|
| **BEK (BitLocker Encryption Key)** | Volume encryption key stored as Key Vault **secret** |
| **KEK (Key Encryption Key)** | Optional — wraps (encrypts) the BEK. Stored as Key Vault **key** |
| **Key Vault** | Stores BEK (as secret) and KEK (as key). Must be same region as VM |

> ⚠️ **EXAM TIP:** BEK = stored as a **secret** in Key Vault. KEK = stored as a **key** in Key Vault. KEK wraps BEK (envelope encryption). KEK is **optional but recommended**.

### 3.2 Supported OS / VM Types

| Platform | Technology | Versions |
|---|---|---|
| **Windows** | BitLocker | Windows Server 2012+ and Windows 10+ |
| **Linux** | DM-Crypt (LUKS) | Most Azure-endorsed Linux distros (Ubuntu, RHEL, CentOS, SLES, Debian) |

### 3.3 NOT Supported

| Scenario | Supported? |
|---|---|
| Basic-tier VMs | ❌ |
| VMs with < 2 GB RAM (Linux) | ❌ |
| Generation 2 VMs | ✅ (supported) |
| Temp disks | ❌ (use Encryption at Host) |
| Ultra disks | ❌ |
| Ephemeral OS disks | ❌ |
| Shared disks | ❌ |
| Classic VMs | ❌ |
| VMSS (Uniform) | ✅ |
| VMSS (Flexible) | ✅ |
| Custom images without prep | ⚠️ May fail on Linux |

> ⚠️ **EXAM TIP:** ADE does NOT support: **Basic-tier VMs**, **Ultra disks**, **Ephemeral OS disks**, **Shared disks**. Linux VMs need **≥2 GB RAM** for OS disk encryption.

### 3.4 Volume Types

| Option | Encrypts |
|---|---|
| **OS** | OS disk only |
| **Data** | Data disks only |
| **All** | OS + Data disks (recommended) |

> ⚠️ **EXAM TIP:** On Linux, you must encrypt the **OS disk first** before data disks. On Windows, you can encrypt data disks independently.

---

## 4. Server-Side Encryption (SSE)

- **Always enabled** for all Azure managed disks — cannot disable
- Encrypts data at rest at the **storage infrastructure level** (transparent to VM)
- Default: **Platform-Managed Keys (PMK)** — Microsoft manages keys
- Optional: **Customer-Managed Keys (CMK)** via Disk Encryption Set (DES)

### Key Management Options for SSE

| Option | Key Manager | Key Location | Description |
|---|---|---|---|
| **Platform-Managed Keys (PMK)** | Microsoft | Azure-managed | Default — no config needed |
| **Customer-Managed Keys (CMK)** | Customer | Azure Key Vault | Customer controls key in Key Vault |
| **Double encryption** | Both | PMK + CMK | Two layers: infra (PMK) + service (CMK) |

### Disk Encryption Set (DES)

- Azure resource that links a **Key Vault key** to managed disks
- One DES can be assigned to multiple disks
- DES specifies: Key Vault, Key, Encryption type

#### Portal Path — Create Disk Encryption Set
```
Home → Disk Encryption Sets → + Create →
Subscription, Resource Group, Name, Region →
Encryption type:
  ├── Encryption at rest with a customer-managed key
  ├── Double encryption with platform-managed and customer-managed keys
  └── Encryption at rest with a platform-managed key (default — no DES needed)
Key Vault: Select → Key: Select → Create
```

#### Portal Path — Assign DES to Existing Disk
```
Managed Disk → Settings → Encryption →
Encryption type: Encryption at rest with a customer-managed key →
Disk Encryption Set: Select DES →
Save
⚠️ VM must be deallocated to change encryption on OS disk
```

> ⚠️ **EXAM TIP:** To change SSE from PMK to CMK on an **OS disk**, the VM must be **deallocated** first. Data disks can be changed while VM is running.

> ⚠️ **EXAM TIP:** **Disk Encryption Set** = Azure resource linking Key Vault key to disks. Multiple disks can reference the same DES.

---

## 5. Encryption at Host

- Encrypts data on the **VM host** before it reaches Azure Storage
- Covers: **Temp disks**, **OS/Data disk caches**, **OS/Data disks** (end-to-end)
- Transparent — no performance impact in guest OS
- Uses **PMK** (default) or **CMK** (via Disk Encryption Set)

### Encryption at Host vs ADE vs SSE

| What Gets Encrypted | SSE (always on) | ADE | Encryption at Host |
|---|---|---|---|
| OS disk (at rest) | ✅ | ✅ | ✅ |
| Data disk (at rest) | ✅ | ✅ | ✅ |
| Temp disk | ❌ | ❌ | ✅ |
| OS disk cache | ❌ | ❌ | ✅ |
| Data disk cache | ❌ | ❌ | ✅ |
| Data in transit to storage | ❌ | ❌ | ✅ |

### Prerequisites

| Requirement | Detail |
|---|---|
| **VM size** | Must support Encryption at Host (most modern sizes) |
| **Subscription feature** | Register: `Microsoft.Compute/EncryptionAtHost` |
| **Cannot combine** | Cannot use with ADE on same VM |

#### Register Feature (CLI)
```bash
az feature register --name EncryptionAtHost --namespace Microsoft.Compute
az provider register --namespace Microsoft.Compute
```

#### Portal Path — Enable on VM
```
Virtual Machine → Disks → Additional settings →
Encryption at host: ✅ → Save
⚠️ VM must be deallocated to enable/disable
```

> ⚠️ **EXAM TIP:** Encryption at Host must be **registered** as a subscription-level feature before use. VM must be **deallocated** to enable.

> ⚠️ **EXAM TIP:** **Cannot use ADE + Encryption at Host** on the same VM. Choose one. Encryption at Host provides broader coverage (temp + cache).

---

## 6. Confidential Disk Encryption

- Encrypts the **VM guest state** (firmware state, boot data) and **OS disk** with a key tied to the VM's TPM
- For **Confidential VMs** only (DCasv5, DCadsv5, ECasv5, ECadsv5 series)
- Key bound to vTPM — cannot be extracted or used on other VMs
- **Not applicable to regular VMs** — exam rarely tests depth but know it exists

> ⚠️ **EXAM TIP:** Confidential Disk Encryption = Confidential VMs only. Not the same as ADE or Encryption at Host. Uses TPM-bound keys.

---

## 7. Managed Disk Types & Encryption Support

| Disk Type | SSE (Always) | ADE | Encryption at Host | CMK (DES) |
|---|---|---|---|---|
| **Premium SSD v2** | ✅ | ❌ | ✅ | ✅ (DES) |
| **Premium SSD** | ✅ | ✅ | ✅ | ✅ (DES) |
| **Standard SSD** | ✅ | ✅ | ✅ | ✅ (DES) |
| **Standard HDD** | ✅ | ✅ | ✅ | ✅ (DES) |
| **Ultra Disk** | ✅ | ❌ | ✅ | ✅ (DES) |
| **Ephemeral OS Disk** | ❌ (local) | ❌ | ✅ | ❌ |
| **Shared Disk** | ✅ | ❌ | ✅ | ✅ (DES) |
| **Temp Disk** | ❌ | ❌ | ✅ | ❌ |

> ⚠️ **EXAM TIP:** ADE does NOT work with: **Ultra Disks**, **Premium SSD v2**, **Ephemeral OS disks**, **Shared disks**. SSE is always on for managed disks. Encryption at Host has the broadest coverage.

---

## 8. Key Vault Requirements for ADE

| Requirement | Detail |
|---|---|
| **Region** | Same region as VM (mandatory) |
| **Soft delete** | ✅ Required (now mandatory on all vaults) |
| **Purge protection** | ✅ Recommended |
| **SKU** | Standard (software keys) or Premium (HSM keys) |
| **Access** | "Azure Disk Encryption for volume encryption" enabled |
| **Permission model** | Access Policy: Wrap/Unwrap keys, Get/Set secrets. RBAC: Key Vault Crypto User + Secrets User |

#### Portal Path — Enable Key Vault for ADE
```
Key Vault → Settings → Access configuration →
Resource access:
  ✅ Azure Disk Encryption for volume encryption → Save
```

> ⚠️ **EXAM TIP:** Key Vault MUST be in the **same region** as the VM. Cross-region Key Vault references are NOT supported for ADE.

> ⚠️ **EXAM TIP:** Must enable **"Azure Disk Encryption for volume encryption"** checkbox on the Key Vault. Without this, ADE cannot access the vault.

---

## 9. Security & RBAC

### Required Roles

| Role | Purpose |
|---|---|
| **Virtual Machine Contributor** | Enable encryption on VMs |
| **Key Vault Contributor** | Manage Key Vault (not data plane) |
| **Key Vault Crypto Officer** | Manage keys (create KEK) |
| **Key Vault Secrets Officer** | Manage secrets (BEK stored as secret) |
| **Disk Encryption Set Operator** | For CMK with DES |
| **Managed Identity Operator** | If DES uses managed identity |

### Permission Requirements (Access Policy Model)

| Operation | Key Permissions | Secret Permissions |
|---|---|---|
| **Enable ADE (BEK only)** | — | Get, Set, List, Delete |
| **Enable ADE (with KEK)** | Get, WrapKey, UnwrapKey | Get, Set, List, Delete |
| **Disable ADE** | — | Get |

### Managed Identity for DES

| Component | Identity | Purpose |
|---|---|---|
| **Disk Encryption Set** | System-assigned managed identity | Access Key Vault key for CMK |
| **Access needed** | Key Vault Crypto Service Encryption User | Wrap/unwrap keys |

> ⚠️ **EXAM TIP:** DES gets a **system-assigned managed identity** automatically. This identity needs **Key Vault Crypto Service Encryption User** role on the Key Vault.

---

## 10. Monitoring & Compliance

### Check Encryption Status

#### Portal Path
```
Virtual Machine → Disks → Encryption column shows status
Virtual Machine → Extensions + applications → Check AzureDiskEncryption extension
```

#### CLI
```bash
# Check disk encryption status
az vm encryption show -g <rg> --name <vm>
# Output: OsVolumeEncrypted, DataVolumesEncrypted status
```

#### PowerShell
```powershell
Get-AzVmDiskEncryptionStatus -ResourceGroupName <rg> -VMName <vm>
```

### Azure Policy for Compliance

| Policy | Description |
|---|---|
| **Audit VMs without disk encryption** | Reports non-compliant VMs |
| **Deploy ADE on Windows VMs** | Auto-deploy encryption |
| **Managed disks should use CMK** | Audit SSE key type |

#### Portal Path — Assign Policy
```
Policy → Definitions → Search "disk encryption" →
Select policy → Assign → Scope: Subscription/RG → Assign
```

### Azure Security Center / Defender

- Alerts for: unencrypted disks, missing ADE
- Recommendation: "Disk encryption should be applied on virtual machines"

> ⚠️ **EXAM TIP:** Azure Policy can **audit** VMs without disk encryption. Microsoft Defender for Cloud flags unencrypted disks as a security recommendation.

---

## 11. Pricing Key Points

| Component | Cost |
|---|---|
| **SSE (PMK)** | ✅ **Free** (included with managed disks) |
| **SSE (CMK via DES)** | Key Vault key charges (transactions + key/month for HSM) |
| **ADE** | ✅ **Free** (no extra Azure charge) — Key Vault transaction costs apply |
| **Encryption at Host** | ✅ **Free** (no extra charge) |
| **Key Vault (Standard)** | $0.03 / 10K transactions for software keys |
| **Key Vault (Premium)** | $1/key/month + $0.03 / 10K transactions for HSM keys |
| **DES resource** | ✅ **Free** |

> ⚠️ **EXAM TIP:** ADE, SSE (PMK), and Encryption at Host are all **free** — you only pay for Key Vault transactions when using CMK/ADE.

---

## 12. Limitations & Constraints

| Constraint | Detail |
|---|---|
| ADE + Encryption at Host | ❌ Cannot combine on same VM |
| ADE on Basic VMs | ❌ Not supported |
| ADE on Ultra Disks | ❌ Not supported |
| ADE on Ephemeral OS | ❌ Not supported |
| ADE on Shared Disks | ❌ Not supported |
| ADE on Premium SSD v2 | ❌ Not supported |
| Linux OS encryption RAM | ≥ 2 GB required |
| Key Vault region | Must match VM region |
| Change OS disk encryption (SSE) | VM must be deallocated |
| ADE Linux OS + Data | OS must be encrypted first |
| Temp disk / cache | Only Encryption at Host |
| SSE disable | ❌ Cannot disable (always on) |
| DES key rotation | Auto-rotation supported (Key Vault) |
| Max disks per VM | Varies by VM size (up to 64) |

---

## 13. CLI / PowerShell Commands

### Azure CLI

| Action | Command |
|---|---|
| Enable ADE (BEK only) | `az vm encryption enable -g <rg> --name <vm> --disk-encryption-keyvault <vault>` |
| Enable ADE (with KEK) | `az vm encryption enable -g <rg> --name <vm> --disk-encryption-keyvault <vault> --key-encryption-key <key-name> --volume-type All` |
| Enable ADE (OS only) | `az vm encryption enable -g <rg> --name <vm> --disk-encryption-keyvault <vault> --volume-type OS` |
| Enable ADE (Data only) | `az vm encryption enable -g <rg> --name <vm> --disk-encryption-keyvault <vault> --volume-type Data` |
| Check status | `az vm encryption show -g <rg> --name <vm>` |
| Disable ADE (Windows) | `az vm encryption disable -g <rg> --name <vm> --volume-type All` |
| Disable ADE (Linux data) | `az vm encryption disable -g <rg> --name <vm> --volume-type Data` |
| Create KEK | `az keyvault key create --vault-name <vault> -n <key> --kty RSA --size 4096` |
| Create DES | `az disk-encryption-set create -g <rg> -n <des> --key-url <key-id> --source-vault <vault-id>` |
| Show DES | `az disk-encryption-set show -g <rg> -n <des>` |
| Encrypt VMSS | `az vmss encryption enable -g <rg> --name <vmss> --disk-encryption-keyvault <vault>` |

### PowerShell

| Action | Command |
|---|---|
| Enable ADE | `Set-AzVMDiskEncryptionExtension -ResourceGroupName <rg> -VMName <vm> -DiskEncryptionKeyVaultUrl <url> -DiskEncryptionKeyVaultId <id>` |
| Enable with KEK | `Set-AzVMDiskEncryptionExtension -ResourceGroupName <rg> -VMName <vm> -DiskEncryptionKeyVaultUrl <url> -DiskEncryptionKeyVaultId <id> -KeyEncryptionKeyUrl <kek-url> -KeyEncryptionKeyVaultId <kv-id>` |
| Check status | `Get-AzVmDiskEncryptionStatus -ResourceGroupName <rg> -VMName <vm>` |
| Disable ADE | `Disable-AzVMDiskEncryption -ResourceGroupName <rg> -VMName <vm>` |
| Create DES | `New-AzDiskEncryptionSet -ResourceGroupName <rg> -Name <des> -KeyUrl <url> -SourceVaultId <id> -Location <region>` |

---

## 14. Quick-Fire Exam Points ⚡

1. **SSE** = always ON for all managed disks — cannot disable. Default = **PMK** (Microsoft-managed)
2. **ADE** = BitLocker (Windows) / DM-Crypt (Linux), keys in **Key Vault** — must enable manually
3. **Encryption at Host** = encrypts **temp disks + cache + OS + data** — broadest coverage
4. ADE + Encryption at Host = **CANNOT combine** on the same VM — choose one
5. **BEK** stored as Key Vault **secret**. **KEK** stored as Key Vault **key** (wraps BEK)
6. KEK is **optional but recommended** — adds extra layer of protection
7. Key Vault must be in **same region** as VM for ADE — cross-region NOT supported
8. Must enable **"Azure Disk Encryption for volume encryption"** on Key Vault for ADE
9. Key Vault requires: **soft delete** (mandatory) + **purge protection** (recommended) for ADE
10. ADE does NOT encrypt: **temp disks**, **disk cache** — use Encryption at Host for those
11. ADE NOT supported on: **Basic VMs**, **Ultra Disks**, **Premium SSD v2**, **Ephemeral OS**, **Shared disks**
12. Linux: **≥2 GB RAM** required for OS disk encryption
13. Linux: must encrypt **OS disk before** data disks. Windows: can do data independently
14. **Volume types**: OS / Data / All — specify with `--volume-type`
15. **Disable ADE on Linux**: can only disable on **data** volumes — **cannot disable OS encryption** on Linux
16. **SSE with CMK** uses **Disk Encryption Set (DES)** — links Key Vault key to disks
17. DES gets **system-assigned managed identity** → needs **Key Vault Crypto Service Encryption User** role
18. Changing OS disk SSE from PMK → CMK requires **VM deallocated**
19. **Double encryption** = PMK (infra layer) + CMK (service layer) — via DES encryption type
20. ADE installs a **VM extension**: `AzureDiskEncryption` (Windows) / `AzureDiskEncryptionForLinux`
21. Check encryption: `az vm encryption show` or `Get-AzVmDiskEncryptionStatus`
22. ADE, SSE (PMK), Encryption at Host are all **FREE** — only Key Vault transactions cost money
23. Azure Policy: **"Disk encryption should be applied on virtual machines"** — audit compliance
24. **Confidential Disk Encryption** = TPM-bound keys, Confidential VMs only (DCasv5/ECasv5)
25. VMSS supports ADE — `az vmss encryption enable`

---

## 15. Step-by-Step Configuration Mind Maps 🗺️

---

### 15.1 Enable Azure Disk Encryption (ADE) on a VM

> **Portal:** `Virtual Machine → Disks → Additional settings → Encryption settings`

```
Enable ADE on VM
│
├── Prerequisites
│   ├── VM is running (can encrypt online — no deallocation needed for ADE)
│   │   ⚠️ Linux OS encryption may require several hours + temp performance impact
│   ├── Key Vault exists in SAME REGION as VM
│   │   ⚠️ Cross-region NOT supported
│   ├── Key Vault settings configured:
│   │   Portal: Key Vault → Access configuration →
│   │   ✅ Azure Disk Encryption for volume encryption → Save
│   │   ⚠️ Without this, ADE cannot access vault
│   ├── Key Vault: Soft delete enabled (mandatory)
│   ├── Key Vault: Purge protection recommended
│   ├── VM is NOT Basic tier, NOT using Ultra/Shared/Ephemeral disks
│   ├── Linux VM: ≥ 2 GB RAM (for OS disk encryption)
│   └── RBAC: VM Contributor + Key Vault secret/key permissions
│
├── Step 1 (Optional): Create KEK in Key Vault
│   │   Portal: Key Vault → Objects → Keys → + Generate/Import
│   ├── Name: vm-disk-kek
│   ├── Key type: RSA (or RSA-HSM for Premium vault)
│   ├── RSA key size: 2048 / 3072 / 4096
│   └── Create
│       ⚠️ KEK is optional but adds security (wraps BEK)
│
├── Step 2: Enable Encryption
│   │
│   ├── Method 1: Portal
│   │   │   VM → Disks → Additional settings
│   │   ├── Disks to encrypt:
│   │   │   ├── OS and data disks (recommended)
│   │   │   ├── OS disk only
│   │   │   └── Data disks only
│   │   │   ⚠️ Linux: OS must be encrypted first before data disks
│   │   ├── Key vault: Select (same region)
│   │   ├── Key (KEK): Select key (optional)
│   │   └── Save
│   │
│   ├── Method 2: CLI (BEK only)
│   │   az vm encryption enable \
│   │     -g <rg> --name <vm> \
│   │     --disk-encryption-keyvault <vault-name> \
│   │     --volume-type All
│   │
│   ├── Method 3: CLI (with KEK)
│   │   az vm encryption enable \
│   │     -g <rg> --name <vm> \
│   │     --disk-encryption-keyvault <vault-name> \
│   │     --key-encryption-key <kek-name> \
│   │     --volume-type All
│   │
│   └── Method 4: PowerShell
│       $kv = Get-AzKeyVault -VaultName <vault>
│       Set-AzVMDiskEncryptionExtension -ResourceGroupName <rg> -VMName <vm> \
│         -DiskEncryptionKeyVaultUrl $kv.VaultUri \
│         -DiskEncryptionKeyVaultId $kv.ResourceId
│
├── Step 3: Verify
│   ├── CLI: az vm encryption show -g <rg> --name <vm>
│   │   → OsVolumeEncrypted: Encrypted
│   │   → DataVolumesEncrypted: Encrypted
│   ├── Portal: VM → Disks → Encryption column
│   └── Extension: VM → Extensions → AzureDiskEncryption installed
│
└── ⚠️ Notes
    ├── Encryption runs in background — may take 15-60+ minutes
    ├── Linux OS encryption may take longer + require reboot
    ├── VM extension "AzureDiskEncryption" auto-installed
    ├── Do NOT delete Key Vault or key while ADE is active
    │   → VM will fail to boot
    └── BEK stored as secret, KEK as key in Key Vault
```

---

### 15.2 Disable Azure Disk Encryption

> **Portal:** `Virtual Machine → Disks → Additional settings → Disable`

```
Disable ADE
│
├── Windows VM
│   │
│   ├── Portal: VM → Disks → Additional settings →
│   │   Disks to encrypt: None → Save
│   │
│   ├── CLI: az vm encryption disable -g <rg> --name <vm> --volume-type All
│   │
│   ├── PowerShell: Disable-AzVMDiskEncryption -ResourceGroupName <rg> -VMName <vm>
│   │
│   └── ✅ Can disable on both OS and Data disks
│
├── Linux VM
│   │
│   ├── CLI: az vm encryption disable -g <rg> --name <vm> --volume-type Data
│   │
│   └── ⚠️ Linux: can ONLY disable on DATA disks
│       ⚠️ CANNOT disable OS disk encryption on Linux
│       ⚠️ To remove OS encryption: redeploy VM with unencrypted OS disk
│
├── Verify
│   ├── az vm encryption show -g <rg> --name <vm>
│   │   → OsVolumeEncrypted: NotEncrypted (Windows) / Encrypted (Linux — stays)
│   │   → DataVolumesEncrypted: NotEncrypted
│   └── Extension may remain installed (can manually remove)
│
└── ⚠️ Notes
    ├── Linux OS encryption = CANNOT be disabled (exam gotcha!)
    ├── Disabling removes BEK from Key Vault usage
    ├── Disk data remains accessible after disabling
    └── Extensions may need manual cleanup
```

---

### 15.3 Configure SSE with Customer-Managed Keys (DES)

> **Portal:** `Home → Disk Encryption Sets → + Create`

```
Configure SSE with CMK (Disk Encryption Set)
│
├── Prerequisites
│   ├── Key Vault exists with:
│   │   ├── Soft delete: ✅ (mandatory)
│   │   ├── Purge protection: ✅ (required for DES)
│   │   │   ⚠️ DES creation FAILS without purge protection on Key Vault
│   │   └── A key created (RSA 2048+ recommended)
│   └── RBAC: Contributor + Key Vault Crypto Officer
│
├── Step 1: Create Key in Key Vault
│   │   Key Vault → Keys → + Generate/Import
│   ├── Name: disk-cmk-key
│   ├── Key type: RSA / RSA-HSM
│   ├── Size: 2048 / 3072 / 4096
│   └── Create
│
├── Step 2: Create Disk Encryption Set
│   │   Portal: Home → Disk Encryption Sets → + Create
│   ├── Subscription, Resource Group
│   ├── Name: my-disk-encryption-set
│   ├── Region: Same as disks
│   │   ⚠️ DES and disks must be in SAME REGION
│   ├── Encryption type:
│   │   ├── Encryption at rest with a customer-managed key (CMK only)
│   │   └── Double encryption with platform-managed and customer-managed keys
│   │       ⚠️ Double encryption = 2 layers (PMK + CMK)
│   ├── Key Vault: Select
│   ├── Key: Select
│   ├── Enable auto key rotation: ✅ (recommended)
│   │   ⚠️ Auto-rotation: when key rotates in KV, DES auto-uses new version
│   └── Review + create → Create
│
├── Step 3: Grant DES Access to Key Vault
│   │   ⚠️ DES system-assigned managed identity needs Key Vault access
│   │   Portal: Notification after DES creation → "Grant access" link
│   │   OR: Key Vault → Access Control (IAM) → + Add role assignment →
│   │   Role: Key Vault Crypto Service Encryption User →
│   │   Member: DES managed identity → Assign
│
├── Step 4: Assign DES to Disk(s)
│   │
│   ├── New VM:
│   │   VM creation → Disks tab →
│   │   Encryption type: Customer-managed keys →
│   │   Disk Encryption Set: Select DES
│   │
│   ├── Existing Disk (Unattached):
│   │   Managed Disk → Encryption →
│   │   Encryption type: Customer-managed keys →
│   │   DES: Select → Save
│   │
│   └── Existing OS Disk (Attached):
│       ⚠️ VM must be DEALLOCATED first
│       Deallocate VM → Managed Disk → Encryption →
│       Change to CMK → Select DES → Save → Start VM
│
├── Step 5: Verify
│   ├── Managed Disk → Encryption → Shows DES name and key info
│   └── DES → Associated resources → Lists linked disks
│
└── ⚠️ Notes
    ├── DES + Key Vault must be same region as disks
    ├── Purge protection REQUIRED on Key Vault for DES
    ├── DES managed identity needs Key Vault access
    ├── Auto key rotation recommended for compliance
    ├── Changing OS disk encryption type = VM deallocated
    └── One DES can serve multiple disks
```

---

### 15.4 Enable Encryption at Host

> **Portal:** `Virtual Machine → Disks → Additional settings`

```
Enable Encryption at Host
│
├── Prerequisites
│   ├── Subscription feature registered:
│   │   az feature register --name EncryptionAtHost --namespace Microsoft.Compute
│   │   az provider register --namespace Microsoft.Compute
│   │   ⚠️ Registration may take several minutes
│   │   Verify: az feature show --name EncryptionAtHost --namespace Microsoft.Compute
│   ├── VM size supports Encryption at Host
│   │   (most modern sizes: Dsv3, Esv3, Fsv2, Dasv4, Easv4, etc.)
│   ├── VM must NOT have ADE enabled
│   │   ⚠️ ADE + Encryption at Host = CANNOT coexist
│   └── RBAC: Virtual Machine Contributor
│
├── Enable on New VM
│   │   Portal: Create VM → Disks tab →
│   │   Encryption at host: ✅ checkbox
│   └── Create
│
├── Enable on Existing VM
│   │   ⚠️ VM must be DEALLOCATED first
│   │   Step 1: Stop (deallocate) VM
│   │   Step 2: VM → Disks → Additional settings →
│   │     Encryption at host: ✅ → Save
│   │   Step 3: Start VM
│   │
│   │   CLI:
│   │   az vm deallocate -g <rg> --name <vm>
│   │   az vm update -g <rg> --name <vm> --set securityProfile.encryptionAtHost=true
│   │   az vm start -g <rg> --name <vm>
│
├── With CMK (Optional)
│   ├── Create DES (see Mind Map 15.3)
│   ├── Assign DES to disks
│   └── Enable Encryption at Host — temp + cache use CMK from DES
│       If no DES → uses PMK for all
│
├── Verify
│   ├── az vm show -g <rg> --name <vm> --query securityProfile
│   │   → encryptionAtHost: true
│   └── All disk data flows encrypted end-to-end (host to storage)
│
└── ⚠️ Notes
    ├── Encrypts: temp disks + cache + OS + data (most comprehensive)
    ├── Cannot combine with ADE
    ├── Must register subscription feature first
    ├── VM must be deallocated to toggle
    ├── No guest-level impact (transparent)
    ├── Extra security for compliance scenarios
    └── FREE — no additional charges
```

---

### 15.5 Verify and Audit Disk Encryption Compliance

> **Portal:** `Virtual Machine → Disks` or `Azure Policy`

```
Verify & Audit Disk Encryption
│
├── Per-VM Verification
│   │
│   ├── Portal: VM → Disks → Check Encryption column
│   │   ├── SSE: Always shows encryption type (PMK or CMK)
│   │   └── ADE: Shows "AzureDiskEncryption" extension
│   │
│   ├── CLI:
│   │   # ADE status
│   │   az vm encryption show -g <rg> --name <vm>
│   │   # Expected: OsVolumeEncrypted: Encrypted
│   │   #           DataVolumesEncrypted: Encrypted
│   │   
│   │   # Encryption at Host
│   │   az vm show -g <rg> --name <vm> \
│   │     --query securityProfile.encryptionAtHost
│   │   # Expected: true
│   │
│   └── PowerShell:
│       Get-AzVmDiskEncryptionStatus -ResourceGroupName <rg> -VMName <vm>
│
├── Fleet-Wide Audit (Azure Policy)
│   │   Portal: Policy → Definitions → Search:
│   │
│   ├── Built-in Policies:
│   │   ├── "Virtual machines should encrypt temp disks, caches, and data flows"
│   │   │   → Audits Encryption at Host
│   │   ├── "Disk encryption should be applied on virtual machines"
│   │   │   → Audits ADE status
│   │   ├── "Managed disks should use a specific set of disk encryption sets"
│   │   │   → Enforce CMK via DES
│   │   └── "OS and data disks should be encrypted with a CMK"
│   │       → Audit SSE key type
│   │
│   ├── Assign: Policy → Assign → Scope (subscription/RG)
│   │   Effect: Audit / Deny / DeployIfNotExists
│   │
│   └── Compliance: Policy → Compliance → View non-compliant resources
│
├── Microsoft Defender for Cloud
│   │   Portal: Defender for Cloud → Recommendations →
│   │   "Disk encryption should be applied on virtual machines"
│   ├── Severity: High
│   ├── Shows: List of non-compliant VMs
│   └── Quick Fix: Remediate directly
│
└── ⚠️ Notes
    ├── Policy "Deny" effect can PREVENT creating unencrypted VMs
    ├── DeployIfNotExists can AUTO-APPLY ADE
    ├── Regular auditing recommended for compliance
    └── Combine Policy + Defender for comprehensive coverage
```

---


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
