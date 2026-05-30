
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
