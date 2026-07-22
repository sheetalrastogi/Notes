## Create Page Object Model for RestAssured Project

For a RestAssured API Automation Framework, the equivalent of Selenium's Page Object Model (POM) is usually implemented as:

# Endpoint Object Model (EOM) or API Object Model
Each API resource/service is represented by a separate class.
Request creation, endpoint URLs, headers, authentication, and API actions are encapsulated inside service classes.

# Recommended Framework Structure
```text
src/test/java
│
├── api
│   ├── endpoints
│   │    ├── UserAPI.java
│   │    ├── ProductAPI.java
│   │    └── LoginAPI.java
│   │
│   ├── models
│   │    ├── User.java
│   │    ├── Address.java
│   │    └── AuthToken.java
│   │
│   ├── specifications
│   │    ├── RequestSpecs.java
│   │    └── ResponseSpecs.java
│   │
│   ├── utils
│   │    ├── ConfigReader.java
│   │    └── TestDataFactory.java
│   │
│   └── constants
│        └── EndPoints.java
│
├── tests
│   ├── UserTests.java
│   └── LoginTests.java
│
└── base
     └── BaseTest.java
```

# 1. Endpoint Constants
EndPoints.java
```java
public class EndPoints {

    public static final String BASE_URL = "https://reqres.in";
    public static final String USERS = "/api/users";
    public static final String USER_BY_ID = "/api/users/{id}";
}
```
# 2. Request Specification

Similar to WebDriver initialization in Selenium.

RequestSpecs.java
```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.specification.RequestSpecification;

public class RequestSpecs {

    public static RequestSpecification getRequestSpec() {

        return new RequestSpecBuilder()
                .setBaseUri("https://reqres.in")
                .addHeader("Content-Type", "application/json")
                .build();
    }
}
```

# 3. Response Specification
ResponseSpecs.java
```java
import io.restassured.builder.ResponseSpecBuilder;
import io.restassured.specification.ResponseSpecification;

public class ResponseSpecs {

    public static ResponseSpecification getResponseSpec() {

        return new ResponseSpecBuilder()
                .expectStatusCode(200)
                .build();
    }
}
```

# 4. API Object Class (Equivalent of Page Class)
UserAPI.java
```java
import io.restassured.response.Response;
import static io.restassured.RestAssured.given;

public class UserAPI {

    public Response getUser(int userId) {

        return given()
                .spec(RequestSpecs.getRequestSpec())
                .pathParam("id", userId)
        .when()
                .get(EndPoints.USER_BY_ID);
    }

    public Response getAllUsers() {

        return given()
                .spec(RequestSpecs.getRequestSpec())
        .when()
                .get(EndPoints.USERS);
    }

    public Response createUser(User user) {

        return given()
                .spec(RequestSpecs.getRequestSpec())
                .body(user)
        .when()
                .post(EndPoints.USERS);
    }
}
```

# 5. POJO Model Class
User.java

```java
public class User {

    private String name;
    private String job;

    public User() {}

    public User(String name, String job) {
        this.name = name;
        this.job = job;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getJob() {
        return job;
    }

    public void setJob(String job) {
        this.job = job;
    }
}
```

# 6. Base Test
BaseTest.java
```java
import io.restassured.RestAssured;
import org.testng.annotations.BeforeClass;

public class BaseTest {

    @BeforeClass
    public void setup() {

        RestAssured.baseURI = EndPoints.BASE_URL;
    }
}
```

# 7. Test Class
UserTests.java
```java
import org.testng.Assert;
import org.testng.annotations.Test;

public class UserTests extends BaseTest {

    UserAPI userAPI = new UserAPI();

    @Test
    public void verifyGetUser() {
        var response = userAPI.getUser(2);

        Assert.assertEquals(
                response.jsonPath().getString("data.first_name"),
                "Janet");
    }

    @Test
    public void verifyCreateUser() {

        User user = new User("Alex", "QA Engineer");

        var response = userAPI.createUser(user);

        Assert.assertEquals(response.statusCode(), 201);

        Assert.assertEquals(
                response.jsonPath().getString("name"),
                "Alex");
    }
}
```

# Advanced API Object Model with Builder Pattern

For enterprise frameworks, use:
```java
UserAPI userAPI = new UserAPI();

Response response = userAPI
                        .withAuthToken(token)
                        .withHeader("env", "QA")
                        .getUser(10);
```

Example usage:
```java
public class UserAPI {

    private RequestSpecification request;

    public UserAPI() {
        request = given()
                .spec(RequestSpecs.getRequestSpec());
    }

    public UserAPI withAuthToken(String token) {

        request.header(
                "Authorization",
                "Bearer " + token);

        return this;
    }

    public UserAPI withHeader(
            String key,
            String value) {

        request.header(key, value);
        return this;
    }

    public Response getUser(int id) {

        return request
                .pathParam("id", id)
                .get(EndPoints.USER_BY_ID);
    }
}
```

# Comparing Selenium POM vs RestAssured API Object Model
Selenium POM
```java
LoginPage loginPage = new LoginPage(driver);

loginPage.enterUserName("admin");
loginPage.enterPassword("admin123");
loginPage.clickLogin();
```
RestAssured API Object Model
```java
UserAPI userAPI = new UserAPI();

Response response = userAPI.getUser(10);
```

or
```java
LoginAPI loginAPI = new LoginAPI();

String token = loginAPI
                    .login("admin", "admin123")
                    .jsonPath()
                    .getString("token");
```
**Enterprise Best Practices**
- Separate API classes by resource (UserAPI, OrderAPI, PolicyAPI).
- Keep endpoint URLs in constants.
- Use RequestSpecification and ResponseSpecification.
- Use POJO serialization/deserialization.
- Add filters for logging.
- Integrate with TestNG/JUnit.
- Use Jackson ObjectMapper.
- Use Allure reports.
- Implement OAuth/JWT handlers centrally.
- Follow API Object Model instead of writing RestAssured code directly in test classes.

This architecture is the most commonly used POM-style design for RestAssured API automation frameworks in enterprise Selenium/Java projects.
