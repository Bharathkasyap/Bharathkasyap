# Azure Blob Storage — SAS Tokens and Access Tiers

> 📌 AZ-500 Exam Objective: Plan and implement security for Azure Storage
> 🏷️ Domain: 3 — Secure Compute, Storage, and Databases | Weight: 20–25%

***

## What Is This?

A Shared Access Signature (SAS) is a URL you give to someone so they can access a specific blob, container, or storage account for a limited time with limited permissions. You control exactly what they can do and for how long — without sharing the account key. Access tiers (Hot, Cool, Archive) affect where data is physically stored and how fast it can be retrieved.

***

## Why Does This Exist?

You often need to give temporary, scoped access to storage without creating a full Azure user account. SAS tokens let a mobile app upload a single file, let a partner download a specific report, or let a build pipeline read a configuration blob — all without granting broad permissions. Access tiers reduce storage costs while maintaining security controls.

***

## Who Actually Needs This? (Real-World Context)

A media company in Los Angeles lets video editors upload raw footage directly from their laptops to Azure Blob Storage. The production web app generates a user delegation SAS token for each editor that is valid for 4 hours and allows writes to only their assigned upload container. When the upload window expires, the token is useless. The company never shares the storage account key — each editor gets only what they need, when they need it.

***

## How It Works (Pure Concept, No Portal)

**Types of SAS Tokens**

1. **Account SAS**: Backed by the storage account key. Grants access across multiple services (Blob, File, Queue, Table) and multiple resource types. The broadest SAS type. Because it is key-backed, you cannot revoke it without rotating the key.

2. **Service SAS**: Also backed by the storage account key. Scoped to a single storage service (Blob only, or File only). Still cannot be revoked without key rotation unless it is tied to a Stored Access Policy.

3. **User Delegation SAS**: Backed by an Entra ID identity and its permissions. Can only be used for Blob and Data Lake Storage. This is the most secure SAS type because:
   - It uses short-lived Entra ID credentials (delegation keys, up to 7 days max).
   - You can revoke it by removing the user's RBAC role or revoking the delegation key.
   - It produces a full audit trail in Entra ID and storage logs.

**SAS Token Components**

A SAS token is a query string appended to a storage URL. Key parameters:
- `sv` (signedVersion): API version.
- `ss` (signedServices): which services (b=blob, f=file, q=queue, t=table).
- `srt` (signedResourceTypes): service, container, or object level.
- `sp` (signedPermissions): r=read, w=write, d=delete, l=list, a=add, c=create.
- `se` (signedExpiry): expiry date and time in UTC. After this, the token is dead.
- `st` (signedStart): optional start time.
- `sip` (signedIP): restricts the token to specific IP addresses or ranges.
- `spr` (signedProtocol): `https` only (recommended) or `https,http`.
- `sig` (signature): the HMAC-SHA256 signature that proves the token was created by someone with the key.

**Best practices for SAS tokens:**
- Always set `spr=https` to prevent the token from being used over unencrypted HTTP.
- Always set `sip` to restrict to known IP ranges when possible.
- Use the shortest expiry that works for the use case.
- Prefer user delegation SAS over account or service SAS.

**Stored Access Policies**
A Stored Access Policy is a named policy saved on the container or queue that defines permissions and expiry. A service SAS can reference a stored access policy by name. This lets you revoke the SAS without rotating the key — just delete the stored access policy and all SAS tokens referencing it immediately stop working. This is the only way to revoke a key-backed SAS without rotating the account key.

**Anonymous Blob Access**
Azure allows containers to be set to anonymous (public) access so anyone on the internet can read blobs without any credentials. This should almost always be disabled. The only exception is intentionally public content like public website assets. Even then, use a CDN rather than direct anonymous blob access. As of November 2023, new storage accounts have anonymous access disabled by default.

**Blob Versioning**
When blob versioning is enabled, every time a blob is overwritten, the previous version is retained. You can restore a previous version at any time. This protects against accidental overwrites and is complementary to soft delete (which protects against deletions).

**Change Feed**
The change feed is an ordered log of all changes to blobs (create, modify, delete) in a storage account. It is useful for auditing and compliance — you can replay every change to a container over time.

**Access Tiers and Security Implications**
- **Hot**: Frequently accessed data. Highest storage cost, lowest retrieval cost. Best for active data.
- **Cool**: Infrequently accessed data (at least 30 days). Lower storage cost, higher retrieval cost. A minimum storage period of 30 days applies — data deleted before 30 days incurs an early deletion fee.
- **Archive**: Rarely accessed data. Lowest storage cost, highest retrieval cost. Data is **offline** — to read it, you must first **rehydrate** it to Hot or Cool, which takes hours. A minimum storage period of 180 days applies.

Security note: Archived blobs are encrypted the same as Hot/Cool blobs. Rehydration does not change encryption. However, for immutable storage policies, you must rehydrate before the policy expiry check applies.

***

## 2026 Portal Walkthrough — Step by Step

**Goal: Create a user delegation SAS for a blob, create a stored access policy, disable anonymous access**

**Disable anonymous access (container level):**
1. Open your storage account → **Data storage** → **Containers**.
2. Click on a container → **Change access level**.
3. Set to **Private (no anonymous access)**.
4. Click **OK**.

**Create a Stored Access Policy:**
1. Still on the container, click **Access policy** in the left menu.
2. Under **Stored access policies**, click **+ Add policy**.
3. Give it an **Identifier** name (e.g., `uploadpolicy`).
4. Set **Start time**, **Expiry time**, and **Permissions** (e.g., Write, Create).
5. Click **OK** → **Save**.

**Generate a User Delegation SAS for a blob:**
1. Open the container → click on a specific blob.
2. Click the **…** menu → **Generate SAS**.
3. Under **Signing method**, select **User delegation key** (this creates a user delegation SAS).
4. Set **Permissions** (e.g., Read only).
5. Set a short **Expiry** (e.g., 4 hours from now).
6. Set **Allowed protocols** to **HTTPS only**.
7. Optionally set **Allowed IP addresses**.
8. Click **Generate SAS token and URL**. Copy the URL — this is the only time you will see it.

**CLI equivalent:**
```bash
# Generate a user delegation SAS for a blob (requires your Entra ID account to have Storage Blob Data Contributor)
EXPIRY=$(date -u -d "+4 hours" '+%Y-%m-%dT%H:%MZ')

az storage blob generate-sas \
  --account-name myStorage \
  --container-name myContainer \
  --name myfile.txt \
  --permissions r \
  --expiry $EXPIRY \
  --https-only \
  --auth-mode login

# Disable anonymous access on a container
az storage container set-permission \
  --name myContainer \
  --account-name myStorage \
  --public-access off \
  --auth-mode login
```

**PowerShell equivalent:**
```powershell
$ctx = New-AzStorageContext -StorageAccountName "myStorage" -UseConnectedAccount

New-AzStorageBlobSASToken `
  -Context $ctx `
  -Container "myContainer" `
  -Blob "myfile.txt" `
  -Permission "r" `
  -ExpiryTime (Get-Date).AddHours(4) `
  -Protocol HttpsOnly
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| User delegation SAS | SAS backed by Entra ID credentials; revocable | Always prefer this over account/service SAS |
| Service SAS with stored access policy | SAS that can be revoked by deleting the stored policy | When you must use a key-backed SAS but need revocation capability |
| Account SAS | Full-scope SAS backed by account key | Avoid in production; cannot be revoked without key rotation |
| `spr=https` | Forces SAS to only work over HTTPS | Always set this on every SAS token |
| `sip=<IP range>` | Restricts SAS to specific IP addresses | When the consuming client has a known IP (partner systems, on-prem) |
| Anonymous access: Private | Removes all anonymous read access to a container | Always; data should never be publicly readable without intent |
| Blob versioning | Keeps previous blob versions on every write | Protect against accidental overwrites alongside soft delete |
| Change feed | Immutable log of all blob change events | Auditing, compliance, forensic investigation |
| Archive tier | Lowest cost; data is offline; requires rehydration to read | Long-term cold backup storage (180-day minimum retention period) |

***

## Comparison Table

| SAS Type | Backed By | Revocable? | Scope | Best For |
|---|---|---|---|---|
| Account SAS | Storage account key | Only by rotating the key | All services and resource types | Avoid; too broad |
| Service SAS (no policy) | Storage account key | Only by rotating the key | One service, specific resource | Short-lived, low-risk operations |
| Service SAS (with stored access policy) | Storage account key | Yes — delete the stored policy | One service, specific resource | Key-backed SAS where revocation is needed |
| User delegation SAS | Entra ID identity | Yes — revoke RBAC role or delegation key | Blob and Data Lake only | Production; most secure option |

| Access Tier | Retrieval Speed | Min Storage Period | Best For |
|---|---|---|---|
| Hot | Immediate | None | Active, frequently accessed data |
| Cool | Immediate | 30 days | Infrequent access, cost-sensitive |
| Archive | Hours (rehydration) | 180 days | Long-term backup, cold compliance data |

***

## What Happens If You Skip This?

A storage account key embedded in a mobile app or committed to a GitHub repository gives any attacker full, unrestricted access to all storage data forever — until the key is manually rotated. An anonymous container set to public exposes all files to the entire internet. SAS tokens without expiry or IP restrictions become permanent access credentials if intercepted. Without stored access policies, a leaked SAS token cannot be revoked without rotating the account key and breaking every other application that uses it.

***

## AZ-500 Exam Section

- **Trigger Words:** "SAS token", "shared access signature", "user delegation SAS", "stored access policy", "anonymous access", "blob versioning", "change feed", "archive tier", "rehydrate"
- **Common Traps:**
  - Account SAS and service SAS are backed by the storage account key. If the key rotates, those SAS tokens immediately stop working.
  - User delegation SAS is the only SAS type that can be revoked without touching the storage account key.
  - You cannot directly read an archived blob — you must rehydrate it first (hours delay).
  - Stored access policies only work with service SAS, not account SAS or user delegation SAS.
- **How to Tell Apart:**
  - "revoke SAS without rotating key" → stored access policy (service SAS) or user delegation SAS
  - "most secure SAS" → user delegation SAS (Entra ID backed)
  - "anonymous users can read blobs" → anonymous access is enabled — disable it
  - "data cannot be read immediately" → blob is in Archive tier, needs rehydration
- **Likely Question Formats:**
  - You need to give a partner access to download one blob for 2 hours. What is the most secure approach? → user delegation SAS, HTTPS only, short expiry, IP restriction
  - How can you revoke a service SAS token that was issued using the account key, without rotating the key? → use a stored access policy and delete that policy
  - What type of SAS is backed by Entra ID credentials? → user delegation SAS
- **Memorization Tip:** **User Delegation SAS = UD = "Uber Delegated" by Entra ID.** It is the only SAS type tied to your identity — not to a key. Revoke the identity, revoke the SAS.

***

## Pocket Summary (5-Year Refresh)

- Three SAS types: account SAS (broadest, key-backed), service SAS (one service, key-backed), user delegation SAS (Entra ID-backed, most secure, only for Blob and Data Lake).
- Always set `spr=https` on every SAS token; set expiry as short as the use case allows; add `sip` IP restrictions when possible.
- Stored access policies let you revoke a key-backed service SAS without rotating the account key — delete the policy, all referencing tokens are dead.
- Disable anonymous access on all containers unless you are intentionally serving public content.
- Archive tier blobs are encrypted and secure but offline — they require hours-long rehydration before you can read them.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
