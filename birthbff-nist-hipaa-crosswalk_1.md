# BirthBFF NIST-HIPAA Scoped Crosswalk

> Framework mapping based on NIST SP 800-66 Rev. 2 — *Implementing the HIPAA Security Rule: A Cybersecurity Resource Guide* (NIST, February 2023).
> Scoped to HIPAA Security Rule requirements implicated by findings from the BirthBFF AWS Cloud Security and GRC Assessment.
> All findings reflect the simulated BirthBFF environment as assessed. Status: Open pending remediation.

---

## Crosswalk Table

| HIPAA Safeguard | Citation | Standard Name | Implementation Spec | Type | Requirement Summary | NIST 800-53 Controls | BirthBFF Finding | Status |
|---|---|---|---|---|---|---|---|---|
| Technical | §164.312(a)(1) | Access Control | | Standard | Requires technical controls to allow only authorized users and processes to access electronic protected health information | AC-2, AC-3, AC-6 | F-001, F-002, F-004 | Open |
| Technical | §164.312(a)(2)(i) | Access Control | Unique User Identification | Required | Each user must be assigned a unique identifier to enable activity to be traced to a specific individual | IA-2, IA-5 | F-002, F-003 | Open |
| Technical | §164.312(a)(2)(iv) | Access Control | Encryption and Decryption | Addressable | Implement a mechanism to encrypt and decrypt ePHI when deemed appropriate for the organization | SC-12, SC-28, SC-28(1), AU-9 | F-010 | Open |
| Technical | §164.312(b) | Audit Controls | | Standard | Implement hardware, software, and procedural mechanisms to record and examine activity in systems that contain or use ePHI | AU-2, AU-6, AU-12, SI-4 | F-008, F-011 | Open |
| Technical | §164.312(d) | Person or Entity Authentication | | Standard | Implement procedures to verify that a person or entity seeking access to ePHI is the one claimed | IA-2, IA-5 | F-002, F-003 | Open |
| Technical | §164.312(e)(1) | Transmission Security | | Standard | Implement technical security measures to guard against unauthorized access to ePHI transmitted over an electronic communications network | SC-7, AC-17, CM-7 | F-005, F-006, F-007, F-009 | Open |
| Technical | §164.312(e)(2)(ii) | Transmission Security | Encryption | Addressable | Implement a mechanism to encrypt ePHI whenever deemed appropriate | SC-12, SC-28, SC-28(1) | F-004, F-010 | Open |
| Administrative | §164.308(a)(1) | Security Management Process | | Standard | Implement policies and procedures to prevent, detect, contain, and correct security violations | RA-2, MP-4, SI-12, AU-6, SI-4, IR-6 | F-001 through F-011 | Open |
| Administrative | §164.308(a)(1)(ii)(A) | Security Management Process | Risk Analysis | Required | Conduct an accurate and thorough assessment of potential risks and vulnerabilities to the confidentiality, integrity, and availability of ePHI | RA-2, RA-3, RA-5 | F-001 through F-011 | Open |
| Administrative | §164.308(a)(1)(ii)(B) | Security Management Process | Risk Management | Required | Implement security measures sufficient to reduce risks and vulnerabilities to ePHI to a reasonable and appropriate level | CA-5, PL-2, PM-4, RA-3, SI-12 | F-001 through F-011 | Open |
| Administrative | §164.308(a)(1)(ii)(D) | Security Management Process | Information System Activity Review | Required | Implement procedures to regularly review records of information system activity such as audit logs, access reports, and security incident tracking reports | AC-2(4), AU-6, AU-12, CA-7, SI-4 | F-008, F-011 | Open |
| Administrative | §164.308(a)(6) | Security Incident Procedures | | Standard | Implement policies and procedures to address security incidents | IR-1, IR-8 | F-008, F-011 | Open |
| Administrative | §164.308(a)(6)(ii) | Security Incident Procedures | Response and Reporting | Required | Identify and respond to suspected or known security incidents; mitigate harmful effects; document incidents and their outcomes | IR-4, IR-5, IR-6, IR-9 | F-008, F-011 | Open |
| Physical | §164.310(d)(1) | Device and Media Controls | | Standard | Implement policies and procedures governing the receipt and removal of hardware and electronic media containing ePHI, and movement of these items | MA-3, MA-4, MP-2, MP-4, MP-5, MP-6, MP-7 | F-004, F-010 | Open |
| Organizational | §164.316(b)(1) | Documentation | | Required | Maintain written policies and procedures implemented to comply with the Security Rule; retain documentation for six years from creation or last effective date | PM-1, PM-9, PL-1, SA-5 | F-001 through F-011 | Open |

---

## Notes by Citation

### §164.312(a)(2)(i) — Unique User Identification
Unique identifiers exist but the verification chain is broken. Unused programmatic keys with no MFA mean activity cannot be reliably attributed to a specific user.

### §164.312(a)(2)(iv) — Encryption and Decryption
Addressable. Determined applicable given PHI-like maternal health data processed in a cloud environment. Encryption of ePHI at rest is effectively required for this organization.

### §164.312(b) — Audit Controls
F-011 (AU-2, AU-12): no VPC flow logs means a category of activity in a system containing ePHI is not being recorded at all. That is a direct audit controls gap. F-008 (AU-6, SI-4): CloudTrail satisfies the logging requirement but the absence of CloudWatch integration means logs are not being reviewed or acted on. More of an audit review gap than an audit controls gap.

### §164.312(d) — Person or Entity Authentication
No MFA enrolled for any user. Unused programmatic access keys with no usage tracking mean activity cannot be reliably tied to a verified individual.

### §164.312(e)(1) — Transmission Security
Unrestricted SSH and MySQL access expose transmission channels. Absence of a private subnet means all traffic traverses the public network tier. No data classification policy means protection requirements for transmissions are undefined.

### §164.312(e)(2)(ii) — Encryption
Addressable. Determined applicable. ePHI is transmitted between users and platform systems and stored for ongoing access. Encryption in transit and at rest is effectively required given the sensitivity of maternal health data and the cloud-based architecture.

### §164.308(a)(1) — Security Management Process
No formal security management program exists. IAM misconfigurations (no MFA, root account in use, unused access keys) indicate an absence of preventive policies and enforcement procedures. No data classification policy exists to define handling requirements for ePHI. No CloudWatch alerting is in place to detect, contain, and correct violations in a timely manner.

### §164.308(a)(1)(ii)(A) — Risk Analysis
No formal risk analysis process exists. This assessment surfaced 11 findings including a critically misconfigured public S3 bucket containing PHI-like data that a formal ongoing risk analysis program would have identified and remediated earlier.

### §164.308(a)(1)(ii)(B) — Risk Management
No formal POA&M artifact exists. The risk register partially satisfies RA-3.

### §164.308(a)(1)(ii)(D) — Information System Activity Review
No VPC flow logs means network activity in systems containing ePHI is not being recorded. No CloudWatch alerting means logs are not being reviewed or acted on in a timely manner.

### §164.308(a)(6) — Security Incident Procedures
No incident response plan exists as a documented artifact.

### §164.308(a)(6)(ii) — Response and Reporting
IR-9 (Information Spillage) directly applicable given F-004 public S3 bucket containing PHI-like data accessible without authentication.

### §164.310(d)(1) — Device and Media Controls
Cloud interpretation: applies to S3 object lifecycle, EBS volume handling, and data disposal. Document cloud mapping explicitly.

### §164.316(b)(1) — Documentation
Findings report and risk register partially satisfy this requirement. A formal security plan and written policies are still needed.

---

*15 HIPAA Security Rule citations scoped to BirthBFF assessment findings. Framework mapping source: NIST SP 800-66 Rev. 2 (February 2023).*
