To extend PageFactory.initElements() for **custom controls** such as TextBox, Button, Dropdown, Checkbox, etc., you need to implement a **Custom FieldDecorator**.

This is commonly used in enterprise frameworks to wrap Selenium WebElement with reusable business methods.

## Goal

**Instead of**:
```java
@FindBy(id = "username")
private WebElement txtUsername;

txtUsername.sendKeys("admin");
```

**You can write**:
```java
@FindBy(id = "username")
private TextBox txtUsername;

txtUsername.enterText("admin");
```
or
```java
@FindBy(id = "country")
private DropDown ddlCountry;

ddlCountry.selectByVisibleText("India");
```
# Framework Structure
```text
framework
│
├── controls
│   ├── TextBox.java
│   ├── DropDown.java
│   └── Button.java
│
├── decorators
│   └── CustomFieldDecorator.java
│
├── pagefactory
│   └── ExtendedElementLocatorFactory.java
│
└── pages
    └── LoginPage.java
```

# Step 1: Create Custom TextBox
TextBox.java
```java
package controls;

import org.openqa.selenium.WebElement;

public class TextBox {

    private WebElement element;

    public TextBox(WebElement element) {
        this.element = element;
    }

    public void enterText(String value) {

        element.clear();
        element.sendKeys(value);
    }

    public String getValue() {

        return element.getAttribute("value");
    }

    public boolean isDisplayed() {

        return element.isDisplayed();
    }
}
```

# Step 2: Create Custom Dropdown
DropDown.java
```java
package controls;

import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.Select;

public class DropDown {

    private Select select;

    public DropDown(WebElement element) {

        this.select = new Select(element);
    }

    public void selectByText(String text) {

        select.selectByVisibleText(text);
    }

    public void selectByValue(String value) {

        select.selectByValue(value);
    }

    public String getSelectedText() {

        return select.getFirstSelectedOption()
                     .getText();
    }
}
```

# Step 3: Custom FieldDecorator

Extend Selenium's DefaultFieldDecorator.

CustomFieldDecorator.java
```java
package decorators;

import controls.DropDown;
import controls.TextBox;

import java.lang.reflect.Field;

import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.pagefactory.ElementLocator;
import org.openqa.selenium.support.pagefactory.FieldDecorator;
import org.openqa.selenium.support.pagefactory.DefaultFieldDecorator;

public class CustomFieldDecorator
        extends DefaultFieldDecorator {

    public CustomFieldDecorator(
            org.openqa.selenium.support.pagefactory.ElementLocatorFactory factory) {

        super(factory);
    }

    @Override
    public Object decorate(
            ClassLoader loader,
            Field field) {

        if (field.getType().equals(TextBox.class)) {

            ElementLocator locator =
                    factory.createLocator(field);

            WebElement proxyElement =
                    proxyForLocator(loader, locator);

            return new TextBox(proxyElement);
        }

        if (field.getType().equals(DropDown.class)) {

            ElementLocator locator =
                    factory.createLocator(field);

            WebElement proxyElement =
                    proxyForLocator(loader, locator);

            return new DropDown(proxyElement);
        }

        return super.decorate(loader, field);
    }
}
```

# Step 4: Initialize Custom Controls

Instead of:
```java
PageFactory.initElements(driver,this);
```

Use:
```java
PageFactory.initElements(
        new CustomFieldDecorator(
                new DefaultElementLocatorFactory(driver)),
        this);
```

# Step 5: Use Custom Elements in Page Class
LoginPage.java
```java
public class LoginPage {

    public LoginPage(WebDriver driver) {

        PageFactory.initElements(
                new CustomFieldDecorator(
                        new DefaultElementLocatorFactory(driver)),
                this);
    }

    @FindBy(id = "username")
    private TextBox txtUsername;

    @FindBy(id = "password")
    private TextBox txtPassword;

    @FindBy(id = "country")
    private DropDown ddlCountry;

    public void login() {

        txtUsername.enterText("admin");
        txtPassword.enterText("admin123");

        ddlCountry.selectByText("India");
    }
}
```
# Advanced Generic Control Base Class

Most enterprise frameworks create a common base wrapper.

UIElement.java
```java
public abstract class UIElement {

    protected WebElement element;

    public UIElement(WebElement element) {

        this.element = element;
    }

    public void click() {

        element.click();
    }

    public boolean isDisplayed() {

        return element.isDisplayed();
    }

    public String getText() {

        return element.getText();
    }
}
```

TextBox.java
```java
public class TextBox extends UIElement {

    public TextBox(WebElement element) {

        super(element);
    }

    public void enterText(String value) {

        element.clear();
        element.sendKeys(value);
    }
}
```

Button.java
```java
public class Button extends UIElement {

    public Button(WebElement element) {

        super(element);
    }

    public void submit() {

        element.click();
    }
}
```

DropDown.java
```java
public class DropDown extends UIElement {

    private Select select;

    public DropDown(WebElement element) {

        super(element);
        this.select = new Select(element);
    }

    public void selectByText(String text) {

        select.selectByVisibleText(text);
    }
}
```

# Alternative: Custom Element Factory with AjaxElementLocatorFactory

**For lazy loading**:

```java
PageFactory.initElements(
     new CustomFieldDecorator(
          new AjaxElementLocatorFactory(driver, 10)),
     this);
```

This combines:

- Custom controls
- Lazy loading
- Automatic waits

Example:
```java
PageFactory.initElements(
   new CustomFieldDecorator(
      new AjaxElementLocatorFactory(driver,10)),
   this);
```

Enterprise Example
```java
@FindBy(id = "username")
private TextBox username;

@FindBy(id = "password")
private TextBox password;

@FindBy(id = "country")
private DropDown country;

@FindBy(id = "login")
private Button loginBtn;
```

Usage:
```java
loginPage.username.enterText("admin");
loginPage.password.enterText("admin123");
loginPage.country.selectByText("India");
loginPage.loginBtn.click();
```

This pattern is the same approach used by frameworks such as AppiumFieldDecorator, where Selenium/Appium extends PageFactory and injects custom wrappers around WebElement instances through a custom FieldDecorator. It is the cleanest way to build reusable enterprise UI controls in Selenium Java POM frameworks.
