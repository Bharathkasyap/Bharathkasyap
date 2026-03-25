# Azure Role-Based Access Control (RBAC) and Custom Roles

> 📌 AZ-500 Exam Objective: Configure and manage Azure RBAC
> 🏷️ Domain: 1 — Secure Identity and Access | Weight: 15–20%

***

## What Is This?
Azure RBAC (Role-Based Access Control) is the system that controls who can do what to Azure resources. You assign a role to a user, group, or application at a specific scope. The role defines the allowed actions. The scope limits where those actions apply.

***

## Why Does This Exist?
Without RBAC, the only option would be giving everyone full admin rights or no rights at all. Neither extreme works for real organizations. RBAC lets a company give a developer write access to one resource group, a database admin read access to SQL servers only, and an auditor read-only access across the whole subscription — all at the same time, without any of them interfering with each other.

***

## Who Actually Needs This? (Real-World Context)
A logistics company runs separate Azure environments for their web app, database, and analytics pipeline. The web dev team needs to deploy App Service updates. The data team needs access to Azure Data Factory. The finance team needs read-only access to see resource costs. The security team needs to see everything but change nothing. RBAC lets IT assign exactly these four different permission sets to four different groups, all living in the same Azure subscription.

***

## How It Works (Pure Concept, No Portal)

**The three components of every role assignment:**
1. **Security principal** — the WHO: a user, group, service principal, or managed identity.
2. **Role definition** — the WHAT: a named set of allowed (and optionally denied) actions on Azure resources.
3. **Scope** — the WHERE: management group, subscription, resource group, or individual resource.

**Scope hierarchy (broadest to narrowest):**
```
Management Group
  └── Subscription
        └── Resource Group
              └── Individual Resource
```
Permissions assigned at a higher scope are inherited down. Assign Contributor at the subscription level, and the user gets Contributor on every resource group and resource inside that subscription.

**Built-in roles you must know:**
- **Owner** — full control including the ability to assign roles to others.
- **Contributor** — full control over resources, but CANNOT assign roles or manage access.
- **Reader** — view everything, change nothing.
- **User Access Administrator** — can manage role assignments, but has no resource permissions itself.

**Custom roles:**
- When built-in roles are too broad, you create a custom role using a JSON definition.
- You specify: `Actions` (allowed), `NotActions` (excluded from allowed), `DataActions` (data plane like reading blob contents), `NotDataActions`, and `AssignableScopes`.
- Custom roles are defined at the tenant level and can be assigned to any scope within the assignable scopes list.

**Deny assignments:**
- Applied automatically by Azure Blueprints or Managed Applications.
- A deny assignment overrides any role assignment, including Owner.
- You cannot create deny assignments manually through the portal or CLI — they are system-managed.

**Least privilege principle:** Always grant the minimum permissions a principal needs to do their job. Nothing more.

***

## 2026 Portal Walkthrough — Step by Step

### Assign Contributor Role at Resource Group Scope
```
portal.azure.com
  → Top search bar → type "Resource groups" → select it
  → Select your resource group (e.g., "rg-webapp-prod")
  → Left menu → Access control (IAM)
  → + Add → Add role assignment
  → Role tab → search "Contributor" → select it → Next
  → Members tab → + Select members
  → Search for the user or group name → Select → Next
  → Review + assign → Assign
```

### Create a Custom Role (via JSON upload)
```
portal.azure.com
  → Top search bar → type "Subscriptions" → select your subscription
  → Left menu → Access control (IAM)
  → + Add → Add custom role
  → Start from scratch
  → Basics tab → Role name: "VM Operator - ReadStart"
  → Permissions tab → + Add permissions
    → Search "Microsoft.Compute/virtualMachines"
    → Check: read, start, restart, deallocate — uncheck everything else
  → Assignable scopes tab → confirm subscription scope
  → Review + create → Create
```

### CLI Equivalents
```bash
# Assign the Contributor role at resource group scope
az role assignment create \
  --assignee "jsmith@contoso.onmicrosoft.com" \
  --role "Contributor" \
  --resource-group "rg-webapp-prod"

# Create a custom role from a JSON file
cat > vm-operator-role.json << 'EOF'
{
  "Name": "VM Operator - ReadStart",
  "Description": "Can read, start, restart, and deallocate VMs only.",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/deallocate/action"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/YOUR-SUBSCRIPTION-ID"
  ]
}
EOF

az role definition create --role-definition vm-operator-role.json

# List all role assignments in a resource group
az role assignment list --resource-group "rg-webapp-prod" --output table
```

### PowerShell Equivalents
```powershell
# Assign a role
New-AzRoleAssignment `
  -SignInName "jsmith@contoso.onmicrosoft.com" `
  -RoleDefinitionName "Contributor" `
  -ResourceGroupName "rg-webapp-prod"

# Create a custom role
$role = [Microsoft.Azure.Commands.Resources.Models.Authorization.PSRoleDefinition]::new()
$role.Name = "VM Operator - ReadStart"
$role.Description = "Can read, start, restart, and deallocate VMs only."
$role.Actions = @(
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/deallocate/action"
)
$role.AssignableScopes = @("/subscriptions/YOUR-SUBSCRIPTION-ID")
New-AzRoleDefinition -Role $role
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/role-based-access-control/

***

## Key Settings Explained
| Setting | What It Does | When to Use |
|---------|-------------|-------------|
| Actions | Lists allowed Azure Resource Manager operations (control plane) | Core of every role definition |
| NotActions | Subtracts specific operations from Actions | Narrow a broad built-in role without creating a full custom role |
| DataActions | Allows operations on data inside a resource (e.g., read blob data) | When an app needs data-level access, not just resource management |
| AssignableScopes | Limits which scopes this custom role can be assigned at | Scope custom roles to specific subscriptions or RGs |
| Condition (ABAC) | Adds attribute-based conditions to a role assignment | Grant access only to blobs with a specific tag value |
| Deny assignment | Blocks specific actions regardless of role assignments | Applied by Blueprints; you cannot create these manually |

***

## Comparison Table
| Feature | Azure RBAC | Entra ID Directory Roles |
|---------|-----------|--------------------------|
| Controls access to | Azure resources (VMs, storage, SQL) | Entra ID tenant settings (users, groups, apps) |
| Scope | MG / Subscription / RG / Resource | Tenant-wide only |
| Examples | Owner, Contributor, Reader | Global Admin, User Admin, Security Reader |
| Where assigned | Azure Portal → IAM blade | Entra ID → Roles and Administrators |
| Inheritance | Yes — flows down the scope hierarchy | No — flat, tenant-wide |
| Custom roles | ✅ Yes | ✅ Yes (custom directory roles) |

***

## What Happens If You Skip This?
Without RBAC, every person with Azure access gets the same level of control. A junior developer accidentally deletes a production database — and no permission model prevented it. Or worse, a compromised vendor account has Owner rights across your entire subscription and exfiltrates every storage account it can find. In 2021, a misconfigured Azure environment with over-permissioned service accounts led to the ChaosDB vulnerability, where researchers could access other customers' Cosmos DB data. Least-privilege RBAC limits the blast radius of any mistake or breach.

***

## AZ-500 Exam Section
- **Trigger Words:** "least privilege", "custom role", "deny assignment", "scope", "role assignment", "Contributor", "Owner", "Reader", "IAM"
- **Common Traps:**
  - **Entra ID roles ≠ Azure RBAC roles.** Global Administrator in Entra ID does NOT automatically give Owner access to Azure resources. These are two separate planes.
  - **Owner ≠ unlimited power.** A deny assignment from Azure Blueprints will block an Owner too.
  - **NotActions is not a deny.** It removes actions from the allowed set. If another role grants those actions, they still work. Only deny assignments truly block.
  - **Contributor cannot assign roles.** If a question asks who can grant access, it must be Owner or User Access Administrator.
- **How to Tell Apart from Similar Services:** RBAC is for Azure resources (the Azure Resource Manager plane). Entra ID Directory Roles are for the identity plane. PIM manages the activation lifecycle of both.
- **Likely Question Formats:**
  - "A developer needs to deploy VMs but must not be able to change networking settings. What do you do?" → Create a custom role.
  - "Who can grant a user access to a resource group?" → Owner or User Access Administrator.
  - "A role assignment grants access but the user is still blocked. What could cause this?" → A deny assignment exists.
- **Memorization Tip:** Think of RBAC like a job title system. Your title (role) defines your duties. Your office location (scope) defines where you do them. Your employee ID (security principal) is who you are. All three must match before you can act.

***

## Pocket Summary (5-Year Refresh)
- RBAC controls access to Azure resources using three components: security principal + role definition + scope.
- Built-in roles to know: Owner (full + can assign), Contributor (full, no assign), Reader (view only), User Access Administrator (assign only).
- Scope hierarchy: Management Group → Subscription → Resource Group → Resource. Higher scope inherits down.
- Create custom roles in JSON when built-in roles are too broad or too narrow.
- Deny assignments override everything including Owner — but you can only see them, not create them manually.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/role-based-access-control/
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
