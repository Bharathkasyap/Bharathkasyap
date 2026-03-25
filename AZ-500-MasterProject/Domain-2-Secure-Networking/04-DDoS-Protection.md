# Azure DDoS Protection

> 📌 AZ-500 Exam Objective: Plan and implement advanced security for infrastructure
> 🏷️ Domain: 2 — Secure Networking | Weight: 20–25%

***

## What Is This?

A Distributed Denial of Service (DDoS) attack floods your service with so much fake traffic that real users cannot get through. Azure DDoS Protection automatically detects and absorbs this flood before it reaches your resources. It runs silently in the background and activates when it detects an attack — you do not need to do anything during an attack.

***

## Why Does This Exist?

Any public IP address on the internet can be targeted by DDoS attacks. A large enough flood can take down a website, API, or game server even if the application is perfectly coded. Without DDoS Protection, Azure automatically provides a basic level of defense, but it has no SLA and no analytics. The paid tier gives you dedicated mitigation capacity, attack reports, and financial protection against unexpected scale-out costs during an attack.

***

## Who Actually Needs This? (Real-World Context)

An online gaming company runs game servers on Azure with public IPs. Competitors pay for DDoS-for-hire services to knock their game offline. Without DDoS Network Protection, the company faces hours of downtime and unexpected Azure bandwidth bills from the attack traffic. After enabling DDoS Network Protection on their production VNet, attacks are mitigated in under one minute, the gaming service stays online, and Azure provides cost protection so the company is not billed for the attack-generated traffic.

***

## How It Works (Pure Concept, No Portal)

- **Three DDoS attack types:**
  - **Volumetric attacks** — flood the network with massive amounts of traffic to fill the pipe (e.g., UDP floods, ICMP floods). Measured in Gbps.
  - **Protocol attacks** — exploit weaknesses in Layer 3/4 protocols to exhaust resources on network devices (e.g., SYN floods). Measured in Mpps (millions of packets per second).
  - **Resource layer attacks** — target the application layer to exhaust server resources (e.g., HTTP floods). These are lower volume but harder to detect.
- **Azure DDoS Basic (Infrastructure Protection):**
  - Free, always on, applies to all Azure public IPs automatically.
  - Protects the shared Azure infrastructure.
  - No per-customer SLA, no attack analytics, no mitigation reports, no cost protection.
- **Azure DDoS Network Protection:**
  - Paid plan ($2,944/month covers up to 100 public IPs in the protected VNets).
  - Enabled per VNet. All public IPs in that VNet get protection.
  - **Adaptive tuning** — learns your normal traffic baseline and only triggers for true anomalies. Reduces false positives.
  - **Attack analytics** — real-time dashboards showing attack vectors, traffic volume, and mitigation status.
  - **Mitigation reports** — post-attack reports for compliance and review.
  - **DDoS rapid response** — access to the Microsoft DDoS expert team during active attacks.
  - **Cost protection** — Azure credits you for auto-scaled compute and bandwidth used due to the attack.
  - **SLA guarantee** — 99.99% mitigation SLA for protected resources.
- **Azure DDoS IP Protection:**
  - Pay per individual public IP ($199/month per IP).
  - Same technical protection as Network Protection.
  - Does NOT include DDoS Rapid Response or cost protection.
  - Best for small deployments protecting only specific IPs.
- **Mitigation flow:** Azure monitors all inbound traffic to your public IPs at the network edge. When traffic patterns match attack signatures (volume spike, protocol anomaly), Azure routes your traffic through scrubbing centers. Bad traffic is dropped. Clean traffic is forwarded to your resources. This happens in under 60 seconds for volumetric attacks.
- **Always-on traffic monitoring** runs 24/7 without any configuration. You do not turn it on or off during an attack.

***

## 2026 Portal Walkthrough — Step by Step

**Create a DDoS Protection Plan and enable it on a VNet:**

1. Go to **portal.azure.com** → search **"DDoS protection plans"** → click **Create**.
2. Fill in: Subscription, Resource Group, Name (e.g., `ddos-plan-prod`), Region → click **Review + create** → **Create**.
3. Go to your **Virtual Network** (e.g., `vnet-prod`) → left menu → **DDoS protection**.
4. Toggle **DDoS Network Protection** to **Enable**.
5. **DDoS protection plan**: Select `ddos-plan-prod` → click **Save**.

**View attack analytics:**
1. Open the DDoS Protection Plan → left menu → **Metrics**.
2. Add metric: `Under DDoS attack or not` — shows 0 (no attack) or 1 (under attack).
3. Add metric: `Inbound packets dropped DDoS` — shows packets scrubbed during mitigation.
4. Set up an **Alert**: Go to **Alerts** → **+ New alert rule** → condition: `Under DDoS attack or not` equals 1 → notify via email or Action Group.

**Enable diagnostic logs for mitigation reports:**
1. Open the Public IP Address resource → **Diagnostic settings** → **+ Add diagnostic setting**.
2. Check **DDoSProtectionNotifications**, **DDoSMitigationFlowLogs**, **DDoSMitigationReports**.
3. Send to a **Log Analytics workspace** or **Storage account** → **Save**.

**CLI:**
```bash
# Create DDoS Protection Plan
az network ddos-protection create \
  --resource-group rg-prod \
  --name ddos-plan-prod \
  --location eastus

# Enable DDoS Network Protection on a VNet
az network vnet update \
  --resource-group rg-prod \
  --name vnet-prod \
  --ddos-protection true \
  --ddos-protection-plan ddos-plan-prod
```

**PowerShell:**
```powershell
# Create DDoS Protection Plan
$ddosPlan = New-AzDdosProtectionPlan `
  -ResourceGroupName "rg-prod" `
  -Name "ddos-plan-prod" `
  -Location "eastus"

# Get VNet and attach DDoS plan
$vnet = Get-AzVirtualNetwork -Name "vnet-prod" -ResourceGroupName "rg-prod"
$vnet.DdosProtectionPlan = New-Object Microsoft.Azure.Commands.Network.Models.PSResourceId
$vnet.DdosProtectionPlan.Id = $ddosPlan.Id
$vnet.EnableDdosProtection = $true
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/ddos-protection/

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| DDoS Protection Plan | A billable resource that holds the DDoS policy. One plan can protect multiple VNets. | Create one per subscription or business unit. Associate it to each VNet you want to protect. |
| Enabled on VNet | Activates protection for all public IPs in that VNet | Enable on every production VNet that has internet-facing resources |
| Adaptive Tuning | Learns your traffic baseline to reduce false positives | Always on for Network Protection — no configuration needed |
| Attack Analytics | Real-time dashboard and metrics during an active attack | Use to monitor and report during incident response |
| Mitigation Reports | Automated reports generated after an attack subsides | Use for compliance, insurance claims, and post-incident review |
| DDoS Rapid Response | Direct access to Microsoft's DDoS expert team via support ticket | Use during an active attack that is not mitigating as expected |
| Cost Protection | Azure credits you for excess compute/bandwidth caused by an attack | Only available with DDoS Network Protection, not IP Protection |

***

## Comparison Table

| Feature | DDoS Basic (Free) | DDoS Network Protection | DDoS IP Protection |
|---|---|---|---|
| Cost | Free | ~$2,944/month (100 IPs) | ~$199/month per IP |
| SLA | No | Yes (99.99%) | Yes |
| Attack analytics | No | Yes | Yes |
| Adaptive tuning | Basic | Advanced | Advanced |
| Mitigation reports | No | Yes | Yes |
| Cost protection | No | Yes | No |
| DDoS Rapid Response | No | Yes | No |
| Scope | All Azure shared infra | Per VNet | Per public IP |

***

## What Happens If You Skip This?

With only DDoS Basic, a large volumetric attack can degrade or completely take down your internet-facing services. You will have no visibility into the attack traffic, no analytics, and no SLA that your service will survive. Worse, if Azure auto-scales your compute or bandwidth in response to attack traffic, you pay the bill — Basic does not include cost protection. A single large attack can generate thousands of dollars in unexpected Azure charges.

***

## AZ-500 Exam Section

- **Trigger Words:** "volumetric attack", "DDoS", "traffic scrubbing", "attack analytics", "SLA protection", "DDoS plan", "cost protection", "mitigation report"
- **Common Traps:**
  - DDoS Basic ≠ DDoS Network Protection. Basic is free and automatic but has NO SLA, NO analytics, NO cost protection. Many exam questions exploit this confusion.
  - DDoS Network Protection is enabled on the **VNet**, not on individual VMs or NICs.
  - One DDoS Protection Plan can cover multiple VNets across subscriptions.
  - DDoS does NOT protect against application layer (L7) attacks like HTTP floods — that requires WAF.
  - DDoS IP Protection exists but does NOT include cost protection or rapid response.
- **How to Tell Apart:**
  - "Free DDoS protection" → DDoS Basic (infrastructure level, no SLA).
  - "SLA-backed DDoS with analytics" → DDoS Network Protection.
  - "Protect specific IPs cheaply" → DDoS IP Protection.
  - "Protect against HTTP flood / Layer 7 attack" → WAF, not DDoS Protection.
- **Likely Question Formats:**
  - "You need DDoS protection with an SLA guarantee and attack analytics. What do you use?" → DDoS Network Protection.
  - "Your company wants to avoid unexpected bills from DDoS attack traffic causing auto-scale." → DDoS Network Protection (cost protection).
  - "Is DDoS Basic enough for a production workload?" → No, it has no SLA or analytics.
- **Memorization Tip:** DDoS Basic is like having a fence around the neighborhood (everyone benefits, no SLA). DDoS Network Protection is your own personal security guard with a contract, cameras, and a lawyer (SLA, analytics, cost protection).

***

## Pocket Summary (5-Year Refresh)

- DDoS attacks flood resources with fake traffic. Azure DDoS Protection absorbs the flood.
- DDoS Basic is free, always on, but no SLA, no analytics, no cost protection.
- DDoS Network Protection is paid, per-VNet, SLA-backed with analytics and cost protection.
- DDoS IP Protection is per-IP, cheaper, but no cost protection or rapid response.
- DDoS does not protect against L7 app attacks — use WAF for those.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/ddos-protection/ddos-protection-overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
