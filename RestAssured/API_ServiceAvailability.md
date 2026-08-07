## For a 3rd Party Service availability check 
---

eg. AML (Anti-Money Laundering) Service Availability testingobjective is typically to validate:

 - AML service endpoint is reachable
 - Service returns HTTP 200
 - Response time is within SLA
 - Health status is UP
 - Required AML components are healthy
 - Service is not degraded
 - Failover endpoint is available (optional)

# Example 1: Simple AML Service availability validation

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.junit.jupiter.api.Test;

import static org.hamcrest.Matchers.*;

public class AMLAvailabilityTest {

    @Test
    public void validateAMLServiceAvailability() {

        Response response =
                RestAssured
                        .given()
                        .baseUri("https://aml.company.com")
                        .when()
                        .get("/actuator/health");

        response.then()
                .statusCode(200)
                .body("status", equalTo("UP"));

        System.out.println("AML Service Status : "
                + response.jsonPath().getString("status"));
    }
}
```


# Example 2: AML Availability Validation 

**Checks**:
  - Availability
  - SLA
  - No degradation
  - Version available



```java
@Test
public void validateAMLServiceHealth() {

    RestAssured
            .given()
            .baseUri("https://aml.company.com")
            .when()
            .get("/health")
            .then()
            .statusCode(200)
            .time(lessThan(2000L))
            .body("status", equalTo("UP"))
            .body("serviceName", equalTo("AML Screening Service"))
            .body("version", notNullValue())
            .body("degraded", equalTo(false));
}

```


# Example 3: Validate AML Availability Across Multiple Regions


```java
import java.util.Map;

@Test
public void validateAMLRegionalAvailability() {

    Map<String, String> regions = Map.of(
            "US", "https://us-aml.company.com",
            "EU", "https://eu-aml.company.com",
            "APAC", "https://apac-aml.company.com"
    );

    regions.forEach((region, url) -> {

        RestAssured
                .given()
                .baseUri(url)
                .when()
                .get("/health")
                .then()
                .statusCode(200)
                .body("status", equalTo("UP"));

        System.out.println(region + " AML Service Available");
    });
}

```


# Real-World AML Service Availability Checks

In an enterprise platform, a health-check suite would typically validate
✓ AML Service Endpoint Reachable
✓ HTTP 200 Status
✓ Health Status = UP
✓ Database Availability
✓ Watchlist Service Availability
✓ Sanctions Screening Engine Availability
✓ OFAC List Service Availability
✓ PEP Service Availability
✓ Kafka Connectivity
✓ Redis Cache Availability
✓ Response Time SLA
✓ Regional Availability
✓ Failover Endpoint Availability
✓ Error Rate Threshold
✓ SSL Certificate Validity
✓ Authentication Token Validation


