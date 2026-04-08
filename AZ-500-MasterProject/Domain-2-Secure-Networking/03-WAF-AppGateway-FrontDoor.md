# Web Application Firewall with Application Gateway and Azure Front Door

> 📌 AZ-500 Exam Objective: Plan and implement security for Azure services
> 🏷️ Domain: 2 — Secure Networking | Weight: 20–25%

***

## What Is This?

A Web Application Firewall (WAF) protects web applications from common internet attacks like SQL injection and cross-site scripting (XSS). Azure offers WAF in two places: Application Gateway (for apps in one region) and Azure Front Door (for global apps). WAF sits in front of your app and inspects every HTTP/HTTPS request before it reaches your servers.

***

## Why Does This Exist?

Firewalls and NSGs block traffic by IP or port. They cannot tell the difference between a normal HTTP request and one that contains a SQL injection attack — both come in on port 443. WAF understands the content of web requests. It reads the URL, headers, and body to detect attack patterns. Without WAF, a single crafted web request can expose your database or steal session cookies.

***

## Who Actually Needs This? (Real-World Context)

An e-commerce company runs a shopping website on Azure. During a sale event, attackers attempt SQL injection attacks on the checkout page to steal customer credit card data. The security team enables WAF on their Application Gateway in Prevention mode using the OWASP CRS 3.2 rule set. The WAF blocks the malicious requests automatically. The development team never had to patch the application code.

***

## How It Works (Pure Concept, No Portal)

- **WAF** is a set of rules that inspect HTTP/HTTPS request content — URLs, headers, cookies, query strings, and body.
- **OWASP Core Rule Set (CRS)** is the industry standard ruleset that covers the OWASP Top 10 threats. CRS 3.2 is the current recommended version in Azure WAF. CRS 4.0 is also available.
- **Two WAF operating modes:**
  - **Detection mode** — logs matching requests but does NOT block them. Use this first to see what would be blocked.
  - **Prevention mode** — actively blocks requests that match attack rules. Returns HTTP 403 to the attacker.
- **Managed rules** are the OWASP CRS rules maintained by Microsoft. They are updated automatically.
- **Custom rules** are rules you write yourself. Examples: block requests from a specific country, block a specific IP range, rate-limit a specific URL path.
- **Application Gateway** is a regional (single-region) Layer 7 load balancer. It routes HTTP/HTTPS traffic to backend VMs or app services. WAF is an add-on that runs on the **WAF_v2** SKU.
  - Supports: cookie-based session affinity, URL-based routing, SSL termination, autoscaling.
  - WAF lives between the internet and your backend pool.
- **Azure Front Door** is a global (multi-region) CDN, load balancer, and WAF combined. It routes users to the nearest healthy region.
  - WAF on Front Door protects globally distributed apps from the Microsoft edge network, before traffic even reaches Azure data centers.
  - Supports: geo-filtering, bot protection, rate limiting, custom rules.
- **WAF Policy** is a standalone resource containing rules and settings. One WAF Policy can be linked to multiple Application Gateways or Front Door instances.
- **Exclusions** allow you to skip specific WAF rules for specific request fields. Use when a legitimate app request triggers a false positive.

***

## 2026 Portal Walkthrough — Step by Step

**Create Application Gateway with WAF_v2 and enable WAF in Prevention mode:**

1. Go to **portal.azure.com** → search **"Application gateways"** → click **Create**.
2. **Basics tab:**
   - Subscription, Resource Group, Name (e.g., `appgw-web-prod`)
   - **Tier**: Select `WAF V2`
   - **Autoscaling**: Enabled, Min instances: `1`, Max: `10`
   - Region, Availability Zone → **Next**.
3. **Frontends tab:**
   - **Frontend IP type**: `Public`
   - Create or select a Public IP address → **Next**.
4. **Backends tab:**
   - Click **Add a backend pool** → Name: `bp-web-servers` → Add your backend VMs or app service → **Add** → **Next**.
5. **Configuration tab:**
   - Click **+ Add a routing rule** → Name: `rule-https`
   - **Listener**: Port `443`, Protocol `HTTPS`, upload your SSL certificate
   - **Backend targets**: Select `bp-web-servers`, create an HTTP setting (port 80, HTTP, cookie-based affinity off)
   → **Add** → **Next**.
6. **WAF tab:**
   - **WAF Policy**: Click **Create new** → Name: `wafpol-web-prod`
   - **Policy mode**: `Prevention`
   - **Managed rules**: `OWASP_3.2` selected by default
   → **Next** → **Review + create** → **Create**.

**Switch an existing WAF Policy to Prevention mode:**
1. Search **"Web Application Firewall policies"** → open `wafpol-web-prod`.
2. Left menu → **Policy settings** → **Mode**: change to `Prevention` → **Save**.

**Add a custom WAF rule to block a country:**
1. Open WAF Policy → left menu → **Custom rules** → **+ Add custom rule**.
2. Name: `block-country-XX`, Priority: `5`, Rule type: `Match Rule`.
3. Condition: **Match variable** = `RemoteAddr`, **Operation** = `GeoMatch`, **Values** = select country code.
4. **Action**: `Block` → **Add** → **Save**.

**CLI:**
```bash
# Create WAF policy
az network application-gateway waf-policy create \
  --resource-group rg-prod \
  --name wafpol-web-prod

# Create Application Gateway with WAF_v2 SKU
az network application-gateway create \
  --resource-group rg-prod \
  --name appgw-web-prod \
  --location eastus \
  --sku WAF_v2 \
  --capacity 2 \
  --vnet-name vnet-prod \
  --subnet subnet-appgw \
  --public-ip-address appgw-pip \
  --waf-policy wafpol-web-prod

# Set WAF config to Prevention mode (classic approach)
az network application-gateway waf-config set \
  --gateway-name appgw-web-prod \
  --resource-group rg-prod \
  --enabled true \
  --firewall-mode Prevention \
  --rule-set-version 3.2
```

**PowerShell:**
```powershell
# Get VNet and subnet references
$vnet = Get-AzVirtualNetwork -Name "vnet-prod" -ResourceGroupName "rg-prod"
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "subnet-appgw" -VirtualNetwork $vnet
$pip = Get-AzPublicIpAddress -Name "appgw-pip" -ResourceGroupName "rg-prod"

# Create IP config and frontend
$gipconfig = New-AzApplicationGatewayIPConfiguration -Name "appgw-ipconfig" -Subnet $subnet
$fip = New-AzApplicationGatewayFrontendIPConfig -Name "appgw-fip" -PublicIPAddress $pip
$fport = New-AzApplicationGatewayFrontendPort -Name "port-443" -Port 443

# Create WAF config
$wafConfig = New-AzApplicationGatewayWebApplicationFirewallConfiguration `
  -Enabled $true `
  -FirewallMode Prevention `
  -RuleSetType OWASP `
  -RuleSetVersion 3.2

# Create Application Gateway
New-AzApplicationGateway `
  -Name "appgw-web-prod" `
  -ResourceGroupName "rg-prod" `
  -Location "eastus" `
  -Sku (New-AzApplicationGatewaySku -Name WAF_v2 -Tier WAF_v2 -Capacity 2) `
  -GatewayIPConfigurations $gipconfig `
  -FrontendIPConfigurations $fip `
  -FrontendPorts $fport `
  -WebApplicationFirewallConfig $wafConfig
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/web-application-firewall/

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| WAF Mode: Detection | Logs attacks but does not block them | Use first when deploying WAF to baseline false positives |
| WAF Mode: Prevention | Blocks matched attack requests with HTTP 403 | Use in production after confirming no false positives |
| OWASP CRS 3.2 | Industry-standard rule set covering SQL injection, XSS, RFI, and more | Always enable — it is the foundation of WAF protection |
| Custom Rules | Rules you write: IP block, geo-block, rate limiting, string match | Use to handle business-specific threats not covered by CRS |
| Managed Rules | CRS rules maintained and updated by Microsoft | Enable and keep on — do not disable rule groups unless necessary |
| Exclusions | Skip a specific WAF rule for a specific request field | Use when a legitimate app request is being blocked (false positive) |
| WAF_v2 SKU | Required for WAF on Application Gateway | Must select WAF_v2 tier — Standard_v2 does NOT include WAF |
| Bot Protection Rule Set | Identifies and blocks known bad bots | Enable on Front Door WAF for bot mitigation |

***

## Comparison Table

| Feature | Application Gateway WAF | Azure Front Door WAF |
|---|---|---|
| Scope | Regional (one Azure region) | Global (all edge PoPs worldwide) |
| Best for | Apps in a single region | Global multi-region or CDN-backed apps |
| OWASP CRS | Yes | Yes |
| Custom rules | Yes | Yes |
| Geo-filtering | Yes | Yes |
| Bot protection | Yes | Yes |
| SSL termination | Yes | Yes (at edge) |
| Layer | L7 | L7 |

***

## What Happens If You Skip This?

Without WAF, a single HTTP request containing `' OR 1=1 --` can dump your entire database through a vulnerable login page. Without WAF, cross-site scripting attacks can steal users' session tokens and impersonate them. Standard firewalls and NSGs cannot prevent these attacks because the malicious content travels on an allowed port (443). OWASP lists these as the top web application vulnerabilities every year.

***

## AZ-500 Exam Section

- **Trigger Words:** "OWASP", "SQL injection", "XSS protection", "WAF", "Prevention mode", "Application Gateway", "Front Door", "CRS", "custom rules", "geo-filtering"
- **Common Traps:**
  - WAF protects HTTP/HTTPS traffic only. It does NOT protect raw TCP, gRPC, or other API protocols by default.
  - Detection mode does NOT block anything. Exam questions may try to trick you into thinking it provides protection.
  - WAF_v2 SKU is required. If you see `Standard_v2` in an answer, it does NOT include WAF.
  - Application Gateway WAF is regional. Azure Front Door WAF is global. Match the scenario to the right product.
  - Exclusions should be narrow and specific — do not exclude entire rule groups to fix one false positive.
- **How to Tell Apart:**
  - "Regional app, web attacks" → Application Gateway WAF.
  - "Global app, users worldwide, CDN" → Azure Front Door WAF.
  - "Detect only, no blocking" → Detection mode.
  - "Block attacks" → Prevention mode.
- **Likely Question Formats:**
  - "You deploy WAF but attacks still reach your app. What is wrong?" → WAF is in Detection mode, not Prevention mode.
  - "You need to protect a globally distributed app from SQL injection." → Azure Front Door with WAF.
  - "A legitimate request is being blocked by WAF. How do you fix this without disabling the WAF?" → Add a WAF exclusion for the specific rule and field.
- **Memorization Tip:** WAF reads the CONTENT of web requests. NSG reads the IP/port label on the envelope. WAF opens the envelope and checks what is inside.

***

## Pocket Summary (5-Year Refresh)

- WAF inspects HTTP/HTTPS request content to block web attacks like SQL injection and XSS.
- OWASP CRS is the standard rule set. CRS 3.2 is the current recommended version.
- Two modes: Detection (log only) and Prevention (block + log). Use Prevention in production.
- Application Gateway WAF = regional. Azure Front Door WAF = global.
- Custom rules add business-specific logic: block countries, rate-limit endpoints, block specific IPs.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/web-application-firewall/overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
