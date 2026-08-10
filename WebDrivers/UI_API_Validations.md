## Validate all API calls triggered after clicking a Button in UI
---

To validate all API calls triggered after clickingbtnSearch, you should capture every request/response, store them in a collection, and validate each API's:

 - URL
 - HTTP Method
 - Status Code
 - Content-Type Header
 - Security Headers
 - Response Time
 - Response Body (optional)


# Selenium 4 CDP Example 

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.devtools.DevTools;
import org.openqa.selenium.devtools.v139.network.Network;
import org.openqa.selenium.devtools.v139.network.model.RequestId;
import org.openqa.selenium.devtools.v139.network.model.Response;

import java.time.Duration;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class AllApiValidationTest {

    private static final Map<RequestId, ApiTransaction> apiTransactions =
            new ConcurrentHashMap<>();

    public static void main(String[] args) throws Exception {

        ChromeDriver driver = new ChromeDriver();

        DevTools devTools = driver.getDevTools();
        devTools.createSession();

        devTools.send(Network.enable(
                Optional.empty(),
                Optional.empty(),
                Optional.empty()));

        // ===========================================
        // Capture ALL Requests
        // ===========================================

        devTools.addListener(Network.requestWillBeSent(), request -> {

            ApiTransaction api = new ApiTransaction();

            api.setRequestId(request.getRequestId());
            api.setUrl(request.getRequest().getUrl());
            api.setMethod(request.getRequest().getMethod());
            api.setRequestStartTime(System.currentTimeMillis());

            apiTransactions.put(request.getRequestId(), api);

            System.out.println(
                    "[REQUEST] "
                            + request.getRequest().getMethod()
                            + " "
                            + request.getRequest().getUrl());
        });

        // ===========================================
        // Capture ALL Responses
        // ===========================================

        devTools.addListener(Network.responseReceived(), response -> {

            ApiTransaction api =
                    apiTransactions.get(response.getRequestId());

            if (api != null) {

                api.setResponse(response.getResponse());

                api.setStatusCode(
                        response.getResponse()
                                .getStatus()
                                .intValue());

                api.setResponseEndTime(
                        System.currentTimeMillis());

                long responseTime =
                        api.getResponseEndTime()
                                - api.getRequestStartTime();

                api.setResponseTime(responseTime);

                System.out.println(
                        "[RESPONSE] "
                                + response.getResponse().getUrl()
                                + " -> "
                                + api.getStatusCode());
            }
        });

        // ===========================================
        // Capture ALL Responses payload
        // ===========================================

devTools.addListener(Network.responseReceived(), response -> {

    ApiTransaction api =
            apiTransactions.get(response.getRequestId());

    if (api != null) {

        api.setResponse(response.getResponse());

        api.setStatusCode(
                response.getResponse()
                        .getStatus()
                        .intValue());

        api.setResponseEndTime(
                System.currentTimeMillis());

        long responseTime =
                api.getResponseEndTime()
                   - api.getRequestStartTime();

        api.setResponseTime(responseTime);

        try {

            String body =
                    devTools.send(
                            Network.getResponseBody(
                                    response.getRequestId()))
                            .getBody();

            api.setResponseBody(body);

            System.out.println("\n===== RESPONSE BODY =====");
            System.out.println(body);

        } catch (Exception ex) {

            System.out.println(
                    "Unable to capture response body for : "
                            + api.getUrl());
        }

        System.out.println(
                "[RESPONSE] "
                        + api.getUrl()
                        + " -> "
                        + api.getStatusCode());
    }
});



        driver.get("https://example.com/customer-search");

        // ===========================================
        // Click Search Button
        // ===========================================

        driver.findElement(By.id("btnSearch")).click();

        Thread.sleep(5000);

        // ===========================================
        // Validate ALL API Calls
        // ===========================================

        System.out.println(
                "\n========== API VALIDATION ==========");

        for (ApiTransaction api : apiTransactions.values()) {

            if (api.getResponse() == null) {
                continue;
            }

            System.out.println(
                    "\nURL : " + api.getUrl());

            // -----------------------------------
            // Status Code Validation
            // -----------------------------------

            if (api.getStatusCode() >= 400) {

                throw new AssertionError(
                        "Failed API : "
                                + api.getUrl()
                                + " Status : "
                                + api.getStatusCode());
            }

            // -----------------------------------
            // Content Type Validation
            // -----------------------------------

            Object contentType =
                    api.getResponse()
                            .getHeaders()
                            .get("content-type");

            if (contentType != null) {

                if (!(contentType.toString().contains("json")
                        || contentType.toString().contains("xml")
                        || contentType.toString().contains("text"))) {

                    System.out.println(
                            "Unexpected Content Type : "
                                    + contentType);
                }
            }

            // -----------------------------------
            // Security Header Validation
            // -----------------------------------

            Map<String, Object> headers =
                    api.getResponse().getHeaders();

            validateHeader(
                    headers,
                    "Strict-Transport-Security");

            validateHeader(
                    headers,
                    "X-Content-Type-Options");

            validateHeader(
                    headers,
                    "Content-Security-Policy");

            // -----------------------------------
            // SLA Validation
            // -----------------------------------

            if (api.getResponseTime() > 3000) {

                throw new AssertionError(
                        "Response SLA Breached : "
                                + api.getUrl()
                                + " ("
                                + api.getResponseTime()
                                + " ms)");
            }

            System.out.println(
                    "Status       : "
                            + api.getStatusCode());

            System.out.println(
                    "Response Time: "
                            + api.getResponseTime()
                            + " ms");
//        }

    // ==========================================
    // Response Payload Validation
    // ==========================================

    String body = api.getResponseBody();

    if (body != null &&
            !body.trim().isEmpty()) {

        String lowerBody =
                body.toLowerCase();

        if (lowerBody.contains("\"error\"")
                || lowerBody.contains("\"exception\"")
                || lowerBody.contains("\"failed\"")
                || lowerBody.contains("\"failure\"")) {

            throw new AssertionError(
                    "API returned error payload : "
                            + api.getUrl());
        }
    }

    System.out.println(
            "Status Code  : "
                    + api.getStatusCode());

    System.out.println(
            "Response Time: "
                    + api.getResponseTime()
                    + " ms");

    System.out.println(
            "Payload Validation : PASSED");

        }

        // ===========================================
        // UI Validation
        // ===========================================

        WebElement customerCount =
                driver.findElement(
                        By.id("customerCount"));

        if (customerCount.getText().trim().isEmpty()) {

            throw new AssertionError(
                    "Customer count not visible");
        }

        System.out.println(
                "\nUI Validation Passed");

        driver.quit();
    }

    private static void validateHeader(
            Map<String, Object> headers,
            String headerName) {

        if (headers.keySet()
                .stream()
                .noneMatch(
                        key -> key.equalsIgnoreCase(headerName))) {

            System.out.println(
                    "Missing Header : "
                            + headerName);
        }
    }
}

POJO Class
import lombok.Data;
import org.openqa.selenium.devtools.v139.network.model.RequestId;
import org.openqa.selenium.devtools.v139.network.model.Response;

@Data
public class ApiTransaction {

    private RequestId requestId;

    private String url;

    private String method;

    private int statusCode;

    private Response response;

    private long requestStartTime;

    private long responseEndTime;

    private long responseTime;

    private String responseBody;

}

Enterprise Validation Checklist After btnSearch
```text
Complete Validation Flow
Click btnSearch
      ↓
Capture All Requests
      ↓
Capture All Responses
      ↓
Capture Response Body
      ↓
Validate Status Code
      ↓
Validate Security Headers
      ↓
Validate Content Type
      ↓
Validate Response Time SLA
      ↓
Validate Response Payload
           ├─ error
           ├─ exception
           ├─ failed
           └─ business failure
      ↓
Validate UI
      ↓
Pass / Fail Test
```


This pattern is commonly used in enterprise Selenium frameworks to perform UI + API + Network-level validation for every backend call triggered by a business transaction, significantly reducing false positives and improving end-to-end test coverage.


