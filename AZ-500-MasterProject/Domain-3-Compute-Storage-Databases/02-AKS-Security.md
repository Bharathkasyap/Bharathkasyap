# Azure Kubernetes Service (AKS) Security

> 📌 AZ-500 Exam Objective: Plan and implement security for compute and container services
> 🏷️ Domain: 3 — Secure Compute, Storage, and Databases | Weight: 20–25%

***

## What Is This?

Azure Kubernetes Service (AKS) is a managed Kubernetes platform that runs containerized applications. AKS security means locking down who can access the cluster, how containers talk to each other, how images are verified, and how secrets are kept out of container code.

***

## Why Does This Exist?

Kubernetes clusters run many workloads and hold service credentials. A misconfigured cluster can expose internal APIs, let containers escape to the host, or allow one compromised pod to attack all others. AKS security controls shrink that attack surface at every layer.

***

## Who Actually Needs This? (Real-World Context)

A fintech startup in London runs a payment processing platform on AKS. Each microservice must only access its own database credentials. The security team enables workload identity so pods never store secrets in environment variables. They add a Calico network policy so the payment pod can only talk to the database pod — not to the logging or email pods. Microsoft Defender for Containers scans every image before it runs and alerts the team if a new CVE appears in a deployed container.

***

## How It Works (Pure Concept, No Portal)

**Kubernetes RBAC vs Azure RBAC**
These are two separate authorization systems that can both be active on the same cluster.
- **Kubernetes RBAC**: controls what you can do *inside* the cluster (create pods, list secrets, delete deployments). Defined by `Role`, `ClusterRole`, `RoleBinding`, `ClusterRoleBinding` objects in Kubernetes.
- **Azure RBAC**: controls who can manage the AKS resource in Azure (scale the cluster, get credentials, upgrade the version). Defined by Azure role assignments like `Azure Kubernetes Service Cluster Admin`.

You can configure AKS to use Azure AD for Kubernetes RBAC. This means your corporate identities control pod-level permissions — no separate Kubernetes user database to manage.

**Network Policies (Calico / Azure CNI)**
By default every pod can reach every other pod in a cluster. Network policies are Kubernetes firewall rules between pods. Two providers:
- **Azure network policy**: built-in, simpler, limited features.
- **Calico**: open-source, richer rule set, supports more scenarios.

Network policies must be enabled at cluster creation. You cannot add them to an existing cluster.

**Pod Identity and Workload Identity**
Pods sometimes need to call Azure services (Key Vault, Storage, SQL). You should never put a service principal password in a container image or environment variable.
- **Workload Identity** (recommended, GA): binds a Kubernetes service account to an Azure Managed Identity using federated credentials. Pods automatically get an Azure token with no passwords stored anywhere.
- **Pod Identity** (deprecated): older approach using an AKS add-on. Workload Identity replaced it.

**Key Vault CSI Driver**
The Secrets Store CSI Driver mounts Key Vault secrets as files or environment variables inside pods. The secrets are fetched at runtime, never baked into the image.

**Image Scanning**
Microsoft Defender for Containers scans container images stored in Azure Container Registry (ACR). It uses vulnerability data to find known CVEs. It also scans running containers in AKS nodes.

**Private AKS Cluster**
A private cluster has no public API server endpoint. The Kubernetes API is only reachable over a private endpoint inside your VNet. No kubectl from the internet — access requires being on the VNet or a connected network.

**Cluster Auto-Upgrade**
AKS releases new Kubernetes versions regularly. Auto-upgrade channels (patch, stable, rapid, node-image) keep the cluster on a secure version automatically.

**Microsoft Defender for Containers**
Defender for Containers watches:
- Kubernetes audit logs for suspicious API calls
- Running containers for runtime threats (privilege escalation, cryptomining)
- Image vulnerability assessments
- Kubernetes configuration against hardening benchmarks (CIS)

***

## 2026 Portal Walkthrough — Step by Step

**Goal: Create a private AKS cluster with Defender for Containers and Key Vault CSI driver**

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Search for **Kubernetes services** and click **+ Create** → **Kubernetes cluster**.
3. **Basics tab**: choose subscription, resource group, cluster name, region, and Kubernetes version.
4. **Node pools tab**: keep the default system node pool.
5. **Networking tab**:
   - Set **Network configuration** to **Azure CNI** (required for network policies).
   - Under **Network policy**, select **Calico** or **Azure**.
   - Under **Cluster connectivity**, check **Private cluster**. This disables the public API endpoint.
6. **Integrations tab**:
   - Under **Microsoft Defender for Cloud**, enable **Defender for Containers**.
   - Under **Azure Key Vault Secrets Provider**, check **Enable secrets store CSI driver**.
7. **Identity tab**: choose **System-assigned managed identity**.
8. Click **Review + create** → **Create**.
9. After deployment, go to the cluster → **Security** → **Defender for Cloud** to view image scan results.

**CLI equivalent:**
```bash
# Create the cluster
az aks create \
  --resource-group myRG \
  --name myAKS \
  --enable-private-cluster \
  --enable-managed-identity \
  --network-plugin azure \
  --network-policy calico \
  --enable-addons azure-keyvault-secrets-provider \
  --enable-defender \
  --generate-ssh-keys

# Enable workload identity
az aks update \
  --resource-group myRG \
  --name myAKS \
  --enable-workload-identity \
  --enable-oidc-issuer
```

**PowerShell equivalent:**
```powershell
New-AzAksCluster `
  -ResourceGroupName "myRG" `
  -Name "myAKS" `
  -EnablePrivateCluster `
  -NetworkPlugin "azure" `
  -NetworkPolicy "calico" `
  -EnableManagedIdentity
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/aks/concepts-security

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| Private cluster | Removes public API server endpoint | Always for production; prevents internet-based attacks on the API |
| Azure AD integration | Uses corporate identities for Kubernetes RBAC | Any enterprise cluster; eliminates separate Kubernetes user management |
| Network policy (Calico/Azure) | Firewalls between pods | Any cluster with multiple workloads that should be isolated |
| Workload Identity | Gives pods an Azure identity with no passwords | Whenever a pod needs to access Azure services |
| Key Vault CSI driver | Mounts secrets from Key Vault into pods at runtime | Replace hardcoded secrets or environment variable passwords |
| Defender for Containers | Runtime protection + image scanning + Kubernetes hardening | All production clusters; gives CVE alerts and runtime threat detection |
| Auto-upgrade channel | Automatically applies Kubernetes version patches | All production clusters to stay on supported, patched versions |
| Cluster admin role | Azure RBAC role to get cluster credentials | Break-glass admin access only; scope to specific people |

***

## Comparison Table

| Feature | Kubernetes RBAC | Azure RBAC for AKS |
|---|---|---|
| What it controls | Actions inside the cluster (pods, services, secrets) | Azure management plane (scale, upgrade, get credentials) |
| Identity source | Kubernetes users / Azure AD (if integrated) | Azure AD |
| Where rules are stored | Kubernetes etcd | Azure Resource Manager |
| Granularity | Namespace-level or cluster-level | Subscription/RG/resource level |
| Use together? | Yes — they complement each other | Yes — they complement each other |

| Network Policy | Azure | Calico |
|---|---|---|
| Complexity | Simple | Advanced |
| Extra protocols | TCP/UDP only | TCP, UDP, ICMP, named ports |
| Best for | Basic east-west isolation | Complex microsegmentation |

***

## What Happens If You Skip This?

Without network policies, a compromised pod can reach every other pod in the cluster. Without workload identity, developers put service principal secrets in image layers or environment variables — attackers who get one container get full Azure access. Without Defender for Containers, a vulnerable image can run for months with no alert. A public API server lets attackers probe the Kubernetes API from anywhere on the internet.

***

## AZ-500 Exam Section

- **Trigger Words:** "private cluster", "pod identity", "workload identity", "network policy", "Calico", "image scanning", "Defender for Containers", "Key Vault CSI"
- **Common Traps:**
  - Kubernetes RBAC and Azure RBAC are separate — enabling one does not replace the other.
  - Pod Identity is deprecated; workload identity is the current answer.
  - Network policies must be chosen at cluster creation — you cannot add them later.
  - A private cluster still needs a jump box, VPN, or ExpressRoute to run kubectl.
- **How to Tell Apart:**
  - "access Azure services from a pod without passwords" → workload identity
  - "block pod-to-pod traffic" → network policy
  - "no public Kubernetes API" → private cluster
  - "image CVE" or "runtime container threat" → Defender for Containers
  - "mount secrets into pod from Key Vault" → Key Vault CSI driver
- **Likely Question Formats:**
  - How do you prevent a pod from accessing another pod's network traffic? → network policy
  - A pod needs to read a Key Vault secret — what is the recommended approach? → workload identity + Key Vault CSI driver
  - What is the difference between Azure RBAC and Kubernetes RBAC on AKS? → Azure RBAC = management plane; Kubernetes RBAC = data plane inside the cluster
- **Memorization Tip:** Think of AKS security in four zones: **Identity** (workload identity), **Network** (network policy + private cluster), **Image** (image scanning), **Runtime** (Defender for Containers).

***

## Pocket Summary (5-Year Refresh)

- Kubernetes RBAC controls actions inside the cluster; Azure RBAC controls managing the AKS resource — they are separate and both can be active.
- Use workload identity to give pods an Azure identity with no stored passwords; the Key Vault CSI driver mounts secrets at runtime.
- Network policies (Calico or Azure CNI) create pod-to-pod firewall rules; they must be enabled at cluster creation.
- A private cluster removes the public API server endpoint so kubectl is only reachable from inside the VNet.
- Microsoft Defender for Containers scans images for CVEs, monitors runtime behavior, and checks Kubernetes configuration against hardening benchmarks.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/aks/concepts-security
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
