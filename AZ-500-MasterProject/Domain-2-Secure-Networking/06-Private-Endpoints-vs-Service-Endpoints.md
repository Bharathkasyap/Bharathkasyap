# Private Endpoints and Service Endpoints

> 📌 AZ-500 Exam Objective: Plan and implement security for Azure services
> 🏷️ Domain: 2 — Secure Networking | Weight: 20–25%

***

## What Is This?

Service Endpoints and Private Endpoints are two ways to connect your Azure Virtual Network (VNet) securely to Azure PaaS services like Storage, SQL Database, and Key Vault. Service Endpoints keep traffic on the Azure backbone but the service still has a public IP. Private Endpoints give the PaaS service a private IP address directly inside your VNet and remove its public access entirely.

***

## Why Does This Exist?

By default, Azure PaaS services have public endpoints — anyone on the internet can try to reach your storage account or SQL database. You lock them down with firewall rules, but the service is still accessible over the public internet path. Service Endpoints and Private Endpoints provide private, network-level isolation so your data traffic never leaves the Microsoft backbone and the service is not reachable from the public internet.

***

## Who Actually Needs This? (Real-World Context)

A retail bank uses Azure SQL Database to store customer account data. Compliance requires that database traffic never traverse the public internet. The security team creates a Private Endpoint for the SQL Database inside their production VNet. The database gets a private IP address like `10.0.3.5`. They disable all public network access on the SQL server. Now only resources inside the VNet (or connected via VPN/ExpressRoute) can reach the database. A hacker on the internet cannot even discover the endpoint exists.

***

## How It Works (Pure Concept, No Portal)

**Service Endpoints:**
- Extends the VNet's identity to the PaaS service. The traffic travels from the VM through the Azure backbone — not through the public internet.
- The PaaS service still has a public IP. You restrict access by adding the subnet to the PaaS service's firewall allowlist.
- Does NOT give the service a private IP in your VNet.
- Works by adding a route and network policy to the subnet. When a VM in that subnet talks to, say, Azure Storage, traffic goes directly over the Azure backbone.
- Supported services: Storage, SQL Database, Key Vault, Cosmos DB, Service Bus, Event Hub, and more.
- A subnet can have multiple service endpoints for different services.
- **Limitation:** Service Endpoint rules apply per-subnet. On-premises resources (via VPN/ExpressRoute) cannot use Service Endpoints to reach PaaS services privately.

**Private Endpoints:**
- Creates a Network Interface (NIC) inside your VNet with a private IP address assigned to it. That NIC is connected to the PaaS service.
- The PaaS service is reachable at that private IP — from within the VNet, from peered VNets, and from on-premises via VPN/ExpressRoute.
- You can (and should) then disable all public network access on the PaaS service.
- Uses **Azure Private Link** technology under the hood.
- Requires **Private DNS Zone** configuration so that the service's public DNS name (e.g., `mystorageaccount.blob.core.windows.net`) resolves to the private IP instead of the public IP.
  - Example: `mystorageaccount.blob.core.windows.net` → `10.0.3.4` (private IP) instead of the public IP.
  - Without DNS configuration, clients may still reach the public IP.
- **Private Link Service** lets you create your own private endpoint for a custom service you publish, not just Microsoft services.
- Private Endpoints work across VNet peering, VPN, and ExpressRoute — much broader than Service Endpoints.

**Key difference:** Service Endpoint = traffic path changes (no public internet), but public IP still exists. Private Endpoint = private IP in your VNet, public access can be fully disabled.

***

## 2026 Portal Walkthrough — Step by Step

**Create a Private Endpoint for Azure Storage and configure DNS:**

1. Go to **portal.azure.com** → search your **Storage account** → open it.
2. Left menu → **Security + networking** → **Networking** → **Firewalls and virtual networks**.
3. Set **Public network access** to `Disabled` → **Save**. (Do this AFTER the private endpoint is ready.)
4. Left menu → **Private endpoint connections** → click **+ Private endpoint**.
5. **Basics tab**: Subscription, Resource Group, Name (e.g., `pe-storage-prod`), Region → **Next**.
6. **Resource tab**: Resource type = `Microsoft.Storage/storageAccounts`, Resource = your storage account, Target sub-resource = `blob` → **Next**.
7. **Virtual Network tab**: Select VNet and subnet → under **Private DNS integration** → set **Integrate with private DNS zone** to `Yes` → the zone `privatelink.blob.core.windows.net` is created automatically → **Next** → **Review + create** → **Create**.
8. After creation, go to the Private DNS Zone `privatelink.blob.core.windows.net` → confirm an A record exists pointing to the private IP.

**Create a Service Endpoint (alternative, lighter approach):**

1. Open your **Virtual Network** → left menu → **Subnets** → click the subnet name.
2. Under **Service endpoints** → click **+ Add** → select `Microsoft.Storage` → **Save**.
3. Go to your **Storage account** → **Networking** → **Firewalls and virtual networks**.
4. **Public network access**: `Enabled from selected virtual networks and IP addresses`.
5. Click **+ Add existing virtual network** → select VNet and subnet → **Add** → **Save**.

**CLI:**
```bash
# Disable public access on storage account
az storage account update \
  --resource-group rg-prod \
  --name mystorageaccount \
  --default-action Deny \
  --public-network-access Disabled

# Create private endpoint for storage (blob)
az network private-endpoint create \
  --resource-group rg-prod \
  --name pe-storage-prod \
  --location eastus \
  --vnet-name vnet-prod \
  --subnet subnet-data \
  --private-connection-resource-id "/subscriptions/<sub-id>/resourceGroups/rg-prod/providers/Microsoft.Storage/storageAccounts/mystorageaccount" \
  --group-id blob \
  --connection-name conn-storage-prod

# Create private DNS zone and link to VNet
az network private-dns zone create \
  --resource-group rg-prod \
  --name "privatelink.blob.core.windows.net"

az network private-dns link vnet create \
  --resource-group rg-prod \
  --zone-name "privatelink.blob.core.windows.net" \
  --name dns-link-vnet-prod \
  --virtual-network vnet-prod \
  --registration-enabled false
```

**PowerShell:**
```powershell
# Get VNet subnet
$vnet = Get-AzVirtualNetwork -Name "vnet-prod" -ResourceGroupName "rg-prod"
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "subnet-data" -VirtualNetwork $vnet

# Define private link service connection
$plsConnection = New-AzPrivateLinkServiceConnection `
  -Name "conn-storage-prod" `
  -PrivateLinkServiceId "/subscriptions/<sub-id>/resourceGroups/rg-prod/providers/Microsoft.Storage/storageAccounts/mystorageaccount" `
  -GroupId "blob"

# Create private endpoint
New-AzPrivateEndpoint `
  -ResourceGroupName "rg-prod" `
  -Name "pe-storage-prod" `
  -Location "eastus" `
  -Subnet $subnet `
  -PrivateLinkServiceConnection $plsConnection
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/private-link/

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Public network access: Disabled | Completely blocks all internet traffic to the PaaS service | Always set this after creating a private endpoint in production |
| Target sub-resource (Group ID) | Specifies which part of the service to expose privately (e.g., blob, table, file, queue) | Select the sub-resource matching your workload's data plane |
| Private DNS Zone integration | Auto-creates DNS A record so the service's FQDN resolves to the private IP | Always enable — without this, DNS resolves to the public IP |
| Service Endpoint on subnet | Routes subnet traffic to PaaS over Azure backbone | Use when private IP is not required and on-premises connectivity is not needed |
| Private Link Service | Lets you expose YOUR custom service through a private endpoint | Use when you want customers or partners to connect privately to your own service |
| Subnet Network Policies | Must be disabled on the subnet to deploy a private endpoint (`--disable-private-endpoint-network-policies true`) | Required before creating private endpoints in a subnet |

***

## Comparison Table

| Feature | Service Endpoint | Private Endpoint |
|---|---|---|
| Private IP in VNet | No — public IP still used | Yes — dedicated NIC with private IP |
| Public access disabled | No — service still publicly reachable (you restrict via firewall) | Yes — can fully disable public access |
| On-premises (VPN/ER) support | No — only works from VNet subnets | Yes — works from on-prem via VPN or ExpressRoute |
| DNS configuration required | No | Yes — Private DNS Zone must resolve FQDN to private IP |
| Works across VNet peering | Limited | Yes |
| Cost | Free | Small hourly charge + data processing fee |
| Best for | Quick, lightweight PaaS access control | Strict private-only access, compliance requirements |

***

## What Happens If You Skip This?

Without Private Endpoints or Service Endpoints, your PaaS services are accessible from anywhere on the public internet. Even with strong authentication, public endpoints are subject to brute force, credential theft, and exposure. Regulatory frameworks like PCI-DSS, HIPAA, and SOC 2 often require that sensitive data services not be reachable from the public internet. Skipping private networking means failing these audits.

***

## AZ-500 Exam Section

- **Trigger Words:** "private endpoint", "service endpoint", "private link", "disable public access", "VNet integration", "private IP", "on-premises access to PaaS", "DNS configuration"
- **Common Traps:**
  - Service endpoint ≠ private endpoint. Service endpoint does NOT give a private IP. The service still has a public IP. This is the most tested distinction.
  - Without configuring a **Private DNS Zone**, DNS for the service still resolves to the public IP — clients may bypass the private endpoint even though it exists.
  - Private endpoints require **subnet network policies to be disabled** before deployment.
  - Service endpoints do NOT work for on-premises traffic. Private endpoints do.
  - Disabling public network access before a private endpoint is ready will break connectivity.
- **How to Tell Apart:**
  - "Private IP for a PaaS service" → Private Endpoint.
  - "Route VNet traffic over Azure backbone without private IP" → Service Endpoint.
  - "On-premises to Azure SQL privately" → Private Endpoint (Service Endpoint does not support this).
  - "Publish your own service privately to other VNets" → Private Link Service.
- **Likely Question Formats:**
  - "You need Azure SQL to be accessible only from inside your VNet. No public access. What do you use?" → Private Endpoint + disable public network access.
  - "DNS for your storage private endpoint still resolves to the public IP. What is the fix?" → Configure a Private DNS Zone.
  - "You need to allow on-premises servers to reach Azure Storage privately over ExpressRoute." → Private Endpoint (not Service Endpoint).
- **Memorization Tip:** Service Endpoint is a one-way toll road — your subnet's traffic takes a private highway to the PaaS service, but the service still has a front door facing the street. Private Endpoint puts the PaaS service inside your building — no street-facing door at all.

***

## Pocket Summary (5-Year Refresh)

- Service Endpoint: keeps traffic on Azure backbone. Service still has a public IP. On-premises cannot use it.
- Private Endpoint: gives PaaS a private IP in your VNet. Public access can be fully disabled. Works on-prem too.
- Private DNS Zone must be configured so the service FQDN resolves to the private IP.
- Private Link is the underlying technology for Private Endpoints and Private Link Service.
- In strict security scenarios, always use Private Endpoint and disable public network access.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
