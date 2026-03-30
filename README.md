# 🔐 AWS Cloud Security Implementation Project - EmiratesGas IAM 

### (ISO 27001 / NIST / CIS Aligned)

---

## 1. Overview

This project demonstrates the implementation of foundational cloud security controls aligned with **ISO/IEC 27001:2022**, **NIST Cybersecurity Framework (CSF)**, and **CIS Critical Security Controls**.

The objective was to establish a **secure AWS environment** for **emiratesgas.ai** with strong identity governance, access control, logging, monitoring, and alerting mechanisms across multiple UAE locations.

The environment was designed to:

* Secure administrative access
* Enforce separation of duties
* Protect department-specific S3 resources
* Restrict EC2 administrative actions using IAM policies
* Capture and audit API activity via CloudTrail
* Generate automated alerts for sensitive S3, IAM, and EC2 actions

### Cloud Structure
```
emiratesgas.ai
│
└── 🔐 Root Account (MFA Enabled)
    └── 👤 Admin User (emiratesgas.ai-IAM-Admin) MFA Enabled
		│	
        ├── 🧩 IAM (Identity & Access Management)
        │   │
        │   ├── 🟢 emiratesgas.ai-AbuDhabi-Production (Group)
        │   │   └── 👤 emiratesgas.ai-AbuDhabi-Prod-James (User)
        │   │       └── 🖥️ Abu Dhabi Production Instance (EC2)
        │   │           └── 🗂️ abudhabidocuments (S3 Bucket)
        │   │
        │   ├── 🔵 emiratesgas.ai-Dubai-SupplyChain (Group)
        │   │   └── 👤 emiratesgas.ai-Dubai-Supply-Ava (User)
        │   │       └── 🖥️ Dubai Supply Chain Instance (EC2)
        │   │           └── 🗂️ dubai-documents (S3 Bucket)
        │   │
        │   └── 🟣 emiratesgas.ai-Sharjah-HR (Group)
        │       └── 👤 emiratesgas.ai-Sharjah-HR-Fatima (User)
        │           └── 🖥️ Sharjah HR Instance (EC2)
        │               └── 🗂️ sharjah-documents (S3 Bucket)
        │
        ├── 📊 Monitoring & Logging
        │   │
        │   ├── AWS CloudTrail (emiratesgas.ai-event-trail)
        │   │   └── Logs:
        │   │       ├── S3 Data Events
        │   │       └── IAM & EC2 API Activity
        │   │
        │   ├── Amazon EventBridge
        │   │   ├── Rule: S3-Resource-Alert
        │   │   │   └── Events:
        │   │   │       ├── CreateBucket
        │   │   │       └── DeleteBucket
        │   │   │
        │   │   └── Rule: IAM-EC2-Resource-Alert
        │   │       └── Events:
        │   │           └── API Calls via CloudTrail
        │   │
        │   └── Amazon SNS (ResourceAlert)
        │       └── 📧 Email Notifications
        │
        └── 🛡️ Security Principles
            ├── Least Privilege Access
            ├── Role-Based Access Control (RBAC)
            ├── Resource Isolation per Location
            ├── S3 Private & Encrypted Buckets
            ├── EC2 Access Restriction
            ├── CloudTrail Logging Enabled
            └── Real-Time Alerting
```
---

## 2. Governance and Account Security

**ISO 27001 A.5 & A.6 | NIST CSF ID.GV | CIS Control 1**

The AWS root account was secured and designated for **break-glass/emergency use only**. Multi-Factor Authentication (MFA) is recommended to further strengthen account protection.

A dedicated administrative IAM user was created to handle operational activities, ensuring:

* Accountability of administrative actions
* Reduced risk of root account misuse
* Separation between governance and operations

---

## 3. Identity and Access Management (IAM)

**ISO 27001 A.5.15, A.8 | NIST CSF PR.AC | CIS Control 5**

IAM users and groups were implemented using a **Role-Based Access Control (RBAC)** model across three business locations:

### IAM Groups

* emiratesgas.ai-AbuDhabi-Production
* emiratesgas.ai-Dubai-SupplyChain
* emiratesgas.ai-Sharjah-HR

### IAM Users

* emiratesgas.ai-AbuDhabi-Prod-James
* emiratesgas.ai-Dubai-Supply-Ava
* emiratesgas.ai-Sharjah-HR-Fatima

Each user was assigned to a group based on job function, ensuring:

* Least privilege access
* Separation of duties
* Controlled access to specific resources only

---

## 4. Object Storage Security (Amazon S3)

**ISO 27001 A.8.2 | NIST CSF PR.DS | CIS Control 3**

Three S3 buckets were created to isolate data per department:

* `abudhabidocuments` (Production)
* `dubai-documents` (Supply Chain)
* `sharjah-documents` (HR)

Security controls implemented:

* Bucket-level access restricted via IAM policies
* Access limited to assigned group users only
* Logical data isolation across departments

### Access Enforcement

* Abu Dhabi user → Full access to `abudhabidocuments` only
* Dubai user → Full access to `dubai-documents` only
* Sharjah user → Full access to `sharjah-documents` only

Access validation confirmed enforcement of **least privilege principles**.

---

## 5. Compute Resource Access Control (EC2)

**ISO 27001 A.8.9 | NIST CSF PR.AC-4 | CIS Control 4**

Three EC2 instances were deployed to represent departmental workloads:

* AbuDhabi-Production-Instance
* Dubai-Supply-Chain-Instance
* Sharjah-HR-Instance

Access control was implemented using **IAM policies with tag-based conditions**:

* Users can only start/stop instances assigned to their department
* Administrative actions are restricted across environments
* Cross-access between departments is prevented

Policy enforcement ensures:

* Controlled operational access
* Reduced risk of unauthorized instance management

---

## 6. Logging and Monitoring

**ISO 27001 A.8.15 | NIST CSF DE.CM | CIS Control 8**

AWS CloudTrail was configured to capture and log all relevant activities.

### Configuration:

* Trail Name: `emiratesgas.ai-event-trail`
* Logged Events:

  * S3 data events
  * EC2 and IAM API activity

Benefits:

* Full audit trail of user actions
* Support for forensic investigation
* Compliance with audit and governance requirements

---

## 7. Security Event Detection and Alerting

**ISO 27001 A.8.16 | NIST CSF DE.AE | CIS Control 8**

Event-driven monitoring was implemented using EventBridge and SNS.

### EventBridge Rules

1. **S3-Resource-Alert**

   * Detects:

     * CreateBucket
     * DeleteBucket

2. **IAM-EC2-Resource-Alert**

   * Detects:

     * IAM API calls
     * EC2 API activity

### Alerting Mechanism

* SNS Topic: `ResourceAlert`
* Protocol: Email notifications

### Outcome

* Near real-time detection of sensitive actions
* Immediate alerting to administrators
* Improved incident response capability

---

## 8. Custom IAM Policy (Sample)

Below is an example of the **least-privilege IAM policy** used for Abu Dhabi Production:

```json id="AbuDhabi-Bucket-Policy"
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Statement1",
      "Effect": "Allow",
      "Action": [
        "s3:ListAllMyBuckets"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Statement2",
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::abudhabidocuments"
    }
  ]
}
```

---

## 9. Conclusion

This project demonstrates the practical implementation of **internationally recognized cloud security frameworks** within an AWS environment.

The deployed controls align with:

* Governance (IAM, root account protection)
* Protection (S3 & EC2 access control)
* Detection (CloudTrail, EventBridge)
* Response (SNS alerting)

The solution is:

* Audit-ready
* Scalable across multiple locations
* Suitable for enterprise-grade cloud security architecture

---

## 👨‍💻 Author

**Ahmed Olatunji** — Cybersecurity Analyst

---
