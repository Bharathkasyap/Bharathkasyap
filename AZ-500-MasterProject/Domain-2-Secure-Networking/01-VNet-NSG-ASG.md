# Virtual Networks, Network Security Groups, and Application Security Groups

> 📌 AZ-500 Exam Objective: Plan and implement security for virtual networks
> 🏷️ Domain: 2 — Secure Networking | Weight: 20–25%

***

## What Is This?

A Virtual Network (VNet) is your private network inside Azure. You split it into subnets. Network Security Groups (NSGs) are firewall rule lists that control which traffic is allowed in or out. Application Security Groups (ASGs) let you group virtual machines by role so your NSG rules stay clean and readable.

***

## Why Does This Exist?

Without NSGs, every VM in Azure could talk to every other VM and to the internet. That is dangerous. NSGs let you block all traffic by default and only open specific ports you choose. ASGs solve the problem of writing rules with long lists of IP addresses — you tag VMs by name instead.

***

## Who Actually Needs This? (Real-World Context)

A hospital network runs a web tier, an app tier, and a database tier in Azure. The security team needs to make sure the web servers can never talk directly to the database servers. They create three subnets and attach NSGs. The database subnet NSG only allows traffic from the app tier ASG on port 1433. Web servers cannot reach the database, even if a web server is hacked.

***

## How It Works (Pure Concept, No Portal)

- A **VNet** has an address space (like `10.0.0.0/16`). You carve it into subnets (like `10.0.1.0/24` for web, `10.0.2.0/24` for data).
- An **NSG** is a list of rules. Each rule has a priority number (100–4096). Lower number = higher priority. First matching rule wins.
- Every NSG has built-in default rules: deny all inbound from internet, allow all inside VNet, deny all outbound to internet (these sit at priority 65000+).
- You add custom rules above the defaults to allow specific traffic.
- **Inbound rules** filter traffic coming INTO the resource. **Outbound rules** filter traffic going OUT.
- You attach an NSG to a **subnet** (covers all VMs in that subnet) or directly to a **NIC** (covers one VM).
- **Service Tags** are named groups of Microsoft-managed IP ranges. Examples: `Internet`, `AzureLoadBalancer`, `VirtualNetwork`, `Storage`. You use them in rules instead of hard-coding IPs.
- **ASGs** let you tag a VM's NIC with a group name (like `WebServers` or `DBServers`). Then NSG rules can say "allow WebServers to talk to DBServers on port 1433" — no IPs needed.
- **NSG Flow Logs** record which traffic was allowed or denied. Stored in a storage account. Used with Traffic Analytics for visibility.

***

## 2026 Portal Walkthrough — Step by Step

**Create an NSG and allow only port 443 inbound, then attach to a subnet:**

1. Go to **portal.azure.com** → search **"Network security groups"** → click **Create**.
2. Fill in: Subscription, Resource Group, Name (e.g., `nsg-web-prod`), Region → click **Review + create** → **Create**.
3. Open the new NSG → left menu → **Inbound security rules** → click **+ Add**.
4. Set:
   - **Source**: `Any`
   - **Source port ranges**: `*`
   - **Destination**: `Any`
   - **Destination port ranges**: `443`
   - **Protocol**: `TCP`
   - **Action**: `Allow`
   - **Priority**: `100`
   - **Name**: `Allow-HTTPS-Inbound`
   → click **Add**.
5. The default deny rule at priority 65500 blocks everything else automatically.
6. To attach to a subnet: left menu → **Subnets** → click **+ Associate** → choose your VNet and subnet → **OK**.

**Create an ASG and assign a VM NIC:**

1. Search **"Application security groups"** → **Create** → fill in Name (e.g., `asg-web-servers`) → **Create**.
2. Open your VM → **Networking** → **Network Interface** → **Application security groups** → **Configure the application security groups** → select `asg-web-servers` → **Save**.
3. In your NSG inbound rule, set **Source** to `Application security group` → select `asg-web-servers`.

**CLI:**
```bash
# Create NSG
az network nsg create \
  --resource-group rg-prod \
  --name nsg-web-prod \
  --location eastus

# Add inbound rule to allow port 443 only
az network nsg rule create \
  --resource-group rg-prod \
  --nsg-name nsg-web-prod \
  --name Allow-HTTPS-Inbound \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --source-address-prefix '*' \
  --source-port-range '*' \
  --destination-address-prefix '*' \
  --destination-port-range 443 \
  --access Allow

# Attach NSG to subnet
az network vnet subnet update \
  --resource-group rg-prod \
  --vnet-name vnet-prod \
  --name subnet-web \
  --network-security-group nsg-web-prod

# Create ASG
az network asg create \
  --resource-group rg-prod \
  --name asg-web-servers \
  --location eastus
```

**PowerShell:**
```powershell
# Create NSG rule config
$rule1 = New-AzNetworkSecurityRuleConfig `
  -Name "Allow-HTTPS-Inbound" `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 100 `
  -SourceAddressPrefix '*' `
  -SourcePortRange '*' `
  -DestinationAddressPrefix '*' `
  -DestinationPortRange 443 `
  -Access Allow

# Create NSG with the rule
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName "rg-prod" `
  -Location "eastus" `
  -Name "nsg-web-prod" `
  -SecurityRules $rule1

# Attach to subnet
$vnet = Get-AzVirtualNetwork -Name "vnet-prod" -ResourceGroupName "rg-prod"
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "subnet-web"
$subnet.NetworkSecurityGroup = $nsg
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Priority (100–4096) | Lower number wins first. Rules are checked from lowest to highest. | Set critical allow/deny rules at low numbers like 100–200 |
| Direction (Inbound/Outbound) | Controls which way traffic flows | Inbound = traffic coming in; Outbound = traffic going out |
| Action (Allow/Deny) | What happens when the rule matches | Use Deny to block; Allow to permit |
| Source / Destination | Where traffic comes from / goes to | Use Service Tags instead of IP ranges when possible |
| Service Tag | Named group of Microsoft-managed IPs (e.g., `Storage`, `AzureCloud`) | Avoids hardcoding IPs that Microsoft updates automatically |
| Application Security Group | Groups VM NICs by role | Use when you have many VMs and want clean rules without IP lists |
| NSG on Subnet vs NIC | Subnet covers all VMs; NIC covers one VM | Use subnet for broad rules, NIC for VM-specific overrides |

***

## Comparison Table

| Feature | NSG | ASG |
|---|---|---|
| What it is | Firewall rule list | Logical VM grouping |
| Attached to | Subnet or NIC | NIC (referenced in NSG rules) |
| Controls traffic? | Yes | No — only groups VMs for NSG rules |
| Needs IP addresses? | Can use IPs or Service Tags | No — uses group names |
| Works alone? | Yes | No — needs NSG to do anything |

***

## What Happens If You Skip This?

Without NSGs, every VM in your VNet can communicate with every other VM on any port. If one VM is compromised, an attacker can move laterally to every other resource. They could reach your database directly from a hacked web server. Without ASGs, your NSG rules become massive lists of IP addresses that break every time a VM is rebuilt with a new IP.

***

## AZ-500 Exam Section

- **Trigger Words:** "network segmentation", "port filtering", "inbound rule", "service tag", "ASG", "block lateral movement", "subnet isolation"
- **Common Traps:**
  - NSGs do NOT deny traffic by default if a matching allow rule exists at a lower priority number. The deny-all default rules sit at priority 65000+. Your custom rules must be at a lower priority number to override them.
  - Attaching NSG to both a subnet AND a NIC means BOTH sets of rules are evaluated. Traffic must pass both NSGs.
  - Service Tags are updated by Microsoft — you do not maintain the IP list yourself.
- **How to Tell Apart:**
  - NSG vs Azure Firewall: NSG is L3/L4 (IP/port). Azure Firewall does L7 (FQDN, URLs, IDPS).
  - ASG vs NSG: ASG groups VMs. NSG filters traffic. You need both.
  - Service Tag vs ASG: Service Tag is for Microsoft services (Storage, AzureCloud). ASG is for your own VMs.
- **Likely Question Formats:**
  - "You need to allow only HTTPS from the internet to web VMs. What is the MINIMUM change?" → Add NSG inbound allow rule on port 443.
  - "You want to allow the app tier to reach the DB tier on port 1433 without using IP addresses." → Use ASGs.
  - "You need to block all traffic between two subnets." → Add NSG Deny rule between them.
- **Memorization Tip:** Think of NSG as a bouncer with a list. Lower priority number = checked first. ASG = a VIP badge for your VMs so the bouncer recognizes them by role, not face.

***

## Pocket Summary (5-Year Refresh)

- VNets split into subnets. NSGs control traffic in/out of subnets and NICs.
- NSG rules have priorities. Lowest number wins. Default = deny all (at priority 65000+).
- Service Tags replace IP addresses for Microsoft services in NSG rules.
- ASGs group VMs by role. NSG rules reference ASG names instead of IP addresses.
- NSG Flow Logs capture allow/deny decisions and send them to a storage account for analysis.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
