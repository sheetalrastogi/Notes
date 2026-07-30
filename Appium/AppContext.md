## Check Available Contexts
---

```java
Set<String> contexts = driver.getContextHandles();

System.out.println("Available Contexts:");

for(String context : contexts) {
    System.out.println(context);
}

```

Output:	
	Native App:	[NATIVE_APP]
	Hybrid App:	[NATIVE_APP]
			[WEBVIEW_com.mycompany.myapp]

## Determine App Type Programmatically

```java
Set<String> contexts = driver.getContextHandles();

boolean isHybrid = contexts.stream()
        .anyMatch(context -> context.contains("WEBVIEW"));

System.out.println(
        isHybrid
        ? "Hybrid Application"
        : "Native Application");
```


## Switch to WebView

```java
for (String context : driver.getContextHandles()) {

    if (context.contains("WEBVIEW")) {

        driver.context(context);

        break;
    }
}
```

**Return to Native Context**

```java
driver.context("NATIVE_APP");
```


## Java Utility

```java
public static void printContexts(AppiumDriver driver) {

    Set<String> contexts =
            driver.getContextHandles();

    System.out.println("Available Contexts:");

    contexts.forEach(System.out::println);

    if (contexts.stream()
                .anyMatch(c -> c.contains("WEBVIEW"))) {

        System.out.println("Hybrid App Detected");

    } else {

        System.out.println("Native App Detected");
    }
}
```
