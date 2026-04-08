# Microsoft Defender Workload Protection Plans

> 📌 AZ-500 Exam Objective: Configure and manage workload protection using Microsoft Defender plans
> 🏷️ Domain: 4 — Defender for Cloud and Sentinel | Weight: 30–35%

***

## What Is This?

Microsoft Defender workload protection plans are paid add-ons inside Defender for Cloud. Each plan watches a specific type of Azure resource for active threats. When something suspicious happens — like a malware execution, SQL injection attempt, or suspicious API call — the plan fires a security alert. This is the CWPP (Cloud Workload Protection Platform) layer of Defender for Cloud.

***

## Why Does This Exist?

The free tier of Defender for Cloud only scores your posture. It cannot detect active attacks. Defender plans fill that gap. Each plan is purpose-built for a specific workload type because threats look very different on a virtual machine versus a storage account versus a Kubernetes cluster. You pay only for the plans you actually need.

***

## Who Actually Needs This? (Real-World Context)

A retail company called ShopNow runs an e-commerce platform on Azure. They have VMs running their web app, Azure SQL databases holding order data, and Azure Storage storing product images. Their PCI-DSS compliance requires threat detection on all systems that touch cardholder data. They enable Defender for Servers Plan 2, Defender for SQL, and Defender for Storage. When an attacker tries an SQL injection attack during a sale event, Defender for SQL fires a High-severity alert within minutes. The security team blocks the IP before any data is stolen.

***

## How It Works (Pure Concept, No Portal)

Each Defender plan is enabled at the subscription level. Once enabled, it automatically covers all resources of that type in the subscription. Here is what each plan does:

**Defender for Servers Plan 1 (P1)**
- Integrates with Microsoft Defender for Endpoint (MDE) for EDR on Windows and Linux VMs.
- Provides vulnerability assessment powered by Qualys or MDE.
- Works for Azure VMs and Arc-enabled servers.
- Cheaper option when you mainly need endpoint detection.

**Defender for Servers Plan 2 (P2)**
- Everything in Plan 1, plus:
- Just-in-time (JIT) VM access to lock down management ports.
- File integrity monitoring (FIM) to detect changes to critical OS files.
- Adaptive application controls to whitelist approved applications.
- Network map showing exposed VMs.
- 500 MB/day free data ingestion into Log Analytics per server.
- Agentless scanning for vulnerabilities, secrets, and malware.

**Defender for Storage**
- Detects unusual access patterns on Blob storage and Azure Files.
- Flags access from Tor exit nodes, anonymous access attempts, and mass download of files.
- Scans uploaded files for malware using hash reputation (and optional on-access scanning in the enhanced plan).

**Defender for SQL**
- Covers Azure SQL Database, Azure SQL Managed Instance, and SQL Server on VMs.
- Detects SQL injection attempts, anomalous login patterns, and unusual data exports.
- Also runs vulnerability assessments on SQL databases.

**Defender for Containers**
- Protects AKS (Azure Kubernetes Service), Azure Container Registry, and Arc-enabled Kubernetes.
- Scans container images for vulnerabilities in the registry.
- Detects runtime threats inside running containers (e.g., crypto miners, privilege escalation).
- Provides Kubernetes audit log analysis.

**Defender for App Service**
- Monitors Azure App Service (web apps, function apps) for threats.
- Detects web shell installation, suspicious outbound connections, and dangling DNS attacks.
- Works with Windows and Linux App Service plans.

**Defender for Key Vault**
- Detects unusual access to Key Vault — for example, access from an unusual IP, access from a deleted application, or high-volume key operations that suggest credential stuffing.

**Defender for DNS**
- Monitors DNS queries made by Azure resources.
- Detects communication with known malicious domains, DNS tunneling (data exfiltration over DNS), and crypto mining domains.

**Defender for Resource Manager**
- Monitors all Azure Resource Manager (ARM) operations across your subscription.
- Detects suspicious activity like mass resource deletion, unusual role assignments, or operations from suspicious IPs.
- Important: ARM is the management plane — every portal click goes through ARM.

**Defender for APIs (API Security)**
- Protects APIs published in Azure API Management.
- Detects unusual API call volumes, authentication bypass attempts, and data exfiltration via APIs.
- Provides API inventory and risk scoring.

**Defender CSPM (Paid Tier)**
- Not a workload protection plan — this is the paid upgrade to CSPM.
- Adds: attack path analysis, cloud security explorer, agentless scanning, governance rules, and data-aware security posture.

***

## 2026 Portal Walkthrough — Step by Step

**Enable Defender for Servers Plan 2:**
1. Go to **Microsoft Defender for Cloud** in the Azure Portal.
2. Click **Environment settings** in the left menu.
3. Select your subscription.
4. Find **Servers** in the plan list.
5. Toggle it to **On**.
6. Click the **Plan** dropdown next to Servers and select **Plan 2**.
7. Click **Save**.

**View the Coverage Map:**
1. In Defender for Cloud, click **Workload protections** in the left menu.
2. The coverage dashboard shows which Defender plans are enabled and how many resources are covered.
3. Resources not covered by any plan appear in the **Unprotected resources** section.
4. Click any plan tile to see which specific resources it is protecting.

**Review a Threat Alert:**
1. Click **Security alerts** in the left menu.
2. Filter by severity to find High-severity alerts.
3. Click an alert to open the details pane.
4. The pane shows: alert description, affected resource, attack timeline, investigation steps, and MITRE ATT&CK tactic.
5. Click **Take action** to see response options (run a playbook, suppress, export).
6. Click **Investigate in Microsoft Sentinel** if Sentinel is connected.

**CLI Equivalent:**
```bash
# Enable Defender for Servers Plan 2
az security pricing create \
  --name VirtualMachines \
  --tier Standard \
  --subplan P2

# Enable Defender for SQL (Azure SQL servers)
az security pricing create \
  --name SqlServers \
  --tier Standard

# Enable Defender for Storage
az security pricing create \
  --name StorageAccounts \
  --tier Standard

# Enable Defender for Containers
az security pricing create \
  --name Containers \
  --tier Standard

# Enable Defender for Key Vault
az security pricing create \
  --name KeyVaults \
  --tier Standard

# Enable Defender for App Service
az security pricing create \
  --name AppServices \
  --tier Standard

# Enable Defender for Resource Manager
az security pricing create \
  --name Arm \
  --tier Standard

# Enable Defender for DNS
az security pricing create \
  --name Dns \
  --tier Standard

# List all current Defender plan statuses
az security pricing list --output table

# List security alerts
az security alert list --output table
```

**PowerShell Equivalent:**
```powershell
# Enable Defender for Servers Plan 2
Set-AzSecurityPricing -Name "VirtualMachines" -PricingTier "Standard" -SubPlan "P2"

# Enable Defender for SQL
Set-AzSecurityPricing -Name "SqlServers" -PricingTier "Standard"

# Enable Defender for Storage
Set-AzSecurityPricing -Name "StorageAccounts" -PricingTier "Standard"

# Enable Defender for Key Vault
Set-AzSecurityPricing -Name "KeyVaults" -PricingTier "Standard"

# List all Defender plan statuses
Get-AzSecurityPricing | Format-Table Name, PricingTier, SubPlan

# List security alerts
Get-AzSecurityAlert | Format-Table AlertDisplayName, Severity, ResourceId
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Defender for Servers Plan 1 | MDE integration + vulnerability assessment | When you need endpoint detection but not JIT or FIM |
| Defender for Servers Plan 2 | Plan 1 + JIT VM access + FIM + adaptive controls + agentless scan | For production servers needing full protection |
| Defender for SQL | SQL injection and anomalous login detection + vulnerability assessment | When running any Azure SQL service with sensitive data |
| Defender for Storage | Malware detection and unusual access pattern alerts on Blob/Files | When storage contains sensitive or regulated data |
| Defender for Containers | Image scanning + runtime threat detection for AKS | When running containerized workloads in production |
| Defender for Key Vault | Unusual access alerts on Key Vault | Always — Key Vault holds your most sensitive secrets |
| Defender for Resource Manager | Monitors all ARM operations for suspicious management activity | When you want visibility into who is doing what in your subscription |
| Defender for APIs | Protects APIs published via Azure API Management | When you expose APIs that handle sensitive data |
| Auto-provisioning | Automatically installs required agents (AMA, MDE) on new VMs | Enable so new resources are protected without manual steps |

***

## Comparison Table (if applicable)

| Defender Plan | What It Protects | Key Threat It Detects |
|---|---|---|
| Defender for Servers P1 | Azure VMs + Arc servers | Malware, exploits, endpoint threats |
| Defender for Servers P2 | Azure VMs + Arc servers | Everything in P1 + unauthorized management port access, file changes |
| Defender for Storage | Azure Blob + Files | Malware uploads, anonymous access, unusual data access |
| Defender for SQL | Azure SQL DB, MI, SQL on VMs | SQL injection, anomalous logins, data exfiltration |
| Defender for Containers | AKS, ACR, Arc K8s | Vulnerable images, crypto miners in pods, K8s API abuse |
| Defender for App Service | Azure Web Apps | Web shells, dangling DNS, C2 callbacks |
| Defender for Key Vault | Azure Key Vault | Credential stuffing, access from suspicious IPs |
| Defender for DNS | All Azure resource DNS | DNS tunneling, C2 over DNS, malicious domain lookups |
| Defender for Resource Manager | All ARM operations | Mass deletion, suspicious role assignments, management plane attacks |
| Defender for APIs | Azure API Management APIs | Anomalous API calls, auth bypass |

***

## What Happens If You Skip This?

If no Defender plans are enabled, Azure resources generate no threat alerts. A VM can have an active crypto miner running for months without any detection. A SQL database can receive repeated injection attempts with no warning. A storage account can be accessed from a Tor exit node silently. Attackers specifically look for environments with no workload protection because there is no detection, no alert, and no response.

***

## AZ-500 Exam Section

- **Trigger Words:** "Defender for Servers", "workload protection", "threat detection", "Defender plan", "MMA agent", "JIT", "FIM", "Defender for SQL", "Defender for Containers", "CWPP"
- **Common Traps:**
  - Thinking the free tier of Defender for Cloud includes threat detection — the free tier is CSPM only. You need paid plans for CWPP.
  - Thinking Defender for Servers Plan 1 includes JIT VM access — JIT is Plan 2 only.
  - Thinking MMA (Microsoft Monitoring Agent) is current — MMA is legacy. The current agent is AMA (Azure Monitor Agent). Defender for Servers Plan 2 uses agentless scanning and MDE, not MMA.
  - Thinking Defender plans are enabled per resource — they are enabled per subscription and apply to all resources of that type in the subscription.
- **How to Tell Apart:**
  - Plan 1 vs Plan 2: If the question mentions JIT, FIM, or adaptive application controls, that is Plan 2.
  - Defender for SQL vs Defender for Servers: SQL protects the database engine; Servers protects the OS running the VM.
- **Likely Question Formats:**
  - "A company needs to detect SQL injection attacks on Azure SQL Database. Which Defender plan should they enable?" → Defender for SQL.
  - "You need to restrict SSH access to a VM to only when needed and log every connection. Which Defender for Servers feature handles this?" → Just-in-time (JIT) VM access (Plan 2).
  - "Which Defender plan scans container images in Azure Container Registry for vulnerabilities?" → Defender for Containers.
- **Memorization Tip:** Match the plan to the resource name — Defender for **Servers** protects servers, Defender for **SQL** protects SQL, Defender for **Storage** protects storage. Each plan = one resource family.

***

## Pocket Summary (5-Year Refresh)

- Defender workload protection plans are paid add-ons that detect active threats on specific Azure resource types.
- The free tier gives you posture scoring only — no threat detection without a paid plan.
- Defender for Servers Plan 2 is the most comprehensive server protection plan, adding JIT, FIM, and agentless scanning over Plan 1.
- Plans are enabled per subscription and automatically cover all matching resources in that subscription.
- Each plan is purpose-built: Defender for SQL catches injection attacks, Defender for Key Vault catches credential stuffing, Defender for Containers catches runtime threats in pods.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
