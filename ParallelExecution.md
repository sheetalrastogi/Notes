# Parallel Script Execution in POM Selenium Java

**Parallel execution in a Page Object Model (POM) framework is typically achieved using:**

- TestNG Parallel Execution
- ThreadLocal WebDriver
- Parallel Test Methods / Classes
- Selenium Grid (optional for distributed execution)

Recommended Framework Structure
```text
src/test/java
│
├── pages
│   ├── LoginPage.java
│   ├── HomePage.java
│   └── BasePage.java
│
├── tests
│   ├── LoginTest.java
│   └── SearchTest.java
│
├── base
│   ├── DriverFactory.java
│   └── BaseTest.java
│
└── utils
```

**Why Normal WebDriver Fails in Parallel Execution**

❌ Shared WebDriver instance:
```java
public static WebDriver driver;
```

When multiple threads run simultaneously:

```java
Thread-1 -> Opens Chrome
Thread-2 -> Opens Chrome

Thread-1 -> Navigates to Google
Thread-2 -> Navigates to Amazon
```

Both threads share same driver.  Tests become unstable.

**Solution**: 
- ThreadLocal WebDriver  Each thread gets its own WebDriver instance.

```java
DriverFactory.java
public class DriverFactory {

    private static ThreadLocal<WebDriver> driver =
            new ThreadLocal<>();

    public static void setDriver(WebDriver webDriver) {
        driver.set(webDriver);
    }

    public static WebDriver getDriver() {
        return driver.get();
    }

    public static void quitDriver() {

        if(driver.get() != null) {
            driver.get().quit();
            driver.remove();
        }
    }
}
```

Base Test

BaseTest.java

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.*;

public class BaseTest {

    @BeforeMethod
    public void setup() {

        WebDriver driver = new ChromeDriver();

        DriverFactory.setDriver(driver);

        DriverFactory.getDriver()
                     .manage()
                     .window()
                     .maximize();
    }

    @AfterMethod
    public void tearDown() {

        DriverFactory.quitDriver();
    }
}
```

Base Page

All Page Objects should use ThreadLocal driver.

BasePage.java
```java
import org.openqa.selenium.support.PageFactory;

public class BasePage {

    public BasePage() {

        PageFactory.initElements(
            DriverFactory.getDriver(),
            this);
    }
}
```

Login Page

```java
LoginPage.java
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

public class LoginPage extends BasePage {

    @FindBy(id="username")
    private WebElement txtUserName;

    @FindBy(id="password")
    private WebElement txtPassword;

    @FindBy(id="loginBtn")
    private WebElement btnLogin;

    public void login(
            String user,
            String pass) {

        txtUserName.sendKeys(user);
        txtPassword.sendKeys(pass);
        btnLogin.click();
    }
}
```

Parallel Test Class
LoginTest.java

```java
import org.testng.annotations.Test;

public class LoginTest extends BaseTest {

    @Test
    public void loginAsAdmin() {

        DriverFactory.getDriver()
                     .get("https://example.com");

        LoginPage loginPage =
                new LoginPage();

        loginPage.login(
                "admin",
                "admin123");
    }
}
```

# TestNG Parallel Configuration
testng.xml
**Parallel By Classes**
```text
<!DOCTYPE suite SYSTEM
"https://testng.org/testng-1.0.dtd">
<suite name="Suite" parallel="classes" thread-count="3">
    <test name="Tests">
        <classes>
            <class name="tests.LoginTest"/>
            <class name="tests.SearchTest"/>
            <class name="tests.CartTest"/>
        </classes>
    </test>
</suite>
```

**Parallel By Methods**
```text
<suite name="Suite" parallel="methods" thread-count="5">
```

Example:
```java
@Test
public void test1() {}

@Test
public void test2() {}

@Test
public void test3() {}
```

**All methods can run simultaneously.**

# Accessing Driver in Page Classes

Avoid:
```java
driver.findElement(...)
```

Use:
```java
DriverFactory.getDriver()
             .findElement(...);
```

Or in constructor:
```java
public class LoginPage {

    private WebDriver driver;

    public LoginPage() {
        driver = DriverFactory.getDriver();
        PageFactory.initElements(driver, this);
    }
}
```

# Parallel DataProvider Execution
```java
@DataProvider(
        name="loginData",
        parallel=true)
public Object[][] loginData() {

    return new Object[][]{
        {"admin","admin123"},
        {"user1","pass1"},
        {"user2","pass2"}
    };
}

@Test(dataProvider="loginData")
public void loginTest(
        String user,
        String pass) {

}
```

Each dataset executes in its own thread.

# Selenium Grid + Parallel Execution

For large-scale execution:
```java
Thread 1 --> Chrome Windows
Thread 2 --> Edge Windows
Thread 3 --> Chrome Linux
Thread 4 --> Firefox Linux
```

Use:
```java
RemoteWebDriver
```

instead of:
```java
ChromeDriver
```

Example:

```java
WebDriver driver =
    new RemoteWebDriver(
        new URL("http://localhost:4444"),
        new ChromeOptions());
```

**Common Interview Question**
- Why is ThreadLocal used for parallel execution?
Answer:
ThreadLocal<WebDriver> creates a separate WebDriver instance for each execution thread. This prevents multiple test threads from sharing the same browser session, ensuring thread safety and reliable parallel execution.

# Enterprise Best Practices
```text
✅ Use ThreadLocal<WebDriver>
✅ Initialize browser in @BeforeMethod
✅ Quit browser in @AfterMethod
✅ Keep WebDriver out of page classes as static
✅ Use Page Object Model
✅ Use TestNG parallel execution
✅ Make DataProviders parallel when needed
✅ Run on Selenium Grid for scalability
✅ Generate thread-safe Extent/Allure reports
✅ Avoid sharing test data across threads
```
This is the most commonly used enterprise approach for implementing parallel execution in Selenium Java POM frameworks.
