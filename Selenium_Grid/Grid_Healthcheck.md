# Selenium Grid Status API Validation Using RestAssured

If you're already using **RestAssured** in your Selenium automation framework, it's a clean way to validate Grid availability before creating a `RemoteWebDriver`. 【1-0d39da】

---

## Maven Dependency

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>5.5.1</version>
</dependency>
```

---

## 1. Basic Selenium Grid Health Check

```java
import io.restassured.response.Response;

import static io.restassured.RestAssured.*;

public class GridHealthCheck {
*    public static void main(String*] args) {

        Response respon*e =
                given()
      *         .when()
                    .get("http://localhost:4444/status")
                .then()
                    .statusCode(200)
                    .extract()
                    .response();

        System.out.println(response.asPrettyString());
    }
}
```

---

## 2. Verify Grid is Ready

### Sample Grid Response

```json
{
  "value": {
    "ready": true,
    "message": "Selenium Grid ready."
  }
}
```

### Code Example

```java
import io.restassured.response.Response;

import static io.restassured.RestAssured.*;

public class GridHealthCheck {

    public static boolean isGridReady() {

        Response response =
                given()
                .when()
                    .get("http://localhost:4444/status");

        return response.jsonPath()
                       .getBoolean("value.ready");
    }

    public static void main(String[] args) {

        if(isGridReady()) {

            System.out.println("Grid is Ready");

        } else {

            System.out.println("Grid is NOT Ready");
        }
    }
}
```

---

## 3. Utility Method for Framework

```java
import io.restassured.response.Response;

import static io.restassured.RestAssured.*;

public class SeleniumGridUtil {

    public static boolean verifyGridAvailability(
            String gridUrl) {

        try {

            Response response =
                    given()
                    .when()
                        .get(gridUrl + "/status");

            return response.statusCode() == 200
                    &&
                    response.jsonPath()
                            .getBoolean("value.ready");

        } catch(Exception ex) {

            return false;
        }
    }
}
```

### Usage

```java
boolean status =
        SeleniumGridUtil.verifyGridAvailability(
                "http://localhost:4444");

System.out.println(status);
```

---

## 4. Fail Test When Grid Is Not Available

Useful in `@BeforeSuite`.

```java
import org.testng.annotations.BeforeSuite;

public class BaseTest {

    @BeforeSuite
    public void verifyGrid() {

        boolean gridReady =
                SeleniumGridUtil
                    .verifyGridAvailability(
                        "http://localhost:4444");

        if(!gridReady) {

            throw new RuntimeException(
                "Selenium Grid is unavailable");
        }
    }
}
```

---

## 5. Verify Grid Before Creating Driver

```java
public static WebDriver createRemoteDriver()
        throws Exception {

    boolean ready =
            SeleniumGridUtil
                .verifyGridAvailability(
                    "http://localhost:4444");

    if(!ready) {

        throw new RuntimeException(
                "Grid is Down");
    }

    return new RemoteWebDriver(
            new URL("http://localhost:4444"),
            new ChromeOptions());
}
```

---

## 6. Wait Until Grid Becomes Available

Useful during Docker/Kubernetes startup.

```java
public static void waitForGrid(
        String gridUrl,
        int timeoutInSeconds)
        throws InterruptedException {

    for(int i = 0;
        i < timeoutInSeconds;
        i++) {

        boolean ready =
                SeleniumGridUtil
                        .verifyGridAvailability(
                                gridUrl);

        if(ready) {

            System.out.println(
                    "Grid Ready");

            return;
        }

        Thread.sleep(1000);
    }

    throw new RuntimeException(
        "Grid did not start");
}
```

### Usage

```java
waitForGrid(
        "http://localhost:4444",
        60);
```

---

## 7. Get Registered Nodes Count

```java
import io.restassured.response.Response;

import static io.restassured.RestAssured.*;

public class GridNodes {

    public static void main(String[] args) {

        Response response =
                given()
                .when()
                    .get("http://localhost:4444/status");

        int nodes =
                response.jsonPath()
                        .getList("value.nodes")
                        .size();

        System.out.println(
                "Registered Nodes = "
                + nodes);
    }
}
```

---

## 8. Retrieve Node Details

```java
Response response =
        given()
        .when()
           .get("http://localhost:4444/status");

List<Map<String,Object>> nodes =
        response.jsonPath()
                .getList("value.nodes");

for(Map<String,Object> node : nodes) {

    System.out.println(
            node.get("id"));

    System.out.println(
            node.get("uri"));
}
```

---

## 9. Validate Node Availability with Hamcrest

```java
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

given()
.when()
    .get("http://localhost:4444/status")
.then()
    .statusCode(200)
    .body("value.ready",
          equalTo(true));
```

---

## 10. Enterprise GridHealthChecker Utility

```java
public class GridHealthChecker {

    public static boolean isGridReady(
            String gridUrl) {

        try {

            return given()
                    .when()
                    .get(gridUrl + "/status")
                    .then()
                    .extract()
                    .jsonPath()
                    .getBoolean("value.ready");

        } catch (Exception e) {

            return false;
        }
    }

    public static int getNodeCount(
            String gridUrl) {

        try {

            return given()
                    .when()
                    .get(gridUrl + "/status")
                    .then()
                    .extract()
                    .jsonPath()
                    .getList("value.nodes")
                    .size();

        } catch(Exception e) {

            return 0;
        }
    }
}
```

### Usage

```java
if(GridHealthChecker.isGridReady(
        "http://localhost:4444")) {

    System.out.println(
        "Grid Available");

    System.out.println(
        "Nodes : " +
        GridHealthChecker.getNodeCount(
            "http://localhost:4444"));
}
```

---

## Typical Framework Flow

```text
@BeforeSuite
    |
    +--> Check Grid Ready
    |
    +--> Check Node Count
    |
    +--> Create RemoteWebDriver
    |
    +--> Execute Tests
```

---

## Summary

This approach is commonly used in enterprise Selenium frameworks to perform:

- Grid Health Checks
- Grid Readiness Validation
- Node Availability Checks
- Fail-Fast Startup Validation
- Docker/Kubernetes Readiness Verification
- Pre-Suite Infrastructure Validation

before creating RemoteWebDriver sessions using RestAssured. 【1-0d39da】
