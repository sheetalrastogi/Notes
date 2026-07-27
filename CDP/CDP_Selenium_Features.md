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



 





