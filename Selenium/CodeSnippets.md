# Given xPath of an element, check whether it exists in DOM

```java
		String xpath = "//input[@id='username']";
		Boolean exists = (Boolean) ((JavascriptExecutor) driver).executeScript(
				"return document.evaluate(arguments[0], document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue != null;",
				xpath);

		System.out.println("XPath Exists = " + exists);
```

