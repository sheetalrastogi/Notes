Create a **Custom Annotation** with TestNG based frameworks 

This is useful when you want to attach metadata to test methods, such as:
- Test Owner
- Requirement ID
- Jira Story
- Module Name
- Priority
- Retry Policy
- Risk Level
- Automation Type

Following are logical steps you can read the annotation values using an **ITestListener** or  **IInvokedMethodListener**.


# Step 1: Create Custom Annotation
```java
package annotations;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.annotation.ElementType;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface TestInfo {

	String author();

	String module();

	String jiraId() default "";

	String priority() default "MEDIUM";
}

```
# Step 2: Use Annotation in Test Class
```java
package tests;

import org.testng.Assert;
import org.testng.annotations.Test;

import annotations.TestInfo;

public class LoginTest {

	@Test
	@TestInfo(author = "Sheetal", module = "Login", jiraId = "QA-1001", priority = "HIGH")
	public void verifyValidLogin() {

		System.out.println("Executing Login Test");
		Assert.assertTrue(true);
	}

	@Test
	@TestInfo(author = "Sheetal", module = "Login", jiraId = "QA-1002", priority = "CRITICAL")
	public void verifyInvalidLogin() {

		System.out.println("Executing Invalid Login Test");
		Assert.assertTrue(true);
	}
}
```

# Step 3: Read Annotation Using ITestListener

```java
package listeners;

import java.lang.reflect.Method;

import org.testng.ITestListener;
import org.testng.ITestResult;

import annotations.TestInfo;

public class CustomAnnotationListener implements ITestListener {

	@Override
	public void onTestStart(ITestResult result) {

		Method method = result.getMethod().getConstructorOrMethod().getMethod();

		if (method.isAnnotationPresent(TestInfo.class)) {

			TestInfo info = method.getAnnotation(TestInfo.class);

			System.out.println("========== TEST INFO ==========");
			System.out.println("Test Name : " + method.getName());
			System.out.println("Author    : " + info.author());
			System.out.println("Module    : " + info.module());
			System.out.println("Jira ID   : " + info.jiraId());
			System.out.println("Priority  : " + info.priority());
			System.out.println("================================");
		}
	}
}

```

# Step 4: Register Listener

**Option 1: testng.xml**
```xml
<listeners>
    <listener class-name="listeners.CustomAnnotationListener"/>
</listeners>
```

**Option 2: Annotation**
```java
@Listeners(CustomAnnotationListener.class)
public class LoginTest {
}
```

**Sample Output**

```text
========== TEST INFO ==========
Test Name : verifyValidLogin
Author    : Sheetal
Module    : Login
Jira ID   : QA-1001
Priority  : HIGH
================================

Executing Login Test
```

---


## Custom annotations can drive:

- Execution behavior
- Browser selection
- Retry policies
- Reporting
- Traceability
- Defect management
- Governance compliance
- AI-agent invocation
- Test evidence collection
- Risk-based regression

## Selenium + TestNG Custom Annotations Classification Matrix

| # | Custom Annotation | Objective | Purpose |
|---|---|---|---|
| 1 | Retry Control | Control retry behavior at test level | Retry failed tests; Override default retry count |
| 2 | Browser Selection | Define browser per test | Execute on specific browser; Override suite browser |
| 3 | Requirement Traceability | Map tests to requirements | RTM generation; Compliance audits; Coverage tracking |
| 4 | Defect Mapping | Associate tests with known defects | Known bug tracking; Coverage gap analysis |
| 5 | Risk-Based Execution | Classify business risk | Prioritize testing; Risk-based regression |
| 6 | Test Owner Assignment | Identify accountable engineer | Ownership tracking; Failure notifications |
| 7 | Module / Feature Tagging | Group tests by business area | Module-wise execution; Report filtering |
| 8 | Environment Restriction | Control execution environments | Restrict environments; Block PROD execution |
| 9 | Smoke/Sanity/Regression Classification | Categorize test pack | Dynamic suite creation; Selective execution |
| 10 | Feature Toggle Control | Link tests to feature flags | Execute only when feature enabled |
| 11 | Data Source Selection | Specify test data source | Dynamic data loading; Dataset management |
| 12 | Parallel Execution Eligibility | Mark parallel-safe tests | Prevent thread conflicts; Improve stability |
| 13 | Execution Priority | Define business execution order | Run critical tests first |
| 14 | SLA / Timeout Specification | Set custom timeout | Control long-running tests; SLA validation |
| 15 | Security Test Marker | Identify security validation tests | Security suite filtering; Audit reporting |
| 16 | Test Type Classification | Categorize automation type | Separate pipelines; Execution filtering |
| 17 | Performance Benchmark Metadata | Define expected performance target | SLA validation; Performance assertions |
| 18 | Dependency Annotation | Define business dependencies | Workflow orchestration; Dependency tracking |
| 19 | Auto-Healing Control | Control self-healing behavior | Enable AI locator healing; Per-test configuration |
| 20 | Screenshot Capture Policy | Configure screenshot generation | Evidence collection; Failure analysis |
| 21 | Video Recording Policy | Configure session recording | Execution playback; Failure debugging |
| 22 | Accessibility Testing Marker | Tag accessibility validation tests | Accessibility suite execution; Compliance reporting |
| 23 | Flaky Test Tracking | Identify unstable tests | Flaky test analysis; Stability metrics |
| 24 | Production Safe Test | Mark PROD-executable tests | Prevent destructive actions; Safe production validation |
| 25 | AI-Agent Driven Execution | Invoke framework agents dynamically | AI-driven execution; Intelligent automation |

---

