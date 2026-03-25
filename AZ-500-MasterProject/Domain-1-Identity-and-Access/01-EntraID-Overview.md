# Microsoft Entra ID Overview (formerly Azure Active Directory)

> 📌 AZ-500 Exam Objective: Manage Microsoft Entra identities
> 🏷️ Domain: 1 — Secure Identity and Access | Weight: 15–20%

***

## What Is This?
Microsoft Entra ID is Microsoft's cloud-based identity service. It stores user accounts, groups, and apps in one place. Every Azure subscription trusts exactly one Entra ID tenant to handle sign-ins and access decisions.

***

## Why Does This Exist?
Before cloud identity services, every company ran their own on-premises servers (Active Directory Domain Services) to manage logins. That required physical servers, manual patching, and VPN tunnels everywhere. Entra ID moves that work to Microsoft's global infrastructure so companies can manage identities from a browser, at any scale, without owning a single server.

***

## Who Actually Needs This? (Real-World Context)
A mid-size hospital network in Texas has 4,000 nurses and doctors. Each person needs access to patient records, scheduling software, and payroll — from laptops, phones, and shared kiosks. The IT team uses Entra ID to create one account per person, group them by department, and control exactly which apps each group can reach. When a nurse resigns, disabling one account in Entra ID cuts off all their access instantly.

***

## How It Works (Pure Concept, No Portal)
- Every Azure customer gets one **tenant** — a private, isolated directory that belongs only to them.
- A tenant holds **users** (people), **groups** (collections of users), **applications** (registered apps), and **devices**.
- **Directory roles** (like Global Administrator) grant control over the tenant itself — not Azure resources.
- Entra ID uses open standards: **OAuth 2.0** for authorization and **OpenID Connect (OIDC)** for authentication. Any app that speaks these standards can integrate with it.
- Entra ID is **not** the same as on-premises Active Directory. It has no domain controllers, no Group Policy Objects, no Kerberos tickets, and no LDAP queries. It is a flat, cloud-native service.
- **License tiers** change what features you get:
  - **Free** — basic user/group management, SSO for up to 10 apps, no conditional access.
  - **P1** — Conditional Access policies, hybrid identity, self-service password reset for on-prem writeback.
  - **P2** — Everything in P1, plus Privileged Identity Management (PIM) and Identity Protection (risk-based sign-in policies).

***

## 2026 Portal Walkthrough — Step by Step

### Create a User
```
portal.azure.com
  → Top search bar → type "Microsoft Entra ID" → select it
  → Left menu → Users
  → + New user → Create new user
  → Fill in: User principal name (e.g., jsmith@contoso.onmicrosoft.com)
  → Fill in: Display name (e.g., Jane Smith)
  → Under Password → auto-generate or set manually
  → Click Review + create → Create
```

### Create a Group and Add a Member
```
portal.azure.com
  → Microsoft Entra ID
  → Left menu → Groups
  → + New group
  → Group type: Security
  → Group name: e.g., "Nursing-Staff"
  → Membership type: Assigned
  → Click Members → + Add members → search for Jane Smith → Select
  → Click Create
```

### CLI Equivalents
```bash
# Create a user
az ad user create \
  --display-name "Jane Smith" \
  --user-principal-name jsmith@contoso.onmicrosoft.com \
  --password "TempP@ssw0rd!" \
  --force-change-password-next-sign-in true

# Create a group
az ad group create \
  --display-name "Nursing-Staff" \
  --mail-nickname "Nursing-Staff"

# Add user to group
az ad group member add \
  --group "Nursing-Staff" \
  --member-id $(az ad user show --id jsmith@contoso.onmicrosoft.com --query id -o tsv)
```

### PowerShell Equivalents
```powershell
# Connect first
Connect-MgGraph -Scopes "User.ReadWrite.All","Group.ReadWrite.All"

# Create a user
$PasswordProfile = @{ Password = "TempP@ssw0rd!"; ForceChangePasswordNextSignIn = $true }
New-MgUser -DisplayName "Jane Smith" `
           -UserPrincipalName "jsmith@contoso.onmicrosoft.com" `
           -PasswordProfile $PasswordProfile `
           -AccountEnabled

# Create a group
New-MgGroup -DisplayName "Nursing-Staff" `
            -MailNickname "Nursing-Staff" `
            -SecurityEnabled `
            -MailEnabled:$false
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/entra/fundamentals/

***

## Key Settings Explained
| Setting | What It Does | When to Use |
|---------|-------------|-------------|
| User Principal Name (UPN) | The sign-in username (email format) | Set at user creation; used for all logins |
| Account enabled toggle | Allows or blocks the user from signing in | Disable immediately when an employee leaves |
| Group membership type | Assigned = manual; Dynamic = rule-based auto-add | Use Dynamic for large orgs with consistent naming |
| Directory role | Grants admin rights inside the tenant | Only assign to dedicated admin accounts |
| License assignment | Activates P1/P2 features for a specific user | Assign P2 to anyone who needs PIM or Identity Protection |
| External collaboration settings | Controls whether guests can be invited | Restrict in regulated industries like finance and healthcare |

***

## Comparison Table
| Feature | Entra ID (Cloud) | Active Directory DS (On-Prem) | Entra ID + AD Connect (Hybrid) |
|---------|----------------|------------------------------|-------------------------------|
| Authentication protocol | OAuth 2.0 / OIDC / SAML | Kerberos / NTLM | Both |
| Group Policy Objects | ❌ Not supported | ✅ Supported | ❌ Not in cloud portion |
| Domain join (traditional) | ❌ (uses Entra Join instead) | ✅ | ✅ (on-prem machines) |
| Self-service password reset | ✅ (P1/P2) | ❌ (requires add-on) | ✅ with writeback |
| Global scale / HA | ✅ Built-in | ❌ Manual setup | Partial |
| Cost | Pay-per-user license | Server + CAL licensing | Both costs |

***

## What Happens If You Skip This?
If you don't use Entra ID properly — for example, sharing one admin account among five people — you lose all accountability. When a breach happens, you cannot tell which person made a change. Attackers who steal that single shared credential get full admin access with no second factor to stop them. On October 2023, a major cloud breach was traced back to a shared credential with no MFA and over-privileged access. The entire tenant was compromised.

***

## AZ-500 Exam Section
- **Trigger Words:** "Azure AD", "Entra ID", "tenant", "directory", "Global Administrator", "user account", "group"
- **Common Traps:**
  - Entra ID Free does NOT support Conditional Access — that requires P1 or higher.
  - Entra ID is NOT the same as Active Directory Domain Services. There are no OUs, GPOs, or Kerberos in Entra ID.
  - A Global Administrator role in Entra ID is NOT the same as Owner role in Azure RBAC. They control different planes.
- **How to Tell Apart from Similar Services:** Entra ID manages cloud identities. AD DS manages on-prem domain-joined machines. AD Connect syncs between the two. Entra Domain Services is a managed AD DS in the cloud (supports LDAP and Kerberos) — different from Entra ID.
- **Likely Question Formats:**
  - "A company wants to give a contractor access to one Azure app without joining their domain. Which feature do you use?" → B2B Guest in Entra ID
  - "Which license is required to use Conditional Access?" → P1 or P2
  - "An employee left the company. How do you immediately block all their access?" → Disable the user account in Entra ID
- **Memorization Tip:** Think of Entra ID as the "airport security checkpoint" for your Azure environment. Every request for access must pass through it. Free = basic metal detector. P1 = adding bag X-ray. P2 = full biometric screening.

***

## Pocket Summary (5-Year Refresh)
- Entra ID is Microsoft's cloud identity platform — it replaces on-premises Active Directory for cloud workloads.
- Every Azure tenant has one Entra ID directory that controls who can sign in and what they can see.
- Free tier is limited; P1 adds Conditional Access; P2 adds PIM and Identity Protection.
- Entra ID uses OAuth 2.0 and OIDC — not Kerberos or LDAP like classic AD.
- Disabling a user account in Entra ID immediately blocks all cloud access across all apps.

---
📚 Further Reading: https://learn.microsoft.com/en-us/entra/fundamentals/
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
