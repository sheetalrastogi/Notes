# Give example of Page class, Test Data (Pojo based read from CSV data file) and Workflow Test class for designing a Page Object Model (POM) with Page Component Object Pattern

# Page Object Model (POM) using Page Component Object Pattern, where:

- **Page Class** → Represents pages and reusable page components
- **POJO Test Data Class** → Loaded from CSV file
- **Workflow Class** → Encapsulates business flow
- **Test Class** → Uses workflow and data objects only

# Project Structure

```text
src/test/resources
 └── testdata
      └── login.csv

src/main/java

pages
 ├── LoginPage.java
 ├── DashboardPage.java
 └── components
      └── HeaderComponent.java

data
 └── LoginData.java

utils
 ├── CsvReader.java
 └── DriverManager.java

workflows
 └── LoginWorkflow.java

tests
 └── LoginTest.java
```

# CSV Data File

login.csv

```text
username,password,expectedUser
admin,password123,Admin User
john,john123,John Smith
```

# Test Data

POJO Test Data LoginData

```java
package data;

public class LoginData {

    private String username;
    private String password;
    private String expectedUser;

    public LoginData() {
    }

    public LoginData(String username,
                     String password,
                     String expectedUser) {
        this.username = username;
        this.password = password;
        this.expectedUser = expectedUser;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }

    public String getExpectedUser() {
        return expectedUser;
    }

    @Override
    public String toString() {
        return username;
    }
}
```

# CSV Reader Utility

```xml
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>5.10</version>
</dependency>
```

**CsvReader**

```java
package utils;

import com.opencsv.CSVReader;
import data.LoginData;

import java.io.FileReader;
import java.util.ArrayList;
import java.util.List;

public class CsvReader {

	public static List<LoginData> getLoginData(String filePath) {

		List<LoginData> dataList = new ArrayList<>();

		try (CSVReader reader = new CSVReader(new FileReader(filePath))) {

			String[] row;

			reader.readNext(); // skip header

			while ((row = reader.readNext()) != null) {

				LoginData data = new LoginData(row[0], row[1], row[2]);

				dataList.add(data);
			}

		} catch (Exception e) {
			throw new RuntimeException(e);
		}

		return dataList;
	}
}

```



# Base Page

```java
package pages;

import org.openqa.selenium.WebDriver;

public abstract class BasePage {

    protected WebDriver driver;

    protected BasePage(WebDriver driver) {
        this.driver = driver;
    }
}
```

# Page Component

A reusable component present on multiple pages.

example:  HeaderComponent

```java
package pages.components;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class HeaderComponent {

	private WebDriver driver;

	private By loggedInUser = By.id("welcomeUser");

	public HeaderComponent(WebDriver driver) {
		this.driver = driver;
	}

	public String getLoggedInUser() {
		return driver.findElement(loggedInUser).getText();
	}
}

```


# Login Page

```java
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class LoginPage extends BasePage {

	private By txtUsername = By.id("username");
	private By txtPassword = By.id("password");
	private By btnLogin = By.id("loginBtn");

	public LoginPage(WebDriver driver) {
		super(driver);
	}

	public LoginPage enterUsername(String username) {
		driver.findElement(txtUsername).sendKeys(username);
		return this;
	}

	public LoginPage enterPassword(String password) {
		driver.findElement(txtPassword).sendKeys(password);
		return this;
	}

	public DashboardPage clickLogin() {
		driver.findElement(btnLogin).click();
		return new DashboardPage(driver);
	}
}

```

# Dashboard Page

Uses a Page Component.

```java
package pages;

import org.openqa.selenium.WebDriver;
import pages.components.HeaderComponent;

public class DashboardPage extends BasePage {

    private HeaderComponent header;

    public DashboardPage(WebDriver driver) {
        super(driver);

        header = new HeaderComponent(driver);
    }

    public HeaderComponent header() {
        return header;
    }
}
```

# Workflow Layer

Workflow contains complete business actions.

```java
package workflows;

import data.LoginData;
import pages.DashboardPage;
import pages.LoginPage;

public class LoginWorkflow {

	private LoginPage loginPage;

	public LoginWorkflow(LoginPage loginPage) {
		this.loginPage = loginPage;
	}

	public DashboardPage loginAs(LoginData data) {

		return loginPage.enterUsername(data.getUsername()).enterPassword(data.getPassword()).clickLogin();
	}
}

```

# TestNG DataProvider from CSV

```java
	@DataProvider(name = "loginData")
	public Object[][] loginData() {

		List<LoginData> dataList = CsvReader.getLoginData("src/test/resources/testdata/login.csv");

		Object[][] data = new Object[dataList.size()][1];

		for (int i = 0; i < dataList.size(); i++) {
			data[i][0] = dataList.get(i);
		}

		return data;
	}
```

# Workflow 
drives Test Class

```java
package tests;

import data.LoginData;
import org.testng.Assert;
import org.testng.annotations.Test;
import pages.DashboardPage;
import pages.LoginPage;
import workflows.LoginWorkflow;

public class LoginTest extends BaseTest {

	@Test(dataProvider = "loginData")
	public void verifyUserCanLogin(LoginData loginData) {

		LoginPage loginPage = new LoginPage(driver);

		DashboardPage dashboardPage = new LoginWorkflow(loginPage).loginAs(loginData);

		String actualUser = dashboardPage.header().getLoggedInUser();

		Assert.assertEquals(actualUser, loginData.getExpectedUser());
	}
}

```

# Test

```java

DashboardPage dashboardPage = new LoginWorkflow(loginPage).loginAs(loginData);
Assert.assertEquals(dashboardPage.header().getLoggedInUser(), loginData.getExpectedUser());

```




