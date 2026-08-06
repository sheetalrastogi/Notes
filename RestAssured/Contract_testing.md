## Consumer-Driven Contract (CDC) Testing Example with RestAssured


In Consumer-Driven Contract Testing, the consumer defines its expectations from the provider API. The provider then validates that it can satisfy those expectations without breaking existing consumers.


# Popular CDC Tools

- Pact
- Spring Cloud Contract
- Hoverfly
- CDC + RestAssured custom validation


# Scenario

**Consumer Expectation**

Consumer Service (Policy Service) expects:

For Request 

GET /customers/1001


Expected Response
```java
{
  "customerId": 1001,
  "name": "John Smith",
  "email": "john.smith@test.com",
  "status": "ACTIVE"
}
```


**The consumer creates a contract defining**:

- customerId must exist
- name must be String
- email must be valid
- status must be ACTIVE or INACTIVE


# RestAssured contract validation for above:

```java
        given()
        .baseUri("https://api.company.com")
        .pathParam("customerId", 1001)

        .when()
            .get("/customers/{customerId}")

        .then()
            .statusCode(200)

            // Contract Assertions
            .body("customerId", notNullValue())
            .body("customerId", instanceOf(Integer.class))
            .body("name", notNullValue())
            .body("email", matchesPattern("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$"))
            .body("status", anyOf(equalTo("ACTIVE"), equalTo("INACTIVE")));

```


# Example 2: Consumer Validating Mandatory Fields

Consumer requires following mandatory fields:

```java
{
  "policyNumber":"POL123",
  "premium":5000,
  "currency":"INR"
}
```


**RestAssured Contract Test**:

```java
given()
.when()
    .get("/policy/123")
.then()
    .statusCode(200)
    .body("$", hasKey("policyNumber"))
    .body("$", hasKey("premium"))
    .body("$", hasKey("currency"));
```

# Example 3: Contract Validation Using JSON Schema

```java
String schema = "JSON Schema definition";

given()
.when()
    .get("/customers/1001")
.then()
    .statusCode(200)
    .body(matchesJsonSchema("schema"));

    // or 

    .body(matchesJsonSchemaInClasspath("customer-schema.json"));  // if schema file exists in Classpath

```


# Typical CDC Validations in Microservices
- Endpoint exists
- HTTP Method supported
- Response has valid status code
- Mandatory fields present
- Field data types
- Response schema validation
- Headers validation
- Pagination contract
- Error response contract
- Backward compatibility validation
- Version compatibility validation








