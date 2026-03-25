# Azure Managed Identities

> 📌 AZ-500 Exam Objective: Implement and manage managed identities
> 🏷️ Domain: 1 — Secure Identity and Access | Weight: 15–20%

***

## What Is This?
A managed identity is an automatic, Azure-managed service account for an Azure resource. When your virtual machine or App Service needs to access another Azure service — like Key Vault or Storage — it uses its managed identity instead of a username and password stored in code. Azure creates and rotates the credentials behind the scenes. You never see or touch them.

***

## Why Does This Exist?
Hard-coded credentials in application code are one of the most common causes of security breaches. Developers put a storage connection string or API key in a config file, commit it to a repository, and suddenly it is exposed. Managed identities eliminate this entire category of risk. The app never has a secret to lose. Azure handles authentication automatically using certificates that Microsoft rotates internally.

***

## Who Actually Needs This? (Real-World Context)
An e-commerce company in Seattle runs an Azure App Service that retrieves database passwords from Azure Key Vault at startup. Without managed identity, a developer must create a service principal, generate a client secret, store that secret somewhere, and manually rotate it every 90 days. With a managed identity, the App Service has an automatic identity. The developer assigns it "Key Vault Secrets User" RBAC rights. At runtime, the app calls the Azure Instance Metadata Service (IMDS) internally and gets a token with no secret ever touching the codebase.

***

## How It Works (Pure Concept, No Portal)

**Two types of managed identities:**

| Type | Created by | Lifecycle | Shared across resources |
|------|-----------|-----------|------------------------|
| **System-assigned** | Azure, when you enable it on a specific resource | Tied to the resource — deleted when the resource is deleted | No — one identity per resource |
| **User-assigned** | You create it as a standalone Azure resource | Independent — persists even if the resource is deleted | Yes — can be assigned to multiple resources |

**How authentication works:**
1. You enable a managed identity on a resource (e.g., a VM).
2. Entra ID creates a service principal in your tenant automatically. You don't manage it.
3. When the app on the VM needs a token, it calls the **Azure Instance Metadata Service (IMDS)** at the internal IP `169.254.169.254` — not reachable from outside the VM.
4. Azure returns a short-lived token signed by Microsoft.
5. The app presents this token to the target service (Key Vault, Storage, SQL, etc.).
6. The target service validates the token against Entra ID and grants access — if the identity has the right RBAC role.

**Important constraint:** Managed identities only work within the Azure ecosystem. They cannot be used to authenticate to on-premises systems or non-Azure services.

**Supported resources:** VMs, VM Scale Sets, App Service, Function Apps, Container Apps, AKS pods, Logic Apps, Service Fabric, Data Factory, and more.

***

## 2026 Portal Walkthrough — Step by Step

### Enable System-Assigned Identity on a VM
```
portal.azure.com
  → Top search bar → type "Virtual machines" → select it
  → Select your VM (e.g., "vm-webapp-prod")
  → Left menu → Security → Identity
  → System assigned tab
  → Status → toggle to "On"
  → Click Save → confirm in the pop-up
  → The Object ID shown is your VM's service principal ID in Entra ID
```

### Grant the VM's Identity Access to Key Vault
```
portal.azure.com
  → Top search bar → type "Key vaults" → select your Key Vault
  → Left menu → Access control (IAM)
  → + Add → Add role assignment
  → Role tab → search "Key Vault Secrets User" → select it → Next
  → Members tab → Assign access to: Managed identity
  → + Select members
  → Managed identity: Virtual machine
  → Select your VM → Select → Next
  → Review + assign → Assign
```

### Create a User-Assigned Managed Identity
```
portal.azure.com
  → Top search bar → type "Managed Identities" → select it
  → + Create
  → Subscription: your subscription
  → Resource group: rg-shared-identities
  → Region: East US
  → Name: "id-appservice-keyvault"
  → Review + create → Create

  → Then assign it to an App Service:
  → Go to your App Service → Identity
  → User assigned tab → + Add → select "id-appservice-keyvault" → Add
```

### CLI Equivalents
```bash
# Enable system-assigned identity on a VM
az vm identity assign \
  --name "vm-webapp-prod" \
  --resource-group "rg-webapp-prod"

# Get the principal ID of the VM's managed identity
PRINCIPAL_ID=$(az vm identity show \
  --name "vm-webapp-prod" \
  --resource-group "rg-webapp-prod" \
  --query principalId -o tsv)

# Grant Key Vault Secrets User role to the VM identity
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions/SUB-ID/resourceGroups/rg-keyvault/providers/Microsoft.KeyVault/vaults/kv-prod"

# Create a user-assigned managed identity
az identity create \
  --name "id-appservice-keyvault" \
  --resource-group "rg-shared-identities" \
  --location "eastus"

# Assign user-assigned identity to an App Service
az webapp identity assign \
  --name "mywebapp" \
  --resource-group "rg-webapp-prod" \
  --identities "/subscriptions/SUB-ID/resourceGroups/rg-shared-identities/providers/Microsoft.ManagedIdentity/userAssignedIdentities/id-appservice-keyvault"
```

### PowerShell Equivalents
```powershell
# Enable system-assigned identity on a VM
$vm = Get-AzVM -Name "vm-webapp-prod" -ResourceGroupName "rg-webapp-prod"
Update-AzVM -VM $vm -ResourceGroupName "rg-webapp-prod" -IdentityType SystemAssigned

# Get the principal ID
$principalId = (Get-AzVM -Name "vm-webapp-prod" -ResourceGroupName "rg-webapp-prod").Identity.PrincipalId

# Assign role to the managed identity
New-AzRoleAssignment `
  -ObjectId $principalId `
  -RoleDefinitionName "Key Vault Secrets User" `
  -Scope "/subscriptions/SUB-ID/resourceGroups/rg-keyvault/providers/Microsoft.KeyVault/vaults/kv-prod"
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/

***

## Key Settings Explained
| Setting | What It Does | When to Use |
|---------|-------------|-------------|
| System-assigned identity | Creates a 1:1 identity tied to the resource lifetime | VMs and App Services that are long-lived and standalone |
| User-assigned identity | Creates a reusable identity resource | When multiple resources need the same identity (e.g., a fleet of VMs) |
| Object ID (Principal ID) | The unique ID of the managed identity in Entra ID | Use this when assigning RBAC roles to the identity |
| IMDS endpoint (169.254.169.254) | Internal Azure endpoint the app calls to get a token | Referenced in SDK code — not a setting you configure |
| Role assignment on target service | Grants the identity permission to act on the target | Required — creating the identity does NOT grant access automatically |

***

## Comparison Table
| Feature | System-Assigned Identity | User-Assigned Identity | Service Principal (Manual) |
|---------|------------------------|----------------------|--------------------------|
| Created by | Azure (tied to resource) | You (standalone resource) | You (manual) |
| Lifecycle | Deleted with resource | Independent | Independent |
| Shared across resources | ❌ | ✅ | ✅ |
| Credentials to manage | ❌ None | ❌ None | ✅ Yes (secret or cert) |
| Works outside Azure | ❌ | ❌ | ✅ |
| Best for | Single resources | Shared identity scenarios | Non-Azure apps, CI/CD |

***

## What Happens If You Skip This?
When developers skip managed identities, they store credentials directly in code or config files. These secrets get committed to GitHub, included in container images, or written to log files. According to GitHub's annual secret scanning report, millions of secrets are accidentally pushed to public repositories every year. Any one of those exposed credentials can give an attacker direct access to your storage accounts, Key Vault secrets, or databases. Managed identities make this entire class of vulnerability impossible.

***

## AZ-500 Exam Section
- **Trigger Words:** "no credentials in code", "application identity", "system-assigned", "user-assigned", "IMDS", "eliminate secrets", "Key Vault access from VM"
- **Common Traps:**
  - **Managed identities only work within Azure.** If an exam question mentions accessing Azure resources from an on-premises server or a GitHub Action, managed identities cannot help — use a service principal or workload identity federation instead.
  - **Creating the identity does NOT grant access.** You must also create an RBAC role assignment on the target resource. Forgetting this step is the most common real-world mistake.
  - **System-assigned is deleted with the resource.** If you delete a VM with a system-assigned identity, the Entra service principal is gone. If something referenced that identity, it breaks.
  - **User-assigned identity persists independently.** If the VM is deleted, the user-assigned identity still exists in Azure and can be assigned to other resources.
- **How to Tell Apart from Similar Services:** Managed identities are for Azure resources authenticating to other Azure services — no credentials. Service principals are for any application (including non-Azure) that needs an identity — requires managing a secret or certificate. Workload identity federation extends the managed identity concept to external workloads like GitHub Actions.
- **Likely Question Formats:**
  - "An App Service needs to retrieve secrets from Key Vault without storing credentials. What do you configure?" → Enable system-assigned managed identity on the App Service, then assign Key Vault Secrets User role to the identity.
  - "A fleet of 50 VMs all need the same access to a Storage Account. Which managed identity type is most efficient?" → User-assigned managed identity (create one, assign to all 50 VMs).
  - "A developer accidentally committed a Key Vault client secret to a public GitHub repo. How do you prevent this in the future?" → Replace the service principal authentication with a managed identity.
- **Memorization Tip:** Think of managed identities as Azure's version of a building access badge. The building (Azure) issues the badge and renews it automatically. Your app just swipes the badge — no PIN to remember, no badge to lose.

***

## Pocket Summary (5-Year Refresh)
- Managed identities give Azure resources an automatic identity so they can authenticate to other Azure services without any stored credentials.
- System-assigned = tied to one resource, deleted with it. User-assigned = standalone, reusable across multiple resources.
- Always assign an RBAC role on the target service after enabling the identity — the identity alone grants nothing.
- The app gets tokens by calling the Azure IMDS endpoint internally — no code changes to manage secrets.
- Managed identities only work within Azure. For external systems, use service principals.

---
📚 Further Reading: https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
