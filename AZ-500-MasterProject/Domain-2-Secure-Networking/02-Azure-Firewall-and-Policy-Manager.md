# Azure Firewall and Firewall Policy Manager

> 📌 AZ-500 Exam Objective: Plan and implement security for virtual networks
> 🏷️ Domain: 2 — Secure Networking | Weight: 20–25%

***

## What Is This?

Azure Firewall is a cloud-managed firewall that sits in your network and inspects all traffic going in, out, and across your Azure environment. Unlike NSGs (which only filter by IP and port), Azure Firewall understands full website names (FQDNs) and can detect attacks. Firewall Policy Manager lets you manage rules for many firewalls from one central place.

***

## Why Does This Exist?

NSGs can block a port, but they cannot block `bad-site.com` while allowing `good-site.com` — they only see IP addresses. Azure Firewall solves this. It also detects network-level attacks with its built-in Intrusion Detection and Prevention System (IDPS). When you have many Azure regions, Firewall Policy Manager lets one team push the same rules everywhere without logging into each firewall.

***

## Who Actually Needs This? (Real-World Context)

A financial services company runs workloads in three Azure regions. They need every outbound internet request to go through a central security checkpoint. They also need to block malware domains and inspect HTTPS traffic for data exfiltration. They deploy Azure Firewall Premium in a hub VNet in each region and use Firewall Policy Manager to push one master policy to all three firewalls. Any change to the blocked-domain list takes effect in all regions within minutes.

***

## How It Works (Pure Concept, No Portal)

- **Azure Firewall** is a stateful firewall. It tracks connection state, so return traffic is automatically allowed once a connection is established.
- It operates at **Layers 3 through 7**. It can filter by IP, port, protocol, FQDN, URL, and application type.
- **Three rule types:**
  - **Application Rules** — filter outbound HTTP/HTTPS by FQDN or URL (e.g., allow `*.microsoft.com`, block everything else).
  - **Network Rules** — filter any TCP/UDP/ICMP traffic by IP and port (like an advanced NSG).
  - **DNAT Rules** — translate inbound traffic to a private IP (like port forwarding for inbound connections).
- **Rule processing order:** DNAT rules → Network rules → Application rules. First matching rule wins.
- **Azure Firewall Standard** handles FQDN filtering, network rules, DNAT, threat intelligence.
- **Azure Firewall Premium** adds IDPS (signature-based attack detection), TLS inspection (decrypt HTTPS, inspect, re-encrypt), URL filtering (path-level, not just domain), and web categories.
- **TLS Inspection** requires a CA certificate. Azure Firewall decrypts the traffic, inspects it, then re-encrypts it before sending it to the destination.
- **IDPS** uses Microsoft's threat intelligence signature database to identify and block known attack patterns in real time.
- **Firewall Policy** is a resource that holds all your rules. You link a policy to one or more firewalls. Child policies can inherit from a parent policy (great for org-wide baseline rules).
- **Azure Firewall Manager** is the central hub for managing multiple firewalls and their policies, especially in hub-and-spoke topologies.
- In a **hub-and-spoke architecture**, the firewall lives in the hub VNet. All spoke VNets route their traffic through the hub firewall using User Defined Routes (UDRs).

***

## 2026 Portal Walkthrough — Step by Step

**Deploy Azure Firewall and create an application rule to allow *.microsoft.com:**

1. Go to **portal.azure.com** → search **"Firewalls"** → click **Create**.
2. Select Subscription, Resource Group, Name (e.g., `fw-hub-prod`), Region.
3. **Firewall SKU**: Select **Premium** for IDPS/TLS inspection, or **Standard** for basic FQDN filtering.
4. **Firewall management**: Select **Use Firewall Policy** → click **Add new** → name it `fwpol-hub-prod`.
5. **Virtual network**: Select existing hub VNet or create one. Azure Firewall needs its own subnet named exactly `AzureFirewallSubnet` with at least a `/26` prefix.
6. **Public IP address**: Create new or select existing → click **Review + create** → **Create** (deployment takes ~5–10 minutes).
7. Open the **Firewall Policy** (`fwpol-hub-prod`) → left menu → **Application rules** → **+ Add a rule collection**.
8. Set:
   - **Name**: `rc-allow-microsoft`
   - **Rule collection type**: `Application`
   - **Priority**: `100`
   - **Rule collection action**: `Allow`
   - Under Rules, add:
     - **Name**: `allow-microsoft-com`
     - **Source type**: `IP Address`, **Source**: `10.0.0.0/8`
     - **Protocol**: `Https:443`
     - **Destination type**: `FQDN`, **Destination**: `*.microsoft.com`
   → click **Add**.
9. Add a second rule collection at priority `200` with Action `Deny` and destination `*` to block everything else.
10. Associate policy with the firewall: Go to **Firewalls** → open `fw-hub-prod` → **Firewall policy** → confirm policy is linked.

**Enable IDPS (Premium only):**
1. Open Firewall Policy → **IDPS** → set mode to **Alert and Deny** → **Save**.

**CLI:**
```bash
# Create Firewall Policy
az network firewall policy create \
  --resource-group rg-hub \
  --name fwpol-hub-prod \
  --location eastus \
  --sku Premium

# Create Azure Firewall linked to policy
az network firewall create \
  --resource-group rg-hub \
  --name fw-hub-prod \
  --location eastus \
  --firewall-policy fwpol-hub-prod \
  --vnet-name vnet-hub \
  --public-ip fwpip-prod

# Create application rule collection to allow *.microsoft.com
az network firewall policy rule-collection-group collection add-filter-collection \
  --resource-group rg-hub \
  --policy-name fwpol-hub-prod \
  --rule-collection-group-name DefaultApplicationRuleCollectionGroup \
  --name rc-allow-microsoft \
  --collection-priority 100 \
  --action Allow \
  --rule-type ApplicationRule \
  --rule-name allow-microsoft-com \
  --protocols Https=443 \
  --source-addresses "10.0.0.0/8" \
  --target-fqdns "*.microsoft.com"
```

**PowerShell:**
```powershell
# Create application rule
$appRule = New-AzFirewallApplicationRule `
  -Name "allow-microsoft-com" `
  -SourceAddress "10.0.0.0/8" `
  -Protocol "https:443" `
  -TargetFqdn "*.microsoft.com"

# Create application rule collection
$appRuleCollection = New-AzFirewallApplicationRuleCollection `
  -Name "rc-allow-microsoft" `
  -Priority 100 `
  -ActionType "Allow" `
  -Rule $appRule

# Get VNet and public IP
$vnet = Get-AzVirtualNetwork -Name "vnet-hub" -ResourceGroupName "rg-hub"
$pip = Get-AzPublicIpAddress -Name "fwpip-prod" -ResourceGroupName "rg-hub"

# Create the firewall
New-AzFirewall `
  -Name "fw-hub-prod" `
  -ResourceGroupName "rg-hub" `
  -Location "eastus" `
  -VirtualNetwork $vnet `
  -PublicIpAddress $pip `
  -ApplicationRuleCollection $appRuleCollection
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/firewall/

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Firewall SKU (Standard vs Premium) | Standard = FQDN + threat intel. Premium adds IDPS, TLS inspection, URL categories | Use Premium when you need deep packet inspection or HTTPS decryption |
| Application Rules | Filter outbound traffic by FQDN or URL | Use to allow/deny specific websites or Azure service endpoints |
| Network Rules | Filter traffic by IP, port, protocol | Use for non-HTTP traffic like SQL, SMTP, custom TCP apps |
| DNAT Rules | Translate a public IP + port to a private IP + port | Use to expose internal resources without a public IP on the VM |
| IDPS Mode (Off / Alert / Alert and Deny) | Detects or blocks known attack signatures | Set to Alert and Deny in production for active protection |
| TLS Inspection | Decrypts HTTPS to inspect contents, then re-encrypts | Use when you need to detect malware or DLP in encrypted traffic |
| Threat Intelligence | Blocks known malicious IPs and domains from Microsoft's feed | Always enable — it's free and blocks known bad actors automatically |
| Firewall Policy (Parent/Child) | Child policy inherits base rules from parent | Use for multi-team environments where a central team sets baselines |

***

## Comparison Table

| Feature | NSG | Azure Firewall Standard | Azure Firewall Premium |
|---|---|---|---|
| Layer | L3/L4 only | L3–L7 | L3–L7 |
| FQDN filtering | No | Yes | Yes |
| IDPS | No | No | Yes |
| TLS Inspection | No | No | Yes |
| Central management | No (per resource) | Via Firewall Manager | Via Firewall Manager |
| Stateful | Yes | Yes | Yes |
| Cost | Free | ~$1.25/hour | ~$3.00/hour |

***

## What Happens If You Skip This?

Without Azure Firewall, you have no way to block malicious websites by name. Your VMs can freely reach any IP on the internet. You cannot detect or block exploit traffic like port scans or known malware callbacks. In a hub-and-spoke network, every spoke must manage its own security independently, which leads to gaps and inconsistency.

***

## AZ-500 Exam Section

- **Trigger Words:** "FQDN filtering", "application rule", "IDPS", "TLS inspection", "centralized firewall", "hub-and-spoke firewall", "threat intelligence", "URL filtering"
- **Common Traps:**
  - NSG is L3/L4 only — it cannot block `bad-site.com`. Azure Firewall does L7. Do not confuse them.
  - Azure Firewall Premium must be selected at creation for IDPS and TLS inspection — you cannot upgrade Standard to Premium in place.
  - TLS inspection requires a CA certificate stored in Key Vault. It does NOT work without this.
  - `AzureFirewallSubnet` must be named exactly that and must be at least `/26`.
  - Rule processing order is fixed: DNAT → Network → Application. You cannot change it.
- **How to Tell Apart:**
  - NSG vs Azure Firewall: "block port 443" → NSG. "block evil.com" → Azure Firewall.
  - Standard vs Premium: "detect intrusions" or "inspect HTTPS" → Premium.
  - Firewall Policy vs classic rules: Policy is the new recommended approach. Classic rules are legacy.
- **Likely Question Formats:**
  - "You need to allow outbound traffic to *.microsoft.com and block all other internet traffic." → Azure Firewall application rules.
  - "You need to detect and block network-based exploits in encrypted HTTPS traffic." → Azure Firewall Premium with IDPS + TLS inspection.
  - "You manage 10 firewalls across regions and need to push one rule set to all of them." → Firewall Policy + Azure Firewall Manager.
- **Memorization Tip:** NSG is the doorman checking IDs (IP/port). Azure Firewall is the full security team that reads the content of every package, checks it against a known-threats list, and can even open sealed envelopes (TLS inspection).

***

## Pocket Summary (5-Year Refresh)

- Azure Firewall is a stateful, managed L3–L7 firewall. NSGs are L3/L4 only.
- Three rule types: Application (FQDN), Network (IP/port), DNAT (inbound port forward).
- Premium adds IDPS (attack signatures) and TLS inspection (decrypt HTTPS to inspect).
- Firewall Policy holds all rules. Firewall Manager pushes policies to multiple firewalls.
- Hub-and-spoke: Firewall lives in hub. Spoke traffic is routed through it via UDRs.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/firewall/overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
