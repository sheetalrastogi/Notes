# SOAP API Penetration Testing Checklist

## 1. XML-Based Attacks

- XML External Entity (XXE) Injection
- XML Entity Expansion (Billion Laughs Attack)
- SOAP Message Tampering
- XPath Injection
- XSLT Injection
- XML Schema Poisoning
- XML Signature Wrapping Attack
- Malformed XML Parsing Attacks

---

## 2. Authentication & Authorization

### Authentication Tests

- Broken Authentication
- Weak WS-Security Token Validation
- Username Token Replay Attack
- Session Hijacking

### Authorization Tests

- Privilege Escalation
- Broken Object Level Authorization (BOLA)
- Broken Function Level Authorization (BFLA)

---

## 3. Injection Attacks

- SQL Injection
- NoSQL Injection
- LDAP Injection
- Command Injection
- XPath Injection
- SOAP Header Injection

---

## 4. Message-Level Security

- SOAP Header Manipulation
- WS-Security Bypass
- Digital Signature Validation Bypass
- Message Replay Attack
- Encryption Downgrade Attack
- Certificate Validation Bypass

---

## 5. Data Exposure & Information Disclosure

- Sensitive Data Exposure
- Information Leakage through SOAP Fault Messages
- WSDL Information Disclosure
- Unencrypted Payload Transmission
- Sensitive Data in Logs

---

## 6. Transport Layer Security (TLS/SSL)

- Weak TLS/SSL Configuration
- Deprecated Cipher Suites
- Man-in-the-Middle (MITM) Vulnerability
- Certificate Spoofing

---

## 7. Denial of Service (DoS)

- XML Bomb (Billion Laughs Attack)
- Oversized SOAP Request Attack
- Recursive Payload Attack
- Large Attachment (MTOM/SWA) Attack
- Resource Exhaustion Attack

---

## 8. Web Application Security Tests

- Cross-Site Scripting (XSS)
- Cross-Site Request Forgery (CSRF)
- Open Redirect
- Server-Side Request Forgery (SSRF)

---

## 9. API-Specific Security Tests

- WSDL Enumeration
- Endpoint Enumeration
- Rate Limiting Validation
- Brute Force Attack Resistance
- API Abuse Testing
- Input Validation Testing
- Parameter Tampering
- Mass Assignment Testing

---

## 10. Common SOAP Security Test Scenarios

### Authentication

- Invalid UsernameToken
- Expired UsernameToken
- Missing Authentication Headers
- Replay Authentication Requests

### Authorization

- Access Admin Operations as Normal User
- Access Restricted Resources Across Roles
- Force Browse Unauthorized SOAP Operations

### XML Processing

- XXE Payload in SOAP Body
- Billion Laughs XML Payload
- Malformed XML Structures
- External DTD References

### Input Validation

- SQL Injection in XML Elements
- XPath Injection
- Command Injection
- Special Character Fuzzing

### Message Integrity

- Modify Signed SOAP Message
- Alter SOAP Header Values
- Signature Wrapping Tests
- Remove Security Headers

### Confidentiality

- Verify Encrypted SOAP Body
- Verify Encrypted Security Tokens
- Verify Sensitive Field Encryption

### Availability

- Send Large SOAP Requests (>10 MB)
- Recursive XML Payloads
- Large MTOM Attachments
- High Request Volume Testing

### Information Disclosure

- Trigger SOAP Faults and Inspect Stack Traces
- Enumerate WSDL Metadata
- Identify Internal Server Information
- Verify Error Message Sanitization

### Replay Protection

- Resend Signed Requests
- Reuse Security Tokens
- Replay Authentication Messages

### Transport Security

- Test TLS 1.0 / 1.1 Support
- Downgrade TLS Versions
- Weak Cipher Validation
- Certificate Validation Bypass

---

## 11. OWASP-Aligned SOAP Security Checklist

### Injection Vulnerabilities

- SQL Injection
- XPath Injection
- LDAP Injection
- Command Injection

### Authentication & Authorization

- Broken Authentication
- Broken Authorization
- BOLA
- BFLA

### Security Misconfiguration

- Security Headers Missing
- Unnecessary SOAP Endpoints Exposed
- WSDL Exposure

### Cryptographic Failures

- Weak TLS Configuration
- Weak Encryption Algorithms
- Insecure Key Management

### Server-Side Request Forgery (SSRF)

- SSRF Through XML Payloads
- External Resource Access Validation

### Sensitive Data Exposure

- Unencrypted Sensitive Data
- Sensitive Data in Logs
- Excessive Error Information

### API Abuse

- Rate Limiting Bypass
- Resource Abuse
- Excessive Consumption Attacks

### Denial of Service

- XML Bomb
- Recursive Payload Attack
- Oversized Message Attack

### Logging & Monitoring Validation

- Security Event Logging
- Authentication Failure Logging
- Audit Trail Validation
- SIEM Integration Validation

---

# Penetration Testing Coverage Summary

### XML Security
- XXE
- XML Bomb
- XPath Injection
- XML Signature Wrapping
- XML Schema Poisoning

### Authentication & Authorization
- Broken Authentication
- Replay Attacks
- Privilege Escalation
- BOLA
- BFLA

### Injection Testing
- SQL Injection
- NoSQL Injection
- LDAP Injection
- Command Injection
- SOAP Header Injection

### Confidentiality & Integrity
- Encryption Validation
- Signature Validation
- Message Tampering

### Availability
- DoS
- Resource Exhaustion
- Large Payload Testing

### Information Disclosure
- SOAP Fault Analysis
- WSDL Enumeration
- Sensitive Data Exposure

### Transport Security
- TLS Validation
- Cipher Validation
- MITM Resistance

### OWASP API Security Alignment
- API Abuse
- SSRF
- Broken Access Control
- Security Misconfiguration
- Cryptographic Failures

---

**Objective:** Assess SOAP APIs for vulnerabilities related to confidentiality, integrity, availability, authentication, authorization, message security, transport security, and OWASP API Security risks.
