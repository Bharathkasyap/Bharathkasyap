# 📅 AZ-500 — 30-Day Study Plan

> **Goal:** Pass the AZ-500 Azure Security Engineer Associate exam in 30 days.  
> **Daily Commitment:** 2–3 hours per day  
> **Structure:** 4 domain weeks + 2-day final review sprint

---

## 🗓️ Quick Reference Table

| Day | Domain | Topic | File | Task | Time |
|-----|--------|-------|------|------|------|
| 1 | Identity & Access | Entra ID Fundamentals | [01-EntraID-and-Azure-AD.md](../Domain1-Identity-Access/01-EntraID-and-Azure-AD.md) | Read + enable MFA in sandbox | 2h |
| 2 | Identity & Access | Conditional Access | [02-Conditional-Access.md](../Domain1-Identity-Access/02-Conditional-Access.md) | Create CA policy (block legacy auth) | 2h |
| 3 | Identity & Access | Azure RBAC | [03-Azure-RBAC.md](../Domain1-Identity-Access/03-Azure-RBAC.md) | Assign custom role via CLI | 2h |
| 4 | Identity & Access | PIM | [04-PIM.md](../Domain1-Identity-Access/04-PIM.md) | Enable eligible assignment + activate role | 3h |
| 5 | Identity & Access | Identity Governance | [05-Identity-Governance.md](../Domain1-Identity-Access/05-Identity-Governance.md) | Create access review | 2h |
| 6 | Identity & Access | Managed Identities + App Reg | [06-Managed-Identities.md](../Domain1-Identity-Access/06-Managed-Identities.md) | Assign system-assigned identity to VM | 2h |
| 7 | Identity & Access | External Identities B2B/B2C | [07-External-Identities.md](../Domain1-Identity-Access/07-External-Identities.md) | Invite B2B guest, review cross-tenant policy | 2h |
| 8 | Secure Networking | VNet + NSG + ASG | [08-VNet-NSG-ASG.md](../Domain2-Secure-Networking/08-VNet-NSG-ASG.md) | Create VNet, NSG rules, ASG group | 2h |
| 9 | Secure Networking | Azure Firewall | [09-Azure-Firewall.md](../Domain2-Secure-Networking/09-Azure-Firewall.md) | Deploy Azure Firewall, FQDN rule | 3h |
| 10 | Secure Networking | WAF + App Gateway + Front Door | [10-WAF-AppGateway-FrontDoor.md](../Domain2-Secure-Networking/10-WAF-AppGateway-FrontDoor.md) | Enable WAF policy, test OWASP rules | 2h |
| 11 | Secure Networking | DDoS Protection + Bastion | [11-DDoS-Bastion.md](../Domain2-Secure-Networking/11-DDoS-Bastion.md) | Enable DDoS Standard, deploy Bastion | 2h |
| 12 | Secure Networking | Private Endpoint + Service Endpoint | [12-Private-Endpoint.md](../Domain2-Secure-Networking/12-Private-Endpoint.md) | Create private endpoint for Storage | 2h |
| 13 | Secure Networking | VPN Gateway + ExpressRoute | [13-VPN-Gateway-ExpressRoute.md](../Domain2-Secure-Networking/13-VPN-Gateway-ExpressRoute.md) | Configure site-to-site VPN (conceptual) | 2h |
| 14 | Secure Networking | Network Watcher + Flow Logs | [14-Network-Watcher.md](../Domain2-Secure-Networking/14-Network-Watcher.md) | Enable NSG Flow Logs, review Traffic Analytics | 2h |
| 15 | Compute & Storage | VM Disk Encryption (ADE) | [15-VM-Security.md](../Domain3-Compute-Storage/15-VM-Security.md) | Enable ADE with Key Vault key | 2h |
| 16 | Compute & Storage | JIT VM Access | [16-JIT-Access.md](../Domain3-Compute-Storage/16-JIT-Access.md) | Enable JIT, request access for RDP | 2h |
| 17 | Compute & Storage | AKS Security | [17-AKS-Security.md](../Domain3-Compute-Storage/17-AKS-Security.md) | Enable RBAC on AKS, configure workload identity | 3h |
| 18 | Compute & Storage | Container Registry Security | [18-Container-Registry.md](../Domain3-Compute-Storage/18-Container-Registry.md) | Enable content trust, Defender for Containers | 2h |
| 19 | Compute & Storage | App Service Security | [19-App-Service-Security.md](../Domain3-Compute-Storage/19-App-Service-Security.md) | Enable managed identity, HTTPS only | 2h |
| 20 | Compute & Storage | Storage Security + SAS | [20-Storage-Security.md](../Domain3-Compute-Storage/20-Storage-Security.md) | Create user-delegation SAS, disable anonymous access | 2h |
| 21 | Compute & Storage | Key Vault + SQL Security | [21-Key-Vault.md](../Domain3-Compute-Storage/21-Key-Vault.md) | Enable soft delete, purge protection, TDE | 3h |
| 22 | Defender & Sentinel | Defender for Cloud Fundamentals | [22-Defender-for-Cloud.md](../Domain4-Defender-Sentinel/22-Defender-for-Cloud.md) | Enable CSPM, review Secure Score | 2h |
| 23 | Defender & Sentinel | Defender Plans | [23-Defender-Plans.md](../Domain4-Defender-Sentinel/23-Defender-Plans.md) | Enable Defender for Servers + Storage | 2h |
| 24 | Defender & Sentinel | Azure Policy + Initiatives | [24-Azure-Policy.md](../Domain4-Defender-Sentinel/24-Azure-Policy.md) | Assign built-in policy, create initiative | 2h |
| 25 | Defender & Sentinel | Sentinel Setup + Data Connectors | [25-Sentinel-Setup.md](../Domain4-Defender-Sentinel/25-Sentinel-Setup.md) | Connect Entra ID + Activity Log to Sentinel | 3h |
| 26 | Defender & Sentinel | KQL Fundamentals | [26-KQL-Fundamentals.md](../Domain4-Defender-Sentinel/26-KQL-Fundamentals.md) | Write 5 KQL queries, create analytics rule | 3h |
| 27 | Defender & Sentinel | Sentinel Analytics Rules + Incidents | [27-Sentinel-Analytics.md](../Domain4-Defender-Sentinel/27-Sentinel-Analytics.md) | Create scheduled rule, simulate alert | 2h |
| 28 | Defender & Sentinel | Playbooks + Automation Rules | [28-Sentinel-Automation.md](../Domain4-Defender-Sentinel/28-Sentinel-Automation.md) | Create automation rule, trigger playbook | 2h |
| 29 | Full Review | Trigger Words + Cheatsheet | [Trigger-Words-and-Traps.md](./Trigger-Words-and-Traps.md) + [Quick-Reference-Cheatsheet.md](./Quick-Reference-Cheatsheet.md) | Review all cheatsheets, identify weak spots | 3h |
| 30 | Final Sprint | Practice Tests + Exam Tips | All files | Full practice exam, review wrong answers | 3h |

---

## 📆 Week 1 (Days 1–7): Domain 1 — Identity and Access

**Theme:** Master the identity plane — authentication, authorization, and governance.

### Day 1 — Entra ID Fundamentals
- **Read:** [01-EntraID-and-Azure-AD.md](../Domain1-Identity-Access/01-EntraID-and-Azure-AD.md)
- **Lab:** Enable MFA for a user in the Microsoft Entra admin center (free sandbox at [learn.microsoft.com](https://learn.microsoft.com))
- **Focus:** Tenant vs subscription, Entra ID tiers (Free / P1 / P2), directory roles
- **Practice Questions:** What features require Entra ID P2? What is the difference between Entra ID and Azure RBAC?
- ⏱ **Time:** 2 hours

### Day 2 — Conditional Access
- **Read:** [02-Conditional-Access.md](../Domain1-Identity-Access/02-Conditional-Access.md)
- **Lab:** Create a Conditional Access policy blocking legacy authentication protocols
- **Focus:** Signal → Decision → Enforcement model; Named Locations; sign-in risk levels
- **Practice Questions:** When does Conditional Access enforce MFA? What is "report-only" mode?
- ⏱ **Time:** 2 hours

### Day 3 — Azure RBAC
- **Read:** [03-Azure-RBAC.md](../Domain1-Identity-Access/03-Azure-RBAC.md)
- **Lab:** Create a custom role with JSON definition using `az role definition create`
- **Focus:** Built-in roles (Owner/Contributor/Reader), scope hierarchy, deny assignments
- **Practice Questions:** What is the difference between Azure RBAC and Entra ID roles? What is a deny assignment?
- ⏱ **Time:** 2 hours

### Day 4 — Privileged Identity Management (PIM)
- **Read:** [04-PIM.md](../Domain1-Identity-Access/04-PIM.md)
- **Lab:** Configure an eligible assignment and activate a role with justification
- **Focus:** Eligible vs active assignments, approval workflows, access reviews in PIM
- **Practice Questions:** What triggers an alert in PIM? What is the difference between PIM and JIT VM Access?
- ⏱ **Time:** 3 hours

### Day 5 — Identity Governance
- **Read:** [05-Identity-Governance.md](../Domain1-Identity-Access/05-Identity-Governance.md)
- **Lab:** Create an access review for group members
- **Focus:** Access packages, entitlement management, access reviews, lifecycle workflows
- **Practice Questions:** Who can approve access packages? What happens when an access review expires?
- ⏱ **Time:** 2 hours

### Day 6 — Managed Identities + App Registration
- **Read:** [06-Managed-Identities.md](../Domain1-Identity-Access/06-Managed-Identities.md)
- **Lab:** Assign a system-assigned managed identity to a VM and grant Key Vault access
- **Focus:** System-assigned vs user-assigned, service principal vs managed identity
- **Practice Questions:** When should you use managed identity vs service principal? How does managed identity authenticate?
- ⏱ **Time:** 2 hours

### Day 7 — External Identities (B2B / B2C)
- **Read:** [07-External-Identities.md](../Domain1-Identity-Access/07-External-Identities.md)
- **Lab:** Invite a B2B guest user and apply a Conditional Access policy to guests
- **Focus:** B2B vs B2C differences, cross-tenant access settings, redemption flow
- **Practice Questions:** What authentication does a B2B guest use? What is B2C used for?
- ⏱ **Time:** 2 hours

---

## 📆 Week 2 (Days 8–14): Domain 2 — Secure Networking

**Theme:** Control the network perimeter — from VNets to firewalls to private connectivity.

### Day 8 — VNet + NSG + ASG
- **Read:** [08-VNet-NSG-ASG.md](../Domain2-Secure-Networking/08-VNet-NSG-ASG.md)
- **Lab:** Create a VNet with two subnets, add NSG rules, assign ASG to a VM NIC
- **Focus:** NSG rule priority, default rules, ASG as source/destination
- **Practice Questions:** What is the default NSG inbound behavior? How does ASG simplify NSG rules?
- ⏱ **Time:** 2 hours

### Day 9 — Azure Firewall
- **Read:** [09-Azure-Firewall.md](../Domain2-Secure-Networking/09-Azure-Firewall.md)
- **Lab:** Deploy Azure Firewall Standard, create FQDN application rule
- **Focus:** DNAT/SNAT/Application/Network rules, FQDN filtering, Firewall Policy vs Classic
- **Practice Questions:** What can Azure Firewall do that NSG cannot? When do you need Firewall Premium?
- ⏱ **Time:** 3 hours

### Day 10 — WAF + Application Gateway + Front Door
- **Read:** [10-WAF-AppGateway-FrontDoor.md](../Domain2-Secure-Networking/10-WAF-AppGateway-FrontDoor.md)
- **Lab:** Deploy Application Gateway with WAF policy in Detection mode
- **Focus:** WAF on App Gateway vs WAF on Front Door, OWASP core rule sets, custom rules
- **Practice Questions:** What layer does WAF operate at? When use Front Door vs App Gateway?
- ⏱ **Time:** 2 hours

### Day 11 — DDoS Protection + Azure Bastion
- **Read:** [11-DDoS-Bastion.md](../Domain2-Secure-Networking/11-DDoS-Bastion.md)
- **Lab:** Deploy Azure Bastion in a VNet; review DDoS protection plan tiers
- **Focus:** DDoS Basic vs Network Protection vs IP Protection, Bastion tiers
- **Practice Questions:** What does DDoS Standard protect that Basic doesn't? Why use Bastion over a jump server?
- ⏱ **Time:** 2 hours

### Day 12 — Private Endpoint + Service Endpoint
- **Read:** [12-Private-Endpoint.md](../Domain2-Secure-Networking/12-Private-Endpoint.md)
- **Lab:** Create a Private Endpoint for a Storage Account; verify private DNS zone
- **Focus:** Private Endpoint (private IP in VNet) vs Service Endpoint (VNet identity extension)
- **Practice Questions:** Does a Service Endpoint give the service a private IP? Which is more secure?
- ⏱ **Time:** 2 hours

### Day 13 — VPN Gateway + ExpressRoute
- **Read:** [13-VPN-Gateway-ExpressRoute.md](../Domain2-Secure-Networking/13-VPN-Gateway-ExpressRoute.md)
- **Lab:** Review site-to-site VPN configuration (diagram the topology)
- **Focus:** IKEv2/IPsec, BGP routing, ExpressRoute circuits, coexistence scenarios
- **Practice Questions:** What is the encryption difference between VPN Gateway and ExpressRoute? What is ExpressRoute Direct?
- ⏱ **Time:** 2 hours

### Day 14 — Network Watcher + Flow Logs
- **Read:** [14-Network-Watcher.md](../Domain2-Secure-Networking/14-Network-Watcher.md)
- **Lab:** Enable NSG Flow Logs version 2; review Traffic Analytics workspace
- **Focus:** Flow Logs, Connection Monitor, IP Flow Verify, Packet Capture
- **Practice Questions:** What storage does NSG Flow Log use? What does Traffic Analytics add?
- ⏱ **Time:** 2 hours

---

## 📆 Week 3 (Days 15–21): Domain 3 — Compute, Storage, and Databases

**Theme:** Protect workloads — VMs, containers, storage, and data services.

### Day 15 — VM Disk Encryption (ADE)
- **Read:** [15-VM-Security.md](../Domain3-Compute-Storage/15-VM-Security.md)
- **Lab:** Enable Azure Disk Encryption on a Windows VM using Key Vault
- **Focus:** ADE (BitLocker/dm-crypt) vs SSE (platform-managed), CMK vs PMK
- **Practice Questions:** What is the difference between ADE and SSE? What does ADE protect that SSE doesn't?
- ⏱ **Time:** 2 hours

### Day 16 — JIT VM Access
- **Read:** [16-JIT-Access.md](../Domain3-Compute-Storage/16-JIT-Access.md)
- **Lab:** Enable JIT VM Access in Defender for Cloud, request RDP access
- **Focus:** JIT vs PIM, time-bound NSG rules, required permissions
- **Practice Questions:** How does JIT modify NSG rules? Who can approve JIT requests?
- ⏱ **Time:** 2 hours

### Day 17 — AKS Security
- **Read:** [17-AKS-Security.md](../Domain3-Compute-Storage/17-AKS-Security.md)
- **Lab:** Enable RBAC + Azure AD integration on AKS, configure pod-managed identity
- **Focus:** Workload identity, network policies, private cluster, Defender for Containers
- **Practice Questions:** What is workload identity in AKS? What is a network policy?
- ⏱ **Time:** 3 hours

### Day 18 — Container Registry Security
- **Read:** [18-Container-Registry.md](../Domain3-Compute-Storage/18-Container-Registry.md)
- **Lab:** Enable content trust on ACR, run Defender for Containers scan
- **Focus:** Content trust, image signing, vulnerability scanning, private endpoint for ACR
- **Practice Questions:** What does content trust prevent? What does Defender for Containers scan?
- ⏱ **Time:** 2 hours

### Day 19 — App Service Security
- **Read:** [19-App-Service-Security.md](../Domain3-Compute-Storage/19-App-Service-Security.md)
- **Lab:** Enable HTTPS only, configure managed identity, add Key Vault reference
- **Focus:** TLS minimum version, managed identity for Key Vault, VNet integration, auth middleware
- **Practice Questions:** How do you use Key Vault from App Service without storing credentials?
- ⏱ **Time:** 2 hours

### Day 20 — Storage Account Security + SAS
- **Read:** [20-Storage-Security.md](../Domain3-Compute-Storage/20-Storage-Security.md)
- **Lab:** Create a user delegation SAS token via CLI; disable anonymous access
- **Focus:** SAS types (service/account/user-delegation), shared access policies, Azure AD auth
- **Practice Questions:** What is the most secure SAS type? What does "disable anonymous access" prevent?
- ⏱ **Time:** 2 hours

### Day 21 — Key Vault + Azure SQL Security
- **Read:** [21-Key-Vault.md](../Domain3-Compute-Storage/21-Key-Vault.md)
- **Lab:** Enable soft delete + purge protection on Key Vault; review TDE and Always Encrypted
- **Focus:** Key Vault access policies vs RBAC, TDE vs Always Encrypted vs DDM, HSM tiers
- **Practice Questions:** What is the difference between TDE and Always Encrypted? Who does DDM hide data from?
- ⏱ **Time:** 3 hours

---

## 📆 Week 4 (Days 22–28): Domain 4 — Defender for Cloud and Microsoft Sentinel

**Theme:** Detect, respond, and automate — SOC operations in Azure.

### Day 22 — Defender for Cloud Fundamentals
- **Read:** [22-Defender-for-Cloud.md](../Domain4-Defender-Sentinel/22-Defender-for-Cloud.md)
- **Lab:** Navigate Defender for Cloud dashboard; review Secure Score recommendations
- **Focus:** CSPM vs CWPP, free tier vs Defender plans, Secure Score calculation
- **Practice Questions:** What is the difference between CSPM and CWPP? What lowers Secure Score?
- ⏱ **Time:** 2 hours

### Day 23 — Defender Plans
- **Read:** [23-Defender-Plans.md](../Domain4-Defender-Sentinel/23-Defender-Plans.md)
- **Lab:** Enable Defender for Servers Plan 2; review threat detection alerts
- **Focus:** Per-resource pricing, Defender for Servers vs Storage vs SQL vs Containers vs DNS
- **Practice Questions:** What does Defender for Servers add over basic Defender for Cloud?
- ⏱ **Time:** 2 hours

### Day 24 — Azure Policy + Initiatives
- **Read:** [24-Azure-Policy.md](../Domain4-Defender-Sentinel/24-Azure-Policy.md)
- **Lab:** Assign "Audit VMs that do not use managed disks" policy; create a custom initiative
- **Focus:** Effect types (Audit/Deny/DeployIfNotExists/Modify), compliance view, remediation tasks
- **Practice Questions:** What is the difference between Audit and Deny effect? What does DeployIfNotExists do?
- ⏱ **Time:** 2 hours

### Day 25 — Sentinel Setup + Data Connectors
- **Read:** [25-Sentinel-Setup.md](../Domain4-Defender-Sentinel/25-Sentinel-Setup.md)
- **Lab:** Enable Microsoft Sentinel; connect Microsoft Entra ID and Azure Activity Log
- **Focus:** Log Analytics workspace, data connectors (native/CEF/Syslog/API), costs
- **Practice Questions:** What is a CEF connector? What is the difference between a data connector and an analytics rule?
- ⏱ **Time:** 3 hours

### Day 26 — KQL Fundamentals
- **Read:** [26-KQL-Fundamentals.md](../Domain4-Defender-Sentinel/26-KQL-Fundamentals.md)
- **Lab:** Write 5 KQL queries in Log Analytics — filter, summarize, join, render
- **Focus:** `where`, `project`, `summarize`, `join`, `extend`, `render`, time functions
- **Practice Questions:** Write a KQL query for failed sign-ins. How do you count events by hour?
- ⏱ **Time:** 3 hours

### Day 27 — Sentinel Analytics Rules + Incidents
- **Read:** [27-Sentinel-Analytics.md](../Domain4-Defender-Sentinel/27-Sentinel-Analytics.md)
- **Lab:** Create a scheduled analytics rule; simulate a brute-force alert
- **Focus:** Rule types (Scheduled/NRT/Fusion/ML/TI), alert grouping, entity mapping
- **Practice Questions:** What is a Fusion rule? What is NRT? How does alert grouping work?
- ⏱ **Time:** 2 hours

### Day 28 — Playbooks + Automation Rules
- **Read:** [28-Sentinel-Automation.md](../Domain4-Defender-Sentinel/28-Sentinel-Automation.md)
- **Lab:** Create an automation rule to auto-assign incidents; trigger a Logic App playbook
- **Focus:** Automation rules vs playbooks, Logic App connectors, SOAR capabilities
- **Practice Questions:** What is the difference between an automation rule and a playbook? When does an automation rule run?
- ⏱ **Time:** 2 hours

---

## 🏁 Final Stretch (Days 29–30): Full Review and Exam Sprint

### Day 29 — Comprehensive Review
- **Read:** [Trigger-Words-and-Traps.md](./Trigger-Words-and-Traps.md) — full pass
- **Read:** [Quick-Reference-Cheatsheet.md](./Quick-Reference-Cheatsheet.md) — full pass
- **Review Diagrams:** All 4 diagram files in the Diagrams folder
- **Activity:** Identify your top 5 weak topics; re-read those sections
- **Practice:** Take a timed 30-question practice quiz covering all 4 domains
- ⏱ **Time:** 3 hours

### Day 30 — Final Exam Sprint
- **Activity:** Take a full 65-question AZ-500 practice exam (timed, simulated conditions)
- **Review:** Go through every wrong answer — understand **why** it was wrong
- **Last-minute checklist:**
  - [ ] Know the PIM vs JIT distinction cold
  - [ ] Know Private Endpoint vs Service Endpoint cold
  - [ ] Know NSG vs Azure Firewall use cases
  - [ ] Know Sentinel Automation Rule vs Playbook
  - [ ] Know all encryption types (ADE / SSE / TDE / Always Encrypted)
  - [ ] Review trigger word list one final time
- ⏱ **Time:** 3 hours

---

## 💡 General Study Tips

### 1. Use Microsoft Learn Sandbox (Free!)
- Go to [Microsoft Learn](https://learn.microsoft.com) and use the free sandbox environment
- No Azure subscription required for most hands-on exercises
- Activate sandbox before each study session; sessions last 4 hours

### 2. Scenario-Based Thinking
- AZ-500 questions describe a **business problem**, not a technical spec
- Always ask: *"What security requirement is this scenario describing?"*
- Keywords like "least privilege" → RBAC + PIM; "zero trust" → Conditional Access + identity

### 3. Learn to Distinguish Similar Services
- When you see two services that seem similar, build a comparison table
- Key pairs to master: NSG vs Firewall, Private vs Service Endpoint, PIM vs JIT, B2B vs B2C

### 4. Use the Trigger Words List
- Before the exam, review [Trigger-Words-and-Traps.md](./Trigger-Words-and-Traps.md) daily for the last week
- Train yourself to spot trigger phrases in questions instantly
- Many questions can be eliminated immediately with the right trigger word

### 5. Understand "Why", Not Just "What"
- Don't memorize answers — understand the reasoning
- For every service, know: *What problem does it solve? What does it NOT do?*
- This lets you answer novel scenario questions you've never seen before

### 6. Time Management on Exam Day
- 65 questions / 120 minutes = ~1.8 minutes per question
- Mark difficult questions and return; don't get stuck
- Eliminate obviously wrong answers first to improve odds
- Read the **last sentence** of each question — it usually contains the actual ask

### 7. Practice KQL Weekly
- KQL appears on the exam — know basic syntax
- Practice on real data at [aka.ms/lademo](https://aka.ms/lademo) (free demo workspace)

---

## 📋 Recommended Practice Test Resources

| Resource | Type | Cost |
|----------|------|------|
| Microsoft Learn (official) | Free modules + knowledge checks | Free |
| MeasureUp Official Practice Tests | Exam-style questions | Paid |
| Whizlabs AZ-500 | Large question bank | Paid |
| TutorialsDojo AZ-500 | Scenario-based practice | Paid |
| GitHub AZ-500 dumps (community) | Community-sourced | Free (use carefully) |

---

📚 **Further Reading:** https://learn.microsoft.com/en-us/credentials/certifications/azure-security-engineer/  
🔄 **Last Verified:** 2026 (AZ-500 January 2026 objectives)
