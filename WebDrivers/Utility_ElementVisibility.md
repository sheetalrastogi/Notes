
```java
		WebElement element = driver.findElement(By.id("submitBtn"));

		JavascriptExecutor js = (JavascriptExecutor) driver;

		String script = """
				const e = arguments[0];
				const s = window.getComputedStyle(e);

				return {
				    display : s.display,
				    visibility : s.visibility,
				    opacity : s.opacity,
				    width : e.offsetWidth,
				    height : e.offsetHeight,
				    inViewport :
				        e.getBoundingClientRect().top <
				        window.innerHeight
				};
				""";

		Map<String, Object> properties = (Map<String, Object>) js.executeScript(script, element);

		System.out.println("Display    : " + properties.get("display"));
		System.out.println("Visibility : " + properties.get("visibility"));
		System.out.println("Opacity    : " + properties.get("opacity"));
		System.out.println("Width      : " + properties.get("width"));
		System.out.println("Height     : " + properties.get("height"));
		System.out.println("InViewport : " + properties.get("inViewport"));
```

Usage:
```java
Map<String, Object> details = getElementVisibilityDetails(element);
System.out.println(details);
```

This is particularly **useful when debugging Selenium failures** such as:
 - ElementNotInteractableException
 - ElementClickInterceptedException
 - Hidden elements (display:none)
 - Transparent elements (opacity:0)
 - Off-screen elements
 - Elements with zero width/height
 - Disabled controls
 - Responsive UI rendering issues
