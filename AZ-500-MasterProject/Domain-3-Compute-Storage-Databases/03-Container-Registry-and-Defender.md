# Azure Container Registry Security and Microsoft Defender for Containers

> 📌 AZ-500 Exam Objective: Plan and implement security for compute and container services
> 🏷️ Domain: 3 — Secure Compute, Storage, and Databases | Weight: 20–25%

***

## What Is This?

Azure Container Registry (ACR) is a private storage service for Docker container images. Securing ACR means controlling who can push or pull images, scanning those images for known vulnerabilities, and verifying that images have not been tampered with before they run in production.

***

## Why Does This Exist?

A container image is the blueprint for every container that runs. If an attacker can push a malicious image to your registry or if a vulnerable image goes undetected, every container spun from that image is compromised. ACR security and Defender for Containers stop threats before they reach your running workloads.

***

## Who Actually Needs This? (Real-World Context)

A retail e-commerce company in Chicago deploys its checkout service as a container on AKS. The DevOps team pushes new images several times a day. The security team disables the ACR admin user so no one can log in with a shared password. They assign the `AcrPush` role to the CI/CD pipeline's service principal and the `AcrPull` role to the AKS managed identity. Defender for Containers scans every new image and blocks deployment if a critical CVE is found. Content trust ensures only images signed by the release team can run in the production cluster.

***

## How It Works (Pure Concept, No Portal)

**ACR Access Control**
ACR has three ways to control access:

1. **Admin user**: A single username and password tied to the registry. Simple but dangerous — you cannot audit who used it, and the password is shared. Microsoft recommends disabling this immediately.
2. **Azure RBAC**: Assign roles to identities (users, service principals, managed identities).
   - `AcrPull`: read-only, pull images.
   - `AcrPush`: pull and push images.
   - `AcrDelete`: can delete images.
   - `Owner/Contributor`: full management including access control.
3. **Repository-scoped tokens**: Fine-grained tokens that grant access to specific repositories within a registry. Useful for giving a third-party vendor access to only one image, not the whole registry.

**ACR Private Endpoint**
By default, ACR is reachable over the internet. A private endpoint puts ACR on your VNet with a private IP address. You can then disable public network access completely. Images are pushed and pulled over the private network only.

**Image Scanning with Defender for Containers (Qualys)**
When you enable Microsoft Defender for Containers, every image pushed to ACR is automatically scanned for CVEs using the Qualys engine. Scan results appear in Microsoft Defender for Cloud. You can see which packages have vulnerabilities and what the fix version is.

**Content Trust (Image Signing)**
Content trust uses Notary to sign container images. A signed image has a cryptographic signature. The AKS cluster can be configured to only pull images that carry a valid signature from a trusted signer. This prevents "image swap" attacks where someone pushes a malicious image with the same tag.

**Geo-Replication Security**
ACR can replicate images to multiple Azure regions. Each replica shares the same access policies, firewall rules, and private endpoint configuration as the primary registry. You must ensure firewall rules are consistent across all replicas or a region with looser rules becomes a backdoor.

**Defender for Containers — Full Scope**
Defender for Containers does more than image scanning:
- **Vulnerability assessment**: scans ACR images and running container images on AKS nodes.
- **Runtime protection**: watches containers for suspicious behavior — privilege escalation, unusual system calls, cryptomining, reverse shells.
- **Kubernetes hardening**: checks cluster configuration against CIS Kubernetes benchmark.
- **Kubernetes audit log analysis**: detects suspicious API calls like creating cluster-admin role bindings or accessing secrets at scale.

***

## 2026 Portal Walkthrough — Step by Step

**Goal: Create an ACR, disable admin user, enable Defender for Containers, view vulnerability scan**

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for **Container registries** and click **+ Create**.
3. Fill in subscription, resource group, registry name (must be globally unique), location, and SKU (choose **Premium** for private endpoint and geo-replication).
4. Click **Review + create** → **Create**.
5. After deployment, open the registry → **Settings** → **Access keys**.
6. Set **Admin user** to **Disabled**. This removes the shared username/password.
7. Go to **Access control (IAM)** → **+ Add role assignment**.
   - Assign `AcrPush` to your CI/CD pipeline service principal.
   - Assign `AcrPull` to your AKS cluster managed identity.
8. Go to **Networking** → under **Public network access**, select **Disabled**.
9. Click **+ Add** under **Private endpoint connections** to create a private endpoint inside your VNet.
10. To enable Defender: go to **Microsoft Defender for Cloud** → **Environment settings** → select your subscription → **Defender plans** → enable **Containers**.
11. Push an image to the registry: wait 5–10 minutes → go to **Microsoft Defender for Cloud** → **Recommendations** → search "Container registry images should have vulnerability findings resolved" to see scan results.

**CLI equivalent:**
```bash
# Create a Premium ACR
az acr create \
  --resource-group myRG \
  --name myRegistry \
  --sku Premium \
  --admin-enabled false

# Assign AcrPush to a service principal
az role assignment create \
  --assignee <service-principal-id> \
  --role AcrPush \
  --scope $(az acr show --name myRegistry --query id --output tsv)

# Disable public access
az acr update \
  --name myRegistry \
  --public-network-enabled false
```

**PowerShell equivalent:**
```powershell
New-AzContainerRegistry `
  -ResourceGroupName "myRG" `
  -Name "myRegistry" `
  -Sku "Premium" `
  -EnableAdminUser:$false

# Assign role
New-AzRoleAssignment `
  -ObjectId "<service-principal-object-id>" `
  -RoleDefinitionName "AcrPush" `
  -Scope (Get-AzContainerRegistry -ResourceGroupName "myRG" -Name "myRegistry").Id
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/container-registry/container-registry-intro

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Admin user disabled | Removes shared username/password login | Always; use RBAC roles or tokens instead |
| AcrPull role | Allows pulling images; no push or delete | Assign to AKS managed identity, read-only automation |
| AcrPush role | Allows pull and push; no delete | Assign to CI/CD pipeline service principal |
| Repository-scoped tokens | Limits access to a specific repository | Third-party vendors; limited contractor access |
| Private endpoint | Puts registry on a VNet private IP | Production registries; removes internet exposure |
| Public network access disabled | Blocks all internet access to ACR | Use with private endpoint; tightest security |
| Content trust | Requires images to be signed before use | Regulated environments; prevents tampered images from running |
| Defender for Containers | Image CVE scanning + runtime threat detection | All production registries and AKS clusters |
| Geo-replication (Premium) | Copies images to other regions | Multi-region apps; ensure firewall rules match all replicas |

***

## Comparison Table

| Access Method | Security Level | Best For |
|---|---|---|
| Admin user | Low — shared credentials, no audit trail | Development only; should be disabled in production |
| Service principal | Medium — individual identity, rotatable | CI/CD pipelines without managed identity support |
| Managed identity | High — no credential, auto-rotated by Azure | AKS, Azure DevOps, Azure Functions, Logic Apps |
| Repository-scoped token | High — least privilege per repo | Vendor or contractor access to a single image repository |

| Defender Feature | What It Catches |
|---|---|
| Image scanning (Qualys) | Known CVEs in OS packages and application libraries |
| Runtime protection | Active attacks on running containers (privilege escalation, cryptomining) |
| Kubernetes hardening | Misconfigured RBAC, privileged pods, host path mounts |
| Audit log analysis | Suspicious Kubernetes API calls at the management layer |

***

## What Happens If You Skip This?

If the admin user stays enabled and that password leaks, an attacker can push malicious images to every repository in your registry. Without image scanning, a developer can accidentally ship an image with a critical CVE that gets deployed to thousands of containers before anyone notices. Without a private endpoint, your registry is reachable from any IP on the internet, making it a target for credential-stuffing attacks.

***

## AZ-500 Exam Section

- **Trigger Words:** "container registry", "ACR", "admin user", "image scanning", "content trust", "Defender for Containers", "AcrPull", "AcrPush", "private endpoint ACR"
- **Common Traps:**
  - You do not need the admin user for automated pipelines. Use a service principal or managed identity — the exam specifically tests this.
  - Content trust and image scanning are different things. Scanning finds CVEs. Content trust verifies the image was signed by a trusted publisher.
  - Geo-replication copies images but does NOT automatically copy private endpoint or firewall settings — you must configure security settings per replica.
- **How to Tell Apart:**
  - "CI/CD needs to push images without storing passwords" → managed identity + `AcrPush` role
  - "only signed images should run" → content trust (Notary)
  - "scan images for vulnerabilities" → Defender for Containers (Qualys engine)
  - "registry should not be accessible from internet" → private endpoint + disable public access
- **Likely Question Formats:**
  - A build pipeline needs to push Docker images to ACR. What is the most secure authentication method? → managed identity with `AcrPush` role
  - What does disabling the admin user on ACR do? → removes the shared username/password; RBAC and tokens still work
  - How does content trust improve ACR security? → it ensures only cryptographically signed images are pulled, preventing tampered images
- **Memorization Tip:** ACR security = **Disable admin → RBAC roles → Private endpoint → Defender scan → Content trust**. That order also represents increasing security maturity.

***

## Pocket Summary (5-Year Refresh)

- The ACR admin user is a shared password — disable it immediately and use managed identities or service principals with specific RBAC roles (`AcrPull`, `AcrPush`).
- Private endpoints remove public internet access to ACR; combine with disabling public network access for the tightest lockdown.
- Defender for Containers scans images with Qualys for CVEs and monitors running containers for runtime threats.
- Content trust uses image signing (Notary) so only verified images from trusted signers can be deployed.
- Geo-replicated registries share access policies, but private endpoint and firewall rules must be configured separately in each replicated region.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/container-registry/container-registry-intro
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
