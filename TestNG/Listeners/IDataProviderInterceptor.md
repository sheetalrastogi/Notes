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



