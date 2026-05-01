# Environment Overview

## System Description
BirthBFF is an AI-powered maternal health platform designed to support expecting mothers through education, planning, and personalized guidance.

The platform allows users to:
- Interact with an AI chatbot for childbirth education
- Save conversations and notes
- Create and export birth plans
- Access resources provided by doulas or administrators

## Users
- Clients (expecting mothers)
- Doulas (support providers)
- Administrators (platform management)

## Data Types
- User account information
- Chat conversations (may contain sensitive health-related information)
- Birth plans and notes
- Uploaded or generated documents

## AWS Components (Planned)
- Amazon S3 (storage for documents and exports)
- IAM (user roles and access control)
- AWS CloudTrail (logging and audit trail)
- AWS CloudWatch (monitoring and alerts)
- Amazon EC2 (web/API application layer)
- Amazon RDS MySQL (relational database used to store user account metadata, saved chat history metadata, and birth plan records)
- RDS or DynamoDB (data storage)

## Security Considerations
This assessment focuses on:
- Identity and access management (IAM)
- Data protection and classification
- Logging and monitoring
- Risk identification and remediation

## Compliance Scope

The BirthBFF platform processes user-generated content that may include
health-related and potentially sensitive personal information. For the
purposes of this assessment, this data is treated as Protected Health
Information (PHI)-like data to evaluate security and privacy risks.

Controls are assessed against:
- SP 800-53 Rev. 5
- Security Rule (45 CFR Part 164)
- AWS Foundations Benchmark v1.4

This is a pre-production security and GRC assessment and does not represent
a certified HIPAA-compliant environment. The assessment focuses on identifying gaps and risks relative to these frameworks, not achieving full compliance.
