# Give example of Page class, Test Data (Pojo based) and Workflow Test class for designing a Page Object Model (POM) with PageFactory



# A clean POM design generally separates:

- Page Class → Page elements and actions.
- Test Data POJO → Strongly typed test data model.
- Workflow/Test Class → Business flow orchestration.


```text
LoginData - POJO
LoginPage - Element locators and Interactions
LoginDataFactory - Generates Test Data
LoginWorkflow - steps to call LoginPage method (abstraction)

Test:  	retrieves WebDriver
	Instantiates LoginPage with (driver)
	Creates Object of LoginDataFactory
		Creates Object of LoginWorkflow
			Calls (loginWorkflow(with loginDataFactory object))
```

# 1. Login Test Data POJO

```java
package model;

public class LoginData {

	private String username;
	private String password;
	private boolean rememberMe;

	public LoginData(String username, String password, boolean rememberMe) {
		this.username = username;
		this.password = password;
		this.rememberMe = rememberMe;
	}

	public String getUsername() {
		return username;
	}

	public String getPassword() {
		return password;
	}

	public boolean isRememberMe() {
		return rememberMe;
	}

	@Override
	public String toString() {
		return "LoginData{" + "username='" + username + '\'' + ", rememberMe=" + rememberMe + '}';
	}
}

```

# 2. Login Page (PageFactory)

Contains page locators and page actions.

```java
package pages;

import model.LoginData;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

public class LoginPage {

	private WebDriver driver;

	@FindBy(id = "username")
	private WebElement txtUsername;

	@FindBy(id = "password")
	private WebElement txtPassword;

	@FindBy(id = "rememberMe")
	private WebElement chkRememberMe;

	@FindBy(id = "loginBtn")
	private WebElement btnLogin;

	public LoginPage(WebDriver driver) {
		this.driver = driver;
		PageFactory.initElements(driver, this);
	}

	public LoginPage enterUsername(String username) {
		txtUsername.clear();
		txtUsername.sendKeys(username);
		return this;
	}

	public LoginPage enterPassword(String password) {
		txtPassword.clear();
		txtPassword.sendKeys(password);
		return this;
	}

	public LoginPage selectRememberMe() {
		if (!chkRememberMe.isSelected()) {
			chkRememberMe.click();
		}
		return this;
	}

	public DashboardPage login(LoginData data) {

		enterUsername(data.getUsername());
		enterPassword(data.getPassword());

		if (data.isRememberMe()) {
			selectRememberMe();
		}

		btnLogin.click();

		return new DashboardPage(driver);
	}
}

```


# 3. Dashboard Page

```java
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

public class DashboardPage {

	@FindBy(id = "welcomeText")
	private WebElement lblWelcome;

	public DashboardPage(WebDriver driver) {
		PageFactory.initElements(driver, this);
	}

	public String getWelcomeMessage() {
		return lblWelcome.getText();
	}
}

```


# 4. Test Data Builder

Useful when test data grows.

```java
package testdata;

import model.LoginData;

public class LoginDataFactory {

	public static LoginData validUser() {
		return new LoginData("admin", "Password123", true);
	}

	public static LoginData invalidUser() {
		return new LoginData("invalidUser", "invalidPwd", false);
	}
}

```


# 5. Workflow Class

Encapsulates business workflow.

```java
package workflows;

import model.LoginData;
import pages.DashboardPage;
import pages.LoginPage;

public class LoginWorkflow {

	private LoginPage loginPage;

	public LoginWorkflow(LoginPage loginPage) {
		this.loginPage = loginPage;
	}

	public DashboardPage loginAs(LoginData data) {

		return loginPage.login(data);
	}
}

```


# 6. Test Class

The test remains very clean.

```java
package tests;

import model.LoginData;
import org.testng.Assert;
import org.testng.annotations.Test;
import pages.DashboardPage;
import pages.LoginPage;
import testdata.LoginDataFactory;
import workflows.LoginWorkflow;

public class LoginTest extends BaseTest {

	@Test
	public void verifyUserCanLogin() {

		LoginData loginData = LoginDataFactory.validUser();

		LoginPage loginPage = new LoginPage(driver);

		DashboardPage dashboardPage = new LoginWorkflow(loginPage).loginAs(loginData);

		Assert.assertTrue(dashboardPage.getWelcomeMessage().contains("Welcome"));
	}
}

```


# Execution Flow

```text
LoginTest
    |
    V
LoginDataFactory
    |
    V
LoginData (POJO)
    |
    V
LoginWorkflow
    |
    V
LoginPage (PageFactory)
    |
    V
DashboardPage
    |
    V
Assertion
```

Benefits
-----------

- Page Object contains only UI interactions.
- POJO encapsulates test data.
- Workflow encapsulates business process.
- Test class becomes highly readable.
- Easy to integrate with DataProvider, Faker, JSON, Excel, DB, or API driven test data.
- Supports enterprise-scale Selenium 4 Java frameworks.

