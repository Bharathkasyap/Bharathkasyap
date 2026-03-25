# ⚡ AZ-500 — Quick Reference Cheatsheet

> **Purpose:** Last-minute review. One-liners for every service, key port numbers, encryption mapping, and CLI commands.

---

## Section 1: Master Service Table

| Service | One-Line Definition | Exam Trap | Similar To |
|---------|--------------------|-----------|-----------:|
| **Microsoft Entra ID** | Cloud identity provider (authentication, SSO, MFA) | Not the same as Azure RBAC — controls directory, not resources | Active Directory (on-prem) |
| **Conditional Access** | Policy engine: signal → decision → enforcement for sign-in | "Report-only" mode does not enforce; named location ≠ MFA | Zero Trust gateway |
| **Azure RBAC** | Role-based access control for Azure resources (subscription/RG/resource) | Global Admin ≠ Owner; scope inheritance goes subscription→RG→resource | Entra ID roles (separate system) |
| **PIM (Privileged Identity Management)** | Time-bound, approval-based role activation with audit trail | PIM = roles; JIT VM Access = VM ports. Both are "just-in-time" but different | JIT VM Access |
| **Identity Governance / Access Reviews** | Automated review cycle to certify who still needs access | Access reviews can be delegated to resource owners or managers | Manual access audits |
| **Managed Identities** | Auto-managed Entra ID identity for Azure resources — no credentials | System-assigned (tied to resource lifecycle) vs user-assigned (reusable) | Service Principal |
| **App Registration / Service Principal** | Manual Entra ID identity for applications — client ID + secret/cert | App Registration = the template; Service Principal = the tenant instance | Managed Identity (no secret needed) |
| **External Identities (B2B)** | Invite external users (partners) to your tenant using their own credentials | B2B guest authenticates with **home tenant**, not yours | B2C (consumer-facing, different service) |
| **External Identities (B2C)** | Separate Azure AD B2C tenant for consumer-facing apps at scale | Completely separate tenant — not a feature of your main Entra ID | B2B (for business partners) |
| **Virtual Network (VNet)** | Isolated private network in Azure (subnets, address spaces, routing) | Peered VNets don't share NSGs — each VNet/subnet has its own | On-prem VLAN |
| **NSG (Network Security Group)** | Layer 3/4 firewall rules: allow/deny by IP, port, protocol | Default rules exist (DenyAllInbound); lower priority number = higher priority | Azure Firewall (different layer/purpose) |
| **ASG (Application Security Group)** | Logical grouping of VMs for use in NSG rules (replaces IP addresses) | ASG simplifies rules — no need to maintain IP lists | NSG tags |
| **Azure Firewall** | Managed, stateful, L3-L7 network firewall with FQDN filtering and TI | Standard = FQDN; Premium = IDPS + TLS inspection + URL filtering | NSG (different scope — centralized vs per-subnet) |
| **WAF (Web Application Firewall)** | L7 HTTP/S protection using OWASP CRS rules against web attacks | WAF is for HTTP attacks (SQLi, XSS); Azure Firewall is for network-level | Azure Firewall (different attack vectors) |
| **Application Gateway** | L7 regional load balancer with optional WAF, SSL termination, URL routing | Regional only; use Front Door for global load balancing | Azure Front Door |
| **Azure Front Door** | Global CDN + L7 load balancer + WAF at the edge (Microsoft POP locations) | Global scope; WAF on Front Door has slightly different rule sets than App Gateway WAF | Application Gateway (regional) |
| **DDoS Protection** | Absorbs volumetric DDoS attacks against Azure public IPs | Basic is automatic and free; Network Protection adds adaptive tuning and SLA | On-prem DDoS scrubbing |
| **Azure Bastion** | Browser-based RDP/SSH over TLS to VMs with no public IP required | Deployed per VNet in a dedicated AzureBastionSubnet (/26 min) | Jump box / jump server |
| **JIT VM Access** | On-demand, time-limited NSG port opening for VM management | JIT = VM ports; PIM = directory roles. Both "just-in-time" — context matters | PIM |
| **Private Endpoint** | Private NIC in your VNet with private IP pointing to a PaaS service | Requires private DNS zone for name resolution | Service Endpoint (no private IP) |
| **Service Endpoint** | Extends VNet identity to PaaS service over Azure backbone | Does NOT give the service a private IP; less secure than Private Endpoint | Private Endpoint |
| **VPN Gateway** | Site-to-site or P2S encrypted VPN tunnel over public internet (IPsec/IKEv2) | Max bandwidth limited by SKU; not suitable for high-throughput workloads | ExpressRoute (private, not over internet) |
| **ExpressRoute** | Private, dedicated circuit from on-premises to Azure (no internet) | Does NOT encrypt by default (private physical circuit) — add MACsec for encryption | VPN Gateway (internet-based) |
| **Virtual WAN** | Microsoft-managed hub-and-spoke network with automated routing | Use for large-scale branch connectivity; Standard tier required for firewall integration | Manual hub-spoke topology |
| **Network Watcher** | Diagnostic toolset: IP Flow Verify, Connection Monitor, Packet Capture | Network Watcher ≠ monitoring — it's a diagnostics tool | Azure Monitor (metrics/alerts) |
| **NSG Flow Logs** | Records all inbound/outbound flows through NSGs to Storage Account | Version 2 adds bytes/packets; Traffic Analytics requires Log Analytics workspace | Azure Monitor logs |
| **VM Disk Encryption (ADE)** | BitLocker (Windows) or dm-crypt (Linux) encryption inside the VM OS | ADE = OS-level; SSE = storage-layer. Both can use CMK in Key Vault | SSE (different layer) |
| **AKS Security** | Kubernetes cluster security: RBAC, network policies, workload identity, Defender | Enable RBAC + Entra ID integration; private cluster hides API server | Generic Kubernetes |
| **Container Registry + Defender for Containers** | ACR stores container images; Defender scans for vulnerabilities | Enable content trust for image signing; use private endpoint for ACR | Docker Hub (public) |
| **App Service Security** | HTTPS-only, TLS min version, managed identity, VNet integration, auth middleware | Use managed identity + Key Vault references instead of storing secrets in app settings | IIS / web hosting |
| **Storage Account Security** | Secure blob/file/queue/table storage: HTTPS, Azure AD auth, SAS tokens, private endpoint | Disable anonymous access; prefer Azure AD auth over SAS; user-delegation SAS is most secure | On-prem file server |
| **Blob SAS Tokens** | Shared Access Signatures grant time-limited, scoped access to storage | User-delegation SAS (Entra ID) > Account SAS > Service SAS in security level | Storage Account keys (full access) |
| **Azure Key Vault** | Managed secrets, keys, and certificate store with HSM option | Enable soft delete + purge protection; use RBAC (not Access Policies) for data plane | AWS KMS |
| **Azure SQL Security (TDE)** | Transparent Data Encryption — encrypts database files at rest by default | TDE is on by default; CMK option lets you manage the key | Always Encrypted (different purpose) |
| **Azure SQL Security (Always Encrypted)** | Client-side column encryption — even DBAs cannot see plaintext data | Requires client-side driver; protects from privileged DB users | DDM (which only masks, doesn't encrypt) |
| **Azure SQL Security (DDM)** | Dynamic Data Masking — replaces data in query results for non-privileged users | DDM does NOT encrypt; real data is still stored in plaintext | Always Encrypted (actual encryption) |
| **API Management Security** | Gateway for APIs: OAuth 2.0, JWT validation, IP filtering, rate limiting | APIM is the control point for API security; backend APIs still need their own auth | Azure Front Door (different layer) |
| **Defender for Cloud** | CSPM + CWPP — security posture assessment and threat protection for workloads | Secure Score lives here, NOT in Sentinel; free tier = CSPM; paid tier = CWPP | Microsoft Sentinel (SIEM, different purpose) |
| **Secure Score** | Percentage score showing security posture based on recommendation compliance | Affected by recommendations, NOT by alerts | Compliance score (Defender CSPM) |
| **Defender Plans (Servers, Storage, SQL, Containers, DNS)** | Per-resource paid plans enabling threat detection for specific workload types | Each plan has separate pricing; enabling all = high cost | Defender for Cloud free tier |
| **Azure Policy + Initiatives** | Govern resource configurations; enforce, audit, or auto-remediate compliance | Audit = report only; Deny = block; DeployIfNotExists = auto-deploy; Initiative = group of policies | ARM templates (provisioning, not governance) |
| **Microsoft Sentinel** | Cloud-native SIEM + SOAR: collect, detect, investigate, respond | Sentinel needs a Log Analytics Workspace; it IS the workspace | Defender for Cloud (posture, not SIEM) |
| **Sentinel Data Connectors** | Plugins that ingest log data from sources into the Log Analytics Workspace | Native connectors (Entra ID, Azure); CEF for third-party; Syslog for Linux; API for custom | Azure Monitor Data Collection Rules |
| **KQL (Kusto Query Language)** | Query language for Log Analytics / Sentinel analytics rules / workbooks | KQL is used in analytics rules, hunting, workbooks — exam tests basic syntax | SQL (different syntax) |
| **Analytics Rules + Incidents** | KQL-based rules that fire alerts and create/group incidents in Sentinel | Scheduled / NRT / Fusion / ML / TI — know the difference | Defender for Cloud alerts |
| **Playbooks + Automation Rules** | Playbooks = Logic App workflows; Automation Rules = Sentinel-native triggers | Automation rules can run without playbooks (e.g., auto-assign); playbooks are triggered by automation rules | Power Automate flows |
| **Workbooks** | Interactive dashboards in Sentinel / Azure Monitor using KQL visualizations | For visualization, not alerting | Power BI dashboards |

---

## Section 2: Port Number Quick Reference

| Port | Protocol | Use | Security Note |
|------|----------|-----|---------------|
| **22** | SSH | Remote shell access to Linux VMs | Block with NSG; use Azure Bastion instead; JIT if needed |
| **443** | HTTPS | Secure web traffic (TLS) | Always enforce HTTPS; minimum TLS 1.2 |
| **80** | HTTP | Unencrypted web traffic | Redirect to 443; never allow 80 as final endpoint |
| **3389** | RDP | Remote Desktop to Windows VMs | Block with NSG; use Azure Bastion or JIT VM Access |
| **1433** | TCP | SQL Server (Azure SQL) | Restrict with NSG or private endpoint; disable public endpoint |
| **8443** | HTTPS (alt) | Alternative HTTPS port for internal services | Same TLS requirements as 443 |
| **5671** | AMQP (TLS) | Azure Event Hubs / Service Bus (encrypted) | Use over unencrypted 5672 |
| **5672** | AMQP | Azure Event Hubs / Service Bus (unencrypted) | Prefer 5671 (TLS); block 5672 where possible |
| **445** | SMB | Azure Files (SMB protocol) | Requires firewall allow for port 445 outbound; use private endpoint |
| **636** | LDAPS | LDAP over SSL (AD DS) | Use 636 instead of 389 for secure LDAP |

---

## Section 3: Encryption Quick Reference

| Data State | Location / Scenario | Azure Service | Encryption Type | Key Management |
|------------|---------------------|---------------|-----------------|----------------|
| **At rest** | VM OS/data disk | Azure Disk Encryption (ADE) | BitLocker / dm-crypt (OS-level) | Key Vault (KEK) |
| **At rest** | VM disk (storage layer) | SSE (Server-Side Encryption) | AES-256 (storage platform) | PMK (default) or CMK in Key Vault |
| **At rest** | Azure SQL database | TDE (Transparent Data Encryption) | AES-256 (database files) | Service-managed or CMK in Key Vault |
| **At rest** | SQL specific columns | Always Encrypted | AES-256 (client-side, column-level) | Column Master Key in Key Vault or cert store |
| **At rest** | Storage blobs/files | SSE for Storage | AES-256 | PMK (default) or CMK in Key Vault |
| **At rest** | Secrets / keys themselves | Azure Key Vault | Software HSM (Standard) or FIPS 140-2 Level 3 HSM (Premium) | Customer-owned |
| **In transit** | All Azure services | TLS 1.2+ | TLS (asymmetric + symmetric) | Certificate (Azure-managed or custom) |
| **In transit** | Storage Account | HTTPS enforcement | TLS | Enforced at account level |
| **In transit** | App Service | HTTPS-only setting | TLS min 1.2 | Managed certificate option available |
| **Columns (masking)** | Azure SQL — non-privileged users | Dynamic Data Masking (DDM) | Masking only (NOT encryption) | N/A — real data still plaintext |

---

## Section 4: Key CLI Commands

```bash
# ─── Identity & Access ────────────────────────────────────────────────────────

# List all role assignments in a subscription
az role assignment list --subscription <sub-id> --output table

# Create a custom role from a JSON definition file
az role definition create --role-definition @custom-role.json

# Assign a built-in role to a user at resource group scope
az role assignment create \
  --assignee <user-email-or-object-id> \
  --role "Contributor" \
  --scope /subscriptions/<sub-id>/resourceGroups/<rg-name>

# ─── Key Vault ────────────────────────────────────────────────────────────────

# Create a Key Vault with soft delete and purge protection enabled
az keyvault create \
  --name <kv-name> \
  --resource-group <rg-name> \
  --location eastus \
  --enable-soft-delete true \
  --enable-purge-protection true

# Set a secret in Key Vault
az keyvault secret set --vault-name <kv-name> --name "MySecret" --value "MyValue"

# ─── Storage Account ──────────────────────────────────────────────────────────

# Disable anonymous blob access on a storage account
az storage account update \
  --name <storage-name> \
  --resource-group <rg-name> \
  --allow-blob-public-access false

# Create a user-delegation SAS token (requires Entra ID auth)
az storage blob generate-sas \
  --account-name <storage-name> \
  --container-name <container> \
  --name <blob-name> \
  --permissions r \
  --expiry 2025-12-31 \
  --auth-mode login \
  --as-user

# ─── Networking ───────────────────────────────────────────────────────────────

# Create a Network Security Group
az network nsg create --name <nsg-name> --resource-group <rg-name>

# Add an NSG rule to deny all inbound on port 3389 except specific IP
az network nsg rule create \
  --nsg-name <nsg-name> \
  --resource-group <rg-name> \
  --name DenyRDP \
  --priority 100 \
  --direction Inbound \
  --access Deny \
  --protocol Tcp \
  --destination-port-ranges 3389

# Enable NSG Flow Logs version 2
az network watcher flow-log create \
  --resource-group <rg-name> \
  --nsg <nsg-name> \
  --storage-account <storage-account-id> \
  --workspace <log-analytics-workspace-id> \
  --format JSON \
  --log-version 2

# ─── Defender for Cloud ───────────────────────────────────────────────────────

# Enable Defender for Servers plan on a subscription
az security pricing create \
  --name VirtualMachines \
  --tier Standard

# List Defender for Cloud security recommendations
az security assessment list --output table

# ─── Microsoft Sentinel ───────────────────────────────────────────────────────

# Enable Microsoft Sentinel on a Log Analytics workspace
az sentinel workspace onboard \
  --resource-group <rg-name> \
  --workspace-name <workspace-name>

# List Sentinel analytics rules
az sentinel alert-rule list \
  --resource-group <rg-name> \
  --workspace-name <workspace-name> \
  --output table
```

---

📚 **Further Reading:** https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-500  
🔄 **Last Verified:** 2026 (AZ-500 January 2026 objectives)
