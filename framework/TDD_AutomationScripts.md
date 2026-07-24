For an enterprise Selenium Java framework, avoid creating one test method per data row. Instead, design your framework so that **one test script executes multiple business scenarios**, with **multiple validations (assertions) driven entirely from a test data file**.

This makes the framework:

* Completely data-driven
* Easy for business users to maintain
* Scalable to thousands of test cases
* Suitable for CI/CD execution

---

# Recommended Architecture

```text
Test Data (Excel/JSON/YAML/CSV/Database)
            │
            ▼
     Data Loader
            │
            ▼
     Test Executor
            │
            ▼
    Business Keywords
            │
            ▼
     Selenium Actions
            │
            ▼
      Assertion Engine
            │
            ▼
 HTML Report / Extent Report
```

---

# Test Data Design

Instead of keeping only input data, include **expected results and assertions**.

Example (JSON)

```json
{
  "testName": "Verify Login",

  "execute": true,

  "steps": [

    {
      "action": "enter",

      "locator": "username",

      "value": "admin"
    },

    {
      "action": "enter",

      "locator": "password",

      "value": "password"
    },

    {
      "action": "click",

      "locator": "loginButton"
    }

  ],

  "assertions": [

    {
      "type": "title",

      "expected": "Dashboard"
    },

    {
      "type": "elementVisible",

      "locator": "welcomeMessage"
    },

    {
      "type": "text",

      "locator": "userName",

      "expected": "Admin"
    }

  ]

}
```

Notice that:

* Test actions are data-driven.
* Assertions are also data-driven.
* No hardcoded validations exist in Java code.

---

# Framework Design

```text
tests
   │
   ▼
LoginTest.java
   │
   ▼
TestExecutor
   │
   ▼
ActionEngine
   │
   ▼
AssertionEngine
```

The test class becomes very small.

```java
@Test(dataProvider = "loginData")
public void loginTest(TestScenario scenario) {

    TestExecutor.execute(scenario);

}
```

---

# Data Provider

```java
@DataProvider

public Object[][] loginData(){

    return JsonLoader.load("login.json");

}
```

Each row becomes one execution.

---

# Test Executor

```java
public class TestExecutor {

    public static void execute(TestScenario scenario){

        ActionEngine.executeSteps(
                scenario.getSteps());

        AssertionEngine.validate(
                scenario.getAssertions());

    }

}
```

---

# Action Engine

Instead of writing Selenium code inside tests:

```java
driver.findElement(...).click();
```

Create an action engine.

```java
switch(action){

    case "click":

        element.click();

        break;

    case "enter":

        element.sendKeys(value);

        break;

    case "select":

        select.selectByVisibleText(value);

        break;

}
```

The engine understands business keywords.

---

# Assertion Engine

Instead of

```java
Assert.assertEquals(...);

Assert.assertTrue(...);
```

Execute assertions dynamically.

```java
for(Assertion assertion : assertions){

    switch(assertion.getType()){

        case "title":

            Assert.assertEquals(
                    driver.getTitle(),
                    assertion.getExpected());

            break;

        case "text":

            Assert.assertEquals(
                    element.getText(),
                    assertion.getExpected());

            break;

        case "visible":

            Assert.assertTrue(
                    element.isDisplayed());

            break;

    }

}
```

---

# Multiple Assertions

Suppose the data file contains

```json
"assertions":[

{

"type":"title",

"expected":"Dashboard"

},

{

"type":"visible",

"locator":"welcome"

},

{

"type":"enabled",

"locator":"logout"

},

{

"type":"text",

"locator":"username",

"expected":"Admin"

}
]
```

The framework executes all validations.

No Java code changes.

---

# Use Soft Assertions

Never stop execution after the first failed validation.

```java
SoftAssert softAssert = new SoftAssert();

softAssert.assertEquals(...);

softAssert.assertTrue(...);

softAssert.assertFalse(...);

softAssert.assertAll();
```

Output

```text
Assertion 1 Passed

Assertion 2 Passed

Assertion 3 Failed

Assertion 4 Passed
```

This provides much richer reporting.

---

# Sample Excel Design

| TestCase | Execute | Username | Password | Expected Title | Expected User | Welcome Visible | Logout Enabled |
| -------- | ------- | -------- | -------- | -------------- | ------------- | --------------- | -------------- |
| Login01  | Yes     | admin    | admin123 | Dashboard      | Admin         | Yes             | Yes            |
| Login02  | Yes     | invalid  | invalid  | Login Failed   | NA            | No              | No             |

The framework converts each row into:

* Actions
* Assertions

---

# Generic Validation Interface

```java
public interface Validator {

    void validate(Assertion assertion);

}
```

Implementations

```text
TitleValidator

TextValidator

VisibilityValidator

URLValidator

AttributeValidator

CssValidator

TableValidator

APIValidator

DatabaseValidator
```

The assertion engine selects the appropriate validator based on the assertion type.

---

# Assertion Factory

```java
Validator validator =

ValidatorFactory.getValidator(

assertion.getType());

validator.validate(assertion);
```

This follows the **Factory Pattern** and keeps the framework extensible.

---

# Reporting

Generate detailed assertion-level results.

| Assertion       | Expected  | Actual    | Status |
| --------------- | --------- | --------- | ------ |
| Title           | Dashboard | Dashboard | ✅ Pass |
| Welcome Visible | True      | True      | ✅ Pass |
| Username        | Admin     | Admin     | ✅ Pass |
| Logout Enabled  | True      | False     | ❌ Fail |

---

# Framework Flow

```text
Read JSON/Excel
        │
        ▼
Load Test Data
        │
        ▼
Execute Business Steps
        │
        ▼
Capture Actual Results
        │
        ▼
Run Assertion Engine
        │
        ▼
Soft Assertions
        │
        ▼
Generate HTML Report
```

---

# Enterprise Enhancements

For large-scale automation frameworks, consider these additional capabilities:

| Feature                        | Benefit                                           |
| ------------------------------ | ------------------------------------------------- |
| JSON/YAML-based test data      | Easier version control than Excel                 |
| Keyword-driven action engine   | Non-developers can define scenarios               |
| Dynamic assertion engine       | New assertion types without changing test classes |
| Soft assertions                | Complete validation results in one execution      |
| Screenshot on failed assertion | Faster debugging                                  |
| Retry only failed assertions   | Reduce re-execution time                          |
| Parallel execution             | Run multiple data sets simultaneously             |
| Environment-specific test data | Same test across QA, UAT, and Production          |
| HTML/Extent assertion reports  | Detailed visibility into every validation         |

## Recommended Design Pattern

For enterprise Selenium Java frameworks, a combination of the following patterns works well:

```text
TestNG DataProvider
        │
        ▼
Test Executor
        │
        ▼
Command Pattern (Action Engine)
        │
        ▼
Factory Pattern (Validator Factory)
        │
        ▼
Strategy Pattern (Individual Validators)
        │
        ▼
Soft Assertion Manager
        │
        ▼
Reporting Engine
```

This architecture allows a **single test script** to execute hundreds of data-driven scenarios, each with dozens of assertions defined externally, while keeping test classes small, maintainable, and reusable. It also makes it straightforward to add new actions or validation types without modifying existing test logic.
