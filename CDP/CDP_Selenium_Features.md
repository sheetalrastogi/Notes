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

