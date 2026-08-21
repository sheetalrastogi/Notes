## 1. Given xPath of an element, check whether it exists in DOM

```java
		String xpath = "//input[@id='username']";
		Boolean exists = (Boolean) ((JavascriptExecutor) driver).executeScript(
				"return document.evaluate(arguments[0], document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue != null;",
				xpath);

		System.out.println("XPath Exists = " + exists);
```

## 2. Working with Tooltips
```xml
<button id="help">Help</button>

<div class="tooltip">
    Click here for assistance
</div>
```

```java
	WebElement helpButton = driver.findElement(By.id("help"));
	Actions actions = new Actions(driver);
	actions.moveToElement(helpButton).perform();
	WebElement tooltip = driver.findElement(By.className("tooltip"));
	System.out.println("Tooltip Text: " + tooltip.getText());
```

