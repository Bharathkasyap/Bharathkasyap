# Microsoft Sentinel Data Connectors

> 📌 AZ-500 Exam Objective: Configure data ingestion and connectors in Microsoft Sentinel
> 🏷️ Domain: 4 — Defender for Cloud and Sentinel | Weight: 30–35%

***

## What Is This?

Data connectors are the pipelines that bring log data into Microsoft Sentinel. Without data connectors, Sentinel has nothing to analyze. Each connector handles a specific data source — like Microsoft Entra ID, Defender for Cloud, Windows event logs, Cisco firewalls, or AWS CloudTrail. Once connected, the logs flow into Log Analytics Workspace tables where Sentinel can query, detect, and alert on them.

***

## Why Does This Exist?

A SIEM is only as useful as the data it receives. Sentinel needs logs from every part of your environment to detect threats. A single failed login means little. But when Sentinel sees failed logins from one IP, followed by a successful login, followed by a mass data download — across three different log sources — it can detect that chain of events. Data connectors make that cross-source visibility possible.

***

## Who Actually Needs This? (Real-World Context)

A manufacturing company called IndustraTech runs a hybrid environment. They have Azure VMs, Microsoft 365 email, on-premises Linux servers, and Palo Alto firewalls at each factory. Their SOC needs to see all of this in Sentinel. They use:
- The Microsoft Entra ID connector for sign-in and audit logs.
- The Microsoft 365 Defender connector for email threat alerts.
- The Syslog connector with AMA agent for Linux servers.
- The Common Event Format (CEF) connector for Palo Alto firewall logs.
- The AWS CloudTrail connector to monitor their small AWS footprint.
After connecting all sources, Sentinel can detect an attack that starts with a phishing email in M365, leads to a compromised identity in Entra ID, and then moves to an on-premises Linux server via the firewall.

***

## How It Works (Pure Concept, No Portal)

**Connector Categories**

There are four main categories of data connectors in Sentinel:

**1. Native Microsoft Connectors (Service-to-Service)**
These are built-in connectors that connect directly to Microsoft services with a few clicks. No agents needed. Examples:
- Microsoft Entra ID (sign-in logs, audit logs, provisioning logs)
- Microsoft Defender XDR (alerts from Defender for Endpoint, Defender for Office 365, Defender for Identity)
- Microsoft Defender for Cloud (security alerts)
- Microsoft 365 (SharePoint, Teams, Exchange activity via Office 365 connector)
- Azure Activity (subscription-level operations)
- Azure Active Directory Identity Protection (risky user and sign-in events)

**2. Agent-Based Connectors (Windows and Linux)**
For on-premises machines, VMs, or third-party appliances that do not have a native connector, you install an agent on the machine to forward logs.

- **Azure Monitor Agent (AMA)** — the current recommended agent. Replaces the legacy MMA. Configured using Data Collection Rules (DCRs). More flexible and efficient than MMA.
- **Microsoft Monitoring Agent (MMA) / Log Analytics Agent** — the legacy agent. Still supported but Microsoft plans to retire it. If you see MMA in a question, it refers to the older agent.
- **Syslog connector** — uses AMA or the legacy Linux syslog daemon to forward Syslog messages from Linux machines to Sentinel.
- **CEF (Common Event Format) connector** — a standardized log format used by many security appliances (Palo Alto, Check Point, F5). A Linux machine runs a syslog forwarder that receives CEF messages from the appliance and forwards them to Sentinel.

**3. Custom Connectors**
When no built-in connector exists, you can build one using:
- **Azure Logic Apps** with the Sentinel HTTP data connector.
- **Azure Functions** (serverless) to pull data from an external API on a schedule.
- **Data Collector REST API** — a Log Analytics HTTP endpoint that any application can POST JSON log data to.
- **Azure Event Hub** — a high-throughput event stream that can receive data from almost any source.

**4. Third-Party and Community Connectors**
Many third-party security vendors provide connectors in the Sentinel Content Hub. Examples:
- AWS CloudTrail (Amazon Web Services activity logs)
- Google Cloud Platform (GCP) audit logs
- Cisco ASA and Umbrella
- Palo Alto Networks
- Fortinet FortiGate
- CrowdStrike Falcon
- Okta SSO

**Azure Monitor Agent (AMA) vs MMA (Legacy)**
This is important for the exam:

| Feature | AMA (Current) | MMA / Log Analytics Agent (Legacy) |
|---|---|---|
| Configuration method | Data Collection Rules (DCRs) | Workspace settings |
| Multi-homing | Supported (send to multiple workspaces) | Limited |
| Performance | More efficient, lower CPU/memory overhead | Higher overhead |
| OS support | Windows + Linux | Windows + Linux |
| Status | Current — actively developed | Legacy — being retired |
| Sentinel usage | Recommended for all new deployments | Use only if migrating from legacy |

**Data Connector Health Monitoring**
Sentinel includes a Data Connector Health workbook. It shows:
- Which connectors are active.
- Last log received timestamp.
- Log ingestion latency.
- Data volume per connector over time.

Use this to catch connectors that stop sending data — a silent connector failure means blind spots in your detection coverage.

**Ingestion Cost Management**
Every GB of data ingested into Sentinel costs money. Some sources are free:
- Microsoft Entra ID audit and sign-in logs: free up to 90 days of retention.
- Microsoft Defender for Cloud alerts: free for basic data.
- Azure Activity logs: free.

Higher-volume sources like Windows Security Events, Syslog, and CEF logs cost money per GB. To manage costs:
- Filter what you ingest (send only Security event IDs you actually need, not all Windows events).
- Use workspace transformation rules to drop noisy, low-value events before they are stored.
- Monitor daily ingestion volume using the Sentinel cost workbook.

***

## 2026 Portal Walkthrough — Step by Step

**Enable Microsoft Entra ID Connector:**
1. In Microsoft Sentinel, click **Data connectors** in the left menu.
2. Search for **Microsoft Entra ID** (formerly Azure Active Directory).
3. Click the connector, then click **Open connector page**.
4. Under **Configuration**, check the boxes for the log types you want: Sign-in logs, Audit logs, Provisioning logs, Non-interactive user sign-ins.
5. Click **Apply Changes**.
6. Logs begin flowing into `SigninLogs`, `AuditLogs`, and related tables in Log Analytics.

**Enable Defender for Cloud Connector:**
1. In Data connectors, search for **Microsoft Defender for Cloud**.
2. Open the connector page.
3. Select your subscription(s) from the list.
4. Toggle **Connect** to On.
5. Optionally enable **Bi-directional sync** so incidents closed in Sentinel also close in Defender for Cloud.
6. Security alerts from Defender for Cloud now appear in the `SecurityAlert` table.

**Add a Syslog Connector via AMA:**
1. In Data connectors, search for **Syslog via AMA**.
2. Open the connector page and click **Open connector page**.
3. Click **Create data collection rule (DCR)**.
4. Name the DCR, select the subscription and resource group.
5. Add the Linux VMs or Arc-enabled servers as resources.
6. Configure which Syslog facilities and severity levels to collect (e.g., auth, authpriv at Warning and above).
7. Select the Log Analytics Workspace as the destination.
8. Click **Create**.
9. The AMA agent is automatically installed on the selected machines via the DCR.
10. Syslog messages flow into the `Syslog` table in Log Analytics.

**CLI — Workspace and Connector Management:**
```bash
# Create a Log Analytics Workspace (prerequisite for Sentinel connectors)
az monitor log-analytics workspace create \
  --resource-group rg-security \
  --workspace-name sentinel-prod-workspace \
  --location eastus

# Get workspace customer ID (needed for REST API data ingestion)
az monitor log-analytics workspace show \
  --resource-group rg-security \
  --workspace-name sentinel-prod-workspace \
  --query "customerId" -o tsv

# List data collection rules (DCRs used by AMA)
az monitor data-collection rule list \
  --resource-group rg-security \
  --output table

# Create a data collection rule for Syslog via AMA
az monitor data-collection rule create \
  --name dcr-syslog-linux \
  --resource-group rg-security \
  --location eastus \
  --data-flows '[{"streams":["Microsoft-Syslog"],"destinations":["sentinel-workspace"]}]' \
  --syslog '[{"name":"syslog-auth","streams":["Microsoft-Syslog"],"facilityNames":["auth","authpriv"],"logLevels":["Warning","Error","Critical","Alert","Emergency"]}]' \
  --destinations '{"logAnalytics":[{"workspaceResourceId":"/subscriptions/{subId}/resourceGroups/rg-security/providers/Microsoft.OperationalInsights/workspaces/sentinel-prod-workspace","name":"sentinel-workspace"}]}'

# Note: Most Sentinel connector management uses the Sentinel REST API
# Example: Enable Azure Activity connector via REST
az rest \
  --method PUT \
  --url "https://management.azure.com/subscriptions/{subId}/resourceGroups/rg-security/providers/Microsoft.OperationalInsights/workspaces/sentinel-prod-workspace/providers/Microsoft.SecurityInsights/dataConnectors/{connectorId}?api-version=2022-11-01" \
  --body '{"kind":"AzureActivity","properties":{"subscriptionId":"{subId}","state":"Enabled"}}'
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/sentinel/connect-data-sources

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Service-to-service connectors | One-click connection to Microsoft services (Entra ID, M365, Defender) | Always enable for Microsoft services first — no agents required |
| CEF connector | Forwards Common Event Format logs from security appliances via a Linux syslog proxy | When connecting firewalls (Palo Alto, Check Point, F5) to Sentinel |
| Syslog via AMA | Collects Linux syslog messages using the current Azure Monitor Agent | When collecting logs from Linux VMs or on-premises Linux servers |
| Data Collection Rules (DCRs) | Configures what the AMA agent collects and where it sends data | Required when using AMA for Syslog, Windows events, or custom logs |
| Bi-directional sync | Syncs incident state between Sentinel and Defender for Cloud | Enable so closing an incident in one tool updates the other |
| Connector health monitoring | Shows which connectors are active and last log received time | Check weekly to ensure no connectors silently stopped sending data |
| Data transformation rules | Filters or transforms log data before storage | Use to drop noisy events before they incur ingestion costs |

***

## Comparison Table (if applicable)

| Connector Type | Example Sources | Method | Agent Required? |
|---|---|---|---|
| Native Microsoft (service-to-service) | Entra ID, Defender XDR, M365, Azure Activity | One-click in portal | No |
| Agent-based (AMA) | Windows VMs, Linux VMs, Arc servers | Install AMA + create DCR | Yes (AMA) |
| CEF / Syslog | Palo Alto, Check Point, Cisco, Fortinet | Linux proxy + syslog forwarding | Yes (on Linux proxy) |
| REST API / Function App | Custom apps, third-party SaaS | HTTP POST or scheduled pull | No (uses Azure Functions) |
| Event Hub | High-throughput streaming sources | Event Hub → Sentinel connector | No (uses Event Hub) |
| Third-party (Content Hub) | AWS CloudTrail, Okta, CrowdStrike | Varies per connector | Varies |

***

## What Happens If You Skip This?

Without data connectors, Sentinel is an empty shell. It can run analytics rules, but those rules have no data to query. Threats happen outside of Azure — in Microsoft 365 email, on-premises servers, cloud firewalls — and Sentinel never sees them. The SOC team is blind to every attack that does not originate in Azure. Multi-stage attacks that span on-premises and cloud environments go completely undetected.

***

## AZ-500 Exam Section

- **Trigger Words:** "data connector", "CEF", "Syslog", "AMA", "MMA", "log ingestion", "Data Collection Rule", "service-to-service", "syslog forwarder"
- **Common Traps:**
  - Thinking all connectors are free — data ingestion is billed by GB. Free ingestion applies only to a limited set of Microsoft first-party sources.
  - Thinking MMA is the current agent — AMA (Azure Monitor Agent) is current. MMA is legacy and being retired.
  - Thinking CEF and Syslog are the same — CEF is a structured log format built on top of Syslog. Not all Syslog messages are CEF. Firewalls typically use CEF; Linux system logs typically use plain Syslog.
  - Thinking you can connect appliances directly to Sentinel — most non-Azure appliances require an intermediate Linux syslog forwarder machine to receive and forward logs.
- **How to Tell Apart:**
  - AMA = current agent, uses DCRs, recommended for new deployments
  - MMA = legacy agent, being retired, used in older Sentinel deployments
  - CEF = structured format from security appliances (firewalls, IDS)
  - Syslog = general Linux system log format
- **Likely Question Formats:**
  - "A company wants to send Palo Alto firewall logs to Sentinel. What connector type do they need?" → CEF connector with a Linux syslog forwarder.
  - "Which agent should be used for new Sentinel deployments to collect Windows and Linux logs?" → Azure Monitor Agent (AMA).
  - "A Sentinel data connector shows it last received data 3 days ago. Where do you go to investigate?" → Data Connector Health workbook or check the connector page's last data received timestamp.
- **Memorization Tip:** Think of connectors as **pipes**. Native Microsoft connectors are copper pipes (built-in, clean). CEF/Syslog connectors are garden hoses (need a fitting at the appliance end — the Linux proxy machine). Custom connectors are DIY pipes you build yourself with REST API or Logic Apps.

***

## Pocket Summary (5-Year Refresh)

- Data connectors bring log data from various sources into Sentinel's Log Analytics Workspace.
- Native Microsoft connectors (Entra ID, Defender XDR, M365) connect with a few clicks — no agents needed.
- CEF connectors require a Linux syslog proxy machine between the security appliance and Sentinel.
- AMA (Azure Monitor Agent) is the current recommended agent; MMA is legacy and being retired.
- Data ingestion costs money per GB — filter noisy events early using workspace transformation rules to control costs.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/sentinel/connect-data-sources
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
