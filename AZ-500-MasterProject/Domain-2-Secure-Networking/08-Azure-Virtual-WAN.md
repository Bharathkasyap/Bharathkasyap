# Azure Virtual WAN

> 📌 AZ-500 Exam Objective: Plan and implement advanced network security
> 🏷️ Domain: 2 — Secure Networking | Weight: 20–25%

***

## What Is This?

Azure Virtual WAN (vWAN) is a Microsoft-managed networking service that connects your branch offices, remote users, and Azure VNets through a central hub. Think of it as a global network backbone that Microsoft builds and operates for you. Instead of manually stitching together VPN gateways, VNet peerings, and routing tables, Virtual WAN does it all in one unified service.

***

## Why Does This Exist?

Large organizations with dozens of branch offices and Azure regions face a complex networking problem. Every branch needs a path to every other branch and to every Azure VNet. Building that yourself requires hundreds of VPN connections, custom routing tables, and ongoing maintenance. Virtual WAN replaces all of that with a managed hub that handles routing automatically. Adding a new branch takes minutes instead of weeks.

***

## Who Actually Needs This? (Real-World Context)

A global retail chain has 80 store locations, 5 distribution centers, and workloads in three Azure regions. Their legacy MPLS network is expensive and slow to provision new locations. They migrate to Azure Virtual WAN Standard. Each store gets an SD-WAN device that connects to the nearest Virtual WAN hub. The hubs connect to each other automatically over the Microsoft global backbone. All internet-bound traffic from stores routes through a centralized Azure Firewall in each hub (Secured Virtual Hub). The entire migration takes 3 months instead of the 18 months a traditional WAN redesign would take.

***

## How It Works (Pure Concept, No Portal)

- **Virtual WAN** is the top-level resource. It is a container that holds one or more **hubs**.
- A **Hub** is a Microsoft-managed VNet in an Azure region. You do not control the hub VNet directly. It lives inside the Virtual WAN service.
- Resources that connect to a hub:
  - **VNets** (Azure spoke VNets via hub-VNet connections)
  - **Site-to-Site VPN** (for branch offices with VPN devices)
  - **ExpressRoute** (for branch offices with private circuits)
  - **Point-to-Site VPN** (for remote user clients)
  - **SD-WAN partners** (integrated SD-WAN devices from vendors like Cisco, VMware, Palo Alto)
- All connected sites and VNets can reach each other automatically — this is called **any-to-any** or **global transit** connectivity. No manually configured peering or routing tables needed.
- **Virtual WAN SKUs:**
  - **Basic:** Only supports S2S VPN connectivity. No VNet-to-VNet through hub, no ExpressRoute, no User VPN. Used for simple branch VPN scenarios.
  - **Standard:** Supports everything — S2S, P2S, ExpressRoute, VNet connections, VNet-to-VNet transit, Secured Hub, NVA in hub.
- **Secured Virtual Hub:** A Standard hub that has Azure Firewall (and/or third-party NVA) deployed inside it. All traffic flowing through the hub is inspected by the firewall. Managed via Azure Firewall Manager.
- **Routing policies:** In a Secured Virtual Hub, you configure routing policies to send all private traffic and/or all internet traffic through the Azure Firewall. This gives you centralized security inspection for all your branches and VNets.
- **Hub-to-hub:** Multiple hubs in different regions automatically route traffic between each other over the Microsoft backbone — no user-managed peering required.
- **NVA in hub:** You can deploy third-party Network Virtual Appliances (NVAs) like Palo Alto or Fortinet directly into the hub alongside or instead of Azure Firewall.

***

## 2026 Portal Walkthrough — Step by Step

**Create a Virtual WAN and add a hub:**

1. Go to **portal.azure.com** → search **"Virtual WANs"** → click **Create**.
2. Fill in: Subscription, Resource Group, Name (e.g., `vwan-global-prod`), Region, **Type**: `Standard` → **Review + create** → **Create**.
3. Open `vwan-global-prod` → left menu → **Hubs** → **+ New hub**.
4. Fill in:
   - **Region**: `East US`
   - **Name**: `hub-eastus-prod`
   - **Hub private address space**: `10.10.0.0/23` (must not overlap with any connected VNets)
   → **Next: Site to site**.
5. **Site to site tab**: If enabling S2S VPN in the hub → **Gateway scale units**: `1` (each unit = 500 Mbps) → **Next**.
6. **Point to site tab**: Optional — skip or configure P2S VPN.
7. **ExpressRoute tab**: Optional — skip or configure ER gateway → **Review + create** → **Create** (hub takes ~20–30 minutes to deploy).

**Connect a VNet to the hub:**

1. Open the vWAN → **Virtual network connections** → **+ Add connection**.
2. Fill in: Connection name, **Hubs**: select `hub-eastus-prod`, **Subscription**, **Virtual network**: select your spoke VNet → **Create**.

**Create a Secured Virtual Hub (add Azure Firewall to hub):**

1. Open the hub (`hub-eastus-prod`) → left menu → **Azure Firewall**.
2. Click **Enable Azure Firewall** → Select a Firewall Policy → **Confirm**.
   *(Alternatively, create the hub as a Secured Hub from the start via Azure Firewall Manager.)*
3. Go to **Azure Firewall Manager** → **Virtual hubs** → select `hub-eastus-prod` → **Security configuration**.
4. Set **Private traffic** → `Send via Azure Firewall`, **Internet traffic** → `Send via Azure Firewall` → **Save**.

**CLI:**
```bash
# Create Virtual WAN
az network vwan create \
  --resource-group rg-network \
  --name vwan-global-prod \
  --location eastus \
  --type Standard

# Create Virtual Hub
az network vhub create \
  --resource-group rg-network \
  --name hub-eastus-prod \
  --location eastus \
  --vwan vwan-global-prod \
  --address-prefix 10.10.0.0/23

# Connect a spoke VNet to the hub
az network vhub connection create \
  --resource-group rg-network \
  --name conn-vnet-prod \
  --vhub-name hub-eastus-prod \
  --remote-vnet vnet-prod \
  --remote-vnet-resource-group rg-prod
```

**PowerShell:**
```powershell
# Create Virtual WAN
$vwan = New-AzVirtualWan `
  -ResourceGroupName "rg-network" `
  -Name "vwan-global-prod" `
  -Location "eastus" `
  -VirtualWANType Standard

# Create Virtual Hub
$hub = New-AzVirtualHub `
  -ResourceGroupName "rg-network" `
  -Name "hub-eastus-prod" `
  -Location "eastus" `
  -VirtualWan $vwan `
  -AddressPrefix "10.10.0.0/23"

# Connect a VNet to the hub
$vnet = Get-AzVirtualNetwork -Name "vnet-prod" -ResourceGroupName "rg-prod"
New-AzVirtualHubVnetConnection `
  -ResourceGroupName "rg-network" `
  -VirtualHubName "hub-eastus-prod" `
  -Name "conn-vnet-prod" `
  -RemoteVirtualNetwork $vnet
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/virtual-wan/

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| vWAN Type: Basic | Only S2S VPN. No VNet transit, no ExpressRoute, no Secured Hub. | Only for simple branch-to-Azure VPN. Not suitable for enterprise. |
| vWAN Type: Standard | Full global transit — S2S, P2S, ER, VNet-to-VNet, Secured Hub | Use for all enterprise and multi-region deployments |
| Hub address prefix | Private IP space for the hub itself. Must not overlap with connected VNets. | Set a dedicated /23 or larger, separate from all spoke CIDRs |
| Gateway scale units | Controls VPN or ER gateway capacity inside the hub. Each S2S unit = 500 Mbps. | Size based on total branch traffic. Over-provision slightly for bursts. |
| Secured Virtual Hub | Azure Firewall (or NVA) inside the hub inspects all transit traffic | Use when you need centralized security across all branches and VNets |
| Routing Policy: Private traffic via Firewall | All VNet-to-VNet and branch-to-VNet traffic inspected by hub firewall | Enable in regulated environments requiring east-west traffic inspection |
| Routing Policy: Internet traffic via Firewall | All internet-bound traffic routed through hub firewall | Use to enforce consistent internet security policy for all connected sites |
| Hub-to-Hub | Automatic inter-hub routing over Microsoft backbone | No action needed — enabled by default in Standard vWAN |

***

## Comparison Table

| Feature | Traditional Hub-and-Spoke | Azure Virtual WAN |
|---|---|---|
| Hub VNet management | Customer-managed | Microsoft-managed |
| Routing configuration | Manual UDRs and peering | Automatic |
| Adding a new branch | Configure VPN gateway, routing, peering manually | Connect SD-WAN device or VPN — done |
| VNet-to-VNet transit | Requires NVA or additional peering | Built-in (Standard SKU) |
| Centralized firewall | Azure Firewall in hub VNet (manual routing) | Secured Virtual Hub (automated routing policy) |
| Global multi-hub | Requires manual inter-region routing | Automatic hub-to-hub over Microsoft backbone |
| Best for | Small deployments, simple topologies | Large enterprises, global networks, many branches |

***

## What Happens If You Skip This?

Without Virtual WAN, a company with many branches and regions builds a tangled web of individual VPN connections, VNet peerings, and custom routing tables. Each new branch takes days to onboard. Route changes must be made manually in dozens of places. Security policies differ between branches because there is no centralized enforcement point. Troubleshooting connectivity issues takes hours because there is no unified view of the network topology.

***

## AZ-500 Exam Section

- **Trigger Words:** "hub-and-spoke", "SD-WAN", "virtual WAN", "secured hub", "global transit", "branch connectivity", "centralized routing", "any-to-any connectivity"
- **Common Traps:**
  - Virtual WAN is NOT just a VPN service. It is a full global transit network architecture that includes VPN, ExpressRoute, P2S, VNet connectivity, and optional centralized firewall.
  - vWAN Basic only supports S2S VPN. It does NOT support VNet-to-VNet transit, ExpressRoute, or Secured Hub. Many questions rely on this distinction.
  - The hub VNet is managed by Microsoft — you cannot add your own resources directly into the hub VNet (except via Azure Firewall Manager for Secured Hubs, or NVA partners).
  - Secured Virtual Hub requires Standard vWAN — Basic does not support it.
  - Hub-to-hub routing happens automatically — you do not configure BGP or UDRs between hubs.
- **How to Tell Apart:**
  - "Manage many branch offices from one network" → Virtual WAN.
  - "Central security policy across all branches and VNets" → Secured Virtual Hub.
  - "Simple spoke VNets in one region" → Traditional hub-and-spoke (no need for vWAN).
  - "Global transit between regions automatically" → Virtual WAN Standard.
- **Likely Question Formats:**
  - "You need to connect 50 branch offices and inspect all traffic centrally. What is the most scalable solution?" → Azure Virtual WAN Standard with Secured Virtual Hubs.
  - "You use vWAN Basic but VNets cannot communicate with each other through the hub. Why?" → Basic does not support VNet-to-VNet transit. Upgrade to Standard.
  - "You want all internet traffic from branches to route through Azure Firewall in the hub." → Secured Virtual Hub with Internet routing policy.
- **Memorization Tip:** Virtual WAN is like a Microsoft-operated airport hub system. Every city (branch) flies to the hub. The hub connects to all other hubs. You book the flights (connections) — Microsoft runs the airport (hub infrastructure and routing).

***

## Pocket Summary (5-Year Refresh)

- Azure Virtual WAN is a managed global transit network: branches, VNets, ExpressRoute, and remote users all connect through Microsoft-managed hubs.
- Basic vWAN = S2S VPN only. Standard vWAN = everything including VNet transit and Secured Hub.
- Secured Virtual Hub = hub with Azure Firewall inside, centrally managed routing and security policy.
- Hub-to-hub routing is automatic — no manual UDRs between regions.
- Best for: large enterprises with many branches needing scalable, centrally managed hybrid networking.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/virtual-wan/virtual-wan-about
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
