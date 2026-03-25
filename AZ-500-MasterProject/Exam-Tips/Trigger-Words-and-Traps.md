# 🎯 AZ-500 — Trigger Words, Exam Traps & Decision Framework

> **How to use this file:** Read this list daily during the final week before your exam.  
> When you see a trigger phrase in a question, it should immediately point you to the correct service.

---

## 📋 Master Trigger Word Table (35+ Entries)

| # | Trigger Phrase | Service It Points To | Why |
|---|----------------|---------------------|-----|
| 1 | "just-in-time access" to a VM | JIT VM Access (Defender for Cloud) | JIT temporarily opens NSG ports on-demand. PIM is for directory/Azure roles, not VM ports. |
| 2 | "just-in-time" for a privileged role | PIM (Privileged Identity Management) | When the context is role activation, not VM access, the answer is PIM. |
| 3 | "FQDN filtering" | Azure Firewall | NSGs filter by IP/port only. Only Azure Firewall can filter by fully qualified domain name. |
| 4 | "no public IP on VM" / "RDP without public IP" | Azure Bastion | Bastion provides browser-based RDP/SSH over TLS to VMs with no public IP required. |
| 5 | "customer-managed key" / "CMK" | Azure Key Vault | CMK means you own the encryption key stored in Key Vault, not Microsoft. |
| 6 | "OWASP rules" / "SQL injection protection" / "XSS protection" | WAF (Web Application Firewall) | WAF implements OWASP Core Rule Sets. Azure Firewall does NOT do layer-7 HTTP inspection. |
| 7 | "soft delete" + "purge protection" | Azure Key Vault | These two settings together prevent accidental or malicious permanent key deletion. |
| 8 | "eligible assignment" | PIM | Eligible = user must activate the role; it is not active permanently. |
| 9 | "anonymous blob access" | Storage Account (disable it!) | Anonymous access allows unauthenticated reads of blobs. Always disable unless explicitly required. |
| 10 | "KQL" / "Kusto Query Language" | Microsoft Sentinel or Log Analytics | KQL is the query language of Azure Monitor / Log Analytics / Sentinel analytics rules. |
| 11 | "CSPM" / "Cloud Security Posture Management" | Defender for Cloud | CSPM = assessing configuration, Secure Score, and compliance posture. Not Sentinel. |
| 12 | "CWPP" / "Cloud Workload Protection Platform" | Defender for Cloud (Defender Plans) | CWPP = active threat detection for specific workloads (VMs, SQL, Storage, Containers). |
| 13 | "DeployIfNotExists" | Azure Policy | This policy effect automatically deploys a resource (e.g., Log Analytics agent) if missing. |
| 14 | "audit log" | Context-dependent! | Could be Entra ID Audit Logs, Azure Activity Log, or SQL Auditing — read the scenario carefully. |
| 15 | "service principal" | App Registration | Service principals are the identity of an App Registration within a specific tenant. |
| 16 | "user delegation SAS" | Storage Account — most secure SAS type | Signed with Entra ID credentials, not storage account key. Most secure SAS type available. |
| 17 | "CEF connector" / "Common Event Format" | Microsoft Sentinel data ingestion | CEF over Syslog is used to ingest logs from third-party firewalls and security appliances. |
| 18 | "automation rule" | Microsoft Sentinel (NOT Logic App) | Automation rules are Sentinel-native triggers. They can call playbooks but are distinct from them. |
| 19 | "playbook" | Sentinel Logic App automation | A playbook is a Logic App workflow triggered by Sentinel to automate response actions. |
| 20 | "secure score" | Defender for Cloud (NOT Sentinel) | Secure Score measures security posture in Defender for Cloud based on recommendation compliance. |
| 21 | "named location" | Conditional Access | Named Locations define trusted IP ranges or countries used in CA policies as a signal. |
| 22 | "sign-in risk" / "user risk" | Conditional Access + Entra ID Identity Protection | Identity Protection computes risk; Conditional Access enforces a response (block, MFA, password reset). |
| 23 | "always encrypted" | Azure SQL / Azure SQL Managed Instance | Protects sensitive column data from anyone with DB access including DBAs — encryption is client-side. |
| 24 | "dynamic data masking" / "DDM" | Azure SQL | Masks column data for non-privileged users in query results. DBA can still see real data. |
| 25 | "TDE" / "transparent data encryption" | Azure SQL | Encrypts the entire database file at rest. Transparent to the application. Enabled by default. |
| 26 | "private endpoint" | Private Endpoint (PaaS via private IP) | Creates a NIC with a private IP inside your VNet pointing to a PaaS service. Data stays off public internet. |
| 27 | "service endpoint" | VNet Service Endpoint | Extends VNet identity to PaaS service. Does NOT give the service a private IP. Less secure than Private Endpoint. |
| 28 | "BGP routing" | VPN Gateway or ExpressRoute | BGP is used for dynamic route exchange in VPN Gateway (BGP-enabled) and all ExpressRoute connections. |
| 29 | "IDPS" / "Intrusion Detection and Prevention" | Azure Firewall Premium | IDPS is only available in Firewall Premium SKU, not Standard. |
| 30 | "content trust" / "image signing" | Azure Container Registry (ACR) | Content trust uses Notary to sign and verify container images pushed to ACR. |
| 31 | "workload identity" / "pod identity" | AKS (Azure Kubernetes Service) | Workload identity allows pods to authenticate to Azure services using Entra ID without storing credentials. |
| 32 | "TLS inspection" / "SSL termination" | Azure Firewall Premium or Application Gateway | TLS inspection requires Premium Firewall (outbound) or App Gateway (inbound termination). |
| 33 | "hybrid identity" / "on-premises sync" | Microsoft Entra Connect (formerly Azure AD Connect) | Entra Connect synchronizes on-premises AD accounts to Entra ID for hybrid identity scenarios. |
| 34 | "access package" / "entitlement management" | Azure Identity Governance | Access packages bundle resources and are used for self-service access requests with approval workflows. |
| 35 | "threat intelligence feed" / "TI indicator" | Microsoft Sentinel Threat Intelligence | Sentinel can ingest IoCs (IPs, domains, hashes) from TI platforms via TAXII or native connectors. |
| 36 | "disk encryption at OS level" | Azure Disk Encryption (ADE) | ADE uses BitLocker (Windows) or dm-crypt (Linux) inside the VM OS. SSE works below the OS level. |
| 37 | "DDoS mitigation" / "volumetric attack" | Azure DDoS Protection Network/IP SKU | Basic is automatic; Network Protection adds adaptive tuning, telemetry, and SLA guarantees. |
| 38 | "cross-tenant access" / "B2B collaboration" | External Identities + Cross-Tenant Access Settings | Controls inbound/outbound B2B access between Entra ID tenants, including trust for MFA/device compliance. |

---

## ⚠️ Most Commonly Wrong — Top 10 Exam Traps

### Trap 1: NSG vs Azure Firewall

| | NSG | Azure Firewall |
|---|-----|----------------|
| **Layer** | Layer 3/4 (IP, port, protocol) | Layer 3–7 (including FQDN, HTTP headers) |
| **Scope** | Applied to subnet or NIC | Centralized hub service |
| **FQDN filtering** | ❌ No | ✅ Yes |
| **Threat intelligence** | ❌ No | ✅ Yes |
| **Cost** | Free | Per-hour + data processing |

**Wrong thinking:** "I need to block traffic, so I use NSG."  
**Correct understanding:** Use NSG for port/IP filtering within a VNet. Use Azure Firewall when you need FQDN filtering, centralized policy, or threat intelligence — especially for outbound internet traffic.

---

### Trap 2: Service Endpoint vs Private Endpoint

| | Service Endpoint | Private Endpoint |
|---|-----------------|-----------------|
| **Private IP in VNet** | ❌ No | ✅ Yes |
| **Traffic path** | Via Azure backbone (still "public" endpoint) | Entirely within VNet |
| **DNS change needed** | No | Yes (private DNS zone) |
| **Network isolation** | Partial | Full |
| **Cost** | Free | Per-hour charge |

**Wrong thinking:** "Service Endpoint gives a PaaS service a private IP."  
**Correct understanding:** Service Endpoint only extends your VNet's identity to the PaaS service; the service still has its public IP. Private Endpoint creates an actual private NIC inside your VNet.

---

### Trap 3: PIM vs JIT VM Access

| | PIM | JIT VM Access |
|---|-----|--------------|
| **What it protects** | Entra ID roles and Azure RBAC roles | VM management ports (RDP 3389, SSH 22) |
| **How it works** | Time-bound role activation with approval | Temporarily opens NSG rule for specific IP |
| **Where configured** | Entra ID — PIM blade | Defender for Cloud — JIT VM Access |
| **Requires** | Entra ID P2 | Defender for Servers plan |

**Wrong thinking:** "Just-in-time access always means PIM."  
**Correct understanding:** "Just-in-time VM access" = JIT in Defender for Cloud (NSG ports). "Just-in-time role activation" = PIM. Context in the question determines which.

---

### Trap 4: Defender for Cloud vs Microsoft Sentinel

| | Defender for Cloud | Microsoft Sentinel |
|---|--------------------|--------------------|
| **Primary function** | Security posture + threat protection | SIEM + SOAR |
| **Secure Score** | ✅ Yes | ❌ No |
| **Incidents & investigations** | Limited | ✅ Full SOC workflow |
| **KQL analytics rules** | ❌ No | ✅ Yes |
| **Playbook automation** | Limited | ✅ Full Logic App integration |
| **Scope** | Azure resources primarily | Any data source (multi-cloud, on-prem) |

**Wrong thinking:** "Defender for Cloud is my SIEM."  
**Correct understanding:** Defender for Cloud is posture management + workload protection (CSPM + CWPP). Sentinel is the SIEM/SOAR for investigation and response. They complement each other — Defender alerts feed into Sentinel.

---

### Trap 5: Key Vault Access Policy vs RBAC

| | Access Policy | Azure RBAC |
|---|--------------|------------|
| **Scope** | Key Vault resource level | Key, Secret, Certificate level (granular) |
| **Management plane** | RBAC always | RBAC always |
| **Data plane** | Access Policy OR RBAC | Access Policy OR RBAC |
| **Recommended** | Legacy | ✅ Microsoft recommended (newer) |

**Wrong thinking:** "Access Policies and RBAC do the same thing."  
**Correct understanding:** Both control **data plane** access to Key Vault. RBAC is preferred as it supports finer-grained permissions (e.g., `Key Vault Secrets User` = read secrets only). Access Policies are all-or-nothing per object type.

---

### Trap 6: ADE vs SSE (Disk Encryption)

| | ADE (Azure Disk Encryption) | SSE (Server-Side Encryption) |
|---|-----------------------------|-----------------------------|
| **Where encryption happens** | Inside the VM OS (BitLocker/dm-crypt) | Azure Storage layer (below OS) |
| **Protects from** | Azure platform staff with storage access | Unauthorized physical disk access |
| **Key management** | Key Vault required | Platform-managed or CMK in Key Vault |
| **VM restart needed** | Yes | No |

**Wrong thinking:** "SSE and ADE are equivalent."  
**Correct understanding:** SSE protects data at the storage layer and is always on. ADE adds OS-level encryption, protecting against scenarios where an attacker gains raw disk access outside the VM (e.g., disk detach).

---

### Trap 7: B2B vs B2C

| | B2B (Business-to-Business) | B2C (Business-to-Customer) |
|---|---------------------------|---------------------------|
| **Use case** | Partner/vendor access to your apps | Customer-facing apps (millions of users) |
| **Identity** | Guest uses their own org credentials | Customer creates a new account (social/local) |
| **Directory** | Guest object in your Entra ID tenant | Separate Azure AD B2C tenant |
| **Scale** | Hundreds to thousands | Millions of consumers |

**Wrong thinking:** "B2B and B2C are just different flavors of guest access."  
**Correct understanding:** B2B is for collaborating with **other organizations**. B2C is a completely separate service for **consumer-facing applications** with custom user flows and branding.

---

### Trap 8: Automation Rule vs Playbook (Sentinel)

| | Automation Rule | Playbook (Logic App) |
|---|----------------|---------------------|
| **Runs when** | Incident created/updated | Triggered by automation rule or manually |
| **Actions** | Assign, change status, add tag, run playbook | Any Logic App action (notify, block, ticket) |
| **Logic** | Simple conditions | Complex multi-step workflows |
| **Without the other** | Can run without playbook (e.g., auto-assign) | Must be triggered by something |

**Wrong thinking:** "Playbook = automation rule."  
**Correct understanding:** An automation rule is the **trigger and condition** layer. A playbook is the **action** layer. Automation rules can call playbooks, but they are distinct components.

---

### Trap 9: Azure AD Roles vs Azure RBAC Roles

| | Entra ID (Azure AD) Roles | Azure RBAC Roles |
|---|--------------------------|-----------------|
| **Scope** | Entra ID tenant (directory level) | Azure resources (subscription/RG/resource) |
| **Examples** | Global Admin, Security Admin, User Admin | Owner, Contributor, Reader, custom roles |
| **Manages** | Users, groups, apps, devices, policies | VMs, storage, networking, databases |
| **Defined in** | Entra ID portal | Azure portal / ARM |

**Wrong thinking:** "Global Admin can do anything in Azure."  
**Correct understanding:** Global Admin manages the **directory**. To manage Azure **resources**, you need Azure RBAC. A Global Admin can *elevate* to User Access Administrator in Azure once, but they're separate systems.

---

### Trap 10: Audit Effect vs Deny Effect (Azure Policy)

| | Audit | Deny |
|---|-------|------|
| **Blocks deployment** | ❌ No | ✅ Yes |
| **Creates compliance record** | ✅ Yes | ✅ Yes |
| **Use case** | Visibility / monitoring without enforcement | Hard enforcement |
| **Risk** | Existing non-compliant resources remain | Could break deployments |

**Wrong thinking:** "Audit enforces the policy rule."  
**Correct understanding:** Audit only **reports** non-compliance — it never blocks anything. Deny **prevents** the non-compliant resource from being created or modified. Use Audit first to understand impact before switching to Deny.

---

## ⚡ Quick Decision Framework

Use these mental shortcuts when reading exam questions:

| When you see... | Think... |
|----------------|---------|
| "encrypt data **at rest** in VM disk" | ADE (OS-level) or SSE (storage-level) — read for CMK or OS-level context |
| "encrypt data **at rest** in SQL" | TDE (transparent, default on) |
| "encrypt specific **columns** in SQL" | Always Encrypted (client-side, hides from DBAs) |
| "hide data from non-privileged users" | Dynamic Data Masking (DDM) |
| "encrypt data **in transit**" | TLS 1.2+ enforcement, HTTPS-only settings |
| "**identity** protection / risky sign-in" | Entra ID Identity Protection + Conditional Access |
| "block inbound traffic by **port**" | NSG (quick, cheap, port-level) |
| "block outbound traffic by **domain**" | Azure Firewall (FQDN rules) |
| "protect web app from **SQL injection**" | WAF (OWASP rules) — not Azure Firewall |
| "**monitor all traffic** / diagnose connectivity" | Network Watcher + NSG Flow Logs |
| "**investigate** security incidents" | Microsoft Sentinel |
| "**posture management** / what's my score" | Defender for Cloud Secure Score |
| "**automate** response to incidents" | Sentinel Playbook (Logic App) |
| "**alert** on specific log pattern" | Sentinel Analytics Rule (KQL) |
| "**least privilege** access for a period of time" | PIM (eligible assignment) |
| "**no public IP** but need remote access" | Azure Bastion |
| "**private connectivity** to PaaS service" | Private Endpoint (gives private IP) |
| "**restrict** PaaS to specific VNet" | Service Endpoint (no private IP, but restricts access) |
| "**approve** access requests" | PIM approval workflow or Entitlement Management |
| "**guest user** from another company" | B2B External Identities |
| "**millions** of consumer users" | Azure AD B2C (separate tenant) |
| "**automatically deploy** missing config" | Azure Policy — DeployIfNotExists effect |
| "**block** non-compliant resources" | Azure Policy — Deny effect |
| "**report** non-compliance without blocking" | Azure Policy — Audit effect |

---

📚 **Further Reading:** https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-500  
🔄 **Last Verified:** 2026 (AZ-500 January 2026 objectives)
