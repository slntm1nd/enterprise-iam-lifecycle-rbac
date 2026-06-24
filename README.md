
# Enterprise IAM Identity Lifecycle & RBAC Framework

## 1. Project Overview
This repository contains the architecture, documentation, and logic profiles for a production-ready Identity and Access Management (IAM) framework designed for an enterprise environment. The project focuses on automating the **Identity Lifecycle (Joiner-Mover-Leaver)**, implementing **Role-Based Access Control (RBAC)**, and enforcing strict compliance alignment with **NIST SP 800-63 (Digital Identity Guidelines)**.

The goal is to eliminate privilege creep, enforce the **Principle of Least Privilege**, and reduce the operational burden on IT Service Desks via automation logis.

---

## 2. Identity Lifecycle Architecture (JML Process)

### 🟢 Joiner (Onboarding Workflow)
1. **Trigger:** A new employee record is created in the HR Information System (HRIS - e.g., Workday).
2. **Identity Creation:** Automated script/IdP connector provisions an identity in **Microsoft Entra ID / Active Directory**.
3. **Birthright Access:** The user automatically receives basic access (Corporate Email, Corporate Chat, ServiceNow portal) based on membership in the `All_Employees` group.
4. **Role Assignment:** Dynamic groups assign functional access based on the department attribute (see `automation-logic.json`).

### 🟡 Mover (Cross-boarding Workflow)
1. **Trigger:** HR updates the employee's department, job title, or manager attribute.
2. **Access Reconciliation:** An automated review ticket is generated in ServiceNow. 
3. **Revocation Before Grant:** Old department-specific security groups are revoked within 24 hours to prevent privilege creep.
4. **New Provisioning:** New role-based permissions are applied based on the updated RBAC Matrix.

### 🔴 Leaver (Offboarding Workflow)
*To mitigate insider threat and orphaned account risks, a strict **4-step immediate separation checklist** is executed:*
1. **Account Disabling:** Instant account suspension (`accountEnabled = false`) in the primary Identity Provider (IdP) upon termination timestamp.
2. **Session Revocation:** Active OAuth tokens, VPN sessions, and SaaS application connections are forcefully terminated via Revoke-AzureADUserAllRefreshToken or Okta Clear Sessions API.
3. **Object Management:** The user object is moved to the `Disabled_Users` Organizational Unit (OU) and hidden from the Global Address List (GAL).
4. **Deprovisioning & Reclamation:** After a 30-day legal hold, SaaS licenses are automatically reclaimed, and the account is flagged for deletion.

---

## 3. Core Technical Artifacts

* **[RBAC Access Matrix (CSV)](./rbac-matrix.csv):** A defined breakdown mapping corporate roles to specific application entitlements, including required multi-factor authentication (MFA) parameters and business ownership approvals.
* **[Automated Provisioning Logic (JSON)](./automation-logic.json):** A simulation of an IdP workflow policy engine routing attributes from HR systems down to directory services and ticketing queues.

---

## 4. Security & Compliance Enforcement

* **Multi-Factor Authentication (MFA):** Enforced globally via Entra ID Conditional Access Policies. Legacy authentication protocols (IMAP, POP3) are blocked entirely.
* **Privileged Access Management (PAM):** Administrative accounts (Global Admin, Domain Admin) are prohibited from having permanent access. Just-In-Time (JIT) access must be requested through a PAM tool with dual-authorization approval workflows.
* **Access Reviews:** Automated quarterly access certification schedules are configured for high-risk financial and infrastructure roles.

---

## 5. Frameworks & Standards Aligned
* **NIST SP 800-63B:** Authentication and Lifecycle Management.
* **ISO/IEC 27001 (A.9):** Access Control requirements.
* **SOX / HIPAA Compliance:** Documentation of strict evidence trails for user provisioning and timely deprovisioning.

* ## 🛠️ 5. Practical Lab Simulation & Security Auditing (Azure)

To validate this IAM framework, I built a controlled cloud infrastructure staging environment within Microsoft Azure to simulate enterprise operations and verify audit trails.

### Environment Topology
* **Identity Provider (IdP):** Windows Server 2022 Domain Controller (`DC01`) managing the `corp.local` active directory forest.
* **Endpoint Workspace:** Windows Client (`CLIENT01`) joined to the domain for interactive access validation.

### Hands-on Implementation Workflows
1. **Directory Taxonomy & Provisioning:** Created a dedicated Organizational Unit (OU) (`TestUsers`) within Active Directory Users and Computers (ADUC). Provisioned baseline identities (`Alice`, `Bob`, `Charlie`) with enforced initial credential parameters.
2. **RBAC & Privilege Escalation Simulation:** Executed an administrative password reset workflow and modified local host security matrices (`sysdm.cpl`) on `CLIENT01` to explicitly grant `corp\alice` entry into the **Remote Desktop Users** local security group, isolating access tracking pathways.
3. **Telemetry & Log Aggregation:** Conducted cross-system interactive logons to populate the Windows Security database, validating domain communication states and establishing a centralized audit baseline.

---

## 📊 6. Incident Monitoring & Event Log Correlation

An IAM framework is only as secure as its visibility. To support audit readiness and incident detection patterns, I isolated and correlated specific high-value Microsoft Security Event IDs generated during the lab actions:

| Event ID | Log Category | Technical Event Context | GRC / Audit Significance |
| :---: | :--- | :--- | :--- |
| **4720** | Account Management | A user account was created | Validates automated provisioning (**Joiner** phase audit). |
| **4724** | Account Management | An attempt was made to reset an account's password | Monitors admin actions and detects potential credential hijacking. |
| **4732** | Group Management | A member was added to a security group | Detects **Privilege Escalation** (e.g., granting RDP access to Alice). |
| **4624** | Authentication | An account was successfully logged on | Confirms identity validation and logs explicit Logon Types. |
| **4625** | Authentication | An account failed to log on | Core indicator for **Brute-Force** or password-guessing detection. |
| **4672** | Authentication | Special privileges assigned to new logon | Flags user sessions carrying elevated administrative rights. |

> 💾 **SIEM Readiness:** The raw directory telemetry database was structured and exported as `DC01-SecurityLogs.evtx`. This log architecture is specifically engineered to be ingested into SIEM environments (**Splunk Core Power User / Azure Sentinel**) for automated log parsing, compliance reporting, and alerting rules.
>
> ## 👥 Contact & Project Author
* **Author:** Jane Nikolaichuk
* **Role:** IAM & Access Governance Analyst (Charlotte, NC)
* **LinkedIn:** www.linkedin.com/in/iamjanenikolaichuk
* **GitHub Portfolio:** https://github.com/slntm1nd

---
**Skills Demonstrated:** Identity Lifecycle Management (JML) • Active Directory Administration (ADDS) • Role-Based Access Control (RBAC) • Windows Security Event Log Analysis • Compliance Tracking & Auditing • Incident Monitoring Focus
