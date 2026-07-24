**First Pass Yield (FPY)** is one of the most valuable metrics for measuring the effectiveness and stability of an automation framework. It tells you how many test cases **passed on their very first execution without requiring any retry**.

Unlike pass rate, FPY is **not inflated by retries**, making it an excellent KPI for enterprise CI/CD pipelines.

---

# What is First Pass Yield (FPY)?

### Formula

```text
FPY (%) =

(Number of Test Cases Passed in First Attempt)
------------------------------------------------ × 100
(Total Test Cases Executed)
```

Example

```text
Total Tests = 100

Passed First Attempt = 92

Failed First Attempt = 8

Retries = 8

Retry Passed = 6

Final Pass = 98
```

Results

```text
Final Pass Rate

98%

First Pass Yield

92%
```

Notice the difference:

| Metric           | Value |
| ---------------- | ----: |
| Final Pass Rate  |   98% |
| First Pass Yield |   92% |

A high pass rate with a low FPY often indicates flaky tests or environmental instability.

---

# Enterprise Architecture

```text
Automation Framework
│
├── RetryAnalyzer
├── MetricsManager
├── FPYManager
├── TestNG Listener
│
├── reports
│      metrics.csv
│      metrics.json
│      fpy.csv
│
└── dashboard
```

---

# Step 1 – Capture Test Attempts

Create a class to track the number of attempts for each test.

```java
public class FPYManager {

    private static Map<String, Integer> attempts =
            new ConcurrentHashMap<>();

    public static void incrementAttempt(String testName){

        attempts.merge(testName,1,Integer::sum);

    }

    public static int getAttempt(String testName){

        return attempts.getOrDefault(testName,1);

    }

}
```

---

# Step 2 – Update Retry Analyzer

```java
public class RetryAnalyzer implements IRetryAnalyzer {

    private int retry = 0;

    private static final int MAX_RETRY = 2;

    @Override
    public boolean retry(ITestResult result) {

        FPYManager.incrementAttempt(
                result.getMethod().getMethodName());

        if(retry < MAX_RETRY){

            retry++;

            return true;

        }

        return false;

    }

}
```

---

# Step 3 – Track First Attempt Passes

```java
public class MetricsListener
        implements ITestListener {

    @Override
    public void onTestSuccess(
            ITestResult result) {

        String name =
                result.getMethod().getMethodName();

        if(FPYManager.getAttempt(name) == 1){

            MetricsManager.incrementFPY();

        }

    }

}
```

---

# Step 4 – Store Metrics

```java
public class MetricsManager {

    private static AtomicInteger totalTests =
            new AtomicInteger();

    private static AtomicInteger firstPass =
            new AtomicInteger();

    public static void incrementTotal(){

        totalTests.incrementAndGet();

    }

    public static void incrementFPY(){

        firstPass.incrementAndGet();

    }

}
```

---

# Step 5 – Calculate FPY

```java
public static double calculateFPY(){

    return

            (double)

            firstPass.get()

            /

            totalTests.get()

            *100;

}
```

---

# Step 6 – Print Metrics

```java
Logger.info(

"First Pass Yield : "

+

String.format("%.2f",

MetricsManager.calculateFPY())

+

"%"

);
```

Output

```text
First Pass Yield : 94.72 %
```

---

# Step 7 – Export to CSV

```csv
Metric,Value
Total Tests,520
Passed First Attempt,491
Retry Passed,21
Failed,8
FPY,94.42%
```

---

# Step 8 – Dashboard Example

```text
=====================================

Automation Quality Metrics

=====================================

Total Tests              : 520

Passed                   : 512

Failed                   : 8

Retry Passed             : 21

Retry Failed             : 4

First Pass Yield         : 94.42 %

Final Pass Rate          : 98.46 %

Retry Rate               : 4.04 %

Flaky Tests              : 18

=====================================
```

---

# Trend Analysis

Track FPY across builds to identify framework stability.

| Build | FPY | Final Pass Rate |
| ----: | --: | --------------: |
|   101 | 89% |             96% |
|   102 | 91% |             97% |
|   103 | 94% |             98% |
|   104 | 96% |             98% |
|   105 | 98% |             99% |

A rising FPY generally indicates improved application quality and more stable automation.

---

# Related Metrics

To make FPY more actionable, calculate these alongside it:

| Metric                     | Formula                          | Purpose                       |
| -------------------------- | -------------------------------- | ----------------------------- |
| First Pass Yield           | First-pass passes / Total tests  | Measures automation stability |
| Final Pass Rate            | Final passed / Total tests       | Overall execution success     |
| Retry Rate                 | Retried tests / Total tests      | Indicates need for reruns     |
| Flaky Test Rate            | Retry-passed tests / Total tests | Identifies unstable tests     |
| Automation Stability Index | FPY × Pass Rate                  | Overall framework health      |
| Mean Retry Count           | Total retries / Retried tests    | Retry effectiveness           |

---

# Enterprise Design Recommendation

For large Selenium frameworks, separate the concerns of execution, retry tracking, and metrics collection:

```text
TestNG
   │
   ▼
RetryAnalyzer
   │
   ▼
FPYManager
   │
   ├── Attempt Tracker
   ├── First Pass Counter
   ├── Retry Counter
   ├── Flaky Test Detector
   └── Retry Statistics
           │
           ▼
MetricsManager
   │
   ├── CSV Export
   ├── JSON Export
   ├── Extent Report
   ├── Grafana / Prometheus
   └── Power BI Dashboard
```

## Best Practices

* Count **only the initial execution** when calculating FPY.
* Do **not** include skipped tests unless your organization's reporting policy requires it.
* Track FPY at multiple levels:

  * Suite
  * Test class
  * Test method
  * CI/CD pipeline
* Publish FPY trends over time rather than focusing on a single build.
* Pair FPY with **Retry Rate** and **Flaky Test Rate** to quickly identify whether failures are caused by application defects or automation/environment instability.

For enterprise Selenium frameworks, an FPY target of **95% or higher** is typically considered a strong indicator of a stable automation suite.
