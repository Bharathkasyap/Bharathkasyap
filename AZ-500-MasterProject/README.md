# AZ-500 Master Project: Microsoft Azure Security Technologies

> A complete, self-contained study guide for the AZ-500 certification exam.
> Read each domain in order. Follow the 30-day study plan. Pass the exam.

---

## 📋 Table of Contents

- [About This Project](#about-this-project)
- [Exam Domain Weightage](#exam-domain-weightage)
- [How to Use This Guide](#how-to-use-this-guide)
- [30-Day Study Plan](#30-day-study-plan)
- [Domain 1 – Secure Identity and Access](#domain-1--secure-identity-and-access)
- [Domain 2 – Secure Networking](#domain-2--secure-networking)
- [Domain 3 – Secure Compute, Storage, and Databases](#domain-3--secure-compute-storage-and-databases)
- [Domain 4 – Secure Azure using Microsoft Defender for Cloud and Sentinel](#domain-4--secure-azure-using-microsoft-defender-for-cloud-and-sentinel)
- [Exam Tips and Quick References](#exam-tips-and-quick-references)
- [Diagrams](#diagrams)
- [Exam Info](#exam-info)

---

## About This Project

This project covers every topic on the AZ-500 exam in one place. Each file maps to a specific exam skill. You do not need any other resource to start studying — everything is here.

**Who this is for:**
- Cloud engineers preparing for AZ-500
- Security professionals moving into Azure
- Anyone who has already passed AZ-104 and wants to specialise in security

---

## Exam Domain Weightage

| Domain | Topic | Weight |
|--------|-------|--------|
| 1 | Secure Identity and Access | 15–20% |
| 2 | Secure Networking | 20–25% |
| 3 | Secure Compute, Storage, and Databases | 20–25% |
| 4 | Secure Azure using Microsoft Defender for Cloud and Sentinel | 30–35% |

> Domain 4 carries the highest weight. Give it the most study time.

---

## How to Use This Guide

1. Read this README fully first.
2. Work through each domain **in order** (1 → 4).
3. Use the [30-Day Study Plan](Exam-Tips/30-Day-Study-Plan.md) to keep on schedule.
4. Review the [Quick Reference Cheatsheet](Exam-Tips/Quick-Reference-Cheatsheet.md) before exam day.
5. Study the [Trigger Words and Traps](Exam-Tips/Trigger-Words-and-Traps.md) to avoid common mistakes.
6. Check the [Diagrams](#diagrams) section to understand traffic flows and architectures visually.

---

## 30-Day Study Plan

Follow a structured daily schedule to cover all four domains and review before exam day.

📅 [View the 30-Day Study Plan →](Exam-Tips/30-Day-Study-Plan.md)

---

## Domain 1 – Secure Identity and Access

**Estimated reading time: ~3 hours**
**Exam weight: 15–20%**

This domain covers Microsoft Entra ID, authentication, authorisation, and identity governance.

| # | File | Topic |
|---|------|-------|
| 1 | [01-EntraID-Overview.md](Domain-1-Identity-and-Access/01-EntraID-Overview.md) | Microsoft Entra ID fundamentals |
| 2 | [02-MFA-and-Conditional-Access.md](Domain-1-Identity-and-Access/02-MFA-and-Conditional-Access.md) | Multi-Factor Authentication and Conditional Access policies |
| 3 | [03-RBAC-and-Custom-Roles.md](Domain-1-Identity-and-Access/03-RBAC-and-Custom-Roles.md) | Role-Based Access Control and custom role definitions |
| 4 | [04-PIM-Privileged-Identity-Management.md](Domain-1-Identity-and-Access/04-PIM-Privileged-Identity-Management.md) | Privileged Identity Management (PIM) |
| 5 | [05-Identity-Governance-Access-Reviews.md](Domain-1-Identity-and-Access/05-Identity-Governance-Access-Reviews.md) | Identity Governance and Access Reviews |
| 6 | [06-Managed-Identities.md](Domain-1-Identity-and-Access/06-Managed-Identities.md) | System-assigned and user-assigned Managed Identities |
| 7 | [07-App-Registrations-and-Service-Principals.md](Domain-1-Identity-and-Access/07-App-Registrations-and-Service-Principals.md) | App Registrations and Service Principals |
| 8 | [08-External-Identities-B2B-B2C.md](Domain-1-Identity-and-Access/08-External-Identities-B2B-B2C.md) | External Identities — Azure AD B2B and B2C |

---

## Domain 2 – Secure Networking

**Estimated reading time: ~4 hours**
**Exam weight: 20–25%**

This domain covers virtual networks, firewalls, DDoS protection, private connectivity, and network monitoring.

| # | File | Topic |
|---|------|-------|
| 1 | [01-VNet-NSG-ASG.md](Domain-2-Secure-Networking/01-VNet-NSG-ASG.md) | Virtual Networks, NSGs, and Application Security Groups |
| 2 | [02-Azure-Firewall-and-Policy-Manager.md](Domain-2-Secure-Networking/02-Azure-Firewall-and-Policy-Manager.md) | Azure Firewall and Firewall Policy Manager |
| 3 | [03-WAF-AppGateway-FrontDoor.md](Domain-2-Secure-Networking/03-WAF-AppGateway-FrontDoor.md) | Web Application Firewall, App Gateway, and Front Door |
| 4 | [04-DDoS-Protection.md](Domain-2-Secure-Networking/04-DDoS-Protection.md) | Azure DDoS Protection plans |
| 5 | [05-Bastion-and-JIT-VM-Access.md](Domain-2-Secure-Networking/05-Bastion-and-JIT-VM-Access.md) | Azure Bastion and Just-In-Time VM Access |
| 6 | [06-Private-Endpoints-vs-Service-Endpoints.md](Domain-2-Secure-Networking/06-Private-Endpoints-vs-Service-Endpoints.md) | Private Endpoints vs Service Endpoints |
| 7 | [07-VPN-Gateway-and-ExpressRoute.md](Domain-2-Secure-Networking/07-VPN-Gateway-and-ExpressRoute.md) | VPN Gateway and ExpressRoute |
| 8 | [08-Azure-Virtual-WAN.md](Domain-2-Secure-Networking/08-Azure-Virtual-WAN.md) | Azure Virtual WAN |
| 9 | [09-Network-Watcher-and-Flow-Logs.md](Domain-2-Secure-Networking/09-Network-Watcher-and-Flow-Logs.md) | Network Watcher and NSG Flow Logs |

---

## Domain 3 – Secure Compute, Storage, and Databases

**Estimated reading time: ~4 hours**
**Exam weight: 20–25%**

This domain covers securing VMs, containers, App Service, storage accounts, Key Vault, databases, and APIs.

| # | File | Topic |
|---|------|-------|
| 1 | [01-VM-Security-and-Disk-Encryption.md](Domain-3-Compute-Storage-Databases/01-VM-Security-and-Disk-Encryption.md) | VM security and Azure Disk Encryption |
| 2 | [02-AKS-Security.md](Domain-3-Compute-Storage-Databases/02-AKS-Security.md) | Azure Kubernetes Service (AKS) security |
| 3 | [03-Container-Registry-and-Defender.md](Domain-3-Compute-Storage-Databases/03-Container-Registry-and-Defender.md) | Container Registry and Defender for Containers |
| 4 | [04-App-Service-Security.md](Domain-3-Compute-Storage-Databases/04-App-Service-Security.md) | App Service security and authentication |
| 5 | [05-Storage-Account-Security.md](Domain-3-Compute-Storage-Databases/05-Storage-Account-Security.md) | Storage Account security controls |
| 6 | [06-Blob-SAS-Access-Tiers.md](Domain-3-Compute-Storage-Databases/06-Blob-SAS-Access-Tiers.md) | Blob storage, SAS tokens, and access tiers |
| 7 | [07-Azure-Key-Vault.md](Domain-3-Compute-Storage-Databases/07-Azure-Key-Vault.md) | Azure Key Vault — secrets, keys, and certificates |
| 8 | [08-SQL-Database-and-Managed-Instance-Security.md](Domain-3-Compute-Storage-Databases/08-SQL-Database-and-Managed-Instance-Security.md) | Azure SQL Database and Managed Instance security |
| 9 | [09-API-Management-Security.md](Domain-3-Compute-Storage-Databases/09-API-Management-Security.md) | API Management security |

---

## Domain 4 – Secure Azure using Microsoft Defender for Cloud and Sentinel

**Estimated reading time: ~5 hours**
**Exam weight: 30–35%**

This domain is the heaviest on the exam. It covers Defender for Cloud, Azure Policy, Microsoft Sentinel, KQL, analytics rules, playbooks, and threat dashboards.

| # | File | Topic |
|---|------|-------|
| 1 | [01-Defender-for-Cloud-Overview.md](Domain-4-Defender-and-Sentinel/01-Defender-for-Cloud-Overview.md) | Microsoft Defender for Cloud overview |
| 2 | [02-Secure-Score-and-Recommendations.md](Domain-4-Defender-and-Sentinel/02-Secure-Score-and-Recommendations.md) | Secure Score and security recommendations |
| 3 | [03-Defender-Workload-Protection-Plans.md](Domain-4-Defender-and-Sentinel/03-Defender-Workload-Protection-Plans.md) | Defender workload protection plans |
| 4 | [04-Azure-Policy-and-Initiatives.md](Domain-4-Defender-and-Sentinel/04-Azure-Policy-and-Initiatives.md) | Azure Policy and Policy Initiatives |
| 5 | [05-Microsoft-Sentinel-Overview.md](Domain-4-Defender-and-Sentinel/05-Microsoft-Sentinel-Overview.md) | Microsoft Sentinel overview |
| 6 | [06-Sentinel-Data-Connectors.md](Domain-4-Defender-and-Sentinel/06-Sentinel-Data-Connectors.md) | Sentinel data connectors |
| 7 | [07-KQL-Queries-for-Security-Operations.md](Domain-4-Defender-and-Sentinel/07-KQL-Queries-for-Security-Operations.md) | KQL queries for security operations |
| 8 | [08-Analytics-Rules-and-Incidents.md](Domain-4-Defender-and-Sentinel/08-Analytics-Rules-and-Incidents.md) | Analytics rules and incident management |
| 9 | [09-Playbooks-Logic-Apps-Automation.md](Domain-4-Defender-and-Sentinel/09-Playbooks-Logic-Apps-Automation.md) | Playbooks, Logic Apps, and automation |
| 10 | [10-Workbooks-and-Threat-Dashboards.md](Domain-4-Defender-and-Sentinel/10-Workbooks-and-Threat-Dashboards.md) | Workbooks and threat dashboards |

---

## Exam Tips and Quick References

| File | Purpose |
|------|---------|
| [Quick-Reference-Cheatsheet.md](Exam-Tips/Quick-Reference-Cheatsheet.md) | One-page summary of key services and when to use them |
| [Trigger-Words-and-Traps.md](Exam-Tips/Trigger-Words-and-Traps.md) | Common exam question traps and how to spot them |
| [30-Day-Study-Plan.md](Exam-Tips/30-Day-Study-Plan.md) | Day-by-day study schedule across all four domains |

---

## Diagrams

Visual references for core security flows and architectures.

| File | Diagram |
|------|---------|
| [identity-access-flow.md](Diagrams/identity-access-flow.md) | Identity and access request flow |
| [network-traffic-path.md](Diagrams/network-traffic-path.md) | Network traffic path through Azure security layers |
| [defender-alert-lifecycle.md](Diagrams/defender-alert-lifecycle.md) | Defender for Cloud alert lifecycle |
| [sentinel-incident-workflow.md](Diagrams/sentinel-incident-workflow.md) | Sentinel incident detection and response workflow |

---

## Exam Info

| Field | Detail |
|-------|--------|
| Exam Code | AZ-500 |
| Exam Name | Microsoft Azure Security Technologies |
| Certification | Microsoft Certified: Azure Security Engineer Associate |
| Passing Score | 700 / 1000 |
| Duration | 150 minutes |
| Cost | ~$165 USD (varies by country) |

📖 [Full exam details, registration link, and renewal instructions → EXAM-INFO.md](EXAM-INFO.md)

🔗 Register at: https://learn.microsoft.com/en-us/credentials/certifications/azure-security-engineer/

---

> **Tip:** Start with Domain 4. It carries the most weight. Then work backwards through Domains 3, 2, and 1.

