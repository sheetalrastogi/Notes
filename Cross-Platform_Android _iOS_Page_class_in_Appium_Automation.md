# Cross-Platform Page Object Model in Appium (Android + iOS)

In Appium, you can create a **single Cross-Platform Page Object** that works for both Android and iOS using platform-specific annotations such as:

- `@AndroidFindBy`
- `@iOSXCUITFindBy`

The Appium PageFactory automatically selects the appropriate locator based on the active driver instance (`AndroidDriver` or `IOSDriver`).

---

# Base Page

The base page initializes Appium PageFactory with `AppiumFieldDecorator`, enabling support for both Android and iOS locators.

```java
package pages;

import io.appium.java_client.AppiumDriver;
import io.appium.java_client.pagefactory.AppiumFieldDecorator;
import org.openqa.selenium.support.PageFactory;

import java.time.Duration;

public class BasePage {

    protected AppiumDriver driver;

    public BasePage(AppiumDriver driver) {

        this.driver = driver;

        PageFactory.initElements(
                new AppiumFieldDecorator(
                        driver,
                        Duration.ofSeconds(10)
                ),
                this
        );
    }
}
```

---

# Cross-Platform Login Page

A single page object supports both Android and iOS.

```java
package pages;

import io.appium.java_client.AppiumDriver;
import io.appium.java_client.pagefactory.AndroidFindBy;
import io.appium.java_client.pagefactory.iOSXCUITFindBy;
import org.openqa.selenium.WebElement;

public class LoginPage extends BasePage {

    @AndroidFindBy(id = "username")
    @iOSXCUITFindBy(accessibility = "username")
    private WebElement txtUsername;

    @AndroidFindBy(id = "password")
    @iOSXCUITFindBy(accessibility = "password")
    private WebElement txtPassword;

    @AndroidFindBy(id = "loginBtn")
    @iOSXCUITFindBy(accessibility = "loginBtn")
    private WebElement btnLogin;

    public LoginPage(AppiumDriver driver) {
        super(driver);
    }

    public void enterUsername(String username) {

        txtUsername.clear();
        txtUsername.sendKeys(username);
    }

    public void enterPassword(String password) {

        txtPassword.clear();
        txtPassword.sendKeys(password);
    }

    public void clickLogin() {

        btnLogin.click();
    }

    public void login(String username, String password) {

        enterUsername(username);
        enterPassword(password);
        clickLogin();
    }
}
```

---

# Test Usage

The same page class works for both platforms.

```java
AppiumDriver driver;

if(platform.equalsIgnoreCase("android")) {

    driver = new AndroidDriver(
            serverUrl,
            options
    );
}
else {

    driver = new IOSDriver(
            serverUrl,
            options
    );
}

LoginPage loginPage = new LoginPage(driver);

loginPage.login(
        "admin",
        "admin123"
);
```

---

# Multiple Locators for the Same Platform

Sometimes Android UI implementations differ across devices or app versions.

## Example

```java
@AndroidFindBy(id = "username")
@AndroidFindBy(xpath = "//android.widget.EditText")
@iOSXCUITFindBy(accessibility = "username")
private WebElement txtUsername;
```

---

# Recommended Approach: AndroidFindAll

Use multiple fallback locators.

```java
@AndroidFindAll({
    @AndroidBy(id = "username"),
    @AndroidBy(xpath = "//android.widget.EditText")
})
private WebElement txtUsername;
```

This improves reliability when applications have device-dependent locator variations.

---

# Using WebElement vs MobileElement

Historically Appium provided `MobileElement`.

```java
private MobileElement btnLogin;
```

Today most modern frameworks use:

```java
private WebElement btnLogin;
```

or

```java
private RemoteWebElement btnLogin;
```

because Appium 2.x aligns closely with Selenium WebDriver standards.

---

# Platform-Specific Logic in the Same Page

Sometimes business actions require platform-specific implementation.

## Example: Hide Keyboard

```java
public void hideKeyboard() {

    if(driver.getPlatformName()
             .equalsIgnoreCase("Android")) {

        driver.hideKeyboard();
    }
    else {

        driver.executeScript(
            "mobile: hideKeyboard"
        );
    }
}
```

This allows sharing page objects while handling platform-specific behavior.

---

# Better Enterprise Design

A common Page Object Model hierarchy:

```text
BasePage
    |
    +-- LoginPage
    +-- HomePage
    +-- DashboardPage
```

---

# Enterprise Login Page Example

```java
public class LoginPage extends BasePage {

    @AndroidFindBy(id = "username")
    @iOSXCUITFindBy(accessibility = "username")
    private WebElement txtUsername;

    public LoginPage(AppiumDriver driver) {
        super(driver);
    }

    public HomePage login(
            String user,
            String pass) {

        enterUsername(user);
        enterPassword(pass);
        clickLogin();

        return new HomePage(driver);
    }
}
```

This pattern enables page chaining and improves test readability.

---

# Using FieldDecorator with PageFactory

For Android and iOS support together:

```java
PageFactory.initElements(
    new AppiumFieldDecorator(
            driver,
            Duration.ofSeconds(10)
    ),
    this
);
```

---

# Benefits of AppiumFieldDecorator

`AppiumFieldDecorator` automatically:

- Handles `@AndroidFindBy`
- Handles `@iOSXCUITFindBy`
- Creates lazy-loaded elements
- Supports implicit waiting during element discovery
- Works with both `AndroidDriver` and `IOSDriver`
- Integrates seamlessly with PageFactory

---

# Real-World Cross-Platform Home Page Example

```java
public class HomePage extends BasePage {

    @AndroidFindBy(accessibility = "menu")
    @iOSXCUITFindBy(accessibility = "menu")
    private WebElement menu;

    @AndroidFindBy(
        xpath = "//android.widget.TextView[@text='Settings']"
    )
    @iOSXCUITFindBy(
        xpath = "//XCUIElementTypeStaticText[@name='Settings']"
    )
    private WebElement settings;

    public HomePage(AppiumDriver driver) {
        super(driver);
    }

    public void openSettings() {

        menu.click();
        settings.click();
    }
}
```

---

# Best Practices

✅ Maintain a single page class for Android and iOS when possible

✅ Keep platform-specific locators together on the same element

✅ Place common functionality in `BasePage`

✅ Use `AppiumFieldDecorator` instead of standard Selenium `PageFactory`

✅ Return page objects from page actions for fluent design

✅ Use accessibility identifiers whenever possible

✅ Use platform-specific methods only when behavior truly differs

✅ Prefer `WebElement` for better compatibility with Selenium and Appium 2.x

✅ Use `@AndroidFindAll` and `@iOSXCUITFindAll` for fallback locators

---

# Cross-Platform POM Architecture

```text
                     BasePage
                         |
      -----------------------------------------
      |                   |                   |
  LoginPage          HomePage          DashboardPage
      |                   |                   |
 Android + iOS     Android + iOS      Android + iOS
   Locators           Locators          Locators
```

---

# Interview Points

### Can one Page Object support both Android and iOS?

**Yes.** A single Page Object can support both Android and iOS using:

```java
@AndroidFindBy(...)
@iOSXCUITFindBy(...)
```

on the same element.

---

### How does Appium choose the locator?

`AppiumFieldDecorator` detects the current driver type:

- `AndroidDriver` → uses `@AndroidFindBy`
- `IOSDriver` → uses `@iOSXCUITFindBy`

automatically.

---

### Why is this approach preferred?

- Reduces code duplication
- Improves maintainability
- Simplifies framework design
- Supports cross-platform test automation
- Keeps business logic in a single page class

This is the most commonly used Page Object Model design in enterprise Appium automation frameworks because it enables **one page class for Android and iOS**, minimizes duplication, and supports platform-specific locators wherever required.
