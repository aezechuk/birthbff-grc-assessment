# BirthBFF Cloud Security & GRC Assessment
## Findings Report

**Assessment Scope:** AWS environment — birthbff-dev (us-east-1)
**Framework Alignment:** NIST SP 800-53 Rev. 5 | HIPAA Security Rule | CIS AWS Foundations Benchmark v1.4
**Total Findings:** 11
**Critical:** 2 | **High:** 7 | **Medium:** 2

---

## Severity Summary

| Finding ID | Title | Severity | Status |
|---|---|---|---|
| F-001 | IAM admin user with AdministratorAccess directly attached | Critical | Open |
| F-002 | Developer user excessive permissions and unused access keys | High | Open |
| F-003 | No MFA enforcement for any IAM user | High | Open |
| F-004 | S3 bucket publicly readable containing PHI-like data | Critical | Open |
| F-005 | No private subnet — all resources in public network tier | High | Open |
| F-006 | EC2 security group allows unrestricted SSH access | High | Open |
| F-007 | Database security group permits unrestricted MySQL access | High | Open |
| F-008 | No CloudWatch alerting for privileged actions | High | Open |
| F-009 | No formal data classification policy | Medium | Open |
| F-010 | S3 bucket uses SSE-S3 instead of SSE-KMS CMK | Medium | Open |
| F-011 | VPC flow logs not enabled | High | Open |

---

### Finding F-001: Overly Broad IAM Admin Permissions

**Title:** IAM user birthbff-dev-admin has AdministratorAccess attached directly  
**Asset:** IAM user — birthbff-dev-admin  
**Description:** The IAM user birthbff-dev-admin has the AWS managed policy AdministratorAccess attached directly to the user account rather than via a role or group. This grants unrestricted access to all AWS services and resources. In a healthcare platform handling PHI-like data, this violates the principle of least privilege and creates excessive risk of unauthorized access, data exfiltration, or accidental resource destruction.  
**Evidence:** `08-evidence/screenshots/01-iam/admin-user-administratoraccess-attached.png`  
**Likelihood:** High  
**Impact:** Critical  
**Severity:** Critical  
**NIST 800-53 Rev. 5:** AC-2 (Account Management), AC-6 (Least Privilege), AC-6(1) (Authorize Access to Security Functions)  
**HIPAA Security Rule:** §164.312(a)(1) — Access Control; §164.312(a)(2)(i) — Unique User Identification  
**CIS AWS Benchmark:** 1.16 — Ensure IAM policies are attached only to groups or roles  
**Recommended Remediation:** Remove AdministratorAccess from the user. Create an IAM group called `birthbff-admins`, attach a scoped admin policy to the group, and add the user to the group. Implement a break-glass role for emergency access with CloudTrail alerting on assumption.  
**Status:** Open

---

### Finding F-002: Developer User Has Excessive Permissions and Unused Access Keys

**Title:** IAM user birthbff-dev-developer has S3FullAccess and EC2FullAccess with unused programmatic credentials  
**Asset:** IAM user — birthbff-dev-developer  
**Description:** The developer IAM user has been granted full access to both S3 and EC2 via directly attached managed policies, exceeding the permissions required for typical development tasks. Additionally, programmatic access keys have been created but show no usage activity. Unused access keys with broad permissions represent a significant credential exposure risk — if the keys were exfiltrated, an attacker would have full access to all S3 data including PHI-like patient records.  
**Evidence:** `08-evidence/screenshots/01-iam/developer-user-excessive-permissions.png`, `developer-user-access-key-created.png`  
**Likelihood:** Medium  
**Impact:** High  
**Severity:** High  
**NIST 800-53 Rev. 5:** AC-2(3) (Disable Inactive Accounts), AC-6 (Least Privilege), IA-5 (Authenticator Management)  
**HIPAA Security Rule:** §164.312(a)(2)(i) — Unique User Identification; §164.312(d) — Person or Entity Authentication  
**CIS AWS Benchmark:** 1.12 — Ensure credentials unused for 45 days or greater are disabled; 1.14 — Ensure access keys are rotated every 90 days  
**Recommended Remediation:** Scope permissions to only required resources (specific S3 buckets and EC2 actions). Deactivate unused access keys immediately. Implement a key rotation policy of 90 days maximum.  
**Status:** Open

---

### Finding F-003: No MFA Enforcement for IAM Users

**Title:** No MFA device enrolled for any IAM user; no MFA enforcement policy exists  
**Asset:** All IAM users — birthbff-dev-admin, birthbff-dev-developer, birthbff-dev-readonly  
**Description:** The IAM credential report confirms that no MFA devices are enrolled for any IAM user in the BirthBFF environment. Additionally, no IAM policy exists requiring MFA before API or console access is granted. This means that compromised credentials alone (via phishing, credential stuffing, or key exposure) would grant an attacker full access to the platform. For a healthcare platform, MFA is a baseline control under HIPAA workforce access requirements.  
**Evidence:** `08-evidence/screenshots/01-iam/credential-report-download.png` (mfa_active column shows false for all users), `no-mfa-enforcement-policy.png`  
**Likelihood:** High  
**Impact:** High  
**Severity:** High  
**NIST 800-53 Rev. 5:** IA-2(1) (Multi-Factor Authentication to Privileged Accounts), IA-2(2) (MFA to Non-Privileged Accounts)  
**HIPAA Security Rule:** §164.312(d) — Person or Entity Authentication  
**CIS AWS Benchmark:** 1.10 — Ensure MFA is enabled for all IAM users with console access; 1.14 — Ensure hardware MFA is enabled for the root account  
**Recommended Remediation:** Attach an IAM policy requiring MFA for all console actions. Enroll virtual MFA devices (Google Authenticator or Authy) for all users. Require MFA before any sensitive API operations including S3 bucket policy changes and IAM modifications.  
**Status:** Open

---

### Finding F-004: S3 Bucket with Public Read Access Containing PHI-Like Data

**Title:** S3 bucket birthbff-dev-patient-documents is publicly readable and contains unencrypted mock PHI-like data  
**Asset:** S3 bucket — birthbff-dev-patient-documents  
**Description:** The S3 bucket used for BirthBFF patient document storage has Block Public Access disabled and a bucket policy granting s3:GetObject to Principal: * (all unauthenticated users). A mock patient intake CSV containing names, dates of birth, due dates, and clinical conditions was confirmed accessible via unauthenticated HTTP request using the object's public URL. In a production environment, this configuration would constitute an unauthorized disclosure of Protected Health Information under HIPAA, potentially triggering mandatory breach notification under the HIPAA Breach Notification Rule (45 CFR Part 164, Subpart D).  
**Evidence:** `08-evidence/screenshots/03-s3/s3-bucket-publicly-accessible-banner.png`, `s3-bucket-policy-public-read.png`, `s3-object-public-url-accessible.png`  
**Likelihood:** High  
**Impact:** Critical  
**Severity:** Critical  
**NIST 800-53 Rev. 5:** AC-3 (Access Enforcement), SC-28 (Protection of Information at Rest), SC-28(1) (Cryptographic Protection), MP-4 (Media Storage)  
**HIPAA Security Rule:** §164.312(a)(1) — Access Control; §164.312(e)(2)(ii) — Encryption; §164.308(a)(1) — Security Management Process  
**CIS AWS Benchmark:** 2.1.1 — Ensure S3 bucket policy is set to deny HTTP requests; 2.1.2 — Ensure S3 bucket Block Public Access is enabled  
**Recommended Remediation:** Enable Block All Public Access on the bucket immediately. Remove the public read bucket policy. Enable SSE-KMS encryption on all objects. Implement S3 Object Ownership controls. Restrict bucket access to the birthbff-dev-app-role IAM role only via a resource-based policy.  
**Status:** Open — Critical Priority

---

### Finding F-005: No Private Subnet — All Resources in Public Network Tier

**Title:** VPC birthbff-dev-vpc contains only a public subnet; no network segmentation for database tier  
**Asset:** VPC — birthbff-dev-vpc, Database tier  
**Description:** The BirthBFF VPC is configured with a single public subnet (10.0.1.0/24) and no private subnet. In a multi-tier application architecture handling sensitive health data, the database and application layers should be separated into private subnets with no direct internet routing. The absence of a private subnet means any future database deployment would be on the same network tier as the public-facing application server, increasing the blast radius of a network compromise.  
**Evidence:** `08-evidence/screenshots/02-vpc/vpc-public-subnet-only.png`  
**Likelihood:** Medium  
**Impact:** High  
**Severity:** High  
**NIST 800-53 Rev. 5:** SC-7 (Boundary Protection), SC-7(5) (Deny by Default), AC-4 (Information Flow Enforcement)  
**HIPAA Security Rule:** §164.312(e)(1) — Transmission Security; §164.308(a)(1) — Risk Analysis  
**CIS AWS Benchmark:** 5.1 — Ensure no Network ACLs allow ingress from 0.0.0.0/0 to remote server administration ports  
**Recommended Remediation:** Create a private subnet (10.0.2.0/24) in a separate availability zone. Create a NAT Gateway for outbound internet access from the private tier. Move the database security group and any future RDS instance into the private subnet. Restrict application server to database communication via security group rules only (no public routing).  
**Status:** Open

---

### Finding F-006: EC2 Security Group Allows Unrestricted SSH Access

**Title:** Security group birthbff-dev-app-sg permits inbound SSH (port 22) from 0.0.0.0/0  
**Asset:** EC2 security group — birthbff-dev-app-sg; EC2 instance — birthbff-dev-app-server  
**Description:** The application server security group permits inbound SSH connections from any IP address on the public internet (0.0.0.0/0). This exposes the server to brute-force attacks, credential stuffing, and exploitation of any SSH daemon vulnerabilities. For a healthcare platform, unrestricted administrative access to application servers represents a direct risk to the confidentiality and integrity of PHI-like data processed by the platform.  
**Evidence:** `08-evidence/screenshots/02-vpc/app-sg-ssh-open-to-world.png`, `08-evidence/screenshots/04-ec2/ec2-security-group-attached.png`  
**Likelihood:** High  
**Impact:** High  
**Severity:** High  
**NIST 800-53 Rev. 5:** SC-7 (Boundary Protection), AC-17 (Remote Access), CM-7 (Least Functionality)  
**HIPAA Security Rule:** §164.312(e)(1) — Transmission Security; §164.310(d)(1) — Device and Media Controls  
**CIS AWS Benchmark:** 5.2 — Ensure no security groups allow ingress from 0.0.0.0/0 to port 22; 5.4 — Ensure the default security group restricts all traffic  
**Recommended Remediation:** Remove the 0.0.0.0/0 SSH rule immediately. Restrict SSH to a known administrative IP CIDR or use AWS Systems Manager Session Manager for passwordless, keyless administrative access with full audit logging. If SSH must remain, implement an IP allowlist.  
**Status:** Open

---

### Finding F-007: Database Security Group Permits Unrestricted MySQL Access

**Title:** Security group birthbff-dev-db-sg permits inbound MySQL (port 3306) from 0.0.0.0/0  
**Asset:** Security group — birthbff-dev-db-sg  
**Description:** The database tier security group is pre-configured to allow inbound MySQL connections from any IP address. While no RDS instance is currently deployed, this security group is staged for use with the planned database layer. Deploying any RDS instance with this security group would immediately expose the database to internet-accessible brute force and injection attacks. Patient records, birth plans, and health data stored in the database would be at risk.  
**Evidence:** `08-evidence/screenshots/02-vpc/db-sg-mysql-open-to-world.png`  
**Likelihood:** Medium (no active database) → High (upon RDS deployment)  
**Impact:** Critical  
**Severity:** High  
**NIST 800-53 Rev. 5:** SC-7 (Boundary Protection), AC-3 (Access Enforcement), SI-3 (Malicious Code Protection)  
**HIPAA Security Rule:** §164.312(a)(1) — Access Control; §164.312(e)(1) — Transmission Security  
**CIS AWS Benchmark:** 5.3 — Ensure no security groups allow ingress from 0.0.0.0/0 to port 3306  
**Recommended Remediation:** Remove the 0.0.0.0/0 inbound rule. Restrict MySQL access to the application server security group only (source: birthbff-dev-app-sg). Place the database in a private subnet with no internet routing.  
**Status:** Open

---

### Finding F-008: No CloudWatch Alerting for Privileged Actions

**Title:** No CloudWatch alarms configured for IAM policy changes, S3 public access changes, or root account usage  
**Asset:** CloudWatch, CloudTrail  
**Description:** While CloudTrail captures API activity, no CloudWatch metric filters or alarms are configured to alert on high-risk events including: root account login, IAM policy changes, S3 bucket policy modifications, console authentication failures, or security group changes. Without alerting, a security incident could occur and remain undetected for an extended period. HIPAA requires the ability to detect and respond to security incidents, which requires active monitoring, not just passive logging.  
**Evidence:** `08-evidence/screenshots/05-cloudtrail/cloudtrail-no-cloudwatch-integration.png`  
**Likelihood:** High  
**Impact:** High  
**Severity:** High  
**NIST 800-53 Rev. 5:** AU-6 (Audit Record Review), SI-4 (System Monitoring), IR-6 (Incident Reporting)  
**HIPAA Security Rule:** §164.308(a)(1)(ii)(D) — Information System Activity Review; §164.308(a)(6) — Security Incident Procedures  
**CIS AWS Benchmark:** 3.1 — 3.14 (CloudWatch metric filter alarms for all critical events)  
**Recommended Remediation:** Enable CloudWatch Logs integration on the CloudTrail trail. Create metric filters and alarms for: root account usage, IAM policy changes, unauthorized API calls, S3 bucket policy changes, security group changes, and console auth failures. Route alarms to an SNS topic with email notification to the security team.  
**Status:** Open

---

### Finding F-009: No Formal Data Classification Policy

**Title:** Sensitive maternal health data has not been formally classified; no data handling policy exists  
**Asset:** Platform-wide — all data stores  
**Description:** The BirthBFF platform processes user-generated content including chat conversations, birth plans, and clinical notes that may contain sensitive personal health information. No formal data classification policy exists to define sensitivity tiers, handling requirements, retention periods, or disposal procedures for this data. Without classification, technical controls cannot be consistently applied — for example, it is unclear which S3 buckets, database tables, or API responses require encryption, access logging, or restricted access.  
**Evidence:** Absence of policy documentation (gap finding — no screenshot required)  
**Likelihood:** High  
**Impact:** Medium  
**Severity:** Medium  
**NIST 800-53 Rev. 5:** RA-2 (Security Categorization), MP-4 (Media Storage), SI-12 (Information Management and Retention)  
**HIPAA Security Rule:** §164.308(a)(1)(ii)(A) — Risk Analysis; §164.316(b)(1) — Documentation  
**CIS AWS Benchmark:** Not directly mapped — governance control  
**Recommended Remediation:** Create a Data Classification and Handling Policy defining at minimum: Restricted (PHI-like data — birth plans, health notes, patient intake), Internal (platform operational data), and Public (marketing content, educational resources) tiers. Map each tier to technical controls including encryption requirements, access restrictions, retention schedules, and disposal procedures.  
**Status:** Open — Policy work required

---


### Finding F-010: S3 Bucket Uses AWS-Managed Encryption Instead of Customer-Managed KMS Key

**Title:** S3 bucket birthbff-dev-patient-documents uses SSE-S3 (AWS-managed) encryption rather than SSE-KMS with a customer-managed key  
**Asset:** S3 bucket — birthbff-dev-patient-documents  
**Description:** The birthbff-dev-patient-documents bucket uses default SSE-S3 encryption (AES-256) with AWS-managed keys. While SSE-S3 provides baseline encryption at rest, it does not satisfy HIPAA requirements for covered entities handling ePHI at the level of control required. SSE-S3 uses AWS-managed keys where the organization has no visibility into key usage, cannot audit key access events, and cannot revoke key access in response to a security incident. NIST SP 800-53 SC-12 requires that organizations establish and manage cryptographic keys for required cryptography employed within the information system. Customer-managed KMS keys (CMKs) provide key usage logs in CloudTrail, enabling the organization to audit every decrypt operation against patient data.  
**Evidence:** 08-evidence/screenshots/03-s3/s3-encryption-sse-s3-default.png  
**Likelihood:** Medium  
**Impact:** High  
**Severity:** Medium  
**NIST 800-53 Rev. 5:** SC-12 (Cryptographic Key Establishment and Management), SC-28 (Protection of Information at Rest), SC-28(1) (Cryptographic Protection), AU-9 (Protection of Audit Information)  
**HIPAA Security Rule:** §164.312(a)(2)(iv) — Encryption and Decryption; §164.312(e)(2)(ii) — Encryption of ePHI in Transit and at Rest; §164.308(a)(1)(ii)(B) — Risk Management  
**CIS AWS Benchmark:** 2.1.1 — Ensure all S3 buckets employ encryption-at-rest; 3.7 — Ensure CloudTrail logs are encrypted at rest using KMS CMKs (same control principle applied to data storage)  
**Recommended Remediation:** Create a customer-managed KMS key named birthbff-cmk-phi with key rotation enabled (annual minimum). Update the bucket default encryption to SSE-KMS using the CMK. Apply a bucket policy denying any PutObject request that does not include the SSE-KMS encryption header, preventing unencrypted uploads. Confirm CloudTrail captures KMS key usage events for audit purposes.  
**Status:** Open  

---

### Finding F-011: VPC Flow Logs Not Enabled — No Network Traffic Audit Trail

**Title:** VPC birthbff-dev-vpc has no flow logs configured; network traffic to and from resources processing ePHI is not captured  
**Asset:** VPC — birthbff-dev-vpc  
**Description:** VPC Flow Logs are not enabled for birthbff-dev-vpc. Flow logs provide a record of IP traffic to and from network interfaces within the VPC, including source and destination IPs, ports, protocols, and allow/deny decisions. Without flow logs, there is no audit trail for network-level activity, no ability to investigate the scope of a potential data exfiltration event, and no mechanism to detect unusual traffic patterns such as unexpected outbound connections from the application server or repeated connection attempts to the database tier. For a platform processing maternal health data, network visibility is a foundational detective control required under HIPAA's Security Incident Procedures standard.  
**Evidence:** 08-evidence/screenshots/02-vpc/vpc-flow-logs-not-enabled.png  
**Likelihood:** High  
**Impact:** High  
**Severity:** High  
**NIST 800-53 Rev. 5:** AU-2 (Event Logging), AU-12 (Audit Record Generation), SI-4 (System Monitoring), IR-5 (Incident Monitoring)  
**HIPAA Security Rule:** §164.312(b) — Audit Controls; §164.308(a)(1)(ii)(D) — Information System Activity Review; §164.308(a)(6)(ii) — Response and Reporting  
**CIS AWS Benchmark:** 3.9 — Ensure VPC flow logging is enabled in all VPCs  
**Recommended Remediation:** Enable VPC Flow Logs on birthbff-dev-vpc capturing ALL traffic (Accept and Reject). Deliver logs to a dedicated CloudWatch Log Group or S3 bucket separate from application logs. Encrypt the log destination using the birthbff-cmk-phi KMS key. Set a retention period of 365 days minimum to support HIPAA audit requirements. Consider creating a CloudWatch metric filter to alert on high volumes of REJECT traffic, which may indicate reconnaissance activity.  
**Status:** Open  

