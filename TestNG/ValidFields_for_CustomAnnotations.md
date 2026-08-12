# Example: TestNG Custom Annotation Using Multiple Annotation Data Types

# Step 1: Enum Definition
```java
package annotations;
public enum Priority {
    CRITICAL,
    HIGH,
    MEDIUM,
    LOW
}
```

# Step 2: Custom Annotation

The following annotation demonstrates:
- Primitive Types: Boolean, byte, short, int, long, float, double, char, 
- String
- Enum
- Arrays of Primitive Types
- Arrays of Strings
- Arrays of Enums

```java
package annotations;

import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface FrameworkConfig {

	/* Primitive Types */

	boolean enabled() default true;

	byte retryCount() default 1;

	short executionOrder() default 1;

	int timeoutSeconds() default 30;

	long maxExecutionTimeMs() default 300000L;

	float passPercentage() default 90.0f;

	double threshold() default 99.99;

	char executionGroup() default 'A';

	/* String */

	String owner() default "Unknown";

	/* Enum */

	Priority priority() default Priority.MEDIUM;

	/* Primitive Arrays */

	int[] supportedWaits() default { 5, 10, 20 };

	boolean[] environmentsEnabled() default { true, false };

	/* String Array */

	String[] tags() default { "Regression" };

	/* Enum Array */

	Priority[] applicablePriorities() default { Priority.MEDIUM };
}
```

# Step 3: Usage in Selenium TestNG Test Method

```java
	@Test
	@FrameworkConfig(

			enabled = true,

			retryCount = 3,

			executionOrder = 10,

			timeoutSeconds = 60,

			maxExecutionTimeMs = 600000L,

			passPercentage = 95.5f,

			threshold = 99.999,

			executionGroup = 'P',

			owner = "Sheetal",

			priority = Priority.HIGH,

			supportedWaits = { 5, 15, 30 },

			environmentsEnabled = { true, true },

			tags = { "Smoke", "Claims", "UI" },

			applicablePriorities = { Priority.CRITICAL, Priority.HIGH })
	public void verifyClaimSubmission() {

		System.out.println("Executing claim submission test");
	}
```

# Step 4: Reading Values in Listener

```java
		Method method = result.getMethod().getConstructorOrMethod().getMethod();
		FrameworkConfig config = method.getAnnotation(FrameworkConfig.class);

		System.out.println("Owner      : " + config.owner());
		System.out.println("Priority   : " + config.priority());
		System.out.println("Retry Count: " + config.retryCount());
		System.out.println(Arrays.toString(config.tags()));
		System.out.println(Arrays.toString(config.applicablePriorities()));
```

Output
```text
Owner       : Sheetal
Priority    : HIGH
Retry Count : 3

Tags :
[Smoke, Claims, UI]

Applicable Priorities :
[CRITICAL, HIGH]
```

This example demonstrates all commonly supported annotation value types used in enterprise Selenium-TestNG frameworks.



# Annotation Value Types NOT Allowed
- set<String>
- Set<String>
- Map<String,String>
- HashMap
- ArrayList
- WebDriver
- By
- WebElement
- Date
- LocalDate
- LocalDateTime
- BigDecimal
- Object
- Interface instances

