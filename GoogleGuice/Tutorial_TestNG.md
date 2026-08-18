Selenium 4 Page Object Model (POM) with Google Guice
Complete Java Tutorial

This tutorial demonstrates how to build a modern Selenium 4 + TestNG + Google Guice automation framework using:

Selenium 4
TestNG
Google Guice Dependency Injection
Page Object Model (POM)
Workflow Layer
Thread-safe WebDriver
Maven
1. Why Google Guice with Selenium?

Without Dependency Injection:

LoginPage loginPage = new LoginPage(driver);
DashboardPage dashboardPage = new DashboardPage(driver);


Problems:

Tight coupling
Too many constructors
Difficult unit testing
Hard to maintain

With Guice:

@Inject
LoginPage loginPage;


Benefits:

Loose coupling
Better maintainability
Easier mocking
Better scalability
Enterprise-grade design
2. Architecture
src
├── main
│
├── driver
│   ├── DriverManager.java
│   └── DriverProvider.java
│
├── modules
│   └── SeleniumModule.java
│
├── pages
│   ├── LoginPage.java
│   └── DashboardPage.java
│
├── workflows
│   └── LoginWorkflow.java
│
├── models
│   └── LoginData.java
│
└── utils
    └── ConfigReader.java

test
├── BaseTest.java
└── LoginTest.java

3. Maven Dependencies
<dependencies>

    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.35.0</version>
    </dependency>

    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.11.0</version>
    </dependency>

    <dependency>
        <groupId>com.google.inject</groupId>
        <artifactId>guice</artifactId>
        <version>7.0.0</version>
    </dependency>

    <dependency>
        <groupId>io.github.bonigarcia</groupId>
        <artifactId>webdrivermanager</artifactId>
        <version>6.3.2</version>
    </dependency>

</dependencies>

4. Create Driver Manager
DriverManager.java
package driver;

import org.openqa.selenium.WebDriver;

public class DriverManager {

    private static final ThreadLocal<WebDriver> DRIVER =
            new ThreadLocal<>();

    public static void setDriver(WebDriver driver) {
        DRIVER.set(driver);
    }

    public static WebDriver getDriver() {
        return DRIVER.get();
    }

    public static void removeDriver() {
        DRIVER.remove();
    }
}

5. Create Driver Provider

Provider supplies objects to Guice.

DriverProvider.java
package driver;

import com.google.inject.Provider;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class DriverProvider implements Provider<WebDriver> {

    @Override
    public WebDriver get() {

        WebDriver driver = DriverManager.getDriver();

        if (driver == null) {

            WebDriverManager.chromedriver().setup();

            driver = new ChromeDriver();

            DriverManager.setDriver(driver);
        }

        return driver;
    }
}

6. Create Guice Module
SeleniumModule.java
package modules;

import com.google.inject.AbstractModule;
import com.google.inject.Singleton;

import driver.DriverProvider;

import org.openqa.selenium.WebDriver;

public class SeleniumModule extends AbstractModule {

    @Override
    protected void configure() {

        bind(WebDriver.class)
                .toProvider(DriverProvider.class)
                .in(Singleton.class);
    }
}

7. Create POJO Test Data
LoginData.java
package models;

public class LoginData {

    private String username;
    private String password;

    public LoginData(String username,
                     String password) {

        this.username = username;
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }
}

8. Create Login Page
LoginPage.java
package pages;

import com.google.inject.Inject;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class LoginPage {

    private final WebDriver driver;

    @Inject
    public LoginPage(WebDriver driver) {
        this.driver = driver;
    }

    private final By username =
            By.id("user-name");

    private final By password =
            By.id("password");

    private final By loginButton =
            By.id("login-button");

    public void open() {

        driver.get(
         "https://www.saucedemo.com/");
    }

    public void enterUsername(String value) {

        driver.findElement(username)
              .sendKeys(value);
    }

    public void enterPassword(String value) {

        driver.findElement(password)
              .sendKeys(value);
    }

    public void clickLogin() {

        driver.findElement(loginButton)
              .click();
    }
}

9. Create Dashboard Page
DashboardPage.java
package pages;

import com.google.inject.Inject;
import org.openqa.selenium.WebDriver;

public class DashboardPage {

    private final WebDriver driver;

    @Inject
    public DashboardPage(WebDriver driver) {
        this.driver = driver;
    }

    public String getTitle() {

        return driver.getTitle();
    }
}

10. Create Workflow Layer

Instead of putting business actions inside tests.

LoginWorkflow.java
package workflows;

import com.google.inject.Inject;

import models.LoginData;
import pages.DashboardPage;
import pages.LoginPage;

public class LoginWorkflow {

    private final LoginPage loginPage;
    private final DashboardPage dashboardPage;

    @Inject
    public LoginWorkflow(
            LoginPage loginPage,
            DashboardPage dashboardPage) {

        this.loginPage = loginPage;
        this.dashboardPage = dashboardPage;
    }

    public DashboardPage loginAs(
            LoginData data) {

        loginPage.open();

        loginPage.enterUsername(
            data.getUsername());

        loginPage.enterPassword(
            data.getPassword());

        loginPage.clickLogin();

        return dashboardPage;
    }
}

11. Create Base Test
BaseTest.java
package tests;

import com.google.inject.Guice;
import com.google.inject.Injector;

import driver.DriverManager;
import modules.SeleniumModule;

import org.openqa.selenium.WebDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;

public class BaseTest {

    protected Injector injector;

    @BeforeMethod
    public void setup() {

        injector =
            Guice.createInjector(
                new SeleniumModule());

        WebDriver driver =
            injector.getInstance(
                WebDriver.class);

        driver.manage()
              .window()
              .maximize();
    }

    @AfterMethod
    public void tearDown() {

        WebDriver driver =
            DriverManager.getDriver();

        if (driver != null) {
            driver.quit();
            DriverManager.removeDriver();
        }
    }
}

12. Create Test Class
LoginTest.java
package tests;

import models.LoginData;

import org.testng.Assert;
import org.testng.annotations.Test;

import pages.DashboardPage;
import workflows.LoginWorkflow;

public class LoginTest
        extends BaseTest {

    @Test
    public void verifyLogin() {

        LoginWorkflow workflow =
                injector.getInstance(
                        LoginWorkflow.class);

        LoginData data =
                new LoginData(
                        "standard_user",
                        "secret_sauce");

        DashboardPage dashboard =
                workflow.loginAs(data);

        Assert.assertTrue(
                dashboard.getTitle()
                         .contains("Swag"));
    }
}

13. Using TestNG @Guice Annotation

Instead of manually creating Injector:

@Guice(modules = SeleniumModule.class)
public class LoginTest {

    @Inject
    LoginWorkflow loginWorkflow;

    @Test
    public void verifyLogin() {

        LoginData data =
                new LoginData(
                    "standard_user",
                    "secret_sauce");

        loginWorkflow.loginAs(data);
    }
}


This is the cleaner and preferred approach.

14. Parallel Execution Support

Ensure:

ThreadLocal<WebDriver>


is used.

TestNG:

<suite name="Suite"
       parallel="methods"
       thread-count="5">

    <test name="Regression">
        <classes>
            <class name="tests.LoginTest"/>
        </classes>
    </test>

</suite>


Each thread receives its own:

WebDriver


instance.

15. Enterprise Enhancements

A production-ready framework should additionally include:

Framework Layer
factory/
listeners/
annotations/
retry/
reporting/

Reporting
Allure Reports
Extent Reports
Test Data
JSON
CSV
Excel
Database
Design Patterns
POM
Page Component Object
Workflow Pattern
Factory Pattern
Strategy Pattern
Singleton Pattern
TestNG Listeners
ITestListener
IInvokedMethodListener
IDataProviderInterceptor
IAnnotationTransformer
IConfigurationListener
Recommended Dependency Flow
Test
 ↓
Workflow
 ↓
Page
 ↓
Selenium WebDriver
 ↓
Browser


This keeps tests business-focused, pages UI-focused, and Guice responsible for object creation, resulting in a scalable enterprise Selenium 4 automation framework.
