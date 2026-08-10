## Re-trigger AJAX Call on WebPage for dependent States dropdown interaction

Scenario:  If the Country dropdown selection triggers an Ajax request and occasionally the request is delayed or the UI event is not fully processed, 

Following is a robust Selenium approach:

 - Select the country.
 - Wait for Ajax/network activity to complete.
 - Trigger a change event again if required.
 - Wait until the States dropdown becomes visible and populated.
 - Retry once if the States dropdown does not appear.

Selenium Java 4 Solution

```java
import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.*;

import java.time.Duration;

public class CountryStateHelper {

    private final WebDriver driver;
    private final WebDriverWait wait;

    public CountryStateHelper(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(30));
    }

    public void selectCountryAndWaitForStates(String countryName) {

        By countryDropdown = By.id("country");
        By stateDropdown = By.id("state");

        Select countrySelect =
                new Select(wait.until(
                        ExpectedConditions.elementToBeClickable(countryDropdown)));

        countrySelect.selectByVisibleText(countryName);

        try {
            // Wait for state dropdown to become visible
            wait.until(ExpectedConditions.visibilityOfElementLocated(stateDropdown));

        } catch (TimeoutException e) {

            System.out.println("States dropdown not visible. Retrying event.");

            // Trigger change event explicitly using JavaScript
            WebElement countryElement = driver.findElement(countryDropdown);

            JavascriptExecutor js = (JavascriptExecutor) driver;

            js.executeScript(
                "arguments[0].dispatchEvent(new Event('change', {bubbles:true}));",
                countryElement);

            wait.until(ExpectedConditions.visibilityOfElementLocated(stateDropdown));
        }

        // Optional check: ensure states are loaded
        wait.until(driver -> {
            Select stateSelect =
                    new Select(driver.findElement(stateDropdown));
            return stateSelect.getOptions().size() > 1;
        });

        System.out.println("States dropdown loaded successfully.");
    }
}
```

# More Reliable Solution Using Ajax Monitoring

If your framework already injects an Ajax tracker, wait for Ajax completion before checking States dropdown.

```java
public void waitForAjaxCompletion() {

    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));

    wait.until(d -> {
        JavascriptExecutor js = (JavascriptExecutor) d;

        Object activeJquery = js.executeScript(
                        "return window.jQuery ? jQuery.active : 0;");

        return Long.valueOf(String.valueOf(activeJquery)) == 0;
    });
}
```

Usage:
```java
countrySelect.selectByVisibleText("India");

waitForAjaxCompletion();

wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("state")));
```

# Enterprise-Grade Retry Logic

For intermittent Ajax failures:

```java
public void loadStatesWithRetry(String country) {

    int maxAttempts = 3;

    for (int attempt = 1; attempt <= maxAttempts; attempt++) {

        try {

            new Select(driver.findElement(By.id("country"))).selectByVisibleText(country);

            waitForAjaxCompletion();

            wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("state")));

            wait.until(driver -> new Select(driver.findElement(By.id("state")))
                            .getOptions().size() > 1);

            System.out.println("States loaded successfully.");
            return;

        } catch (Exception ex) {

            System.out.println("Attempt " + attempt + " failed. Retrying...");
        }
    }

    throw new TimeoutException("States dropdown did not load after retry attempts.");
}

# For Modern React/Angular Applications

Sometimes the dropdown is populated only after the browser loses focus from the Country control. In such cases trigger both events:

```java
JavascriptExecutor js = (JavascriptExecutor) driver;

WebElement country = driver.findElement(By.id("country"));

js.executeScript(
    "arguments[0].dispatchEvent(new Event('input', {bubbles:true}));" +
    "arguments[0].dispatchEvent(new Event('change', {bubbles:true}));" +
    "arguments[0].dispatchEvent(new Event('blur', {bubbles:true}));",
    country
);
```

This is usually the most reliable way to force Angular/React/Vue listeners to execute and display the dependent States dropdown.
