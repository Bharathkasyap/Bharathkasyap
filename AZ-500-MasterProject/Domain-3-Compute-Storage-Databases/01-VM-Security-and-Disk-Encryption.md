# Azure VM Security and Disk Encryption

> 📌 AZ-500 Exam Objective: Plan and implement security for compute and container services
> 🏷️ Domain: 3 — Secure Compute, Storage, and Databases | Weight: 20–25%

***

## What Is This?

Azure Virtual Machine (VM) security is a set of tools and settings that protect your cloud servers. Disk encryption scrambles the data on your VM's hard drive so no one can read it without a key. Key Vault stores those encryption keys so they stay safe and separate from the data.

***

## Why Does This Exist?

If a VM's disk is stolen or copied, unencrypted data is readable by anyone. Laws like HIPAA and PCI-DSS require encryption at rest. Azure gives you multiple layers of encryption so you can meet those rules and protect sensitive workloads.

***

## Who Actually Needs This? (Real-World Context)

A hospital network in Texas runs 50 Windows VMs that store patient billing records. Their compliance officer needs proof that every OS and data disk is encrypted before the annual HIPAA audit. The security team enables Azure Disk Encryption on all VMs and stores the keys in a Key Vault that only the billing app's managed identity can access. The auditor gets a single report showing every disk is protected.

***

## How It Works (Pure Concept, No Portal)

**Azure Disk Encryption (ADE)**
ADE installs an encryption extension inside the guest OS. On Windows it uses BitLocker. On Linux it uses DM-Crypt. The encryption key (called a Key Encryption Key, or KEK) lives in Key Vault. The OS disk and data disks are both encrypted before data is written to physical storage.

**Server-Side Encryption (SSE)**
SSE happens at the Azure storage layer — below the OS. Azure encrypts every managed disk automatically. You can let Azure manage the key (Platform-Managed Key, PMK) or bring your own (Customer-Managed Key, CMK) stored in Key Vault.

**Encryption at Host**
Encryption at host extends SSE so that temp disks and OS disk caches are also encrypted. The data is encrypted on the physical host machine before it even reaches Azure Storage.

**Layer Comparison**
- ADE: guest OS encrypts → BitLocker or DM-Crypt → key in Key Vault
- SSE with PMK: Azure Storage encrypts → Azure-managed key → automatic, no setup
- SSE with CMK: Azure Storage encrypts → your Key Vault key → you control rotation
- Encryption at host: host hardware encrypts → covers temp/cache disks too

**Endpoint Protection**
Microsoft Defender for Endpoint integrates with Azure VMs through Microsoft Defender for Servers. It provides antivirus, EDR (endpoint detection and response), and vulnerability scanning.

**Update Management**
Azure Update Manager keeps VMs patched. Unpatched VMs are the top attack vector. You schedule maintenance windows and see compliance status across all VMs in one dashboard.

**Microsoft Defender for Servers**
Defender for Servers (Plan 1 and Plan 2) watches every VM for suspicious behavior — brute-force logins, unusual processes, lateral movement. Plan 2 includes file integrity monitoring and just-in-time VM access.

**Security Baselines**
Microsoft publishes security baselines for Windows Server and Linux. These are hundreds of configuration settings — password policies, audit logging, firewall rules — that harden the OS. Azure Policy can check and enforce these baselines.

**Trusted Launch VMs**
Trusted Launch adds Secure Boot, vTPM, and integrity monitoring to Generation 2 VMs. It prevents rootkits and boot-level malware. It is enabled at VM creation time and cannot be added to existing VMs.

***

## 2026 Portal Walkthrough — Step by Step

**Goal: Enable Azure Disk Encryption on an existing Windows VM**

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for **Virtual machines** and open your target VM.
3. In the left menu under **Settings**, click **Disks**.
4. Click **Additional settings** at the top.
5. Under **Encryption type**, select **Azure Disk Encryption**.
6. In the **Key vault** field, select or create a Key Vault. The Key Vault must have **soft delete** and **purge protection** enabled.
7. Optionally select a **Key encryption key (KEK)** from the Key Vault.
8. Click **Save**. Azure deploys the ADE extension to the VM. This takes 5–15 minutes.
9. Verify: go back to **Disks** → the OS disk and data disks show **Encrypted** under the encryption column.

**CLI equivalent:**
```bash
# Create a Key Vault with required settings
az keyvault create \
  --name myKV \
  --resource-group myRG \
  --location eastus \
  --enable-soft-delete true \
  --enable-purge-protection true

# Enable ADE on the VM
az vm encryption enable \
  --resource-group myRG \
  --name myVM \
  --disk-encryption-keyvault myKV \
  --volume-type All
```

**PowerShell equivalent:**
```powershell
$kv = Get-AzKeyVault -VaultName "myKV" -ResourceGroupName "myRG"

Set-AzVMDiskEncryptionExtension `
  -ResourceGroupName "myRG" `
  -VMName "myVM" `
  -DiskEncryptionKeyVaultUrl $kv.VaultUri `
  -DiskEncryptionKeyVaultId $kv.ResourceId `
  -VolumeType "All" `
  -Force
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/virtual-machines/disk-encryption-overview

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| ADE (BitLocker/DM-Crypt) | Encrypts disks inside the guest OS using Key Vault | When compliance requires OS-level encryption control |
| SSE with PMK | Azure auto-encrypts all managed disks | Default; no action needed; lowest management overhead |
| SSE with CMK | Your Key Vault key controls storage encryption | When regulations require you to own and rotate the key |
| Encryption at host | Encrypts temp disks and caches at the physical host | When you need end-to-end encryption including temp storage |
| Purge protection on Key Vault | Prevents anyone from permanently deleting the vault for 90 days | Required before enabling ADE; prevents accidental data loss |
| Trusted Launch | Secure Boot + vTPM for Gen2 VMs | New VMs in regulated environments; stops boot-level attacks |
| Defender for Servers Plan 2 | Full EDR + JIT VM access + file integrity monitoring | Production servers handling sensitive data |
| Just-in-Time VM access | Blocks RDP/SSH ports by default; opens them on-demand | Any internet-facing VM to cut brute-force attack surface |

***

## Comparison Table

| Feature | ADE | SSE with CMK | Encryption at Host |
|---|---|---|---|
| Where encryption happens | Inside guest OS | Azure Storage layer | Physical host hardware |
| Key location | Key Vault (required) | Key Vault (required) | Key Vault (required) |
| Covers temp disks | No | No | Yes |
| Covers OS disk cache | No | No | Yes |
| Works with unmanaged disks | Yes | No | No |
| Extra setup required | Yes (extension install) | Yes (disk access object) | Yes (enable on subscription) |
| Protects against Azure admin | No | Partially | Partially |

***

## What Happens If You Skip This?

A VM disk is a VHD file sitting in Azure Storage. Without encryption, anyone who can access that storage blob — a rogue admin, a misconfigured access policy, a data center breach — can mount the disk and read everything on it. You will fail PCI-DSS, HIPAA, and ISO 27001 audits. Microsoft's shared responsibility model puts data encryption squarely on the customer.

***

## AZ-500 Exam Section

- **Trigger Words:** "BitLocker", "DM-Crypt", "disk encryption", "CMK", "ADE", "encrypt OS disk", "Key Encryption Key", "KEK"
- **Common Traps:**
  - ADE and SSE both encrypt disks but at different layers. ADE is inside the OS; SSE is at the storage platform. They can run together.
  - Purge protection must be ON before you enable ADE — the exam will test this prerequisite.
  - Trusted Launch cannot be enabled on existing Generation 1 VMs — it is a creation-time setting.
- **How to Tell Apart:**
  - "BitLocker" or "DM-Crypt" → always ADE (guest OS level)
  - "storage platform" or "at rest automatically" → SSE with PMK
  - "bring your own key" or "CMK" → SSE with CMK
  - "temp disk encrypted" → encryption at host
- **Likely Question Formats:**
  - Scenario: a compliance officer needs proof that OS disks are encrypted using a company-owned key → answer: ADE with KEK stored in Key Vault
  - Which encryption type covers temporary disks? → encryption at host
  - What two Key Vault properties are required before enabling ADE? → soft delete + purge protection
- **Memorization Tip:** ADE = **A**pplication layer (inside the OS). SSE = **S**torage layer (below the OS). Host = **H**ardware layer (physical server).

***

## Pocket Summary (5-Year Refresh)

- ADE encrypts VM disks using BitLocker (Windows) or DM-Crypt (Linux) from inside the guest OS; keys live in Key Vault.
- SSE with CMK uses your own Key Vault key at the Azure Storage layer — it is separate from and can layer with ADE.
- Encryption at host covers temp disks and caches that ADE and SSE miss.
- Trusted Launch (Secure Boot + vTPM) blocks boot-level rootkits; enable it at VM creation on Gen2 VMs.
- Key Vault must have soft delete AND purge protection enabled before you can use it with ADE.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/virtual-machines/disk-encryption-overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
