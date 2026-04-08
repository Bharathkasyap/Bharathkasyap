# Azure App Service Security

> 📌 AZ-500 Exam Objective: Plan and implement security for compute and container services
> 🏷️ Domain: 3 — Secure Compute, Storage, and Databases | Weight: 20–25%

***

## What Is This?

Azure App Service hosts web applications, REST APIs, and mobile backends without managing servers. App Service security covers who can log in to your app, how the app connects to other Azure services, how the app receives traffic, and how App Service itself is protected from threats.

***

## Why Does This Exist?

Web apps are the most attacked layer in any cloud environment. Without proper authentication, TLS enforcement, and network controls, your app can be the entry point for credential theft, data leaks, and lateral movement into your Azure environment.

***

## Who Actually Needs This? (Real-World Context)

A law firm in New York builds a client portal on Azure App Service. Only employees with a company Microsoft Entra ID account should be able to log in. The App Service connects to an Azure SQL database to pull case files. The IT team enables Easy Auth with Entra ID so the portal requires a valid corporate login. They assign the App Service a managed identity so it can read database credentials from Key Vault — no passwords in application settings. They restrict the App Service to only accept traffic through a private endpoint so the portal is not reachable from the public internet.

***

## How It Works (Pure Concept, No Portal)

**Easy Auth (Built-In Authentication)**
App Service has a built-in authentication module called Easy Auth. It handles the full OAuth2/OpenID Connect login flow for you — no code changes to your application. Supported identity providers: Microsoft Entra ID, Google, Facebook, Twitter, GitHub, and any OpenID Connect provider. When a user hits your app, Easy Auth redirects them to log in. Only authenticated users reach your application code.

**Managed Identity for App Service**
A managed identity is an Azure-managed service principal tied to your App Service. It gets an automatically rotated token. Your app uses this identity to authenticate to Key Vault, Storage, SQL, or any Azure service that supports Azure AD auth. You never store passwords or connection strings in application settings.

Two types:
- **System-assigned**: tied to the app lifecycle; deleted when the app is deleted.
- **User-assigned**: exists independently; can be shared across multiple apps.

**App Service Environment (ASE)**
An ASE is an App Service deployment that runs entirely inside your own VNet on dedicated infrastructure. It provides full network isolation. There is no shared multi-tenant infrastructure. ASE is the highest isolation tier for App Service — used for financial services, healthcare, and government workloads where shared compute is not allowed.

**VNet Integration**
VNet integration lets an App Service make *outbound* calls to resources inside your VNet (databases, internal APIs, Key Vault with private endpoint). It is **not** the same as receiving inbound traffic from the VNet. VNet integration = outbound only.

**Private Endpoints for App Service**
A private endpoint gives your App Service a private IP address on your VNet. *Inbound* traffic to the app comes through that private IP. The public internet endpoint is blocked. Private endpoint = inbound traffic control.

**TLS / HTTPS Enforcement**
You can force all HTTP traffic to redirect to HTTPS. You can also set the minimum TLS version (1.2 is the current standard; TLS 1.0 and 1.1 are deprecated). Custom domains require you to upload a TLS certificate or use App Service Managed Certificates (free, auto-renewed for custom domains).

**CORS Configuration**
Cross-Origin Resource Sharing (CORS) tells browsers which other domains can call your API. Misconfigured CORS with wildcard `*` allows any website on the internet to make authenticated calls to your API on behalf of a logged-in user.

**Microsoft Defender for App Service**
Defender for App Service monitors your app for common web attack patterns: SQL injection, XSS, command injection, dangling DNS entries, malicious IP connections. It uses threat intelligence from millions of Azure tenants to detect attacks in real time.

***

## 2026 Portal Walkthrough — Step by Step

**Goal: Enable Easy Auth with Entra ID, enforce HTTPS, and assign a managed identity**

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for **App Services** and open your app.
3. **Enable Easy Auth:**
   - In the left menu, under **Settings**, click **Authentication**.
   - Click **Add identity provider**.
   - Select **Microsoft** (Entra ID).
   - Under **App registration type**, choose **Create new app registration** (or pick an existing one).
   - Set **Unauthenticated requests** to **HTTP 302 Redirect: Recommended for websites**.
   - Click **Add**. Easy Auth is now active — any unauthenticated request redirects to the Entra ID login page.
4. **Enforce HTTPS only:**
   - In the left menu, under **Settings**, click **Configuration** → **General settings** tab.
   - Set **HTTPS Only** to **On**.
   - Set **Minimum TLS Version** to **1.2**.
   - Click **Save**.
5. **Assign a system-assigned managed identity:**
   - In the left menu, under **Settings**, click **Identity**.
   - On the **System assigned** tab, toggle **Status** to **On**.
   - Click **Save**. Azure creates a service principal for this app.
6. Go to your Key Vault → **Access control (IAM)** → **+ Add role assignment**.
7. Assign the `Key Vault Secrets User` role to the App Service's managed identity.
8. The app can now retrieve secrets from Key Vault using the managed identity — no stored passwords.

**CLI equivalent:**
```bash
# Enable Easy Auth with Entra ID
az webapp auth update \
  --resource-group myRG \
  --name myApp \
  --enabled true \
  --action LoginWithAzureActiveDirectory

# Enforce HTTPS
az webapp update \
  --resource-group myRG \
  --name myApp \
  --https-only true

# Assign system-assigned managed identity
az webapp identity assign \
  --resource-group myRG \
  --name myApp
```

**PowerShell equivalent:**
```powershell
# Enforce HTTPS and minimum TLS
Set-AzWebApp `
  -ResourceGroupName "myRG" `
  -Name "myApp" `
  -HttpsOnly $true

# Assign system-assigned managed identity
Set-AzWebApp `
  -ResourceGroupName "myRG" `
  -Name "myApp" `
  -AssignIdentity $true
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/app-service/overview-security

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Easy Auth | Handles authentication before requests reach app code | Employee portals, partner-facing apps; no-code auth layer |
| HTTPS only | Redirects HTTP to HTTPS; blocks unencrypted traffic | Always; TLS 1.2 minimum is a compliance requirement |
| System-assigned managed identity | Gives the app an auto-managed Azure identity | When the app needs to access other Azure services securely |
| User-assigned managed identity | Shared identity across multiple apps | When multiple apps need the same permissions |
| VNet integration | Routes outbound app traffic through your VNet | When the app needs to reach private resources (SQL, Key Vault) |
| Private endpoint | Assigns the app a private IP for inbound traffic | When the app should not be reachable from the public internet |
| App Service Environment (ASE) | Fully isolated deployment in your own VNet | High-compliance workloads needing dedicated infrastructure |
| Defender for App Service | Real-time web attack detection and alerting | All production apps; especially public-facing APIs |
| CORS settings | Controls which domains can call your API from a browser | APIs consumed by known front-end origins; never use `*` in production |

***

## Comparison Table

| Feature | VNet Integration | Private Endpoint |
|---|---|---|
| Traffic direction | Outbound (app → VNet resources) | Inbound (VNet/internet → app) |
| Public endpoint blocked? | No | Yes (when public access disabled) |
| Use case | App calls private SQL or Key Vault | App should not be accessible from internet |
| Can be combined? | Yes — use both together for full isolation | Yes |

| Authentication Option | Complexity | Best For |
|---|---|---|
| Easy Auth (Entra ID) | Low — no code changes | Internal employee apps, single-org scenarios |
| Custom auth in code | High — you manage token validation | Multi-tenant SaaS, complex auth flows |
| Managed identity (app-to-service) | Low — no credentials to manage | App-to-Azure-service authentication |

***

## What Happens If You Skip This?

Without Easy Auth or custom authentication, your app is open to anyone with the URL. Without HTTPS enforcement, credentials and session tokens travel over unencrypted HTTP. Without managed identity, connection strings and API keys end up in application settings or hardcoded in source code — a common cause of secrets leaking on GitHub. Without Defender for App Service, a SQL injection attack can go undetected for months.

***

## AZ-500 Exam Section

- **Trigger Words:** "Easy Auth", "built-in authentication", "App Service authentication", "HTTPS only", "ASE", "App Service Environment", "managed identity App Service", "VNet integration", "private endpoint App Service"
- **Common Traps:**
  - VNet integration is **outbound** only. Private endpoint is **inbound** only. The exam frequently tests this distinction.
  - Easy Auth does not require code changes — it is a platform-level module sitting in front of your app.
  - ASE provides full network isolation on dedicated hardware. Regular App Service in a VNet (via VNet integration) still runs on shared multi-tenant infrastructure.
- **How to Tell Apart:**
  - "app needs to call an internal database" → VNet integration (outbound)
  - "app should not be reachable from the internet" → private endpoint (inbound)
  - "no-code login with Entra ID" → Easy Auth
  - "app needs to read Key Vault secrets without passwords" → managed identity
  - "dedicated, fully isolated hosting environment" → App Service Environment (ASE)
- **Likely Question Formats:**
  - You need to ensure only Entra ID authenticated users can access your App Service. What is the simplest solution? → Easy Auth with Entra ID provider
  - An App Service needs to connect to a private Azure SQL database. What feature enables this? → VNet integration
  - What is the difference between VNet integration and private endpoint for App Service? → VNet integration = outbound; private endpoint = inbound
- **Memorization Tip:** **VNet = eXit traffic (outbound)**. **Private Endpoint = Entry traffic (inbound)**. VNet integration lets the app EXIT to your private network. Private endpoint is the ENTRY point for your private network.

***

## Pocket Summary (5-Year Refresh)

- Easy Auth is a no-code authentication layer; it redirects users to Entra ID (or other providers) before any request reaches your app code.
- Managed identity eliminates stored passwords; assign it to the App Service and grant it roles on Key Vault, Storage, or SQL.
- VNet integration routes **outbound** traffic from the app into your VNet; private endpoints control **inbound** access — these two are complementary, not interchangeable.
- App Service Environment (ASE) is fully isolated hosting on dedicated infrastructure for high-compliance scenarios.
- Always enforce HTTPS-only and TLS 1.2 minimum; enable Defender for App Service on all production apps.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/app-service/overview-security
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
