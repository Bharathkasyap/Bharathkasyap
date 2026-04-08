# Microsoft Entra Privileged Identity Management (PIM)

> 📌 AZ-500 Exam Objective: Implement and manage Microsoft Entra Privileged Identity Management
> 🏷️ Domain: 1 — Secure Identity and Access | Weight: 15–20%

***

## What Is This?
Privileged Identity Management (PIM) is a feature in Microsoft Entra ID that controls when high-powered roles are active. Instead of a user permanently holding Global Administrator or Owner rights, PIM lets them be "eligible" — meaning they can request to activate the role when they actually need it. The role turns on for a limited time, then expires automatically.

***

## Why Does This Exist?
Permanent privileged access is a massive security risk. If an admin's account is compromised at 2 AM while they sleep, the attacker has full admin rights with no time limit. PIM changes this model. Admins only have elevated access during the minutes or hours they explicitly need it. This limits the window an attacker can exploit a compromised account. It also creates a clear audit trail of every activation request.

***

## Who Actually Needs This? (Real-World Context)
A financial services company in New York has five people with Global Administrator rights in their Azure tenant. Their security audit reveals these five accounts are always active targets for phishing. Using PIM, the company converts all five from permanent admins to eligible admins. Now those accounts have no elevated rights at rest. When an admin needs to make a tenant-wide change, they open PIM, request activation for four hours, enter their MFA code, and optionally wait for a manager to approve. Every activation is logged with the business justification the admin typed in.

***

## How It Works (Pure Concept, No Portal)

**Two assignment types:**
- **Eligible assignment** — the user can activate the role but it is not active. This is the recommended state for all privileged roles.
- **Active assignment** — the role is always on. Use only for break-glass emergency accounts or service accounts that cannot interactively activate.

**Activation flow:**
1. User opens PIM in the portal or navigates to My Access.
2. User requests role activation — they set a duration and provide a business justification.
3. If the role requires **approval**, a designated approver receives a notification and must approve or deny the request.
4. If MFA is required on activation (it should always be enabled), the user completes MFA.
5. The role becomes active for the specified duration (e.g., 2 hours).
6. After the duration expires, the role deactivates automatically.

**PIM covers two planes:**
- **Entra ID roles** — e.g., Global Administrator, Security Administrator, Privileged Role Administrator.
- **Azure resource roles** — e.g., Owner, Contributor on subscriptions, resource groups, or specific resources.

**Access reviews in PIM:**
- PIM can trigger periodic **access reviews** for eligible role holders.
- A reviewer (or the user themselves) certifies whether the access is still needed.
- If not recertified, access is automatically removed.

**Audit logs:**
- Every activation, approval, and denial is logged.
- Logs are available in Entra ID → PIM → Audit history and can be exported to Azure Monitor or a SIEM.

**Licensing:** PIM requires **Entra ID P2**.

***

## 2026 Portal Walkthrough — Step by Step

### Enable PIM for Your Tenant (First Time)
```
portal.azure.com
  → Top search bar → type "Privileged Identity Management" → select it
  → Overview page → Click "Consent to PIM"
  → Follow the consent prompt → PIM is now active for your tenant
```

### Make a User Eligible for Global Administrator
```
portal.azure.com
  → Privileged Identity Management
  → Left menu → Microsoft Entra roles
  → Roles → search for "Global Administrator" → select it
  → + Add assignments
  → Select member(s) → search for your user → Select
  → Assignment type: Eligible
  → Set start and end dates (or leave as permanent eligible)
  → Next → Assign
```

### Configure Activation Settings (Require MFA + Approval)
```
portal.azure.com
  → Privileged Identity Management
  → Microsoft Entra roles → Roles → Global Administrator
  → Role settings (gear icon)
  → Edit
  → Activation tab:
    → Activation maximum duration: 4 hours
    → On activation, require: Azure MFA ✅
    → Require approval to activate: ✅
    → Approvers: add your Security Admin or manager
  → Update
```

### Activate a Role as an Eligible User
```
portal.azure.com
  → Privileged Identity Management
  → Left menu → My roles
  → Eligible assignments tab
  → Find "Global Administrator" → click Activate
  → Set duration (e.g., 2 hours)
  → Type a reason: "Deploying tenant-wide Conditional Access policy"
  → Activate
  → Complete MFA prompt
  → Wait for approver notification (if approval required)
```

### CLI Equivalent (PIM uses REST API via az rest)
```bash
# List eligible role assignments for a user
az rest \
  --method GET \
  --uri "https://graph.microsoft.com/v1.0/roleManagement/directory/roleEligibilityScheduleInstances?\$filter=principalId eq 'USER-OBJECT-ID'"

# Create an eligible assignment (simplified)
az rest \
  --method POST \
  --uri "https://graph.microsoft.com/v1.0/roleManagement/directory/roleEligibilityScheduleRequests" \
  --headers "Content-Type=application/json" \
  --body '{
    "action": "adminAssign",
    "principalId": "USER-OBJECT-ID",
    "roleDefinitionId": "62e90394-69f5-4237-9190-012177145e10",
    "directoryScopeId": "/",
    "scheduleInfo": {
      "startDateTime": "2026-01-01T00:00:00Z",
      "expiration": { "type": "noExpiration" }
    }
  }'
```

### PowerShell Equivalent
```powershell
Connect-MgGraph -Scopes "RoleEligibilitySchedule.ReadWrite.Directory"

$params = @{
    Action = "adminAssign"
    PrincipalId = "USER-OBJECT-ID"
    RoleDefinitionId = "62e90394-69f5-4237-9190-012177145e10" # Global Admin
    DirectoryScopeId = "/"
    ScheduleInfo = @{
        StartDateTime = "2026-01-01T00:00:00Z"
        Expiration = @{ Type = "noExpiration" }
    }
}

Update-MgRoleManagementDirectoryRoleEligibilityScheduleRequest -BodyParameter $params
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/

***

## Key Settings Explained
| Setting | What It Does | When to Use |
|---------|-------------|-------------|
| Eligible assignment | User can activate role on demand | All admin roles — recommended default |
| Active assignment | Role is always on | Break-glass accounts, service accounts that can't do interactive MFA |
| Activation duration | Maximum hours a role stays active per request | Set to the minimum needed — 1–4 hours for most admin tasks |
| Require MFA on activation | Forces MFA challenge when activating | Always enable for all privileged roles |
| Require approval | A designated approver must approve before role activates | Critical roles like Global Admin, Subscription Owner |
| Require justification | User must type a reason when requesting activation | Audit compliance — keeps a paper trail |
| Assignment expiration | Eligible assignment auto-expires after X days | Temporary contractors, project-based access |
| Access reviews | Periodic review of who holds eligible assignments | Quarterly review cycle for compliance frameworks |

***

## Comparison Table
| Feature | PIM | Static RBAC Assignment |
|---------|-----|----------------------|
| Role active all the time? | ❌ Only during activation window | ✅ Always |
| Requires justification | ✅ Configurable | ❌ No |
| Supports approval workflow | ✅ Yes | ❌ No |
| Audit log of usage | ✅ Full activation history | ⚠️ Basic Azure Activity Log only |
| Reduces standing privilege | ✅ Core purpose | ❌ Standing privilege by design |
| License required | Entra ID P2 | Any (free) |
| Works for Entra roles | ✅ Yes | ✅ Yes |
| Works for Azure resource roles | ✅ Yes | ✅ Yes |

***

## What Happens If You Skip This?
Without PIM, every admin has permanent standing access. Attackers specifically target admin accounts because the payoff is highest. A Global Administrator account sitting with always-on permissions, even when the person is asleep or on vacation, is a 24/7 open door. Security frameworks like CIS Benchmark for Azure, NIST 800-53, and ISO 27001 all specifically call for just-in-time access to privileged functions. Skipping PIM can cause compliance audit failures and dramatically increases the damage an attacker can do with one compromised credential.

***

## AZ-500 Exam Section
- **Trigger Words:** "just-in-time", "eligible", "activate role", "time-limited access", "approval required", "standing privilege", "privileged access"
- **Common Traps:**
  - **PIM does not replace RBAC.** PIM manages WHEN a role is active. RBAC determines WHAT the role allows. Both are required.
  - **Active assignment ≠ eligible assignment.** Active means always on. Eligible means activatable. Never make critical roles active unless there is a specific need.
  - **PIM requires Entra ID P2.** Questions that describe PIM features but ask "what license is needed" — the answer is always P2.
  - **PIM access reviews ≠ Identity Governance access reviews.** PIM has its own access review mechanism for eligible role holders. Identity Governance access reviews target group memberships and app assignments.
- **How to Tell Apart from Similar Services:** PIM is about managing privileged role activation lifecycle. Conditional Access is about controlling sign-in policies. Identity Protection is about detecting compromised identities. They work together but are distinct tools.
- **Likely Question Formats:**
  - "A user should only have Owner access to a subscription during a 2-hour maintenance window. How do you configure this?" → PIM eligible assignment for Azure resource role with 2-hour activation maximum.
  - "How do you ensure every activation of Global Admin is logged with a business reason?" → Configure PIM role settings to require justification on activation.
  - "A contractor needs temporary access to manage VMs for 30 days. Which PIM feature do you use?" → Eligible assignment with an expiration date 30 days out.
- **Memorization Tip:** PIM = "Borrow, don't own." Admin rights are like a library book. You check them out when needed, return them when done, and the librarian (audit log) tracks every checkout.

***

## Pocket Summary (5-Year Refresh)
- PIM converts permanent privileged access into just-in-time, time-limited access.
- Eligible assignment = can activate; Active assignment = always on. Use eligible for everything possible.
- Activation can require MFA, a written justification, and/or manager approval.
- PIM covers both Entra ID directory roles and Azure resource roles.
- Requires Entra ID P2 license.

---
📚 Further Reading: https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
