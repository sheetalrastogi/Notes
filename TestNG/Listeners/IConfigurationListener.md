## IConfigurationListener 
---

**IConfigurationListener** listener is used to configure and monitor Test execution with TestNG Annotation such as

- @BeforeSuite
- @BeforeTest
- @BeforeClass
- @BeforeMethod
- @BeforeGroups
- @AfterSuite
- @AfterTest
- @AfterClass
- @AfterMethod
- @AfterGroups

**Typical use cases are**:
  
- Setup / Monitor setup and cleanup test pre-requisites eg. 
 - DB Connections, 
 - Browser Instantiation, 
 - Logging, 
 - Reporting, 
 - TestData



**Example implementation**:

IConfigurationListener can be implemented to audit framework setup/teardown activity and capture information required for compliance, traceability, and production support eg.

# Example Audit Information Captured

When any configuration method executes:

- Execution Timestamp
- Environment (DEV/UAT/PROD)
- Browser Name and Version
- Selenium Grid Node
- Session ID
- Test Class Name
- Configuration Method Name
- User ID
- Machine Name
- Thread ID
- Pass/Fail Status
- Exception Details
- Execution Duration


# Step 1: Sample Audit Logger

```java
package framework.audit;

import java.time.LocalDateTime;

public class AuditLogger {

	public static void log(String event, String testClass, String configMethod, String status, String remarks) {

		System.out.printf("[AUDIT] Time=%s | Event=%s | Class=%s | Method=%s | Status=%s | Remarks=%s%n",
				LocalDateTime.now(), event, testClass, configMethod, status, remarks);
	}
}
```


# Step 2: IConfigurationListener Implementation

```java
package framework.listeners;

import framework.audit.AuditLogger;
import org.testng.IConfigurationListener;
import org.testng.ITestResult;

public class AuditConfigurationListener implements IConfigurationListener {

	@Override
	public void onConfigurationSuccess(ITestResult result) {

		AuditLogger.log("CONFIGURATION", result.getTestClass().getName(), result.getMethod().getMethodName(), "PASSED",
				"Configuration executed successfully");
	}

	@Override
	public void onConfigurationFailure(ITestResult result) {

		String reason = result.getThrowable() == null ? "Unknown" : result.getThrowable().getMessage();

		AuditLogger.log("CONFIGURATION", result.getTestClass().getName(), result.getMethod().getMethodName(), "FAILED",
				reason);
	}

	@Override
	public void onConfigurationSkip(ITestResult result) {

		AuditLogger.log("CONFIGURATION", result.getTestClass().getName(), result.getMethod().getMethodName(), "SKIPPED",
				"Configuration skipped");
	}
}
```


# Step 3: Selenium Framework Base Class

```java
public class BaseTest {

	@BeforeMethod
	public void launchBrowser() {

		DriverManager.createDriver();

		System.out.println("Browser Started");
	}

	@AfterMethod
	public void closeBrowser() {

		DriverManager.quitDriver();

		System.out.println("Browser Closed");
	}
}
```

# Step 4: Register Listener

**Option 1** via testng.xml

```xml
<listeners>
    <listener class-name="framework.listeners.AuditConfigurationListener"/>
</listeners>
```

**Option 2** via @Listeners annotation

```java
@Listeners(AuditConfigurationListener.class)
public class LoginTest {
}
```


# Sample Audit Output

```text
[AUDIT] Time=2026-08-14T10:05:12
Event=CONFIGURATION
Class=tests.LoginTest
Method=launchBrowser
Status=PASSED
Remarks=Configuration executed successfully

[AUDIT] Time=2026-08-14T10:08:40
Event=CONFIGURATION
Class=tests.LoginTest
Method=closeBrowser
Status=PASSED
Remarks=Configuration executed successfully
```


Other Enhancements to custom logger:

SessionID, Browser and Browser Version:
```java

 WebDriver driver = DriverManager.getDriver();
 String sessionId = ((RemoteWebDriver) driver).getSessionId().toString();
 String browser = ((RemoteWebDriver) driver).getCapabilities().getBrowserName();
 String version = driver.getCapabilities().getBrowserVersion();
 String environment = Config.get("environment");
 String executor = System.getProperty("user.name");
 long threadId = Thread.currentThread().getId();

```

# Typical Configuration Methods Audited
```text
@BeforeSuite → Framework startup
@BeforeTest → Environment initialization
@BeforeClass → Test data loading
@BeforeMethod → Browser launch/login
@AfterMethod → Cleanup activities
@AfterClass → Resource release
@AfterSuite → Report generation
```
