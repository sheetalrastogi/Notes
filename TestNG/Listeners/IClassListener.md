IClassListener


IClassListener is invoked before and after execution of every Test Class

Example use cases:

1. Test Class Execution Logging
	Track start and end of every test class.
		Before Class : LoginTest
		After Class  : LoginTest

2. Class-Level Statistics Collection
	LoginTest
  	  Pass : 10
	  Fail : 1
	  Skip : 0

3. Initialize Class-Specific Resources
	Before LoginTest
	  Launch Browser
  	  Load Test Data
	  Create DB Connection
	After LoginTest
	  cleanup


Example Implementation
-------------------------

IClassListener Example: Launch Browser, Load Test Data, Create DB Connection Before LoginTest


## Step 1: Browser Manager

```java
package framework.driver;

public class DriverManager {

	public static void startBrowser() {

		System.out.println("[DriverManager] Launching Chrome Browser");
	}

	public static void closeBrowser() {

		System.out.println("[DriverManager] Closing Browser");
	}
}

```

## Step 2: Test Data Manager

```java
package framework.data;

public class TestDataManager {

	public static void loadTestData(String testClassName) {

		System.out.println("[TestDataManager] Loading data for " + testClassName);
	}

	public static void unloadTestData() {

		System.out.println("[TestDataManager] Releasing test data");
	}
}

```

## Step 3: Database Manager

```java
package framework.db;

public class DatabaseManager {

	public static void createConnection() {

		System.out.println("[DatabaseManager] Creating DB Connection");
	}

	public static void closeConnection() {

		System.out.println("[DatabaseManager] Closing DB Connection");
	}
}

```


## Step 4:  IClassListener Implementation


```java
package listeners;

import org.testng.IClass;
import org.testng.IClassListener;

import framework.data.TestDataManager;
import framework.db.DatabaseManager;
import framework.driver.DriverManager;
import tests.LoginTest;

public class LoginClassListener implements IClassListener {

	@Override
	public void onBeforeClass(IClass testClass) {

		if (testClass.getRealClass().equals(LoginTest.class)) {

			System.out.println("\n===== Before LoginTest =====");

			DriverManager.startBrowser();

			TestDataManager.loadTestData(testClass.getName());

			DatabaseManager.createConnection();
		}
	}

	@Override
	public void onAfterClass(IClass testClass) {

		if (testClass.getRealClass().equals(LoginTest.class)) {

			System.out.println("\n===== After LoginTest =====");

			DatabaseManager.closeConnection();

			TestDataManager.unloadTestData();

			DriverManager.closeBrowser();
		}
	}
}

```


# Step 5. Login Test

```java
package tests;

import org.testng.annotations.Listeners;
import org.testng.annotations.Test;

import listeners.LoginClassListener;

@Listeners(LoginClassListener.class)
public class LoginTest {

	@Test
	public void verifyValidLogin() {

		System.out.println("Executing Valid Login Test");
	}

	@Test
	public void verifyInvalidLogin() {

		System.out.println("Executing Invalid Login Test");
	}
}

```

Output:

```text
Sample Execution Output
===== Before LoginTest =====

[DriverManager] Launching Chrome Browser
[TestDataManager] Loading data for tests.LoginTest
[DatabaseManager] Creating DB Connection

Executing Valid Login Test

Executing Invalid Login Test

===== After LoginTest =====

[DatabaseManager] Closing DB Connection
[TestDataManager] Releasing test data
[DriverManager] Closing Browser
```
