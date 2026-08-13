## IAnnotationTransformer

IAnnotationTransformer allows modification of TestNG annotations before test execution starts.


Use Cases:

 1. Auto Attach Retry Analyzer
 2. Dynamic Test Enable/Disable
 3. Modify Invocation Count
 4. Modify Thread Pool Size
 5. Change Test Timeout
 6. Convert Smoke Tests into Regression Tests
 7. Inject Test Groups
 8. Risk-Based Execution
 9. Dynamic Test Prioritization


Following example demonstrates adding "Custom Annotation" @TestInfo via IAnnotationTransformer  to a @Test method name starting with "Critical"

achieve the same business outcome by:

 - Detecting methods whose names start with "Critical"
 - Modifying TestNG's @Test annotation settings
 - Storing custom metadata in a framework registry/map
 - Treating the method as if it has a @TestInfo annotation

# Step 1: Custom Annotation

```java
package annotations;

import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface TestInfo {

    String owner() default "Framework";

    String module() default "General";

    String priority() default "MEDIUM";
}
```


# Step 2: Metadata Registry

Since we cannot attach the annotation physically, store equivalent metadata.

```java
package framework.metadata;

import java.util.concurrent.ConcurrentHashMap;
import java.util.Map;

public final class TestInfoRegistry {

	private static final Map<String, TestInfoData> REGISTRY = new ConcurrentHashMap<>();

	private TestInfoRegistry() {
	}

	public static void register(String methodName, TestInfoData data) {

		REGISTRY.put(methodName, data);
	}

	public static TestInfoData get(String methodName) {

		return REGISTRY.get(methodName);
	}
}
```


# Step 3: Metadata POJO

```java
package framework.metadata;

public class TestInfoData {

	private String owner;
	private String module;
	private String priority;

	public TestInfoData(String owner, String module, String priority) {

		this.owner = owner;
		this.module = module;
		this.priority = priority;
	}

	public String getOwner() {
		return owner;
	}

	public String getModule() {
		return module;
	}

	public String getPriority() {
		return priority;
	}
}
```


# Step 4: IAnnotationTransformer

```java
package listeners;

import java.lang.reflect.Constructor;
import java.lang.reflect.Method;

import org.testng.IAnnotationTransformer;
import org.testng.annotations.ITestAnnotation;

import framework.metadata.TestInfoData;
import framework.metadata.TestInfoRegistry;

public class CriticalTestTransformer implements IAnnotationTransformer {

	@Override
	public void transform(ITestAnnotation annotation, Class testClass, Constructor constructor, Method method) {

		if (method == null) {
			return;
		}

		if (method.getName().startsWith("Critical")) {

			// Modify TestNG behavior

			annotation.setTimeOut(60000);

			annotation.setPriority(1);

			// Register virtual TestInfo

			TestInfoRegistry.register(method.getName(), new TestInfoData("Framework", "Critical Flow", "HIGH"));

			System.out.println("[Transformer] TestInfo added to " + method.getName());
		}
	}
}

```

# Step 5: Test Class

Notice there is no @TestInfo annotation.

```java
public class LoginTest {

    @Test
    public void CriticalVerifyLogin() {

        System.out.println("Executing Critical Login");
    }

    @Test
    public void verifyLogout() {

        System.out.println("Executing Logout");
    }
}
```


# Step 6: Read Virtual Annotation in Listener

```java
public class TestInfoListener implements ITestListener {

	@Override
	public void onTestStart(ITestResult result) {

		String methodName = result.getMethod().getMethodName();

		TestInfoData data = TestInfoRegistry.get(methodName);

		if (data != null) {

			System.out.println("\n=== VIRTUAL TEST INFO ===");

			System.out.println("Owner    : " + data.getOwner());

			System.out.println("Module   : " + data.getModule());

			System.out.println("Priority : " + data.getPriority());
		}
	}
}

```

Execution Flow

```text

CriticalVerifyLogin() 
	- > IAnnotationTransformer 
		-> Method Name Starts With "Critical" 
			-> Register Virtual TestInfo 
				-> ITestListener

```

**Sample Output**

```text
[Transformer] TestInfo added to CriticalVerifyLogin

=== VIRTUAL TEST INFO ===
Owner    : Framework
Module   : Critical Flow
Priority : HIGH

Executing Critical Login

Executing Logout
```


