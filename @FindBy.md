# Selenium Page Object Model: @FindBy vs @FindBy(how = How.XYZ, using = "...")

In Selenium Page Object Model (POM), both `@FindBy` and `@FindBy(how = How.XYZ, using = "...")` are used to identify web elements through PageFactory.

---

# 1. Using `@FindBy`

This is the modern and most commonly used approach.

## Page Class

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

public class LoginPage {

    private WebDriver driver;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    @FindBy(id = "username")
    private WebElement username;

    @FindBy(id = "password")
    private WebElement password;

    @FindBy(xpath = "//button[text()='Login']")
    private WebElement loginButton;

    public void login(String user, String pass) {
        username.sendKeys(user);
        password.sendKeys(pass);
        loginButton.click();
    }
}
```

## Usage

```java
LoginPage loginPage = new LoginPage(driver);

loginPage.login("admin", "admin123");
```

---

# 2. Using `@FindBy(how = How.XYZ, using = "...")`

This is an older but still supported style.

## Page Class

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.How;
import org.openqa.selenium.support.PageFactory;

public class LoginPage {

    public LoginPage(WebDriver driver) {
        PageFactory.initElements(driver, this);
    }

    @FindBy(how = How.ID, using = "username")
    private WebElement username;

    @FindBy(how = How.ID, using = "password")
    private WebElement password;

    @FindBy(how = How.XPATH,
            using = "//button[text()='Login']")
    private WebElement loginButton;

    public void login(String user, String pass) {
        username.sendKeys(user);
        password.sendKeys(pass);
        loginButton.click();
    }
}
```

---

# 3. Comparison

## ID Locator

### Modern Style

```java
@FindBy(id = "username")
WebElement username;
```

### Equivalent Legacy Style

```java
@FindBy(how = How.ID,
        using = "username")
WebElement username;
```

---

## XPath Locator

### Modern Style

```java
@FindBy(xpath = "//input[@type='text']")
WebElement txtUser;
```

### Equivalent Legacy Style

```java
@FindBy(how = How.XPATH,
        using = "//input[@type='text']")
WebElement txtUser;
```

---

# 4. Common Locator Examples

## ID

```java
@FindBy(id = "userName")
WebElement userName;
```

---

## Name

```java
@FindBy(name = "password")
WebElement password;
```

---

## Class Name

```java
@FindBy(className = "login-btn")
WebElement loginBtn;
```

---

## Tag Name

```java
@FindBy(tagName = "a")
WebElement link;
```

---

## CSS Selector

```java
@FindBy(css = "input[type='submit']")
WebElement submitBtn;
```

---

## Link Text

```java
@FindBy(linkText = "Home")
WebElement homeLink;
```

---

## Partial Link Text

```java
@FindBy(partialLinkText = "Forgot")
WebElement forgotPwd;
```

---

## XPath

```java
@FindBy(xpath = "//button[@type='submit']")
WebElement loginButton;
```

---

# 5. Using `@CacheLookup`

Use `@CacheLookup` for static elements that do not refresh.

```java
@FindBy(id = "username")
@CacheLookup
private WebElement username;
```

## Benefits

- Improves performance.
- Reduces repeated element lookups.

## Warning

⚠️ Avoid using `@CacheLookup` for dynamic elements because it can cause:

```java
StaleElementReferenceException
```

Common examples of dynamic elements:

- AJAX-loaded controls
- Refreshing tables
- Dynamic menus
- React/Angular components

---

# 6. Multiple Elements (`List<WebElement>`)

When multiple elements match the locator, use a collection.

```java
@FindBy(tagName = "a")
private List<WebElement> links;
```

## Usage

```java
for (WebElement link : links) {
    System.out.println(link.getText());
}
```

---

# 7. Complete Enterprise-Style Page Class

```java
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

    @FindBy(css = "button[type='submit']")
    private WebElement btnLogin;

    @FindBy(className = "error-message")
    private WebElement errorMessage;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    public LoginPage enterUsername(String user) {

        txtUsername.clear();
        txtUsername.sendKeys(user);

        return this;
    }

    public LoginPage enterPassword(String pass) {

        txtPassword.clear();
        txtPassword.sendKeys(pass);

        return this;
    }

    public HomePage clickLogin() {

        btnLogin.click();

        return new HomePage(driver);
    }

    public String getErrorMessage() {

        return errorMessage.getText();
    }
}
```

---

# Test Example

```java
new LoginPage(driver)
        .enterUsername("admin")
        .enterPassword("admin123")
        .clickLogin();
```

This follows the **Fluent Page Object Pattern**, making test code more readable and maintainable.

---

# Supported `How` Locator Strategies

```java
How.ID
How.NAME
How.CLASS_NAME
How.TAG_NAME
How.CSS
How.XPATH
How.LINK_TEXT
How.PARTIAL_LINK_TEXT
```

Example:

```java
@FindBy(how = How.CSS,
        using = "button.login")
private WebElement loginButton;
```

---

# Best Practices

✅ Prefer `@FindBy(id = "...")` over `@FindBy(how = How.ID, using = "...")`

✅ Keep locators private inside page classes

✅ Expose page actions instead of WebElements

✅ Use meaningful element names

✅ Use CSS selectors whenever possible for better performance

✅ Apply Fluent APIs for readability

✅ Use explicit waits instead of thread sleeps

✅ Avoid `@CacheLookup` for dynamic pages

---

# Interview Point

The following two declarations are functionally identical:

```java
@FindBy(id = "username")
private WebElement username;
```

```java
@FindBy(how = How.ID,
        using = "username")
private WebElement username;
```

The first version is simply a shorthand for the second.

### Current Industry Practice

Most modern Selenium frameworks prefer the cleaner syntax:

```java
@FindBy(id = "username")
```

while `How`-based declarations are primarily found in older or legacy automation frameworks.

**Combining @FindBy**
# Combining Multiple Locators in Appium for the Same Platform

When an Appium application supports multiple versions, device variants, or UI implementations, a single locator may not always be sufficient.

To combine multiple locators for the **same platform**, Appium provides:

- `@AndroidFindAll` → **OR Logic** (find using any locator)
- `@AndroidFindBys` → **AND Logic / Chained Lookup**
- `@HowToUseLocators(androidAutomation = LocatorGroupStrategy.ALL_POSSIBLE)` → **Recommended in Appium 8+**

---

# Option 1: Using `@AndroidFindAll` (OR Logic)

Appium tries all locators and uses the first matching element.

## Example

```java
@AndroidFindAll({
    @AndroidBy(id = "username"),
    @AndroidBy(xpath = "//android.widget.EditText")
})
@iOSXCUITFindBy(accessibility = "username")
private WebElement txtUsername;
```

### Equivalent Logic

```text
id = username

OR

xpath = //android.widget.EditText
```

### Use Cases

✅ App Version A uses:

```text
id = username
```

✅ App Version B uses different attributes

✅ Need fallback locator strategy

✅ Support multiple device variations

---

# Option 2: Using `@HowToUseLocators` (Recommended)

This is the preferred approach in modern Appium frameworks.

## Example

```java
@HowToUseLocators(
    androidAutomation = LocatorGroupStrategy.ALL_POSSIBLE
)
@AndroidFindBy(id = "username")
@AndroidFindBy(xpath = "//android.widget.EditText")
@iOSXCUITFindBy(accessibility = "username")
private WebElement txtUsername;
```

---

## How It Works

Appium attempts:

```text
1. id = username

2. xpath = //android.widget.EditText
```

and returns the first successful match.

---

## Benefits

✅ Cleaner syntax

✅ Easier maintenance

✅ Better readability

✅ Preferred for Appium 8+

✅ Reduces the need for nested locator annotations

---

# Option 3: Using `@AndroidFindBys` (AND / Chained Lookup)

Use this when one locator must be searched inside another locator.

## Example

```java
@AndroidFindBys({
    @AndroidBy(id = "loginForm"),
    @AndroidBy(id = "username")
})
private WebElement txtUsername;
```

---

## Equivalent Selenium Code

```java
driver.findElement(By.id("loginForm"))
      .findElement(By.id("username"));
```

---

## Search Flow

```text
loginForm
   |
   +--> username
```

---

## Use Cases

✅ Nested controls

✅ Forms inside containers

✅ Tables

✅ Complex layouts

✅ Parent-child relationships

---

# Cross-Platform Example

A single page object can support Android and iOS while maintaining multiple Android locator options.

```java
@HowToUseLocators(
    androidAutomation = LocatorGroupStrategy.ALL_POSSIBLE
)
@AndroidFindBy(id = "username")
@AndroidFindBy(accessibility = "usernameField")
@AndroidFindBy(
    xpath = "//android.widget.EditText[@content-desc='username']"
)
@iOSXCUITFindBy(accessibility = "username")
private WebElement txtUsername;
```

---

## Android Search Order

Appium attempts:

```text
id = username

OR

accessibility = usernameField

OR

xpath = //android.widget.EditText[@content-desc='username']
```

---

## iOS Search

Only this locator is used:

```text
accessibility = username
```

---

# Multiple Locators for Both Android and iOS

Both platforms can have fallback locator chains.

## Example

```java
@HowToUseLocators(
    androidAutomation = LocatorGroupStrategy.ALL_POSSIBLE,
    iOSXCUITAutomation = LocatorGroupStrategy.ALL_POSSIBLE
)
@AndroidFindBy(id = "username")
@AndroidFindBy(xpath = "//android.widget.EditText")

@iOSXCUITFindBy(accessibility = "username")
@iOSXCUITFindBy(xpath = "//XCUIElementTypeTextField")

private WebElement txtUsername;
```

---

# Android Search Order

```text
id = username

OR

xpath = //android.widget.EditText
```

---

# iOS Search Order

```text
accessibility = username

OR

xpath = //XCUIElementTypeTextField
```

---

# Enterprise Best Practice

Prefer:

```java
@HowToUseLocators(
    androidAutomation = LocatorGroupStrategy.ALL_POSSIBLE
)
@AndroidFindBy(id = "username")
@AndroidFindBy(accessibility = "usernameField")
private WebElement txtUsername;
```

instead of relying heavily on XPath fallbacks.

---

# Recommended Locator Priority

## Android

```text
AccessibilityId
        ↓
Id
        ↓
UIAutomator
        ↓
Class Name
        ↓
XPath
```

---

## iOS

```text
AccessibilityId
        ↓
Id / Name
        ↓
iOS Class Chain
        ↓
iOS Predicate String
        ↓
XPath
```

---

# Why Avoid Excessive XPath Usage?

### Accessibility ID

✅ Fastest

✅ Stable

✅ Cross-platform

```java
@AndroidFindBy(accessibility = "username")
```

---

### ID

✅ Fast

✅ Readable

✅ Less brittle

```java
@AndroidFindBy(id = "username")
```

---

### XPath

❌ Slower

❌ More brittle

❌ Sensitive to UI hierarchy changes

```java
@AndroidFindBy(
    xpath = "//android.widget.EditText[@content-desc='username']"
)
```

---

# Comparison of Locator Grouping Approaches

## OR Logic

```java
@AndroidFindAll({
    @AndroidBy(id = "username"),
    @AndroidBy(xpath = "//android.widget.EditText")
})
```

Behavior:

```text
Find element using ANY matching locator.
```

---

## ALL_POSSIBLE Strategy

```java
@HowToUseLocators(
    androidAutomation = LocatorGroupStrategy.ALL_POSSIBLE
)
```

Behavior:

```text
Try all defined locators until one succeeds.
```

---

## AND / Chained Logic

```java
@AndroidFindBys({
    @AndroidBy(id = "form"),
    @AndroidBy(id = "username")
})
```

Behavior:

```text
Find username inside form.
```

---

# Interview Points

### What is the difference between `@AndroidFindAll` and `@AndroidFindBys`?

**`@AndroidFindAll`**

```text
OR Logic
```

Any locator can match.

---

**`@AndroidFindBys`**

```text
AND / Chained Logic
```

Each locator is applied sequentially.

---

### What is the preferred approach in Appium 8+?

```java
@HowToUseLocators(
    androidAutomation = LocatorGroupStrategy.ALL_POSSIBLE
)
```

because it provides cleaner and more maintainable code.

---

### What locator strategy should be prioritized?

```text
AccessibilityId
↓
Id
↓
Class Chain (iOS)
↓
UIAutomator (Android)
↓
XPath
```

The most common enterprise Appium frameworks use **`@HowToUseLocators` with `ALL_POSSIBLE`** along with **Accessibility ID and ID-based locators** to create robust, maintainable, and high-performing cross-platform mobile automation frameworks.
