Validate @TestInfo.priority Against Allowed Values and Stop Test Execution

The goal is:

- Validate annotation values before test execution
- Validate values loaded from properties file
- Fail fast for invalid values
- Prevent execution of incorrectly tagged tests
- Ensure governance and reporting consistency


# Step 1. Enum definition
```java
public enum Priority {
    CRITICAL,
    HIGH,
    MEDIUM,
    LOW
}
```

# Step 2. Annotation:
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface TestInfo {
    // define annotation fields and their default values; 	
    Priority[] priority() default {Priority.MEDIUM};
}
```

# Step 3: Properties File
```text
default.author=FrameworkOwner
default.module=General
default.jiraId=NOT-ASSIGNED
default.priority=MEDIUM,REGRESSION
```

# Step 4: Create Validation Exception
```java
package framework.exceptions;

public class InvalidPriorityException extends RuntimeException {

    public InvalidPriorityException(String message) {
        super(message);
    }
}
```

# Step 6: Property reader (specific for Priority enum)
```java
public final class PriorityUtils {

    private PriorityUtils() {
    }

public static Priority[] getPrioritiesFromProperty(
        String value) {

    if (value == null ||
        value.isBlank()) {

        throw new InvalidPriorityException(
                "Property default.priority is missing.");
    }

    try {

        return Arrays.stream(value.split(","))
                .map(String::trim)
                .map(String::toUpperCase)
                .map(Priority::valueOf)
                .toArray(Priority[]::new);

    } catch (IllegalArgumentException ex) {

        throw new InvalidPriorityException(
                "Invalid Priority configured in property file : "
                        + value);
    }
}

}
```


# Step 5: Priority Validator

Since all values are already validated by the compiler, the validator mainly checks for null or empty configurations.

```java
package framework.validation;

import framework.constants.Priority;
import framework.exceptions.InvalidPriorityException;

public final class PriorityValidator {

    private PriorityValidator() {
    }

    public static void validate(
            Priority[] priorities) {

        if (priorities == null ||
            priorities.length == 0) {

            throw new InvalidPriorityException(
                    "At least one Priority must be specified.");
        }

        for (Priority priority : priorities) {

            if (priority == null) {

                throw new InvalidPriorityException(
                        "Null Priority detected.");
            }
        }
    }
}
```

# Step x:  Validation Listener (Preferred Implementation)

```java
package listeners;

import java.lang.reflect.Method;

import org.testng.ITestListener;
import org.testng.ITestResult;
import org.testng.SkipException;

import framework.exceptions.InvalidPriorityException;
import framework.metadata.TestMetadata;
import framework.metadata.TestMetadataResolver;

public class AnnotationValidationListener
        implements ITestListener {

    @Override
    public void onTestStart(
            ITestResult result) {

        Method method =
                result.getMethod()
                        .getConstructorOrMethod()
                        .getMethod();

        try {

            TestMetadata metadata =
                    TestMetadataResolver
                            .resolve(method);

            result.setAttribute(
                    "TEST_METADATA",
                    metadata);

        } catch (InvalidPriorityException e) {

            throw new SkipException(
                    "\nTest Execution Blocked\n"
                            + "Method : "
                            + method.getName()
                            + "\nReason : "
                            + e.getMessage());
        }
    }
}
```

# Step 6. Test usage:

Valid ✅

```java
@TestInfo(
        author = "Sheetal",
        module = "Claims",
        jiraId = "QA-101",
	priority = {Priority.MEDIUM, Priority.LOW}
)
public void verifyClaimCreation() {

}
```
Invalid ❌

```java
@TestInfo(
        author = "Sheetal",
        module = "Claims",
        jiraId = "QA-101",
        priority = {
                Priority.HIGH,
                Priority.URGENT
        })
public void verifyClaimCreation() {

}
```
Output:  **Compilation failure**
- This will fail during compilation.   
- TestNG will never start.  
- Listener will never execute. 
- MetadataResolver will never execute.

**Actual compiler error**
-The enum Priority has no constant URGENT


# Step 7: Validate During Metadata Resolution

public static TestMetadata resolve(
        Method method) {

    TestMetadata metadata =
            new TestMetadata();

    Priority[] priorities;

    if (method.isAnnotationPresent(
            TestInfo.class)) {

        TestInfo info =
                method.getAnnotation(
                        TestInfo.class);

        priorities = info.priority();

    } else {

        priorities =
                PriorityUtils
                        .getPrioritiesFromProperty(
                                propertyLoader.get(
                                        "default.priority"));
    }

    PriorityValidator.validate(
            priorities);

    metadata.setPriorities(
            Arrays.asList(priorities));

    return metadata;
}
```

# Step 8: Stop Execution in Listener

**Option A (Preferred)**: Skip Test Execution
@Override
public void onTestStart(
        ITestResult result) {

    Method method =
            result.getMethod()
                  .getConstructorOrMethod()
                  .getMethod();

    try {

        TestMetadata metadata =
                TestMetadataResolver
                        .resolve(method);

        result.setAttribute(
                "TEST_METADATA",
                metadata);

    }
    catch (InvalidPriorityException ex) {

        throw new SkipException(
                "Test Execution Blocked"
                + "\nMethod : "
                + method.getName()
                + "\nReason : "
                + ex.getMessage());
    }
}


Output:

SKIPPED

Reason:
Invalid Priority : URGENT



**Option B: Hard Fail Test**
@Override
public void onTestStart(
        ITestResult result) {

    Method method =
            result.getMethod()
                    .getConstructorOrMethod()
                    .getMethod();

    TestMetadataResolver
            .resolve(method);
}

Output:

FAILED

Invalid Priority : URGENT




# Improved Validation Responsibility Matrix

| Validation Scenario | Validator / Responsible Component |
|---------------------|-----------------------------------|
| `Priority.HIGH` in annotation | Java Compiler |
| `Priority.MEDIUM` in annotation | Java Compiler |
| `Priority.URGENT` in annotation | Java Compiler (Compile Failure) |
| Missing `default.priority` property | `PriorityUtils` |
| Blank property value | `PriorityUtils` |
| Invalid property value (`URGENT`) | `PriorityUtils` |
| Null priority array | `PriorityValidator` |
| Empty priority array | `PriorityValidator` |
| Null enum instance | `PriorityValidator` |
| Metadata storage in `ITestResult` | Listener |

