# Azure Key Vault

> 📌 AZ-500 Exam Objective: Plan and implement security for data, including Azure Key Vault
> 🏷️ Domain: 3 — Secure Compute, Storage, and Databases | Weight: 20–25%

***

## What Is This?

Azure Key Vault is a secure, cloud-based safe for secrets, encryption keys, and certificates. Instead of storing passwords and API keys in your application code or configuration files, you store them in Key Vault and retrieve them at runtime. Key Vault uses hardware security modules (HSMs) to protect keys from ever leaving secure hardware.

***

## Why Does This Exist?

Hardcoded secrets in code are the most common cause of data breaches. Developers commit `.env` files to GitHub. Passwords end up in deployment logs. Key Vault eliminates these risks by centralizing secret storage, controlling who can access each secret, logging every access, and rotating credentials automatically.

***

## Who Actually Needs This? (Real-World Context)

A pharmaceutical company in New Jersey builds an application that queries a patient database and calls a third-party lab API. Both connections require credentials. Instead of putting database passwords and API keys in Azure App Service application settings, the development team stores them in Key Vault. The App Service uses a system-assigned managed identity, which has the `Key Vault Secrets User` role. At startup, the app fetches the credentials from Key Vault. The security team gets a complete audit log showing every time any application or person retrieved a secret, from which IP, and at what time.

***

## How It Works (Pure Concept, No Portal)

**Key Vault Objects**

Key Vault stores three types of objects:

1. **Secrets**: Any text value you want to protect — passwords, API keys, connection strings. Versioned automatically when you update a secret. Old versions are retained and accessible.
2. **Keys**: Cryptographic keys used for encryption, decryption, signing, and verification. Key Vault performs cryptographic operations *with* the key — the key material never leaves the vault. Supports RSA and Elliptic Curve keys.
3. **Certificates**: X.509 TLS certificates. Key Vault can create, import, auto-renew, and manage certificate lifecycle. It integrates with DigiCert and GlobalSign as certificate authorities.

**Standard vs Premium SKU**
- **Standard**: Secrets, keys, and certificates backed by software protection.
- **Premium**: Adds HSM-backed keys. The key is generated and stored inside a FIPS 140-2 Level 2 certified hardware security module. The raw key material never exists in software. Required for workloads that mandate HSM key protection.

**Managed HSM**
Managed HSM is a dedicated, single-tenant HSM cluster. Unlike Key Vault Premium (shared infrastructure, HSM-backed keys), Managed HSM gives you a dedicated HSM that you administer. FIPS 140-2 Level 3 certified. Used by financial institutions and governments that cannot share HSM infrastructure with other Azure tenants.

**Authorization: Access Policies vs RBAC**

There are two authorization models:

- **Vault access policies (legacy)**: Assign permissions per identity at the vault level. If you grant a user `Get` on secrets, they can get *any* secret in the vault. You cannot restrict to specific secrets. Up to 1024 access policy assignments per vault. This model is being superseded.

- **Azure RBAC (recommended)**: Use standard Azure role assignments. Built-in roles:
  - `Key Vault Administrator`: full control of vault data plane.
  - `Key Vault Secrets Officer`: create and manage secrets.
  - `Key Vault Secrets User`: read secrets (the role you assign to apps).
  - `Key Vault Crypto Officer`: create and manage keys.
  - `Key Vault Crypto User`: use keys for crypto operations.
  - `Key Vault Certificate Officer`: create and manage certificates.
  - `Key Vault Reader`: read metadata only (not secret values).
  
  RBAC supports scoping to individual secrets, keys, or certificates — not just the vault. This is more granular than access policies.

**Soft Delete and Purge Protection**
- **Soft delete**: When enabled (now on by default for all vaults), deleted secrets/keys/certificates and the vault itself are retained in a soft-deleted state for 7–90 days. You can recover them.
- **Purge protection**: When enabled, even a soft-deleted vault or object cannot be permanently deleted (purged) until the retention period expires. This prevents ransomware or malicious admins from destroying Key Vault permanently. **Required** before using Key Vault with Azure Disk Encryption.

**Key Vault Firewall and Private Endpoints**
By default Key Vault accepts traffic from all networks. You can:
- Set the firewall to **Allow selected networks** and specify IP ranges or VNet service endpoints.
- Add a **private endpoint** so Key Vault gets a private IP on your VNet. Combined with disabling public access, this makes Key Vault only reachable from inside the VNet.
- Check **Allow trusted Microsoft services** so Azure services like Azure Backup, Azure Disk Encryption, and Azure Monitor can still access Key Vault when the firewall is enabled.

**Key Rotation**
You can define a rotation policy on a key in Key Vault. For example, rotate every 90 days. When the key rotates, a new key version is created. Applications referencing the key by name (not version) automatically get the new version. Azure Managed Storage Account Keys lets Key Vault rotate storage account access keys automatically.

**Key Vault Logging**
Diagnostic logs can be sent to Log Analytics, Storage, or Event Hub. Every access — who accessed what secret/key, from which IP, at what time, whether it succeeded or failed — is logged. This log is the audit trail required by SOC 2, PCI-DSS, and HIPAA.

***

## 2026 Portal Walkthrough — Step by Step

**Goal: Create a Key Vault with RBAC model, add a secret, assign Key Vault Secrets User to a managed identity**

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for **Key vaults** and click **+ Create**.
3. Fill in subscription, resource group, vault name (globally unique), and region.
4. Under **Pricing tier**, choose **Standard** (or **Premium** for HSM-backed keys).
5. Under **Permission model**, select **Azure role-based access control (RBAC)**. (This replaces the legacy access policy model.)
6. Leave **Soft delete** enabled (it is on by default). Set retention to 90 days.
7. Check **Enable purge protection**.
8. Click **Review + create** → **Create**.
9. After deployment, open the vault → **Objects** → **Secrets** → **+ Generate/Import**.
10. Set **Upload options** to **Manual**. Enter a **Name** (e.g., `db-password`) and **Value** (the actual password). Click **Create**.
11. Go to **Access control (IAM)** → **+ Add role assignment**.
12. Select the role **Key Vault Secrets User**.
13. Under **Members**, select **Managed identity** and choose your App Service's managed identity.
14. Click **Review + assign**. The app can now read secrets from this vault.
15. To enable logging: go to **Monitoring** → **Diagnostic settings** → **+ Add diagnostic setting**. Check **AuditEvent** and send to a Log Analytics workspace.

**CLI equivalent:**
```bash
# Create Key Vault with RBAC model and purge protection
az keyvault create \
  --resource-group myRG \
  --name myKV \
  --location eastus \
  --enable-rbac-authorization true \
  --enable-soft-delete true \
  --retention-days 90 \
  --enable-purge-protection true

# Add a secret
az keyvault secret set \
  --vault-name myKV \
  --name "db-password" \
  --value "MyS3cureP@ssword"

# Assign Key Vault Secrets User to a managed identity
KV_ID=$(az keyvault show --name myKV --query id --output tsv)
az role assignment create \
  --assignee <managed-identity-object-id> \
  --role "Key Vault Secrets User" \
  --scope $KV_ID
```

**PowerShell equivalent:**
```powershell
# Create Key Vault with RBAC and purge protection
New-AzKeyVault `
  -Name "myKV" `
  -ResourceGroupName "myRG" `
  -Location "eastus" `
  -EnableRbacAuthorization `
  -EnablePurgeProtection `
  -SoftDeleteRetentionInDays 90

# Add a secret
$secretValue = ConvertTo-SecureString "MyS3cureP@ssword" -AsPlainText -Force
Set-AzKeyVaultSecret `
  -VaultName "myKV" `
  -Name "db-password" `
  -SecretValue $secretValue
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/key-vault/general/overview

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| RBAC permission model | Uses Azure role assignments for fine-grained access | Always for new vaults; more granular than access policies |
| Soft delete (default ON) | Retains deleted vaults/objects for 7–90 days | Always; allows recovery from accidental deletion |
| Purge protection | Prevents permanent deletion until retention period expires | Always for production; required for ADE Key Vault |
| Premium SKU | HSM-backed keys | When regulations mandate FIPS 140-2 Level 2 key protection |
| Managed HSM | Dedicated single-tenant HSM cluster | Government, banking, highest-assurance requirements |
| Key rotation policy | Auto-rotates cryptographic keys on a schedule | Encryption keys; comply with key rotation policies |
| Private endpoint | Gives vault a private VNet IP | Production vaults; remove internet reachability |
| Diagnostic logging (AuditEvent) | Logs every read/write/delete operation | Always for production; required for SOC 2, PCI, HIPAA |
| Key Vault Secrets User role | Allows reading secret values; no management rights | Assign to application managed identities |

***

## Comparison Table

| Feature | Access Policies (Legacy) | Azure RBAC (Recommended) |
|---|---|---|
| Granularity | Vault level only | Vault, secret, key, or certificate level |
| Max assignments | 1024 per vault | Thousands (Azure RBAC limits) |
| Scope to specific secret | No | Yes |
| Supports Azure PIM | No | Yes |
| Audit integration | Basic | Full Azure Activity Log + Entra ID logs |

| SKU | Key Protection | HSM Type | Cost |
|---|---|---|---|
| Standard | Software | None | Lower |
| Premium | FIPS 140-2 Level 2 | Shared HSM | Medium |
| Managed HSM | FIPS 140-2 Level 3 | Dedicated single-tenant HSM | Highest |

***

## What Happens If You Skip This?

Without Key Vault, secrets live in application settings, environment variables, or source code. Each place is a potential leak point. A developer accidentally pushes a `.env` file — credentials are on GitHub in seconds and scraped by bots within minutes. Without purge protection, a ransomware attack that gains Key Vault admin rights can delete every secret permanently. Without logging, you have no audit trail to tell you when, by whom, and from where your secrets were accessed.

***

## AZ-500 Exam Section

- **Trigger Words:** "HSM", "purge protection", "soft delete", "customer-managed key", "Key Vault access policy vs RBAC", "Managed HSM", "key rotation", "Key Vault Secrets User", "private endpoint Key Vault"
- **Common Traps:**
  - Access policies and RBAC are mutually exclusive permission models on a vault. You pick one. RBAC is the recommended model.
  - Soft delete is now on by default but purge protection is NOT — you must explicitly enable purge protection.
  - Key Vault Standard uses software-protected keys. Premium uses HSM-backed keys. The exam may describe a requirement for "HSM-backed keys" — answer is Premium SKU or Managed HSM.
  - "Customer-managed key" (CMK) for services like Storage or SQL means those services store their encryption key in YOUR Key Vault. You own and control the key.
- **How to Tell Apart:**
  - "app needs to read a secret without storing passwords" → managed identity + Key Vault Secrets User role
  - "prevent permanent deletion of Key Vault" → purge protection
  - "recover an accidentally deleted secret" → soft delete (recover from soft-deleted state)
  - "FIPS 140-2 Level 3, dedicated hardware" → Managed HSM
  - "restrict Key Vault access to a single VNet" → private endpoint + disable public access
- **Likely Question Formats:**
  - What role should you assign to an App Service managed identity that needs to read secrets from Key Vault? → Key Vault Secrets User
  - What two properties must be enabled on a Key Vault before it can be used with Azure Disk Encryption? → soft delete + purge protection
  - What is the difference between Key Vault access policies and RBAC? → access policies = vault-level only, up to 1024 entries; RBAC = can scope to individual secrets/keys/certs, supports PIM
- **Memorization Tip:** Key Vault objects: **SKC** — **S**ecrets (passwords), **K**eys (encryption), **C**ertificates (TLS). SKUs: **Standard** = software, **Premium** = HSM-backed, **Managed HSM** = your own dedicated hardware.

***

## Pocket Summary (5-Year Refresh)

- Key Vault stores secrets (passwords/API keys), keys (cryptographic), and certificates (TLS) — the raw key material for HSM-backed keys never leaves the HSM.
- Use the RBAC permission model (not legacy access policies); assign the `Key Vault Secrets User` role to application managed identities.
- Enable soft delete (default ON) and purge protection (must be manually enabled) — purge protection is required for Azure Disk Encryption use.
- Key Vault Premium provides HSM-backed keys (FIPS 140-2 Level 2); Managed HSM provides a dedicated single-tenant HSM (FIPS 140-2 Level 3).
- Enable diagnostic logging to Log Analytics for a full audit trail of every secret, key, and certificate access.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/key-vault/general/overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
