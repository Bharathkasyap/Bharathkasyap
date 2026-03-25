# Secure Score and Security Recommendations

> 📌 AZ-500 Exam Objective: Assess and improve security posture using Secure Score and recommendations
> 🏷️ Domain: 4 — Defender for Cloud and Sentinel | Weight: 30–35%

***

## What Is This?

Secure Score is a number between 0 and 100 that measures how well your Azure environment follows security best practices. The higher the score, the better your posture. It is made up of security controls — groups of related recommendations — each worth a certain number of points.

***

## Why Does This Exist?

Security teams need a clear, measurable way to track progress. Without a score, it is hard to know if things are getting better or worse over time. Secure Score gives leadership a simple metric to report on, and it gives engineers a prioritized list of exactly what to fix next to have the most impact.

***

## Who Actually Needs This? (Real-World Context)

A financial services firm called CapitalEdge has 200 Azure subscriptions managed by 12 different teams. The CISO needs a weekly report showing whether overall cloud security is improving. The security team uses Secure Score at the management group level to see a rolled-up score across all subscriptions. They track score trends over 90 days and assign remediation tasks to specific teams using the governance features in Defender CSPM. Each team owns their subscription's score.

***

## How It Works (Pure Concept, No Portal)

**Score Calculation**
Your Secure Score is calculated based on security controls. Each control has a maximum point value — for example, "Enable MFA" might be worth 10 points. If you fix all unhealthy resources in that control, you earn the full 10 points. If half the resources are fixed, you earn partial points.

The formula is:
> Score = (Points you earned) ÷ (Total possible points) × 100

**Security Controls**
A security control is a group of related recommendations. Instead of hundreds of individual items, Defender for Cloud groups them logically. Examples include:
- Enable MFA
- Encrypt data in transit
- Manage access and permissions
- Remediate vulnerabilities
- Restrict unauthorized network access

Each control shows you: the max points possible, the current points earned, the number of unhealthy resources, and the list of recommendations inside the control.

**Recommendations**
A recommendation is a specific action to take on a specific resource. Examples:
- "Enable MFA for accounts with owner permissions on your subscription"
- "Storage accounts should restrict network access"
- "Virtual machines should have vulnerability assessment solution enabled"

Each recommendation has a severity (High, Medium, Low) and a description of the risk and how to fix it.

**Unhealthy Resources**
Resources that do not meet a recommendation are called unhealthy. Each recommendation shows you which specific resources are unhealthy and which are healthy. You can drill down to a single resource and see all recommendations that apply to it.

**Exemptions**
Sometimes a recommendation does not apply to your environment. For example, a dev/test VM does not need the same vulnerability scanner as a production VM. You can create an exemption on a specific resource for a specific recommendation. The exemption removes that resource from the score calculation — it does not count against you. But it does NOT fix the underlying security issue. The resource is still misconfigured; you just told Defender to ignore it.

There are two exemption types:
- **Waiver** — you acknowledge the risk but choose not to fix it.
- **Mitigated** — the risk is addressed through a compensating control outside of Azure.

**Subscription vs Management Group Score**
- At the subscription level, you see a score for that single subscription.
- At the management group level, you see a rolled-up score across all subscriptions in the group.
- This lets CISOs track overall posture without looking at each subscription individually.

**Score Trends**
Defender for Cloud stores historical score data. You can view a trend graph showing how your score has changed over time — whether remediation efforts are working or if new resources are causing drift.

***

## 2026 Portal Walkthrough — Step by Step

**View Secure Score:**
1. Go to **Microsoft Defender for Cloud** in the Azure Portal.
2. Click **Security posture** in the left menu.
3. Your overall Secure Score is shown at the top as a percentage and a fraction (e.g., 65/100).
4. Below the score, you see the breakdown by security control with current and max points.
5. Use the **Subscriptions** dropdown to switch between subscriptions or management groups.

**Find and Remediate a Recommendation:**
1. Click **Recommendations** in the left menu.
2. Filter by **Severity: High** to find the most critical items first.
3. Click a recommendation — for example, "MFA should be enabled on accounts with owner permissions."
4. You see the list of affected (unhealthy) resources.
5. Click a resource to see the specific fix needed.
6. If a Quick Fix is available, click **Fix** and follow the guided remediation.
7. If no Quick Fix exists, follow the manual remediation steps described in the recommendation.

**Create an Exemption:**
1. Open a recommendation.
2. Select the resource you want to exempt.
3. Click the **Exempt** button (three-dot menu or top action bar).
4. Choose exemption type: **Waiver** or **Mitigated**.
5. Enter a justification, set an optional expiry date.
6. Click **Save**.
7. The exempted resource moves to the **Not applicable** tab and no longer affects the score.

**CLI Equivalent:**
```bash
# List all Secure Score controls with current and max points
az security secure-score-controls list --output table

# List all security assessments (recommendations) and their status
az security assessments list --output table

# View Secure Score for a specific subscription
az security secure-scores list --output table

# Create an exemption on a recommendation (requires REST API via az rest)
az rest --method PUT \
  --url "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Compute/virtualMachines/{vmName}/providers/Microsoft.Security/assessments/{assessmentName}/exemptions/{exemptionName}?api-version=2022-01-01-preview" \
  --body '{"properties":{"exemptionCategory":"Waiver","justification":"Dev VM, risk accepted"}}'
```

**PowerShell Equivalent:**
```powershell
# List Secure Score controls
Get-AzSecuritySecureScoreControl | Format-Table DisplayName, Score, Max

# List all security assessments
Get-AzSecurityAssessment | Format-Table DisplayName, StatusCode, ResourceDetails

# Get Secure Score for a subscription
Get-AzSecuritySecureScore | Format-Table DisplayName, Percentage
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/defender-for-cloud/secure-score-security-controls

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Security Controls | Groups recommendations by theme; each control has a point value | Use to prioritize which group of issues to fix first for maximum score gain |
| Quick Fix | One-click remediation for supported recommendations | Use when available to speed up bulk remediation across many resources |
| Exemption (Waiver) | Removes a resource from score calculation with acknowledged risk | Use for dev/test resources where the risk is accepted |
| Exemption (Mitigated) | Removes a resource from score calculation when a compensating control exists | Use when the risk is addressed outside Azure (e.g., network firewall appliance) |
| Management Group Score | Shows a rolled-up score across all child subscriptions | Use for executive reporting across large, multi-subscription environments |
| Score Trends | Historical graph of score changes over time | Use to measure whether remediation efforts are having an impact |
| Governance Rules | Assigns recommendation ownership and sets due dates for remediation | Use in Defender CSPM (paid) to hold teams accountable for their score |

***

## Comparison Table (if applicable)

| Concept | What It Is | Effect on Score |
|---|---|---|
| Healthy Resource | Resource meets the recommendation | Contributes positively to control score |
| Unhealthy Resource | Resource fails the recommendation | Reduces control score |
| Not Applicable | Recommendation does not apply (exemption or auto-excluded) | Removed from score calculation entirely — no penalty |
| Exempted Resource (Waiver) | Risk accepted; not fixed | Removed from score — issue still exists |
| Exempted Resource (Mitigated) | Addressed by compensating control | Removed from score — risk considered handled |
| Remediated Resource | Issue actually fixed | Score increases |

***

## What Happens If You Skip This?

If you ignore Secure Score and recommendations, your environment drifts further from security best practices with every new resource added. Misconfigured storage accounts, VMs without vulnerability scanning, and accounts without MFA pile up silently. Attackers actively look for these exact misconfigurations. By the time a breach happens, a quick look at Secure Score would have shown dozens of high-severity warnings that were never addressed.

***

## AZ-500 Exam Section

- **Trigger Words:** "secure score", "security control", "recommendation", "remediation", "exemption", "posture", "unhealthy resource", "management group score"
- **Common Traps:**
  - Thinking an exemption fixes the security issue — it does NOT. The resource is still misconfigured. The exemption only removes it from the score calculation.
  - Thinking remediating one resource fixes the whole control — you need to fix ALL unhealthy resources in a control to earn its full points.
  - Thinking Secure Score updates instantly — it can take up to 24 hours to reflect changes after remediation.
- **How to Tell Apart:**
  - Exemption = "we know it is broken, we accept the risk" (score improves, problem stays)
  - Remediation = "we actually fixed it" (score improves, problem is gone)
- **Likely Question Formats:**
  - "A VM was added as an exemption in a security recommendation. What is the effect on the Secure Score?" → The VM is removed from the score calculation; score may improve, but the underlying issue is not fixed.
  - "You need to see a combined Secure Score across 50 subscriptions. Where do you look?" → Management group Secure Score in Defender for Cloud.
  - "Which Azure feature assigns point values to groups of security recommendations?" → Security controls in Defender for Cloud.
- **Memorization Tip:** Secure Score is like a class grade. Each control is a homework assignment. Fixing resources earns points. Exemptions are excused absences — you still missed class, but it does not count against your grade.

***

## Pocket Summary (5-Year Refresh)

- Secure Score is a 0–100 number showing how well your environment follows security best practices.
- Score is based on security controls — groups of recommendations — each worth a set number of points.
- Fixing unhealthy resources raises your score; exempting them removes them from the calculation without fixing anything.
- You can view scores at the subscription level or rolled up at the management group level.
- Exemptions have two types: Waiver (risk accepted) and Mitigated (handled by compensating control).

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/defender-for-cloud/secure-score-security-controls
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
