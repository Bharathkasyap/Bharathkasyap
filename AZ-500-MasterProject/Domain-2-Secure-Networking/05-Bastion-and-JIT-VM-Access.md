# Azure Bastion and Just-in-Time VM Access

> 📌 AZ-500 Exam Objective: Plan and implement advanced security for infrastructure
> 🏷️ Domain: 2 — Secure Networking | Weight: 20–25%

***

## What Is This?

Azure Bastion gives you a secure way to connect to your virtual machines using a web browser — no public IP address needed on the VM. Just-in-Time (JIT) VM Access opens management ports (like RDP and SSH) for a limited time window when you need them, then closes them automatically. Both features protect your VMs from being attacked through open management ports.

***

## Why Does This Exist?

Every VM with a public IP and open port 3389 (RDP) or port 22 (SSH) is constantly being scanned and attacked by bots looking for weak passwords. The traditional solution was to put VMs on a VPN, but that requires VPN client setup and certificates. Azure Bastion removes the public IP from your VM entirely. JIT removes the always-open management port and only opens it for approved users for a defined time window.

***

## Who Actually Needs This? (Real-World Context)

A healthcare IT team manages 50 Windows VMs running electronic health record software. Leaving RDP open 24/7 is a compliance violation under HIPAA. They enable Azure Bastion so administrators connect through the Azure portal — no public IP on any VM. They also enable JIT so that when a developer needs emergency access, they request it, a manager approves it, and port 3389 opens only for that developer's IP for 2 hours. The audit log captures every access request automatically.

***

## How It Works (Pure Concept, No Portal)

**Azure Bastion:**
- Bastion is a managed PaaS service that lives inside a special subnet (`AzureBastionSubnet`) in your VNet.
- You connect to a VM through the Azure portal (or the Bastion native client). The connection travels over HTTPS (port 443) from your browser to the Bastion host. Bastion then connects to your VM over port 3389 (RDP) or port 22 (SSH) on the private IP — inside Azure, never over the public internet.
- Your VM has no public IP and no exposed management port. The Bastion host is the only thing with a public IP.
- **Bastion SKUs:**
  - **Basic**: Browser-based RDP/SSH. No file transfer, no audio.
  - **Standard**: Adds shareable links, IP-based connection, file transfer (RDP), custom ports, native client support.
  - **Premium**: Adds session recording, private-only Bastion (no public IP on Bastion itself), and more advanced features.
- The `AzureBastionSubnet` must be named exactly that and must be at least `/26`.
- NSG on `AzureBastionSubnet` must allow: inbound HTTPS 443 from internet, outbound 3389/22 to VirtualNetwork, inbound GatewayManager 443, outbound AzureCloud 443.

**Just-in-Time (JIT) VM Access:**
- JIT is a feature of **Microsoft Defender for Cloud** (specifically, Defender for Servers).
- When enabled on a VM, JIT creates NSG rules that block the management ports (3389, 22, etc.) by default — always.
- When a user requests access: they specify their source IP and how many hours they need. An admin approves the request (or it can be auto-approved based on policy).
- JIT then temporarily adds an NSG allow rule for that specific port and source IP for the approved duration. After the timer expires, the allow rule is automatically removed.
- JIT is NOT a connection tool — it only controls the firewall rule. You still need a connection method (Bastion, or your own client).
- Bastion and JIT complement each other: JIT controls WHO can open the port and WHEN; Bastion provides the HOW to connect.
- JIT requires the VM to be associated with a Network Security Group.

***

## 2026 Portal Walkthrough — Step by Step

**Deploy Azure Bastion:**

1. Go to **portal.azure.com** → search **"Bastions"** → click **Create**.
2. Fill in: Subscription, Resource Group, Name (e.g., `bastion-hub-prod`), Region.
3. **Tier**: Select `Standard` (for file transfer and native client) or `Basic`.
4. **Virtual network**: Select your VNet (e.g., `vnet-prod`).
5. **Subnet**: Must be named `AzureBastionSubnet`. If it does not exist, click **Manage subnet configuration** → add a new subnet named exactly `AzureBastionSubnet` with at least `/26` address prefix → return.
6. **Public IP address**: Create new (e.g., `bastion-pip-prod`) → **Review + create** → **Create** (takes ~5 minutes).
7. To connect to a VM: Open any VM → left menu → **Connect** → **Bastion** → enter credentials → **Connect**. Browser opens an RDP or SSH session.

**Enable JIT VM Access:**

1. Go to **Microsoft Defender for Cloud** → **Workload protections** → **Just-in-time VM access**.
   *(Or: open the VM → left menu → **Configuration** → **Just-in-time VM access** → **Enable just-in-time**.)*
2. Under **Not configured** tab → select your VM → click **Enable JIT on 1 VMs**.
3. JIT default policy adds rules for ports 22, 3389, 5985, 5986. To customize: click the VM name → edit ports, allowed source IPs, max request time.

**Request JIT access:**
1. **Defender for Cloud** → **Just-in-time VM access** → **Configured** tab → select VM → **Request access**.
2. Toggle on the ports you need → enter **Source IP** (your IP or range) → set **Time range** (e.g., 2 hours) → **Open ports**.
3. The NSG rule is created automatically. It expires after the time window.

**CLI:**
```bash
# Create Bastion host
az network bastion create \
  --resource-group rg-prod \
  --name bastion-hub-prod \
  --location eastus \
  --vnet-name vnet-prod \
  --public-ip-address bastion-pip-prod \
  --sku Standard

# Enable JIT policy on a VM (port 3389, max 3 hours)
az security jit-policy create \
  --resource-group rg-prod \
  --vm-name vm-web-01 \
  --location eastus \
  --port 3389 \
  --protocol TCP \
  --max-request-access-duration PT3H \
  --allowed-source-address-prefix '*'
```

**PowerShell:**
```powershell
# Create Public IP for Bastion
$pip = New-AzPublicIpAddress `
  -ResourceGroupName "rg-prod" `
  -Name "bastion-pip-prod" `
  -Location "eastus" `
  -AllocationMethod Static `
  -Sku Standard

# Get VNet and subnet
$vnet = Get-AzVirtualNetwork -Name "vnet-prod" -ResourceGroupName "rg-prod"
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "AzureBastionSubnet" -VirtualNetwork $vnet

# Create Bastion host
New-AzBastion `
  -ResourceGroupName "rg-prod" `
  -Name "bastion-hub-prod" `
  -PublicIpAddress $pip `
  -VirtualNetworkId $vnet.Id `
  -Sku "Standard"
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/bastion/ and https://learn.microsoft.com/en-us/azure/defender-for-cloud/just-in-time-access-usage

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| AzureBastionSubnet | Dedicated subnet for Bastion. Must be /26 or larger. | Required — Bastion cannot deploy without this exact subnet name |
| Bastion SKU Basic | Browser-based RDP/SSH only | Use for simple VM access without file transfer needs |
| Bastion SKU Standard | Adds file transfer, native client, shareable links | Use in most enterprise deployments |
| Bastion SKU Premium | Adds session recording, private Bastion (no public IP) | Use for regulated environments requiring audit recordings |
| JIT Default Ports | Blocks 22, 3389, 5985, 5986 by default | Customize to match only the ports your VMs actually use |
| JIT Max Request Time | Maximum time a user can request access (1 hour to 24 hours) | Set to minimum needed — 1–3 hours for most admin tasks |
| JIT Source IP | Restricts the opened port to a specific requestor IP | Always set to the requestor's IP, never wildcard (*) in production |
| JIT Approval Flow | Require manager approval before access is granted | Enable for production environments and compliance requirements |

***

## Comparison Table

| Feature | Azure Bastion | JIT VM Access |
|---|---|---|
| What it does | Provides a secure browser-based RDP/SSH connection | Temporarily opens a management port in the NSG |
| Where it lives | `AzureBastionSubnet` in your VNet | NSG on the VM's NIC or subnet |
| Requires Defender for Cloud | No | Yes (Defender for Servers plan) |
| VM needs a public IP | No — that's the point | Optional — works with private or public IP VMs |
| Is it a connection method | Yes | No — it only controls the firewall rule |
| Audit logging | Azure Activity Log | Defender for Cloud Activity Log |
| Works together | Yes — use both for maximum protection | Yes — JIT opens the port, Bastion connects through it |

***

## What Happens If You Skip This?

Without Bastion, VMs need public IPs or VPN access for management. Public IPs with open RDP/SSH ports are attacked within minutes of exposure. Microsoft data shows internet-exposed RDP ports are attacked thousands of times per hour. Without JIT, management ports stay open 24/7 — a permanent attack surface. A single compromised admin credential can give an attacker full VM access and a foothold into your entire network.

***

## AZ-500 Exam Section

- **Trigger Words:** "no public IP on VM", "browser-based RDP", "browser-based SSH", "time-limited access", "JIT", "Bastion", "management port", "temporary access", "just-in-time"
- **Common Traps:**
  - Bastion and JIT are NOT the same thing and NOT interchangeable.
    - Bastion = HOW you connect (the transport method).
    - JIT = WHEN the port is open (the firewall rule).
  - JIT requires **Microsoft Defender for Cloud** with **Defender for Servers** enabled. It does NOT work without it.
  - `AzureBastionSubnet` must be named EXACTLY that. A different name breaks the deployment.
  - `AzureBastionSubnet` must be at least `/26`. A `/27` will fail.
  - Bastion Premium is the only tier that supports **session recording**.
  - JIT opens ports on the NSG — it does NOT deploy a proxy or agent on the VM.
- **How to Tell Apart:**
  - "Secure connection to VM without public IP" → Azure Bastion.
  - "Temporarily open port 3389 for 2 hours" → JIT.
  - "Record admin sessions" → Bastion Premium.
  - "Approve access requests before port opens" → JIT with approval flow.
- **Likely Question Formats:**
  - "You need to connect to a VM that has no public IP. What do you use?" → Azure Bastion.
  - "You need to open RDP only when an admin needs it and close it automatically after 2 hours." → JIT VM Access.
  - "Which Bastion tier supports session recording?" → Premium.
  - "JIT is not working on your VMs. What is missing?" → Microsoft Defender for Servers is not enabled.
- **Memorization Tip:** Bastion is the secure tunnel you walk through. JIT is the guard who opens the door only for the time you need and locks it behind you.

***

## Pocket Summary (5-Year Refresh)

- Azure Bastion provides browser-based RDP/SSH without a public IP on the VM.
- Bastion lives in `AzureBastionSubnet` (must be /26+). SKUs: Basic, Standard, Premium.
- JIT VM Access temporarily opens management ports (3389, 22) for approved users, then closes them.
- JIT requires Microsoft Defender for Cloud with Defender for Servers.
- Use both together: JIT controls the firewall rule; Bastion provides the connection.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/bastion/bastion-overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
