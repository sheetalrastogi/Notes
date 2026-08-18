# Give example of Page class, Test Data (Pojo based read from json) and Workflow Test class for designing a Page Object Model (POM) without PageFactory

A **POM without PageFactory** typically uses By locators directly and avoids @FindBy annotations. 

## Project Structure

```text
src/test/java
|
├── pages
│   ├── LoginPage.java
│   └── DashboardPage.java
│
├── workflow
│   └── LoginWorkflow.java
│
├── model
│   └── LoginData.java
│
├── utils
│   └── JsonDataReader.java
│
├── testdata
│   └── loginData.json
│
└── tests
    └── LoginTest.java
```


# 1. Test Data JSON

loginData.json

```json
{
  "username": "admin",
  "password": "Password123",
  "rememberMe": true
}
```


# 2. LoginData POJO

```java
package model;

public class LoginData {

	private String username;
	private String password;
	private boolean rememberMe;

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


# 3. JSON Reader

Using Jackson.

```java
package utils;

import com.fasterxml.jackson.databind.ObjectMapper;
import model.LoginData;

import java.io.File;

public class JsonDataReader {

    public static LoginData getLoginData() {

        try {

            ObjectMapper mapper = new ObjectMapper();
            return mapper.readValue(new File("src/test/resources/testdata/loginData.json"), LoginData.class);
        } catch (Exception e) {
            throw new RuntimeException("Unable to load test data", e);
        }
    }
}

```

# 4. Login Page (Without PageFactory)


```java
package pages;

import model.LoginData;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class LoginPage {

	private WebDriver driver;

	private By txtUsername = By.id("username");

	private By txtPassword = By.id("password");

	private By chkRememberMe = By.id("rememberMe");

	private By btnLogin = By.id("loginBtn");

	public LoginPage(WebDriver driver) {

		this.driver = driver;
	}

	public LoginPage enterUsername(String username) {
		driver.findElement(txtUsername).clear();
		driver.findElement(txtUsername).sendKeys(username);
		return this;
	}

	public LoginPage enterPassword(String password) {
		driver.findElement(txtPassword).clear();
		driver.findElement(txtPassword).sendKeys(password);
		return this;
	}

	public LoginPage selectRememberMe() {
		if (!driver.findElement(chkRememberMe).isSelected()) {
			driver.findElement(chkRememberMe).click();
		}
		return this;
	}

	public DashboardPage login(LoginData data) {
		enterUsername(data.getUsername());
		enterPassword(data.getPassword());
		if (data.isRememberMe()) {
			selectRememberMe();
		}

		driver.findElement(btnLogin).click();

		return new DashboardPage(driver);
	}
}

```

# 5. Dashboard Page

```java
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class DashboardPage {

	private WebDriver driver;

	private By lblWelcome = By.id("welcomeText");

	public DashboardPage(WebDriver driver) {

		this.driver = driver;
	}

	public String getWelcomeMessage() {

		return driver.findElement(lblWelcome).getText();
	}
}
```

# 6. Workflow Class

Business flow layer.

```java
package workflow;

import model.LoginData;
import pages.DashboardPage;
import pages.LoginPage;

public class LoginWorkflow {

    private final LoginPage loginPage;

    public LoginWorkflow(LoginPage loginPage) {

        this.loginPage = loginPage;
    }

    public DashboardPage loginAs(LoginData loginData) {
        return loginPage.login(loginData);
    }
}

```

# 7. Test Class

```java
package tests;

import model.LoginData;
import org.testng.Assert;
import org.testng.annotations.Test;
import pages.DashboardPage;
import pages.LoginPage;
import utils.JsonDataReader;
import workflow.LoginWorkflow;

public class LoginTest extends BaseTest {

	@Test
	public void verifyLogin() {

		LoginData loginData = JsonDataReader.getLoginData();

		DashboardPage dashboardPage = new LoginWorkflow(new LoginPage(driver)).loginAs(loginData);

		Assert.assertTrue(dashboardPage.getWelcomeMessage().contains("Welcome"));
	}
}

```

