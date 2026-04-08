# Microsoft Sentinel Workbooks and Threat Dashboards

> 📌 AZ-500 Exam Objective: Monitor and visualize security data using Microsoft Sentinel workbooks
> 🏷️ Domain: 4 — Defender for Cloud and Sentinel | Weight: 30–35%

***

## What Is This?

Sentinel Workbooks are interactive, KQL-powered dashboards built on Azure Monitor Workbooks. They turn raw log data into visual charts, tables, and maps that security teams can use to monitor threats, track trends, and present security posture to leadership. They are live — every time you open one, it queries the latest data.

***

## Why Does This Exist?

Raw KQL query results in a table are hard to interpret quickly. A security dashboard that shows a map of sign-in locations, a bar chart of alert trends, and a table of top risky users lets an analyst understand the situation in seconds. Workbooks make that possible without requiring a separate BI tool. They live inside Sentinel and query the same Log Analytics data everything else uses.

***

## Who Actually Needs This? (Real-World Context)

A CISO at an insurance company called SafeGuard Holdings needs a weekly executive briefing on cloud security. Their security engineer builds a custom Sentinel workbook that shows: total incidents this week vs last week, top 5 attacked resources, sign-in failures by geography, and Defender for Cloud Secure Score trend. The workbook is shared with the CISO's Azure Dashboard. Every Monday morning, the CISO opens the dashboard and gets the full security picture in under two minutes — no manual reports, no spreadsheets.

***

## How It Works (Pure Concept, No Portal)

**What Workbooks Are**
Sentinel Workbooks use the Azure Monitor Workbook framework. Each workbook is a collection of components — text blocks, KQL queries rendered as charts or tables, parameter dropdowns, and time range selectors. Everything is driven by KQL queries running against the Log Analytics Workspace in real time.

**Built-In Workbooks**
Sentinel ships with over 100 built-in workbooks covering common use cases. You do not need to build from scratch. Examples:

| Workbook | What It Shows |
|---|---|
| Azure AD Audit Logs | Directory changes, role assignments, app registrations over time |
| Azure AD Sign-in Analysis | Sign-in success/failure trends, risky sign-ins, MFA usage |
| Azure Activity | Subscription-level operations, resource creation/deletion trends |
| Security Events | Windows Security Event frequency, logon types, top failing accounts |
| Office 365 | Teams, SharePoint, Exchange activity and anomalies |
| Microsoft Defender for Cloud | Secure Score trend, alert severity distribution, recommendation status |
| Threat Intelligence | IOC matches, threat feed coverage, indicator types |
| UEBA (User and Entity Behavior Analytics) | Anomalous user behavior, entity risk scores, investigation timelines |
| Insecure Protocols | Legacy auth usage (Basic auth, NTLMv1) that should be blocked |

**Custom Workbook Creation**
You can build custom workbooks from scratch or by editing a built-in one. A custom workbook lets you:
- Pick exactly which KQL queries to show.
- Combine data from multiple tables in one dashboard.
- Add parameter controls (time range picker, subscription filter, severity dropdown).
- Mix text, charts, tables, maps, and tiles in one page.
- Save it to the Sentinel workspace so the whole team can use it.

**Pinning to Azure Dashboard**
Any workbook component (a chart, a table, a tile) can be pinned to an Azure Dashboard. This lets you build an executive dashboard in the Azure portal home page that shows live security data at a glance without opening Sentinel at all.

**Sharing Workbooks**
Workbooks are stored as Azure resources in the resource group where Sentinel lives. To share a workbook:
- Save it in the shared gallery (visible to everyone with access to the Sentinel workspace).
- Export it as an ARM template to deploy to other workspaces.
- Assign viewers the Log Analytics Reader role so they can run the queries.

**Defender for Cloud Workbooks**
Defender for Cloud also has its own workbook gallery — accessible from Defender for Cloud → Workbooks. These focus on security posture, regulatory compliance trends, and Defender coverage. They use the same Azure Monitor Workbook engine.

**Threat Intelligence Workbook**
The TI workbook shows threat indicators imported into the `ThreatIntelligenceIndicator` table. It visualizes: indicator types (IP, domain, file hash), confidence scores, expiration dates, and match activity (how many times an IOC matched your logs).

**UEBA Workbook (Entity Behavior Analytics)**
UEBA in Sentinel builds a baseline of normal behavior for each user and device. The UEBA workbook shows:
- Entity risk scores (which users or hosts are behaving anomalously).
- Peer group comparisons (is this user acting differently than others in the same department?).
- Activity timelines for risky entities.
- Anomaly counts over time.

UEBA must be enabled separately in Sentinel settings before the workbook has data.

***

## 2026 Portal Walkthrough — Step by Step

**Open a Built-In Workbook (Azure AD Sign-in Analysis):**
1. In Microsoft Sentinel, click **Workbooks** in the left menu.
2. Click the **Templates** tab.
3. Search for **Azure AD Sign-in Analysis** (or browse the gallery).
4. Click the workbook tile, then click **View template** to preview it.
5. Click **Save** to save it to your workspace so you can edit it.
6. After saving, click **View saved workbook** to open the live version.
7. Use the time range picker at the top to change the time window.
8. Interact with the charts — click a bar or point to filter the rest of the workbook.

**Create a Custom Workbook with a KQL Chart:**
1. In Workbooks, click **+ New** (top left).
2. Click **Add** → **Add query**.
3. In the query editor, enter a KQL query — for example, sign-in failures grouped by hour.
4. Set **Visualization** to **Bar chart** or **Time chart**.
5. Click **Run Query** to preview the chart.
6. Click **Done editing** on the component.
7. Click **Add** → **Add text** to add a title or description.
8. Click **Save** — give it a name and save to your resource group.
9. The workbook now appears under the **My workbooks** tab.

**Share the Workbook:**
1. Open your saved workbook.
2. Click the **Share** button at the top.
3. Choose **Share to gallery** to make it visible to all Sentinel users on the workspace.
4. Or click **Export ARM template** to download the workbook JSON for deployment to other workspaces.
5. To pin a component to a dashboard: click **Pin** (pushpin icon) on any chart or table → select an existing Azure Dashboard or create a new one.

**CLI / ARM Deployment:**
```bash
# Export a workbook as ARM template and deploy to another workspace
# First, get the workbook resource ID
az monitor app-insights component show \
  --resource-group rg-security \
  --output table

# Deploy a workbook from an ARM template
az deployment group create \
  --resource-group rg-security \
  --template-file workbook-template.json \
  --parameters workspaceName=sentinel-prod-workspace

# List all workbooks saved in a resource group
az resource list \
  --resource-group rg-security \
  --resource-type "microsoft.insights/workbooks" \
  --output table

# Get a specific workbook's details
az resource show \
  --resource-group rg-security \
  --resource-type "microsoft.insights/workbooks" \
  --name {workbook-guid} \
  --output json
```

**PowerShell Equivalent:**
```powershell
# List all workbooks in a resource group
Get-AzResource `
  -ResourceGroupName "rg-security" `
  -ResourceType "microsoft.insights/workbooks" |
  Format-Table Name, ResourceId, Tags

# Deploy a workbook ARM template
New-AzResourceGroupDeployment `
  -ResourceGroupName "rg-security" `
  -TemplateFile "workbook-template.json" `
  -workspaceName "sentinel-prod-workspace"
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/sentinel/get-visibility

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Templates tab | Browse 100+ built-in workbooks from Microsoft | Start here before building from scratch |
| Save to gallery | Makes a workbook available to all users of the Sentinel workspace | After customizing a template — save so the team can use it |
| Time range parameter | Controls the query time window across all components | Always expose this as a parameter so users can change it without editing the workbook |
| Pin to dashboard | Adds a workbook component to the Azure portal home dashboard | Use for executive-facing KPI tiles that leadership checks daily |
| Export ARM template | Exports the workbook as deployable JSON | Use for deploying the same workbook across multiple Sentinel workspaces |
| UEBA settings | Enables entity behavior analytics to populate anomaly data | Must be turned on in Sentinel Settings → Entity behavior before UEBA workbook has data |
| Auto-refresh | Refreshes workbook data on a set interval | Enable on SOC monitoring screens showing live threat activity |

***

## Comparison Table (if applicable)

| Workbook | Primary Use | Key Tables Queried |
|---|---|---|
| Azure AD Sign-in Analysis | Monitor sign-in health, risky sign-ins, MFA adoption | SigninLogs, AADNonInteractiveUserSignInLogs |
| Azure AD Audit Logs | Track directory changes, role assignments | AuditLogs |
| Azure Activity | Monitor ARM operations, resource changes | AzureActivity |
| Security Events | Windows logon activity, brute force trends | SecurityEvent |
| Office 365 | M365 email, Teams, SharePoint activity | OfficeActivity |
| Threat Intelligence | IOC match rate, indicator health | ThreatIntelligenceIndicator, CommonSecurityLog |
| UEBA | User and entity anomaly scores, risk timelines | BehaviorAnalytics, UserPeerAnalytics |
| Defender for Cloud | Secure Score trends, alert distribution | SecurityAlert, SecurityRecommendation |
| Insecure Protocols | Legacy auth usage that should be blocked | SigninLogs filtered by legacy auth client |

***

## What Happens If You Skip This?

Without workbooks, your Sentinel data stays invisible. Analysts have to write individual KQL queries every time they want to see a trend. Leadership has no dashboard to look at. UEBA anomalies go unreviewed because there is no visual summary. Compliance auditors ask for reports and you have to manually build them each time. Workbooks convert your log data investment into actionable, visible intelligence that the whole organization can use.

***

## AZ-500 Exam Section

- **Trigger Words:** "workbook", "dashboard", "UEBA", "entity behavior", "visualize", "KQL chart", "built-in workbook", "pin to dashboard", "Azure Monitor Workbook"
- **Common Traps:**
  - Thinking workbooks are static reports — they are fully interactive, live-querying KQL dashboards. Every time you open one, it runs fresh queries.
  - Thinking UEBA workbook works immediately — UEBA must be explicitly enabled in Sentinel settings first, and it needs time to establish baselines before it shows meaningful anomaly data.
  - Thinking workbooks are only for Sentinel — Azure Monitor Workbooks are a shared framework used by Sentinel, Defender for Cloud, Azure Monitor, and Application Insights.
  - Thinking you need to build everything from scratch — Sentinel has 100+ built-in templates. Always check templates before building a custom workbook.
- **How to Tell Apart:**
  - Workbook = visual dashboard (charts, maps, tables) — for monitoring and reporting
  - Analytics rule = detection logic that fires alerts — for detecting threats
  - Playbook = automated response — for acting on threats
- **Likely Question Formats:**
  - "A security team needs an interactive visual dashboard showing sign-in failure trends over the last 30 days in Sentinel. What do they use?" → Sentinel Workbook (Azure AD Sign-in Analysis template or custom workbook).
  - "A workbook shows no data in the UEBA section. What is the most likely cause?" → UEBA / Entity behavior analytics is not enabled in Sentinel settings.
  - "How can a CISO view live security KPIs on the Azure portal home page without opening Sentinel?" → Pin workbook components to an Azure Dashboard.
- **Memorization Tip:** Workbooks = **security TV screen** in the SOC. Always on, always live, showing what is happening right now. Analytics rules = the **alarm sensors**. Playbooks = the **response team**. Workbooks just show the picture — they do not alert or respond.

***

## Pocket Summary (5-Year Refresh)

- Sentinel Workbooks are live, interactive KQL dashboards built on Azure Monitor Workbooks — not static reports.
- Sentinel ships with 100+ built-in workbook templates covering Entra ID, Azure Activity, Office 365, UEBA, Threat Intelligence, and more.
- Custom workbooks can combine KQL queries, charts, tables, maps, and text into a single dashboard tailored to your organization.
- UEBA must be explicitly enabled before the entity behavior workbook has data — it needs time to build behavioral baselines.
- Workbook components can be pinned to the Azure portal Dashboard for executive-facing live security KPIs.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/sentinel/get-visibility
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
