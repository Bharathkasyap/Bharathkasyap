# Azure VPN Gateway and ExpressRoute

> 📌 AZ-500 Exam Objective: Plan and implement secure connectivity between networks
> 🏷️ Domain: 2 — Secure Networking | Weight: 20–25%

***

## What Is This?

Azure VPN Gateway and ExpressRoute are two ways to connect your on-premises network to Azure. VPN Gateway creates an encrypted tunnel over the public internet. ExpressRoute is a private, dedicated circuit through a connectivity provider — it never touches the public internet. Both let your on-premises servers and Azure resources communicate as if they were on the same private network.

***

## Why Does This Exist?

Businesses cannot move all their data and applications to the cloud overnight. They need hybrid connectivity — some workloads stay on-premises, some move to Azure, and they need to talk to each other securely. VPN Gateway provides an affordable, quick-to-deploy encrypted link. ExpressRoute provides a high-bandwidth, low-latency, high-reliability private connection for businesses with strict performance or compliance requirements.

***

## Who Actually Needs This? (Real-World Context)

A manufacturing company runs its ERP system on-premises and is slowly migrating to Azure. Their HR team now uses Azure-hosted applications, but those apps need to query the on-premises Active Directory and file servers. The IT team sets up a Site-to-Site VPN Gateway so both sides see each other as private networks. Two years later, when the company processes enough data that VPN latency is hurting performance, they upgrade to an ExpressRoute circuit for a dedicated 1 Gbps private connection.

***

## How It Works (Pure Concept, No Portal)

**VPN Gateway:**
- A VPN Gateway is an Azure resource deployed into a special subnet called `GatewaySubnet` inside your VNet.
- It creates **IPsec/IKE tunnels** — encrypted tunnels that carry your network traffic over the public internet.
- **Three connection types:**
  - **Site-to-Site (S2S)** — connects an on-premises network to Azure. Requires a physical VPN device on-premises with a public IP.
  - **Point-to-Site (P2S)** — connects individual client computers to Azure. Great for remote workers. Uses SSL/TLS or IKEv2.
  - **VNet-to-VNet** — connects two Azure VNets in different regions using the VPN Gateway.
- **VPN Gateway SKUs:** Basic, VpnGw1–VpnGw5. Higher SKUs = more throughput, more tunnels, support for BGP and zone redundancy.
  - Basic SKU does NOT support BGP and is not recommended for production.
- **Active-Active vs Active-Passive:** Active-Active uses two gateway IPs for higher availability. Active-Passive is the default.
- **BGP (Border Gateway Protocol):** Dynamic routing protocol. Instead of manually adding static routes, BGP automatically exchanges routes between your on-premises router and Azure. Required for ExpressRoute coexistence.
- **Traffic is encrypted** because IPsec encrypts the tunnel. This is a key difference from ExpressRoute.

**ExpressRoute:**
- A private circuit from your on-premises location to Azure through a Microsoft partner (connectivity provider) or direct from your facility (ExpressRoute Direct).
- Traffic NEVER traverses the public internet. It travels over a private MPLS or Ethernet circuit.
- **NOT encrypted by default.** The circuit is private but not encrypted. To add encryption, you can add **MACsec** (Layer 2) or run **IPsec over ExpressRoute**.
- **Peering types:**
  - **Azure private peering** — connects to Azure VNets (your private resources).
  - **Microsoft peering** — connects to Microsoft 365 and Azure public services.
- **Circuit speeds:** 50 Mbps to 100 Gbps (with ExpressRoute Direct).
- **ExpressRoute Global Reach** — connects two on-premises locations to each other through the Microsoft backbone, using their ExpressRoute circuits.
- **ExpressRoute + VPN failover:** Run both. ExpressRoute is primary. VPN Gateway takes over if ExpressRoute fails. Provides maximum availability for hybrid connectivity.
- **SLA:** ExpressRoute has a 99.95% SLA. VPN Gateway has a 99.9–99.99% SLA depending on SKU.

**GatewaySubnet:**
- Both VPN Gateway and ExpressRoute Gateway require a subnet named exactly `GatewaySubnet`.
- Recommended size: `/27` or larger. Do not put other resources in this subnet.

***

## 2026 Portal Walkthrough — Step by Step

**Create a VPN Gateway and configure a Site-to-Site connection:**

1. Go to **portal.azure.com** → search **"Virtual network gateways"** → click **Create**.
2. Fill in:
   - Name: `vgw-prod`
   - **Gateway type**: `VPN`
   - **VPN type**: `Route-based` (recommended — supports BGP, IKEv2, active-active)
   - **SKU**: `VpnGw2` (for production; supports BGP and zone redundancy)
   - **Generation**: `Generation2`
   - **Virtual network**: Select your VNet — the `GatewaySubnet` is used automatically
   - **Public IP address**: Create new (e.g., `vgw-pip-prod`)
   - **Enable active-active mode**: Recommended for HA
   → **Review + create** → **Create** (takes 20–45 minutes).

3. Create a **Local Network Gateway** (represents your on-premises VPN device):
   - Search **"Local network gateways"** → **Create**
   - Name: `lgw-onprem`, IP address: your on-premises VPN device's public IP
   - **Address space**: your on-premises network CIDR (e.g., `192.168.0.0/16`)
   → **Review + create** → **Create**.

4. Create the **Connection** (links the VPN Gateway to the Local Network Gateway):
   - Open your VPN Gateway (`vgw-prod`) → **Connections** → **+ Add**
   - **Connection type**: `Site-to-site (IPsec)`
   - **Local network gateway**: Select `lgw-onprem`
   - **Shared key (PSK)**: Enter a strong pre-shared key (copy this — you'll use it on the on-prem VPN device)
   → **OK**.

5. Configure the matching settings on your on-premises VPN device using the Azure VPN Gateway's public IP and the shared key.

**CLI:**
```bash
# Create VPN Gateway
az network vnet-gateway create \
  --resource-group rg-prod \
  --name vgw-prod \
  --location eastus \
  --vnet vnet-prod \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw2 \
  --generation Generation2 \
  --public-ip-addresses vgw-pip-prod \
  --no-wait

# Create Local Network Gateway (on-prem side)
az network local-gateway create \
  --resource-group rg-prod \
  --name lgw-onprem \
  --location eastus \
  --gateway-ip-address 203.0.113.10 \
  --local-address-prefixes 192.168.0.0/16

# Create S2S VPN Connection
az network vpn-connection create \
  --resource-group rg-prod \
  --name conn-s2s-onprem \
  --vnet-gateway1 vgw-prod \
  --local-gateway2 lgw-onprem \
  --shared-key "YourStr0ngPSK!"
```

**PowerShell:**
```powershell
# Get VNet and GatewaySubnet
$vnet = Get-AzVirtualNetwork -Name "vnet-prod" -ResourceGroupName "rg-prod"
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
$pip = New-AzPublicIpAddress -Name "vgw-pip-prod" -ResourceGroupName "rg-prod" -Location "eastus" -AllocationMethod Dynamic

# Create IP config for gateway
$ipconfig = New-AzVirtualNetworkGatewayIpConfig `
  -Name "gwipconfig" `
  -SubnetId $subnet.Id `
  -PublicIpAddressId $pip.Id

# Create VPN Gateway
New-AzVirtualNetworkGateway `
  -Name "vgw-prod" `
  -ResourceGroupName "rg-prod" `
  -Location "eastus" `
  -IpConfigurations $ipconfig `
  -GatewayType Vpn `
  -VpnType RouteBased `
  -GatewaySku VpnGw2 `
  -VpnGatewayGeneration Generation2
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/vpn-gateway/ and https://learn.microsoft.com/en-us/azure/expressroute/

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| VPN Type: Route-based | Uses routing tables. Supports BGP, IKEv2, P2S. | Always use Route-based for new deployments |
| VPN Type: Policy-based | Legacy. Only supports IKEv1, one S2S tunnel. | Only for compatibility with very old VPN devices |
| Gateway SKU (VpnGw1–5) | Controls max throughput and tunnel count | Use at least VpnGw2 for production. Match to bandwidth needs. |
| Active-Active Mode | Two gateway instances, higher availability | Use in production for HA |
| BGP | Dynamic route exchange. Required for ExpressRoute coexistence. | Use when you have complex routing or failover requirements |
| ExpressRoute Circuit SKU | Local, Standard, Premium — controls which Azure regions are reachable | Standard for same geo. Premium for global reach. |
| MACsec / IPsec over ExpressRoute | Adds encryption to the ExpressRoute circuit | Required when compliance demands encrypted private circuit traffic |
| Global Reach | Connects two on-prem sites through Microsoft backbone via their ER circuits | Use to replace MPLS between branch offices |

***

## Comparison Table

| Feature | VPN Gateway (S2S) | ExpressRoute |
|---|---|---|
| Travels over public internet | Yes | No — private circuit |
| Encrypted | Yes (IPsec) | NOT by default |
| Max bandwidth | ~10 Gbps (VpnGw5) | Up to 100 Gbps |
| Latency | Variable (internet path) | Consistent, low (private circuit) |
| SLA | 99.9–99.99% | 99.95% |
| Cost | Lower (pays for gateway + data) | Higher (circuit + gateway fees) |
| Setup time | Hours | Days to weeks (provider provisioning) |
| Good for | Dev/test, small orgs, remote workers | Enterprise, high-throughput, compliance |

***

## What Happens If You Skip This?

Without hybrid connectivity, on-premises systems cannot securely communicate with Azure resources. Teams resort to exposing Azure resources publicly, which creates massive security risks. Applications that need both cloud and on-premises data either cannot function or have to send data through insecure paths. Compliance frameworks like PCI-DSS require private, encrypted channels for sensitive data. Skipping this means either broken hybrid applications or exposed data paths.

***

## AZ-500 Exam Section

- **Trigger Words:** "site-to-site", "hybrid connectivity", "IPsec", "ExpressRoute", "dedicated circuit", "private peering", "point-to-site", "VPN Gateway", "BGP", "failover", "MACsec"
- **Common Traps:**
  - **ExpressRoute is NOT encrypted by default.** This is the most tested fact. The circuit is private but plaintext unless you add MACsec or run IPsec over it.
  - VPN Gateway uses `GatewaySubnet` — it must be named exactly that.
  - Policy-based VPN only supports a single S2S tunnel. Exam questions may use legacy VPN device scenarios.
  - BGP is NOT supported on the Basic VPN Gateway SKU.
  - ExpressRoute peering types: Azure private peering (your VNets) vs Microsoft peering (M365 and public Azure services).
- **How to Tell Apart:**
  - "Encrypted tunnel over internet" → VPN Gateway.
  - "Private dedicated circuit, no internet" → ExpressRoute.
  - "Encrypt ExpressRoute" → MACsec or IPsec over ExpressRoute.
  - "Remote worker VPN" → Point-to-Site VPN.
  - "Failover for ExpressRoute" → ExpressRoute + VPN Gateway coexistence.
- **Likely Question Formats:**
  - "You need a private connection from on-premises to Azure that never traverses the internet. Encryption is required." → ExpressRoute + MACsec or IPsec.
  - "Is ExpressRoute encrypted?" → No, not by default.
  - "You need individual remote workers to connect to Azure securely." → Point-to-Site VPN.
  - "You need high availability for ExpressRoute. What do you add?" → VPN Gateway as failover.
- **Memorization Tip:** VPN = encrypted mail truck driving on public roads. ExpressRoute = private railway only for you, but the cargo is not in a locked box unless you add MACsec.

***

## Pocket Summary (5-Year Refresh)

- VPN Gateway creates encrypted IPsec tunnels over the public internet for S2S, P2S, and VNet-to-VNet connectivity.
- ExpressRoute is a private dedicated circuit — traffic never touches the internet but is NOT encrypted by default.
- To encrypt ExpressRoute traffic, add MACsec (Layer 2) or IPsec (Layer 3).
- Both require a `GatewaySubnet` in the VNet.
- Use ExpressRoute + VPN failover for maximum availability in enterprise hybrid scenarios.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/expressroute/expressroute-introduction
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
