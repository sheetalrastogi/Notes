# using TestNG Selenium 4 Java, Example to execute all test in a Thread-safe parallel execution mode


## For Thread-safe Parallel Execution in Selenium 4 + TestNG, the key principle is:

```text
Every test thread must get its own WebDriver instance using ThreadLocal<WebDriver>.
```

# This prevents issues such as:

- Browser sessions overwriting each other
- Element references crossing threads
- Test data corruption
- Random parallel execution failures


# 1. Thread Safe DriverManager


```java
package driver;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public final class DriverManager {

	private DriverManager() {
	}

	private static final ThreadLocal<WebDriver> DRIVER = new ThreadLocal<>();

	public static void createDriver() {

		WebDriver driver = new ChromeDriver();

		DRIVER.set(driver);
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


# 2. Base Test

Each test method gets its own browser.

```java
package tests;

import driver.DriverManager;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;

public class BaseTest {

	@BeforeMethod
	public void setup() {

		DriverManager.createDriver();
	}

	@AfterMethod(alwaysRun = true)
	public void teardown() {

		DriverManager.quitDriver();
	}
}
```

# 3. Thread Safe Page Object

Never store WebDriver in static variables.

```java
package pages;

import driver.DriverManager;
import org.openqa.selenium.By;

public class LoginPage {

	private By username = By.id("username");
	private By password = By.id("password");
	private By loginBtn = By.id("loginBtn");

	public LoginPage enterUsername(String user) {
		DriverManager.getDriver().findElement(username).sendKeys(user);
		return this;
	}

	public LoginPage enterPassword(String pass) {
		DriverManager.getDriver().findElement(password).sendKeys(pass);
		return this;
	}

	public DashboardPage login() {
		DriverManager.getDriver().findElement(loginBtn).click();
		return new DashboardPage();
	}
}

```

# 4. Dashboard Page

```java
package pages;

import driver.DriverManager;
import org.openqa.selenium.By;

public class DashboardPage {

	private By welcomeText = By.id("welcomeUser");

	public String getLoggedInUser() {

		return DriverManager.getDriver().findElement(welcomeText).getText();
	}
}
```


# 5. Parallel Test Class

```java
package tests;

import org.testng.Assert;
import org.testng.annotations.Test;
import pages.DashboardPage;
import pages.LoginPage;

public class LoginTest extends BaseTest {

	@Test
	public void loginUser1() {

		DriverManager.getDriver().get("https://app.com");

		DashboardPage dashboard = new LoginPage().enterUsername("user1").enterPassword("pwd1").login();

		Assert.assertEquals(dashboard.getLoggedInUser(), "user1");
	}

	@Test
	public void loginUser2() {

		DriverManager.getDriver().get("https://app.com");

		DashboardPage dashboard = new LoginPage().enterUsername("user2").enterPassword("pwd2").login();

		Assert.assertEquals(dashboard.getLoggedInUser(), "user2");
	}
}

```

# 6. Parallel DataProvider

TestNG can run data rows in parallel.

```java
@DataProvider(name = "users", parallel = true)
public Object[][] users() {

    return new Object[][]{
        {"admin","admin123"},
        {"john","john123"},
        {"mary","mary123"},
        {"alex","alex123"}
    };
}

```

# 7. Usage

```java
	@Test(dataProvider = "users")
	public void loginTest(String user, String password) {

		DriverManager.getDriver().get("https://app.com");

		new LoginPage().enterUsername(user).enterPassword(password).login();
	}
```

# Parallel testng.xml

**Method Level Parallelism**

```xml
<!DOCTYPE suite SYSTEM
"https://testng.org/testng-1.0.dtd">

<suite name="Regression"
       parallel="methods"
       thread-count="5">

    <test name="UI Tests">

        <classes>
            <class name="tests.LoginTest"/>
        </classes>

    </test>

</suite>
```


**Class Level Parallelism**

```xml
<suite name="Regression"
       parallel="classes"
       thread-count="4">
</suite>
```

**Test Level Parallelism**

```xml
<suite name="Regression"
       parallel="tests"
       thread-count="3">
</suite>

```


# Logging Thread-ID

```java
@BeforeMethod
public void setup(Method method) {

    DriverManager.createDriver();

    System.out.println(
        method.getName()
        + " Thread="
        + Thread.currentThread().getId()
    );
}
```

# Thread Safe Extent Report

Store test objects in ThreadLocal.

```java
public class ExtentManager {

    private static ThreadLocal<ExtentTest>
            extentTest = new ThreadLocal<>();

    public static void setTest(ExtentTest test) {
        extentTest.set(test);
    }

    public static ExtentTest getTest() {
        return extentTest.get();
    }
}
```

