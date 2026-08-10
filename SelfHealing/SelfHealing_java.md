## Create a simple implementation of attached Self healing framework through AI agent for Selneium Java 4
---

Simple implementation for Selenium 4 + Java AI Agent for a Self Healing:

```text
Normal Locator
      ↓
Found ?
  Yes → Execute
      ↓
  No
      ↓
Check Cache
      ↓
Found ?
  Yes → Execute
      ↓
  No
      ↓
Capture DOM
      ↓
AI Agent Generates Locator
      ↓
Validation Engine
      ↓
Execute Action
      ↓
Cache Result
      ↓
Log Healing Event
```

**The design follows the principle that AI is invoked only when the original locator fails.**

Project Structure
```text
src
├── ai
│   ├── HealingAgent.java
│   ├── LocatorParser.java
│   └── LocatorSuggestion.java
│
├── cache
│   └── LocatorCache.java
│
├── validation
│   └── LocatorValidator.java
│
├── healing
│   └── SelfHealingDriver.java
│
├── pages
│   └── CheckoutPage.java
│
└── tests
    └── CheckoutTest.java
```

# Step 1: Locator Cache

```java
package cache;

import java.util.concurrent.ConcurrentHashMap;

public class LocatorCache {

    private static final ConcurrentHashMap<String, String> CACHE =
            new ConcurrentHashMap<>();

    public static void put(String key, String locator) {
        CACHE.put(key, locator);
    }

    public static String get(String key) {
        return CACHE.get(key);
    }
}
```

# Step 2: Locator Suggestion Model

```java
package ai;

public class LocatorSuggestion {

    private String locator;
    private double confidence;

    public LocatorSuggestion(String locator,
                             double confidence) {
        this.locator = locator;
        this.confidence = confidence;
    }

    public String getLocator() {
        return locator;
    }

    public double getConfidence() {
        return confidence;
    }
}
```


# Step 3: Locator Parser

Extract useful metadata before invoking AI.

```java
package ai;

import org.openqa.selenium.By;

public class LocatorParser {

    public static String parse(By locator,
                               String pageName,
                               String action) {

        return """
                Locator=%s
                Page=%s
                Action=%s
                """
                .formatted(locator,
                           pageName,
                           action);
    }
}
```


# Step 4: AI Healing Agent

For production this invokes:

Azure OpenAI
OpenAI
Copilot Studio Agent
Internal LLM Service


Sample implementation:

```java
package ai;

public class HealingAgent {

    public LocatorSuggestion heal(String dom,
                                  String context) {

        /*
         Prompt sent to LLM

         Original Locator Failed

         Context:
         %s

         DOM:
         %s

         Return BEST XPath
        */

        String healedXpath =
                "//button[contains(text(),'Next')]";

        return new LocatorSuggestion(
                healedXpath,
                0.92
        );
    }
}
```


# Step 5: Validation Layer

Never trust AI directly.

```java
package validation;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;

import java.util.List;

public class LocatorValidator {

    public boolean validate(
            WebDriver driver,
            String locator) {

        List<WebElement> elements =
                driver.findElements(
                        By.xpath(locator));

        if(elements.size() != 1)
            return false;

        WebElement element =
                elements.get(0);

        return element.isDisplayed()
                && element.isEnabled();
    }
}
```

# Step 6: Self-Healing Driver

Core framework component.

```java
package healing;

import ai.HealingAgent;
import ai.LocatorParser;
import ai.LocatorSuggestion;
import cache.LocatorCache;
import validation.LocatorValidator;

import org.openqa.selenium.*;

public class SelfHealingDriver {

    private WebDriver driver;

    private HealingAgent agent =
            new HealingAgent();

    private LocatorValidator validator =
            new LocatorValidator();

    public SelfHealingDriver(
            WebDriver driver) {

        this.driver = driver;
    }

    public WebElement findElement(
            By locator,
            String page,
            String action) {

        try {

            return driver.findElement(locator);

        } catch (NoSuchElementException ex) {

            return heal(locator,
                        page,
                        action);
        }
    }

    private WebElement heal(
            By locator,
            String page,
            String action) {

        String cacheKey = locator.toString();

        String cached =
                LocatorCache.get(cacheKey);

        if(cached != null) {

            return driver.findElement(
                    By.xpath(cached));
        }

        String context =
                LocatorParser.parse(
                        locator,
                        page,
                        action);

        String dom =
                driver.getPageSource();

        LocatorSuggestion suggestion =
                agent.heal(dom,
                           context);

        if(suggestion.getConfidence()
                < 0.80) {

            throw new RuntimeException(
                    "Low confidence locator");
        }

        if(!validator.validate(
                driver,
                suggestion.getLocator())) {

            throw new RuntimeException(
                    "Locator validation failed");
        }

        LocatorCache.put(
                cacheKey,
                suggestion.getLocator());

        System.out.println(
                "HEALED: "
                        + locator
                        + " -> "
                        + suggestion.getLocator());

        return driver.findElement(
                By.xpath(
                        suggestion.getLocator()));
    }
}
```


# Step 7: Page Object Usage

```java
public class CheckoutPage {

    private SelfHealingDriver driver;

    private By continueBtn =
            By.id("continueBtn");

    public CheckoutPage(
            SelfHealingDriver driver) {

        this.driver = driver;
    }

    public void clickContinue() {

        driver.findElement(
                continueBtn,
                "Checkout",
                "Click")
                .click();
    }
}
```

Test Case

```java
@Test
public void checkoutTest() {

    CheckoutPage page =
            new CheckoutPage(
                    healingDriver);

    page.clickContinue();
}
```


**Enterprise Enhancements**

 - Agent Capabilities 
 - DOM Analysis Agent
 - Context Detection Agent
 - Locator Generation Agent
 - Validation Agent
 - Logging Agent
 - PR Creation Agent
 - Additional Signals
 - Element Text
 - ARIA Labels
 - CSS Classes
 - Parent Hierarchy
 - Relative XPath
 - Coordinates
 - User Action Type
 - Storage
 - Redis
 - MongoDB
 - CosmosDB
 - PostgreSQL

**LLM Providers**

 - Azure OpenAI GPT-4o
 - Azure AI Foundry Agent
 - Copilot Studio Agent
 - Local Llama Model
 - OpenAI GPT Models

## Safe Strategy 
```text
Locator Fails
      ↓
AI Suggests Locator
      ↓
Validation Layer
      ↓
Runtime Healing
      ↓
Logging
      ↓
Human Review
```

This aligns with the framework's recommended Runtime Only Healing approach, where AI acts as a controlled fallback, validation remains mandatory, and healed locators are cached and logged for future executions.



