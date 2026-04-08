# Azure API Management Security

> 📌 AZ-500 Exam Objective: Plan and implement security for Azure services
> 🏷️ Domain: 3 — Secure Compute, Storage, and Databases | Weight: 20–25%

***

## What Is This?

Azure API Management (APIM) is a gateway that sits in front of your backend APIs. It controls who can call your APIs, how many times they can call them, and what happens to the request before it reaches the backend. Think of APIM as a bouncer, traffic cop, and translator all in one — standing between the internet and your internal services.

***

## Why Does This Exist?

Without a gateway, every API is exposed directly to clients. Rate limiting, authentication, and monitoring must be built into each API individually — an inconsistent mess. APIM centralizes all of these controls in one place. You add security once in the gateway and it applies to every API automatically.

***

## Who Actually Needs This? (Real-World Context)

A healthcare company in Toronto publishes several internal APIs for their patient management system and exposes them to partner hospitals. Each partner hospital has its own subscription key, with a rate limit of 100 requests per minute. The APIM gateway validates a JWT token on every call to confirm the caller is an authorized partner app registered in Entra ID. Backend API connections use the APIM managed identity so no API keys are stored in APIM policy files. Microsoft Defender for APIs alerts the security team if a partner hospital's subscription starts downloading an unusually large number of patient records — a possible data exfiltration indicator.

***

## How It Works (Pure Concept, No Portal)

**APIM Authentication Methods**

1. **Subscription keys**: A simple secret string (`Ocp-Apim-Subscription-Key` header or `subscription-key` query parameter). Every API product in APIM can require a subscription key. Easy to set up but low security — the key is a static secret. If it leaks, anyone can call your API.

2. **OAuth 2.0 with JWT validation**: Clients get a bearer token from an OAuth 2.0 server (like Entra ID). APIM validates the JWT signature, expiry, audience, and issuer using a policy. This is the secure enterprise pattern.

3. **Client certificates (mutual TLS)**: Clients present a certificate when connecting to APIM. APIM validates the certificate against a trusted CA or a specific certificate thumbprint. Used for machine-to-machine API calls where certificate management is feasible.

4. **Managed identity for backend**: APIM uses its managed identity to get an Entra ID token and passes it to the backend API. No backend credentials stored in APIM. The backend API validates the Entra ID token from APIM.

**APIM Policies**
Policies are XML-based rules that transform and control API traffic. They run in four phases:
- **Inbound**: Runs before the request reaches the backend. Authentication checks, rate limiting, IP filtering, request transformation.
- **Backend**: Modifies how APIM connects to the backend.
- **Outbound**: Modifies the response before returning it to the client. Response transformation, sensitive header removal.
- **On-error**: Handles errors.

Key security policies:
- `validate-jwt`: Validates a JWT token's signature, expiry, audience, and required claims.
- `rate-limit-by-key`: Limits calls per subscription or per IP per time window.
- `quota-by-key`: Hard cap on total calls per subscription in a time period.
- `ip-filter`: Allow or deny specific IP ranges.
- `check-header`: Requires specific request headers to be present.
- `set-backend-service`: Routes requests to different backends based on conditions.
- `authentication-managed-identity`: Acquires a token from APIM's managed identity to call the backend.

**Rate Limiting and Throttling**
- `rate-limit`: Limits calls per subscription per second or per minute. Client gets `429 Too Many Requests` when exceeded.
- `rate-limit-by-key`: More flexible — limit per any key (IP address, user claim, etc.).
- `quota-by-key`: Monthly or weekly total call count cap per subscription. Useful for billing enforcement on partner APIs.

**APIM VNet Integration (Internal vs External)**
APIM can be deployed inside a VNet in two modes:
- **External mode**: APIM gateway is reachable from the internet AND from inside the VNet. Backends can be private.
- **Internal mode**: APIM gateway is only reachable from inside the VNet. No public internet access to the gateway. You need an Application Gateway or Azure Front Door in front of APIM to expose APIs to the internet — this is a common enterprise pattern for layered security.

**Private Endpoint for APIM**
Incoming traffic to APIM can use a private endpoint (for the Developer and Premium tiers) so API calls come in through a private VNet IP. Different from VNet integration — private endpoint is for inbound traffic.

**Microsoft Defender for APIs**
Defender for APIs (integrated with Defender for Cloud) discovers all APIs published through APIM, monitors call patterns, and alerts on:
- Unusual spike in calls to a specific endpoint.
- Calls from unexpected geographies.
- API enumeration (probing endpoints systematically).
- Attempts to access undocumented parameters.

Defender for APIs helps surface shadow APIs — APIs you forgot were published — and data exfiltration patterns.

***

## 2026 Portal Walkthrough — Step by Step

**Goal: Create an APIM instance, add JWT validation policy, configure subscription key requirement, put APIM in a VNet**

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for **API Management services** and click **+ Create**.
3. Fill in subscription, resource group, name (globally unique), region, organization name, and admin email.
4. Under **Pricing tier**, select **Developer** (for non-production) or **Premium** (for VNet and multiple regions).
5. Click **Review + create** → **Create**. (Deployment takes 30–45 minutes.)
6. After deployment, open the APIM instance.
7. **Require subscription key on an API:**
   - Go to **APIs** → select your API → click the **Settings** tab.
   - Confirm **Subscription required** is checked.
8. **Add JWT validation policy:**
   - On the API, go to the **Design** tab.
   - Click **All operations** → click the **</>** (policy) icon on the **Inbound** phase.
   - Add the following policy inside `<inbound>`:
   ```xml
   <validate-jwt header-name="Authorization" failed-validation-httpcode="401"
                 failed-validation-error-message="Unauthorized. JWT token is invalid or missing.">
     <openid-config url="https://login.microsoftonline.com/{tenant-id}/v2.0/.well-known/openid-configuration" />
     <required-claims>
       <claim name="aud">
         <value>api://{your-app-id}</value>
       </claim>
     </required-claims>
   </validate-jwt>
   ```
   - Click **Save**.
9. **Put APIM in a VNet (Internal mode — Premium tier required):**
   - Go to **Deployment + infrastructure** → **Network**.
   - Select **Virtual network** → **Internal**.
   - Select your VNet and subnet.
   - Click **Apply**.
10. **Enable Defender for APIs:**
    - Go to **Microsoft Defender for Cloud** → **Environment settings** → your subscription → enable **APIs** under Defender plans.

**CLI equivalent:**
```bash
# Create APIM instance
az apim create \
  --resource-group myRG \
  --name myAPIM \
  --publisher-name "My Company" \
  --publisher-email "admin@mycompany.com" \
  --sku-name Developer

# Enable VNet integration (requires Premium SKU)
az apim update \
  --resource-group myRG \
  --name myAPIM \
  --virtual-network Internal \
  --vnet-name myVNet \
  --subnet myAPIMSubnet
```

**PowerShell equivalent:**
```powershell
# Create APIM instance
New-AzApiManagement `
  -ResourceGroupName "myRG" `
  -Name "myAPIM" `
  -Location "eastus" `
  -Organization "My Company" `
  -AdminEmail "admin@mycompany.com" `
  -Sku "Developer"
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/api-management/api-management-security-controls-policy

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Subscription key required | Forces callers to include a valid subscription key | All APIs by default; easy baseline authentication |
| `validate-jwt` policy | Validates OAuth2 bearer tokens before forwarding to backend | Enterprise APIs; OAuth2/Entra ID authentication |
| `rate-limit-by-key` policy | Limits API calls per key per time window | Prevent abuse, protect backend from overload |
| `quota-by-key` policy | Hard cap on total calls per subscription in a billing period | Partner API billing enforcement |
| `ip-filter` policy | Allow or deny by IP address or CIDR range | Lock API to known partner IP ranges |
| Managed identity (APIM) | APIM uses its own Entra ID identity to call backends | Replace backend credentials with token-based auth |
| VNet Internal mode | Hides APIM gateway from internet; only accessible from VNet | Production enterprise APIs not meant for public access |
| VNet External mode | Gateway accessible from internet AND VNet backends | Internet-facing APIs with private backend services |
| Private endpoint (APIM) | Assigns APIM a private VNet IP for inbound traffic | Specific consumer networks calling APIM over private link |
| Defender for APIs | Discovers all published APIs; detects anomalous call patterns | All production APIM instances with sensitive data APIs |

***

## Comparison Table

| Authentication Method | Security Level | Best For |
|---|---|---|
| Subscription key only | Low — static secret, no identity | Internal/dev APIs; low-sensitivity endpoints |
| Subscription key + IP filter | Medium — adds network restriction | Partner APIs from known IP ranges |
| Client certificate | High — PKI-based mutual auth | Machine-to-machine; partners with managed certificates |
| JWT validation (OAuth2/Entra ID) | Highest — time-limited, revocable, identity-linked | All public and partner APIs; enterprise standard |

| VNet Mode | Internet Access to Gateway | Backend Access | Requires |
|---|---|---|---|
| No VNet (default) | Yes | Internet only | Nothing extra |
| External | Yes | Public + private VNet | VNet with APIM subnet |
| Internal | No (VNet only) | Private VNet only | VNet + Application Gateway for internet exposure |

***

## What Happens If You Skip This?

Without APIM security policies, every backend API is exposed to direct internet traffic with no rate limiting, no auth validation, and no audit trail. A stolen subscription key gives an attacker unlimited API access until someone manually rotates it. Without VNet integration, your backend services need public endpoints. Without Defender for APIs, a slow data exfiltration attack — where an attacker systematically calls an endpoint once per minute to download records — can run for weeks undetected.

***

## AZ-500 Exam Section

- **Trigger Words:** "API gateway", "JWT validation", "subscription key", "rate limiting", "APIM policy", "VNet Internal", "managed identity backend", "Defender for APIs", "OAuth2 APIM"
- **Common Traps:**
  - Subscription keys and OAuth JWT tokens are NOT equivalent in security. Subscription keys are static secrets. JWT tokens are time-limited and tied to an identity. The exam tests this distinction.
  - VNet External still allows internet access to APIM. VNet Internal does NOT — you need an Application Gateway in front of APIM to expose APIs from Internal mode.
  - APIM managed identity is for APIM calling backends — not for clients authenticating to APIM.
  - `rate-limit` is per-subscription per time window (resettable). `quota` is a total hard cap per period (does not reset mid-period).
- **How to Tell Apart:**
  - "validate a caller's JWT before forwarding to backend" → `validate-jwt` inbound policy
  - "limit a partner to 100 calls per minute" → `rate-limit-by-key` policy
  - "APIM calls a backend API without storing credentials" → managed identity + `authentication-managed-identity` policy
  - "APIM not accessible from internet" → VNet Internal mode
  - "detect unusual API call patterns" → Defender for APIs
- **Likely Question Formats:**
  - What is the most secure authentication mechanism for an enterprise API published via APIM? → JWT validation with OAuth2 (Entra ID-issued tokens)
  - A partner calls your APIM API with a subscription key. The key leaks. What is the quickest way to stop unauthorized access? → regenerate the subscription key for that subscription
  - How do you prevent APIM from being accessible from the public internet? → deploy in Internal VNet mode
  - What APIM feature allows the gateway to call a backend API using an Azure-managed identity? → managed identity authentication policy (`authentication-managed-identity`)
- **Memorization Tip:** APIM security = **SRVD** — **S**ubscription keys (basic), **R**ate limiting (protect backends), **V**alidate JWT (identity), **D**efender for APIs (monitoring). Strongest auth goes last: subscription key → client cert → JWT/OAuth.

***

## Pocket Summary (5-Year Refresh)

- Subscription keys provide basic API authentication but are static secrets — for enterprise security, use JWT validation with OAuth2 (Entra ID-issued tokens).
- APIM policies control traffic in four phases: inbound (auth, rate limiting, IP filtering), backend, outbound (response transformation), and on-error.
- APIM VNet Internal mode hides the gateway from the internet; put an Application Gateway in front to expose specific APIs publicly while keeping backends private.
- Use APIM's managed identity with the `authentication-managed-identity` policy so APIM calls backends using an Azure-managed token — no stored credentials.
- Microsoft Defender for APIs discovers all published APIs and detects anomalous patterns like sudden call spikes, geographic anomalies, and systematic endpoint enumeration.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/api-management/api-management-security-controls-policy
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
