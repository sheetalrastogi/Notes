TestNG Listeners

```text
ITestListener
IInvokedMethodListener
IRetryAnalyzer
```



# 1. TestNG ITestListener implementation 
------

**ITestListener logs test status**

TestNG ITestListener implementation that logs:
- Test Method Invocation
- Test Start
- Test Success
- Test Failure
- Test Skipped
- Test Failed But Within Success Percentage
- Suite Start/Finish
- Execution Duration

# Sample ITestListener implementation

```java
package assurant;

package listeners;

import org.testng.ITestContext;
import org.testng.ITestListener;
import org.testng.ITestResult;

import java.time.LocalDateTime;

public class TestExecutionListener implements ITestListener {

	@Override
	public void onStart(ITestContext context) {
		System.out.println("SUITE STARTED : " + context.getName()+ "at " + LocalDateTime.now());
	}

	@Override
	public void onTestStart(ITestResult result) {
		System.out.println("TEST STARTED - for Class: "+ result.getTestClass().getName() +"Method : " + result.getMethod().getMethodName());
	}

	@Override
	public void onTestSuccess(ITestResult result) {
		long duration = result.getEndMillis() - result.getStartMillis();
		System.out.println("✅ TEST PASSED" + "Method :" + result.getMethod().getMethodName() + "Duration" + duration + " ms");
	}

	@Override
	public void onTestFailure(ITestResult result) {

		long duration = result.getEndMillis() - result.getStartMillis();
		System.out.println("❌ TEST FAILED" + "Method :" + result.getMethod().getMethodName() + "Duration" + duration + " ms");

		System.out.println("Exception: " + result.getThrowable());
	}

	@Override
	public void onTestSkipped(ITestResult result) {
		System.out.println("⚠ TEST SKIPPED"+"Method : " + result.getMethod().getMethodName());
	}

	@Override
	public void onTestFailedButWithinSuccessPercentage(ITestResult result) {
		System.out.println("TEST FAILED WITHIN SUCCESS PERCENTAGE" + "Method : " + result.getMethod().getMethodName());
	}

	@Override
	public void onFinish(ITestContext context) {

		System.out.println("\n======================================");
		System.out.println("SUITE FINISHED : " + context.getName());
		System.out.println("End Time       : " + LocalDateTime.now());
		System.out.println("Passed Tests   : " + context.getPassedTests().size());
		System.out.println("Failed Tests   : " + context.getFailedTests().size());
		System.out.println("Skipped Tests  : " + context.getSkippedTests().size());
		System.out.println("======================================\n");
	}
}
```


# 2. IInvokedMethodListener

**If you want to log every invocation before and after execution, implement IInvokedMethodListener.**


```java
import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.StandardCopyOption;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;

import org.testng.IInvokedMethod;
import org.testng.IInvokedMethodListener;
import org.testng.ITestResult;

public class InvocationLogger implements IInvokedMethodListener {

	@Override
	public void beforeInvocation(IInvokedMethod method, ITestResult result) {

		System.out.println(">>> STARTING : " + method.getTestMethod().getMethodName());
	}

	@Override
	public void afterInvocation(IInvokedMethod method, ITestResult result) {

		String methodName = method.getTestMethod().getMethodName();

		String status = getStatus(result.getStatus());

		System.out.println("<<< FINISHED : " + methodName + " Status = " + status);

		if (result.getStatus() == ITestResult.FAILURE) {

			System.out.println("❌ FAILURE DETECTED FOR : " + methodName);

			if (result.getThrowable() != null) {
				System.out.println("Exception : " + result.getThrowable().getMessage());
			}

			captureScreenshot(methodName);
		}
	}

	private void captureScreenshot(String methodName) {

		try {

			WebDriver driver = DriverManager.getDriver(); // ThreadLocal Driver

			if (driver == null) {
				System.err.println("Driver is null. Screenshot cannot be captured.");
				return;
			}

			File source = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);

			String timestamp = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss"));

			String screenshotPath = "screenshots/" + methodName + "_" + timestamp + ".png";

			File destination = new File(screenshotPath);

			destination.getParentFile().mkdirs();

			Files.copy(source.toPath(), destination.toPath(), StandardCopyOption.REPLACE_EXISTING);

			System.out.println("📷 Screenshot Saved : " + destination.getAbsolutePath());

		} catch (IOException e) {

			System.err.println("Unable to capture screenshot : " + e.getMessage());
		}
	}

	private String getStatus(int status) {

		return switch (status) {

		case ITestResult.SUCCESS -> "PASS";

		case ITestResult.FAILURE -> "FAIL";

		case ITestResult.SKIP -> "SKIP";

		default -> "UNKNOWN";
		};
	}
}

```

# ThreadLocal Driver Manager Example


```java
	public final class DriverManager {

		private static final ThreadLocal<WebDriver> DRIVER = new ThreadLocal<>();

		private DriverManager() {
		}

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


**Sample Output**
```text
>>> STARTING : loginTest

<<< FINISHED : loginTest Status = FAIL Duration = 4312 ms

❌ FAILURE DETECTED FOR : loginTest

Exception :
ElementClickInterceptedException

📷 Screenshot Saved :
D:\Automation\screenshots\loginTest_20260812_163045.png
```



# 3. IRetryAnalyzer

A good enterprise approach is to combine **root-cause extraction** + **failure categorization** + **retry logging**. This provides meaningful retry diagnostics in reports and logs.


```java

import org.openqa.selenium.NoSuchElementException;
import org.openqa.selenium.StaleElementReferenceException;
import org.openqa.selenium.TimeoutException;
import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class RetryAnalyzer implements IRetryAnalyzer {

	private int retryCount = 0;
	private static final int MAX_RETRY_COUNT = 2;

	@Override
	public boolean retry(ITestResult result) {

		Throwable throwable = result.getThrowable();

		String rootCause = getRootCause(throwable);
		String category = categorizeFailure(throwable);

		System.out.println("\n========================================");
		System.out.println("Retry Attempt : " + (retryCount + 1));
		System.out.println("Test Name     : " + result.getMethod().getMethodName());
		System.out.println("Category      : " + category);
		System.out.println("Root Cause    : " + rootCause);
		System.out.println("========================================\n");

		if (retryCount < MAX_RETRY_COUNT) {
			retryCount++;
			return true;
		}

		System.out.println("\n******** FINAL FAILURE ********");
		System.out.println("Test Name     : " + result.getMethod().getMethodName());
		System.out.println("Category      : " + category);
		System.out.println("Root Cause    : " + rootCause);
		System.out.println("Retries Used  : " + MAX_RETRY_COUNT);
		System.out.println("*******************************\n");

		return false;
	}

	/**
	 * Returns deepest exception in chain.
	 */
	private String getRootCause(Throwable throwable) {

		if (throwable == null) {
			return "Unknown Failure";
		}

		Throwable root = throwable;

		while (root.getCause() != null) {
			root = root.getCause();
		}

		return root.getClass().getSimpleName() + " : " + root.getMessage();
	}

	/**
	 * Classifies exception into meaningful retry category.
	 */
	private String categorizeFailure(Throwable throwable) {

		if (throwable == null) {
			return "Unknown Failure";
		}

		Throwable root = throwable;

		while (root.getCause() != null) {
			root = root.getCause();
		}

		if (root instanceof TimeoutException) {
			return "Synchronization Issue";
		} else if (root instanceof StaleElementReferenceException) {
			return "Stale Element";
		} else if (root instanceof NoSuchElementException) {
			return "Element Not Found";
		} else {
			return "Framework/Application Failure";
		}
	}
}

```
