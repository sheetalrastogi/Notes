## API Security Testing Examples

API Security Testing ensures that APIs are protected against unauthorized access, data exposure, injection attacks, rate abuse, and other security vulnerabilities.

## 1. Authentication Testing

Verify that only authenticated users can access protected APIs.

# Valid Token
	GET /api/customers
	Authorization: Bearer valid_token

# Expected:
	200 OK

# Invalid Token
	GET /api/customers
	Authorization: Bearer invalid_token
# Expected:
	401 Unauthorized

# Missing Token
	GET /api/customers
# Expected:
	401 Unauthorized

**Rest Assured Example**
```java
    given()
    .header("Authorization", "Bearer invalid_token")
    .when()
    .get("/customers")
    .then()
    .statusCode(401);
```

## 2. Authorization Testing (Role-Based Access)

Verify users cannot access resources beyond their role.

Customer Accessing Admin API
GET /api/admin/users
Authorization: Bearer customer_token

Expected:
	403 Forbidden

Admin Access
GET /api/admin/users
Authorization: Bearer admin_token

Expected:
	200 OK

## 3. Broken Object Level Authorization (BOLA)

Most common API vulnerability.
User A:		GET /api/account/1001
User B: 	Attempts Access non-existent resource:
	GET /api/account/1001

Expected:
	403 Forbidden

Not:	
	200 OK

## 4. SQL Injection Testing
Vulnerable Request
GET /api/customer?id=1' OR '1'='1

Expected:
	400 Bad Request  or   403 Forbidden

Never:
	Entire Customer Database

Rest Assured Example

```java
given()
    .queryParam("id",
         "1' OR '1'='1")
.when()
    .get("/customer")
.then()
    .statusCode(anyOf(
        is(400),
        is(403)));
```

## 5. JWT Security Testing
**Tampered JWT Token** 
	Original:  eyJh...

Modified:  eyJhAAAA...

Expected:
	401 Unauthorized

**Expired JWT**
exp = Yesterday

Expected:
	401 Unauthorized

## 7. Sensitive Data Exposure

Verify API responses do not expose:

Bad Example
{
   "password":"Admin123",
   "ssn":"123-45-6789",
   "creditCard":"4111111111111111"
}

Good Example
{
   "name":"John",
   "email":"john@gmail.com"
}


Check for:
- Passwords
- Credit Card Numbers
- CVV
- SSN
- Aadhaar
- PAN
- Tokens

## 8. Mass Assignment Testing
Request
{
   "name":"John",
   "email":"john@test.com",
   "isAdmin":true
}

Expected:
	isAdmin ignored

**Attackers should not gain elevated privileges.**

## 9. Rate Limiting Testing

Simulate 1000 Requests
```java
for(int i=0; i<1000; i++) {
    given()
      .when()
      .get("/login");
}
```

Expected:
	429 Too Many Requests

Verify:
- Rate limits
- Account lockout
- Throttling

## 10. Brute Force Login Testing

Repeated Login Attempts
	POST /api/login

with:
	admin/admin1
	admin/admin2
	admin/admin3
	...

Expected:
	Account Locked  or
	429 Too Many Requests

## 11. HTTP Method Tampering

Allowed:
	GET /customers
Try:
	PUT /customers
	DELETE /customers
	PATCH /customers
	TRACE /customers

Expected:
	405 Method Not Allowed

## 12. Security Headers Validation

Verify headers explicitly:
- X-Frame-Options
- X-XSS-Protection
- Content-Security-Policy
- Strict-Transport-Security

Example:
```java
given()
.when()
.get("/api/user")
.then()
.header("Strict-Transport-Security", notNullValue());
```

## 13. HTTPS Enforcement

Bad:
	http://company.com/api
Expected:
	301 Redirect
	to:		https://company.com/api

## 14. File Upload Security Testing

Non supported Uploads eg:
- virus.exe
- malware.bat
- shell.jsp

Expected:
	Upload Rejected

## 15. XML Injection Testing

Payload

```xml

<user>
 <name>' OR '1'='1</name>
</user>
```

Expected:
	Input Validation Failure

## 16. XXE Testing
Malicious Payload
```xml

<?xml version="1.0"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<user>
    <name>&xxe;</name>
</user>
```
Expected:
	Request Rejected

## 17. Parameter Tampering
Original Request
	GET /policy/1001

Try:
	GET /policy/1002

Expected (if user doesn't own policy.):
	Forbidden



## 18. API Version Security Testing
/api/v1/customer
/api/v2/customer
/api/v3/customer

Verify:
- Old versions disabled
- Deprecated versions blocked
- Same security rules enforced

## 19. CORS Security Testing

Validate:
	Access-Control-Allow-Origin

Bad:
	*

Expected:
	https://company.com

## 20. Business Logic Security Testing

Example:  Insurance Domain:
	User policy limit:	$10,000
	Claim submitted:	$100,000

Expected:
	Validation Error
	Not:	Claim Approved


# OWASP API Security Top 10 Test Scenarios

| Vulnerability | Example Test Scenario |
|--------------|----------------------|
| **Broken Object Level Authorization (BOLA)** | Access another user's policy by modifying Policy ID in the request URL. |
| **Broken Authentication** | Verify whether an invalid, expired, or tampered JWT token is accepted. |
| **Broken Object Property Authorization** | Attempt to update hidden or restricted fields such as `isAdmin=true`. |
| **Unrestricted Resource Consumption** | Flood the API with excessive requests and verify rate limiting/throttling controls. |
| **Broken Function Level Authorization** | Verify that a customer user cannot invoke Admin-only APIs. |
| **Sensitive Business Flows** | Simulate mass claim submissions or bulk transactions to detect abuse scenarios. |
| **Server-Side Request Forgery (SSRF)** | Attempt to force the API to access internal servers or restricted network resources. |
| **Security Misconfiguration** | Verify that verbose error messages, stack traces, or sensitive configuration details are not exposed. |
| **Improper Inventory Management** | Check for unsecured, deprecated, or undocumented API versions. |
| **Unsafe Consumption of APIs** | Verify that data received from external APIs is properly validated and not blindly trusted. |

## Key Security Validation Areas

- Authorization Controls
- Authentication Mechanisms
- Input Validation
- Rate Limiting & Throttling
- Secure Business Logic
- Third-Party API Security
- Error Handling & Logging
- API Version Governance
- Secure Configuration Management
- Data Protection & Privacy
