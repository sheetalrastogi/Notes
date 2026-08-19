## Thread-Safe Singleton DriverFactory

```java
package framework.driver;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public final class DriverFactory {

	private static final DriverFactory INSTANCE = new DriverFactory();
	private final ThreadLocal<WebDriver> driver = new ThreadLocal<>();

	private DriverFactory() { }

	public static DriverFactory getInstance() {
		return INSTANCE;
	}

	public void createDriver() {
		if (driver.get() == null) {
			WebDriver webDriver = new ChromeDriver();
			driver.set(webDriver);
		}
	}

	public WebDriver getDriver() {
		return driver.get();
	}

	public void quitDriver() {
		if (driver.get() != null) {
			driver.get().quit();
			driver.remove();
		}
	}
}
```

Usage:

```java
@BeforeMethod
public void setup() {

    DriverFactory.getInstance()
                 .createDriver();
}

@Test
public void loginTest() {

    WebDriver driver =
            DriverFactory.getInstance()
                         .getDriver();

    driver.get("https://www.google.com");
}

@AfterMethod
public void teardown() {

    DriverFactory.getInstance()
                 .quitDriver();
}
```


---

# Thread-Safe Extent Test Manager

```java
package framework.reporting;

import com.aventstack.extentreports.ExtentTest;

public final class ExtentTestManager {

	private static final ThreadLocal<ExtentTest> TEST = new ThreadLocal<>();

	private ExtentTestManager() {
	}

	public static void setTest(ExtentTest extentTest) {
		TEST.set(extentTest);
	}

	public static ExtentTest getTest() {
		return TEST.get();
	}

	public static void unload() {
		TEST.remove();
	}
}

```

**TestNG Listener Integration for ExtentReports**

```java
package framework.listeners;

import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.ExtentTest;
import framework.reporting.ExtentReportManager;
import framework.reporting.ExtentTestManager;
import org.testng.ITestListener;
import org.testng.ITestResult;

public class TestListener implements ITestListener {

	private final ExtentReports extent = ExtentReportManager.getInstance();

	@Override
	public void onTestStart(ITestResult result) {
		ExtentTest test = extent.createTest(result.getMethod().getMethodName());
		ExtentTestManager.setTest(test);
	}

	@Override
	public void onTestSuccess(ITestResult result) {
		ExtentTestManager.getTest().pass("Test Passed");
	}

	@Override
	public void onTestFailure(ITestResult result) {
		ExtentTestManager.getTest().fail(result.getThrowable());
	}

	@Override
	public void onFinish(org.testng.ITestContext context) {
		extent.flush();
	}
}
```

## POM Usage

- BasePage

```java

public abstract class BasePage {

	protected WebDriver driver;

	protected BasePage() {
		this.driver = DriverFactory.getInstance().getDriver();
	}
}

```

- Login Page

```java
package pages;

import org.openqa.selenium.By;

public class LoginPage extends BasePage {

	private final By username = By.id("user-name");
	private final By password = By.id("password");
	private final By loginButton = By.id("login-button");

	public void open() {
		driver.get("https://www.saucedemo.com");
	}

	public void login(String user, String pwd) {
		driver.findElement(username).sendKeys(user);
		driver.findElement(password).sendKeys(pwd);
		driver.findElement(loginButton).click();
	}
}
```

- Test Class

```java
@Listeners(TestListener.class)
public class LoginTest {

	@BeforeMethod
	public void setup() {
		DriverFactory.getInstance().createDriver();
	}

	@Test
	public void verifyLogin() {
		LoginPage page = new LoginPage();
		page.open();
		page.login("standard_user", "secret_sauce");
		ExtentTestManager.getTest().info("User logged in");
	}

	@AfterMethod
	public void cleanup() {
		DriverFactory.getInstance().quitDriver();
		ExtentTestManager.unload();
	}
}
```

