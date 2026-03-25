# Azure Network Watcher and NSG Flow Logs

> 📌 AZ-500 Exam Objective: Configure and manage network security monitoring
> 🏷️ Domain: 2 — Secure Networking | Weight: 20–25%

***

## What Is This?

Azure Network Watcher is a set of diagnostic and monitoring tools for your Azure network. It lets you see what traffic is flowing, test whether two resources can talk to each other, capture packets from a VM's NIC, and diagnose VPN issues. NSG Flow Logs record every allowed and denied connection through your Network Security Groups and store the data so you can analyze network patterns and security events.

***

## Why Does This Exist?

When something breaks on your network — or when a security incident happens — you need evidence. Which IP was talking to which port? Did the NSG block it or allow it? Was that connection normal or suspicious? Without Network Watcher and Flow Logs, you are flying blind. These tools give you the equivalent of network camera footage and a flight data recorder for your Azure infrastructure.

***

## Who Actually Needs This? (Real-World Context)

A technology startup's security team gets an alert that a VM is sending unusual amounts of outbound traffic to an unknown IP in Eastern Europe. They use Network Watcher's **packet capture** to record the live traffic from that VM's NIC. They check **NSG Flow Logs** to see the full history of connections over the past week. They use **Traffic Analytics** to visualize the connection map and confirm the VM is part of a botnet. They isolate the VM, file an incident report, and use the flow logs as evidence for their cyber insurance claim.

***

## How It Works (Pure Concept, No Portal)

**Network Watcher** is a regional service. It is enabled automatically when you create a VNet in a region. It provides:

- **IP Flow Verify:** Tests whether a specific inbound or outbound packet would be allowed or denied by the NSG rules on a VM's NIC. You specify: VM, direction, source IP, destination IP, port, protocol. It returns "Allow" or "Deny" and names the specific NSG rule that made the decision. Great for troubleshooting connectivity quickly.

- **Connection Monitor:** Continuously monitors end-to-end connectivity between resources. You set up a test between a source (VM or Log Analytics agent) and a destination (IP, FQDN, or Azure resource). It checks latency, packet loss, and reachability on a schedule. Alerts when connectivity degrades.

- **Packet Capture:** Captures raw network packets from a VM's NIC. Stores captures in a storage account or locally on the VM. Useful for deep inspection of what is actually in the traffic. Requires the Network Watcher extension on the VM.

- **VPN Diagnostics:** Runs diagnostics on VPN Gateways and connections. Stores results in a storage account. Useful for troubleshooting S2S VPN connection failures.

- **Topology:** Generates a visual map of all resources in a VNet — VMs, NICs, NSGs, subnets, peerings, gateways. Useful for understanding complex network layouts.

- **Next Hop:** For a given VM and destination IP, shows which routing path the traffic would take. Helps diagnose asymmetric routing or misrouted traffic.

**NSG Flow Logs:**
- Records every network flow (connection tuple) that is evaluated by an NSG rule.
- Captures: timestamp, source IP, destination IP, source port, destination port, protocol, direction, allow/deny, bytes, packets.
- Stored in a **storage account** in JSON format.
- **Version 2** (current) includes byte and packet counts per flow. Version 1 only records whether allowed or denied.
- NSG Flow Logs have a **1–10 minute delay** — they are NOT real-time. Data is collected in batches.
- Retention policy: you set a retention period (1–365 days) in the storage account. After that, logs are deleted automatically.
- Flow logs are enabled per NSG — not per subnet or VM.

**Traffic Analytics:**
- Built on top of NSG Flow Logs. Processes the logs through a **Log Analytics workspace** and visualizes traffic patterns on a geographic map.
- Shows: top talkers, malicious flows (matched against Microsoft threat intelligence), open ports, VMs communicating with risky IPs, flow trends over time.
- Requires a Log Analytics workspace and adds a small processing cost per GB of flow log data.
- Processing interval: 10 minutes or 60 minutes (configurable). Not real-time.

***

## 2026 Portal Walkthrough — Step by Step

**Enable NSG Flow Logs and set up Traffic Analytics:**

1. Go to **portal.azure.com** → search **"Network Watcher"** → open it (make sure you are in the right region).
2. Left menu → **NSG flow logs** → click **+ Create**.
3. **Basics tab:**
   - **Flow log type**: `Network security group`
   - **NSG**: Select your NSG (e.g., `nsg-web-prod`)
   - **Storage account**: Select or create a storage account in the same region
   - **Retention (days)**: `30` (or more for compliance requirements)
   - **Flow log version**: `Version 2`
   → **Next: Analytics**.
4. **Analytics tab:**
   - Toggle **Enable Traffic Analytics**: `On`
   - **Log Analytics workspace**: Select or create one
   - **Traffic Analytics processing interval**: `Every 10 mins`
   → **Review + create** → **Create**.
5. View results: Open Network Watcher → **Traffic Analytics** → interactive map appears after ~30 minutes of data collection.

**Run IP Flow Verify (troubleshoot connectivity):**

1. Network Watcher → left menu → **IP flow verify**.
2. Fill in:
   - **Virtual machine**: Select the VM
   - **Network interface**: Select the NIC
   - **Protocol**: `TCP`
   - **Direction**: `Inbound`
   - **Local IP address**: VM's private IP
   - **Local port**: `443`
   - **Remote IP address**: `203.0.113.50` (the source you want to test)
   - **Remote port**: `54321`
   → click **Check**. Result shows Allow or Deny plus the rule name.

**Enable packet capture:**

1. Network Watcher → **Packet capture** → **+ Add**.
2. Select VM, set capture file name, max bytes, max packets, time limit (seconds).
3. Storage: select storage account or VM local path → **Start packet capture**.

**CLI:**
```bash
# Enable NSG Flow Logs (Version 2) with Traffic Analytics
az network watcher flow-log create \
  --resource-group rg-prod \
  --name fl-nsg-web-prod \
  --nsg nsg-web-prod \
  --storage-account mystorageaccount \
  --enabled true \
  --format JSON \
  --log-version 2 \
  --retention 30 \
  --traffic-analytics true \
  --workspace myloganalyticsworkspace \
  --interval 10 \
  --location eastus

# Run IP flow verify
az network watcher test-ip-flow \
  --resource-group rg-prod \
  --vm vm-web-01 \
  --direction Inbound \
  --protocol TCP \
  --local-ip 10.0.1.4 \
  --local-port 443 \
  --remote-ip 203.0.113.50 \
  --remote-port 54321

# Capture packets from a VM
az network watcher packet-capture create \
  --resource-group rg-prod \
  --vm vm-web-01 \
  --name capture-web01 \
  --storage-account mystorageaccount \
  --time-limit 300
```

**PowerShell:**
```powershell
# Get Network Watcher for the region
$nw = Get-AzNetworkWatcher -ResourceGroupName "NetworkWatcherRG" -Name "NetworkWatcher_eastus"

# Get NSG and Storage Account
$nsg = Get-AzNetworkSecurityGroup -Name "nsg-web-prod" -ResourceGroupName "rg-prod"
$storageAccount = Get-AzStorageAccount -Name "mystorageaccount" -ResourceGroupName "rg-prod"
$workspace = Get-AzOperationalInsightsWorkspace -Name "myloganalyticsworkspace" -ResourceGroupName "rg-prod"

# Enable NSG Flow Logs with Traffic Analytics
Set-AzNetworkWatcherFlowLog `
  -NetworkWatcher $nw `
  -TargetResourceId $nsg.Id `
  -StorageId $storageAccount.Id `
  -EnableFlowLog $true `
  -FormatVersion 2 `
  -EnableRetention $true `
  -RetentionPolicyDays 30 `
  -EnableTrafficAnalytics $true `
  -WorkspaceResourceId $workspace.ResourceId `
  -WorkspaceGUID $workspace.CustomerId `
  -WorkspaceLocation $workspace.Location `
  -TrafficAnalyticsInterval 10

# Run IP flow verify
Test-AzNetworkWatcherIPFlow `
  -NetworkWatcher $nw `
  -TargetVirtualMachineId (Get-AzVM -Name "vm-web-01" -ResourceGroupName "rg-prod").Id `
  -Direction Inbound `
  -Protocol TCP `
  -RemoteIPAddress "203.0.113.50" `
  -LocalIPAddress "10.0.1.4" `
  -LocalPort 443 `
  -RemotePort 54321
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/network-watcher/

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| NSG Flow Log Version 2 | Captures byte and packet counts in addition to allow/deny. More detailed than Version 1. | Always use Version 2 for new deployments |
| Retention (days) | How long flow log data stays in the storage account before auto-deletion | Set to 30+ days for operational use. 90+ days for security/compliance. |
| Traffic Analytics enabled | Processes flow logs through Log Analytics and visualizes traffic on a map | Enable for security monitoring and anomaly detection |
| Traffic Analytics interval | 10 minutes or 60 minutes. Controls how often logs are processed into Analytics. | Use 10 minutes for near-real-time insights (adds cost). 60 min for cost savings. |
| IP Flow Verify | Tests whether a hypothetical packet would be allowed or denied. Point-in-time test. | Use first when troubleshooting connectivity — gives instant NSG rule diagnosis |
| Connection Monitor | Continuous, scheduled connectivity testing between two endpoints | Use for ongoing monitoring of critical application paths |
| Packet Capture | Captures raw packets from a VM NIC. Stored in storage account. | Use for deep security investigation or application debugging |
| VPN Diagnostics | Runs checks on VPN gateways and connections | Use when a Site-to-Site VPN connection is down or flapping |

***

## Comparison Table

| Tool | What It Answers | Real-time? | Requires Storage? |
|---|---|---|---|
| IP Flow Verify | "Would this specific packet be allowed or denied right now?" | Yes | No |
| NSG Flow Logs | "What traffic has passed through this NSG over time?" | No (1–10 min delay) | Yes |
| Traffic Analytics | "What are the traffic patterns, anomalies, and risky flows?" | No (10–60 min delay) | Yes (+ Log Analytics) |
| Connection Monitor | "Is this endpoint reachable right now and over time?" | Near real-time | No |
| Packet Capture | "What are the exact bytes in this network traffic?" | Near real-time | Yes |
| Next Hop | "What routing path does traffic take from this VM?" | Yes | No |

***

## What Happens If You Skip This?

Without NSG Flow Logs, you have no evidence of past network activity. If a breach happens, you cannot determine which connections were made, by whom, or to where. Security investigations become guesswork. Compliance frameworks like PCI-DSS and ISO 27001 require network traffic logging. Without Connection Monitor, you find out about connectivity problems only when users complain. Without IP Flow Verify, every NSG troubleshooting session takes hours of trial-and-error instead of seconds.

***

## AZ-500 Exam Section

- **Trigger Words:** "flow logs", "Traffic Analytics", "IP flow verify", "packet capture", "connection monitor", "NSG diagnostics", "network monitoring", "traffic logging"
- **Common Traps:**
  - NSG Flow Logs are NOT real-time. They have a 1–10 minute delay. Exam questions may imply you can use them for instant detection — you cannot.
  - NSG Flow Logs require a **storage account**. Traffic Analytics additionally requires a **Log Analytics workspace**.
  - Flow logs are enabled per **NSG**, not per subnet or per VM.
  - Network Watcher is **regional** — you need it enabled in every region where you want to use its tools.
  - **IP Flow Verify** tests a hypothetical packet at that moment. It tests NSG rules only — not Azure Firewall or UDR routing.
  - The Network Watcher **VM extension** must be installed on VMs for packet capture to work.
- **How to Tell Apart:**
  - "Troubleshoot why a VM cannot receive traffic" → IP Flow Verify.
  - "See historical connection logs" → NSG Flow Logs.
  - "Visualize traffic patterns and anomalies" → Traffic Analytics.
  - "Monitor if app A can always reach app B" → Connection Monitor.
  - "Capture raw packet data from a VM" → Packet Capture.
- **Likely Question Formats:**
  - "You need to find out which NSG rule is blocking traffic to a VM." → IP Flow Verify.
  - "You need to retain network traffic logs for 90 days for a PCI-DSS audit." → NSG Flow Logs with 90-day retention in a storage account.
  - "NSG Flow Logs are enabled but you see no data in Traffic Analytics." → Traffic Analytics requires a linked Log Analytics workspace and takes 20–60 minutes to populate after first enable.
  - "You need to monitor network latency between a VM and an on-premises server continuously." → Connection Monitor.
- **Memorization Tip:** Network Watcher is your security camera system. IP Flow Verify is asking the camera "did that car pass?" Traffic Analytics is reviewing the week's footage for suspicious patterns. Packet Capture is watching live video in full detail.

***

## Pocket Summary (5-Year Refresh)

- Network Watcher is a regional diagnostic service: IP Flow Verify, Connection Monitor, Packet Capture, Next Hop, VPN Diagnostics.
- NSG Flow Logs record every allow/deny decision through an NSG. Stored in a storage account. 1–10 min delay.
- Use Flow Log Version 2 for byte/packet counts.
- Traffic Analytics processes flow logs through Log Analytics and shows geographic traffic visualizations and anomaly detection.
- IP Flow Verify is the fastest tool for "why is this connection blocked?" — instant NSG rule diagnosis.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/network-watcher/network-watcher-overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
