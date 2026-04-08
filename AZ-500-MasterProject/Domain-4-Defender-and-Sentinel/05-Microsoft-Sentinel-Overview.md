# Microsoft Sentinel Overview

> 📌 AZ-500 Exam Objective: Plan and implement Microsoft Sentinel
> 🏷️ Domain: 4 — Defender for Cloud and Sentinel | Weight: 30–35%

***

## What Is This?

Microsoft Sentinel is a cloud-native SIEM (Security Information and Event Management) and SOAR (Security Orchestration, Automation, and Response) platform. It collects log data from across your environment, detects threats using analytics rules, and helps your security team investigate and respond to incidents — all from one cloud-based console.

***

## Why Does This Exist?

Traditional on-premises SIEM tools like Splunk or IBM QRadar are expensive, hard to scale, and require dedicated hardware. As organizations move to the cloud, they need a SIEM that can scale automatically, ingest petabytes of log data, and connect natively to Azure, Microsoft 365, and third-party services. Sentinel runs entirely in the cloud with no infrastructure to manage and charges based on how much data you ingest.

***

## Who Actually Needs This? (Real-World Context)

A large energy company called PowerGrid Inc. has a security operations center (SOC) with 15 analysts. They receive alerts from Azure Defender, Microsoft 365, on-premises firewalls, and Cisco network devices. Before Sentinel, analysts had to log into five different portals to investigate a single incident. With Sentinel enabled on a central Log Analytics Workspace, all data flows into one place. When Sentinel detects a suspicious login followed by a large data download, it automatically creates an incident, maps it to MITRE ATT&CK tactics, and triggers a playbook that notifies the on-call analyst in Microsoft Teams within two minutes.

***

## How It Works (Pure Concept, No Portal)

**Sentinel as a SIEM**
A SIEM collects and stores security logs from many sources. It runs detection rules on those logs to find threats. When a rule fires, it creates an alert. Sentinel does all of this natively in Azure using Log Analytics as its data store.

**Log Analytics Workspace — The Data Store**
Sentinel does not have its own database. It sits on top of a Log Analytics Workspace. All data ingested into Sentinel is stored in Log Analytics tables. You query that data with KQL (Kusto Query Language). This means Sentinel and Log Analytics are tightly coupled — you cannot use Sentinel without a Log Analytics Workspace.

Best practices for the workspace:
- Use a dedicated Log Analytics Workspace for Sentinel (do not mix Sentinel data with operational monitoring data).
- Place the workspace in the same region as your primary data sources to minimize egress costs.
- Use a single workspace for most organizations — split workspaces only for strict data sovereignty requirements.

**Sentinel as a SOAR**
A SOAR automates the response to threats. In Sentinel, this is done through:
- **Automation rules** — simple if/then logic (if incident severity is High, assign to Tier-2 team).
- **Playbooks** — Azure Logic Apps that run automated workflows (if a high-severity incident is created, send a Teams message, block the IP in the firewall, and open a ticket in ServiceNow).

**Pricing Model**
Sentinel charges based on data ingestion into the Log Analytics Workspace. There are two pricing options:
- **Pay-as-you-go** — charged per GB of data ingested per day. Best for low-volume environments.
- **Commitment tiers** — a fixed daily GB commitment (e.g., 100 GB/day) at a discounted per-GB rate. Best for high-volume production SOCs.

Note: Some Microsoft data sources are free to ingest into Sentinel — including Microsoft Entra ID audit logs, Microsoft 365 Defender alerts, Azure Activity logs, and Microsoft Defender for Cloud alerts (up to a limit).

**Sentinel Roles (RBAC)**
Sentinel uses Azure RBAC roles. Built-in roles specific to Sentinel:

| Role | What They Can Do |
|---|---|
| Microsoft Sentinel Reader | View incidents, logs, workbooks. Cannot modify anything. |
| Microsoft Sentinel Responder | Everything Reader can do, plus manage incidents (assign, close, comment). |
| Microsoft Sentinel Contributor | Everything Responder can do, plus create/edit analytics rules, workbooks, playbooks, and data connectors. |
| Microsoft Sentinel Automation Contributor | Used by automation to run playbooks. Not for humans. |

You also need Log Analytics Contributor or Reader role on the workspace for full access to the log data.

**Microsoft Sentinel vs Defender for Cloud — They Are Different**
This is one of the most common points of confusion on the AZ-500 exam:

| | Defender for Cloud | Microsoft Sentinel |
|---|---|---|
| Primary role | Posture management + workload threat detection | SIEM/SOAR — collects, detects, investigates, responds |
| Generates alerts? | ✅ Yes — fires alerts about your Azure resources | Consumes alerts from Defender + other sources |
| Investigates incidents? | ❌ Limited | ✅ Yes — full incident investigation graph |
| Automates response? | ❌ Limited (basic export) | ✅ Yes — playbooks, automation rules |
| Data sources | Azure resources only (plus multi-cloud with connectors) | Azure + M365 + on-prem + third-party + any log source |
| Underlying store | Azure Resource Graph | Log Analytics Workspace |

They are complementary. Defender for Cloud generates alerts → those alerts feed into Sentinel → Sentinel correlates them into incidents → Sentinel triggers playbooks to respond.

***

## 2026 Portal Walkthrough — Step by Step

**Enable Sentinel on a New Log Analytics Workspace:**
1. Go to **Microsoft Sentinel** in the Azure Portal search bar.
2. Click **Create Microsoft Sentinel**.
3. Click **Create a new workspace**.
4. Fill in: Subscription, Resource Group, Name (e.g., sentinel-prod-workspace), Region.
5. Click **Review + Create** → **Create**.
6. Once the workspace is created, you are returned to the Sentinel workspace selection.
7. Select your new workspace.
8. Click **Add**.
9. Sentinel is now enabled on the workspace. The overview dashboard loads.

**Explore the Overview Dashboard:**
1. The Overview page shows tiles for: Events and Alerts over time, Open Incidents, Active Automation rules, and Recent incidents.
2. Click **Incidents** in the left menu to see all current open incidents.
3. Click **Logs** to open the Log Analytics query editor and run KQL queries.
4. Click **Data connectors** to see which log sources are connected.
5. Click **Analytics** to see detection rules that are running.
6. Click **Workbooks** to view dashboards and reports.

**CLI Equivalent:**
```bash
# Create a Log Analytics Workspace for Sentinel
az monitor log-analytics workspace create \
  --resource-group rg-security \
  --workspace-name sentinel-prod-workspace \
  --location eastus \
  --sku PerGB2018

# Enable Microsoft Sentinel on the workspace
az sentinel onboarding-state create \
  --resource-group rg-security \
  --workspace-name sentinel-prod-workspace \
  --sentinel-onboarding-state-name default

# List Sentinel onboarding state
az sentinel onboarding-state list \
  --resource-group rg-security \
  --workspace-name sentinel-prod-workspace

# View workspace details
az monitor log-analytics workspace show \
  --resource-group rg-security \
  --workspace-name sentinel-prod-workspace \
  --output table
```

**PowerShell Equivalent:**
```powershell
# Create a Log Analytics Workspace
New-AzOperationalInsightsWorkspace `
  -ResourceGroupName "rg-security" `
  -Name "sentinel-prod-workspace" `
  -Location "eastus" `
  -Sku "PerGB2018"

# Enable Sentinel (requires Az.SecurityInsights module)
Set-AzSentinelOnboardingState `
  -ResourceGroupName "rg-security" `
  -WorkspaceName "sentinel-prod-workspace"

# Get workspace ID (needed for connector configuration)
$workspace = Get-AzOperationalInsightsWorkspace `
  -ResourceGroupName "rg-security" `
  -Name "sentinel-prod-workspace"
$workspace.CustomerId
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/sentinel/overview

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Log Analytics Workspace | Underlying data store for all Sentinel logs and alerts | Required — Sentinel cannot function without it |
| Data retention period | How long logs are kept in the hot tier before moving to archive | Set based on compliance requirements (many regulations require 90–365 days minimum) |
| Commitment tier pricing | Fixed daily GB rate at a discount vs pay-per-GB | Enable when ingesting more than ~100 GB/day to save costs |
| Sentinel RBAC roles | Controls what Sentinel users can view and modify | Assign Reader to junior analysts, Responder to SOC analysts, Contributor to security engineers |
| Daily cap on workspace | Limits maximum data ingestion per day to control costs | Use in dev/test environments — avoid in production (you might miss critical logs) |
| Workspace region | Physical location of data storage | Choose the same region as your primary data sources; consider data residency requirements |

***

## Comparison Table (if applicable)

| Feature | On-Premises SIEM (Splunk, QRadar) | Microsoft Sentinel |
|---|---|---|
| Infrastructure | Requires dedicated servers and storage | Fully cloud-managed, no servers |
| Scaling | Manual — add hardware to scale | Automatic — scales with Azure |
| Data ingestion | License- or hardware-limited | Unlimited (pay per GB) |
| Setup time | Days to weeks | Hours |
| Built-in connectors | Hundreds of connectors, many require configuration | 200+ native connectors including all Microsoft services |
| Cost model | Upfront license + hardware + maintenance | Pay-per-GB or commitment tier (OpEx, no CapEx) |
| AI/ML detection | Third-party add-ons typically | Built-in UEBA, Fusion, and anomaly detection |

***

## What Happens If You Skip This?

Without a SIEM like Sentinel, your security team has no single place to investigate threats. Logs from Azure, Microsoft 365, firewalls, and servers live in separate systems. When a multi-stage attack happens — someone steals credentials, logs in from an unusual location, moves laterally, and exfiltrates data — there is no way to connect those dots across systems. Mean time to detect (MTTD) grows from hours to days or weeks. By the time the breach is discovered, the attacker has been inside for months.

***

## AZ-500 Exam Section

- **Trigger Words:** "SIEM", "SOAR", "Log Analytics Workspace", "Sentinel", "security operations", "incident", "data ingestion", "KQL", "playbook", "automation rule"
- **Common Traps:**
  - Thinking Defender for Cloud and Sentinel are the same tool — Defender for Cloud generates alerts about your Azure resources; Sentinel aggregates alerts from many sources, correlates them into incidents, and helps you investigate and respond.
  - Thinking Sentinel has its own database — it uses Log Analytics Workspace as its data store. No workspace = no Sentinel.
  - Thinking Sentinel is free — data ingestion into Sentinel is billed by GB. Some Microsoft data sources have free ingestion limits.
- **How to Tell Apart:**
  - Defender for Cloud = protection for your Azure resources (posture + alerts)
  - Sentinel = the SOC brain — collects everything, detects patterns, manages incidents
- **Likely Question Formats:**
  - "Where does Microsoft Sentinel store its log data?" → Log Analytics Workspace.
  - "A SOC analyst needs to view and close incidents in Sentinel but should not be able to modify detection rules. Which role should they have?" → Microsoft Sentinel Responder.
  - "A company wants to ingest 500 GB of log data per day into Sentinel at the lowest cost. What pricing model should they use?" → Commitment tier.
- **Memorization Tip:** Sentinel = the **brain** of your SOC. It collects all signals, detects patterns, and coordinates the response. Defender for Cloud = one of the **sensors** feeding data into that brain.

***

## Pocket Summary (5-Year Refresh)

- Microsoft Sentinel is a cloud-native SIEM + SOAR that collects logs, detects threats, manages incidents, and automates responses.
- Sentinel stores all data in a Log Analytics Workspace — you must create the workspace first before enabling Sentinel.
- Defender for Cloud generates security alerts; Sentinel ingests those alerts and correlates them into incidents.
- Pricing is per-GB of data ingested — use commitment tiers for high-volume production environments to reduce costs.
- Sentinel RBAC has three main roles: Reader (view only), Responder (manage incidents), Contributor (build and configure).

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/sentinel/overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
