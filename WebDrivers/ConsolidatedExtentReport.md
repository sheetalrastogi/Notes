## Consolidated Extent Report for Selenium Grid Execution
---


Steps to Create a Consolidated Extent Report During Selenium Grid Execution (Selenium Java 4)

When running tests in parallel across multiple Selenium Grid nodes, the goal is to generate one unified Extent Report regardless of where the browser session executes.

# Recommended Architecture

```text
                    TestNG/JUnit Runner
                            |
                            v
                    Selenium Grid Hub
                            |
           ---------------------------------
           |               |               |
           v               v               v
      Chrome Node    Firefox Node     Edge Node
           |               |               |
           +---------------+---------------+
                           |
                    Extent Manager
                           |
                           v
                Consolidated Extent Report
```

# Typical Framework Structure

```text
src
 ├─ driver
 │    └─ DriverManager.java
 │
 ├─ reports
 │    ├─ ExtentManager.java
 │    ├─ ExtentTestManager.java
 │    └─ ScreenshotUtil.java
 │
 ├─ listeners
 │    └─ ExtentListener.java
 │
 ├─ tests
 │    └─ LoginTest.java
 │
 └─ resources
      └─ extent-config.xml
```

# Key Principle
- Extent Report should be generated only from the Test Runner machine
- Screenshots should be captured as Base64 or transferred to a common location
- ThreadLocal must be used for ExtentTest objects
- Each parallel thread logs to its own ExtentTest instance


# Step 1: Create Singleton Extent Report Manager

```java
public class ExtentManager {

    private static ExtentReports extent;

    public static synchronized ExtentReports getInstance() {

        if(extent == null) {

            ExtentSparkReporter spark =
                    new ExtentSparkReporter(
                            "reports/ExtentReport.html");

            spark.config().setDocumentTitle("Grid Execution Report");
            spark.config().setReportName("Selenium Grid Results");

            extent = new ExtentReports();
            extent.attachReporter(spark);

            extent.setSystemInfo("Execution", "Grid");
            extent.setSystemInfo("Framework", "Selenium Java 4");
        }

        return extent;
    }
}
```
# Step 2: Create ThreadLocal ExtentTest

Parallel execution requires thread-safe reporting.

```java

public class ExtentTestManager {

    private static ThreadLocal<ExtentTest> extentTest =
            new ThreadLocal<>();

    public static ExtentTest getTest() {
        return extentTest.get();
    }

    public static void startTest(String testName) {

        ExtentTest test =
                ExtentManager.getInstance()
                        .createTest(testName);

        extentTest.set(test);
    }
}
```

# Step 3: Capture Browser & Node Information

For Grid visibility add:

```java
RemoteWebDriver driver = (RemoteWebDriver)DriverManager.getDriver();

Capabilities caps = driver.getCapabilities();

String browser = caps.getBrowserName();

String version = caps.getBrowserVersion();

String sessionId = driver.getSessionId().toString();
```

Add to report:

```java
ExtentTestManager.getTest().info("Browser : " + browser);

ExtentTestManager.getTest().info("Version : " + version);

ExtentTestManager.getTest().info("Session : " + sessionId);
```

# Step 4: Capture Node Name

**Selenium Grid provides node information.**

```java
String nodeName = String.valueOf(caps.getCapability("se:containerName"));

or

String nodeName = String.valueOf(caps.getCapability("nodeName"));
```

**Log it:**

```java
ExtentTestManager.getTest().info("Node : " + nodeName);
```

# Report Example:

```text
Test : LoginTest

Browser : Chrome
Version : 138
Node : chrome-node-01
Session : e7347eab2...
```

# Step 5: Start Reporting Inside Test Listener

TestNG Listener
```java
public class ExtentListener
        implements ITestListener {

    @Override
    public void onTestStart(ITestResult result) {

        ExtentTestManager.startTest(
                result.getMethod()
                      .getMethodName());
    }
}
```

# Step 6: Log Pass/Fail Status

Pass
```java
@Override
public void onTestSuccess(
        ITestResult result) {

    ExtentTestManager.getTest()
        .pass("Test Passed");
}
```

Fail
```java
@Override
public void onTestFailure(
        ITestResult result) {

    ExtentTestManager.getTest()
        .fail(result.getThrowable());
}
```

# Step 7: Capture Screenshot as Base64

This is the most Grid-friendly approach.
```java

String base64 = ((TakesScreenshot)driver).getScreenshotAs(OutputType.BASE64);
```

Attach it:
```java

ExtentTestManager.getTest().fail(MediaEntityBuilder.createScreenCaptureFromBase64String(base64).build());
```

Why Base64?

- Avoids:
```text
  Node1 Screenshot Folder
  Node2 Screenshot Folder
  Node3 Screenshot Folder
```

- No file copying required.
Works on:
```text
Selenium Grid
Docker Grid
Kubernetes
BrowserStack
LambdaTest
```

Step 8: Optional - Save Physical Screenshots

If audit evidence is required:

File src = ((TakesScreenshot)driver).getScreenshotAs(OutputType.FILE);

String fileName = testName + "_" + browser + "_" + nodeName + "_" + timestamp + ".png";


Save under:
```text
reports/
   screenshots/
      Chrome/
      Firefox/
      Edge/
```

# Step 9: Flush Report Once After Entire Suite

Very important.

Do not flush after every test.

@Override
public void onFinish(
        ITestContext context) {
	    ExtentManager.getInstance().flush();
}

# Step 10: Add Execution Dashboard Metrics

Maintain counters:

Passed
Failed
Skipped
Execution Time
Browser
Node
Environment
Build Number


Example:

extent.setSystemInfo(
        "Environment",
        "QA");

extent.setSystemInfo(
        "Grid URL",
        "http://grid:4444");

extent.setSystemInfo(
        "Build",
        "Build-201");




## complete enterprise-ready Selenium Java 4 + Selenium Grid 4 + TestNG + Extent Report example that demonstrates:

- Parallel Execution
- ThreadLocal Driver
- ThreadLocal ExtentTest
- Browser Details Capture
- Grid Node Details Capture
- Base64 Screenshot Capture
- Screenshot Attachment to Extent Report
- Single Consolidated Extent Report


# Framework Structure

```text
src/test/java
|
├── driver
│     └── DriverManager.java
│
├── reports
│     ├── ExtentManager.java
│     └── ExtentTestManager.java
│
├── listeners
│     └── ExtentListener.java
│
├── tests
│     └── LoginTest.java
│
└── utils
      └── ScreenshotUtil.java
```

# 1. DriverManager (ThreadLocal Driver)

```java
package driver;

import org.openqa.selenium.WebDriver;

public class DriverManager {

    private static final ThreadLocal<WebDriver> DRIVER =
            new ThreadLocal<>();

    public static void setDriver(WebDriver driver) {
        DRIVER.set(driver);
    }

    public static WebDriver getDriver() {
        return DRIVER.get();
    }

    public static void unload() {
        DRIVER.remove();
    }
}
```

# 2. ExtentManager (Singleton Report)

```java
package reports;

import com.aventstack.extentreports.*;
import com.aventstack.extentreports.reporter.ExtentSparkReporter;

public class ExtentManager {

    private static ExtentReports extent;

    public static synchronized ExtentReports getExtent() {

        if(extent == null) {

            ExtentSparkReporter spark =
                    new ExtentSparkReporter(
                            "reports/GridExecutionReport.html");

            spark.config().setDocumentTitle(
                    "Selenium Grid Report");

            spark.config().setReportName(
                    "Parallel Grid Execution");

            extent = new ExtentReports();
            extent.attachReporter(spark);

            extent.setSystemInfo(
                    "Automation Tool",
                    "Selenium Java 4");

            extent.setSystemInfo(
                    "Execution Mode",
                    "Selenium Grid");
        }

        return extent;
    }
}
```

# 3. ExtentTestManager (ThreadLocal ExtentTest)

```java
package reports;

import com.aventstack.extentreports.ExtentTest;

public class ExtentTestManager {

    private static ThreadLocal<ExtentTest> TEST =
            new ThreadLocal<>();

    public static void createTest(String testName) {

        ExtentTest extentTest =
                ExtentManager.getExtent()
                        .createTest(testName);

        TEST.set(extentTest);
    }

    public static ExtentTest getTest() {
        return TEST.get();
    }
}
```

# 4. Screenshot Utility (Base64)

```java
package utils;

import org.openqa.selenium.*;

public class ScreenshotUtil {

    public static String captureBase64(
            WebDriver driver) {

        return ((TakesScreenshot) driver)
                .getScreenshotAs(OutputType.BASE64);
    }
}
```

# 5. BaseTest (Grid Setup)

```java
package tests;

import driver.DriverManager;
import org.openqa.selenium.Platform;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.remote.RemoteWebDriver;

import org.testng.annotations.*;

import java.net.URL;

public class BaseTest {

    @Parameters({"browser"})
    @BeforeMethod
    public void setup(String browser)
            throws Exception {

        ChromeOptions options =
                new ChromeOptions();

        options.setPlatformName("windows");

        RemoteWebDriver driver =
                new RemoteWebDriver(
                        new URL(
                          "http://localhost:4444"),
                        options);

        DriverManager.setDriver(driver);
    }

    @AfterMethod(alwaysRun = true)
    public void tearDown() {

        if(DriverManager.getDriver() != null) {
            DriverManager.getDriver().quit();
        }

        DriverManager.unload();
    }
}
```

# 6. Extent Listener

```java
package listeners;

import com.aventstack.extentreports.MediaEntityBuilder;

import driver.DriverManager;
import reports.ExtentManager;
import reports.ExtentTestManager;
import utils.ScreenshotUtil;

import org.openqa.selenium.Capabilities;
import org.openqa.selenium.remote.RemoteWebDriver;

import org.testng.*;

public class ExtentListener
        implements ITestListener {

    @Override
    public void onTestStart(
            ITestResult result) {

        ExtentTestManager.createTest(
                result.getMethod()
                        .getMethodName());

        RemoteWebDriver driver =
            (RemoteWebDriver)
                DriverManager.getDriver();

        Capabilities caps =
                driver.getCapabilities();

        String browser =
                caps.getBrowserName();

        String version =
                caps.getBrowserVersion();

        String sessionId =
                driver.getSessionId().toString();

        ExtentTestManager.getTest()
                .info("Browser : " + browser);

        ExtentTestManager.getTest()
                .info("Version : " + version);

        ExtentTestManager.getTest()
                .info("Session : " + sessionId);

        Object nodeName =
                caps.getCapability(
                        "se:containerName");

        if(nodeName != null) {

            ExtentTestManager
                    .getTest()
                    .info("Node : " + nodeName);
        }
    }

    @Override
    public void onTestSuccess(
            ITestResult result) {

        ExtentTestManager.getTest()
                .pass("Test Passed");
    }

    @Override
    public void onTestFailure(
            ITestResult result) {

        String base64 =
                ScreenshotUtil
                        .captureBase64(
                                DriverManager
                                   .getDriver());

        ExtentTestManager.getTest()
                .fail(result.getThrowable());

        ExtentTestManager.getTest()
                .fail(
                    MediaEntityBuilder
                        .createScreenCaptureFromBase64String(
                                base64)
                        .build());
    }

    @Override
    public void onFinish(
            ITestContext context) {

        ExtentManager
                .getExtent()
                .flush();
    }
}
```


# 7. Sample Test

```java

package tests;

import driver.DriverManager;

import org.testng.Assert;
import org.testng.annotations.Listeners;
import org.testng.annotations.Test;

@Listeners(listeners.ExtentListener.class)
public class LoginTest extends BaseTest {

    @Test
    public void googleTest() {

        DriverManager.getDriver()
                .get("https://www.google.com");

        Assert.assertTrue(
                DriverManager.getDriver()
                        .getTitle()
                        .contains("Google"));
    }

    @Test
    public void failedTest() {

        DriverManager.getDriver()
                .get("https://www.google.com");

        Assert.assertEquals(
                "ABC",
                DriverManager.getDriver()
                         .getTitle());
    }
}

```

# 8. TestNG Parallel Configuration

```xml
<!DOCTYPE suite SYSTEM
"https://testng.org/testng-1.0.dtd">

<suite name="GridExecutionSuite"
       parallel="tests"
       thread-count="3">

    <test name="Chrome">

        <parameter name="browser"
                   value="chrome"/>

        <classes>
            <class name="tests.LoginTest"/>
        </classes>

    </test>

    <test name="Chrome2">

        <parameter name="browser"
                   value="chrome"/>

        <classes>
            <class name="tests.LoginTest"/>
        </classes>

    </test>

</suite>
```


## Enhanced Node Identification (Grid 4)

- Sometimes capability values contain node details:

Capabilities caps = driver.getCapabilities();
System.out.println(caps.asMap());


Sample output:

browserName=chrome
browserVersion=138
platformName=windows
se:containerName=chrome-node-01


Capture and log:

```java
caps.asMap().forEach((k,v) ->
        ExtentTestManager
        .getTest()
        .info(k + " : " + v));
```

# Sample Consolidated Extent Report Output

```text
Test : googleTest

Browser : chrome
Version : 138
Node : chrome-node-01
Session : d3f4c123

Result : PASS
--------------------------------------------------

Test : failedTest

Browser : chrome
Version : 138
Node : chrome-node-02
Session : ae56f567

Result : FAIL

Screenshot Attached
Stack Trace Attached
```

# Suggested Improvements

For production frameworks, additionally capture:

test.info("Grid URL");
test.info("Node Name");
test.info("Browser");
test.info("Browser Version");
test.info("Platform");
test.info("Session ID");
test.info("Test Start Time");
test.info("Execution Duration");
test.info("Thread ID");
test.info("Build Number");
test.info("Git Commit ID");


This design is fully compatible with Selenium Grid 4 Hub/Node, Fully Distributed Grid, Docker Selenium Grid, Jenkins distributed agents, Azure DevOps, and GitHub Actions, while producing a single consolidated Extent Report for all parallel executions.


