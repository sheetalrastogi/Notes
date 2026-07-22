# Contract Testing in Microservices

## Definition

**Contract Testing** is a testing technique used in microservices architecture to verify that the communication contract (request and response structure) between a service consumer and a service provider remains consistent and compatible.

It ensures that:

- The consumer sends requests in the expected format.
- The provider returns responses as agreed.
- Changes in one microservice do not break dependent microservices.
- Integration issues are identified early without requiring full end-to-end testing.

---

# Example Contract

## Scenario

### Consumer Service
**Order Service**

Calls:

```http
GET /customers/101
```

### Expected Response

```json
{
  "id": 101,
  "name": "John Smith",
  "email": "john@test.com"
}
```

The above request and response together form the **API Contract**.

If the Customer Service changes the response from:

```json
{
  "id": 101,
  "name": "John Smith",
  "email": "john@test.com"
}
```

to:

```json
{
  "id": 101,
  "customerName": "John Smith",
  "email": "john@test.com"
}
```

Then the contract test should fail, identifying the breaking change before deployment.

---

# Contract Testing Using Rest Assured

## Common RequestSpecification

```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.specification.RequestSpecification;

public class SpecBuilderUtil {

    public static RequestSpecification getRequestSpec() {

        return new RequestSpecBuilder()
                .setBaseUri("https://api.test.com")
                .addHeader("Content-Type", "application/json")
                .addHeader("Accept", "application/json")
                .build();
    }
}
```

---

## Common ResponseSpecification

```java
import io.restassured.builder.ResponseSpecBuilder;
import io.restassured.specification.ResponseSpecification;

public class SpecBuilderUtil {

    public static ResponseSpecification getResponseSpec() {

        return new ResponseSpecBuilder()
                .expectStatusCode(200)
                .expectContentType("application/json")
                .build();
    }
}
```

---

# Consumer Contract Test Example

## Validate Customer API Contract

```java
import org.testng.annotations.Test;

import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

public class CustomerContractTest {

    @Test
    public void validateCustomerContract() {

        given()
            .spec(SpecBuilderUtil.getRequestSpec())
        .when()
            .get("/customers/101")
        .then()
            .spec(SpecBuilderUtil.getResponseSpec())
            .body("id", equalTo(101))
            .body("name", notNullValue())
            .body("email", containsString("@"));
    }
}
```

---

# Validate Complete Response Schema (Contract)

## JSON Schema

### `customer-schema.json`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": [
    "id",
    "name",
    "email"
  ],
  "properties": {
    "id": {
      "type": "integer"
    },
    "name": {
      "type": "string"
    },
    "email": {
      "type": "string"
    }
  }
}
```

---

## Rest Assured Schema Validation

```java
import static io.restassured.module.jsv.JsonSchemaValidator.*;

@Test
public void verifyCustome*ContractSchema() {

    given()
  *     .spec(SpecBuilderUtil.getRequ*stSpec())
    .when()
        .get*"/customers/101")
    .then()
    *   .spec(SpecBuilderUtil.getRespon*eSpec())
        .body(matchesJson*chemaInClasspath("customer-schema.*son"));
}
```

---

# Reusable Bas* Class Approach

## Base API Test
*```java
import io.restassured.buil*er.RequestSpecBuilder;
import io.r*stassured.builder.ResponseSpecBuil*er;
import io.restassured.specific*tion.RequestSpecification;
import *o.restassured.specification.Respon*eSpecification;

public class Base*piTest {

    protected RequestSpe*ification requestSpec =
          * new RequestSpecBuilder()
                    .setBaseUri("https://api.test.com")
                    .setContentType("application/json")
                    .build();

    protected ResponseSpecification responseSpec =
            new ResponseSpecBuilder()
                    .expectStatusCode(200)
                    .expectContentType("application/json")
                    .build();
}
```

---

## Customer Contract Test

```java
import org.testng.annotations.Test;

import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

public class CustomerContractT*st extends BaseApiTest {

    @Tes*
    public void validateContract(* {

        given()
            .s*ec(requestSpec)
        .when()
  *         .get("/customers/101")
  *     .then()
            .spec(res*onseSpec)
            .body("id", *qualTo(101))
            .body("na*e", notNullValue())
            .b*dy("email", notNullValue());
    }*}
```

---

# Interview Definition*(One-Liner)

> **Contract Testing *s the process of verifying that th* API interaction contract between * microservice consumer and provide* (request, response, headers, stat*s codes, and schemas) remains comp*tible, enabling independent deploy*ents without breaking downstream s*rvices.**

---

# What Contract Te*ting Verifies

✅ HTTP Method (`GET*, `POST`, `PUT`, `DELETE`, etc.)

* Endpoint URI

✅ Request Headers

* Request Payload Structure

✅ Resp*nse Status Code

✅ Response Header*

✅ Response Body Fields

✅ Data T*pes

✅ JSON/XML Schema

✅ Backward*Compatibility

✅ Mandatory Fields
*✅ Optional Fields

✅ API Version C*mpatibility

---

# Contract Testi*g Workflow

```text
Consumer Servi*e
       |
       |  Defines Expec*ations
       v
Contract
       |
*      |  Verified Against
       v*Provider Service
       |
       |* Contract Validation
       v
Buil* Pass / Fail
```

---

# Consumer-*riven Contract (CDC) Testing Flow
*```text
Consumer Team
      |
    * | Creates Contract
      v
Contra*t Repository
      |
      | Provi*er Verification
      v
Provider S*rvice
      |
      | Publish Resu*ts
      v
CI/CD Pipeline
```

---*
# Example Breaking Changes Detect*d by Contract Testing

## Field Re*oval

### Old Response

```json
{
* "id": 101,
  "name": "John Smith"*}
```

### New Response

```json
{*  "id": 101
}
```

✅ Contract Test*Fails

---

## Field Rename

### O*d Response

```json
{
  "name": "J*hn Smith"
}
```

### New Response
*```json
{
  "customerName": "John *mith"
}
```

✅ Contract Test Fails*
---

## Data Type Change

### Old*Response

```json
{
  "id": 101
}
*``

### New Response

```json
{
  *id": "101"
}
```

✅ Contract Test *ails

---

## Response Code Change*
### Expected

```http
200 OK
```
*### Actual*
```http
202 Accepted
```

✅ Contr*ct Test Fails

---

# Popular Cont*act Testing Tools

##*Pact

- Consumer-Driven Contract (*DC) Testing
- Widely adopted*in*microservices ecosystems
- Support* multiple programming languages

#* Spring Cloud Contract

- Contract*first testing for Spring applicati*ns
- Auto-generates tests*and stubs

## Rest Assured + JSON *chema Validation

-*Lightweight*validation of API*contracts
- Commonly used in API a*tomation*frameworks

## Postman Contract Te*ts

- Schema and API validation us*ng collections

## Karate DSL

- A*I testing with built-in contract v*lidation support

---

# Pact vs S*ring Cloud Contract vs Rest Assure*

| Capability | Pact | Spring Clo*d Contract | Rest Assured |
|-----*------|------|--------------------*-|--------------|
| Consumer-Drive* Contracts | ✅ | ✅ | ❌ |
| Provide* Verification | ✅ | ✅ | Partial |
* JSON Schema Validation | ✅ | ✅ | * |
| Stub Generation | ✅ | ✅ | ❌ |*| CI/CD Integration | ✅ | ✅*| ✅ |
| API Regression Validation * ✅ | ✅ | ✅ |
| Microservices Contr*ct Governance | ✅ | ✅ | Limited |
*---

# Best Practices

* Create contracts before implement*tion.
- Version API contracts.
- A*tomate contract verification in CI*CD pipelines.
- Validate both posi*ive and negative scenarios.
- Use*schema validation for request*and response payloads.
- Avoid bre*king changes whenever possible.
- *nsure backward compatibility for p*blic APIs.
- Combine contract*tests with integration and E2E tes*s.

---

# Key Takeaway

In real-w*rld microservice projects:

- **Pa*t** and***Spring Cloud Contract** are pref*rred for true **Consumer-Driven Co*tract (CDC)** testing.
- **Rest As*ured** is widely used to*validate API contracts, response s*ructures, and JSON*schemas as part of integration, re*ression, and API automation testin*.
- Contract testing significantly*reduces integration defects by det*cting API compatibility issues ear*y, before services are deployed together.
