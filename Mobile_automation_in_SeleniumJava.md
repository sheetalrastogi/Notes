# Selenium Classes Used for Mobile Automation (Appium + Selenium Java)

For **Mobile Automation (Appium + Selenium Java)**, several Selenium classes and interfaces are commonly used because Appium drivers implement Selenium WebDriver APIs.

---

# Core Selenium Classes & Interfaces

## 1. WebDriver

Base interface for mobile drivers.

```java
AndroidDriver driver;
IOSDriver driver;
```

### Common Methods

```java
driver.getTitle();
driver.getCurrentUrl();
driver.quit();
```

---

## 2. WebElement

Represents mobile UI elements.

```java
WebElement loginBtn = driver.findElement(By.id("login"));
loginBtn.click();
```

### Common Methods

```java
click();
sendKeys();
getText();
isDisplayed();
isEnabled();
```

---

## 3. By

Locator strategy class.

```java
By.id("username");
By.xpath("//android.widget.TextView");
By.className("android.widget.Button");
```

### Examples

```java
driver.findElement(By.id("email"));
driver.findElement(By.xpath("//XCUIElementTypeButton"));
```

---

## 4. JavascriptExecutor

Used mostly for Mobile Web, Hybrid Apps, and browser automation.

```java
JavascriptExecutor js = (JavascriptExecutor) driver;

js.executeScript("arguments[0].click();", element);
```

---

## 5. TakesScreenshot

Captures screenshots.

```java
TakesScreenshot ts = (TakesScreenshot) driver;

File src = ts.getScreenshotAs(OutputType.FILE);
```

---

## 6. OutputType

Used with screenshots.

```java
driver.getScreenshotAs(OutputType.FILE);

driver.getScreenshotAs(OutputType.BYTES);

driver.getScreenshotAs(OutputType.BASE64);
```

---

## 7. Alert

Handles native or browser alerts.

```java
Alert alert = driver.switchTo().alert();

alert.accept();
alert.dismiss();
alert.getText();
```

---

## 8. Actions

Mostly useful for:

- Mobile Web
- Hybrid Apps
- W3C Gestures

```java
Actions actions = new Actions(driver);

actions.moveToElement(element)
       .click()
       .perform();
```

---

## 9. ActionBuilder & PointerInput (Selenium 4)

Used for modern Appium gestures.

```java
PointerInput finger =
    new PointerInput(PointerInput.Kind.TOUCH, "finger");
```

### Examples

- Swipe
- Drag & Drop
- Long Press
- Pinch
- Zoom

```java
Sequence swipe = new Sequence(finger, 1);

driver.perform(Arrays.asList(swipe));
```

---

## 10. WebDriverWait

Provides explicit wait support.

```java
WebDriverWait wait =
    new WebDriverWait(driver, Duration.ofSeconds(20));

wait.until(
    ExpectedConditions.visibilityOf(element)
);
```

---

## 11. ExpectedConditions

Common wait conditions:

```java
ExpectedConditions.visibilityOf(element);

ExpectedConditions.elementToBeClickable(element);

ExpectedConditions.presenceOfElementLocated(locator);
```

---

## 12. FluentWait

Advanced waiting mechanism.

```java
FluentWait<WebDriver> wait =
    new FluentWait<>(driver)
        .withTimeout(Duration.ofSeconds(30))
        .pollingEvery(Duration.ofSeconds(2));
```

---

## 13. PageFactory

Used in the Page Object Model.

```java
PageFactory.initElements(driver, this);
```

### With Appium

```java
PageFactory.initElements(
    new AppiumFieldDecorator(driver),
    this
);
```

---

## 14. SearchContext

Parent interface of both WebDriver and WebElement.

```java
SearchContext context = driver;

context.findElement(By.id("login"));
```

### Useful For

- Nested searches
- Shadow DOM concepts

---

## 15. Dimension

Represents device screen size.

```java
Dimension size = driver.manage()
                       .window()
                       .getSize();

int width = size.width;
int height = size.height;
```

Used in swipe calculations.

---

## 16. Point

Represents element location.

```java
Point location = element.getLocation();

int x = location.getX();
int y = location.getY();
```

---

## 17. Rectangle

Represents element position and size.

```java
Rectangle rect = element.getRect();

rect.getWidth();
rect.getHeight();
rect.getX();
rect.getY();
```

Useful for gesture automation.

---

## 18. RemoteWebDriver

Parent class of Appium drivers.

```java
RemoteWebDriver driver;
```

### Inheritance Hierarchy

```text
WebDriver
   |
RemoteWebDriver
   |
AppiumDriver
   |------ AndroidDriver
   |
   |------ IOSDriver
```

---

# Selenium 4 Classes Frequently Used with Appium 2.x

```text
WebDriver
WebElement
By
WebDriverWait
ExpectedConditions
FluentWait
Actions
PointerInput
Sequence
Pause
JavascriptExecutor
TakesScreenshot
OutputType
Dimension
Point
Rectangle
RemoteWebDriver
PageFactory
SearchContext
```

---

# Appium-Specific Classes Often Used Together

Although not Selenium classes, these are commonly used in most mobile automation frameworks:

```text
AppiumDriver
AndroidDriver
IOSDriver
AppiumBy
AppiumFieldDecorator
AndroidFindBy
iOSXCUITFindBy
MobileElement
UiAutomator2Options
XCUITestOptions
```

---

# Most Important Selenium Classes for Appium 2.x + Selenium 4

```text
WebDriver
WebElement
By
WebDriverWait
ExpectedConditions
Actions
PointerInput
Sequence
PageFactory
JavascriptExecutor
TakesScreenshot
Dimension
Point
Rectangle
RemoteWebDriver
```

---

## Summary

These Selenium classes form the foundation of most **Page Object Model (POM)-based Android and iOS automation frameworks** built using **Appium 2.x and Selenium 4**.
