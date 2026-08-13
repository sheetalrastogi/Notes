# Filter data via "IDataProviderInterceptor" listener 

example to execute only data rows where type = "SMOKE" 


# DataProvider

```java
@DataProvider(name = "testData")
public Object[][] testData() {

    return new Object[][]{
            {"TC001", "SMOKE"},
            {"TC002", "REGRESSION"},
            {"TC003", "SMOKE"}
    };
}
```


# IDataProviderInterceptor Implementation

This interceptor filters the DataProvider data and allows only rows where the second column (type) equals "SMOKE".

```java
package listeners;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

import org.testng.IDataProviderInterceptor;
import org.testng.IDataProviderMethod;
import org.testng.ITestContext;
import org.testng.ITestNGMethod;

public class SmokeDataProviderInterceptor implements IDataProviderInterceptor {

	@Override
	public Iterator<Object[]> intercept(Iterator<Object[]> original, IDataProviderMethod dataProviderMethod,
			ITestNGMethod testMethod, ITestContext context) {

		List<Object[]> filteredData = new ArrayList<>();

		while (original.hasNext()) {

			Object[] row = original.next();
			String testId = String.valueOf(row[0]);
			String type = String.valueOf(row[1]);
			if ("SMOKE".equalsIgnoreCase(type)) {
				filteredData.add(row);

			} else {
				System.out.println("Filtered : " + testId + " [" + type + "]");
			}
		}

		return filteredData.iterator();
	}
}
```


# Test Method

```java
@Test(dataProvider = "testData")
public void executeTest(String testCaseId, String type) {

    System.out.println("Executing : " + testCaseId);
}
```


# Register Listener

**Option 1: testng.xml**

```xml
<listeners>
    <listener class-name="listeners.SmokeDataProviderInterceptor"/>
</listeners>
```

**Option 2: Service Loader**

Step 1: Create file:
    src/test/resources/META-INF/services/org.testng.ITestNGListener

Step 2: Add Content to above file 
    listeners.SmokeDataProviderInterceptor


**Execution Result (output)**:
    Executing : TC001
    Executing : TC003


## Similarly - IDataProviderInterceptor implementation that uses both:

IDataProviderMethod → Get DataProvider details
ITestNGMethod → Get Test Method details

and logs:

- Test Class
- Test Method Name
- DataProvider Name
- Number of Records Processed
- Included/Excluded Rows

1. DataProvider

```java
@DataProvider(name = "testData")
public Object[][] testData() {

    return new Object[][]{

            {"TC001", "SMOKE"},
            {"TC002", "REGRESSION"},
            {"TC003", "SMOKE"},
            {"TC004", "SANITY"}
    };
}
```

2. TestNG Method

```java
@Test(dataProvider = "testData")
public void executeTest(String testCaseId, String type) {
    System.out.println("Executing : " + testCaseId + " Type : " + type);
}
```

3. IDataProviderInterceptor implementation

```java
package listeners;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

import org.testng.IDataProviderInterceptor;
import org.testng.IDataProviderMethod;
import org.testng.ITestContext;
import org.testng.ITestNGMethod;

public class SmokeDataProviderInterceptor implements IDataProviderInterceptor {

	@Override
	public Iterator<Object[]> intercept(Iterator<Object[]> original, IDataProviderMethod dataProviderMethod,
			ITestNGMethod testMethod, ITestContext context) {

		String dataProviderName = dataProviderMethod.getName();

		String testMethodName = testMethod.getMethodName();

		String testClass = testMethod.getRealClass().getSimpleName();

		System.out.println("\n========== IDataProviderInterceptor ==========");

		System.out.println("Test Class      : " + testClass);

		System.out.println("Test Method     : " + testMethodName);

		System.out.println("DataProvider    : " + dataProviderName);

		List<Object[]> filteredData = new ArrayList<>();

		int totalRecords = 0;
		int selectedRecords = 0;

		while (original.hasNext()) {

			Object[] row = original.next();

			totalRecords++;

			String testCaseId = String.valueOf(row[0]);

			String executionType = String.valueOf(row[1]);

			if ("SMOKE".equalsIgnoreCase(executionType)) {

				filteredData.add(row);

				selectedRecords++;

				System.out.println("[INCLUDED] " + testCaseId + " , " + executionType);

			} else {

				System.out.println("[FILTERED] " + testCaseId + " , " + executionType);
			}
		}

		System.out.println("Total Records   : " + totalRecords);

		System.out.println("Selected Records: " + selectedRecords);

		System.out.println("==============================================\n");

		return filteredData.iterator();
	}
}
```


Output:

```text
========== IDataProviderInterceptor ==========
Test Class      : LoginTest
Test Method     : executeTest
DataProvider    : testData

[INCLUDED] TC001 , SMOKE
[FILTERED] TC002 , REGRESSION
[INCLUDED] TC003 , SMOKE
[FILTERED] TC004 , SANITY

Total Records   : 4
Selected Records: 2
==============================================

Executing : TC001 Type : SMOKE
Executing : TC003 Type : SMOKE
```


