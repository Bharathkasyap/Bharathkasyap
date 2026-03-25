# KQL (Kusto Query Language) for Security Operations

> 📌 AZ-500 Exam Objective: Manage and monitor Microsoft Sentinel using KQL
> 🏷️ Domain: 4 — Defender for Cloud and Sentinel | Weight: 30–35%

***

## What Is This?

KQL (Kusto Query Language) is the query language used to search and analyze log data in Microsoft Sentinel, Azure Monitor, and Log Analytics. Think of it like SQL, but designed specifically for time-series log data. You write a KQL query, and it searches through millions of log records in seconds to find exactly what you are looking for.

***

## Why Does This Exist?

Security operations teams deal with massive amounts of log data. You cannot manually read through millions of events to find the three that matter. KQL lets you write precise queries that filter, group, and summarize log data instantly. It powers everything in Sentinel — analytics rules, workbooks, threat hunting, and incident investigation all run on KQL queries underneath.

***

## Who Actually Needs This? (Real-World Context)

A SOC analyst at a bank called TrustBank is investigating a potential account takeover. A customer reported unauthorized transfers. The analyst opens Sentinel, goes to Logs, and writes a KQL query against the `SigninLogs` table to find all sign-ins for that account in the last 7 days. The query returns 12 sign-ins — 10 from the user's normal location in Chicago and 2 from an IP in Eastern Europe that happened one hour before the transfers. In 30 seconds, the analyst has the evidence they need to confirm the account was compromised.

***

## How It Works (Pure Concept, No Portal)

**KQL Query Structure**
A KQL query always starts with a table name, then pipes (`|`) commands together to filter and transform data. Each step passes its output to the next step.

```
TableName
| command1
| command2
| command3
```

**Essential KQL Commands**

| Command | What It Does | Example |
|---|---|---|
| `where` | Filters rows based on a condition | `where EventID == 4625` |
| `project` | Selects specific columns to show | `project TimeGenerated, Account, IpAddress` |
| `summarize` | Groups and aggregates data | `summarize count() by Account` |
| `extend` | Adds a new calculated column | `extend HourOfDay = hourofday(TimeGenerated)` |
| `order by` | Sorts results | `order by TimeGenerated desc` |
| `top` | Returns the first N results | `top 10 by TimeGenerated` |
| `join` | Combines two tables on a key | `join (OtherTable) on $left.IP == $right.IP` |
| `ago()` | Calculates time relative to now | `where TimeGenerated > ago(24h)` |
| `bin()` | Groups time into buckets | `bin(TimeGenerated, 1h)` |
| `distinct` | Returns unique values | `distinct Account` |
| `count` | Counts rows | `summarize count()` |
| `render` | Visualizes output as a chart | `render timechart` |

**Key Sentinel Tables**

| Table | Contains |
|---|---|
| `SecurityEvent` | Windows Security event log events (Event IDs 4624, 4625, 4720, etc.) |
| `SigninLogs` | Microsoft Entra ID interactive sign-in logs |
| `AADNonInteractiveUserSignInLogs` | Non-interactive Entra ID sign-ins (service accounts, tokens) |
| `AuditLogs` | Entra ID directory changes (user creation, role assignment, app registration) |
| `AzureActivity` | Azure subscription-level operations (resource creation, deletion, role changes) |
| `SecurityAlert` | Security alerts from Defender for Cloud, Defender XDR, and other sources |
| `CommonSecurityLog` | CEF-formatted logs from firewalls and security appliances (Palo Alto, Check Point) |
| `OfficeActivity` | Microsoft 365 activity (SharePoint, Teams, Exchange, OneDrive) |
| `Syslog` | Linux syslog messages from VMs and on-premises servers |
| `DeviceLogonEvents` | Defender for Endpoint device logon events |
| `DeviceProcessEvents` | Defender for Endpoint process execution events |
| `ThreatIntelligenceIndicator` | Threat intelligence indicators (IOCs) imported into Sentinel |

***

## 2026 Portal Walkthrough — Step by Step

**Open Logs in Sentinel:**
1. In Microsoft Sentinel, click **Logs** in the left menu.
2. The Log Analytics query editor opens.
3. The left panel shows all available tables grouped by category (Security, Azure, Custom, etc.).
4. Type your KQL query in the query editor.
5. Click **Run** (or press Shift+Enter).
6. Results appear in the table below. Switch to the Chart tab to visualize results.
7. Click **Save** to save a query for reuse, or **New alert rule** to turn the query into an analytics rule.

***

## KQL Example Queries

**Example 1 — Failed Sign-ins in Last 24 Hours**
Find all failed interactive sign-ins in the past 24 hours, showing who failed and how many times.

```kql
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType != "0"                          // Non-zero = failed sign-in
| summarize FailedAttempts = count() by UserPrincipalName, IPAddress, ResultDescription
| order by FailedAttempts desc
```

**Example 2 — New Admin Account Created**
Detect when a new account was added to a privileged role or admin group in Entra ID.

```kql
AuditLogs
| where TimeGenerated > ago(7d)
| where OperationName == "Add member to role"
| extend TargetUser = tostring(TargetResources[0].userPrincipalName)
| extend RoleName = tostring(TargetResources[0].displayName)
| extend InitiatedByUser = tostring(InitiatedBy.user.userPrincipalName)
| project TimeGenerated, TargetUser, RoleName, InitiatedByUser, Result
| order by TimeGenerated desc
```

**Example 3 — Unusual Azure Resource Deletion**
Find Azure resource deletions in the last 48 hours — useful for detecting destructive attacks or insider threats.

```kql
AzureActivity
| where TimeGenerated > ago(48h)
| where OperationNameValue endswith "delete"
| where ActivityStatusValue == "Success"
| project TimeGenerated, Caller, ResourceGroup, ResourceId, OperationNameValue
| order by TimeGenerated desc
```

**Example 4 — Brute Force Detection (Multiple Failures from Same IP)**
Find IP addresses that have had 10 or more failed sign-ins in the last 1 hour — a strong indicator of brute force.

```kql
SigninLogs
| where TimeGenerated > ago(1h)
| where ResultType != "0"
| summarize FailedCount = count(), 
            DistinctUsers = dcount(UserPrincipalName),
            UserList = make_set(UserPrincipalName) by IPAddress
| where FailedCount >= 10
| order by FailedCount desc
```

**Example 5 — Privilege Escalation (Role Assignment)**
Detect when someone assigns an Azure RBAC role — especially Owner or Contributor — which could indicate privilege escalation.

```kql
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue == "microsoft.authorization/roleassignments/write"
| where ActivityStatusValue == "Success"
| extend RoleInfo = parse_json(Properties)
| extend AssignedRole = tostring(RoleInfo.requestbody)
| project TimeGenerated, Caller, ResourceGroup, SubscriptionId, AssignedRole
| order by TimeGenerated desc
```

**Example 6 — Outbound Connection to Suspicious IP**
Find outbound network connections to IPs flagged in threat intelligence. Useful for detecting command-and-control (C2) communication.

```kql
CommonSecurityLog
| where TimeGenerated > ago(24h)
| where CommunicationDirection == "Outbound"
| join kind=inner (
    ThreatIntelligenceIndicator
    | where TimeGenerated > ago(7d)
    | where isnotempty(NetworkIP)
    | project TI_IP = NetworkIP, ThreatType, Description
) on $left.DestinationIP == $right.TI_IP
| project TimeGenerated, SourceIP, DestinationIP, ThreatType, Description, DeviceName
| order by TimeGenerated desc
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| `TimeGenerated` | Timestamp of the log event | Always filter by TimeGenerated first — it dramatically speeds up queries |
| `ago()` function | Returns a datetime relative to now | Use instead of hardcoded dates so queries always cover the right window |
| `summarize ... by` | Groups data and computes aggregates (count, average, max) | Use for detection queries that need to count events per user, IP, or resource |
| `render timechart` | Draws a time-series chart of results | Use in workbooks or when hunting for patterns over time |
| `join kind=inner` | Returns only rows that match in BOTH tables | Use when correlating IOCs or combining two event streams |
| `make_set()` | Collects distinct values into a list (array) | Use to gather all user accounts seen from a suspicious IP in one row |
| `dcount()` | Counts distinct values | Use to count unique users or IPs without listing them all |
| Query time range selector | Limits the time window queried from Log Analytics | Always set a time range — queries with no time filter are very slow and expensive |

***

## Comparison Table (if applicable)

| KQL Concept | SQL Equivalent | Notes |
|---|---|---|
| `where` | `WHERE` | Same logic — filter rows |
| `project` | `SELECT column1, column2` | Select specific columns |
| `summarize count() by X` | `SELECT X, COUNT(*) FROM ... GROUP BY X` | Group and count |
| `order by` | `ORDER BY` | Sort results |
| `join` | `JOIN` | Multiple join types: inner, leftouter, fullouter |
| `extend` | computed column in `SELECT` | Adds a derived column |
| `top 10 by col` | `SELECT TOP 10 ... ORDER BY col` | First N results by a column |
| `ago(24h)` | `GETDATE() - 1` | Time relative to now |
| `bin(time, 1h)` | `DATEPART(hour, time)` | Time bucketing for histograms |

***

## What Happens If You Skip This?

If your SOC team does not know KQL, they cannot write custom detection rules, hunt for threats, or investigate incidents effectively. They are limited to clicking through pre-built dashboards and default alerts. When a new attack technique emerges — one that no built-in rule catches — analysts have no way to query raw log data to find it. Attackers stay undetected for longer because the team cannot write the queries needed to surface the threat.

***

## AZ-500 Exam Section

- **Trigger Words:** "KQL", "Kusto", "SecurityEvent", "SigninLogs", "hunt", "query", "Log Analytics", "CommonSecurityLog", "AzureActivity", "where", "summarize"
- **Common Traps:**
  - Thinking KQL is only for Sentinel — KQL is used across all Log Analytics queries, Azure Monitor, Application Insights, and Azure Data Explorer.
  - Thinking `SecurityEvent` contains all security logs — `SecurityEvent` is Windows-specific. Linux logs go to `Syslog`. Firewall logs go to `CommonSecurityLog` (CEF). Entra ID sign-ins go to `SigninLogs`.
  - Thinking a failed sign-in has ResultType == 0 — ResultType == "0" means SUCCESS. Non-zero means failure. This is the reverse of what most people expect.
- **How to Tell Apart:**
  - `SecurityEvent` = Windows OS security events
  - `SigninLogs` = Entra ID user sign-ins
  - `AuditLogs` = Entra ID directory changes (users, groups, roles)
  - `AzureActivity` = Azure subscription management operations (ARM)
  - `CommonSecurityLog` = CEF logs from firewalls and appliances
- **Likely Question Formats:**
  - "A query needs to find all failed Windows logons in the last 4 hours. Which table and Event ID should you use?" → `SecurityEvent` table, `EventID == 4625`, with `TimeGenerated > ago(4h)`.
  - "A query needs to detect when a new user is assigned the Global Administrator role. Which table should you query?" → `AuditLogs`, filtering for `OperationName == "Add member to role"`.
  - "You need to visualize sign-in failures over the last 7 days as a time chart. Which KQL command renders the chart?" → `render timechart` at the end of the query.
- **Memorization Tip:** For the exam, memorize the big 6 tables: **SecurityEvent** (Windows), **SigninLogs** (Entra logins), **AuditLogs** (Entra changes), **AzureActivity** (ARM operations), **CommonSecurityLog** (CEF firewalls), **OfficeActivity** (M365). Match the source to the table.

***

## Pocket Summary (5-Year Refresh)

- KQL is the query language for Sentinel, Log Analytics, and Azure Monitor — learning it is essential for security operations work.
- Every KQL query starts with a table name and chains commands with the pipe (`|`) operator.
- The most important commands are: `where` (filter), `project` (columns), `summarize` (aggregate), `extend` (add column), `join` (combine tables), `render` (chart).
- Key tables to know: SecurityEvent (Windows), SigninLogs (Entra logins), AuditLogs (Entra changes), AzureActivity (ARM), CommonSecurityLog (firewalls).
- Always filter by `TimeGenerated > ago(Xh)` first in every query — it dramatically improves query performance.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
