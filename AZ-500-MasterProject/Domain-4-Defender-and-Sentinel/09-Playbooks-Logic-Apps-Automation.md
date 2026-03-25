# Microsoft Sentinel Playbooks and Logic Apps Automation

> 📌 AZ-500 Exam Objective: Configure automation and playbooks in Microsoft Sentinel
> 🏷️ Domain: 4 — Defender for Cloud and Sentinel | Weight: 30–35%

***

## What Is This?

Playbooks in Microsoft Sentinel are automated response workflows built on Azure Logic Apps. When Sentinel detects a threat and creates an incident, a playbook can automatically take action — send an alert to Teams, disable a user account, block an IP in a firewall, or open a ticket in ServiceNow. Automation rules are simpler, lighter-weight automations that run before playbooks and handle basic incident management tasks without requiring a Logic App.

***

## Why Does This Exist?

A human analyst can only respond to one alert at a time. In a large organization, hundreds of alerts fire every day. Without automation, the SOC is always behind — triaging manually, copying information between tools, and taking minutes or hours to perform basic response actions. Playbooks let Sentinel respond in seconds, 24 hours a day, 7 days a week, without human intervention for routine tasks.

***

## Who Actually Needs This? (Real-World Context)

A retail company called QuickBuy has a SOC team of 4 analysts covering a 24/7 operation. At 2 AM on Black Friday, Sentinel detects a high-severity brute force attack on a service account. No analyst is actively watching the console. An automation rule instantly assigns the incident to the Tier-2 team. A playbook fires simultaneously — it sends a Teams message to the on-call channel with incident details, disables the targeted service account in Entra ID using a managed identity, and creates a P1 ticket in ServiceNow. By the time an analyst wakes up and checks their phone, the account is already locked and a ticket is already open. The threat is contained without human delay.

***

## How It Works (Pure Concept, No Portal)

**Two Layers of Automation**

Sentinel has two distinct automation layers. This is critical to understand for the exam.

**Layer 1 — Automation Rules**
Automation rules are simple if/then conditions that execute instantly when an incident is created or updated. They do not require Azure Logic Apps. They are fast and lightweight.

What automation rules can do:
- Assign an incident to a specific user or team.
- Change the incident severity (e.g., promote from Medium to High based on entity).
- Add a tag to an incident.
- Change the incident status (e.g., auto-close known false positives).
- Suppress incidents that match a known benign pattern.
- Trigger a playbook.

Automation rules run in order (by priority number). They stop executing once an incident matches a rule unless you configure them to continue.

**Layer 2 — Playbooks (Azure Logic Apps)**
A playbook is an Azure Logic App that Sentinel can trigger. Logic Apps are no-code/low-code workflow builders with hundreds of connectors to external services. A playbook can do virtually anything that a Logic App can do.

**Playbook Triggers**
There are three trigger types in Sentinel playbooks:
- **Incident trigger** — runs when a new incident is created or updated. Has access to all incident details, entities, and alerts. Most commonly used.
- **Alert trigger** — runs when a specific analytics rule fires an alert (before incidents are created). Less common.
- **Entity trigger** — runs on a specific entity (user, IP, host) from within the investigation graph. Allows analysts to manually kick off playbooks against specific entities.

**Common Playbook Actions**

| Action | What It Does |
|---|---|
| Send Teams/Slack message | Notifies the on-call team with incident details, a link, and key entities |
| Send email via Outlook or SendGrid | Emails a security contact with incident summary |
| Disable user in Entra ID | Calls Microsoft Graph API to set the account enabled state to false |
| Block IP in Azure Firewall | Adds the attacker's IP to the Azure Firewall deny list |
| Block IP in Palo Alto / Fortinet | Uses the firewall's API to block the IP |
| Create ticket in ServiceNow / Jira | Opens an incident ticket in the ITSM tool with auto-populated fields |
| Get threat intelligence on IP | Queries VirusTotal or Microsoft Threat Intelligence for context |
| Add indicator to TI | Adds the malicious IP/domain to Sentinel's threat intelligence as an IOC |
| Run an Azure Automation runbook | Triggers more complex PowerShell scripts via Azure Automation |
| Change incident status | Closes or reassigns the incident in Sentinel after remediation |

**Managed Identity for Playbooks**
Logic Apps (playbooks) need permission to perform actions on Azure resources and Microsoft APIs. The best practice is to use a **system-assigned managed identity** on the Logic App. The managed identity gets granted the specific RBAC roles it needs — for example:
- `Microsoft Sentinel Responder` role on the Sentinel workspace (to update incidents).
- `User Administrator` role in Entra ID (to disable user accounts).
- `Network Contributor` on the Azure Firewall resource (to modify firewall rules).

Avoid using service principal secrets or hardcoded credentials in playbooks. Managed identities are the secure, credential-free approach.

**Defender for Cloud Automation**
Defender for Cloud also supports workflow automation — you can trigger Logic Apps (playbooks) when a Defender alert is generated or when a recommendation appears. This is similar to Sentinel playbooks but fires specifically on Defender for Cloud events, not Sentinel incidents. It uses the same Logic App infrastructure.

***

## 2026 Portal Walkthrough — Step by Step

**Create an Automation Rule to Auto-Assign Incidents:**
1. In Microsoft Sentinel, click **Automation** in the left menu.
2. Click **Create** → **Automation rule**.
3. Name: `Auto-assign High Severity to Tier2`.
4. Under **Trigger**: Select **When incident is created**.
5. Under **Conditions**: Add condition — **Incident severity → Equals → High**.
6. Under **Actions**:
   - Click **Add action** → **Assign owner**.
   - Select the Tier-2 team group or a specific analyst.
7. Set **Order** to 1 (runs first).
8. Set an optional expiration date.
9. Click **Apply**.

**Create a Playbook to Send Teams Notification:**
1. In Sentinel, click **Automation** → **Playbook templates** tab.
2. Find a template like **Post a message in a Teams channel** and click **Create playbook** to use it as a starting point, OR create from scratch:
3. Click **Create** → **Playbook with incident trigger**.
4. Fill in: Subscription, Resource Group, Playbook name (`notify-teams-high-incident`), Region.
5. Click **Next: Connections** — authorize the Microsoft Teams connection with your account.
6. Click **Review + create** → **Create and open designer**.
7. In the Logic App designer, the trigger is pre-set to **Microsoft Sentinel Incident**.
8. Click **+ New step** → search for **Microsoft Teams** → select **Post a message in a chat or channel**.
9. Configure: Channel = Security Operations channel; Message = Use dynamic content to include **Incident title**, **Severity**, **Description**, **Incident URL**, and **Entities** from the trigger.
10. Add another step: **Microsoft Sentinel - Update incident** → set Status to whatever is appropriate.
11. Click **Save**.
12. Go back to Sentinel → **Automation** → create an automation rule that triggers this playbook when a High-severity incident is created.

**CLI Equivalent:**
```bash
# Create a Logic App (playbook) definition
az logic workflow create \
  --resource-group rg-security \
  --name notify-teams-high-incident \
  --location eastus \
  --definition @playbook-definition.json

# Assign managed identity to the Logic App
az logic workflow identity assign \
  --resource-group rg-security \
  --name notify-teams-high-incident \
  --identity-type SystemAssigned

# Get the managed identity principal ID
az logic workflow show \
  --resource-group rg-security \
  --name notify-teams-high-incident \
  --query "identity.principalId" -o tsv

# Assign Sentinel Responder role to the managed identity
az role assignment create \
  --assignee {principalId} \
  --role "Microsoft Sentinel Responder" \
  --scope "/subscriptions/{subId}/resourceGroups/rg-security/providers/Microsoft.OperationalInsights/workspaces/sentinel-prod-workspace"

# List Logic Apps in a resource group
az logic workflow list \
  --resource-group rg-security \
  --output table

# List automation rules in Sentinel (via REST)
az rest \
  --method GET \
  --url "https://management.azure.com/subscriptions/{subId}/resourceGroups/rg-security/providers/Microsoft.OperationalInsights/workspaces/sentinel-prod-workspace/providers/Microsoft.SecurityInsights/automationRules?api-version=2022-11-01"
```

**PowerShell Equivalent:**
```powershell
# Create a Logic App workflow
$definition = Get-Content "playbook-definition.json" | ConvertFrom-Json
New-AzLogicApp `
  -ResourceGroupName "rg-security" `
  -Name "notify-teams-high-incident" `
  -Location "eastus" `
  -Definition $definition

# Enable system-assigned managed identity on Logic App
$logicApp = Set-AzLogicApp `
  -ResourceGroupName "rg-security" `
  -Name "notify-teams-high-incident" `
  -Identity (New-Object Microsoft.Azure.Management.Logic.Models.ManagedServiceIdentity -Property @{Type = "SystemAssigned"})

# Assign Sentinel Responder role to managed identity
New-AzRoleAssignment `
  -ObjectId $logicApp.Identity.PrincipalId `
  -RoleDefinitionName "Microsoft Sentinel Responder" `
  -Scope "/subscriptions/{subId}/resourceGroups/rg-security/providers/Microsoft.OperationalInsights/workspaces/sentinel-prod-workspace"
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/sentinel/automate-responses-with-playbooks

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Automation rule conditions | Filters which incidents trigger the rule (by severity, tactic, entity, etc.) | Always add conditions — avoid running playbooks on every single incident |
| Automation rule order (priority) | Controls which rules run first when multiple rules match | Set critical rules to run first; suppression rules usually go last |
| Incident trigger vs alert trigger | Incident trigger has full incident context; alert trigger fires before incident creation | Use incident trigger for most playbooks; alert trigger for pre-incident enrichment |
| Managed identity on Logic App | Allows the playbook to call Azure APIs without stored credentials | Always use managed identity — never hardcode secrets or service principal passwords in playbooks |
| Playbook permissions | The managed identity needs specific RBAC roles for each action | Assign minimum required permissions (least privilege) — only the roles the playbook needs |
| Logic App connector authorization | OAuth connections to external services (Teams, ServiceNow, etc.) | Must be authenticated by a user account during setup; use service accounts for production |
| Defender for Cloud workflow automation | Triggers Logic Apps on Defender alerts or recommendations | Use when you want playbooks to fire on Defender events independently of Sentinel |

***

## Comparison Table (if applicable)

| Feature | Automation Rules | Playbooks (Logic Apps) |
|---|---|---|
| Complexity | Simple if/then conditions | Full workflow with multiple steps and external integrations |
| Azure Logic App needed? | No | Yes |
| Speed | Instant — executes synchronously | Near-instant but asynchronous |
| What they can do | Assign, tag, close, suppress incidents; trigger a playbook | Anything a Logic App can do — Teams, email, firewall, ITSM, APIs |
| Credential handling | No credentials needed | Uses managed identity or OAuth connections |
| Best for | First-layer triage and routing | Complex multi-step response actions |
| Triggers | Incident created, incident updated, alert created | Incident, alert, or entity (manual or rule-triggered) |
| Requires Sentinel role | Sentinel Contributor to create | Sentinel Contributor + Logic App Contributor |

***

## What Happens If You Skip This?

Without automation, every alert requires a human analyst to manually respond. At 2 AM, 3 AM, and on holidays, that means delayed response. Mean time to respond (MTTR) stretches from minutes to hours. A compromised account stays active for hours while analysts sleep. Attacker dwell time — the amount of time between breach and detection/containment — increases dramatically. Automation is what turns a reactive security team into a proactive one.

***

## AZ-500 Exam Section

- **Trigger Words:** "playbook", "automation rule", "Logic App", "SOAR", "auto-respond", "notify Teams", "managed identity", "incident trigger", "alert trigger", "entity trigger"
- **Common Traps:**
  - Thinking automation rules and playbooks are the same thing — automation rules are simple if/then logic with no Logic App. Playbooks are full Azure Logic App workflows that can do anything.
  - Thinking playbooks need a service principal with a password — the correct approach is a system-assigned managed identity. No passwords, no rotation, no leakage.
  - Thinking automation rules are only for triggering playbooks — automation rules can assign, tag, change severity, close, and suppress incidents without ever calling a playbook.
  - Thinking Logic Apps and playbooks are different things — a playbook IS a Logic App. Sentinel just adds the Sentinel trigger connector to it.
- **How to Tell Apart:**
  - Automation rule = simple, lightweight, no Logic App needed, instant
  - Playbook = Logic App, full workflow, external integrations, slightly more complex setup
- **Likely Question Formats:**
  - "You need to automatically close all Low-severity incidents that match a known benign pattern without involving a Logic App. What do you use?" → Automation rule with a suppression action.
  - "A playbook needs to disable a user in Entra ID. What authentication method should the Logic App use?" → System-assigned managed identity with User Administrator role.
  - "Which Sentinel trigger type should a playbook use to get full incident context including all linked entities and alerts?" → Incident trigger.
- **Memorization Tip:** Automation rules = **traffic cop** (quick decisions, wave through or stop). Playbooks = **SWAT team** (complex coordinated response). The traffic cop can call in the SWAT team when needed (automation rule triggers a playbook).

***

## Pocket Summary (5-Year Refresh)

- Playbooks are Azure Logic Apps that Sentinel triggers to automate response actions like sending alerts, blocking IPs, or disabling users.
- Automation rules are simpler, lighter-weight automations that run instantly to assign, tag, close, or suppress incidents — no Logic App needed.
- The incident trigger is the most common playbook trigger; it gives the playbook full access to all incident details, alerts, and entities.
- Always use a system-assigned managed identity on the Logic App instead of storing credentials or service principal secrets.
- Automation rules can also trigger playbooks — use automation rules as the first filter and playbooks for the heavy lifting.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/sentinel/automate-responses-with-playbooks
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
