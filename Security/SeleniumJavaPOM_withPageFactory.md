A **Factory Design Pattern** is one of the most common ways to **create and manage WebDriver instances** in Selenium Automation Frameworks. 

It helps:

- Centralize driver creation logic
- Support multiple browsers (Chrome, Firefox, Edge)
- Support local and remote execution
- Improve maintainability
- Easily integrate with TestNG parallel execution


Framework Structure

```text
src/test/java
│
├── base
│   ├── BaseTest.java
│
├── factory
│   ├── DriverFactory.java
│
├── pages
│   ├── LoginPage.java
│   └── HomePage.java
│
├── tests
│   └── LoginTest.java
│
└── utils
    └── ConfigReader.java
```

##1. Driver Factory Class

This class is responsible for creating WebDriver objects.

```java

package factory;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.edge.EdgeDriver;
import org.openqa.selenium.firefox.FirefoxDriver;

public class DriverFactory {

    private static ThreadLocal<WebDriver> driver = new ThreadLocal<>();

    public static WebDriver initDriver(String browser) {

        switch (browser.toLowerCase()) {

            case "chrome":
                driver.set(new ChromeDriver());
                break;

            case "firefox":
                driver.set(new FirefoxDriver());
                break;

            case "edge":
                driver.set(new EdgeDriver());
                break;

            default:
                throw new RuntimeException("Browser not supported : " + browser);
        }

        getDriver().manage().window().maximize();

        return getDriver();
    }

    public static WebDriver getDriver() {
        return driver.get();
    }

    public static void quitDriver() {

        if (driver.get() != null) {
            driver.get().quit();
            driver.remove();
        }
    }
}
```


##2. Base Test Class

All tests inherit from this class.

```java
package base;

import factory.DriverFactory;
import org.openqa.selenium.WebDriver;
import org.testng.annotations.*;

public class BaseTest {

    protected WebDriver driver;

    @Parameters("browser")
    @BeforeMethod
    public void setup(String browser) {

        driver = DriverFactory.initDriver(browser);

        driver.get("https://opensource-demo.orangehrmlive.com/");
    }

    @AfterMethod
    public void tearDown() {

        DriverFactory.quitDriver();
    }
}
```


##3. Page Object Model (Login Page)

Rather than passing driver everywhere, fetch it from Factory.

```java

package pages;

import factory.DriverFactory;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class LoginPage {

    WebDriver driver;

    public LoginPage() {

        driver = DriverFactory.getDriver();
    }

    private By username = By.name("username");

    private By password = By.name("password");

    private By loginBtn = By.xpath("//button[@type='submit']");

    public LoginPage enterUsername(String user) {
        driver.findElement(username).sendKeys(user);
        return this;
    }

    public LoginPage enterPassword(String pass) {
        driver.findElement(password).sendKeys(pass);
        return this;
    }

    public HomePage clickLogin() {
        driver.findElement(loginBtn).click();
        return new HomePage();
    }
}
```


##4. Home Page

```java

package pages;

import factory.DriverFactory;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class HomePage {

    private WebDriver driver;

    public HomePage() {

        driver = DriverFactory.getDriver();
    }

    private By dashboard = By.xpath("//h6[text()='Dashboard']");

    public boolean isDashboardDisplayed() {

        return driver.findElement(dashboard).isDisplayed();
    }
}

```


## 5. Test Class Using POM

```java

package tests;

import base.BaseTest;
import org.testng.Assert;
import org.testng.annotations.Test;

import pages.LoginPage;
import pages.HomePage;

public class LoginTest extends BaseTest {

    @Test
    public void verifyLogin() {

        HomePage homePage = new LoginPage()
                        .enterUsername("Admin")
                        .enterPassword("admin123")
                        .clickLogin();

        Assert.assertTrue(homePage.isDashboardDisplayed());
    }
}
```

##6. TestNG XML

```xml

<!DOCTYPE suite SYSTEM 
"https://testng.org/testng-1.0.dtd">

<suite name="CrossBrowserSuite" parallel="tests" thread-count="3">
    <test name="Chrome">
        <parameter name="browser" value="chrome"/>

        <classes>
            <class name="tests.LoginTest"/>
        </classes>

    </test>

    <test name="Firefox">

        <parameter name="browser" value="firefox"/>

        <classes>
            <class name="tests.LoginTest"/>
        </classes>

    </test>

</suite>
```


**Advanced Enterprise Version**

##Most frameworks extend the factory to support:

```java
DriverFactory.initDriver(
        BrowserType.CHROME,
        ExecutionMode.REMOTE,
        headless=true,
        incognito=true);
```

Example:

```java
WebDriver driver =
        WebDriverFactory.createDriver(
                Browser.CHROME,
                Environment.QA,
                ExecutionMode.REMOTE);
```
