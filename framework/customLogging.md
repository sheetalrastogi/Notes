Creating a **custom logging framework** in a Selenium Java automation
project provides much better control than relying only on
`System.out.println()` or even the default Log4j messages. A
well-designed logger should capture:

-   Test execution steps
-   Element interactions
-   API requests/responses
-   Exceptions
-   Performance information
-   Screenshots
-   Thread information (Parallel execution)
-   Browser/Platform details

Below is an enterprise-grade approach.

------------------------------------------------------------------------

# Architecture

    Automation Framework
    │
    ├── DriverFactory
    ├── BaseTest
    ├── Pages
    ├── Utilities
    │     ├── Logger.java
    │     ├── LogManager.java
    │     ├── ScreenshotUtil.java
    │     ├── ReportManager.java
    │     └── ConfigManager.java
    │
    ├── logs
    │      execution.log
    │      Error.log
    │
    └── reports

------------------------------------------------------------------------

# Step 1 Create Log Levels

``` java
public enum LogLevel {

    INFO,
    DEBUG,
    WARN,
    ERROR,
    PASS,
    FAIL,
    STEP,
    TRACE

}
```

------------------------------------------------------------------------

# Step 2 Create Custom Logger

``` java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Logger {

    private static final DateTimeFormatter formatter =
            DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    public static void log(LogLevel level, String message){

        String time = LocalDateTime.now().format(formatter);

        System.out.println(
                String.format("%s [%s] %s",
                        time,
                        level,
                        message));

    }

}
```

Usage

``` java
Logger.log(LogLevel.INFO,"Launching Browser");
Logger.log(LogLevel.STEP,"Entering Username");
Logger.log(LogLevel.PASS,"Login Successful");
Logger.log(LogLevel.FAIL,"Unable to locate Login Button");
```

Output

    2026-07-24 11:10:01 [INFO] Launching Browser
    2026-07-24 11:10:04 [STEP] Entering Username
    2026-07-24 11:10:08 [PASS] Login Successful

------------------------------------------------------------------------

# Step 3 Write Logs to File

``` java
import java.io.FileWriter;
import java.io.IOException;

public class FileLogger {

    private static final String LOG_FILE = "logs/execution.log";

    public static synchronized void write(String message){

        try(FileWriter fw = new FileWriter(LOG_FILE,true)){
            fw.write(message + System.lineSeparator());
        }
        catch(IOException e){
            e.printStackTrace();
        }

    }

}
```

Modify Logger

``` java
public static void log(LogLevel level,String message){

    String log =
            LocalDateTime.now()
            + " [" + level + "] "
            + message;

    System.out.println(log);

    FileLogger.write(log);

}
```

------------------------------------------------------------------------

# Step 4 Thread Safe Logging

For parallel execution

``` java
public class FrameworkLogger {

    private static ThreadLocal<String> testName = new ThreadLocal<>();

    public static void setTestName(String name){

        testName.set(name);

    }

    public static void info(String message){

        Logger.log(LogLevel.INFO,
                "[" + testName.get() + "] " + message);

    }

}
```

Output

    [ChromeTest] Browser Started

    [FirefoxTest] Browser Started

No mixing of logs.

------------------------------------------------------------------------

# Step 5 Automatic Selenium Logging

Instead of writing

``` java
driver.findElement(By.id("username")).click();
```

Create a wrapper.

``` java
public class ElementActions {

    WebDriver driver;

    public ElementActions(WebDriver driver){

        this.driver = driver;

    }

    public void click(By locator){

        Logger.log(LogLevel.STEP,
                "Clicking : " + locator);

        driver.findElement(locator).click();

        Logger.log(LogLevel.PASS,
                "Clicked Successfully");

    }

}
```

Usage

``` java
actions.click(By.id("login"));
```

Logs

    STEP Clicking login button

    PASS Click Successful

------------------------------------------------------------------------

# Step 6 Log Exceptions

``` java
try{

    driver.findElement(By.id("abc")).click();

}
catch(Exception e){

    Logger.log(LogLevel.ERROR,
            e.getMessage());

}
```

Better

``` java
Logger.error("Unable to click Login Button",e);
```

Output

    ERROR Unable to click Login Button

    Exception

    org.openqa.selenium.NoSuchElementException...

------------------------------------------------------------------------

# Step 7 Capture Screenshot Automatically

``` java
catch(Exception e){

    String screenshot =
            ScreenshotUtil.capture(driver);

    Logger.log(LogLevel.ERROR,
            "Screenshot : " + screenshot);

}
```

Output

    ERROR Login Failed

    Screenshot

    reports/images/LoginFailed.png

------------------------------------------------------------------------

# Step 8 Selenium Wrapper

Create all interactions.

``` java
click()

type()

select()

hover()

scroll()

dragDrop()

upload()

wait()
```

Each method logs automatically.

Example

``` java
public void enterText(By locator,String text){

    Logger.log(LogLevel.STEP,
            "Entering : " + text);

    driver.findElement(locator)
            .sendKeys(text);

}
```

No logging code is required in test cases.

------------------------------------------------------------------------

# Step 9 Measure Execution Time

``` java
long start = System.currentTimeMillis();

actions.click(locator);

long end = System.currentTimeMillis();

Logger.log(LogLevel.INFO,
        "Execution Time : "
                +(end-start)+" ms");
```

Output

    Execution Time : 435 ms

------------------------------------------------------------------------

# Step 10 Log Browser Information

``` java
Logger.log(INFO,"Browser : Chrome");
Logger.log(INFO,"Version : 139");
Logger.log(INFO,"OS : Windows 11");
Logger.log(INFO,"Environment : QA");
```

------------------------------------------------------------------------

# Step 11 Integrate with Extent Report

``` java
Logger.pass("Login Successful");
```

Internally

``` java
ExtentTestManager.getTest().pass(message);

FileLogger.write(message);

ConsoleLogger.write(message);
```

One call updates:

-   Console
-   Log File
-   Extent Report

------------------------------------------------------------------------

# Step 12 Central Logging API

Provide simple methods:

``` java
Logger.info("Launching Browser");

Logger.step("Click Login");

Logger.pass("User Logged In");

Logger.warn("Retrying...");

Logger.fail("Login Failed");

Logger.error(exception);

Logger.debug("API Response");

Logger.trace("Locator Details");
```

------------------------------------------------------------------------

# Sample Execution Log

    =================================================

    TEST : Login Test

    =================================================

    INFO  Launching Chrome Browser

    INFO  Browser Version : 139

    STEP  Navigate to Login Page

    STEP  Enter Username

    PASS  Username Entered

    STEP  Enter Password

    PASS  Password Entered

    STEP  Click Login

    PASS  Login Successful

    INFO  Dashboard Loaded

    INFO  Execution Time : 8.4 sec

    =================================================

------------------------------------------------------------------------

# Recommended Enterprise Logging Design

For a scalable Selenium framework, structure logging around these
components:

  ----------------------------------------------------------------------------
  Component              Responsibility
  ---------------------- -----------------------------------------------------
  `FrameworkLogger`      Public API (`info()`, `step()`, `pass()`, `fail()`,
                         `debug()`)

  `LogManager`           Coordinates all logging destinations

  `ConsoleAppender`      Writes colorized output to the console (optional)

  `FileAppender`         Persists execution logs to timestamped files

  `ExtentAppender`       Sends logs to Extent Reports

  `ScreenshotAppender`   Attaches screenshots on failures

  `ThreadContext`        Maintains per-test context for parallel execution

  `ElementActions`       Wraps Selenium interactions and logs automatically

  `ExceptionHandler`     Logs stack traces and captures screenshots
  ----------------------------------------------------------------------------

### Should you build from scratch or use a logging library?

For most enterprise Selenium frameworks, the best practice is to **use a
mature logging framework like Log4j2 or SLF4J + Logback as the
foundation**, then build your own `FrameworkLogger` facade on top. This
gives you:

-   High-performance asynchronous logging
-   Rolling log files
-   Configurable log levels
-   Thread-safe logging for parallel execution
-   Flexible appenders (console, file, reports)

Your custom layer then adds automation-specific features such as test
steps, screenshots, browser details, and Extent Report integration
without exposing the underlying logging library throughout your
codebase. This approach combines enterprise reliability with a clean,
automation-focused API.
