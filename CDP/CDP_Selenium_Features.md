# Selenium 4 – Execute Commands and Capture Output

## Overview

In Selenium 4, there are multiple ways to execute commands and capture their output, depending on what you mean by "command":

- Execute JavaScript in browser and capture result
- Execute Chrome DevTools Protocol (CDP) commands and capture output
- Execute OS/Shell commands from Java and capture output
- Capture browser console logs
- Capture network response data

---

## Execute JavaScript and Capture Output

### Example: Get Browser User Agent

```java
String agent = (String) js.executeScript(
        "return navigator.userAgent");

System.out.println(agent);
```

### Example: Get Current URL

```java
String url = (String) js.executeScript(
        "return window.location.href");

System.out.println(url);
```

---

# Execute CDP Command and Read Returned Values

## Example: Get Performance Metrics

```java
Map<String, Object> metrics =
        ((ChromeDriver) driver)
                .executeCdpCommand(
                        "Performance.getMetrics",
                        Map.of());

System.out.println(metrics);
```

### Extract Metrics

```java
List<Map<String, Object>> metricList =
        (List<Map<String, Object>>)
                metrics.get("metrics");

for (Map<String, Object> item : metricList)
{
    System.out.println(item);
}
```

---

## Example: Get Browser Version

```java
Map<String, Object> result =
        ((ChromeDriver) driver)
                .executeCdpCommand(
                        "Browser.getVersion",
                        new HashMap<>());

System.out.println(result);
```

### Sample Output

```text
{
  product=Chrome/138.0.7204.115,
  revision=123456,
  userAgent=Mozilla/5.0 ...
}
```

## Capture Browser Console Output
-------------------------------------
```Java
	DevTools devTools = ((ChromeDriver) driver).getDevTools();

	devTools.createSession();

	devTools.send(Log.enable());

	devTools.addListener(Log.entryAdded(),entry-> {
	            System.out.println(entry.getText());
	 });
	
	// Sample output:
	Uncaught TypeError
	Resource failed to load
	API response error
```

## Capture Network Response Content
---------------------------------------
```java
	DevTools devTools = ((ChromeDriver) driver).getDevTools();

	devTools.createSession();

	devTools.send(Network.enable(Optional.empty(),Optional.empty(),Optional.empty()));

	devTools.addListener(Network.responseReceived(),response->
	{
	            System.out.println(response.getResponse().getUrl());
	            System.out.println(response.getResponse().getStatus());
	});
```	
	//Sample output:
	https://api.company.com/customer
	200

	https://api.company.com/policy
	500


## Reusable Utility for Command Output
## Execute Shell Commands Using ProcessBuilder
-----------------------------------------------
```Java
	public static String executeCommand(String command) throws Exception {
		ProcessBuilder builder = new ProcessBuilder("cmd", "/c", command);

		Process process = builder.start();

		BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));

		StringBuilder output = new StringBuilder();

		String line;

		while ((line = reader.readLine()) != null) {
			output.append(line).append("\n");
		}

		return output.toString();
	}
```
	// Usage:
	String result = executeCommand("ipconfig");
	System.out.println(result);

	// Useful for: ipconfig, ping, dir, curl, tasklist


## Network Monitoring & Interception

## Example: Capture All Network Requests and Responses
-------------------------------------------------------


```java
	public static void main(String[] args) {

		WebDriver driver = new ChromeDriver();

		DevTools devTools = ((ChromeDriver) driver).getDevTools();

		devTools.createSession();

		devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));

		// Capture Requests
		devTools.addListener(Network.requestWillBeSent(), request -> {

			System.out.println("Request URL: " + request.getRequest().getUrl());

			System.out.println("Method: " + request.getRequest().getMethod());

			System.out.println("------------------------------------------------");
		});

		// Capture Responses
		devTools.addListener(Network.responseReceived(), response -> {

			System.out.println("Response URL: " + response.getResponse().getUrl());

			System.out.println("Status Code: " + response.getResponse().getStatus());

			System.out.println("------------------------------------------------");
		});

		driver.get("https://myapp.com");

		// user actions here
	}
```

Output:
--------
Request URL:
https://myapp.com/api/policies/12345

Method:
GET

------------------------------------------

Response URL:
https://myapp.com/api/policies/12345

Status Code:
200



## Example: Monitor API Triggered by Button Click
----------------------------------------------------

```java
 	public static void main(String[] args) {
		// Example: Monitor API Triggered by Button Click
		devTools.addListener(Network.responseReceived(), response -> {

			String url = response.getResponse().getUrl();

			if (url.contains("/api/policies")) {
				System.out.println("Policy API Called");

				System.out.println("Status: " + response.getResponse().getStatus());
			}
		});

		driver.findElement(By.id("policyNumber")).sendKeys("12345");

		driver.findElement(By.id("searchBtn")).click();

	}
```
	
	// Output
	Policy API Called
	Status: 200

## Example:  Capture Response Headers:
----------------------------------------


```java
devTools.addListener(
        Network.requestWillBeSent(),
        req -> {

            System.out.println(req.getRequest().getHeaders());
        });
```
// Output:
{
 Authorization=Bearer xyz123,
 Content-Type=application/json,
 User-Agent=Chrome
}

-----------

```java
devTools.addListener(
        Network.responseReceived(),
        resp -> {

            System.out.println(
                resp.getResponse()
                    .getHeaders());
        });
```

Sample Output
{
 Content-Type=application/json,
 Cache-Control=no-cache,
 Server=nginx
}



## Example: Capture Only Failed API Calls
---------------------------------------------
```java

	public static void main(String[] args) {
		devTools.addListener(Network.responseReceived(), response -> {

			int status = response.getResponse().getStatus().intValue();

			if (status >= 400) {
				System.out.println("FAILED API");

				System.out.println(response.getResponse().getUrl());

				System.out.println(status);
			}
		});

	}
```

	// Output 
	FAILED API
	https:// myapp.com/api/policy
	500


## Example: Collect Network Traffic in a Framework
---------------------------------------------------------

```java

public class NetworkLogger {

	public static void startNetworkCapture(ChromeDriver driver) {

		DevTools devTools = driver.getDevTools();

		devTools.createSession();

		devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));

		devTools.addListener(Network.responseReceived(), response -> {

			System.out.println(response.getResponse().getUrl() + " : " + response.getResponse().getStatus());
		});
	}
}

```
Usage

```java
ChromeDriver driver = new ChromeDriver();
NetworkLogger.startNetworkCapture(driver);
driver.get("https://myapp.com");
```


## Request Blocking with CDP
========================

## Request Blocking with CDP
--------------------------------

## Example: Simulate a scenario where stylesheets fail to load.
```java
		ChromeDriver driver = new ChromeDriver();

		DevTools devTools = driver.getDevTools();
		devTools.createSession();

		devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));

		devTools.send(Network.setBlockedURLs(List.of("*.css")));

		driver.get("https://example.com");
```

Use Cases:
- Verify page functionality without styles
- Test graceful degradation
- Identify CSS dependencies

## Example:  Simulate Block Images to Prevent image loading.
```java
		devTools.send(Network.setBlockedURLs(List.of("*.png", "*.jpg", "*.jpeg", "*.gif", "*.webp")));
```
Use Cases:
- Faster test execution
- Reduced bandwidth usage
- Performance testing


## Example: Block JavaScript Files, Simulate broken frontend dependencies.

```java
		devTools.send(Network.setBlockedURLs(List.of("*.js")));

		// Validation
		driver.get("https://example.com");
		System.out.println(driver.getTitle());
```
Use Cases:
- Application resilience validation
- Error handling verification
- Progressive enhancement testing


## Example: Block Specific API Endpoint

Suppose during execution application calls:   https://api.company.com/policy/search  and as tester, you need to Block only this API.
```java
		devTools.send(Network.setBlockedURLs(List.of("*policy/search*")));
```

Use Cases:
- Simulate API downtime
- Verify user-friendly error messages
- Validate retry mechanisms


## Example: Block entire domain:
```java
		devTools.send(Network.setBlockedURLs(List.of("*api.company.com*")));
```

Use Cases:
- Block Analytics Services
```java
		devTools.send(Network.setBlockedURLs(List.of("*google-analytics.com*", "*adservice.google.com*")));
```

Benefits:
- Faster execution
- Cleaner network traffic
- Focus on business APIs


## Example: Simulate Third-Party Service Failure
```java
		devTools.send(Network.setBlockedURLs(List.of("*paymentgateway.com*")));
```

Example usage:
- Payment Gateway
- Address Validation Service
- Credit Score Service
- SMS Service

## Example: Validate Request Blocking with Listener

```java
		devTools.addListener(Network.loadingFailed(), failed -> {

			System.out.println("Blocked URL: " + failed.getRequestId());

			System.out.println("Reason: " + failed.getErrorText());
		});
```
Sample Output
- Blocked URL: 12345
- Reason: net::ERR_BLOCKED_BY_CLIENT


## Network throttling features with CDP
--------------------------------------------

## Network throttling:
--------------------------

The highest-value network throttling scenarios are:

- Loading Spinner Validation
- API Timeout Handling
- Retry Mechanism Testing
- Duplicate Transaction Prevention
- Slow Mobile Network Simulation
- Performance SLA Validation
- Large File Download Testing
- Distributed Microservice Latency Testing
- Progress Indicator Validation
- Customer Experience Testing Under Poor Connectivity



## Example 1: Simulate Slow 3G Network


```java
import java.util.Optional;

import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.devtools.DevTools;
import org.openqa.selenium.devtools.v138.network.Network;

ChromeDriver driver = new ChromeDriver();

DevTools devTools = driver.getDevTools();
devTools.createSession();

devTools.send(Network.enable(
        Optional.empty(),
        Optional.empty(),
        Optional.empty()));

// Simulate Slow 3G
devTools.send(
        Network.emulateNetworkConditions(
                false,      // offline
                100,        // latency (ms)
                50000,      // download speed
                20000,      // upload speed
                Optional.empty()));

driver.get("https://example.com");
```

## Use Case 1: Verify Loading Spinner

**Scenario**
```text
User clicks Search Policy
       ↓
API takes longer than normal
       ↓
Loading Spinner displayed
       ↓
Response received
       ↓
Policy details displayed
```

```java
// Enable slow network
devTools.send(
        Network.emulateNetworkConditions(
                false,
                3000,
                50000,
                20000,
                Optional.empty()));

// Perform search
driver.findElement(By.id("policyNumber"))
      .sendKeys("P12345");

driver.findElement(By.id("searchBtn"))
      .click();

// Verify spinner visible
Assert.assertTrue(
    driver.findElement(By.id("loadingSpinner"))
          .isDisplayed());
```
- Spinner appears
- UI does not freeze
- User gets feedback


## Use Case 2: Verify API Timeout Handling

```text
Scenario
Search Customer
      ↓
Backend API is very slow
      ↓
Request exceeds timeout
      ↓
Error message displayed

Test Flow
```
```java
devTools.send(
        Network.emulateNetworkConditions(
                false,
                10000,
                1000,
                1000,
                Optional.empty()));

driver.findElement(By.id("searchBtn"))
      .click();
```

Verify
```java

String error = driver.findElement(By.id("errorMsg")).getText();

Assert.assertEquals(error, "Request timed out. Please try again.");

```


## Use Case 3: Validate Retry Logic

```text
Scenario
Submit Claim
      ↓
Slow Network
      ↓
First Call Fails
      ↓
Automatic Retry
      ↓
Success

Example Validation
```

```java
devTools.send(
        Network.emulateNetworkConditions(
                false,
                5000,
                5000,
                5000,
                Optional.empty()));

driver.findElement(By.id("submitBtn"))
      .click();

```

Verify:

- Retry attempted
- User notification shown
- Request eventually succeeds



## Use Case 4: Performance Testing - Measure page load under slow networks.

```java
long startTime =
        System.currentTimeMillis();

driver.get("https://example.com");

long endTime =
        System.currentTimeMillis();

System.out.println(
        "Page Load Time: "
                + (endTime - startTime));
```
Compare Results
- Normal Network  : 2 Seconds
- 3G Network      : 8 Seconds
- 2G Network      : 20 Seconds

Useful for SLA validation.



## Use Case 5: Mobile User Simulation

Simulate mobile internet speeds.

devTools.send(
        Network.emulateNetworkConditions(
                false,
                400,
                1600000,
                750000,
                Optional.empty()));

Validate
- Responsive design
- Lazy loading
- Image loading
- Mobile user experience


## Use Case 6: Verify Report Download Progress
```text
Scenario
User downloads PDF report
      ↓
Slow Network
      ↓
Progress Indicator Appears
      ↓
File Download Completes

Validation
```
```java
Assert.assertTrue(
        driver.findElement(By.id("downloadProgress"))
              .isDisplayed());
```


## Use Case 7: Validate Search Auto-Suggestions

```text
Scenario
User types customer name
      ↓
Autocomplete API called
      ↓
Network slow
      ↓
Suggestions still appear correctly

Verify
```
```java
driver.findElement(By.id("customer"))
      .sendKeys("John");

List<WebElement> suggestions =
        driver.findElements(
                By.className("suggestion"));
```

```text
Enterprise QA Use Cases
Banking
Fund Transfer
      ↓
Slow Network
      ↓
Verify transaction not duplicated

Insurance
Claim Submission
      ↓
High Latency
      ↓
Verify claim submitted once

Healthcare
Patient Search
      ↓
Slow API response
      ↓
Verify timeout handling

E-Commerce
Add To Cart
      ↓
Network delay
      ↓
Verify no duplicate orders

```

## Reusable Utility
```java
public class NetworkThrottleUtil {

    public static void simulateSlow3G(
            ChromeDriver driver) {

        DevTools devTools =
                driver.getDevTools();

        devTools.createSession();

        devTools.send(Network.enable(
                Optional.empty(),
                Optional.empty(),
                Optional.empty()));

        devTools.send(
                Network.emulateNetworkConditions(
                        false,
                        200,
                        50000,
                        20000,
                        Optional.empty()));
    }
}

// Usage

ChromeDriver driver = new ChromeDriver();

NetworkThrottleUtil.simulateSlow3G(driver);

driver.get("https://example.com");

```


## CDP Usage case for "Geo location" testing
-------------------------------------------------

## Geolocation Override Using Chrome DevTools Protocol
-----------------------------------------------------------

Geolocation Override allows Selenium tests to simulate users from different geographic locations without physically being in those locations.

This is useful for testing:

- Location-based services
- Maps applications
- Insurance premium calculations
- Regional content
- Store locator functionality
- Geo-restricted features
- Tax and pricing calculations


## Basic Geolocation Override Example - Simulate User in Mumbai, India

```Java
import java.util.Optional;

import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.devtools.DevTools;
import org.openqa.selenium.devtools.v138.emulation.Emulation;

ChromeDriver driver = new ChromeDriver();

DevTools devTools = driver.getDevTools();

devTools.createSession();

devTools.send(
        Emulation.setGeolocationOverride(
                Optional.of(19.0760),   // Latitude
                Optional.of(72.8777),   // Longitude
                Optional.of(1)));       // Accuracy

driver.get("https://www.google.com/maps");
```


## Example: Store Locator Testing 

```text
User opens application
↓
Application detects location
↓
Nearest stores displayed
```

**Override Location to New York**
```java
devTools.send(
      Emulation.setGeolocationOverride(
              Optional.of(40.7128),
              Optional.of(-74.0060),
              Optional.of(1)));
```
Validation:
```java
List<WebElement> stores = driver.findElements(By.className("store"));

Assert.assertTrue(stores.size() > 0);
```


## Example:  Google Maps Validation

Test route generation from different locations

```java
devTools.send(
      Emulation.setGeolocationOverride(
              Optional.of(28.6139),
              Optional.of(77.2090),
              Optional.of(1)));

driver.get("https://maps.google.com");
```

Validate:
- Current location pinned correctly
- Directions generated correctly
- Nearby locations displayed


## Example: Geo-Fencing Validation

```text
Scenario: Feature available only in a specific country.

USA Users
↓
Feature Enabled

Other Countries
↓
Feature Hidden
```

usage:  Set USA Location

```java
devTools.send(
      Emulation.setGeolocationOverride(
              Optional.of(37.7749),
              Optional.of(-122.4194),
              Optional.of(1)));
```

Verify
```java
Assert.assertTrue(
        driver.findElement(
                By.id("premiumFeature"))
                .isDisplayed());
```


## Example: Regional Language Testing

```text
Germany
      ↓
German Content

France
      ↓
French Content

India
      ↓
English Content
```


```java
devTools.send(
      Emulation.setGeolocationOverride(
              Optional.of(52.5200),
              Optional.of(13.4050),
              Optional.of(1)));

```

Verify:
```java
String language =
        driver.findElement(By.tagName("html"))
              .getAttribute("lang");

Assert.assertEquals(language, "de");
```


Other examples:
- Weather Application Testing
- Food Delivery Application
- Switch Locations During Same Test


## Reusable utility method:

```java
public class GeoLocationUtil {

    public static void setLocation(
            ChromeDriver driver,
            double latitude,
            double longitude) {

        DevTools devTools =
                driver.getDevTools();

        devTools.createSession();

        devTools.send(
                Emulation.setGeolocationOverride(
                        Optional.of(latitude),
                        Optional.of(longitude),
                        Optional.of(1)));
    }
}
```

Usage:
```java
GeoLocationUtil.setLocation(
        driver,
        19.0760,
        72.8777);

driver.get("https://myapp.com");
```



**Common Latitude & Longitude Values**
------------------------------------------
Mumbai        : 19.0760, 72.8777
Delhi         : 28.6139, 77.2090
Bangalore     : 12.9716, 77.5946
London        : 51.5074, -0.1278
Paris         : 48.8566, 2.3522
Tokyo         : 35.6762, 139.6503
New York      : 40.7128, -74.0060
San Francisco : 37.7749, -122.4194


**QA Use Cases**
-------------------

- Insurance
  - Location-Based Premium Rating
  - Territory Risk Calculation
  - Policy Availability Validation

- Banking
  - Country-Specific Features
  - Regional Compliance Rules
  - Branch Locator Validation

- Retail
  - Store Locator
  - Regional Offers
  - Localized Pricing

- Travel
  - Nearby Hotels
  - Taxi Availability
  - Trip Recommendations

- Healthcare
  - Nearest Hospital Search
  - Doctor Availability
  - Emergency Service Routing


## Emulating iPhone and Other Devices Using Chrome DevTools Protocol
-----------------------------------------------------------------------

Chrome DevTools Protocol (CDP) allows Selenium 4 to emulate different mobile devices by overriding:

- Screen Width
- Screen Height
- Device Scale Factor
- Mobile Viewport
- Touch Events
- User Agent


A realistic mobile simulation typically combines:


```text
Device Metrics Override
+ User Agent Override
+ Touch Emulation
+ Geolocation Override
+ Network Throttling
+ Responsive Layout Validation
```


## Example 1: Emulate iPhone 14 Pro

```java
import java.util.Optional;

import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.devtools.DevTools;
import org.openqa.selenium.devtools.v138.emulation.Emulation;

ChromeDriver driver = new ChromeDriver();

DevTools devTools = driver.getDevTools();
devTools.createSession();

// Device Metrics
devTools.send(
    Emulation.setDeviceMetricsOverride(
            393,    // width
            852,    // height
            3,      // device scale factor
            true,   // mobile
            Optional.empty(),
            Optional.empty(),
            Optional.empty(),
            Optional.empty(),
            Optional.empty(),
            Optional.empty(),
            Optional.empty(),
            Optional.empty(),
            Optional.empty()));

// User Agent
driver.executeCdpCommand(
        "Network.setUserAgentOverride",
        java.util.Map.of(
                "userAgent",
                "Mozilla/5.0 (iPhone; CPU iPhone OS 16_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/16.0 Mobile/15E148 Safari/604.1"));

driver.get("https://www.google.com");
```


## Example 2: Emulate iPhone 15 Pro

```java
devTools.send(
    Emulation.setDeviceMetricsOverride(
            393,
            852,
            3,
            true,
            Optional.empty(),
            Optional.empty(),
            Optional.empty(),
            Optional.empty(),
            Optional.empty(),
            Optional.empty(),
            Optional.empty(),
            Optional.empty(),
            Optional.empty()));
```


**Common Device Profiles -- iOS**

Device             Width   Height  DPR
---------------------------------------
iPhone SE          375     667     2
iPhone XR          414     896     2
iPhone 12 Pro      390     844     3
iPhone 13 Pro      390     844     3
iPhone 14 Pro      393     852     3
iPhone 15 Pro      393     852     3
iPhone 15 Pro Max  430     932     3
iPad Mini          768     1024    2
iPad Air           820     1180    2
iPad Pro 11"       834     1194    2
iPad Pro 12.9"     1024    1366    2

**Common Device Profiles -- Android**

Device                 Width   Height   DPR
--------------------------------------------
Pixel 5                393     851      2.75
Pixel 7                412     915      2.63
Pixel 8 Pro            448     998      3
Galaxy S21             384     854      2.81
Galaxy S22 Ultra       412     915      3.5
Galaxy S23             360     780      3
Galaxy S24 Ultra       412     915      3.5
OnePlus 11             412     919      3
Xiaomi 13 Pro          393     873      3


## Reusable Utility Method

```java
public static void emulateDevice(
        DevTools devTools,
        int width,
        int height,
        double scaleFactor) {

    devTools.send(
        Emulation.setDeviceMetricsOverride(
                width,
                height,
                scaleFactor,
                true,
                Optional.empty(),
                Optional.empty(),
                Optional.empty(),
                Optional.empty(),
                Optional.empty(),
                Optional.empty(),
                Optional.empty(),
                Optional.empty(),
                Optional.empty()));
}

```
Usage:
```java
emulateDevice(
        devTools,
        393,
        852,
        3);
```


## Other CDP Features for mobile emulation

## Emulate Touch Screen

```java
driver.executeCdpCommand(
    "Emulation.setTouchEmulationEnabled",
    java.util.Map.of(
        "enabled", true));
```


## Emulate Mobile Network + Device

Combine device emulation with network throttling.

```java

devTools.send(
    org.openqa.selenium.devtools.v138.network.Network
        .emulateNetworkConditions(
            false,
            200,
            50000,
            20000,
            Optional.empty()));
```

## Console logs example usages with Chrome Dev Protocol (CDP)
--------------------------------------------------------------------

## Capture Browser Console Logs Using Chrome DevTools Protocol
----------------------------------------------------------------

Capturing browser console logs is extremely useful for:

- Detecting JavaScript errors
- API failures surfaced in the browser
- Monitoring application warnings
- Troubleshooting UI issues
- Capturing client-side exceptions
- Identifying failed resource loading


## Example 1: Capture All Browser Console Logs

```java
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.devtools.DevTools;
import org.openqa.selenium.devtools.v138.log.Log;

ChromeDriver driver = new ChromeDriver();

DevTools devTools = driver.getDevTools();
devTools.createSession();

// Enable Browser Logs
devTools.send(Log.enable());

// Listen for Console Events
devTools.addListener(
        Log.entryAdded(),
        logEntry -> {

            System.out.println(
                    "Level : "
                    + logEntry.getLevel());

            System.out.println(
                    "Text  : "
                    + logEntry.getText());

            System.out.println(
                    "URL   : "
                    + logEntry.getUrl());

            System.out.println(
                    "------------------------------------------------");
        });

driver.get("https://example.com");
```

Sample output:

```text
Level : INFO
Text  : Application loaded successfully
URL   : https://example.com

------------------------------------------------

Level : WARNING
Text  : Deprecated API detected
URL   : https://example.com

------------------------------------------------

Level : ERROR
Text  : Uncaught TypeError: Cannot read property 'name'
URL   : https://example.com

------------------------------------------------

```


## Example 2: Capture JavaScript Errors Only

Many organizations fail automation execution whenever JavaScript errors occur.

```java
devTools.send(Log.enable());

devTools.addListener(
        Log.entryAdded(),
        log -> {

            if(log.getLevel().toString()
                  .equalsIgnoreCase("error"))
            {
                System.out.println(
                        "JS Error Found");

                System.out.println(
                        log.getText());
            }
        });
```

Sample output:

```text
JS Error Found

Uncaught TypeError:
Cannot read property 'value' of undefined
```



## Example 3: Fail Test When Console Error Appears

```java
AtomicBoolean errorFound =
        new AtomicBoolean(false);

devTools.send(Log.enable());

devTools.addListener(
        Log.entryAdded(),
        log -> {

            if(log.getLevel().toString()
                  .equalsIgnoreCase("error"))
            {
                errorFound.set(true);

                System.out.println(
                        log.getText());
            }
        });

driver.get("https://myapp.com");

// Test Steps

Assert.assertFalse(
        errorFound.get(),
        "Console error found");
```


## Example 4: Capture Console Logs During User Actions

Scenario:
```text
Login
 ↓
Search Policy
 ↓
Submit Claim
 ↓
Capture Any JS Errors
```

```java
devTools.send(Log.enable());

devTools.addListener(
        Log.entryAdded(),
        log -> {

            System.out.println(
                    log.getLevel()
                    + " : "
                    + log.getText());
        });

driver.findElement(By.id("username")).sendKeys("admin");

driver.findElement(By.id("password")).sendKeys("password");

driver.findElement(By.id("loginBtn")).click();

```


## Example 5: Capture Failed Resource Loading

Browser console often reports missing files.

```java

devTools.addListener(
        Log.entryAdded(),
        log -> {

            if(log.getText()
                  .contains("Failed"))
            {
                System.out.println(
                        "Resource Load Failure");

                System.out.println(
                        log.getText());
            }
        });

```

Sample output:
```text
Failed to load resource:
404 (Not Found)
```


## Example 6: Capture API Errors Reported in Console

Many SPA applications log API failures.

```java
devTools.addListener(
        Log.entryAdded(),
        log -> {

            if(log.getText()
                  .contains("API"))
            {
                System.out.println(
                        "API Error Detected");

                System.out.println(
                        log.getText());
            }
        });
```

Sample output:

```text
API Error Detected

Failed to fetch customer details
HTTP Status 500
```


## Example 7: Store Console Logs in Framework List

```java
List<String> browserLogs =
        new ArrayList<>();

devTools.send(Log.enable());

devTools.addListener(
        Log.entryAdded(),
        log -> {

            browserLogs.add(
                    log.getLevel()
                    + " : "
                    + log.getText());
        });

```

Usage:  Print at End of Test

```java

browserLogs.forEach(System.out::println);

```

## Example 8: Capture Logs with Timestamp

```java

devTools.addListener(
        Log.entryAdded(),
        log -> {

            System.out.println(
                    java.time.LocalDateTime.now()
                    + " | "
                    + log.getLevel()
                    + " | "
                    + log.getText());
        });

```

Sample output:

```text
2026-07-27T09:15:10 | INFO | Page Loaded

2026-07-27T09:15:15 | ERROR | API Timeout

2026-07-27T09:15:20 | WARNING | Deprecated API
```



## Framework Utility Class

```java

public class ConsoleLogger {

    public static void startLogging(
            ChromeDriver driver) {

        DevTools devTools =
                driver.getDevTools();

        devTools.createSession();

        devTools.send(Log.enable());

        devTools.addListener(
                Log.entryAdded(),
                log -> {

                    System.out.println(
                            log.getLevel()
                            + " : "
                            + log.getText());
                });
    }
}

```

Usage:

```java
ChromeDriver driver = new ChromeDriver();

ConsoleLogger.startLogging(driver);

driver.get("https://myapp.com");

```



## Chrome Dev Protocol - usage for capturing Performance Metrics
-------------------------------------------------------------------


Performance metrics collected through Chrome DevTools Protocol (CDP) provide browser-level insights that are not available through traditional Selenium WebDriver APIs.

These metrics help measure:

- Page performance
- Browser resource consumption
- JavaScript execution efficiency
- Memory usage
- DOM complexity
- Rendering performance
- Frontend bottlenecks



## Example 1: Capture All Browser Performance Metrics

```java

import java.util.List;

import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.devtools.DevTools;
import org.openqa.selenium.devtools.v138.performance.Performance;
import org.openqa.selenium.devtools.v138.performance.model.Metric;

ChromeDriver driver = new ChromeDriver();

DevTools devTools = driver.getDevTools();

devTools.createSession();

// Enable Performance Domain
devTools.send(Performance.enable());

driver.get("https://www.google.com");

// Get Metrics
List<Metric> metrics =
        devTools.send(
                Performance.getMetrics());

for(Metric metric : metrics)
{
    System.out.println(
            metric.getName()
            + " = "
            + metric.getValue());
}
```


Sample output:

```text
Timestamp = 12345678

Documents = 1

Frames = 1

JSEventListeners = 22

Nodes = 520

LayoutCount = 12

RecalcStyleCount = 18

JSHeapUsedSize = 3500000

JSHeapTotalSize = 5500000

TaskDuration = 2.45

```

## Example 2: Monitor Page Load Performance


```java
devTools.send(Performance.enable());

driver.get("https://example.com");

List<Metric> metrics =
        devTools.send(
                Performance.getMetrics());

for(Metric metric : metrics)
{
    if(metric.getName()
            .equals("TaskDuration"))
    {
        System.out.println(
                "Page Load Duration: "
                + metric.getValue());
    }
}
```


## Example 3: Capture JavaScript Heap Memory Usage

```java

List<Metric> metrics =
        devTools.send(
                Performance.getMetrics());

for(Metric metric : metrics)
{
    if(metric.getName()
            .equals("JSHeapUsedSize"))
    {
        System.out.println(
                "Heap Used: "
                + metric.getValue());
    }

    if(metric.getName()
            .equals("JSHeapTotalSize"))
    {
        System.out.println(
                "Heap Total: "
                + metric.getValue());
    }
}

```

Sample output:

```text

Heap Used  : 4.2 MB

Heap Total : 8.5 MB

```


## Example 4: Detect Memory Leaks

Capture memory before and after repeated actions.

```java

double initialHeap = 0;
double finalHeap = 0;

List<Metric> initialMetrics =
        devTools.send(
                Performance.getMetrics());

for(Metric metric : initialMetrics)
{
    if(metric.getName()
            .equals("JSHeapUsedSize"))
    {
        initialHeap = metric.getValue();
    }
}

// Perform repeated actions

for(int i=0;i<50;i++)
{
    driver.navigate().refresh();
}

List<Metric> finalMetrics =
        devTools.send(
                Performance.getMetrics());

for(Metric metric : finalMetrics)
{
    if(metric.getName()
            .equals("JSHeapUsedSize"))
    {
        finalHeap = metric.getValue();
    }
}

System.out.println(
        "Heap Growth = "
        + (finalHeap-initialHeap));

```


## Example 5: Capture DOM Node Count

Useful for detecting excessive DOM growth.


```java

List<Metric> metrics =
        devTools.send(
                Performance.getMetrics());

for(Metric metric : metrics)
{
    if(metric.getName().equals("Nodes"))
    {
        System.out.println(
                "DOM Nodes = "
                + metric.getValue());
    }
}

```

Sample output:  DOM Nodes = 1560


## Example 6: Monitor Layout Recalculations

Excessive layout calculations can indicate rendering inefficiencies.

```java

List<Metric> metrics =
        devTools.send(
                Performance.getMetrics());

for(Metric metric : metrics)
{
    if(metric.getName()
             .equals("LayoutCount"))
    {
        System.out.println(
                "Layout Count = "
                + metric.getValue());
    }

    if(metric.getName()
             .equals("RecalcStyleCount"))
    {
        System.out.println(
                "Style Recalculation Count = "
                + metric.getValue());
    }
}

```

Sample output:

```text

Layout Count = 15

Style Recalculation Count = 20

```


## Example 7: Compare Performance Before and After User Action

Scenario:
```text

Search Policy
      ↓
Measure Browser Metrics

```

```java

List<Metric> beforeMetrics =
        devTools.send(
                Performance.getMetrics());

driver.findElement(By.id("searchBtn"))
      .click();

List<Metric> afterMetrics =
        devTools.send(
                Performance.getMetrics());

```

Compare:

- DOM Growth
- Heap Growth
- Layout Count Increase
- Task Duration Increase


## Example 8: Capture Metrics During Login

```java

devTools.send(Performance.enable());

driver.get("https://myapp.com");

driver.findElement(By.id("user"))
      .sendKeys("admin");

driver.findElement(By.id("password"))
      .sendKeys("password");

driver.findElement(By.id("loginBtn"))
      .click();

List<Metric> metrics =
        devTools.send(
                Performance.getMetrics());

metrics.forEach(System.out::println);

```


##  Reusable Performance Utility
----------------------------------


```java

public class PerformanceUtil {

    public static List<Metric> getMetrics(
            ChromeDriver driver) {

        DevTools devTools =
                driver.getDevTools();

        devTools.createSession();

        devTools.send(
                Performance.enable());

        return devTools.send(
                Performance.getMetrics());
    }
}

```

Usage:

```java

List<Metric> metrics =
        PerformanceUtil.getMetrics(driver);

metrics.forEach(System.out::println);

```


## Example 10: Performance KPI Validation

Validate the application against performance SLAs.


```java

double heapSize = 0;

for(Metric metric : metrics)
{
    if(metric.getName()
            .equals("JSHeapUsedSize"))
    {
        heapSize = metric.getValue();
    }
}

Assert.assertTrue(
        heapSize < 10000000,
        "Heap Usage exceeded threshold");

```

**Common CDP Performance Metrics**

Metric	DescriptionTimestamp	Metric collection timestamp
Documents	Number of loaded documents
Frames	Active browser frames
Nodes	Total DOM nodes
LayoutCount	Number of layout calculations
RecalcStyleCount	CSS recalculations
JSEventListeners	Registered JS listeners
JSHeapUsedSize	Actual memory consumed
JSHeapTotalSize	Total allocated JS heap
ScriptDuration	JavaScript execution time
TaskDuration	Browser task execution time
LayoutDuration	Time spent rendering layout
RecalcStyleDuration	CSS recalculation time









 





