## Selenium 4 + TestNG + OWASP ZAP Integration Example

**This example demonstrates how to**:
- Launch browser through OWASP ZAP Proxy
- Execute Selenium UI tests
- Trigger **ZAP Spider Scan**
- Trigger **Active Security Scan**
- Analyze **security alerts**
- Fail TestNG test if High-risk vulnerabilities are found
- Generate HTML security report

## Pre-Requisites

# 1. Install OWASP ZAP
Download and install: https://www.zaproxy.org/download/

# 2. Start ZAP in daemon mode
zap.bat -daemon -port 8080 -config api.disablekey=true

Verify: by visiting http://localhost:8080

# 3. Maven Dependencies 

```xml
<dependencies>
    <!-- Selenium -->
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.24.0</version>
    </dependency>

    <!-- TestNG -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.10.2</version>
        <scope>test</scope>
    </dependency>

    <!-- WebDriverManager -->
    <dependency>
        <groupId>io.github.bonigarcia</groupId>
        <artifactId>webdrivermanager</artifactId>
        <version>5.9.2</version>
    </dependency>

    <!-- ZAP API -->
    <dependency>
        <groupId>org.zaproxy</groupId>
        <artifactId>zap-clientapi</artifactId>
        <version>1.14.0</version>
    </dependency>

</dependencies>
```

# Project structure
```text
src/test/java

 ├── driver
 │     └── DriverFactory.java
 │
 ├── security
 │     ├── ZapManager.java
 │     └── SecurityScanner.java
 │
 └── tests
       └── ZapSecurityTest.java
```

# DriverFactory  
Routes browser traffic through ZAP.

```java
package driver;

import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.Proxy;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

public class DriverFactory {

	public static WebDriver getDriver() {

		WebDriverManager.chromedriver().setup();

		Proxy proxy = new Proxy();

		proxy.setHttpProxy("localhost:8080");
		proxy.setSslProxy("localhost:8080");

		ChromeOptions options = new ChromeOptions();
		options.setProxy(proxy);

		return new ChromeDriver(options);
	}
}
```

# ZapManager
Creates ZAP API connection.

```java
package security;

import org.zaproxy.clientapi.core.ClientApi;

public class ZapManager {

	private static final String HOST = "localhost";
	private static final int PORT = 8080;

	private static ClientApi api = new ClientApi(HOST, PORT, "");

	public static ClientApi getApi() {
		return api;
	}
}
```

# SecurityScanner

Reusable utility class.

```java
package security;

import org.zaproxy.clientapi.core.*;

import java.nio.file.Files;
import java.nio.file.Paths;

public class SecurityScanner {

	private final ClientApi api;

	public SecurityScanner() {
		this.api = ZapManager.getApi();
	}

	public void spider(String target) throws Exception {

		System.out.println("Starting Spider Scan");

		api.spider.scan(target, null, null, null, null);

		int progress = 0;

		while (progress < 100) {

			progress = Integer.parseInt(((ApiResponseElement) api.spider.status("")).getValue());

			System.out.println("Spider Progress = " + progress);

			Thread.sleep(2000);
		}
	}

	public void activeScan(String target) throws Exception {

		System.out.println("Starting Active Scan");

		api.ascan.scan(target, "True", "False", null, null, null);

		int progress = 0;

		while (progress < 100) {

			progress = Integer.parseInt(((ApiResponseElement) api.ascan.status("")).getValue());

			System.out.println("Active Scan Progress = " + progress);

			Thread.sleep(5000);
		}
	}

	public String getAlerts(String target) throws Exception {

		ApiResponse response = api.alert.alerts(target, null, null);

		return response.toString();
	}

	public void generateHtmlReport() throws Exception {

		byte[] report = api.core.htmlreport();

		Files.write(Paths.get("target/ZAP_Report.html"), report);
	}
}
```

## Actual TestNG Security Test

```java
package tests;

import driver.DriverFactory;
import security.SecurityScanner;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.testng.Assert;
import org.testng.annotations.Test;

public class ZapSecurityTest {

	@Test
	public void verifyApplicationSecurity() throws Exception {

		String target = "https://the-internet.herokuapp.com";

		WebDriver driver = DriverFactory.getDriver();

		try {
			driver.get(target);
			driver.findElement(By.linkText("Form Authentication")).click();

			driver.findElement(By.id("username")).sendKeys("tomsmith");

			driver.findElement(By.id("password")).sendKeys("SuperSecretPassword!");

			driver.findElement(By.cssSelector("button[type='submit']")).click();

			// ZAP Security Scan

			SecurityScanner scanner = new SecurityScanner();
			scanner.spider(target);
			scanner.activeScan(target);

			String alerts = scanner.getAlerts(target);
			System.out.println(alerts);

			scanner.generateHtmlReport();

			// Example Validation
			Assert.assertFalse(alerts.contains("High"), "High Severity Vulnerabilities Detected");

		} finally {

			driver.quit();
		}
	}
}
```

# testng.xml

```xml
<!DOCTYPE suite SYSTEM"https://testng.org/testng-1.0.dtd">
<suite name="SecuritySuite">
<test name="ZapSecurityTest">

<classes>
	<class name= "tests.ZapSecurityTest"/>
        </classes>
    </test>
</suite>
```
