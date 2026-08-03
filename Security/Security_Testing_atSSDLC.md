## Security Activities during SSDLC (Secure Software Development Lifecycle) phases

Explain what security activities and validations occur at each lifecycle stage rather than listing security testing types such as SAST, DAST, or Pen Testing.

```text
SSDLC Security Flow
Requirements
│
├── Security Requirements
├── Compliance Requirements
└── Data Classification
        ↓

Design
│
├── Threat Modeling
├── Architecture Review
└── Risk Assessment
        ↓

Development
│
├── Secure Coding
├── Code Reviews
└── Secret Management
        ↓

Build
│
├── Dependency Review
├── Pipeline Security
└── Container Validation
        ↓

Testing
│
├── Authentication Validation
├── Authorization Validation
├── Data Protection Verification
└── Business Logic Validation
        ↓

Deployment
│
├── Hardening
├── Secure Configuration
└── Production Readiness
        ↓

Operations
│
├── Monitoring
├── Logging
└── Incident Detection
        ↓

Maintenance
│
├── Patch Management
├── CVE Remediation
└── Continuous Improvement

```


# SSDLC Security Activities by Lifecycle Stage

```text
Requirements
      ↓
Design
      ↓
Development
      ↓
Build & Integration
      ↓
Testing & Validation
      ↓
Deployment
      ↓
Operations & Monitoring
      ↓
Maintenance
```

# 1. Requirements Phase
**Objective**

Identify security requirements before implementation begins.

- Security Activities
- Define security requirements
- Identify regulatory requirements
- Identify sensitive data (PII, PHI, PCI)
- Define authentication requirements
- Define authorization model
- Define encryption requirements
- Define audit and logging requirements

Examples: Banking Application
---

- MFA required
- Password expiry every 90 days
- Account lock after 5 failed attempts

Insurance Application
---

- Encrypt customer SSN/PAN
- Mask PII in UI
- Maintain 7-year audit logs

Deliverables
---
- Security Requirements Specification
- Compliance Requirements Matrix
- Data Classification Document

## 2. Design Phase
---
**Objective**

Build security into architecture before coding starts.

**Security Activities**

- Threat Modeling
- Attack Surface Analysis
- Security Architecture Review
- Trust Boundary Identification
- Abuse Case Analysis
- Security Design Reviews

Examples
-----

**Threat Modeling**
Threat: Unauthorized access to claims data

Mitigation: RBAC + Encryption

**Architecture Review**

```text
API Gateway
      ↓
Authentication Layer
      ↓
Microservices
      ↓
Database
```

Validate:
----
- TLS enabled
- Secrets management
- Secure API design
- Network segmentation

Deliverables
------

- Threat Model Document
- Security Architecture Diagram
- Risk Register

## 3. Development Phase
**Objective**

- Develop secure code from the beginning.

**Security Activities**

- Secure coding practices
- Code reviews
- Input validation verification
- Output encoding design
- Error handling validation
- Cryptography implementation review
- Secrets management review

**Examples**
---

- Input Validation
- Validate email format
- Validate file type
- Validate request schema
- Secure Coding Checks
- No hardcoded passwords
- No SQL string concatenation
- No insecure deserialization
- No plaintext logging

Deliverables
---

- Secure Code Review Checklist
- Coding Standards Compliance Report

## 4. Build & Continuous Integration Phase
**Objective**

Prevent vulnerable code from entering release pipelines.

Security Activities
- Dependency validation
- Open-source package review
- Build pipeline security checks
- Secrets scanning
- Container security validation
- Infrastructure-as-Code review

Examples
---
- Dependency Validation
- Spring Framework version reviewed
- Log4j vulnerabilities checked
- Apache Commons vulnerabilities reviewed
- Build Validation
- No secrets in repository
- Secure environment variables
- Signed build artifacts

Deliverables
---

- Dependency Risk Report
- Pipeline Security Report

## 5. Testing & Validation Phase
**Objective**

- Validate security controls implemented during development.

**Security Activities**

- Authentication validation
- Authorization validation
- Session management verification
- Input validation verification
- Encryption validation
- API security validation
- Business logic abuse validation
- Security regression validation

Examples
----

**Authentication**
- Invalid login attempts blocked
- MFA enforcement verified
- Password policy enforced

**Authorization**
- Customer cannot access another customer's policy
- Agent cannot access admin functions

**Data Security**
- PII masked in UI
- Data encrypted in transit
- Data encrypted at rest

Deliverables
-----

- Security Validation Report
- Risk Assessment Report

## 6. Deployment Phase

**Objective**

- Ensure secure production release.

Security Activities
---

- Secure configuration review
- Environment hardening
- TLS certificate verification
- Secret rotation verification
- Access control verification
- Deployment checklist validation

Examples
---

- Server Validation
- Debug mode disabled
- Directory browsing disabled
- TLS certificates configured

Cloud Validation
---

- Storage buckets private
- Security groups restricted
- Least privilege IAM roles

Deliverables
---

- Deployment Security Checklist
- Production Readiness Review

## 7. Operations & Monitoring Phase
**Objective**

Detect and respond to security threats in production.

Security Activities
---

- Security monitoring
- Audit log review
- Incident detection
- Threat intelligence monitoring
- Anomaly detection
- User activity analysis

Examples
-----

**Monitoring**
- 100 failed login attempts detected
- Potential brute-force attack
- Alerting
- Admin login from unknown country
- Generate SOC alert

Deliverables
---

- Security Monitoring Dashboard
- Incident Reports
- SOC Metrics

## 8. Maintenance Phase
**Objective**

Maintain security posture after release.

Security Activities
---
- Security patching
- Vulnerability remediation
- Certificate renewal
- Dependency upgrades
- Security enhancements
- Compliance reassessment

Examples
---

Vulnerability Fix:  Spring Framework CVE discovered
- Upgrade framework version
- Redeploy application

Certificate Management: TLS certificate nearing expiration

- Renew before expiry

Deliverables
---

- Patch Management Report
- Vulnerability Remediation Report


QA/Test Architect Perspective

For each SSDLC stage, the focus is:

SSDLC Stage	Security Validation FocusRequirements	Security and compliance requirements coverage
Design	Threat modeling and architecture risks
Development	Secure coding and code reviews
Build	Dependency, secret, and pipeline security
Testing	Verification of implemented security controls
Deployment	Hardening and configuration validation
Operations	Monitoring, logging, and incident detection
Maintenance	Patch management and vulnerability remediation

This lifecycle-oriented view is how mature organizations implement Security by Design and Shift-Left Security within SSDLC.
