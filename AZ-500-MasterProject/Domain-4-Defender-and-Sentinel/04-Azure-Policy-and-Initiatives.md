# Azure Policy and Security Initiatives

> 📌 AZ-500 Exam Objective: Manage security policies using Azure Policy and regulatory compliance initiatives
> 🏷️ Domain: 4 — Defender for Cloud and Sentinel | Weight: 30–35%

***

## What Is This?

Azure Policy is a service that enforces rules on Azure resources. You define what a resource must look like — for example, "all storage accounts must use HTTPS only" — and Azure Policy checks every resource against that rule. When resources break the rule, Azure Policy can log the violation, block the action, or automatically fix it depending on how you configure it.

***

## Why Does This Exist?

In large organizations, many people create Azure resources. Without enforcement, someone will inevitably create an insecure resource — an unencrypted disk, a storage account with public access, a VM without tags. Azure Policy automates the guardrails so security rules apply consistently across every resource, every team, and every subscription without relying on manual checks.

***

## Who Actually Needs This? (Real-World Context)

A government contractor called SecureGov must comply with NIST SP 800-53. They have 30 Azure subscriptions across three regions. Their compliance team assigns the NIST SP 800-53 initiative to their root management group. Azure Policy now evaluates every resource in all 30 subscriptions against the NIST controls. The compliance dashboard shows which specific controls are passing and which are failing — with the exact resources that are non-compliant. During an audit, the compliance team exports this report to prove their posture to auditors.

***

## How It Works (Pure Concept, No Portal)

**Azure Policy — The Basics**
A policy is a rule written in JSON. It has two parts:
1. **Condition** — what to check (e.g., "does this storage account have HTTPS-only enabled?")
2. **Effect** — what to do if the condition is met or not met.

**Policy Effects**
Effects control what happens when a resource does not comply with a policy rule:

| Effect | What It Does |
|---|---|
| **Audit** | Logs non-compliant resources. No action taken. Resource is created or changed normally. |
| **Deny** | Blocks the action. Non-compliant resources cannot be created or updated. |
| **Modify** | Adds or changes properties on a resource to make it compliant. |
| **DeployIfNotExists** | Deploys a related resource if it does not exist (e.g., deploy a diagnostic setting to a storage account). |
| **AuditIfNotExists** | Logs non-compliance if a related resource does not exist. |
| **Append** | Adds properties to a resource during creation or update. |
| **Disabled** | Turns the policy off without removing the assignment. |

**Initiatives (Policy Sets)**
An initiative is a bundle of policies grouped together to meet a compliance standard. Instead of assigning 200 individual policies, you assign one initiative. Examples of built-in security initiatives:
- **Azure Security Benchmark (ASB)** — Microsoft's default security recommendations. This is what powers Defender for Cloud's Secure Score.
- **CIS Microsoft Azure Foundations Benchmark** — Center for Internet Security standard.
- **NIST SP 800-53** — U.S. federal security standard.
- **PCI-DSS** — Payment card industry standard.
- **ISO 27001** — International information security standard.
- **HIPAA** — U.S. healthcare privacy standard.

**Policy Assignment Scope**
Policies can be assigned at these levels (broadest to narrowest):
1. Management Group — applies to all subscriptions in the group.
2. Subscription — applies to all resource groups in the subscription.
3. Resource Group — applies to all resources in the group.
4. Resource — applies to one specific resource.

Assignments at a higher scope inherit down. You can exclude specific child scopes from an assignment.

**Compliance State**
After assigning a policy, Azure evaluates all existing resources. Each resource gets a compliance state:
- **Compliant** — the resource meets the policy rule.
- **Non-compliant** — the resource violates the policy rule.
- **Exempt** — the resource has been excluded from evaluation.
- **Conflicting** — the resource has conflicting policy assignments.

**Remediation Tasks**
For policies with the **DeployIfNotExists** or **Modify** effect, you can create remediation tasks. A remediation task scans all existing non-compliant resources and fixes them by running the policy's deployment or modification. New resources are fixed at creation time. Existing resources need a remediation task to be fixed retroactively.

A managed identity is required to run remediation tasks. Azure Policy creates this identity automatically when needed. You assign the identity the appropriate RBAC role to make changes to resources.

***

## 2026 Portal Walkthrough — Step by Step

**Assign Azure Security Benchmark Initiative:**
1. Go to **Microsoft Defender for Cloud** in the Azure Portal.
2. Click **Environment settings**, then select your subscription.
3. Click **Security policies** at the top.
4. Find **Azure Security Benchmark** in the default policies list.
5. Toggle it to **On** (it is on by default for most subscriptions).
6. To assign it manually: go to **Azure Policy** from the portal search.
7. Click **Assignments** → **Assign initiative**.
8. Set scope to your subscription or management group.
9. Search for and select **Azure Security Benchmark**.
10. Click **Review + Create** → **Create**.

**View Compliance:**
1. In **Azure Policy**, click **Compliance** in the left menu.
2. Select your initiative assignment from the list.
3. The compliance dashboard shows overall compliance percentage and per-policy breakdown.
4. Click any non-compliant policy to see the list of non-compliant resources.

**Create a Remediation Task:**
1. In Azure Policy, click **Remediation** in the left menu.
2. You see a list of policies that need remediation (DeployIfNotExists or Modify effects).
3. Click **Remediate** next to a policy.
4. Confirm the scope and the managed identity permissions.
5. Click **Remediate** to start the task.
6. The task runs in the background. Refresh to see the remediation progress.

**CLI Equivalent:**
```bash
# List all built-in policy definitions
az policy definition list --query "[?policyType=='BuiltIn']" --output table

# Assign Azure Security Benchmark initiative to a subscription
az policy assignment create \
  --name "ASB-Assignment" \
  --display-name "Azure Security Benchmark" \
  --scope "/subscriptions/{subscriptionId}" \
  --policy-set-definition "1f3afdf9-d0c9-4c3d-847f-89da613e70a8"

# List policy assignments at subscription scope
az policy assignment list --output table

# Get compliance summary for a subscription
az policy state summarize --subscription {subscriptionId}

# List non-compliant resources for a specific policy
az policy state list \
  --filter "complianceState eq 'NonCompliant'" \
  --output table

# Create a remediation task
az policy remediation create \
  --name "remediate-diagnostic-settings" \
  --policy-assignment "ASB-Assignment" \
  --resource-discovery-mode ReEvaluateCompliance
```

**PowerShell Equivalent:**
```powershell
# Assign a policy initiative to a subscription
$scope = "/subscriptions/{subscriptionId}"
$initiative = Get-AzPolicySetDefinition -Name "1f3afdf9-d0c9-4c3d-847f-89da613e70a8"
New-AzPolicyAssignment -Name "ASB-Assignment" -PolicySetDefinition $initiative -Scope $scope

# List all policy assignments
Get-AzPolicyAssignment | Format-Table Name, DisplayName, Scope

# Get compliance state for a subscription
Get-AzPolicyState -SubscriptionId "{subscriptionId}" | 
  Where-Object {$_.ComplianceState -eq "NonCompliant"} |
  Format-Table ResourceId, PolicyDefinitionName, ComplianceState

# Start a remediation task
Start-AzPolicyRemediation `
  -Name "fix-storage-https" `
  -PolicyAssignmentId "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyAssignments/ASB-Assignment"
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/governance/policy/overview

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Audit effect | Logs non-compliant resources without blocking anything | When you want visibility without disrupting existing workloads |
| Deny effect | Blocks creation or update of non-compliant resources | When you need hard enforcement — new resources must comply or they are rejected |
| DeployIfNotExists | Automatically deploys a companion resource if missing | When you need to enforce configuration that requires a separate resource (e.g., diagnostic settings, extensions) |
| Modify effect | Automatically changes a property on a resource to make it compliant | When you want Azure to auto-fix tag or property issues |
| Remediation task | Fixes existing non-compliant resources | Needed for DeployIfNotExists and Modify policies — they only apply automatically to new resources without a remediation task |
| Exclusion | Removes a specific scope from a policy assignment | When a child resource group or subscription has a valid reason to be exempt from the policy |
| Compliance dashboard | Shows overall and per-policy compliance percentages | For audit reporting, executive dashboards, and tracking remediation progress |
| Managed identity for policy | Allows Azure Policy to make changes during remediation | Required for DeployIfNotExists and Modify effects — must have appropriate RBAC |

***

## Comparison Table (if applicable)

| Standard | Organization | Focus |
|---|---|---|
| Azure Security Benchmark (ASB) | Microsoft | Azure-specific security best practices; default in Defender for Cloud |
| CIS Microsoft Azure Foundations | Center for Internet Security | Hardening guide for Azure; widely used in enterprise |
| NIST SP 800-53 | U.S. NIST | U.S. federal agencies and contractors; broad coverage |
| PCI-DSS | PCI Security Standards Council | Organizations that process payment cards |
| ISO 27001 | ISO / IEC | International information security management |
| HIPAA | U.S. HHS | U.S. healthcare organizations protecting patient data |
| SOC 2 | AICPA | Service organizations; security, availability, confidentiality |

***

## What Happens If You Skip This?

Without Azure Policy, teams create whatever they want without guardrails. Over time, your environment fills with unencrypted disks, open storage accounts, VMs without tags, and resources in unauthorized regions. You cannot prove compliance to auditors because there is no automated evidence. When the CIS benchmark says "all storage must use HTTPS only," you have no way to verify that 3,000 storage accounts across 50 subscriptions all comply — unless Azure Policy is doing it automatically.

***

## AZ-500 Exam Section

- **Trigger Words:** "policy assignment", "initiative", "DeployIfNotExists", "compliance state", "regulatory standard", "Audit", "Deny", "remediation task", "Azure Security Benchmark", "non-compliant"
- **Common Traps:**
  - Thinking Audit effect prevents non-compliant resources from being created — it does NOT. Audit only logs. Only Deny blocks.
  - Thinking DeployIfNotExists fixes existing resources automatically — it only applies to new resources at creation time. You need a remediation task to fix existing ones.
  - Thinking an initiative is a single policy — an initiative is a collection (bundle) of multiple policies grouped together.
  - Thinking Azure Policy and Azure RBAC are the same — RBAC controls who can do actions, Policy controls what resources must look like.
- **How to Tell Apart:**
  - RBAC = who can do what (identity-based)
  - Azure Policy = what resources must look like (configuration-based)
- **Likely Question Formats:**
  - "You need to prevent users from creating storage accounts with public access. Which policy effect should you use?" → Deny.
  - "You assigned a DeployIfNotExists policy, but existing resources are still non-compliant. What do you need to do?" → Create a remediation task.
  - "A company needs to show NIST SP 800-53 compliance across all Azure subscriptions. What do you assign?" → NIST SP 800-53 initiative at the management group level.
- **Memorization Tip:** Effects from weakest to strongest: **Disabled → Audit → AuditIfNotExists → Append → Modify → DeployIfNotExists → Deny**. Audit watches, Deny blocks, Deploy fixes.

***

## Pocket Summary (5-Year Refresh)

- Azure Policy enforces configuration rules on Azure resources — a resource either complies or it does not.
- Initiatives are bundles of policies used to meet compliance standards like NIST, PCI-DSS, CIS, or HIPAA.
- The Audit effect only logs violations — it never blocks or changes anything.
- The Deny effect prevents non-compliant resources from being created or updated.
- DeployIfNotExists and Modify effects can auto-fix configurations, but existing resources need a remediation task to be fixed retroactively.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/governance/policy/overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
