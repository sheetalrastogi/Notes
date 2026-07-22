# Deserializing JSON Responses into POJOs Using Rest Assured

In Rest Assured, you can convert a JSON response directly into a POJO (Plain Old Java Object) using the `as()` method.

---

# 1. Sample Response

Assume the API returns the following JSON response:

```json
{
  "id": 101,
  "name": "John Doe",
  "email": "john@test.com"
}
```

---

# 2. Create POJO Class

```java
public class User {

    private int id;
    private String name;
    private String email;

    // Getters and Setters

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }


    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

---

# 3. Convert Response to POJO

```java
import io.restassured.response.Response;

Response response =
    given()
        .baseUri("https://api.example.com")
    .when()
        .get("/users/101");

User user = response.as(User.class);

System.out.println(user.getName());
System.out.println(user.getEmail());
```

---

# 4. Using ValidatableResponse

You can directly extract the response as a POJO after validations.

```java
User user =
    given()
    .when()
        .get("/users/101")
    .then()
        .statusCode(200)
        .extract()
        .as(User.class);

System.out.println(user.getId());
```

---

# 5. Response Containing Nested Objects

## Sample JSON

```json
{
  "id": 101,
  "name": "John",
  "address": {
    "city": "Mumbai",
    "state": "MH"
  }
}
```

## Address POJO

```java
public class Address {

    private String city;
    private String state;

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }
}
```

## User POJO

```java
public class User {

    private int id;
    private String name;
    private Address address;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }
}
```

## Usage

```java
User user = response.as(User.class);

System.out.println(
    user.getAddress().getCity()
);
```

---

# 6. Response Containing Array/List

## Sample JSON

```json
[
  {
    "id": 1,
    "name": "John"
  },
  {
    "id": 2,
    "name": "Mary"
  }
]
```

## Convert to List of POJOs

```java
List<User> users =
    response.jsonPath()
            .getList("", User.class);

users.forEach(user ->
    System.out.println(user.getName())
);
```

---

# 7. Generic Wrapper Response

## Sample JSON

```json
{
  "status": "SUCCESS",
  "data": {
    "id": 101,
    "name": "John"
  }
}
```

## Generic Wrapper POJO

```java
public class ApiResponse<T> {

    private String status;
    private T data;

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}
```

## User POJO

```java
public class User {

    private int id;
    private String name;

    // Getters and Setters
}
```

## Deserialize Using Jackson ObjectMapper

```java
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;

ObjectMapper mapper = new ObjectMapper();

ApiResponse<User> apiResponse =
    mapper.readValue(
        response.asString(),
        new TypeReference<ApiResponse<User>>() {}
    );

System.out.println(
    apiResponse.getData().getName()
);
```

---

# 8. Using Lombok (Recommended)

Lombok eliminates boilerplate getter/setter code.

```java
import lombok.Data;

@Data
public class User {

    private int id;
    private String name;
    private String email;
}
```

### Usage

```java
User user = response.as(User.class);
```

---

# Real-World Rest Assured Example

## RequestSpecification

```java
RequestSpecification requestSpec =
    new RequestSpecBuilder()
        .setBaseUri("https://reqres.in")
        .build();
```

## ResponseSpecification

```java
ResponseSpecification responseSpec =
    new ResponseSpecBuilder()
        .expectStatusCode(200)
        .build();
```

## Extract Response into POJO

```java
User user =
    given()
        .spec(requestSpec)
    .when()
        .get("/api/users/2")
    .then()
        .spec(responseSpec)
        .extract()
        .jsonPath()
        .getObject("data", User.class);

System.out.println(
    "User Name: " + user.getName()
);
```

---

# Other Useful POJO Deserialization Methods

## Entire Response to POJO

```java
response.as(User.class);
```

---

## Specific Object to POJO

```java
response.jsonPath()
        .getObject("data", User.class);
```

---

## JSON Array to List of POJOs

```java
response.jsonPath()
        .getList("", User.class);
```

---

## Nested Array to List of POJOs

```java
response.jsonPath()
        .getList("data", User.class);
```

---

# Common Interview Questions

## Q1. How do you deserialize a JSON response into a POJO in Rest Assured?

```java
User user = response.as(User.class);
```

---

## Q2. How do you deserialize a nested object?

Create nested POJO classes and use:

```java
User user = response.as(User.class);
```

Rest Assured automatically maps nested objects.

---

## Q3. How do you deserialize a JSON array?

```java
List<User> users =
    response.jsonPath()
            .getList("", User.class);
```

---

## Q4. How do you deserialize a nested array?

```java
List<User> users =
    response.jsonPath()
            .getList("data", User.class);
```

---

## Q5. When should ObjectMapper be used instead of `response.as()`?

Use `ObjectMapper` when:

- Working with generic types
- Deserializing wrapper responses
- Performing custom serialization/deserialization
- Handling complex object mappings

---

# Summary

The most common POJO deserialization patterns in Rest Assured are:

```java
response.as(User.class);
```

```java
jsonPath().getObject("data", User.class);
```

```java
jsonPath().getList("", User.class);
```

```java
jsonPath().getList("data", User.class);
```

```java
ObjectMapper.readValue(
    response.asString(),
    new TypeReference<ApiResponse<User>>() {}
);
```

These patterns are widely used in enterprise API automation frameworks, microservices testing, and Rest Assured interview scenarios for converting JSON responses into strongly typed Java POJO objects.
