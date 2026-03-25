# Multi-Factor Authentication and Conditional Access

> 📌 AZ-500 Exam Objective: Implement and manage Microsoft Entra authentication and authorization
> 🏷️ Domain: 1 — Secure Identity and Access | Weight: 15–20%

***

## What Is This?
Multi-Factor Authentication (MFA) requires users to prove their identity in two or more ways before signing in — typically a password plus a phone approval. Conditional Access is a policy engine that decides, based on real-time signals, whether to allow, block, or require additional verification for a sign-in attempt.

***

## Why Does This Exist?
Passwords alone fail constantly. Phishing, credential stuffing, and data breaches expose millions of passwords every year. MFA stops most of those attacks because stealing a password is not enough — the attacker still needs the second factor. Conditional Access goes further: it evaluates context (location, device health, user risk) and applies the right level of friction automatically, without prompting every user every time.

***

## Who Actually Needs This? (Real-World Context)
A national accounting firm has 800 employees who work from home, client offices, and coffee shops. The security team cannot control every network those employees use. They configure Conditional Access to require MFA when an employee signs in from an unknown location or a non-compliant device. Employees at the office on a managed, Intune-enrolled laptop sign in seamlessly with no extra prompt. The policy adds friction only when the risk is higher — protecting the firm without slowing down normal work.

***

## How It Works (Pure Concept, No Portal)

**MFA Methods (strongest to weakest):**
- **FIDO2 security key** — physical hardware key (YubiKey); phishing-resistant; strongest option.
- **Microsoft Authenticator app (number matching)** — push notification with a number to type; resistant to MFA fatigue attacks.
- **OATH hardware/software tokens** — time-based one-time passwords (TOTP).
- **SMS / voice call** — weakest; vulnerable to SIM-swap attacks; avoid for privileged accounts.

**How Conditional Access Works:**
- Microsoft collects **signals**: user identity, device compliance state, IP address / named location, application being accessed, sign-in risk score (from Entra ID Protection), and user risk score.
- A Conditional Access **policy** evaluates: IF [these signals match] THEN [grant / block / require MFA / require compliant device].
- Policies are evaluated at sign-in time, not at account creation.
- **Named locations** let you define trusted IP ranges (like your office subnet) or countries.
- **Sign-in risk** is a real-time score (low/medium/high) calculated by Microsoft's AI based on behavioral signals like impossible travel or anonymous IP use.
- **Compliance requirement** works with Microsoft Intune — the device must meet your security baseline (screen lock, encryption, OS patch level) before access is granted.

**Key Rule:** Conditional Access requires Entra ID P1 or P2. MFA itself is free for some scenarios (Security Defaults) but policy-based MFA requires P1.

***

## 2026 Portal Walkthrough — Step by Step

### Create a Conditional Access Policy Requiring MFA for All Admins
```
portal.azure.com
  → Top search bar → type "Microsoft Entra ID" → select it
  → Left menu → Security
  → Conditional Access
  → + New policy
  → Name: "Require MFA for Admins"

  → Assignments section:
    → Users: Select "Directory roles" → check Global Administrator,
      Privileged Role Administrator, Security Administrator
    → Target resources: All cloud apps

  → Conditions (optional but recommended):
    → Sign-in risk: Include Medium and High

  → Access controls → Grant:
    → Select "Grant access"
    → Check "Require multifactor authentication"
    → Click Select

  → Enable policy: toggle to "On"
  → Click Create
```

### Create a Named Location (Trusted Office IP)
```
portal.azure.com
  → Microsoft Entra ID → Security → Conditional Access
  → Left menu → Named locations
  → + IP ranges location
  → Name: "Corporate HQ - Austin TX"
  → Check "Mark as trusted location"
  → + Add IP range → enter your office public IP in CIDR (e.g., 203.0.113.0/24)
  → Create
```

### CLI Equivalent
```bash
# Conditional Access policies are managed via Microsoft Graph API.
# Use az rest to call the Graph endpoint directly.

az rest \
  --method POST \
  --uri "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies" \
  --headers "Content-Type=application/json" \
  --body '{
    "displayName": "Require MFA for Admins",
    "state": "enabled",
    "conditions": {
      "users": {
        "includeRoles": ["62e90394-69f5-4237-9190-012177145e10"]
      },
      "applications": {
        "includeApplications": ["All"]
      }
    },
    "grantControls": {
      "operator": "OR",
      "builtInControls": ["mfa"]
    }
  }'
```

### PowerShell Equivalent
```powershell
Connect-MgGraph -Scopes "Policy.ReadWrite.ConditionalAccess"

$params = @{
    DisplayName = "Require MFA for Admins"
    State = "enabled"
    Conditions = @{
        Users = @{
            IncludeRoles = @("62e90394-69f5-4237-9190-012177145e10") # Global Admin role ID
        }
        Applications = @{
            IncludeApplications = @("All")
        }
    }
    GrantControls = @{
        Operator = "OR"
        BuiltInControls = @("mfa")
    }
}

New-MgIdentityConditionalAccessPolicy -BodyParameter $params
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/entra/identity/conditional-access/

***

## Key Settings Explained
| Setting | What It Does | When to Use |
|---------|-------------|-------------|
| Grant: Require MFA | Forces a second factor before granting access | All admin accounts, always |
| Grant: Require compliant device | Checks Intune compliance before granting access | Corporate-owned device scenarios |
| Block access | Hard deny — no way through | Legacy auth protocols, high-risk countries |
| Sign-in risk condition | Triggers policy when AI detects anomalous sign-in | Detect impossible travel, anonymous IPs |
| Named location (trusted) | Marks IP ranges as safe — can skip MFA for these | Office networks where device control is high |
| Session: Sign-in frequency | Forces re-authentication after X hours/days | Sensitive apps like finance or HR systems |
| Session: Persistent browser session | Controls whether "Stay signed in" is allowed | Disable on shared/kiosk computers |

***

## Comparison Table
| Feature | Security Defaults | Conditional Access (P1) | Conditional Access + Identity Protection (P2) |
|---------|-----------------|------------------------|----------------------------------------------|
| MFA for admins | ✅ Automatic | ✅ Policy-based | ✅ Policy-based |
| MFA for all users | ✅ When risky | ✅ Configurable | ✅ Risk-adaptive |
| Named locations | ❌ | ✅ | ✅ |
| Sign-in risk policies | ❌ | ❌ | ✅ |
| Custom policy logic | ❌ | ✅ | ✅ |
| Licensing required | Free | P1 | P2 |

***

## What Happens If You Skip This?
Without MFA, a single stolen password gives full account access. In 2022, the Lapsus$ group bypassed enterprise security at multiple tech companies using only phished credentials — no malware, no exploits, just passwords. Every one of those breaches would have been stopped or severely limited by enforced MFA. Without Conditional Access, you also can't stop sign-ins from high-risk countries or compromised devices — your data walks out through a browser from anywhere in the world.

***

## AZ-500 Exam Section
- **Trigger Words:** "require MFA", "block legacy auth", "named location", "sign-in risk", "compliance", "Conditional Access policy", "impossible travel"
- **Common Traps:**
  - MFA alone is NOT Conditional Access. MFA is a control. Conditional Access is the policy that decides when to apply that control.
  - Security Defaults enable MFA for everyone but are all-or-nothing. You cannot customize them. Once you create any Conditional Access policy, Microsoft recommends disabling Security Defaults.
  - Conditional Access evaluates policies at sign-in time, not at role assignment time.
  - "Block legacy authentication" is its own policy — you set the condition to client apps = Exchange ActiveSync + other legacy clients, then block.
- **How to Tell Apart from Similar Services:** MFA per-user settings (the old method) are being retired in favor of Conditional Access. Identity Protection sets risk levels; Conditional Access consumes those risk levels and acts on them.
- **Likely Question Formats:**
  - "A user signs in from an anonymous IP. How do you automatically require MFA?" → Conditional Access policy with sign-in risk condition (requires P2).
  - "How do you block all access from legacy authentication clients?" → Conditional Access policy targeting legacy auth client apps with Block control.
  - "An admin needs MFA enforced but regular users should not be affected. What do you configure?" → Conditional Access targeting specific directory roles.
- **Memorization Tip:** Think of Conditional Access as a bouncer with a checklist: Who are you? Where are you? What device are you on? What app do you want? The bouncer decides: let in, ask for ID (MFA), or turn away (block).

***

## Pocket Summary (5-Year Refresh)
- MFA adds a second proof of identity — the Microsoft Authenticator app with number matching is the recommended method.
- Conditional Access is a policy engine that evaluates signals (user, device, location, risk) and decides how to respond.
- Requires Entra ID P1 at minimum; sign-in risk features require P2.
- Always block legacy authentication protocols — they cannot support MFA and are a top attack vector.
- Named locations let you skip MFA for trusted offices while still enforcing it everywhere else.

---
📚 Further Reading: https://learn.microsoft.com/en-us/entra/identity/conditional-access/
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
