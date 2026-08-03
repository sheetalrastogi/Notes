## Security testing Key Areas to test for a distributed client-server application

Security defects can originate from both the Software Development phase (secure coding/design issues) and the Deployment/Infrastructure phase (configuration and environment weaknesses).

# 1. Security Defects from Software Development Perspective
**Authentication Defects**
```text
| Defect                         | Example                                             |
|--------------------------------|-----------------------------------------------------|
| Weak Password Policy           | Password "Welcome123" accepted                      |
| Credential Stored in Plain Text| Passwords saved directly in database                |
| Missing MFA Enforcement        | Privileged users can bypass MFA                     |
| Session Fixation               | Session ID not regenerated after successful login   |
| Broken Login Throttling        | Unlimited login attempts allowed                    |
| Password Reset Vulnerability   | Reset link is predictable or can be reused          |
```

**Authorization Defects**

```text
| Defect                                | Example                                           |
|----------------------------------------|--------------------------------------------------|
| Broken Access Control                  | User accesses another customer's records         |
| Privilege Escalation                   | Standard user accesses admin APIs                |
| Insecure Direct Object Reference (IDOR)| Changing AccountID reveals other accounts        |
| Missing Role Validation                | UI hides feature but API still allows access     |
| Horizontal Privilege Escalation        | User edits another user's profile                |
| Vertical Privilege Escalation          | Tester accesses administrator functions          |
```
**Input Validation Defects**
```text
| Defect Type     | Example |
|----------------|----------|
| SQL Injection | `' OR 1=1--` bypasses authentication |
| NoSQL Injection | Mongo query manipulation |
| Command Injection | User input executed by server shell |
| LDAP Injection | LDAP filters manipulated |
| XML Injection | Crafted XML changes processing logic |
| Path Traversal | `../../etc/passwd` exposure |
```
**API Security Defects**
```text
| Defect                     | Example                                        |
|----------------------------|------------------------------------------------|
| Missing Authentication     | Public API accessible anonymously              |
| Excessive Data Exposure    | API returns SSN, DOB unnecessarily             |
| Weak JWT Implementation    | Token tampering possible                       |
| Hardcoded API Keys         | API keys visible in source code                |
| Missing Rate Limiting      | API vulnerable to Denial-of-Service (DoS) attacks |
| Missing Schema Validation  | Invalid payload causes application/API crash   |
```
**Session Management Defects**
```text
| Defect                           | Example                                        |
|----------------------------------|------------------------------------------------|
| Long-Lived Sessions              | Session never expires                          |
| Session Token Exposed in URL     | Token appears in browser history and logs      |
| Weak Token Generation            | Predictable session identifiers                |
| Missing Logout Invalidation      | Token remains active after user logout         |
| Concurrent Session Abuse         | Same session used simultaneously from multiple devices |

```

**Encryption Defects**

```text
| Defect                          | Example                                      |
|---------------------------------|----------------------------------------------|
| Weak Cryptographic Algorithms   | Use of MD5 or SHA-1 for hashing/encryption   |
| Hardcoded Encryption Keys       | Encryption key embedded in source code       |
| Sensitive Data Stored Unencrypted | PAN, PII, or confidential data stored in plaintext |
| Insecure Random Generation      | Predictable OTP or token values generated    |
| Improper Key Rotation           | Expired or compromised keys continue to be used |
```

**Client Side Security Defects**
```text
| Defect                        | Example                                          |
|-------------------------------|--------------------------------------------------|
| Cross-Site Scripting (XSS)    | Script injection through input fields            |
| Sensitive Data in Browser Cache | Customer data remains accessible offline        |
| Secrets in JavaScript         | API tokens or credentials exposed in client-side code |
| Clickjacking Vulnerability    | Missing X-Frame-Options or frame protection headers |
| Local Storage Abuse           | JWT or sensitive tokens stored insecurely in local storage |
```

# 2. Security Defects from Deployment Perspective

**Network Security Defects**

```text
| Defect                       | Example                                      |
|------------------------------|----------------------------------------------|
| Open Unnecessary Ports       | Telnet service open on production servers    |
| Insecure Communication       | HTTP used instead of HTTPS                   |
| Missing Network Segmentation | Application server directly exposed to internet |
| Weak Firewall Rules          | Any-to-any traffic allowed across network zones |
| Misconfigured Load Balancer  | Load balancer configuration bypasses WAF protection |
```

**SSL/TLS Configuration Defects**

```text
| Defect                    | Example                               |
|---------------------------|---------------------------------------|
| Expired Certificate       | TLS certificate has expired           |
| Self-Signed Certificate   | Certificate not issued by a trusted CA |
| Weak Cipher Suites        | RC4 cipher enabled                    |
| TLS 1.0/1.1 Enabled       | Legacy TLS protocols still supported  |
| Missing HSTS              | Downgrade attacks possible            |
```

**Server Hardening Issues**

```text
| Defect                         | Example                                  |
|--------------------------------|------------------------------------------|
| Default Admin Accounts Enabled | admin/admin credentials active           |
| Unpatched OS                   | Known CVEs remain exploitable            |
| Unpatched Middleware           | Vulnerable Tomcat/WebLogic deployment    |
| Directory Listing Enabled      | Sensitive files exposed through browsing |
| Debug Mode Enabled             | Detailed stack traces visible to users   |
```

**Container Security Defects**

```text
| Defect Type | Example |
|------------|----------|
| Container Runs as Root | Privilege escalation risk |
| Vulnerable Base Images | Known CVEs present in the container image |
| Secrets Baked into Image | Database passwords stored in Dockerfile or image layers |
| Insecure Image Repository | Unsigned or untrusted images deployed |
| Excessive Container Privileges | Container granted host-level access |
```

**Kubernetes Security Defects**
```text
| Defect Type | Example |
|------------|----------|
| Privileged Pods | Direct node access possible |
| Open Dashboard | Public Kubernetes dashboard accessible without proper restrictions |
| Weak RBAC Policies | Cluster-admin privileges granted to developers |
| Secrets Stored Unencrypted | Kubernetes secrets exposed in plain text |
| Missing Network Policies | Unrestricted pod-to-pod network communication |
```
***Cloud Deployment Defects**
```text
| Defect Type | Example |
|------------|----------|
| Public Storage Buckets | Sensitive documents exposed to unauthorized users |
| Open Security Groups | Database accessible directly from the internet |
| Excessive IAM Permissions | Administrative access granted unnecessarily |
| Missing Encryption at Rest | Database data stored without encryption |
| Lack of Audit Logging | User activities and actions cannot be traced |
```

## 3. Distributed-System-Specific Security Defects

**Service-to-Service Communication**
```text
| Defect Type | Example |
|------------|----------|
| No Mutual TLS (mTLS) | Services trust any caller without verifying service identity |
| Missing Service Authentication | Internal APIs exposed without authentication controls |
| Internal API Directly Reachable | Requests bypass API Gateway and associated security policies |
| Token Forwarding Vulnerability | Authentication token replay possible across multiple services |
| Sensitive Data in Message Queue | Plaintext messages containing sensitive data stored in Kafka/RabbitMQ |
```
**Microservices Security Defects**

```text
| Defect Type | Example |
|------------|----------|
| Missing API Gateway Validation | Requests directly bypass gateway controls and hit backend services |
| Shared Credentials Across Services | Compromise of one service affects all connected services |
| Insecure Service Discovery | Rogue or unauthorized service registration possible |
| Weak Inter-Service Authorization | Any service can access resources belonging to other services |
| Missing Request Signing | Request tampering or replay attacks possible |
```
**Distributed Cache Defects**
```text
| Defect Type | Example |
|------------|----------|
| Open Redis Access | No authentication enabled |
| Cached Sensitive Information | Authentication tokens or sensitive data stored in cache |
| Cache Poisoning | Malicious response inserted into cache |
| Unencrypted Cache Replication | Data interception possible during replication |
```

## Summary of Security Defects Typically Found During Security Testing of Applications

- Authentication bypass through API.
- SQL Injection leading to database compromise.
- Privilege escalation from User to Admin.
- Hardcoded production credentials in source code.
- Publicly exposed cloud storage buckets.
- Unencrypted PII data stored in databases.
- Service-to-service communication without mTLS.
- Kubernetes dashboard exposed to internet.
- JWT token tampering leading to account takeover.
- Remote Code Execution (RCE) via vulnerable components.
- Secret keys exposed in Git repositories.
- Missing rate limiting causing DoS vulnerability.
- Open database ports accessible externally.
- Session hijacking due to insecure cookies.
- Broken access control exposing customer data.


## Network Communication Security Defects in Distributed Applications

When monitoring network traffic using tools such as **Wireshark, Fiddler, Burp Suite, TCPDump, Charles Proxy, Browser DevTools, or API Gateways**, the following security defects are commonly identified.

```text
| Defect | Example |
|----------|----------|
| Unencrypted Communication | Application transmits credentials over HTTP instead of HTTPS |
| Sensitive Data in Transit | PAN, SSN, account numbers visible in network packets |
| Weak TLS Configuration | TLS 1.0 enabled with vulnerable cipher suites |
| Missing Certificate Validation | Client accepts self-signed certificates |
| API Token Exposure | JWT token visible in URL or request parameters |
| Session Token Leakage | Session IDs transmitted without encryption |
| Insecure Cookie Transmission | Secure flag not enabled on authentication cookies |
| Excessive Data Exposure | API response contains confidential fields not required by client |
| Missing Request Integrity Validation | Requests can be modified during transmission |
| Weak Mutual Authentication | Microservices trust requests without service authentication |
| Replay Attack Vulnerability | Same API request can be replayed multiple times |
| Hardcoded Secrets in Headers | API keys exposed in network requests |
| Missing Rate Limiting | Large volume of requests accepted without throttling |
| Internal Service Exposure | Backend endpoints accessible from public network |
| Data Compression Attacks | Sensitive data vulnerable to BREACH/CRIME attacks |
| Insecure WebSocket Communication | WebSocket traffic transmitted without encryption |
| DNS Leakage | Internal service names exposed through DNS requests |
| Open Service Discovery Endpoints | Eureka/Consul/Kubernetes endpoints publicly accessible |
| Lack of Payload Encryption | Sensitive business data transferred as plain JSON |
| Verbose Error Messages | API returns stack traces and infrastructure details |
```



