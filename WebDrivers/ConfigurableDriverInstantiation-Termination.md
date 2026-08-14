Objective:
Design a configurable Selenium 4 Java browser lifecycle management solution that controls WebDriver instantiation and termination through a properties file, enabling browser closure either after each test method execution or once at the end of the entire test suite without requiring code changes.

A clean approach is to make the browser lifecycle configurable using a property such as:
	- browser.lifecycle=METHOD   or
	- browser.lifecycle=SUITE
	
Where:

METHOD → Browser starts and closes for every @Test
SUITE → Single browser instance shared across entire suite


# Step 1. Framework Properties

```text
# browser.properties
browser=chrome

# METHOD | SUITE
browser.lifecycle=METHOD
```

# Step 2. Browser Lifecycle Enum

```java
package framework.config;

public enum BrowserLifecycle {

	METHOD, SUITE;

	public static BrowserLifecycle from(String value) {
		return BrowserLifecycle.valueOf(value.trim().toUpperCase());
	}
}

```

# Step 3. Configuration Reader

```java
package framework.config;

import java.io.InputStream;
import java.util.Properties;

public class ConfigManager {

	private static final Properties PROPERTIES = new Properties();

	static {
		try (InputStream is = ConfigManager.class.getClassLoader().getResourceAsStream("browser.properties")) {

			PROPERTIES.load(is);

		} catch (Exception e) {
			throw new RuntimeException("Unable to load properties", e);
		}
	}

	public static String get(String key) {
		return PROPERTIES.getProperty(key);
	}

	public static BrowserLifecycle lifecycle() {

		return BrowserLifecycle.from(get("browser.lifecycle"));
	}
}
```

# Step 4. Driver Manager

```java
package framework.driver;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public final class DriverManager {

	private static final ThreadLocal<WebDriver> DRIVER = new ThreadLocal<>();

	private DriverManager() {
	}

	public static void createDriver() {

		if (DRIVER.get() == null) {

			WebDriver driver = new ChromeDriver();

			DRIVER.set(driver);
		}
	}

	public static WebDriver getDriver() {
		return DRIVER.get();
	}

	public static void quitDriver() {

		WebDriver driver = DRIVER.get();

		if (driver != null) {

			driver.quit();
			DRIVER.remove();
		}
	}
}
```

# Step 5. Browser Lifecycle Controller

Central place controlling browser creation and cleanup.

```java
package framework.lifecycle;

import framework.config.BrowserLifecycle;
import framework.config.ConfigManager;
import framework.driver.DriverManager;

public class BrowserLifecycleManager {

	private static final BrowserLifecycle MODE = ConfigManager.lifecycle();

	public static void beforeSuite() {

		if (MODE == BrowserLifecycle.SUITE) {

			DriverManager.createDriver();

			System.out.println("Suite Browser Created");
		}
	}

	public static void afterSuite() {

		if (MODE == BrowserLifecycle.SUITE) {

			DriverManager.quitDriver();

			System.out.println("Suite Browser Closed");
		}
	}

	public static void beforeMethod() {

		if (MODE == BrowserLifecycle.METHOD) {

			DriverManager.createDriver();

			System.out.println("Method Browser Created");
		}
	}

	public static void afterMethod() {

		if (MODE == BrowserLifecycle.METHOD) {

			DriverManager.quitDriver();

			System.out.println("Method Browser Closed");
		}
	}
}
```


# Step 6. BaseTest

```java
package framework.base;

import framework.lifecycle.BrowserLifecycleManager;
import org.testng.annotations.*;

public class BaseTest {

	@BeforeSuite(alwaysRun = true)
	public void beforeSuite() {

		BrowserLifecycleManager.beforeSuite();
	}

	@AfterSuite(alwaysRun = true)
	public void afterSuite() {

		BrowserLifecycleManager.afterSuite();
	}

	@BeforeMethod(alwaysRun = true)
	public void beforeMethod() {

		BrowserLifecycleManager.beforeMethod();
	}

	@AfterMethod(alwaysRun = true)
	public void afterMethod() {

		BrowserLifecycleManager.afterMethod();
	}
}
```


# Step 7. Sample Test

```java
public class LoginTest extends BaseTest {

	@Test
	public void testLogin() {

		DriverManager.getDriver().get("https://www.google.com");
	}

	@Test
	public void testSearch() {

		DriverManager.getDriver().get("https://www.bing.com");
	}
}
```

# Execution Scenario 1 : Per Method Browser

property file configuration:
	browser.lifecycle=METHOD


Output:
```text
@BeforeMethod:  Create Chrome

@Test Login

@AfterMethod:   Quit Chrome

@BeforeMethod:  Create Chrome

@Test Search

@AfterMethod    Quit Chrome

```


# Execution Scenario 2 : One Browser Per Suite

property file configuration:
	browser.lifecycle=SUITE

Output:

```text
@BeforeSuite: Create Chrome

@Test Login

@Test Search

@AfterSuite:  Quit Chrome

```


**Other enhancements to consider**:

Allow configuration to reuse Browser at:

- Every Test Method
- Every Test Class
- Every <test> block in testng.xml
- Entire Suite


