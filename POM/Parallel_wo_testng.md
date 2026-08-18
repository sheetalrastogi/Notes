## how to implement thread-safe POM Using Thread/Runnable classes in java (without referring TestNG based parallelism)

# Parallel Selenium tests execution without TestNG parallelism. 

Create and manage threads yourself using Thread, Runnable, ExecutorService, etc.

The key requirement remains:
```text
Each thread must have its own WebDriver instance and its own Page Objects.
```

# Project Structure

```text
Main Thread
    │
    ├── Thread-1
    │      ├── ChromeDriver-1
    │      ├── LoginPage-1
    │      └── DashboardPage-1
    │
    ├── Thread-2
    │      ├── ChromeDriver-2
    │      ├── LoginPage-2
    │      └── DashboardPage-2
    │
    └── Thread-3
           ├── ChromeDriver-3
           ├── LoginPage-3
           └── DashboardPage-3
```


# 1. Thread-Safe DriverManager

```java
public final class DriverManager {

	private DriverManager() {
	}

	private static final ThreadLocal<WebDriver> DRIVER = new ThreadLocal<>();

	public static void createDriver() {

		WebDriver driver = new ChromeDriver();

		DRIVER.set(driver);
	}

	public static WebDriver getDriver() {
		return DRIVER.get();
	}

	public static void quitDriver() {

		WebDriver driver = DRIVER.get();

		if (driver != null) {
			driver.quit();
			DRIVER.remove();
		}
	}
}
```

# 2. Base Page

Every page gets driver from ThreadLocal.

```java
public abstract class BasePage {

    protected WebDriver driver;

    public BasePage() {
        this.driver = DriverManager.getDriver();
    }
}
```


# 3. Login Page

```java
public class LoginPage extends BasePage {

	private By username = By.id("username");

	private By password = By.id("password");

	private By loginButton = By.id("loginBtn");

	public LoginPage enterUsername(String user) {

		driver.findElement(username).sendKeys(user);

		return this;
	}

	public LoginPage enterPassword(String pass) {

		driver.findElement(password).sendKeys(pass);

		return this;
	}

	public DashboardPage login() {

		driver.findElement(loginButton).click();

		return new DashboardPage();
	}
}
```

# 4. Dashboard Page

```java
public class DashboardPage extends BasePage {

    private By welcomeLabel =
            By.id("welcomeUser");

    public String getLoggedInUser() {

        return driver.findElement(welcomeLabel)
                .getText();
    }
}

```

# 5. Workflow Layer

```java
public class LoginWorkflow {

    public DashboardPage loginAs(
            String user,
            String password) {

        return new LoginPage()
                .enterUsername(user)
                .enterPassword(password)
                .login();
    }
}
```


# 6. Runnable Test Class

Each Runnable represents one virtual test execution.

```java
public class LoginTestRunner implements Runnable {

	private final String username;
	private final String password;

	public LoginTestRunner(String username, String password) {
		this.username = username;
		this.password = password;
	}

	@Override
	public void run() {

		try {
			DriverManager.createDriver();
			DriverManager.getDriver().get("https://app.com");
			DashboardPage dashboard = new LoginWorkflow().loginAs(username, password);
			System.out.println(Thread.currentThread().getName() + " -> " + dashboard.getLoggedInUser());

		} finally {
			DriverManager.quitDriver();
		}
	}
}
```


# 7. Execute Using Thread Class

```java
public class ParallelExecution {

	public static void main(String[] args) throws InterruptedException {

		Thread t1 = new Thread(new LoginTestRunner("admin", "admin123"));

		Thread t2 = new Thread(new LoginTestRunner("john", "john123"));

		Thread t3 = new Thread(new LoginTestRunner("mary", "mary123"));

		t1.start();
		t2.start();
		t3.start();

		t1.join();
		t2.join();
		t3.join();
	}
}
```


# Another approach for Parallel Execution

**ExecutorService** : Enterprise frameworks typically use a thread pool.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ParallelRunner {

	public static void main(String[] args) {

		ExecutorService executor = Executors.newFixedThreadPool(5);

		executor.submit(new LoginTestRunner("admin", "admin123"));

		executor.submit(new LoginTestRunner("john", "john123"));

		executor.submit(new LoginTestRunner("mary", "mary123"));

		executor.shutdown();
	}
}
```


## Common mistakes

- avoid static  WebDriver, Page Objects, TestData Objects etc.
