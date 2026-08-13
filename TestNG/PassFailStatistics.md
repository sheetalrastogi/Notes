give example of TestNG Listeners implementations for tracking pass/fail status of execution with following TestNG Listeners

ITestListener
      +
AtomicInteger / ConcurrentHashMap
      +
IExecutionListener
      +
IReporter


----


# 1. ITestListener - Track Pass / Fail / Skip Counts

```Java
package listeners;

import java.util.concurrent.atomic.AtomicInteger;

import org.testng.ITestListener;
import org.testng.ITestResult;

public class ExecutionStatisticsListener implements ITestListener {

	private static final AtomicInteger PASSED = new AtomicInteger();
	private static final AtomicInteger FAILED = new AtomicInteger();
	private static final AtomicInteger SKIPPED = new AtomicInteger();

	@Override
	public void onTestSuccess(ITestResult result) {
		PASSED.incrementAndGet();
		System.out.println("[PASS] " + result.getName());
	}

	@Override
	public void onTestFailure(ITestResult result) {
		FAILED.incrementAndGet();
		System.out.println("[FAIL] " + result.getName());
	}

	@Override
	public void onTestSkipped(ITestResult result) {
		SKIPPED.incrementAndGet();
		System.out.println("[SKIP] " + result.getName());
	}

	public static int getPassed() {
		return PASSED.get();
	}

	public static int getFailed() {
		return FAILED.get();
	}

	public static int getSkipped() {
		return SKIPPED.get();
	}
}
```


# 2. ISuiteListener - Generate Statistics After Suite Completion

```java
package listeners;

import org.testng.ISuite;
import org.testng.ISuiteListener;

public class SuiteStatisticsListener implements ISuiteListener {

	@Override
	public void onStart(ISuite suite) {

		System.out.println("Starting Suite : " + suite.getName());
	}

	@Override
	public void onFinish(ISuite suite) {

		System.out.println("\n===== SUITE SUMMARY =====");

		System.out.println("Passed : " + ExecutionStatisticsListener.getPassed());

		System.out.println("Failed : " + ExecutionStatisticsListener.getFailed());

		System.out.println("Skipped : " + ExecutionStatisticsListener.getSkipped());

		System.out.println("=========================");
	}
}
```

# 3. IExecutionListener - Entire Execution Statistics

```java
package listeners;

import org.testng.IExecutionListener;

public class ExecutionSummaryListener implements IExecutionListener {

	@Override
	public void onExecutionStart() {

		System.out.println("=== Test Execution Started ===");
	}

	@Override
	public void onExecutionFinish() {

		System.out.println("\n=== Test Execution Summary ===");

		System.out.println("Passed : " + ExecutionStatisticsListener.getPassed());

		System.out.println("Failed : " + ExecutionStatisticsListener.getFailed());

		System.out.println("Skipped : " + ExecutionStatisticsListener.getSkipped());
	}
}
```


# 4. IReporter - Generate Final Report

```java
package listeners;

import java.util.List;

import org.testng.IReporter;
import org.testng.ISuite;
import org.testng.xml.XmlSuite;

public class CustomReporter implements IReporter {

	@Override
	public void generateReport(List<XmlSuite> xmlSuites, List<ISuite> suites, String outputDirectory) {

		System.out.println("\n===== REPORT =====");

		System.out.println("Passed : " + ExecutionStatisticsListener.getPassed());

		System.out.println("Failed : " + ExecutionStatisticsListener.getFailed());

		System.out.println("Skipped : " + ExecutionStatisticsListener.getSkipped());

		System.out.println("Output Directory : " + outputDirectory);
	}
}
```

# 5. IInvokedMethodListener - Capture Execution Timing

```java
package listeners;

import org.testng.IInvokedMethod;
import org.testng.IInvokedMethodListener;
import org.testng.ITestResult;

public class MethodInvocationListener implements IInvokedMethodListener {

	@Override
	public void beforeInvocation(IInvokedMethod method, ITestResult result) {

		result.setAttribute("START_TIME", System.currentTimeMillis());
	}

	@Override
	public void afterInvocation(IInvokedMethod method, ITestResult result) {

		Long startTime = (Long) result.getAttribute("START_TIME");

		long duration = System.currentTimeMillis() - startTime;

		System.out.println(result.getMethod().getMethodName() + " Duration(ms): " + duration);
	}
}
```


# 6. IRetryAnalyzer - Track Retry Statistics

```java
package listeners;

import java.util.concurrent.atomic.AtomicInteger;

import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class RetryTracker implements IRetryAnalyzer {

	private int retryCount = 0;

	private static final int MAX_RETRY = 2;

	private static final AtomicInteger RETRIES = new AtomicInteger();

	@Override
	public boolean retry(ITestResult result) {

		if (retryCount < MAX_RETRY) {

			retryCount++;

			RETRIES.incrementAndGet();

			System.out.println("Retrying : " + result.getName() + " Attempt " + retryCount);

			return true;
		}

		return false;
	}

	public static int getTotalRetries() {

		return RETRIES.get();
	}
}
```

# 7. IDataProviderInterceptor - Count Filtered Records

```java
package listeners;

import java.util.Iterator;

import org.testng.IDataProviderInterceptor;
import org.testng.IDataProviderMethod;
import org.testng.ITestContext;
import org.testng.ITestNGMethod;

public class DataProviderStatisticsListener implements IDataProviderInterceptor {

	@Override
	public Iterator<Object[]> intercept(Iterator<Object[]> original, IDataProviderMethod dataProviderMethod,
			ITestNGMethod method, ITestContext context) {

		System.out.println("Processing DataProvider : " + dataProviderMethod.getName());

		return original;
	}
}
```


## Register in testng.xml

```xml
<listeners>

    <listener class-name="listeners.ExecutionStatisticsListener"/>

    <listener class-name="listeners.SuiteStatisticsListener"/>

    <listener class-name="listeners.ExecutionSummaryListener"/>

    <listener class-name="listeners.MethodInvocationListener"/>

    <listener class-name="listeners.CustomReporter"/>

</listeners>
```


## Enterprise Statistics Collection Flow

```text
ITestListener
    │
    ├── Pass Count
    ├── Fail Count
    └── Skip Count
             │
             ▼
      Statistics Manager
             │
   ┌─────────┼─────────┐
   ▼         ▼         ▼
ISuite   IExecution  IReporter
Listener Listener
   │         │         │
   ▼         ▼         ▼
Suite    Global     Final HTML
Summary  Summary    /Dashboard

```

