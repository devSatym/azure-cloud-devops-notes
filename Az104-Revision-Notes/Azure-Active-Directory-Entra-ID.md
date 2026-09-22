<!-- AUTHOR-WATERMARK: TOP -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: TOP -->

---

# Azure Active Directory (Entra ID) — AZ-104 Revision Notes

---

## 1. What is Azure AD (Microsoft Entra ID)?

- **Cloud-based identity and access management (IAM)** service
- Renamed from Azure Active Directory → **Microsoft Entra ID** (July 2023)
- Manages **users, groups, applications, devices**
- Provides **authentication + authorization** for Azure, Microsoft 365, SaaS apps
- **NOT the same** as on-prem Active Directory Domain Services (AD DS)
- Every Azure subscription is **associated with exactly one** Entra ID tenant
- **Tenant** = dedicated instance of Entra ID; represents an organization

> ⚠️ **EXAM TIP:** Azure AD ≠ Windows Server AD DS. Azure AD uses **HTTP/HTTPS** protocols (OAuth, OIDC, SAML). AD DS uses **Kerberos, LDAP, NTLM**. No OUs, GPOs, or domain trusts in Azure AD.

---

## 2. Key Components

| Component | Purpose |
|---|---|
| **Tenant** | Dedicated Entra ID instance (1 per organization) |
| **Users** | Identity for a person (member or guest) |
| **Groups** | Collection of users/devices for access management |
| **App Registrations** | Register apps for SSO and API access |
| **Enterprise Applications** | SaaS/gallery apps + service principals |
| **Devices** | Registered/joined devices for conditional access |
| **Roles & Administrators** | Built-in and custom directory roles |
| **Licenses** | P1/P2 license assignment to users |
| **External Identities (B2B)** | Guest user collaboration |
| **Conditional Access** | Policy-based access controls (P1+) |
| **Identity Protection** | Risk-based policies (P2) |
| **Privileged Identity Management (PIM)** | Just-in-time admin access (P2) |
| **Self-Service Password Reset (SSPR)** | Users reset own passwords |
| **MFA** | Multi-factor authentication |
| **Azure AD Connect / Cloud Sync** | Sync on-prem AD to Entra ID |

---

## 3. Azure AD Editions / Licenses

| Feature | Free | Microsoft 365 | P1 | P2 |
|---|---|---|---|---|
| **Users / Groups** | ✅ | ✅ | ✅ | ✅ |
| **SSO (unlimited apps)** | ✅ (10 apps) | ✅ | ✅ Unlimited | ✅ Unlimited |
| **MFA** | ✅ (Security Defaults) | ✅ | ✅ | ✅ |
| **Self-Service Password Reset** | ❌ Cloud only | ✅ Cloud | ✅ Cloud + On-prem writeback | ✅ Cloud + On-prem writeback |
| **Conditional Access** | ❌ | ❌ | ✅ | ✅ |
| **Dynamic Groups** | ❌ | ❌ | ✅ | ✅ |
| **Group expiration policy** | ❌ | ❌ | ✅ | ✅ |
| **Azure AD Connect Health** | ❌ | ❌ | ✅ | ✅ |
| **Identity Protection** | ❌ | ❌ | ❌ | ✅ |
| **PIM** | ❌ | ❌ | ❌ | ✅ |
| **Access Reviews** | ❌ | ❌ | ❌ | ✅ |
| **Entitlement Management** | ❌ | ❌ | ❌ | ✅ |
| **Object limit** | 50,000 | No limit | No limit | No limit |

> ⚠️ **EXAM TIP:** **Conditional Access = P1**. **Identity Protection + PIM + Access Reviews = P2**. **Free tier** = 50,000 object limit. **SSPR with on-prem writeback = P1+**.

---

## 4. Users

### User Types

| Type | Description |
|---|---|
| **Member** | Belongs to the organization (internal user) |
| **Guest** | External user invited via B2B (default permissions are restricted) |

### User Sources

| Source | Description |
|---|---|
| **Cloud identity** | Created directly in Entra ID |
| **Synced from on-prem** | Azure AD Connect / Cloud Sync |
| **External identity (Guest)** | Invited via B2B from another Entra ID, Microsoft account, Google, etc. |

### Portal Path — Create User
```
Microsoft Entra ID → Users → + New user → Create new user
→ User principal name, Display name, Password (auto-generate or custom)
→ Properties (Job title, Department, Usage location)
→ Assignments (Groups, Roles)
→ Review + Create
```

### Portal Path — Invite Guest User
```
Microsoft Entra ID → Users → + New user → Invite external user
→ Email address, Display name, Personal message (optional)
→ Assignments (Groups, Roles)
→ Review + Create
```

### Bulk User Operations

| Operation | Portal Path |
|---|---|
| **Bulk create** | Users → Bulk operations → Bulk create → Upload CSV |
| **Bulk invite** | Users → Bulk operations → Bulk invite → Upload CSV |
| **Bulk delete** | Users → Bulk operations → Bulk delete → Upload CSV |
| **Download users** | Users → Bulk operations → Download users |

- CSV template downloadable from portal
- Max **50,000 users** per bulk operation

### Key User Properties

| Property | Notes |
|---|---|
| **User Principal Name (UPN)** | username@domain.com — used for sign-in |
| **Display Name** | Required |
| **Usage Location** | Required for license assignment (2-letter country code) |
| **Job Title, Department** | Optional metadata |
| **Manager** | Optional hierarchy |
| **Account enabled** | Toggle on/off |

> ⚠️ **EXAM TIP:** **Usage Location** MUST be set before assigning licenses. Bulk operations use **CSV files**. Guest users have **limited default permissions** (cannot enumerate users/groups by default).

---

## 5. Groups

### Group Types

| Type | Members Can Be | Use Case |
|---|---|---|
| **Security** | Users, devices, groups, service principals | RBAC, resource access, conditional access |
| **Microsoft 365** | Users only | Email distribution, shared mailbox, Teams, SharePoint |

### Membership Types

| Type | Description | License Needed |
|---|---|---|
| **Assigned** | Manually add/remove members | Free |
| **Dynamic User** | Auto-add based on user attributes (rules) | **P1** |
| **Dynamic Device** | Auto-add based on device attributes (Security groups only) | **P1** |

### Dynamic Membership Rules
- Query syntax: `(user.department -eq "Sales")` or `(user.jobTitle -contains "Manager")`
- Operators: `-eq`, `-ne`, `-contains`, `-notContains`, `-startsWith`, `-in`, `-match`
- Can combine with `-and`, `-or`, `-not`
- Processing can take **minutes to hours** depending on tenant size
- Only available with **P1 or P2** license

### Portal Path — Create Group
```
Microsoft Entra ID → Groups → + New group →
Group type (Security / Microsoft 365) →
Group name, Description →
Membership type (Assigned / Dynamic User / Dynamic Device) →
(If Dynamic) → Add dynamic query → Configure rules → Save →
Add Owners → Add Members → Create
```

### Group Nesting Rules

| Scenario | Allowed? |
|---|---|
| Security group inside Security group | ✅ Yes |
| Microsoft 365 group inside Security group | ✅ Yes |
| Security group inside Microsoft 365 group | ❌ No |
| Microsoft 365 group inside Microsoft 365 group | ❌ No |

> ⚠️ **EXAM TIP:** **Dynamic groups = P1**. **Microsoft 365 groups** cannot contain other groups. **Dynamic Device** membership = Security groups only. Dynamic rule processing is **not instant**.

---

## 6. Administrative Units (AUs)

- **Organizational container** to restrict admin role scope to a subset of users/groups/devices
- Like OUs in on-prem AD but for Entra ID
- Requires **P1** license
- Delegate admin roles within an AU (e.g., User Admin only for a specific department)

### Supported Roles in AUs

- User Administrator
- Groups Administrator
- Helpdesk Administrator
- License Administrator
- Password Administrator
- Authentication Administrator

### Portal Path — Create Administrative Unit
```
Microsoft Entra ID → Administrative units → + Add →
Name, Description →
Membership type (Assigned / Dynamic) →
Add Members (users/groups/devices) →
Assign Roles (scoped to this AU) → Create
```

> ⚠️ **EXAM TIP:** AUs = **scope delegation**. A User Administrator scoped to an AU can only manage users **within that AU**. AUs require **P1**. Supports dynamic membership (P1).

---

## 7. Self-Service Password Reset (SSPR)

### SSPR Settings

| Setting | Options |
|---|---|
| **Enabled for** | None / Selected group / All users |
| **Authentication methods required** | 1 or 2 methods |
| **Methods available** | Mobile app notification, Mobile app code, Email, Mobile phone (SMS), Office phone, Security questions |
| **Security questions** | Set number required to register + number required to reset |
| **Registration** | Require users to register at sign-in (Yes/No), Re-confirm after N days (default 180) |
| **Notifications** | Notify users on reset (Yes), Notify admins on other admin reset (Yes) |
| **On-prem writeback** | Write passwords back to on-prem AD (requires Azure AD Connect + P1) |

### Authentication Methods for SSPR

| Method | Usable as |
|---|---|
| Mobile app notification (Authenticator) | Primary |
| Mobile app code (TOTP) | Primary or Secondary |
| Email | Secondary |
| Mobile phone (SMS) | Primary or Secondary |
| Office phone | Secondary |
| Security questions | Secondary (reset only, NOT for MFA) |

### Portal Path — Configure SSPR
```
Microsoft Entra ID → Password reset →
Properties → SSPR enabled: None / Selected / All →
Authentication methods → Methods required (1 or 2) → Select methods →
Registration → Require register at sign-in → Re-confirm days →
Notifications → Configure →
On-premises integration → Enable password writeback → Save
```

### Password Writeback Requirements
- Azure AD Connect (or Cloud Sync) installed
- **P1 or P2** license
- Azure AD Connect configured for password writeback
- Allows users who reset password in cloud to have it written back to on-prem AD

> ⚠️ **EXAM TIP:** SSPR → **Selected** = one group only (nest groups if needed). **Security questions** = NOT usable for MFA, only SSPR. **Password writeback** = P1 + Azure AD Connect. SSPR for **admins always requires 2 methods** (cannot be changed).

---

## 8. Multi-Factor Authentication (MFA)

### MFA Methods

| Method | Type |
|---|---|
| **Microsoft Authenticator** (push notification) | Primary |
| **Authenticator app (TOTP code)** | Primary |
| **SMS** | Primary |
| **Voice call** | Primary |
| **FIDO2 security key** | Primary (passwordless) |
| **Windows Hello for Business** | Primary (passwordless) |
| **Hardware OATH tokens** | Primary |
| **Email OTP** | Secondary (guests) |

### Ways to Enforce MFA

| Method | Description | License |
|---|---|---|
| **Security Defaults** | Enforces MFA for all users (basic) | Free |
| **Conditional Access** | Policy-based, granular control | P1 |
| **Per-user MFA** | Legacy; enable per individual user | Free (not recommended) |

### Security Defaults
- **Enabled by default** on new tenants
- Requires ALL users to register for MFA within 14 days
- Blocks legacy authentication protocols
- Requires MFA for admins on every sign-in
- Requires MFA for all users when necessary (risk-based)
- **Cannot coexist** with Conditional Access policies

### Portal Path — Security Defaults
```
Microsoft Entra ID → Properties → Manage security defaults →
Security defaults: Enabled / Disabled → Save
```

### Portal Path — Per-User MFA (Legacy)
```
Microsoft Entra ID → Users → Per-user MFA →
Select user → Enable / Enforce / Disable
```

> ⚠️ **EXAM TIP:** **Security Defaults** = auto-enabled on new tenants, blocks legacy auth. Must **disable Security Defaults** before using **Conditional Access**. Per-user MFA is **legacy** — use Conditional Access instead.

---

## 9. Conditional Access

- **If-then policy engine** — if user meets condition, then apply access control
- Requires **P1** license
- Evaluates **signals** (user, device, location, app, risk) → applies **controls**

### Conditional Access Policy Components

| Component | Options |
|---|---|
| **Assignments (IF)** | Users/Groups, Cloud apps, Conditions |
| **Conditions** | Sign-in risk, User risk, Device platform, Location, Client app, Device state |
| **Access Controls (THEN)** | Grant (Block / Allow with requirements) or Session controls |
| **Grant controls** | Require MFA, Require compliant device, Require Hybrid Azure AD join, Require approved client app, Require app protection policy, Require password change |
| **Session controls** | App enforced restrictions, Conditional Access App Control, Sign-in frequency, Persistent browser session |
| **Policy state** | On / Off / Report-only |

### Named Locations
- Define **trusted IP ranges** or **country-based locations**
- Used in Conditional Access conditions
- Types: **IP ranges** (IPv4/IPv6 CIDR) or **Countries/regions** (GPS or IP-based)
- Mark as **Trusted location** (skip MFA for trusted locations)

### Portal Path — Create Conditional Access Policy
```
Microsoft Entra ID → Security → Conditional Access → + New policy →
Name →
Assignments: Users (Include/Exclude) → Cloud apps (Include/Exclude) →
Conditions (optional): Sign-in risk, Device platform, Locations, Client apps →
Grant: Block / Grant with controls (Require MFA, Compliant device, etc.) →
Session controls (optional) →
Enable policy: On / Report-only / Off → Create
```

### Portal Path — Named Locations
```
Microsoft Entra ID → Security → Conditional Access → Named locations →
+ IP ranges location / + Countries location →
Name → Mark as trusted → Define ranges/countries → Create
```

### Common Exam Scenarios for Conditional Access

| Scenario | Configuration |
|---|---|
| Require MFA for all users | Users: All users → Apps: All cloud apps → Grant: Require MFA |
| Block legacy authentication | Users: All → Conditions: Client apps = Other clients → Grant: Block |
| Require compliant device for Office 365 | Users: All → Apps: Office 365 → Grant: Require compliant device |
| Exclude break-glass account | Users: All users, Exclude: Break-glass account |
| MFA for admins only | Users: Directory roles (Global Admin, etc.) → Grant: Require MFA |

> ⚠️ **EXAM TIP:** Always **exclude** a **break-glass (emergency) account** from all CA policies. **Report-only** mode = test without enforcement. CA policies are **AND** logic within one policy, **OR** across multiple policies (most restrictive wins). **Block overrides Grant**.

---

## 10. Azure AD Connect / Cloud Sync

### Hybrid Identity — Sync Methods

| Feature | Azure AD Connect | Azure AD Cloud Sync |
|---|---|---|
| **Agent** | Heavy agent on-prem server | Lightweight agent (multiple) |
| **Topology** | Single AD forest → single tenant (most common) | Multi-forest → single tenant |
| **Sync engine** | On-prem sync engine | Cloud-managed |
| **Password Hash Sync** | ✅ | ✅ |
| **Pass-through Auth** | ✅ | ❌ |
| **Federation (AD FS)** | ✅ | ❌ |
| **Password writeback** | ✅ | ✅ |
| **Device writeback** | ✅ | ❌ |
| **Group writeback** | ✅ | ✅ (limited) |
| **HA** | Staging server | Multiple agents |
| **Filtering** | Domain, OU, attribute | OU, attribute |

### Authentication Methods (Hybrid)

| Method | Description | Password in Cloud? |
|---|---|---|
| **Password Hash Sync (PHS)** | Hash of on-prem password synced to Azure AD | Yes (hash) |
| **Pass-through Authentication (PTA)** | Auth validated against on-prem AD in real-time | No |
| **Federation (AD FS)** | Auth redirected to on-prem federation server | No |

- **PHS** = recommended (works even if on-prem is down)
- **PTA** = on-prem validation, requires agent on-prem, no password hash in cloud
- **Federation** = full on-prem control, complex setup

### Sync Cycle
- Default sync interval: **30 minutes**
- Can force sync via PowerShell: `Start-ADSyncSyncCycle -PolicyType Delta`
- Initial sync = full sync; subsequent = delta sync

### Source Anchor
- **Immutable ID** linking on-prem object to cloud object
- Default: `ms-DS-ConsistencyGuid` (recommended) or `ObjectGUID`
- **Cannot change** after initial sync

### Portal Path — Azure AD Connect Health
```
Microsoft Entra ID → Azure AD Connect → Azure AD Connect Health →
View sync status, errors, alerts
```

> ⚠️ **EXAM TIP:** Default sync = **30 minutes**. **PHS** = most resilient (works if on-prem down). **PTA** requires on-prem agent. Synced users (on-prem sourced) can only be managed in **on-prem AD** (not in portal). Source of authority = on-prem AD for synced users.

---

## 11. Device Identity

### Device Registration Types

| Type | Description | MDM Required | Join State |
|---|---|---|---|
| **Azure AD Registered** | Personal (BYOD) devices; user adds work account | Optional | Registered |
| **Azure AD Joined** | Organization-owned devices; sign in with Azure AD account | Yes (Intune) | Joined |
| **Hybrid Azure AD Joined** | Devices joined to both on-prem AD and Azure AD | Optional | Hybrid Joined |

### Portal Path — Device Settings
```
Microsoft Entra ID → Devices → Device settings →
Users may join devices to Azure AD: All / Selected / None →
Users may register devices: All / Selected / None →
Require MFA to register/join: Yes / No →
Max devices per user: 5 / 10 / 20 / 50 / Unlimited (default: 50) →
Save
```

### Device Comparison

| Feature | Registered | Joined | Hybrid Joined |
|---|---|---|---|
| **Ownership** | Personal (BYOD) | Corporate | Corporate |
| **OS** | Windows 10+, iOS, Android, macOS | Windows 10+ | Windows 10+, down-level |
| **Sign-in** | Local/MS account + work account added | Azure AD account | Domain account |
| **SSO to cloud** | ✅ Limited | ✅ Full | ✅ Full |
| **SSO to on-prem** | ❌ | ❌ (unless PHS/PTA) | ✅ |
| **Conditional Access** | ✅ | ✅ | ✅ |
| **Managed by** | User / Intune (optional) | Intune | SCCM / Intune |
| **BitLocker recovery in AD** | ❌ | ✅ | ✅ |

> ⚠️ **EXAM TIP:** **Azure AD Joined** = corporate cloud-only devices. **Hybrid Joined** = corporate with on-prem AD. **Registered** = BYOD. Default max devices per user = **50**. Conditional Access can require **Compliant** or **Hybrid joined** device.

---

## 12. Application & Service Principals

### App Registration vs Enterprise App

| Component | Purpose |
|---|---|
| **App Registration** | Define application identity (app ID, redirect URIs, permissions, secrets/certs) |
| **Enterprise Application** | Instance of the app (service principal) in a tenant for SSO and access control |
| **Service Principal** | Identity for the app in your tenant (created when app is registered or consented) |

### Portal Path — Register an App
```
Microsoft Entra ID → App registrations → + New registration →
Name → Supported account types (single-tenant / multi-tenant / personal) →
Redirect URI (optional) → Register
```

### Portal Path — Enterprise Applications (SSO)
```
Microsoft Entra ID → Enterprise applications → + New application →
Search gallery / Non-gallery → Add →
Configure SSO (SAML / OIDC / Password-based / Linked) →
Assign users/groups
```

### Portal Path — Manage App Secrets / Certificates
```
App registrations → Select app → Certificates & secrets →
+ New client secret → Description, Expiry (6, 12, 24 months, custom) → Add
OR
+ Upload certificate
```

> ⚠️ **EXAM TIP:** **App registration** = global app definition (one per app). **Enterprise Application / Service Principal** = per-tenant instance. Client secrets have **max expiry** configurable by admin. Secrets should be rotated to certificates for security.

---

## 13. External Identities (B2B)

### B2B Collaboration

- Invite **external users** (guests) to your tenant
- Guest users sign in with their **own credentials** (another Entra ID, Microsoft account, Google, email OTP)
- Guest users have **limited permissions** by default
- Guest user licenses: Some features need P1/P2 licensing (ratio: 1:5 — each P1/P2 can support 5 guests)

### External Collaboration Settings

| Setting | Options |
|---|---|
| **Guest user access** | Same as members / Limited / Restricted (most restrictive) |
| **Guest invite restrictions** | Anyone can invite / Members can invite / Only admins / No one |
| **Enable guest self-service sign-up** | Yes / No |
| **Collaboration restrictions** | Allow invitations to any domain / Allow only specific domains / Deny specific domains |

### Portal Path — External Collaboration Settings
```
Microsoft Entra ID → External Identities → External collaboration settings →
Guest user access restrictions → Guest invite restrictions →
Collaboration restrictions (allow/deny domains) → Save
```

### Guest User Default Permissions

| Permission | Guest Default |
|---|---|
| Enumerate users / groups | ❌ Restricted (can see only own profile by default) |
| Read tenant properties | ❌ Limited |
| Register applications | ❌ No |
| Create groups | ❌ No |
| Invite other guests | Depends on setting |

> ⚠️ **EXAM TIP:** Guest licensing ratio = **5 guests per 1 paid license**. Default guest permissions are **restricted** — cannot enumerate directory. Guest invite settings can limit WHO can invite.

---

## 14. Roles & RBAC (Directory Level)

### Azure AD Roles vs Azure RBAC

| Feature | Azure AD Roles (Entra Roles) | Azure RBAC |
|---|---|---|
| **Scope** | Entra ID directory | Azure resources (subscriptions, RGs, resources) |
| **Manages** | Users, groups, apps, devices, policies | VMs, storage, networking, etc. |
| **Assigned via** | Entra ID → Roles | IAM (Access Control) on resources |
| **Examples** | Global Admin, User Admin | Owner, Contributor, Reader |
| **Custom roles** | P1 (custom directory roles) | ✅ Yes |

### Key Directory Roles

| Role | Permissions |
|---|---|
| **Global Administrator** | Full access to everything in Entra ID + can elevate to Azure Subscription access |
| **User Administrator** | Create/manage users and groups, reset passwords, manage licenses |
| **Billing Administrator** | Manage billing, subscriptions, support tickets |
| **Global Reader** | Read everything a Global Admin can, but cannot modify |
| **Helpdesk Administrator** | Reset passwords for non-admins + Helpdesk Admins |
| **Groups Administrator** | Create/manage all groups and group settings |
| **License Administrator** | Manage license assignments |
| **Authentication Administrator** | Set/reset auth methods for non-admin users |
| **Privileged Authentication Administrator** | Set/reset auth methods for ANY user (including admins) |
| **Privileged Role Administrator** | Manage role assignments in Entra ID and PIM |
| **Application Administrator** | Manage all app registrations and enterprise apps |
| **Cloud Application Administrator** | Same as App Admin but CANNOT manage app proxy |
| **Security Administrator** | Read security info + manage security settings, alerts, policies |
| **Security Reader** | Read-only access to security features |
| **Conditional Access Administrator** | Create and manage CA policies |
| **Exchange Administrator** | Manage Exchange Online |
| **SharePoint Administrator** | Manage SharePoint Online |
| **Teams Administrator** | Manage Teams |
| **Intune Administrator** | Manage Intune (device management) |

### Password Reset Hierarchy

| Admin Role | Can Reset Password For |
|---|---|
| **Helpdesk Administrator** | Non-admins, Helpdesk Admins |
| **User Administrator** | Non-admins, Helpdesk, User Admins, some limited admins |
| **Authentication Administrator** | Non-admins (set/reset auth methods) |
| **Privileged Authentication Administrator** | ALL users including Global Admins |
| **Global Administrator** | ALL users |

### Portal Path — Assign Directory Role
```
Microsoft Entra ID → Roles and administrators →
Select role → + Add assignments → Select user(s) → Add
```

> ⚠️ **EXAM TIP:** **Global Admin** can elevate to Azure subscription access via `Properties → Access management for Azure resources`. **Helpdesk Admin** CANNOT reset passwords for Global/User Admins. Only max **5 Global Admins** recommended. **Custom directory roles = P1**.

---

## 15. Identity Protection (P2)

### Risk Types

| Risk | Description | Type |
|---|---|---|
| **Sign-in risk** | Suspicious sign-in activity | Real-time |
| **User risk** | Compromised user account | Offline |

### Risk Levels
- **High** — strong confidence of compromise
- **Medium** — moderate confidence
- **Low** — possible anomaly
- **No risk** — clean

### Risk Detections (Examples)

| Detection | Risk Type |
|---|---|
| Unfamiliar sign-in properties | Sign-in |
| Anonymous IP address | Sign-in |
| Atypical travel | Sign-in |
| Malware-linked IP | Sign-in |
| Password spray | Sign-in |
| Leaked credentials | User |
| Azure AD threat intelligence | Both |

### Risk Policies

| Policy | Trigger | Action |
|---|---|---|
| **Sign-in risk policy** | Sign-in risk level (Low/Medium/High) | Allow, Block, or Require MFA |
| **User risk policy** | User risk level | Allow, Block, or Require password change |

### Portal Path — Identity Protection
```
Microsoft Entra ID → Security → Identity Protection →
Sign-in risk policy → Select risk level → Select controls → Enable → Save
User risk policy → Select risk level → Select controls → Enable → Save
```

> ⚠️ **EXAM TIP:** Identity Protection = **P2 only**. Sign-in risk → require **MFA**. User risk → require **password change**. Works with **Conditional Access** policies.

---

## 16. Privileged Identity Management (PIM) — P2

- **Just-in-time** (JIT) access for privileged roles
- Reduces standing admin access (zero standing access)
- Users **activate** roles when needed (time-bound)

### PIM Key Concepts

| Concept | Description |
|---|---|
| **Eligible** | User CAN activate the role (needs activation) |
| **Active** | Role is currently assigned and usable |
| **Activation** | Process of making eligible role active (can require MFA, approval, justification) |
| **Time-bound** | Active assignment expires after configured duration |
| **Approval workflow** | Designated approver must approve activation |

### PIM Settings per Role

| Setting | Options |
|---|---|
| **Activation max duration** | 0.5 – 24 hours (default: 8 hours) |
| **Require MFA on activation** | Yes / No |
| **Require justification** | Yes / No |
| **Require approval** | Yes / No + select approvers |
| **Allow permanent eligible** | Yes / No |
| **Allow permanent active** | Yes / No |

### Portal Path — PIM
```
Microsoft Entra ID → Identity Governance → Privileged Identity Management →
Azure AD roles → Roles → Select role →
+ Add assignments → Select members → Select type (Eligible / Active) →
Set duration → Assign
```

> ⚠️ **EXAM TIP:** PIM = **P2 only**. **Eligible** = must activate. **Active** = always on. Default max activation = **8 hours**. Can require MFA + justification + approval for activation.

---

## 17. Access Reviews (P2)

- Periodically review user access to resources, groups, roles, apps
- **P2** license required
- Reviewers: Self, Manager, Group owner, Specific users
- Can auto-apply results (remove access if not approved)
- Recurrence: Weekly, Monthly, Quarterly, Semi-annually, Annually

### Portal Path
```
Microsoft Entra ID → Identity Governance → Access reviews →
+ New access review → Select what to review (Groups/Apps/Roles) →
Scope → Reviewers → Recurrence → Settings → Create
```

> ⚠️ **EXAM TIP:** Access Reviews = **P2**. Can auto-remove access for non-responding reviews. Review scope: groups, apps, Azure AD roles, Azure resource roles.

---

## 18. Monitoring & Audit

### Entra ID Logs

| Log | Retention | Content |
|---|---|---|
| **Sign-in logs** | 30 days (Free/P1/P2) | Who signed in, app, location, device, success/failure |
| **Audit logs** | 30 days | Changes to users, groups, roles, apps, policies |
| **Provisioning logs** | 30 days | User provisioning to/from SaaS apps |

### Portal Path — View Logs
```
Microsoft Entra ID → Monitoring → Sign-in logs / Audit logs / Provisioning logs
```

### Export Logs for Longer Retention
```
Microsoft Entra ID → Monitoring → Diagnostic settings →
+ Add diagnostic setting → Select logs → Send to:
Log Analytics / Storage Account / Event Hub → Save
```

> ⚠️ **EXAM TIP:** All Entra ID logs retained **30 days** by default. To keep longer → export via **Diagnostic Settings** to Log Analytics or Storage Account. Sign-in logs require at least **P1** for full features.

---

## 19. CLI & PowerShell Commands

### Azure CLI

```bash
# Create user
az ad user create --display-name "John Doe" \
  --user-principal-name john@contoso.com --password "P@ssw0rd!"

# List users
az ad user list --output table

# Delete user
az ad user delete --id john@contoso.com

# Create group
az ad group create --display-name "Sales Team" --mail-nickname "sales"

# Add member to group
az ad group member add --group "Sales Team" --member-id <user-object-id>

# List groups
az ad group list --output table

# Assign directory role
az rest --method POST \
  --uri "https://graph.microsoft.com/v1.0/roleManagement/directory/roleAssignments" \
  --body '{"roleDefinitionId":"<role-id>","principalId":"<user-id>","directoryScopeId":"/"}'

# Invite guest user
az rest --method POST \
  --uri "https://graph.microsoft.com/v1.0/invitations" \
  --body '{"invitedUserEmailAddress":"guest@external.com","inviteRedirectUrl":"https://portal.azure.com"}'
```

### Azure PowerShell (AzureAD / Microsoft.Graph)

```powershell
# Connect to Entra ID
Connect-MgGraph -Scopes "User.ReadWrite.All", "Group.ReadWrite.All"

# Create user
New-MgUser -DisplayName "Jane Doe" -UserPrincipalName "jane@contoso.com" `
  -AccountEnabled -PasswordProfile @{Password="P@ssw0rd!"; ForceChangePasswordNextSignIn=$true} `
  -MailNickname "jane"

# List all users
Get-MgUser -All

# Delete user
Remove-MgUser -UserId <object-id>

# Create group
New-MgGroup -DisplayName "IT Team" -MailNickname "it" `
  -SecurityEnabled -MailEnabled:$false -GroupTypes @()

# Add member
New-MgGroupMember -GroupId <group-id> -DirectoryObjectId <user-id>

# Force Azure AD Connect sync
Start-ADSyncSyncCycle -PolicyType Delta

# Reset password
$params = @{
  passwordProfile = @{ password = "NewP@ss!"; forceChangePasswordNextSignIn = $true }
}
Update-MgUser -UserId <id> -BodyParameter $params
```

> ⚠️ **EXAM TIP:** PowerShell module `AzureAD` is deprecated → use **Microsoft.Graph** module. Force AD Connect sync = `Start-ADSyncSyncCycle -PolicyType Delta`. Old cmdlets: `New-AzureADUser` → new: `New-MgUser`.

---

## 20. Pricing Key Points

| Component | Cost |
|---|---|
| **Free tier** | Users, groups, SSO (10 apps), Security Defaults, 50K object limit |
| **P1** (~$6/user/month) | Conditional Access, Dynamic Groups, SSPR writeback, AUs, Azure AD Connect Health |
| **P2** (~$9/user/month) | P1 + Identity Protection, PIM, Access Reviews, Entitlement Management |
| **Guest users** | 5 guests per paid P1/P2 license (MAU-based billing also available) |
| **MFA** | Included (Security Defaults = free; CA-based = P1) |

> ⚠️ **EXAM TIP:** Know what features are **Free vs P1 vs P2**. This is frequently tested. Conditional Access = P1. PIM + Identity Protection + Access Reviews = P2.

---

## 21. Quick-Fire Exam Points ⚡

1. **Entra ID tenant** = one per organization; every Azure subscription linked to **exactly one** tenant
2. Azure AD ≠ AD DS — no OUs, no GPOs, no Kerberos/LDAP (uses OAuth/OIDC/SAML)
3. **Free tier** object limit = **50,000**; P1/P2 = no limit
4. **Usage Location** must be set BEFORE assigning licenses
5. **Bulk operations** use CSV files; max **50,000** per operation
6. **Group types**: Security (users, devices, groups, SPs) vs Microsoft 365 (users only)
7. **Dynamic groups** = **P1**; rule processing is **not instant** (can take minutes to hours)
8. Microsoft 365 groups **cannot contain** other groups
9. **SSPR**: Selected = only **one group** (nest if needed); admins always need **2 methods**
10. **Security questions** = SSPR only, **NOT** usable for MFA
11. **Password writeback** = P1 + Azure AD Connect
12. **Security Defaults** = auto-enabled on new tenants; must disable for Conditional Access
13. **Conditional Access** = P1; always exclude **break-glass** account; Block > Grant
14. CA policies: AND within one policy, most restrictive wins across policies
15. **Report-only mode** = test CA policies without enforcement
16. **Named Locations** = IP ranges or countries; can be marked as trusted
17. **PHS** = recommended sync method (resilient; works when on-prem is down)
18. **PTA** = on-prem validation, no password hash in cloud, needs on-prem agent
19. Azure AD Connect sync interval = **30 minutes** (default)
20. Synced users (on-prem sourced) managed **only in on-prem AD** — cannot edit in portal
21. **Source anchor** = `ms-DS-ConsistencyGuid` (default); cannot change post-sync
22. **Azure AD Registered** = BYOD; **Joined** = corporate cloud; **Hybrid Joined** = corporate + on-prem
23. Default max devices per user = **50**
24. **Global Admin** can elevate to Azure subscription via Properties → Access management
25. **Helpdesk Admin** CANNOT reset passwords for Global/User Admins
26. **Privileged Auth Admin** can reset passwords for **ALL** users including Global Admins
27. **Identity Protection** = P2; sign-in risk → MFA; user risk → password change
28. **PIM** = P2; eligible vs active; default max activation = **8 hours**
29. **Access Reviews** = P2; can auto-remove access
30. **Administrative Units** = P1; scope admin delegation to user subsets
31. All Entra ID logs (sign-in, audit, provisioning) retained **30 days** by default
32. Export logs via **Diagnostic Settings** for longer retention
33. **Guest licensing**: 5 guests per 1 P1/P2 license
34. Guests have **restricted** default permissions (cannot enumerate directory)
35. **Custom directory roles** = P1
36. Recommended max Global Admins = **5**
37. App registration = global; Enterprise App / Service Principal = per-tenant
38. Client secret max expiry = configurable; certificates preferred over secrets
39. `New-MgUser` (Microsoft.Graph) replaces deprecated `New-AzureADUser`
40. Force sync: `Start-ADSyncSyncCycle -PolicyType Delta`

---

## 22. Step-by-Step Configuration Mind Maps 🗺️

---

### 22.1 Create a New User

> **Portal:** `Microsoft Entra ID → Users → + New user → Create new user`

```
Create New User
│
├── Step 1: Identity
│   ├── User principal name (username@domain.com)
│   ├── Mail nickname
│   └── Display name (required)
│
├── Step 2: Password
│   ├── Auto-generate (default)
│   └── Custom password
│       ⚠️ User must change at first sign-in (by default)
│
├── Step 3: Properties (optional)
│   ├── First name, Last name
│   ├── Job title, Department, Company
│   ├── Usage location ← ⚠️ Required for license assignment
│   └── Manager
│
├── Step 4: Assignments
│   ├── Add to Groups (optional)
│   └── Assign Directory Roles (optional)
│
├── Step 5: Review + Create
│
├── Required Role: User Administrator or Global Administrator
│
└── ⚠️ Key Points
    ├── UPN must be unique in tenant
    ├── Usage Location required BEFORE license assignment
    └── For bulk create → Users → Bulk operations → Bulk create → CSV upload
```

---

### 22.2 Invite Guest User (B2B)

> **Portal:** `Microsoft Entra ID → Users → + New user → Invite external user`

```
Invite Guest User
│
├── Step 1: Enter email address of external user
│
├── Step 2: Display name
│
├── Step 3: Personal message (optional invitation text)
│
├── Step 4: Assignments
│   ├── Add to Groups (optional)
│   └── Assign Directory Roles (optional)
│
├── Step 5: Review + invite
│
├── Required Role: Guest Inviter / User Admin / Global Admin
│   ⚠️ Or regular members if "Members can invite" is enabled
│
├── Prerequisites
│   └── External collaboration settings allow invitation to user's domain
│
└── ⚠️ Key Points
    ├── Guest signs in with OWN identity provider
    ├── Guest gets limited default permissions
    ├── Licensing: 5 guests per 1 P1/P2 license
    └── Control who can invite: Entra ID → External Identities → External collaboration settings
```

---

### 22.3 Create a Group

> **Portal:** `Microsoft Entra ID → Groups → + New group`

```
Create Group
│
├── Step 1: Group type
│   ├── Security (for RBAC, resource access)
│   └── Microsoft 365 (for collaboration — email, Teams, SharePoint)
│       ⚠️ M365 groups can only contain users (no nested groups)
│
├── Step 2: Group name + Description
│
├── Step 3: Membership type
│   ├── Assigned (manually add members) — Free
│   ├── Dynamic User (auto-populate by user attributes) — ⚠️ P1 required
│   └── Dynamic Device (auto-populate by device attributes) — ⚠️ P1, Security groups only
│
├── Step 4 (if Dynamic): Configure rule
│   ├── Add rule expression
│   │   e.g., (user.department -eq "Sales") -and (user.jobTitle -contains "Manager")
│   └── Validate rules (optional — test with sample users)
│       ⚠️ Dynamic processing is NOT instant (minutes to hours)
│
├── Step 5: Owners (manage group)
├── Step 6: Members (if Assigned type)
│
├── Step 7: Create
│
├── Required Role: Groups Administrator or User Administrator
│
└── ⚠️ Nesting: Security ∈ Security ✅ | M365 ∈ Security ✅ | Groups ∈ M365 ❌
```

---

### 22.4 Configure Self-Service Password Reset (SSPR)

> **Portal:** `Microsoft Entra ID → Password reset`

```
Configure SSPR
│
├── Step 1: Properties
│   └── SSPR enabled: None / Selected (one group) / All
│       ⚠️ "Selected" = only ONE group; nest groups if multiple needed
│
├── Step 2: Authentication methods
│   ├── Number of methods required to reset: 1 or 2
│   └── Methods available:
│       ├── Mobile app notification (Authenticator push)
│       ├── Mobile app code (TOTP)
│       ├── Email
│       ├── Mobile phone (SMS)
│       ├── Office phone
│       └── Security questions
│           ⚠️ Security questions = SSPR only, NOT for MFA
│           ├── Questions required to register: 3–5
│           └── Questions required to reset: 3–5
│
├── Step 3: Registration
│   ├── Require users to register at sign-in: Yes (recommended)
│   └── Re-confirm after: 180 days (default)
│
├── Step 4: Notifications
│   ├── Notify users on password reset: Yes
│   └── Notify admins when other admins reset: Yes
│
├── Step 5: On-premises integration
│   ├── Enable password writeback: Yes/No
│   │   ⚠️ Requires Azure AD Connect + P1
│   └── Allow users to unlock accounts without reset: Yes/No
│
├── Step 6: Save
│
├── Required Role: Global Administrator (or Authentication Policy Admin)
│
└── ⚠️ Admin accounts ALWAYS require 2 methods (cannot be changed)
```

---

### 22.5 Create Conditional Access Policy

> **Portal:** `Microsoft Entra ID → Security → Conditional Access → + New policy`

```
Create Conditional Access Policy
│   ⚠️ Prerequisite: P1 license; Security Defaults MUST be disabled
│
├── Step 1: Name the policy
│
├── Step 2: Assignments (IF)
│   ├── Users
│   │   ├── Include: All users / Select users & groups / Directory roles
│   │   └── Exclude: Specific users/groups
│   │       ⚠️ ALWAYS exclude break-glass (emergency) account
│   │
│   ├── Target resources (Cloud apps or actions)
│   │   ├── Include: All cloud apps / Select apps (Office 365, Azure Management, etc.)
│   │   └── Exclude: Specific apps
│   │
│   └── Conditions (optional)
│       ├── User risk level: High / Medium / Low (⚠️ P2 for risk-based)
│       ├── Sign-in risk level: High / Medium / Low (⚠️ P2 for risk-based)
│       ├── Device platforms: Android, iOS, Windows, macOS, Linux
│       ├── Locations: Named locations (trusted/untrusted IPs or countries)
│       ├── Client apps: Browser, Mobile apps/desktop, Exchange ActiveSync, Other
│       └── Filter for devices: device compliance, trust type, etc.
│
├── Step 3: Access Controls (THEN)
│   ├── Grant
│   │   ├── Block access
│   │   └── Grant access with:
│   │       ├── Require MFA
│   │       ├── Require device to be marked compliant
│   │       ├── Require Hybrid Azure AD joined device
│   │       ├── Require approved client app
│   │       ├── Require app protection policy
│   │       ├── Require password change (needs P2 user risk)
│   │       └── Multiple controls: Require ALL / Require ONE
│   │
│   └── Session (optional)
│       ├── App enforced restrictions
│       ├── Conditional Access App Control
│       ├── Sign-in frequency (e.g., every 1 hour)
│       └── Persistent browser session (Always / Never persistent)
│
├── Step 4: Enable policy
│   ├── On — enforce immediately
│   ├── Report-only — log results without enforcing ⚠️ Test first!
│   └── Off — disabled
│
├── Step 5: Create
│
├── Required Role: Conditional Access Administrator or Security Administrator
│
└── ⚠️ Key Points
    ├── Block overrides Grant (if multiple policies apply)
    ├── Policies are AND within; most restrictive wins across
    └── Test with Report-only before enabling
```

---

### 22.6 Configure Azure AD Connect (Password Hash Sync)

> **On-prem server:** Download Azure AD Connect from Microsoft

```
Configure Azure AD Connect (PHS)
│
├── Prerequisites
│   ├── On-prem AD DS environment
│   ├── Dedicated Windows Server (2016+)
│   ├── Global Administrator account in Entra ID
│   ├── Enterprise Administrator account in on-prem AD
│   ├── .NET 4.7.2+, TLS 1.2, PowerShell 5.0+
│   └── Outbound HTTPS (port 443) to Azure
│
├── Step 1: Download Azure AD Connect from Microsoft
│
├── Step 2: Install and launch wizard
│   ├── Express Settings (recommended for single forest + PHS)
│   └── Custom Settings (for PTA, Federation, filtering, etc.)
│
├── Step 3: Connect to Entra ID
│   └── Enter Global Administrator credentials
│
├── Step 4: Connect to on-prem AD
│   └── Enter Enterprise Admin credentials
│
├── Step 5: Sign-in method
│   ├── Password Hash Synchronization (PHS) ← recommended
│   ├── Pass-through Authentication (PTA)
│   ├── Federation with AD FS
│   └── Do not configure
│
├── Step 6: Domain/OU filtering (what to sync)
│   ⚠️ Default = sync ALL domains and OUs
│
├── Step 7: Optional features
│   ├── Password writeback (needs P1)
│   ├── Group writeback
│   ├── Device writeback
│   └── Azure AD app and attribute filtering
│
├── Step 8: Configure → Install → Complete
│
├── Post-setup
│   ├── Sync runs every 30 minutes (default delta sync)
│   ├── Force sync: Start-ADSyncSyncCycle -PolicyType Delta
│   └── Monitor: Entra ID → Azure AD Connect → Connect Health
│
└── ⚠️ Key Points
    ├── Source anchor (ms-DS-ConsistencyGuid) cannot be changed post-sync
    ├── Synced user = managed in on-prem AD only (not editable in portal)
    └── Do NOT install on a domain controller (not recommended)
```

---

### 22.7 Create Administrative Unit

> **Portal:** `Microsoft Entra ID → Administrative units → + Add`

```
Create Administrative Unit
│   ⚠️ Prerequisite: P1 license
│
├── Step 1: Name and Description
│
├── Step 2: Membership type
│   ├── Assigned (manual)
│   └── Dynamic (P1) — auto-populate based on user attributes
│
├── Step 3: Add Members
│   ├── Users
│   ├── Groups
│   └── Devices
│
├── Step 4: Assign Roles (scoped to this AU)
│   ├── User Administrator
│   ├── Groups Administrator
│   ├── Helpdesk Administrator
│   ├── License Administrator
│   ├── Password Administrator
│   └── Authentication Administrator
│   ⚠️ Admins scoped to AU can ONLY manage objects WITHIN the AU
│
├── Step 5: Review + Create
│
├── Required Role: Privileged Role Administrator or Global Administrator
│
└── ⚠️ AU-scoped admins cannot manage users/groups outside their AU
```

---

### 22.8 Configure Privileged Identity Management (PIM)

> **Portal:** `Microsoft Entra ID → Identity Governance → Privileged Identity Management`

```
Configure PIM
│   ⚠️ Prerequisite: P2 license
│
├── Step 1: Navigate to PIM
│   └── Entra ID → Identity Governance → PIM → Azure AD roles
│
├── Step 2: Select a role (e.g., Global Administrator)
│
├── Step 3: Configure role settings
│   ├── Activation maximum duration: 0.5–24 hours (default 8 hrs)
│   ├── On activation, require: MFA / Justification / Approval
│   ├── Select approvers (if approval required)
│   ├── Allow permanent eligible: Yes/No
│   └── Allow permanent active: Yes/No
│
├── Step 4: Add assignments
│   ├── + Add assignments → Select member(s)
│   ├── Assignment type: Eligible / Active
│   ├── Eligible = must activate when needed (JIT)
│   ├── Active = always-on (use sparingly)
│   └── Set start/end date (or permanent if allowed)
│
├── Step 5: Save
│
├── Required Role: Privileged Role Administrator
│
├── User Activation Flow
│   └── User → PIM → My Roles → Activate →
│       Provide justification → MFA (if required) →
│       Wait for approval (if required) → Role activated for X hours
│
└── ⚠️ Key Points
    ├── Eligible = JIT, must activate; Active = standing access
    ├── PIM audit logs track all activations
    └── Combine with Access Reviews for periodic revalidation (P2)
```

---

### 22.9 Register a Device (Azure AD Join)

> **Portal:** `Microsoft Entra ID → Devices → Device settings`

```
Configure Device Settings & Azure AD Join
│
├── Step 1: Configure device settings
│   └── Entra ID → Devices → Device settings
│       ├── Users may join devices to Azure AD: All / Selected / None
│       ├── Users may register devices: All / Selected / None
│       ├── Require MFA to register/join: Yes / No
│       ├── Max devices per user: 5/10/20/50/Unlimited (default 50)
│       └── Save
│
├── Step 2: Join device (from Windows device)
│   └── Settings → Accounts → Access work or school → Connect →
│       "Join this device to Azure Active Directory" →
│       Enter Azure AD credentials → Join
│
├── Step 3: Verify in portal
│   └── Entra ID → Devices → All devices → Verify device appears
│       Join Type: Azure AD Joined
│
├── Required Role: User (if allowed) or Device Administrator
│
└── ⚠️ Key Points
    ├── Azure AD Joined = corporate-owned, cloud-only
    ├── Enables SSO to cloud resources
    ├── Can be required by Conditional Access (Require compliant / joined device)
    └── Hybrid Join = needs Azure AD Connect device writeback + GPO config
```

---

### 22.10 Export Entra ID Logs to Log Analytics

> **Portal:** `Microsoft Entra ID → Monitoring → Diagnostic settings`

```
Export Entra ID Logs
│
├── Step 1: Navigate
│   └── Microsoft Entra ID → Monitoring → Diagnostic settings
│
├── Step 2: + Add diagnostic setting
│
├── Step 3: Enter Setting Name
│
├── Step 4: Select Log Categories
│   ├── SignInLogs (interactive sign-ins)
│   ├── NonInteractiveUserSignInLogs
│   ├── ServicePrincipalSignInLogs
│   ├── ManagedIdentitySignInLogs
│   ├── AuditLogs
│   ├── ProvisioningLogs
│   ├── RiskyUsers (P2)
│   ├── UserRiskEvents (P2)
│   └── Others as needed
│
├── Step 5: Select Destination
│   ├── Send to Log Analytics workspace
│   ├── Archive to Storage Account
│   └── Stream to Event Hub
│
├── Step 6: Save
│
├── Required Role: Global Administrator or Security Administrator
│
└── ⚠️ Key Points
    ├── Default retention = 30 days in portal
    ├── Log Analytics = query with KQL, create alerts, workbooks
    ├── Storage Account = cheapest for long-term archival
    └── Sign-in logs require at least P1 for full reporting features
```


---

<!-- AUTHOR-WATERMARK: BOTTOM -->
<div align="center">

**© 2026 Satyam Agnihotri (`devSatym`)** · [GitHub](https://github.com/devSatym) · [LinkedIn](https://www.linkedin.com/in/swe1-satyam)

*Original notes by the author — do not republish, mirror, or redistribute without permission.*

</div>
<!-- END AUTHOR-WATERMARK: BOTTOM -->
