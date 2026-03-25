# Identity Governance and Access Reviews

> 📌 AZ-500 Exam Objective: Implement and manage Microsoft Entra ID Governance
> 🏷️ Domain: 1 — Secure Identity and Access | Weight: 15–20%

***

## What Is This?
Microsoft Entra ID Governance is a set of tools that automates the question: "Does this person still need this access?" It includes access reviews (periodic check-ins on who has what), entitlement management (self-service access request workflows), and lifecycle workflows (automatic actions when employees join, move, or leave a company).

***

## Why Does This Exist?
People change jobs, get promoted, go on leave, or leave the company — but their access permissions often stay the same forever. This "access creep" is one of the most common security findings in enterprise audits. ID Governance solves this by making access time-limited, reviewable, and automatically removed when no longer needed. It moves access management from manual, forgettable tasks to automated, auditable processes.

***

## Who Actually Needs This? (Real-World Context)
A pharmaceutical company in New Jersey must comply with FDA 21 CFR Part 11, which requires strict controls over who can access electronic records. Every quarter, managers must certify that each employee on their team still needs access to the clinical trial data system. Without automation, this is hundreds of email threads and spreadsheets. With Entra ID access reviews, the system automatically sends each manager a task list, shows them which people need recertification, and — if a manager doesn't respond in 14 days — automatically removes access as a safe default.

***

## How It Works (Pure Concept, No Portal)

**Access Reviews:**
- A scheduled review where designated reviewers confirm whether specific users still need access to a group, app, or Azure role.
- **Reviewer options:** the resource owner, a specific person (like a manager), or the users themselves (self-review).
- **Auto-apply results:** when the review period ends, approved access stays; denied or no-response access is automatically removed.
- **Frequency:** one-time, weekly, monthly, quarterly, annually.

**Entitlement Management:**
- Lets users request access packages through a self-service portal (myaccess.microsoft.com).
- An **access package** bundles multiple permissions together — for example: "SharePoint site + Teams channel + Azure DevOps project reader" in one package.
- Access packages have policies: who can request, who approves, how long access lasts.
- When access expires, it is automatically removed.
- Catalogs organize access packages by department or purpose.

**Lifecycle Workflows:**
- Automates tasks triggered by HR events: new hire (joiner), role change (mover), departure (leaver).
- Joiner workflow example: when a new hire's start date arrives, automatically create an account, assign licenses, add to onboarding groups, and send a welcome email.
- Leaver workflow example: when termination date is set in HR, disable account, remove group memberships, revoke tokens, notify manager.
- Workflows use tasks — pre-built actions like "Disable user account," "Send email," "Add user to groups."

**Licensing:** All ID Governance features require **Entra ID P2** or **Entra ID Governance** add-on license.

***

## 2026 Portal Walkthrough — Step by Step

### Create an Access Review for a Group
```
portal.azure.com
  → Top search bar → type "Identity Governance" → select it
  → Left menu → Access reviews
  → + New access review
  → Review name: "Q1 2026 Nursing Staff Access Review"
  → Review type: Teams + Groups
  → Select group: "Nursing-Staff"

  → Reviewers section:
    → Select reviewers: Group owners (or choose specific reviewers)

  → Settings tab:
    → Duration (in days): 14
    → Recurrence: Quarterly
    → If reviewers don't respond: Remove access ← critical setting
    → Auto apply results to resource: ✅ Enabled

  → Review + Create → Create
```

### Create an Access Package (Entitlement Management)
```
portal.azure.com
  → Identity Governance
  → Left menu → Entitlement management → Access packages
  → + New access package
  → Name: "Clinical Trial Team Access"
  → Catalog: General (or create a new catalog)

  → Resource roles tab:
    → + Add resources → select groups, apps, SharePoint sites to bundle

  → Requests tab:
    → Who can request: Specific users and groups (or All members)
    → Require approval: ✅
    → Approver: add the team manager

  → Lifecycle tab:
    → Access expires: After 90 days
    → Number of days before expiration to send notification: 14

  → Review + Create → Create
```

### CLI Equivalent (Access Reviews use Microsoft Graph API)
```bash
# Create an access review via Graph REST API
az rest \
  --method POST \
  --uri "https://graph.microsoft.com/v1.0/identityGovernance/accessReviews/definitions" \
  --headers "Content-Type=application/json" \
  --body '{
    "displayName": "Q1 2026 Nursing Staff Access Review",
    "scope": {
      "query": "/groups/GROUP-OBJECT-ID/members",
      "queryType": "MicrosoftGraph"
    },
    "reviewers": [
      {
        "query": "/groups/GROUP-OBJECT-ID/owners",
        "queryType": "MicrosoftGraph"
      }
    ],
    "settings": {
      "recurrence": {
        "pattern": { "type": "absoluteMonthly", "interval": 3 },
        "range": { "type": "noEnd", "startDate": "2026-01-01" }
      },
      "defaultDecisionEnabled": true,
      "defaultDecision": "removeAccess",
      "autoApplyDecisionsEnabled": true
    }
  }'
```

### PowerShell Equivalent
```powershell
Connect-MgGraph -Scopes "AccessReview.ReadWrite.All"

$reviewParams = @{
    DisplayName = "Q1 2026 Nursing Staff Access Review"
    Scope = @{
        Query = "/groups/GROUP-OBJECT-ID/members"
        QueryType = "MicrosoftGraph"
    }
    Reviewers = @(
        @{
            Query = "/groups/GROUP-OBJECT-ID/owners"
            QueryType = "MicrosoftGraph"
        }
    )
    Settings = @{
        Recurrence = @{
            Pattern = @{ Type = "absoluteMonthly"; Interval = 3 }
            Range = @{ Type = "noEnd"; StartDate = "2026-01-01" }
        }
        DefaultDecisionEnabled = $true
        DefaultDecision = "removeAccess"
        AutoApplyDecisionsEnabled = $true
    }
}

New-MgIdentityGovernanceAccessReviewDefinition -BodyParameter $reviewParams
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/entra/id-governance/

***

## Key Settings Explained
| Setting | What It Does | When to Use |
|---------|-------------|-------------|
| Auto-apply results | Applies review decisions automatically at review end | Always enable — manual application defeats the purpose |
| Default decision: Remove access | If reviewer takes no action, access is revoked | Recommended for security-conscious environments |
| Default decision: Approve | If reviewer takes no action, access stays | Only for low-risk groups where uptime matters more |
| Reviewer: Group owners | Group owner receives and performs the review | Best for small teams where owner knows the members |
| Reviewer: Manager | Each user's manager reviews their access | Good for department-based access tied to HR records |
| Reviewer: Users themselves | Each person self-certifies their own access | Low-friction for large groups with low-risk resources |
| Access package expiration | Access is auto-removed after X days | All temporary and project-based access |
| Lifecycle workflow trigger | Joiner, Mover, or Leaver HR event | Automate day-one provisioning and immediate offboarding |

***

## Comparison Table
| Feature | Access Reviews | Entitlement Management | Lifecycle Workflows | PIM Access Reviews |
|---------|--------------|----------------------|--------------------|--------------------|
| Purpose | Review existing access periodically | Self-service access request + auto-expire | Automate HR-event-based tasks | Review PIM eligible role holders |
| Who it covers | Group members, app assignments, Azure roles | Any access package resource | All employees | PIM role holders only |
| Trigger | Time-based schedule | User request | HR system event (or manual) | Time-based schedule |
| Auto-remove access | ✅ Yes | ✅ On expiration | ✅ On leaver workflow | ✅ Yes |
| License needed | P2 or Governance | P2 or Governance | P2 or Governance | P2 |

***

## What Happens If You Skip This?
Without access reviews, permissions accumulate indefinitely. Studies consistently find that 30–50% of active user accounts in large enterprises hold permissions for systems the user no longer uses. This is called "access sprawl." Each over-permissioned account is a potential pivot point for an attacker. When a former employee's account is later compromised — even months after they left — those lingering permissions let attackers reach systems that should have been off-limits. Access reviews are the automated cleanup crew that prevents this buildup.

***

## AZ-500 Exam Section
- **Trigger Words:** "access review", "entitlement management", "lifecycle", "access package", "recertification", "joiner mover leaver", "stale access", "auto-apply"
- **Common Traps:**
  - **Access reviews in ID Governance ≠ PIM access reviews.** PIM has a separate access review feature specifically for eligible role holders. ID Governance access reviews target group memberships and app assignments. Both exist, both are tested.
  - **Entitlement management is NOT just access reviews.** It is the full self-service request portal with catalogs, access packages, approval workflows, and auto-expiration.
  - **Lifecycle workflows require Entra ID Governance license** (or P2) — they are NOT available in Entra Free or P1.
  - **Default decision matters.** If an exam question asks "what happens to access if a reviewer does not respond," the answer depends on the default decision setting — it can either keep or remove access.
- **How to Tell Apart from Similar Services:** ID Governance is about the ongoing hygiene of access (who has what and should they still have it). PIM is about the activation lifecycle of privileged roles (when is elevated access turned on). Conditional Access is about what happens at sign-in time.
- **Likely Question Formats:**
  - "How do you ensure contractor access to a project group is removed after 90 days without manual intervention?" → Access package with 90-day expiration in Entitlement Management.
  - "A quarterly audit requires managers to confirm each team member's app access. Which tool do you use?" → Access reviews in Identity Governance.
  - "How do you automatically disable an account and revoke tokens when an employee is terminated?" → Lifecycle workflow triggered by leaver event.
- **Memorization Tip:** Think of ID Governance as three phases: **Request it** (Entitlement Management), **Check it** (Access Reviews), **Clean it up** (Lifecycle Workflows). Together they cover the full access lifecycle.

***

## Pocket Summary (5-Year Refresh)
- Access reviews let you schedule periodic check-ins on who still needs access to groups, apps, or Azure roles.
- Auto-apply with "remove access" as the default decision ensures stale access is cleaned up automatically.
- Entitlement management bundles permissions into access packages that users can request and that expire automatically.
- Lifecycle workflows automate joiner/mover/leaver tasks triggered by HR events.
- All features require Entra ID P2 or the Entra ID Governance add-on license.

---
📚 Further Reading: https://learn.microsoft.com/en-us/entra/id-governance/
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
