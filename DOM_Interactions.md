# Interacting with DOM Elements using JavascriptExecutor in Selenium Java

## Why use JavascriptExecutor?

`JavascriptExecutor` is useful when:

- Selenium WebDriver cannot interact with an element directly.
- Elements are hidden, disabled, or overlapped.
- You need to access DOM properties/attributes.
- You want to execute custom JavaScript in the browser.

---

## 1. Click an Element using JavaScript

```java
WebElement btnLogin = driver.findElement(By.id("loginBtn"));

JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("arguments[0].click();", btnLogin);
```

---

## 2. Enter Text into a Textbox

```java
WebElement txtUsername = driver.findElement(By.id("username"));

JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("arguments[0].value='john.doe';", txtUsername);
```

---

## 3. Read Text from DOM Element

### HTML

```html
<span id="message">Login Successful</span>
```

### Selenium

```java
WebElement message = driver.findElement(By.id("message"));

JavascriptExecutor js = (JavascriptExecutor) driver;
String text = (String) js.executeScript(
    "return arguments[0].textContent;",
    message
);

System.out.println(text);
```

---

## 4. Read Attribute Value

```java
WebElement txtEmail = driver.findElement(By.id("email"));

JavascriptExecutor js = (JavascriptExecutor) driver;
String value = (String) js.executeScript(
    "return arguments[0].getAttribute('value');",
    txtEmail
);

System.out.println(value);
```

---

## 5. Highlight an Element

Useful for debugging.

```java
WebElement element = driver.findElement(By.id("username"));

JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript(
    "arguments[0].style.border='3px solid red'",
    element
);
```

---

## 6. Scroll Element into View

```java
WebElement element = driver.findElement(By.id("submitBtn"));

JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript(
    "arguments[0].scrollIntoView(true);",
    element
);
```

---

## 7. Scroll to Bottom of Page

```java
JavascriptExecutor js = (JavascriptExecutor) driver;

js.executeScript(
    "window.scrollTo(0, document.body.scrollHeight)"
);
```

---

## 8. Read Hidden DOM Element

### HTML

```html
<input type="hidden" id="token" value="ABC123XYZ">
```

### Selenium

```java
WebElement token = driver.findElement(By.id("token"));

JavascriptExecutor js = (JavascriptExecutor) driver;
String value = (String) js.executeScript(
    "return arguments[0].value;",
    token
);

System.out.println(value);
```

---

## 9. Change DOM Attribute

```java
WebElement textbox = driver.findElement(By.id("username"));

JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript(
    "arguments[0].setAttribute('value','Admin')",
    textbox
);
```

---

## 10. Remove Attribute

Example: Remove `readonly`

```java
WebElement dateField = driver.findElement(By.id("startDate"));

JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript(
    "arguments[0].removeAttribute('readonly');",
    dateField
);
```

---

## 11. Check if Element is Visible in DOM

```java
WebElement element = driver.findElement(By.id("userProfile"));

JavascriptExecutor js = (JavascriptExecutor) driver;

Boolean visible = (Boolean) js.executeScript(
    "return arguments[0].offsetParent !== null;",
    element
);

System.out.println(visible);
```

---

## 12. Access DOM Using Locator Instead of WebElement

### By ID

```java
JavascriptExecutor js = (JavascriptExecutor) driver;

String value = (String) js.executeScript(
    "return document.getElementById('username').value;"
);
```

### By CSS Selector

```java
String text = (String) js.executeScript(
    "return document.querySelector('.welcome-msg').textContent;"
);
```

### By XPath

```java
String text = (String) js.executeScript(
    "return document.evaluate(\"//span[@id='message']\", document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue.textContent;"
);
```

---

## 13. Trigger DOM Event

### Change Event

```java
WebElement element = driver.findElement(By.id("country"));

JavascriptExecutor js = (JavascriptExecutor) driver;

js.executeScript(
    "arguments[0].dispatchEvent(new Event('change'))",
    element
);
```

### Blur Event

```java
js.executeScript(
    "arguments[0].dispatchEvent(new Event('blur'))",
    element
);
```

---

## 14. Get Complete Inner HTML

```java
WebElement div = driver.findElement(By.id("content"));

JavascriptExecutor js = (JavascriptExecutor) driver;

String html = (String) js.executeScript(
    "return arguments[0].innerHTML;",
    div
);

System.out.println(html);
```

---

## 15. Access Shadow DOM Element

```java
JavascriptExecutor js = (JavascriptExecutor) driver;

WebElement shadowElement = (WebElement) js.executeScript(
    "return document.querySelector('my-component')" +
    ".shadowRoot.querySelector('#username');"
);

shadowElement.click();
```

---

# Generic DOM Interaction Utility

```java
public class JavaScriptUtil {

    private final JavascriptExecutor js;

    public JavaScriptUtil(WebDriver driver) {
        this.js = (JavascriptExecutor) driver;
    }

    public void click(WebElement element) {
        js.executeScript("arguments[0].click();", element);
    }

    public void setValue(WebElement element, String value) {
        js.executeScript(
            "arguments[0].value=arguments[1];",
            element,
            value
        );
    }

    public String getText(WebElement element) {
        return (String) js.executeScript(
            "return arguments[0].textContent;",
            element
        );
    }

    public void scrollIntoView(WebElement element) {
        js.executeScript(
            "arguments[0].scrollIntoView(true);",
            element
        );
    }
}
```

## Usage

```java
JavaScriptUtil jsUtil = new JavaScriptUtil(driver);

jsUtil.click(loginButton);
jsUtil.setValue(username, "admin");
jsUtil.scrollIntoView(submitButton);
```
