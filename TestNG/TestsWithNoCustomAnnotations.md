## Test methods without Custom Annotations - to be processed per Property configuration

Create a custom annotation such that if @Test method is not annotated with "Custom Annotation" class add Custom annotation - TestInfo with values read from properties file and process test execution as if the test method was annotated with TestInfo annotation

**Java annotations** are immutable at runtime, you cannot physically add @TestInfo to a method after compilation. However, in an enterprise Selenium-TestNG framework you can implement a Virtual Annotation Injection Pattern where:

```text
                 TestNG Execution
                         │
                         ▼
              CustomAnnotationListener
                         │
                         ▼
             Is @TestInfo Available?
                    /         \
                  Yes          No
                   │            │
                   ▼            ▼
          Read Annotation    Read Defaults
              Values       From Properties File
                   │            │
                   └──────┬─────┘
                          ▼
                  TestMetadata Object
                          │
                          ▼
             Custom logic configuration for Retry / Reporting / Logging etc
                          │
                          ▼
                   Test Execution
```

# Assigning Custom Annotations to @Test methods (even if they are not configured at test Level)

# Step 1: TestInfo Annotation

```java
package annotations;

import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface TestInfo {

	String author();

	String module();

	String jiraId() default "";

	String[] priority() default { "MEDIUM" };
}
```

# Step 2: Properties File

testinfo.properties

```text
default.author=FrameworkOwner
default.module=General
default.jiraId=NOT-ASSIGNED
default.priority=MEDIUM,REGRESSION
```


# Step 3: Metadata Class

This class represents either:
- Actual annotation values
- Default values from properties

```java
package framework.metadata;

import java.util.List;

public class TestMetadata {

	private String author;
	private String module;
	private String jiraId;
	private List<String> priorities;
	
	//getters & setters  eg

	public String getAuthor() {
		return author;
	}

	public void setAuthor(String author) {
		this.author = author;
	}

	…..

	public List<String> getPriorities() {
		return priorities;
	}

	public void setPriorities(List<String> priorities) {
		this.priorities = priorities;
	}
}
```


# Step 4: Property Loader Utility

```java
package framework.utils;

import java.io.InputStream;
import java.util.Properties;

public class TestInfoPropertyLoader {

	private static final Properties properties = new Properties();

	static {

		try (InputStream is = TestInfoPropertyLoader.class.getClassLoader()
				.getResourceAsStream("testinfo.properties")) {

			properties.load(is);

		} catch (Exception e) {

			throw new RuntimeException("Unable to load testinfo.properties", e);
		}
	}

	public static String get(String key) {

		return properties.getProperty(key);
	}
}
```

# Step 5: Metadata Resolver

Central place that behaves like a virtual annotation injector.

```java
package framework.metadata;

import annotations.TestInfo;
import framework.utils.TestInfoPropertyLoader;

import java.lang.reflect.Method;
import java.util.Arrays;

public class TestMetadataResolver {

	public static TestMetadata resolve(Method method) {

		TestMetadata metadata = new TestMetadata();

		if (method.isAnnotationPresent(TestInfo.class)) {

			TestInfo info = method.getAnnotation(TestInfo.class);

			metadata.setAuthor(info.author());
			metadata.setModule(info.module());
			metadata.setJiraId(info.jiraId());

			metadata.setPriorities(Arrays.asList(info.priority()));

		} else {

			metadata.setAuthor(TestInfoPropertyLoader.get("default.author"));

			metadata.setModule(TestInfoPropertyLoader.get("default.module"));

			metadata.setJiraId(TestInfoPropertyLoader.get("default.jiraId"));

			metadata.setPriorities(Arrays.asList(TestInfoPropertyLoader.get("default.priority").split(",")));
		}

		return metadata;
	}
}
```

# Step 6: Listener

```java
package listeners;

import framework.metadata.TestMetadata;
import framework.metadata.TestMetadataResolver;

import java.lang.reflect.Method;

import org.testng.ITestListener;
import org.testng.ITestResult;

public class CustomAnnotationListener implements ITestListener {

	@Override
	public void onTestStart(ITestResult result) {

		Method method = result.getMethod().getConstructorOrMethod().getMethod();

		TestMetadata metadata = TestMetadataResolver.resolve(method);

		System.out.println("\n====== TEST INFO ======");

		System.out.println("Method      : " + method.getName());

		System.out.println("Author      : " + metadata.getAuthor());

		System.out.println("Module      : " + metadata.getModule());

		System.out.println("Jira ID     : " + metadata.getJiraId());

		System.out.println("Priorities  : " + metadata.getPriorities());

		System.out.println("========================");
	}
}
```

# Step 7: Test With Annotation

```java
	@Test
	@TestInfo(author = "xxx", module = "Claims", jiraId = "QA-2001", priority = { "HIGH", "SMOKE" })
	public void verifyClaimCreation() {

	}
```

output:
```text
Method      : verifyClaimCreation
Author      : xxx
Module      : Claims
Jira ID     : QA-2001
Priorities  : [HIGH, SMOKE]
```


# Step 7a: Test without Annotation

```java
@Test
public void verifyCustomerSearch() {

}
```

Output:
```text
Method      : verifyCustomerSearch
Author      : FrameworkOwner
Module      : General
Jira ID     : NOT-ASSIGNED
Priorities  : [MEDIUM, REGRESSION]
```


