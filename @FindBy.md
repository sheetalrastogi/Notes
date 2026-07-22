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
