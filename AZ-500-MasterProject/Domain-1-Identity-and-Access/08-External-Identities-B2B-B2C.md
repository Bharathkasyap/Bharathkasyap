# Microsoft Entra External Identities — B2B and B2C

> 📌 AZ-500 Exam Objective: Implement and manage external identities
> 🏷️ Domain: 1 — Secure Identity and Access | Weight: 15–20%

***

## What Is This?
Microsoft Entra External Identities is the umbrella term for two different products. **B2B (Business-to-Business)** lets you invite users from partner organizations into your own tenant as guest accounts. **B2C (Business-to-Consumer)** is a completely separate identity platform you use to build sign-in and sign-up experiences for your customers — people who use Gmail, Facebook, or any email address.

***

## Why Does This Exist?
Organizations constantly need to work with people who are not their own employees. B2B solves the partner and vendor problem: a law firm needs to share documents with their client's employees, but those clients already have Microsoft 365 accounts. B2C solves the customer problem: a retailer builds a shopping app and needs millions of consumers to create accounts using whatever email or social login they prefer. Both scenarios need secure identity management, but the requirements are completely different — so they are separate products.

***

## Who Actually Needs This? (Real-World Context)
**B2B scenario:** A defense contractor in Virginia regularly works with three subcontractors. Each subcontractor has its own Microsoft 365 tenant. The prime contractor invites key contacts from each subcontractor as B2B guests. Those guests sign in with their existing corporate credentials from their own employer — no new password to create — and access only the SharePoint sites and Teams channels they are invited to. The prime contractor controls what guests can see. The subcontractor controls how their own employees authenticate.

**B2C scenario:** The same defense contractor's commercial arm builds a public-facing portal for veterans to apply for services. Veterans sign in with their personal Gmail, Outlook, or phone number. The application uses Azure AD B2C as its customer identity platform. Millions of customers can register, sign in, reset passwords, and update profiles — all without affecting the corporate Entra ID tenant.

***

## How It Works (Pure Concept, No Portal)

**B2B (Collaboration) — Key Concepts:**
- A guest user is created in YOUR tenant with the type "Guest." Their home tenant handles their actual authentication.
- Guests use their existing credentials from their employer's tenant, Microsoft account, or a one-time passcode sent to their email.
- You control what guests can access in your tenant using standard RBAC and group memberships.
- **B2B Direct Connect** is a newer option: instead of creating guest objects in your tenant, you configure a direct trust between two tenants. Users from the partner tenant can access your shared channels directly as if they were internal — but there is no guest object created. Currently limited to Microsoft Teams shared channels.
- **Cross-tenant access settings** control inbound (who from other tenants can access yours) and outbound (where your users can go as guests) policies. You can allow, block, or require MFA/device compliance from specific partner tenants.
- **External collaboration settings** are the tenant-wide defaults: can anyone invite guests? What do guests default to being able to see in the directory?

**B2C (Business to Consumer) — Key Concepts:**
- Azure AD B2C is a **completely separate Azure resource** — you create a dedicated B2C tenant, separate from your corporate tenant.
- It supports **local accounts** (email + password) and **social/external identity providers** (Google, Facebook, Apple, SAML, OIDC-compatible IdPs).
- You define **user flows** (pre-built templates for sign-up, sign-in, profile edit, password reset) or **custom policies** using the Identity Experience Framework for advanced scenarios.
- B2C stores customer attributes (name, email, custom attributes like loyalty ID) in its own directory — completely isolated from your corporate Entra ID.
- Scales to hundreds of millions of users.
- Does NOT support Azure RBAC, PIM, or most Entra ID features — it is a standalone customer identity platform.

**When to use which:**
| Scenario | Use |
|---------|-----|
| Invite a partner employee to access your SharePoint | B2B |
| Share a Teams channel with a vendor without creating guest accounts | B2B Direct Connect |
| Build a sign-up/sign-in flow for your mobile app's customers | B2C |
| Allow customers to log in with Google or Facebook | B2C |
| Prevent guests from inviting other guests | External collaboration settings |

***

## 2026 Portal Walkthrough — Step by Step

### Invite a B2B Guest User
```
portal.azure.com
  → Top search bar → type "Microsoft Entra ID" → select it
  → Left menu → Users
  → + New user → Invite external user
  → Email: partneruser@vendorcorp.com
  → Display name: Alex Rivera (Vendor Corp)
  → Personal message (optional): "Welcome to the Contoso partner portal"
  → Invite

  → The guest receives an email. They click "Accept invitation."
  → They are redirected to your tenant portal using their own credentials.
  → Their account appears in your Users list with Type = Guest
```

### Configure External Collaboration Settings
```
portal.azure.com
  → Microsoft Entra ID
  → Left menu → External Identities → External collaboration settings
  → Guest invite settings:
    → "Only users assigned to specific admin roles can invite guest users" ← most restrictive
  → Guest user access restrictions:
    → "Guest users have limited access to properties and memberships" ← recommended
  → Collaboration restrictions:
    → Allow invitations only to specified domains: add your partner domains
  → Save
```

### Configure Cross-Tenant Access for a Specific Partner
```
portal.azure.com
  → Microsoft Entra ID → External Identities
  → Cross-tenant access settings
  → Organizational settings tab → + Add organization
  → Enter partner tenant domain or tenant ID → Add
  → Click the row for the partner → Inbound access
    → B2B collaboration: Allow specific users/groups from their tenant
    → Trust settings: Trust their MFA claims (so guests don't get double MFA prompts)
  → Save
```

### CLI Equivalents
```bash
# Send a B2B invitation to an external user
az ad invitation create \
  --invited-user-email-address "partneruser@vendorcorp.com" \
  --invite-redirect-url "https://portal.azure.com" \
  --invited-user-display-name "Alex Rivera"

# List all guest users in the tenant
az ad user list \
  --filter "userType eq 'Guest'" \
  --query "[].{Name:displayName, UPN:userPrincipalName, State:externalUserState}" \
  --output table
```

### PowerShell Equivalent
```powershell
Connect-MgGraph -Scopes "User.Invite.All"

# Invite a B2B guest
$invitation = @{
    InvitedUserEmailAddress = "partneruser@vendorcorp.com"
    InviteRedirectUrl = "https://portal.azure.com"
    InvitedUserDisplayName = "Alex Rivera"
    SendInvitationMessage = $true
}

New-MgInvitation -BodyParameter $invitation
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/entra/external-id/

***

## Key Settings Explained
| Setting | What It Does | When to Use |
|---------|-------------|-------------|
| Guest invite settings | Controls who in your org can send B2B invitations | Restrict to admins only in regulated industries |
| Guest user access restrictions | Limits what guests can see in your directory | Always restrict — guests should not enumerate all users/groups |
| Cross-tenant access: inbound | Controls what users from other tenants can do in your tenant | Set per-partner; enforce device compliance or trust their MFA |
| Cross-tenant access: outbound | Controls where YOUR users can go as guests | Block outbound to unapproved external tenants |
| Collaboration restrictions | Allowlist or blocklist specific partner domains | Enforce partner governance — only invite from approved companies |
| B2C user flow | Pre-built sign-up/sign-in template for customer apps | Use for standard scenarios; custom policies for complex ones |
| B2C identity providers | Social logins (Google, Facebook) or SAML/OIDC providers | Add to user flows so customers can use existing accounts |

***

## Comparison Table
| Feature | B2B (Collaboration) | B2B Direct Connect | B2C |
|---------|--------------------|--------------------|-----|
| Who is the user? | Partner / vendor employee | Partner employee | End customer / consumer |
| Guest object in your tenant? | ✅ Yes | ❌ No | ❌ No (separate tenant) |
| User's home for credentials | Their employer's IdP | Their employer's IdP | B2C tenant (or social IdP) |
| Use case | Shared resources, apps | Teams shared channels | Customer-facing apps |
| Tenant type | Your corporate tenant | Your corporate tenant | Separate B2C tenant |
| Supports Azure RBAC | ✅ Yes | ✅ Limited | ❌ No |
| Scale | Thousands | Thousands | Millions |
| License cost | Free (up to 50,000 MAU) | Entra ID P1+ | Per MAU pricing |

***

## What Happens If You Skip This?
Without B2B guest controls, organizations fall back to creating full internal accounts for external users. A contractor gets an employee account, is added to "all staff" distribution lists, and can see the full internal directory. When the contract ends and IT forgets to remove the account — which happens often — the former contractor retains access indefinitely. Without external collaboration restrictions, any employee can invite anyone from any company, and those guests can then invite others. In documented cases, this "guest sprawl" has allowed corporate espionage by malicious insiders who invited unauthorized third parties.

***

## AZ-500 Exam Section
- **Trigger Words:** "guest user", "B2B", "B2C", "partner access", "external collaboration", "cross-tenant", "invite", "consumer identity", "social login"
- **Common Traps:**
  - **B2B ≠ B2C.** This is the most tested distinction. B2B is for partner/employee federation within your corporate tenant. B2C is a completely separate product for customer-facing applications. They do not share infrastructure, licensing, or feature sets.
  - **B2B guests use their home tenant's credentials.** You do not manage their passwords. If their account is disabled in their home tenant, they lose access to your resources automatically.
  - **B2B Direct Connect does not create a guest object.** A question about "granting access without creating a guest account" points to B2B Direct Connect, not standard B2B.
  - **External collaboration settings ≠ cross-tenant access settings.** External collaboration settings are your tenant's defaults for guest invitations and guest behavior. Cross-tenant access settings are per-partner trust configurations.
- **How to Tell Apart from Similar Services:** B2B is Entra ID → External Identities (same tenant, guest objects). B2C is a standalone Azure service (separate tenant, consumer accounts). Entra External ID is the new name for the combined external identity platform, converging B2B and B2C features in a new tenant type.
- **Likely Question Formats:**
  - "A partner company's employees need to access your SharePoint. They already have Microsoft 365 accounts. What is the simplest solution?" → B2B guest invitations.
  - "You are building a mobile app for retail customers who want to sign in with Google. Which product do you use?" → Azure AD B2C.
  - "How do you prevent any employee from inviting guests from unrelated companies?" → Set Guest invite settings to admin-only, and configure Collaboration restrictions with an allowed-domains list.
- **Memorization Tip:** B2B = **B**ring your own work badge (guests use their employer's identity). B2C = **B**uild your own customer login (you host the identity platform for consumers). Different badge systems, different buildings.

***

## Pocket Summary (5-Year Refresh)
- B2B Collaboration lets you invite external partner users as guests into your corporate tenant; they sign in with their own employer credentials.
- B2C is a completely separate tenant type for customer-facing apps supporting millions of users with social login options.
- B2B Direct Connect enables cross-tenant access without creating guest objects — currently used for Teams shared channels.
- Always configure External Collaboration Settings to restrict who can invite guests and which domains are allowed.
- Cross-tenant access settings let you set per-partner trust policies for MFA and device compliance — so guests from a trusted partner are not double-prompted.

---
📚 Further Reading: https://learn.microsoft.com/en-us/entra/external-id/
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
