## How to do RestAssured security testing

# Client certificates and proxies are only a small part of the overall security validation landscape. Below are the most common security features that can be tested or configured using RestAssured.

# 1. Proxy Configuration

Useful for intercepting API traffic through Burp Suite, OWASP ZAP, Fiddler, Charles Proxy, etc.

```java
RestAssured.given()
    .proxy("localhost", 8080)
    .when()
    .get("https://api.company.com");
```

**Validation Scenarios**
- Request/response tampering
- Security header verification
- Token interception testing
- SSL/TLS inspection
- API penetration testing

# 2. Client Certificate Authentication (mTLS)

```java
RestAssured.given()
    .keyStore("client.p12", "password")
    .when()
    .get("https://secure-api.company.com");
```

**Validation Scenarios**
- Mutual TLS authentication
- Invalid certificate rejection
- Expired certificate handling
- Revoked certificate validation
- Certificate chain verification

# 3. TrustStore Validation

Trust only specific CA certificates.

```java
RestAssured.config =
    RestAssured.config()
        .sslConfig(
            SSLConfig.sslConfig()
                .trustStore(
                    new File("truststore.jks"),
                    "password"));
```

**Test Cases**
- Self-signed certificate rejection
- Untrusted CA validation
- Certificate pinning verification


# 4. Basic Authentication

```java
given()
    .auth()
    .basic("user","password");
```

**Test Cases**
- Weak password validation
- Invalid credentials
- Account lockout
- Brute-force protection

# 5. OAuth 2.0 Authentication
```java
given()
    .auth()
    .oauth2(accessToken);
```

**Test Cases**
- Expired token
- Invalid token
- Missing scope
- Revoked token
- Role-based access


# 6. JWT Security Testing

```java
given()
    .header("Authorization",
            "Bearer " + token);
```
**Test Cases**
- Tampered JWT
- Expired JWT
- Missing signature
- Modified claims
- Weak signing algorithm

Example:
	String tamperedJwt = validJwt.replace("customer","admin");

# 7. API Key Authentication
```java
given()
    .header("x-api-key", apiKey);
```

**Security Checks**
- Missing API key
- Invalid API key
- Expired API key
- Reused API key
- Unauthorized access


# 8. Role-Based Access Control (RBAC)
```java
given()
    .header("Authorization",
             "Bearer userToken");
```

**Tests**
| User Role | Expected Access |
|-----------|-----------------|
| User | Access own data |
| Admin | Access all data |
| Guest | Access denied |
| Anonymous | 401 Unauthorized |

# 9. TLS / SSL Validation

Verify HTTPS-only communication.
```java
given()
    .relaxedHTTPSValidation()
    .get("https://api.company.com");
```

**Test Scenarios**
- Expired certificate
- TLS version validation
- Weak cipher suites
- Protocol downgrade attack
- Invalid hostname
# 10. Security Headers Testing
```java
Response response =
    given().get(url);

response.getHeader("Strict-Transport-Security");
```

- Common Headers

| Header | Purpose |
|---------|---------|
| HSTS | HTTPS enforcement |
| CSP | XSS protection |
| X-Frame-Options | Clickjacking prevention |
| X-Content-Type-Options | MIME protection |

- Referrer-Policy	Privacy control

# 11. Rate Limiting Validation
```java
for(int i=1;i<=100;i++){
    given().get(url);
}
```

**Expected Results**
- HTTP 429
- Retry-After header
- Request throttling

# 12. CORS Security Testing
```java
given()
.header("Origin",
        "http://evil.com")
.get(url);
```
**Verify**
- Allowed origins
- Wildcard origin usage
- Credential leakage

# 13. Authorization Testing

Test vertical and horizontal privilege escalation.

```java
given()
.pathParam("customerId","1002")
.header("Authorization", user1Token)
.get("/customers/{customerId}");
```

**Expected**:
	403 Forbidden

# 14. SQL Injection Testing
```java
given()
.queryParam("username",
            "' OR 1=1 --");
```

**Expected**:
	400 Bad Request

or
	No data returned


# 15. Command Injection Testing
```java
given()
.queryParam("file",
            "; cat /etc/passwd");
```

**Expected**:
	Request rejected

# 16. XXE (XML External Entity)
```xml
String xmlPayload =
"""
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<data>&xxe;</data>
""";
```

**Expected**:
	400 Bad Request

# 17. Sensitive Data Exposure

Validate that PII is masked.

```java
Response response =
    given().get(url);

response.prettyPrint();
```

**Verify Absence Of PII / PHI / Sensitive Data** eg
- PAN
- Aadhaar
- SSN
- Credit Card Number
- CVV
- Passwords
- Secret Keys

# 18. File Upload Security
```java
given()
.multiPart("file",
           maliciousFile)
.post("/upload");
```
**Tests**
- Executable uploads
- Malware uploads
- Oversized files
- Content-type spoofing

# 19. Session Security
```java
String sessionId = response.getCookie("JSESSIONID");
```

**Validate**:
- Session timeout
- Logout invalidation
- Session fixation
- Session hijacking protection

# 20. Security Logging & Audit Validation

Verify sensitive data is not logged.
```java
given()
.log().all();
```

**Check**:
- Password masking
- Token masking
- API key masking
- PII masking

## Areas of Security Testing with RestAssured

- Proxy / Traffic Interception
- SSL/TLS Validation
- Mutual TLS (mTLS)
- TrustStore Validation
- OAuth2
- JWT Security
- API Keys
- RBAC / ABAC
- Rate Limiting
- CORS Validation
- SQL Injection
- XXE Injection
- Command Injection
- Sensitive Data Exposure
- Session Management
- Security Headers
- File Upload Security
- Certificate Expiry Checks
- Audit Logging Validation
- OWASP API Security Top 10 Coverage
