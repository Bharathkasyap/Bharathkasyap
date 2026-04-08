# App Registrations and Service Principals

> 📌 AZ-500 Exam Objective: Register applications in Microsoft Entra ID
> 🏷️ Domain: 1 — Secure Identity and Access | Weight: 15–20%

***

## What Is This?
An app registration is the record you create in Entra ID to tell the system "this application exists and here is how it should authenticate." A service principal is the running instance of that registration inside a specific tenant. The app registration is the blueprint. The service principal is the actual object that gets permissions, signs in, and shows up in audit logs.

***

## Why Does This Exist?
Applications — not just people — need identities too. A CI/CD pipeline needs to deploy code to Azure. A web app needs to call the Microsoft Graph API. A background service needs to read from a storage account. Each of these non-human actors needs its own identity, its own defined permissions, and its own credential rotation story. App registrations and service principals provide exactly that — a formal identity for software.

***

## Who Actually Needs This? (Real-World Context)
A software company in Austin builds a SaaS HR application that reads employee data from their customers' Microsoft 365 tenants. Each customer must explicitly consent to what data the app can access. The company creates one app registration in their own tenant. When a customer signs up, Entra ID creates a service principal in the customer's tenant representing that app. The customer's admin grants consent to specific Microsoft Graph API permissions. From then on, the app can read only the data those permissions cover — in only the consented tenant.

***

## How It Works (Pure Concept, No Portal)

**The relationship between app registration and service principal:**
- You create **one app registration** in your home tenant. It has a globally unique Application ID (client ID).
- Entra ID automatically creates **one service principal** in your home tenant linked to that registration.
- When another tenant installs or consents to your app, a **new service principal** is created in that tenant. The app registration stays in yours.
- Think of the app registration as a cookie cutter. Each service principal is a cookie made from that cutter.

**Authentication credentials:**
- **Client secret** — a password string you generate with an expiration date (max 2 years). Simple but must be stored securely and rotated manually.
- **Certificate** — an X.509 certificate. Stronger than a secret. The private key is stored in Key Vault; only the public key is uploaded to the app registration.
- For production workloads, always prefer certificates over client secrets.

**API permissions (two types):**
- **Delegated permissions** — the app acts ON BEHALF OF a signed-in user. The user's own permissions limit what the app can do. Used in user-facing web and mobile apps.
- **Application permissions** — the app acts AS ITSELF with no user involved. Used in background services and daemons. These are more powerful and always require **admin consent**.

**Admin consent:**
- Required for application permissions and for any delegated permission classified as "admin only."
- A Global Administrator or Privileged Role Administrator (with the right settings) grants consent on behalf of the entire organization.
- Without admin consent, users see a consent prompt error when signing in.

**OAuth 2.0 flows you must know:**
- **Authorization Code flow** — web apps with a signed-in user. Most common for SaaS.
- **Client Credentials flow** — service-to-service with no user. Uses app permissions.
- **On-Behalf-Of (OBO) flow** — a service receives a user token and exchanges it for a token to call another service, while preserving the user's identity context.

***

## 2026 Portal Walkthrough — Step by Step

### Register an Application
```
portal.azure.com
  → Top search bar → type "Microsoft Entra ID" → select it
  → Left menu → App registrations
  → + New registration
  → Name: "HR-SaaS-App"
  → Supported account types: Accounts in any organizational directory (Multitenant)
  → Redirect URI: Web → https://app.hr-saas.com/auth/callback
  → Register
  → Note the Application (client) ID — this is your client_id
```

### Add API Permissions and Grant Admin Consent
```
portal.azure.com
  → Microsoft Entra ID → App registrations → HR-SaaS-App
  → Left menu → API permissions
  → + Add a permission
  → Microsoft Graph → Delegated permissions
  → Search "User.Read" → check it → Add permissions
  → + Add a permission → Microsoft Graph → Application permissions
  → Search "User.Read.All" → check it → Add permissions
  → Grant admin consent for [Your Org] → Yes
  → Status should show green checkmarks
```

### Create a Client Secret
```
portal.azure.com
  → Microsoft Entra ID → App registrations → HR-SaaS-App
  → Left menu → Certificates & secrets
  → Client secrets tab → + New client secret
  → Description: "prod-secret-jan2026"
  → Expires: 12 months (choose shortest acceptable for your use case)
  → Add
  → COPY THE VALUE NOW — it will never be shown again
  → Store it in Azure Key Vault immediately
```

### Upload a Certificate (Preferred over Secret)
```
portal.azure.com
  → Microsoft Entra ID → App registrations → HR-SaaS-App
  → Left menu → Certificates & secrets
  → Certificates tab → Upload certificate
  → Upload your .cer or .pem public key file
  → Description: "prod-cert-2026"
  → Add
  → Store the matching private key in Azure Key Vault
```

### CLI Equivalents
```bash
# Register an application
az ad app create \
  --display-name "HR-SaaS-App" \
  --sign-in-audience AzureADMultipleOrgs \
  --web-redirect-uris "https://app.hr-saas.com/auth/callback"

# Get the app ID
APP_ID=$(az ad app list --display-name "HR-SaaS-App" --query "[0].appId" -o tsv)

# Create a service principal for the app in your tenant
az ad sp create --id $APP_ID

# Create a client secret
az ad app credential reset \
  --id $APP_ID \
  --append \
  --display-name "prod-secret-jan2026" \
  --end-date "2027-01-01"

# Create a service principal with RBAC role (common for CI/CD)
az ad sp create-for-rbac \
  --name "sp-cicd-pipeline" \
  --role "Contributor" \
  --scopes "/subscriptions/SUB-ID/resourceGroups/rg-webapp-prod"
```

### PowerShell Equivalents
```powershell
Connect-MgGraph -Scopes "Application.ReadWrite.All"

# Create an app registration
$app = New-MgApplication `
  -DisplayName "HR-SaaS-App" `
  -SignInAudience "AzureADMultipleOrgs" `
  -Web @{ RedirectUris = @("https://app.hr-saas.com/auth/callback") }

# Create the service principal
New-MgServicePrincipal -AppId $app.AppId

# Add a client secret
$secretParam = @{
    PasswordCredential = @{
        DisplayName = "prod-secret-jan2026"
        EndDateTime = "2027-01-01T00:00:00Z"
    }
}
Add-MgApplicationPassword -ApplicationId $app.Id -BodyParameter $secretParam
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/entra/identity-platform/

***

## Key Settings Explained
| Setting | What It Does | When to Use |
|---------|-------------|-------------|
| Application (client) ID | Unique identifier for the app registration | Used in every authentication request as client_id |
| Supported account types | Controls which tenant users can sign in | Single-tenant for internal apps; multitenant for SaaS |
| Redirect URI | Where Entra ID sends the auth code/token after login | Must match exactly what the app expects |
| Client secret | Password credential for the service principal | Simple but needs rotation — use only when certs aren't feasible |
| Certificate | Asymmetric key credential — stronger than a secret | Production workloads; private key stored in Key Vault |
| Delegated permissions | App acts as the signed-in user | User-facing web and mobile apps |
| Application permissions | App acts as itself, no user | Background services and daemons — always requires admin consent |
| Admin consent | Org-wide approval for app permissions | Required before any app can use Application permissions |

***

## Comparison Table
| Concept | App Registration | Service Principal | Managed Identity |
|---------|----------------|------------------|-----------------|
| Created by | Developer manually | Auto-created with app reg | Azure automatically |
| Location | Home tenant only | Every consenting tenant | Home tenant |
| Credentials to manage | ✅ Secret or cert | Inherits from app reg | ❌ None |
| For human-facing apps | ✅ Yes | ✅ Yes | ❌ No |
| For Azure-hosted services | ✅ Yes (if needed outside Azure) | ✅ Yes | ✅ Preferred |
| Works outside Azure | ✅ Yes | ✅ Yes | ❌ No |

***

## What Happens If You Skip This?
Without formal app registrations, developers use personal accounts or shared admin credentials to authenticate applications. This means an app might have Global Admin rights because the developer who set it up is a Global Admin. When that person leaves, the app breaks. Or the credential is shared in a team chat and exfiltrated. In 2023, the Storm-0558 attack exploited a leaked Microsoft signing key — demonstrating how critical it is to manage app credentials with tight controls and certificate-based authentication rather than long-lived secrets.

***

## AZ-500 Exam Section
- **Trigger Words:** "app registration", "service principal", "client secret", "admin consent", "API permission", "delegated", "application permission", "OAuth", "OIDC", "client_id"
- **Common Traps:**
  - **App registration ≠ service principal.** The registration is the global definition. The service principal is the tenant-specific instance. In a multitenant app, one registration creates many service principals — one per consenting tenant.
  - **Application permissions always require admin consent.** Delegated permissions may not, depending on the permission's classification. If an exam asks "which requires admin consent," Application permissions is always the right answer.
  - **Client secrets expire.** If an exam mentions an app suddenly failing to authenticate, suspect an expired client secret.
  - **az ad sp create-for-rbac creates both an app registration AND a service principal** and assigns an RBAC role in one command. Useful for CI/CD setup.
- **How to Tell Apart from Similar Services:** App registrations/service principals are for non-Azure or cross-tenant app authentication. Managed identities are for Azure resources to authenticate to Azure services — simpler, no credentials, only within Azure.
- **Likely Question Formats:**
  - "A background service needs to read all users in the tenant with no signed-in user. Which permission type do you use?" → Application permission (User.Read.All), which requires admin consent.
  - "A web app lets users sign in with their work account and reads only their own profile. Which permission type do you use?" → Delegated permission (User.Read).
  - "The app was working, then suddenly gets 'invalid_client' errors. What is the most likely cause?" → Expired client secret.
- **Memorization Tip:** App registration = passport office (issues the identity document). Service principal = the actual passport (used in each country/tenant). Delegated = you act as the user. Application = you act as yourself.

***

## Pocket Summary (5-Year Refresh)
- App registration is the blueprint; service principal is the live instance used in each tenant.
- Client secrets expire — use certificates in production and store private keys in Key Vault.
- Delegated permissions = app acts as the signed-in user. Application permissions = app acts as itself (requires admin consent).
- Admin consent is required for all application permissions and some higher-privilege delegated permissions.
- `az ad sp create-for-rbac` is the fast path for CI/CD service principals with built-in RBAC assignment.

---
📚 Further Reading: https://learn.microsoft.com/en-us/entra/identity-platform/
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
