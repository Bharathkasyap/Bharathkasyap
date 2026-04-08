# Azure SQL Database and Managed Instance Security

> 📌 AZ-500 Exam Objective: Plan and implement security for Azure SQL Database
> 🏷️ Domain: 3 — Secure Compute, Storage, and Databases | Weight: 20–25%

***

## What Is This?

Azure SQL Database is a fully managed relational database in the cloud. SQL security means protecting data at rest, in transit, and in use — and making sure only the right people and applications can access specific rows and columns. It also means detecting threats and meeting audit requirements.

***

## Why Does This Exist?

Databases hold the most sensitive data in any organization — customer records, financial transactions, health information, trade secrets. A compromised database is almost always a business-ending event. Azure SQL provides overlapping security layers so you can encrypt data, restrict access at the row and column level, detect attacks, and prove compliance to auditors.

***

## Who Actually Needs This? (Real-World Context)

A bank in Singapore runs its core lending platform on Azure SQL Database. The database holds customer salary information. The compliance team requires that call center agents can query customer accounts but can never see full salary figures — only masked values. The operations team must be able to query the database but cannot see the raw values of social security numbers in any query tool — not even SSMS. The security team enables Transparent Data Encryption with a customer-managed key, Always Encrypted on the SSN column, Dynamic Data Masking on the salary column, Row-Level Security so agents only see their own customers, and SQL auditing to Log Analytics for a 7-year retention requirement.

***

## How It Works (Pure Concept, No Portal)

**Transparent Data Encryption (TDE)**
TDE encrypts the database files at rest. It is enabled by default on all Azure SQL databases. The encryption key (Database Encryption Key) is protected by a TDE protector:
- **Service-managed key**: Azure generates and manages the TDE protector. Automatic, zero-configuration.
- **Customer-managed key (CMK/BYOK)**: Your own key in Key Vault acts as the TDE protector. You control key rotation and can revoke access by disabling the key (effectively making the database unreadable). This is "Bring Your Own Key" (BYOK).

**Important**: TDE encrypts data at the storage file level. A DBA connecting with SQL credentials can still read all data through normal queries. TDE only protects against someone stealing the physical database files.

**Always Encrypted**
Always Encrypted encrypts specific columns on the client side. The data is encrypted *before* it leaves the application and is only decrypted *after* it is returned to the application. The SQL Server engine never sees the plaintext value — not even in memory. Even a DBA with full SQL access cannot read Always Encrypted column values. Column encryption keys are stored in a Key Store (Key Vault, Windows Certificate Store, or Azure Key Vault Managed HSM).

This is the only SQL encryption feature that protects data from database administrators.

Two encryption types:
- **Deterministic**: Same plaintext always produces the same ciphertext. Supports equality comparisons and indexing.
- **Randomized**: Same plaintext produces different ciphertext each time. More secure but does not support equality comparisons or indexing.

**Dynamic Data Masking (DDM)**
DDM masks sensitive column values in query results for non-privileged users. The data is not encrypted — it is full-fidelity in the database. A DDM rule defines the masking pattern:
- `default()`: Replaces value with XXXX or 0 depending on data type.
- `email()`: Shows first letter + `XXX@XXXX.com`.
- `random(X, Y)`: Shows a random number in the given range (for numeric columns).
- `partial(prefix, padding, suffix)`: Shows only the defined prefix and suffix.

Privileged users (db_owner, sysadmin) see unmasked data. DDM is a presentation-layer control, not a strong security control — anyone with direct database access can see real values.

**Row-Level Security (RLS)**
RLS filters query results so users only see rows they are authorized to see. A security policy with a predicate function runs before every `SELECT`, `INSERT`, `UPDATE`, or `DELETE`. For example, a salesperson can only see rows where `SalesRepID = CURRENT_USER`. This filtering is transparent to the application — it does not need to add `WHERE` clauses.

**Microsoft Defender for SQL**
Defender for SQL has two components:
- **Advanced Threat Protection**: Detects suspicious SQL activities — SQL injection probes, unusual login patterns, access from unusual locations, privilege escalation attempts. Sends alerts in real time.
- **Vulnerability Assessment**: Scans the database configuration and identifies misconfigurations — accounts with blank passwords, excessive permissions, databases without auditing.

**Entra ID Authentication for SQL**
Instead of SQL logins (username + password), you can use Entra ID identities to authenticate. Set an Entra ID admin on the SQL server. Then create contained database users mapped to Entra ID identities. This provides:
- No SQL passwords to manage or rotate.
- MFA enforcement on database connections.
- Entra ID Conditional Access policies for database access.
- Full audit trail in Entra ID sign-in logs.

You can force Entra-only auth by enabling **Azure AD-only authentication** mode, which disables all SQL-native logins.

**SQL Auditing**
SQL auditing logs every database event — logins, queries, schema changes, permission grants — to one of three destinations:
- **Azure Storage**: Low-cost, long-term archival. Files stored as `.xel` format.
- **Log Analytics workspace**: Enables real-time querying, alerting, and integration with Microsoft Sentinel.
- **Event Hub**: For streaming to SIEM tools like Splunk.

Server-level auditing covers all databases on the server. Database-level auditing covers only the specific database.

**SQL Firewall Rules**
The SQL Server has a firewall that controls which IP addresses can connect. Rules can be set at the server level (apply to all databases) or at the database level. "Allow Azure services" permits connections from any Azure service — use with caution as it opens the server to all Azure tenants.

**Private Endpoint for SQL**
A private endpoint assigns the SQL server a private IP on your VNet. Combined with disabling public endpoint, the database is only reachable from inside the VNet or connected networks. This eliminates internet-based attacks on the SQL endpoint.

***

## 2026 Portal Walkthrough — Step by Step

**Goal: Enable TDE with CMK, configure Entra ID admin, enable Defender for SQL, set up auditing**

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Open your **SQL server** (not the database — the server resource).
3. **Set Entra ID admin:**
   - Under **Settings**, click **Microsoft Entra ID**.
   - Click **Set admin** and select an Entra ID user or group.
   - Click **Save**.
4. **Enable Entra-only authentication:**
   - In the same blade, check **Support only Microsoft Entra authentication for this server**.
   - Click **Save**. SQL logins are now disabled.
5. **Enable Defender for SQL:**
   - Under **Security**, click **Microsoft Defender for Cloud**.
   - Click **Enable Microsoft Defender for SQL**. Select a storage account for vulnerability assessment results.
6. **Enable SQL Auditing:**
   - Under **Security**, click **Auditing**.
   - Toggle auditing to **ON**.
   - Select **Log Analytics** as a destination.
   - Select your workspace.
   - Click **Save**.
7. Open the **SQL Database** (not the server).
8. **Enable TDE with CMK:**
   - Under **Security**, click **Transparent data encryption**.
   - Select **Customer-managed key**.
   - Select your Key Vault and key. (The Key Vault must have soft delete and purge protection enabled.)
   - Click **Save**.

**CLI equivalent:**
```bash
# Enable Entra-only authentication
az sql server update \
  --resource-group myRG \
  --name mySQLServer \
  --enable-ad-only-auth

# Enable TDE with service-managed key (default; CMK requires portal or ARM template)
az sql db tde set \
  --resource-group myRG \
  --server mySQLServer \
  --database myDB \
  --status Enabled

# Enable Defender for SQL at server level
az security atp sql server update \
  --resource-group myRG \
  --workspace mySQLServer \
  --is-enabled true
```

**PowerShell equivalent:**
```powershell
# Enable TDE on a database
Set-AzSqlDatabaseTransparentDataEncryption `
  -ResourceGroupName "myRG" `
  -ServerName "mySQLServer" `
  -DatabaseName "myDB" `
  -State "Enabled"

# Set Entra ID admin
Set-AzSqlServerActiveDirectoryAdministrator `
  -ResourceGroupName "myRG" `
  -ServerName "mySQLServer" `
  -DisplayName "SQLAdmins" `
  -ObjectId "<entra-group-object-id>"
```

> 📸 SCREENSHOT REFERENCE: https://learn.microsoft.com/en-us/azure/azure-sql/database/security-overview

***

## Key Settings Explained

| Setting | What It Does | When to Use |
|---|---|---|
| TDE with service-managed key | Auto-encrypts database files at rest; Azure manages key | Default for all databases; no action required |
| TDE with CMK (BYOK) | Your Key Vault key is the TDE protector | Regulatory requirement to own the encryption key |
| Always Encrypted | Column-level client-side encryption; DBAs cannot see values | SSNs, credit card numbers, medical record fields |
| Dynamic Data Masking | Masks column values in query results for non-privileged users | Call center, reporting, limited-access users — not a strong control |
| Row-Level Security | Filters rows based on the user's identity before returning results | Multi-tenant apps; user-scoped data access |
| Entra ID admin | Replaces SQL server-level admin with an Entra ID identity | Always set; eliminates the default SA account risk |
| Entra-only authentication | Disables all SQL native logins | Highest security; forces MFA and Conditional Access |
| Defender for SQL | Threat detection + vulnerability assessment | All production databases |
| SQL auditing (Log Analytics) | Logs every database event; queryable in real time | Compliance, forensics, Sentinel integration |
| Private endpoint | Assigns SQL server a private VNet IP; public endpoint disabled | All production databases; eliminates internet exposure |

***

## Comparison Table

| Encryption Feature | Where It Works | Protects from DBAs? | Supports Queries? |
|---|---|---|---|
| TDE (service-managed) | Storage layer | No | Yes — full query support |
| TDE with CMK | Storage layer | No (DBA can still query) | Yes — full query support |
| Always Encrypted (deterministic) | Column, client-side | Yes | Equality only (=, IN) |
| Always Encrypted (randomized) | Column, client-side | Yes | No — cannot query encrypted value |
| Dynamic Data Masking | Presentation layer only | No — DBAs see real data | Yes — masking is display-only |

| Access Control | Granularity | Use Case |
|---|---|---|
| SQL logins | Database level | Legacy apps; being replaced by Entra ID |
| Entra ID users | Database level | Modern apps; MFA, Conditional Access |
| Row-Level Security | Row level | Multi-tenant; user-scoped data |
| Dynamic Data Masking | Column level (display) | Limited visibility without true encryption |
| Always Encrypted | Column level (true encryption) | Maximum column confidentiality |

***

## What Happens If You Skip This?

Without TDE, database files copied off a disk are readable. Without Always Encrypted, a DBA with SQL credentials can read SSNs, credit card numbers, and health records directly. Without Row-Level Security, a single SQL query can return every customer's data to any user. Without auditing, a data breach may go undetected and you cannot prove compliance to regulators. Without Defender for SQL, a SQL injection attack against your application can drain the entire database over days with no alert.

***

## AZ-500 Exam Section

- **Trigger Words:** "TDE", "transparent data encryption", "Always Encrypted", "dynamic data masking", "row-level security", "Defender for SQL", "SQL audit", "BYOK", "CMK", "Entra admin SQL", "private endpoint SQL"
- **Common Traps:**
  - TDE does NOT protect data from DBAs. A DBA can connect to a TDE-protected database and query all data normally. TDE only protects the physical files.
  - Always Encrypted is the ONLY SQL security feature that prevents DBAs from reading sensitive column values.
  - Dynamic Data Masking is NOT encryption. The data is stored in plaintext; only the display is masked. A DBA or anyone with elevated permissions sees the real values.
  - Enabling "Allow Azure services" on the SQL firewall opens the database to all Azure tenants — not just your own. Use private endpoints instead.
- **How to Tell Apart:**
  - "protect database files if the storage is stolen" → TDE
  - "even the DBA cannot see the value" → Always Encrypted
  - "call center agents see XXX-XX-1234 instead of the full SSN" → Dynamic Data Masking
  - "users only see rows belonging to their account" → Row-Level Security
  - "detect SQL injection attempts in real time" → Microsoft Defender for SQL (Advanced Threat Protection)
- **Likely Question Formats:**
  - A DBA must never be able to read credit card numbers. What do you enable? → Always Encrypted
  - What is the difference between TDE with service-managed key and TDE with CMK? → service-managed = Azure owns the key; CMK = you store the key in your Key Vault and control rotation and revocation
  - You want to ensure sales reps only see rows for their own customers. What do you configure? → Row-Level Security
- **Memorization Tip:** **TARDA** — the SQL security stack from most visible to least visible to a DBA:
  - **T**DE = storage encryption (DBA CAN see data in queries)
  - **A**lways Encrypted = column encryption (DBA CANNOT see data)
  - **R**ow-Level Security = row filter (user sees only their rows)
  - **D**ynamic Data Masking = display mask (DBA still sees real data)
  - **A**uditing = logging everything (who did what and when)

***

## Pocket Summary (5-Year Refresh)

- TDE encrypts database files at rest using AES-256; CMK/BYOK lets you store the TDE protector key in your own Key Vault — but TDE does NOT prevent DBAs from reading data through SQL.
- Always Encrypted encrypts specific columns client-side so the SQL engine never sees plaintext values — this is the only feature that truly protects data from database administrators.
- Dynamic Data Masking hides column values in query results for non-privileged users but is not real encryption; it is a display-only control.
- Row-Level Security uses predicate functions to transparently filter rows based on the identity running the query — perfect for multi-tenant databases.
- Microsoft Defender for SQL detects SQL injection, anomalous logins, and misconfigurations; enable SQL auditing to Log Analytics for compliance and forensic investigation.

---
📚 Further Reading: https://learn.microsoft.com/en-us/azure/azure-sql/database/security-overview
🔄 Last Verified: 2026 (AZ-500 January 2026 objectives)
