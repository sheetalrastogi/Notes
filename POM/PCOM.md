## Page Component Object Pattern (PCOP) 

Selenium 4 Java automation framework, the Page Component Object Pattern (PCOP) works best when combined with:

- Page Object Model (POM)
- BasePage and BaseComponent
- Dependency Injection (Optional: Guice/Spring)
- TestNG
- Fluent Page APIs
- Reusable UI Widgets 


# Example Project Structure

```text
src
├── main
│   └── java
│       ├── pages
│       │   ├── LoginPage.java
│       │   ├── DashboardPage.java
│       │   └── ProductsPage.java
│       │
│       ├── components
│       │   ├── HeaderComponent.java
│       │   ├── SideMenuComponent.java
│       │   ├── ProductCardComponent.java
│       │   ├── SearchBarComponent.java
│       │   └── CartWidgetComponent.java
│       │
│       ├── base
│       │   ├── BasePage.java
│       │   └── BaseComponent.java
│       │
│       └── utils
│
└── test
    └── java
        └── tests
            ├── LoginTest.java
            └── AddToCartTest.java
```


# 1. Base Component

Every component drives from this class.

```java
package framework.base;

import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;

public abstract class BaseComponent {

    protected WebElement root;

    public BaseComponent(WebElement root) {
        this.root = root;
    }

    protected WebElement find(By locator) {
        return root.findElement(locator);
    }
}
```

# 2. Base Page

```java
package framework.base;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public abstract class BasePage {

    protected WebDriver driver;

    public BasePage(WebDriver driver) {
        this.driver = driver;
    }

    protected void click(By locator) {
        driver.findElement(locator).click();
    }
}
```


Application Under Test

Imagine an e-commerce application:

```text
Dashboard Page
|
├── Header
│     ├── Logo
│     ├── Search Bar
│     └── Cart Widget
|
├── Side Menu
|
└── Product Grid
        ├── Product Card
        ├── Product Card
        └── Product Card
```

# 3. Header Component

```java
package components;

import framework.base.BaseComponent;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;

public class HeaderComponent extends BaseComponent {

	private final By logo = By.cssSelector(".logo");

	private final By userMenu = By.id("userMenu");

	public HeaderComponent(WebElement root) {
		super(root);
	}

	public boolean isLogoVisible() {
		return find(logo).isDisplayed();
	}

	public void openUserMenu() {
		find(userMenu).click();
	}
}

```


# 4. Search Bar Component

```java
package components;

import framework.base.BaseComponent;
import org.openqa.selenium.By;
import org.openqa.selenium.Keys;
import org.openqa.selenium.WebElement;

public class SearchBarComponent extends BaseComponent {

	private final By searchTextbox = By.id("search");

	public SearchBarComponent(WebElement root) {
		super(root);
	}

	public void search(String product) {

		find(searchTextbox).sendKeys(product + Keys.ENTER);
	}
}

```


# 5. Cart Widget Component

```java
package components;

import framework.base.BaseComponent;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;

public class CartWidgetComponent extends BaseComponent {

	private final By count = By.cssSelector(".cart-count");

	public CartWidgetComponent(WebElement root) {
		super(root);
	}

	public int getItemCount() {

		return Integer.parseInt(find(count).getText());
	}
}

```


# 6. Product Card Component

A Product Card is not a page.  It is a reusable component.

```java
package components;

import framework.base.BaseComponent;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;

public class ProductCardComponent extends BaseComponent {

	private final By productName = By.cssSelector(".product-name");
	private final By addToCart = By.cssSelector(".add-cart");

	public ProductCardComponent(WebElement root) {
		super(root);
	}

	public String getName() {
		return find(productName).getText();
	}

	public void addToCart() {
		find(addToCart).click();
	}
}

```



# 7. Products Page

Page object assembles components.

```java
import org.openqa.selenium.*;
import java.util.List;

public class ProductsPage extends BasePage {
	private final By header = By.id("header");
	private final By productCards = By.cssSelector(".product-card");

	public ProductsPage(WebDriver driver) {
		super(driver);
	}

	public HeaderComponent header() {
		return new HeaderComponent(driver.findElement(header));
	}

	public List<ProductCardComponent> getProducts() {
		return driver.findElements(productCards).stream().map(ProductCardComponent::new).toList();
	}
}

```

# 8. Login Page

Normal page object.

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class LoginPage extends BasePage {

	private final By username = By.id("username");
	private final By password = By.id("password");
	private final By loginButton = By.id("loginBtn");

	public LoginPage(WebDriver driver) {
		super(driver);
	}

	public ProductsPage login(String user, String pass) {
		driver.findElement(username).sendKeys(user);
		driver.findElement(password).sendKeys(pass);
		driver.findElement(loginButton).click();
		return new ProductsPage(driver);
	}
}

```


# 9. Test Implementation

```java
@Test
public void addProductToCart() {

    ProductsPage page =
            new LoginPage(driver)
                    .login("admin", "admin");

    ProductCardComponent iphone =
            page.getProducts()
                .stream()
                .filter(p ->
                    p.getName()
                     .equals("iPhone 16"))
                .findFirst()
                .orElseThrow();

    iphone.addToCart();
}
```


**Rules**

- Page Object
  - Represents an entire page.
  - Handles navigation between pages.
  - Returns components.

- Component Object
  - Represents a section/widget within a page.
  - Never navigates.
  - Works only within its root element.

- Test Class
  - Talks to Page Objects.
  - Uses Components through Page Objects.
  - Contains assertions only.

