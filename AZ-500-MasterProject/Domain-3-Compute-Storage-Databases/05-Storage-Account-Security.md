# Azure Storage Account Security

> 📌 AZ-500 Exam Objective: Plan and implement security for Azure Storage
> 🏷️ Domain: 3 — Secure Compute, Storage, and Databases | Weight: 20–25%

***

## What Is This?

An Azure Storage Account holds blobs, files, queues, and tables. Storage security controls who can access your data, how that access is granted, what happens to data when it is deleted, and how the data is protected from tampering. Getting this right is the foundation of cloud data security.

***

## Why Does This Exist?

Storage accounts hold everything from database backups to medical images to financial records. A misconfigured storage account with a leaked access key or open public access has been the cause of some of the largest cloud data breaches. Azure provides multiple overlapping security controls so you can enforce least-privilege access and meet regulatory requirements.

***

## Who Actually Needs This? (Real-World Context)

An insurance company in Atlanta stores claim documents and underwriting data in Azure Blob Storage. Regulators require that documents cannot be altered or deleted for 7 years. The security team enables immutable storage (WORM) on the container so no one — not even a storage account owner — can delete or overwrite documents during the retention period. They also disable public access, enforce TLS 1.2, rotate storage account keys monthly, and enable Microsoft Defender for Storage to detect anomalous access like an IP from a foreign country downloading thousands of files at 2 AM.

***

## How It Works (Pure Concept, No Portal)

**Access Control Methods**
Four ways to grant access to a storage account:

1. **Storage account keys**: Two 512-bit keys that grant full, unrestricted access to everything in the account. Anyone with a key is effectively an owner. Keys should be rotated regularly and ideally replaced with RBAC or SAS.
2. **Azure RBAC**: Assign Azure roles (`Storage Blob Data Reader`, `Storage Blob Data Contributor`, etc.) to identities. Uses Entra ID tokens. Provides audit logs and per-identity control. This is the preferred method.
3. **Shared Access Signatures (SAS)**: Time-limited, permission-scoped URLs that grant access without sharing the full key. Covered in detail in the next file (06-Blob-SAS-Access-Tiers.md).
4. **Entra ID (OAuth2)**: Applications and users get short-lived tokens from Entra ID. Combined with RBAC roles, this is the most secure and auditable access method.

**Disabling Shared Key Authorization**
You can disable shared key access entirely. This forces all access through Entra ID (RBAC or delegated). Even the portal uses OAuth tokens in this mode. Note: SAS tokens backed by account keys also stop working. Only user delegation SAS (backed by Entra ID) continues to work.

**Network Access Controls**
By default, storage accounts accept traffic from all networks. You can restrict this with:
- **Storage firewall**: Allow only specific IP ranges or Azure VNet subnets.
- **Trusted Microsoft services**: Some Azure services (Azure Backup, Event Grid, Azure Monitor) need access from Microsoft's own IP ranges. Checking "Allow trusted Microsoft services" permits these services while keeping the firewall closed to others.
- **Private endpoints**: Assigns the storage account a private IP on your VNet. Combined with disabling public access, this makes the storage account invisible from the internet.

**Infrastructure Encryption**
Azure Storage encrypts all data at rest by default using AES-256 (SSE with platform-managed keys). For double encryption, you can enable infrastructure encryption at the storage account level. This applies a second, independent layer of encryption. Used in highly regulated environments where single encryption is not sufficient.

**Microsoft Defender for Storage**
Defender for Storage analyzes access logs in near-real-time and alerts on:
- Access from unusual locations (TOR, uncommon countries)
- Anomalous data exfiltration (sudden spike in data downloaded)
- Permission changes that expose data publicly
- Potential malware uploaded to blob storage (hash reputation check)
- Brute-force attempts on SAS tokens

**Soft Delete**
Soft delete keeps deleted blobs (and containers) in a hidden, recoverable state for a configurable number of days (1–365). If a user or ransomware deletes a file, you can restore it within the retention window. Soft delete is available for blobs, containers, and file shares.

**Immutable Storage (WORM — Write Once, Read Many)**
Immutable blob storage prevents any modification or deletion of a blob for a defined period. Two policy types:
- **Time-based retention policy**: Data cannot be deleted or modified until the retention period expires.
- **Legal hold**: Data cannot be deleted or modified until the legal hold tag is explicitly removed.

Once a policy is locked, even the storage account owner cannot delete blobs or shorten the retention period. Used for compliance with SEC Rule 17a-4, FINRA, and HIPAA.

**Storage Account Key Rotation**
Keys should be rotated regularly. You can configure key rotation reminders in Key Vault. Azure Key Vault can store and automatically rotate storage account keys, giving applications a Key Vault secret reference instead of the key itself.

**TLS Minimum Version**
You can set the minimum TLS version for the storage account endpoint to 1.2. Clients using TLS 1.0 or 1.1 are rejected.

***

## 2026 Portal Walkthrough — Step by Step

**Goal: Disable public access, configure firewall, enable Defender for Storage, enable soft delete**

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for **Storage accounts** and open your account.
3. **Disable public blob access:**
   - Under **Settings**, click **Configuration**.
   - Set **Allow Blob public access** to **Disabled**.
   - Set **Minimum TLS version** to **Version 1.2**.
   - Click **Save**.
4. **Configure network firewall:**
   - Under **Security + networking**, click **Networking**.
   - Under **Public network access**, select **Enabled from selected virtual networks and IP addresses**.
   - Under **Firewall**, add your on-premises IP ranges.
   - Under **Virtual networks**, add the subnet of the VMs that need access.
   - Check **Allow Azure services on the trusted services list to access this storage account**.
   - Click **Save**.
5. **Enable Defender for Storage:**
   - Under **Security + networking**, click **Microsoft Defender for Cloud**.
   - Click **Enable Microsoft Defender for Storage**. (Or enable at the subscription level via Defender for Cloud plans for full coverage.)
6. **Enable soft delete for blobs:**
   - Under **Data management**, click **Data protection**.
   - Check **Enable soft delete for blobs**.
   - Set retention days (e.g., 30 days).
   - Check **Enable soft delete for containers** as well.
   - Click **Save**.

**CLI equivalent:**
```bash
# Deny all public network access by default
az storage account update \
  --resource-group myRG \
  --name myStorage \
  --default-action Deny \
  --bypass AzureServices

# Add allowed subnet
az storage account network-rule add \
  --resource-group myRG \
  --account-name myStorage \
  --vnet-name myVNet \
  --subnet mySubnet

# Enforce TLS 1.2
az storage account update \
  --resource-group myRG \
  --name myStorage \
  --min-tls-version TLS1_2

# Disable public blob access
az storage account update \
  --resource-group myRG \
  --name myStorage \
  --allow-blob-public-access false
```

**PowerShell equivalent:**
```powershell
# Deny public access and enforce TLS 1.2
Set-AzStorageAccount `
  -ResourceGroupName "myRG" `
  -Name "myStorage" `
  -DefaultAction Deny `
  -Bypass AzureServices `
  -MinimumTlsVersion TLS1_2 `
  -AllowBlobPublicAccess $false
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/storage/common/storage-security-guide

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Allow Blob public access: Disabled | Prevents any container from being set to anonymous access | Always in production; data should never be world-readable without intent |
| Default action: Deny | Blocks all network traffic by default; requires explicit allow rules | All production storage accounts |
| Minimum TLS version: 1.2 | Rejects connections using TLS 1.0 or 1.1 | Always; TLS 1.0/1.1 are deprecated and insecure |
| Trusted Microsoft services | Allows Azure platform services (Backup, Monitor) through the firewall | When Azure platform services need access alongside a restricted firewall |
| Disable shared key authorization | Forces all access through Entra ID; keys and key-based SAS stop working | High-security environments; requires apps to use Entra ID auth |
| Soft delete (blobs/containers) | Recovers deleted data within the retention period | Always; protects against accidental deletion and ransomware |
| Immutable storage (WORM) | Prevents modification or deletion for a locked retention period | Compliance with SEC, FINRA, HIPAA; legal holds on evidence |
| Defender for Storage | Detects anomalous access, malware uploads, exfiltration | All storage accounts with sensitive data |
| Infrastructure encryption | Adds a second layer of AES-256 encryption at storage platform | Double-encryption requirement in regulated industries |

***

## Comparison Table

| Access Method | Security Level | Revocable Immediately? | Audit Trail? |
|---|---|---|---|
| Storage account key | Low — full access, no scope | Only by rotating the key | No per-operation logs |
| SAS token (account key) | Medium — scoped but key-backed | Only by rotating the key | Limited |
| SAS token (user delegation) | High — Entra ID backed | Yes — revoke the Entra ID identity | Yes |
| Azure RBAC (Entra ID) | Highest — role-scoped, token-based | Yes — remove role assignment | Yes — full audit logs |

| Data Protection Feature | Protects Against |
|---|---|
| Soft delete | Accidental deletion, ransomware encryption followed by delete |
| Immutable storage (time-based) | Modification or deletion during retention period; even by admins |
| Legal hold | Deletion during litigation; holds until tag is removed |
| Versioning | Accidental overwrites; keeps every previous version |
| Defender for Storage | Anomalous access, malware, credential attacks |

***

## What Happens If You Skip This?

A storage account with default settings and a public access enabled is an open book. If the access key leaks through a GitHub commit or a misconfigured application, an attacker gets full read/write access to all blobs, files, queues, and tables. Without soft delete, a disgruntled employee or ransomware can permanently delete data in seconds. Without Defender for Storage, mass exfiltration can happen for days or weeks before anyone notices in log files.

***

## AZ-500 Exam Section

- **Trigger Words:** "storage firewall", "soft delete", "WORM", "immutable storage", "Defender for Storage", "shared key", "disable shared key", "infrastructure encryption", "TLS 1.2", "trusted Microsoft services"
- **Common Traps:**
  - Disabling shared key authorization does NOT lock you out. Entra ID RBAC still works. Only key-based auth and key-based SAS tokens stop working.
  - Soft delete and immutable storage are different. Soft delete is reversible recovery. Immutable storage is irreversible prevention — no one can delete until the period expires.
  - "Trusted Microsoft services" is a firewall bypass for Azure platform services — it is not a general trust setting.
  - Immutable policies that are **locked** cannot be shortened or removed — even by the subscription owner. Unlocked policies can be changed.
- **How to Tell Apart:**
  - "recover deleted file" → soft delete
  - "no one can delete — not even the admin" → immutable storage with locked policy
  - "all access must use Entra ID" → disable shared key authorization
  - "unusual download spike from foreign IP" → Defender for Storage alert
- **Likely Question Formats:**
  - What happens when you disable shared key authorization on a storage account? → key-based access and key-based SAS stop working; Entra ID RBAC and user delegation SAS still work
  - A compliance requirement says records cannot be deleted for 7 years. What do you enable? → time-based retention policy on immutable blob storage, locked
  - How do you protect against ransomware deleting blobs? → soft delete (recoverable), or immutable storage (prevents deletion entirely)
- **Memorization Tip:** Storage security in layers: **Network** (firewall) → **Identity** (disable shared key, use RBAC) → **Data protection** (soft delete, WORM) → **Threat detection** (Defender for Storage).

***

## Pocket Summary (5-Year Refresh)

- Use Azure RBAC and Entra ID for storage access instead of account keys; disable shared key authorization for the highest security posture.
- Set the storage firewall default action to Deny and add only needed subnets, IPs, and trusted Azure services.
- Enable soft delete for blobs and containers to protect against accidental or malicious deletion with a configurable recovery window.
- Immutable storage (WORM) with a locked time-based retention policy prevents anyone — including admins — from deleting or modifying data until the period expires.
- Microsoft Defender for Storage watches for anomalous behavior like mass downloads, access from unexpected locations, and potential malware uploads.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/storage/common/storage-security-guide
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
