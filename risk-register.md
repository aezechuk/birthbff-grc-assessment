# BirthBFF Cloud Security & GRC Assessment
## Risk Register

**Platform:** BirthBFF — AI-powered childbirth education application
**Environment:** AWS (us-east-1) — birthbff-dev
**Framework Alignment:** NIST SP 800-53 Rev. 5 | HIPAA Security Rule | CIS AWS Foundations Benchmark v1.4
**Last Updated:** May 2026

---

## Risk Scoring Methodology

Risks are scored using a 5x5 likelihood and impact matrix.

| Score | Likelihood | Impact |
|---|---|---|
| 1 | Rare | Negligible |
| 2 | Unlikely | Minor |
| 3 | Possible | Moderate |
| 4 | Likely | Major |
| 5 | Almost Certain | Critical |

**Risk Score = Likelihood x Impact**

| Risk Score | Risk Level |
|---|---|
| 1-4 | Low |
| 5-9 | Medium |
| 10-14 | High |
| 15-25 | Critical |

---

## Risk Register

| Risk ID | Risk Title | Related Finding | Threat Scenario | Likelihood | Impact | Risk Score | Risk Level | Recommended Treatment | Treatment Type | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| R-001 | Unauthorized access via overprivileged IAM admin account | F-001 | Threat actor compromises admin credentials and gains unrestricted access to all AWS services, enabling data exfiltration, resource destruction, or ransomware deployment across the BirthBFF environment | 4 | 5 | 20 | Critical | Remove AdministratorAccess from user. Create scoped admin group with least-privilege policy. Implement break-glass role with CloudTrail alerting | Mitigate | Open |
| R-002 | ePHI exposure via publicly accessible S3 bucket | F-004 | Unauthenticated user discovers public S3 URL and downloads patient records CSV containing names, DOB, due dates, and clinical conditions, triggering HIPAA Breach Notification Rule obligations | 5 | 5 | 25 | Critical | Enable Block All Public Access. Remove public read bucket policy. Restrict access to app role only via resource-based policy | Mitigate | Open |
| R-003 | Credential compromise due to missing MFA | F-003 | Phishing or credential stuffing attack succeeds against any IAM user. Without MFA, compromised password alone grants full console or API access to the platform | 4 | 4 | 16 | Critical | Enforce MFA via IAM policy for all users. Enroll virtual MFA devices. Require MFA before sensitive API operations | Mitigate | Open |
| R-004 | Lateral movement via unrestricted SSH access | F-006 | Attacker scans internet-facing EC2 instance, brute forces SSH credentials or exploits SSH daemon vulnerability, gains shell access to application server and pivots to connected resources | 4 | 4 | 16 | Critical | Remove 0.0.0.0/0 SSH rule. Implement AWS Systems Manager Session Manager for keyless, audited administrative access | Mitigate | Open |
| R-005 | Database exposure upon RDS deployment | F-007 | Database security group pre-staged with MySQL port 3306 open to 0.0.0.0/0. Any RDS instance deployed with this security group is immediately internet-accessible | 3 | 5 | 15 | Critical | Remove 0.0.0.0/0 inbound rule. Restrict MySQL to app security group source only. Deploy RDS in private subnet | Mitigate | Open |
| R-006 | Undetected intrusion due to absent network monitoring | F-011 | Attacker establishes persistent access or exfiltrates data over the network. Without VPC flow logs no forensic trail exists to determine scope, entry point, or duration of compromise | 4 | 4 | 16 | Critical | Enable VPC flow logs capturing all traffic. Encrypt log destination with CMK. Retain logs 365 days minimum | Mitigate | Open |
| R-007 | Undetected privileged action abuse | F-008 | Insider threat or compromised account makes high-risk changes such as IAM policy modifications, S3 bucket policy changes, or security group updates. Without CloudWatch alerting the activity goes unnoticed until significant damage is done | 3 | 4 | 12 | High | Enable CloudWatch Logs integration on CloudTrail. Create metric filters and alarms for root usage, IAM changes, S3 policy changes, and auth failures | Mitigate | Open |
| R-008 | Developer credential exfiltration | F-002 | Unused programmatic access keys for developer IAM user with S3FullAccess and EC2FullAccess are exfiltrated via repository exposure or endpoint compromise, granting full access to patient data in S3 | 3 | 4 | 12 | High | Deactivate unused access keys immediately. Scope permissions to required resources only. Implement 90-day key rotation policy | Mitigate | Open |
| R-009 | ePHI encrypted with non-auditable AWS-managed key | F-010 | AWS-managed SSE-S3 key is used to encrypt patient documents. Organization has no visibility into key access, cannot audit decrypt operations, and cannot revoke key access in response to a breach | 3 | 3 | 9 | Medium | Migrate to SSE-KMS with customer-managed key. Enable key rotation. Deny PutObject requests without SSE-KMS header via bucket policy | Mitigate | Open |
| R-010 | Uncontrolled data handling due to absent classification policy | F-009 | Without a data classification policy, technical controls are applied inconsistently. ePHI-containing resources may lack required encryption, access logging, or retention controls leading to undetected compliance gaps | 3 | 3 | 9 | Medium | Create Data Classification and Handling Policy defining Restricted, Internal, and Public tiers with mapped technical controls | Mitigate | Open |
| R-011 | Expanded blast radius due to flat network architecture | F-005 | Absence of private subnet means database and application tiers share the same network layer. Compromise of the public-facing application server provides direct network access to future database resources | 3 | 4 | 12 | High | Create private subnet in separate AZ. Deploy NAT Gateway for outbound access. Migrate database tier to private subnet | Mitigate | Open |

---

## Critical Risks Requiring Immediate Remediation

| Priority | Risk ID | Risk Title | Risk Score |
|---|---|---|---|
| 1 | R-002 | ePHI exposure via publicly accessible S3 bucket | 25 |
| 2 | R-001 | Unauthorized access via overprivileged IAM admin account | 20 |
| 3 | R-003 | Credential compromise due to missing MFA | 16 |
| 4 | R-004 | Lateral movement via unrestricted SSH access | 16 |
| 5 | R-006 | Undetected intrusion due to absent network monitoring | 16 |
| 6 | R-005 | Database exposure upon RDS deployment | 15 |
