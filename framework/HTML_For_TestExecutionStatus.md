A scalable Selenium Java framework should separate **test execution**, **metrics collection**, and **report generation**. Instead of relying only on Extent Reports, create a **custom execution dashboard** that generates an HTML summary after every suite execution.

For your requirements, the framework should capture:

| Metric           | Description                         |
| ---------------- | ----------------------------------- |
| Total Test Cases | Total tests discovered in the suite |
| Executed         | Tests that actually ran             |
| Passed           | Successfully completed tests        |
| Failed           | Failed tests                        |
| Skipped          | Skipped/ignored tests               |
| Pending          | Total − Executed                    |
| Pass Percentage  | Passed / Executed × 100             |
| Fail Percentage  | Failed / Executed × 100             |
| Execution Time   | Overall suite duration              |

---

# Recommended Architecture

```text
Automation Framework
│
├── BaseTest
├── DriverFactory
├── Listeners
│      ├── TestExecutionListener.java
│      ├── SuiteListener.java
│
├── Metrics
│      ├── ExecutionMetrics.java
│      ├── MetricsManager.java
│      ├── HtmlReportGenerator.java
│      └── JsonExporter.java
│
├── reports
│      dashboard.html
│      metrics.json
│      execution.csv
│
└── logs
```

---

# Step 1 – Metrics Class

```java
public class ExecutionMetrics {

    private int totalTests;

    private int executed;

    private int passed;

    private int failed;

    private int skipped;

    public int getPending() {
        return totalTests - executed;
    }

}
```

---

# Step 2 – Capture Metrics Using TestNG

```java
public class TestExecutionListener
        implements ITestListener,
                   ISuiteListener {

    @Override
    public void onTestSuccess(ITestResult result){

        MetricsManager.incrementExecuted();
        MetricsManager.incrementPassed();

    }

    @Override
    public void onTestFailure(ITestResult result){

        MetricsManager.incrementExecuted();
        MetricsManager.incrementFailed();

    }

    @Override
    public void onTestSkipped(ITestResult result){

        MetricsManager.incrementExecuted();
        MetricsManager.incrementSkipped();

    }

}
```

---

# Step 3 – Count Total Tests

```java
@Override
public void onStart(ISuite suite){

    int total = suite.getAllMethods().size();

    MetricsManager.setTotalTests(total);

}
```

---

# Step 4 – Metrics Manager

```java
public class MetricsManager {

    private static AtomicInteger total =
            new AtomicInteger();

    private static AtomicInteger executed =
            new AtomicInteger();

    private static AtomicInteger passed =
            new AtomicInteger();

    private static AtomicInteger failed =
            new AtomicInteger();

    private static AtomicInteger skipped =
            new AtomicInteger();

}
```

---

# Step 5 – Generate HTML Report

Use a simple HTML template and replace placeholders.

```java
public class HtmlReportGenerator {

    public static void generate() throws IOException {

        String html = """
        <html>

        <head>

        <title>Automation Dashboard</title>

        </head>

        <body>

        <h1>Execution Summary</h1>

        <table border='1'>

        <tr>

        <th>Total Tests</th>

        <th>Executed</th>

        <th>Passed</th>

        <th>Failed</th>

        <th>Skipped</th>

        <th>Pending</th>

        </tr>

        <tr>

        <td>%d</td>

        <td>%d</td>

        <td>%d</td>

        <td>%d</td>

        <td>%d</td>

        <td>%d</td>

        </tr>

        </table>

        </body>

        </html>
        """.formatted(
                MetricsManager.getTotal(),
                MetricsManager.getExecuted(),
                MetricsManager.getPassed(),
                MetricsManager.getFailed(),
                MetricsManager.getSkipped(),
                MetricsManager.getPending());

        Files.writeString(
                Path.of("reports/dashboard.html"),
                html);

    }

}
```

---

# Example HTML Dashboard

```text
------------------------------------------------------

Automation Execution Dashboard

------------------------------------------------------

Total Test Cases       : 500

Executed               : 485

Passed                 : 462

Failed                 : 18

Skipped                : 5

Pending                : 15

Pass Percentage        : 95.25 %

Fail Percentage        : 3.71 %

Execution Time         : 1 hr 42 min

------------------------------------------------------
```

---

# Better HTML Design

Use CSS cards for a modern dashboard.

```html
<div class="card total">
    <h2>500</h2>
    <p>Total Tests</p>
</div>

<div class="card pass">
    <h2>462</h2>
    <p>Passed</p>
</div>

<div class="card fail">
    <h2>18</h2>
    <p>Failed</p>
</div>

<div class="card pending">
    <h2>15</h2>
    <p>Pending</p>
</div>
```

This creates a dashboard similar to Jenkins or Azure DevOps test summaries.

---

# Execution Flow

```text
Suite Starts
      │
      ▼
Count Total Tests
      │
      ▼
Execute Test
      │
      ├── Pass
      ├── Fail
      └── Skip
      │
      ▼
Update Metrics
      │
      ▼
Suite Finished
      │
      ▼
Generate HTML Dashboard
```

---

# Sample Dashboard Layout

```text
+------------------------------------------------------+
|             Selenium Automation Dashboard            |
+------------------------------------------------------+

+------------+------------+------------+--------------+
| Total      | Executed   | Pending    | Pass Rate    |
| 500        | 485        | 15         | 95.25 %      |
+------------+------------+------------+--------------+

+------------+------------+------------+
| Passed     | Failed     | Skipped    |
| 462        | 18         | 5          |
+------------+------------+------------+

+-----------------------------------------------+
| Suite Execution Time : 1 hr 42 min            |
+-----------------------------------------------+

+-----------------------------------------------+
| Longest Running Test : LoginTest              |
| Duration : 45 sec                             |
+-----------------------------------------------+

+-----------------------------------------------+
| Top 5 Failed Tests                            |
| LoginTest                                     |
| PaymentTest                                   |
| CheckoutTest                                  |
+-----------------------------------------------+
```

---

# Enterprise Enhancements

For production-grade frameworks, consider enriching the dashboard with:

* **Pie chart** showing Pass / Fail / Skip distribution using Chart.js.
* **Bar chart** for execution time by test class.
* **Trend chart** comparing metrics across previous executions.
* **Environment details** (browser, OS, Java version, environment, Git commit).
* **Execution metadata** (suite name, execution start/end time, CI build number).
* **Links to screenshots** and failure stack traces for failed tests.
* **Drill-down tables** to navigate from suite → class → test method.

This approach gives you a lightweight, self-contained HTML dashboard while remaining independent of third-party reporting tools like Extent Reports, and it can be easily integrated into Jenkins, Azure DevOps, or GitHub Actions pipelines.
