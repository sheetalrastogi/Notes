You can enhance your Selenium Java automation framework by instrumenting it to capture execution timestamps at the **suite, test class, test method, and step levels**. The best approach is to implement a **Metrics Engine** that works alongside your logging and reporting framework (e.g., TestNG + Extent Reports + Log4j2).

## Recommended Architecture

```text
Automation Framework
│
├── DriverFactory
├── BaseTest
├── TestNG Listener
│      ├── MetricsListener.java
│      ├── SuiteMetrics.java
│      └── TestMetrics.java
│
├── Utilities
│      ├── MetricsManager.java
│      ├── ExecutionTimer.java
│      ├── ReportManager.java
│      └── Logger.java
│
├── reports
│      metrics.csv
│      metrics.json
│      execution.html
│
└── dashboard
       metrics.html
```

---

# 1. Total Suite Execution Time

### Formula

```text
Suite Execution Time =
Suite End Time - Suite Start Time
```

### Capture Using TestNG

```java
public class MetricsListener implements ISuiteListener {

    private long suiteStart;

    @Override
    public void onStart(ISuite suite) {
        suiteStart = System.currentTimeMillis();
    }

    @Override
    public void onFinish(ISuite suite) {

        long suiteEnd = System.currentTimeMillis();

        long duration = suiteEnd - suiteStart;

        MetricsManager.setSuiteExecutionTime(duration);

        Logger.info("Suite Execution Time : "
                + duration / 1000 + " sec");
    }
}
```

Example Output

```text
Suite Execution Time : 845 sec
```

---

# 2. Average Execution Time per Test Case

Capture execution time for every test method.

```java
public class MetricsListener implements ITestListener {

    private ThreadLocal<Long> startTime = new ThreadLocal<>();

    @Override
    public void onTestStart(ITestResult result) {

        startTime.set(System.currentTimeMillis());

    }

    @Override
    public void onTestSuccess(ITestResult result) {

        long end = System.currentTimeMillis();

        MetricsManager.addExecutionTime(
                end - startTime.get());

    }

}
```

MetricsManager

```java
private static List<Long> executionTimes = new ArrayList<>();

public static void addExecutionTime(long duration){

    executionTimes.add(duration);

}

public static double averageExecutionTime(){

    return executionTimes.stream()

            .mapToLong(Long::longValue)

            .average()

            .orElse(0);

}
```

Example

```text
Total Tests = 500

Total Execution Time = 18000 sec

Average Test Time

=18000 / 500

=36 sec
```

---

# 3. Test Execution Velocity (Throughput)

Throughput measures how many test cases complete in a unit of time.

### Formula

```text
Throughput

=

Total Tests Executed

/

Total Execution Time
```

Example

```text
800 Tests

400 Minutes

Throughput

=

2 Tests/Minute
```

Implementation

```java
double throughput =

(double) totalTests

/

(totalExecutionTime / 60000.0);

Logger.info("Throughput : "
        + throughput
        + " Tests/Minute");
```

Example Output

```text
Throughput

2.8 Tests/Minute
```

---

# 4. Test Execution Velocity (Per Hour)

You listed "Test Execution Velocity (Throughput)" twice. A complementary metric is throughput per hour.

```text
Velocity =

Executed Tests

/

Execution Hours
```

Example

```text
1000 Tests

5 Hours

Velocity

=

200 Tests/Hour
```

Implementation

```java
double velocity =

totalTests /

(totalExecutionTime / 3600000.0);
```

---

# Additional Metrics Worth Capturing

| Metric                 | Formula                         | Purpose                 |
| ---------------------- | ------------------------------- | ----------------------- |
| Pass Rate              | Passed / Total × 100            | Test quality            |
| Fail Rate              | Failed / Total × 100            | Stability               |
| Skip Rate              | Skipped / Total × 100           | Coverage                |
| Average Suite Duration | Total Suite Time / Suites       | Trend analysis          |
| Longest Running Test   | Max(Test Duration)              | Bottleneck detection    |
| Slow Test Count        | Tests > Threshold               | Optimization            |
| Fastest Test           | Min(Test Duration)              | Baseline                |
| Parallel Efficiency    | Sequential Time / Parallel Time | Parallelization benefit |
| Retry Rate             | Retried / Total                 | Test flakiness          |
| Automation Stability   | Passed First Attempt / Total    | Framework reliability   |

---

# Metrics Dashboard Example

```text
=============================================

Automation Metrics Dashboard

=============================================

Total Suites               : 5

Total Test Cases           : 825

Passed                     : 801

Failed                     : 19

Skipped                    : 5

Pass Rate                  : 97.1 %

Suite Execution Time       : 2h 34m

Average Test Duration      : 11.2 sec

Fastest Test               : 0.8 sec

Slowest Test               : 94 sec

Throughput                 : 321 Tests/Hour

Parallel Threads           : 8

CPU Utilization            : 71 %

Memory Peak                : 2.1 GB

Retries                    : 6

=============================================
```

---

# Export Metrics to CSV

```java
Metric,Value
SuiteExecutionTime,845 sec
AverageTestTime,11 sec
Throughput,321 Tests/Hour
PassRate,97.1
FailRate,2.9
FastestTest,0.8 sec
SlowestTest,94 sec
```

This CSV can be consumed by Excel, Power BI, Grafana, or CI/CD dashboards.

---

# Enterprise Design Recommendation

Introduce a dedicated **MetricsManager** service responsible for collecting and publishing metrics:

```text
MetricsManager
        │
        ├── ExecutionTimer
        ├── SuiteMetrics
        ├── TestMetrics
        ├── ParallelMetrics
        ├── RetryMetrics
        ├── SystemMetrics
        ├── CSVExporter
        ├── JSONExporter
        └── DashboardPublisher
```

The framework flow would be:

```text
TestNG Listener
       │
       ▼
MetricsManager
       │
       ├── Record start/end timestamps
       ├── Calculate durations
       ├── Compute throughput
       ├── Compute pass/fail statistics
       ├── Capture retry metrics
       └── Publish CSV/JSON/Extent Report
```

This design keeps metric collection centralized, thread-safe, and extensible, making it suitable for enterprise-scale Selenium frameworks that execute thousands of tests across multiple parallel nodes.
