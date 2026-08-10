Selenium 4 + CDP Network Response Validation Example


**Scenario**

```text
Click Search Button
      ↓
CDP captures request
      ↓
Validate URL + Method
      ↓
CDP captures response
      ↓
Validate Status Code
      ↓
Validate Headers
      ↓
Validate Response Time
      ↓
Validate Response Payload
      ↓
Validate UI Values
      ↓
Pass / Fail Test
```

This example demonstrates:

 - Capture API Request
 - Capture API Response
 - Validate Status Code
 - Validate Headers
 - Validate Response Time
 - Validate UI Content


```java
package assurant;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.devtools.DevTools;
import org.openqa.selenium.devtools.v139.network.Network;
import org.openqa.selenium.devtools.v139.network.model.Response;

import java.time.Duration;
import java.util.Optional;
import java.util.concurrent.atomic.AtomicLong;
import java.util.concurrent.atomic.AtomicReference;

public class CDPNetworkValidationTest {

	public static void main(String[] args) {

		ChromeDriver driver = new ChromeDriver();

		DevTools devTools = driver.getDevTools();
		devTools.createSession();

		devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));

		AtomicReference<String> apiUrl = new AtomicReference<>();
		AtomicReference<Integer> statusCode = new AtomicReference<>();
		AtomicReference<Response> apiResponse = new AtomicReference<>();
		AtomicLong requestStartTime = new AtomicLong();
		AtomicLong responseEndTime = new AtomicLong();

		// Capture Request
		devTools.addListener(Network.requestWillBeSent(), request -> {

			if (request.getRequest().getUrl().contains("/api/customers")) {

				apiUrl.set(request.getRequest().getUrl());

				requestStartTime.set(System.currentTimeMillis());

				System.out.println("API Request Sent");
				System.out.println("URL : " + request.getRequest().getUrl());
				System.out.println("Method : " + request.getRequest().getMethod());
			}
		});

		// Capture Response
		devTools.addListener(Network.responseReceived(), response -> {

			if (response.getResponse().getUrl().contains("/api/customers")) {

				apiResponse.set(response.getResponse());
				statusCode.set(response.getResponse().getStatus().intValue());

				responseEndTime.set(System.currentTimeMillis());

				System.out.println("API Response Received");
				System.out.println("Status : " + statusCode.get());
			}
		});

		driver.get("https://example.com/customer-search");

		// User clicks button
		driver.findElement(By.id("btnSearch")).click();

		driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(5));

		// ==========================================================
		// Status Code Validation
		// ==========================================================

		if (statusCode.get() != 200) {
			throw new AssertionError("Expected Status Code 200 but found " + statusCode.get());
		}

		// ==========================================================
		// Header Validation
		// ==========================================================

		String contentType = apiResponse.get().getHeaders().get("content-type").toString();

		if (!contentType.contains("application/json")) {
			throw new AssertionError("Content-Type is incorrect : " + contentType);
		}

		// Security Header Example

		boolean hstsPresent = apiResponse.get().getHeaders().containsKey("Strict-Transport-Security");

		if (!hstsPresent) {
			throw new AssertionError("Missing HSTS Security Header");
		}

		// ==========================================================
		// Response Time Validation
		// ==========================================================

		long responseTime = responseEndTime.get() - requestStartTime.get();

		System.out.println("Response Time = " + responseTime + " ms");

		if (responseTime > 3000) {
			throw new AssertionError("Response Time exceeded SLA");
		}

		// ==========================================================
		// UI Validation
		// ==========================================================

		WebElement customerCount = driver.findElement(By.id("customerCount"));

		String countText = customerCount.getText();

		if (countText.isEmpty()) {
			throw new AssertionError("Customer Count not displayed");
		}

		System.out.println("UI Validation Passed");
		System.out.println("Customer Count : " + countText);

		driver.quit();
	}
}

```

> This approach is commonly used in enterprise Selenium frameworks to achieve **UI + API + Network-level validation within a single test execution**, reducing false positives and improving end-to-end test coverage.
