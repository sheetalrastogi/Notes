Page Object Model (POM) in Selenium Java

Page Object Model (POM) is a design pattern in Selenium where:

Each web page is represented by a separate Java class.
Web elements and actions for that page are encapsulated within the page class.
Test classes interact with page classes instead of directly interacting with Selenium APIs.
Improves maintainability, reusability, and readability.
Project Structure
src/test/java
│
├── pages
│   ├── LoginPage.java
│   ├── HomePage.java
│   └── BasePage.java
│
├── tests
│   └── LoginTest.java
│
├── utils
│   ├── DriverFactory.java
│   └── ConfigReader.java
│
└── base
    └── BaseTest.java

1. Base Page Class

Common methods used by all page classes.

package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;

import java.time.Duration;

public class BasePage {

    protected WebDriver driver;
    protected WebDriverWait wait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver,
                Duration.ofSeconds(10));
    }

    protected void click(WebElement element) {
        wait.until(
                ExpectedConditions.elementToBeClickable(element))
                .click();
    }

    protected void type(WebElement element,
                        String text) {

        wait.until(
                ExpectedConditions.visibilityOf(element));

        element.clear();
        element.sendKeys(text);
    }

    protected String getText(WebElement element) {

        return wait.until(
                ExpectedConditions.visibilityOf(element))
                .getText();
    }
}

2. Login Page Class
Using PageFactory
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

public class LoginPage extends BasePage {

    public LoginPage(WebDriver driver) {
        super(driver);
        PageFactory.initElements(driver, this);
    }

    @FindBy(id = "username")
    private WebElement txtUserName;

    @FindBy(id = "password")
    private WebElement txtPassword;

    @FindBy(id = "loginBtn")
    private WebElement btnLogin;

    public LoginPage enterUserName(String username) {

        type(txtUserName, username);
        return this;
    }

    public LoginPage enterPassword(String password) {

        type(txtPassword, password);
        return this;
    }

    public HomePage clickLogin() {

        click(btnLogin);
        return new HomePage(driver);
    }
}

3. Home Page Class
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

public class HomePage extends BasePage {

    public HomePage(WebDriver driver) {
        super(driver);
        PageFactory.initElements(driver, this);
    }

    @FindBy(id = "welcome")
    private WebElement lblWelcome;

    public String getWelcomeMessage() {
        return getText(lblWelcome);
    }
}

4. Base Test Class
package base;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;

public class BaseTest {

    protected WebDriver driver;

    @BeforeMethod
    public void setup() {

        driver = new ChromeDriver();

        driver.manage().window().maximize();

        driver.get(
          "https://example.com/login");
    }

    @AfterMethod
    public void tearDown() {

        if(driver != null) {
            driver.quit();
        }
    }
}

5. Test Class
package tests;

import base.BaseTest;
import pages.HomePage;
import pages.LoginPage;

import org.testng.Assert;
import org.testng.annotations.Test;

public class LoginTest extends BaseTest {

    @Test
    public void verifySuccessfulLogin() {

        HomePage homePage =
                new LoginPage(driver)
                .enterUserName("admin")
                .enterPassword("admin123")
                .clickLogin();

        Assert.assertTrue(
                homePage.getWelcomeMessage()
                .contains("Welcome"));
    }
}

POM Without PageFactory (Recommended in Selenium 4)

Many modern frameworks avoid PageFactory and use By locators directly.

LoginPage.java
public class LoginPage {

    private WebDriver driver;

    private By txtUserName = By.id("username");
    private By txtPassword = By.id("password");
    private By btnLogin = By.id("loginBtn");

    public LoginPage(WebDriver driver) {
        this.driver = driver;
    }

    public LoginPage enterUserName(String username) {

        driver.findElement(txtUserName)
              .sendKeys(username);

        return this;
    }

    public LoginPage enterPassword(String password) {

        driver.findElement(txtPassword)
              .sendKeys(password);

        return this;
    }

    public HomePage clickLogin() {

        driver.findElement(btnLogin).click();

        return new HomePage(driver);
    }
}

Fluent POM Design

A cleaner enterprise approach:

new LoginPage(driver)
        .enterUserName("admin")
        .enterPassword("admin123")
        .clickLogin()
        .verifyUserLoggedIn();

POM with Component Objects

For reusable sections such as Header, Footer, Menu, Dashboard Widgets:

HomePage
 ├── HeaderComponent
 ├── FooterComponent
 └── MenuComponent


Example:

HomePage home = new HomePage(driver);

home.getHeader()
    .clickProfile();

home.getMenu()
    .navigateToOrders();

Typical Enterprise Framework Structure
src/test/java
│
├── base
│   ├── BasePage.java
│   └── BaseTest.java
│
├── pages
│   ├── LoginPage.java
│   ├── HomePage.java
│   ├── DashboardPage.java
│   └── OrdersPage.java
│
├── components
│   ├── HeaderComponent.java
│   └── MenuComponent.java
│
├── utils
│   ├── DriverFactory.java
│   ├── WaitUtils.java
│   ├── ScreenshotUtils.java
│   └── ConfigReader.java
│
├── listeners
│   └── TestListener.java
│
└── tests
    ├── LoginTests.java
    └── OrderTests.java

POM Best Practices

 ✅ One page = One class
 ✅ Hide locators inside page classes
 ✅ Keep assertions in test classes (or dedicated assertion classes)
 ✅ Create reusable BasePage methods
 ✅ Use explicit waits, avoid Thread.sleep()
 ✅ Use Fluent Page Objects for readability
 ✅ Separate Page Objects from Test Data
 ✅ Prefer By locators over PageFactory for Selenium 4 projects
 ✅ Use Component Objects for common page sections

Example Usage
LoginPage loginPage = new LoginPage(driver);

HomePage homePage = loginPage
                        .enterUserName("admin")
                        .enterPassword("admin123")
                        .clickLogin();

String message = homePage.getWelcomeMessage();

