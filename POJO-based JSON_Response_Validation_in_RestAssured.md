# POJO-Based JSON Response Validation in Rest Assured

This document demonstrates a complete example of **POJO-based JSON Response Validation** in **Rest Assured** for responses containing:

- Primitive fields
- Nested Objects
- ArrayList of Objects
- Assertions using deserialized POJOs

---

# Sample JSON Response

```json
{
  "id": 101,
  "name": "Alex",
  "email": "alex@gmail.com",
  "address": {
    "street": "123 Business Park",
    "city": "Noida",
    "state": "UP",
    "zipCode": "201301"
  },
  "contacts": [
    {
      "type": "Mobile",
      "number": "9999999999"
    },
    {
      "type": "Landline",
      "number": "0120123456"
    }
  ]
}
```

---

# POJO Classes

## Address.java

```java
public class Address {

    private String street;
    private String city;
    private String state;
    private String zipCode;

    public String getStreet() {
        return street;
    }

    public void setStreet(String street) {
        this.street = street;
    }

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

    public String getZipCode() {
        return zipCode;
    }

    public void setZipCode(String zipCode) {
        this.zipCode = zipCode;
    }
}
```

---

## Contact.java

```java
public class Contact {

    private String type;
    private String number;

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }


    public String getNumber() {
        return number;
    }

    public void setNumber(String number) {
        this.number = number;
    }
}
```

---

## User.java

```java
import java.util.List;

public class User {

    private int id;
    private String name;
    private String email;
    private Address address;
    private List<Contact> contacts;

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


    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }


    public List<Contact> getContacts() {
        return contacts;
    }

    public void setContacts(List<Contact> contacts) {
        this.contacts = contacts;
    }
}
```

---

# Rest Assured Response Validation

## Deserialize Response to User POJO

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;

public class UserTest {

    public static void main(String[] args) {

        Response response =
                RestAssured.given()
                        .baseUri("https://api.example.com")
                .when()
                        .get("/users/101");

        User user = response.as(User.class);

        System.out.println(user.getName());
        System.out.println(user.getAddress().getCity());
    }
}
```

---

# Validate Primitive Fields

```java
assertEquals(101, user.getId());
assertEquals("Alex", user.getName());
assertEquals("alex@gmail.com", user.getEmail());
```

---

# Validate Nested Object

```java
Address address = user.getAddress();

assertEquals("Noida", address.getCity());
assertEquals("UP", address.getState());
assertEquals("201301", address.getZipCode());
```

---

# Validate ArrayList

```java
List<Contact> contacts = user.getContacts();

assertEquals(2, contacts.size());

assertEquals("Mobile",
             contacts.get(0).getType());

assertEquals("9999999999",
             contacts.get(0).getNumber());

assertEquals("Landline",
             contacts.get(1).getType());

assertEquals("0120123456",
             contacts.get(1).getNumber());
```

---

# Validate Using Java Streams

## Find Mobile Contact

```java
Contact mobileContact =
        user.getContacts()
            .stream()
            .filter(c -> "Mobile".equals(c.getType()))
            .findFirst()
            .orElse(null);

assertNotNull(mobileContact);

assertEquals(
        "9999999999",
        mobileContact.getNumber()
);
```

---

# Response with Array of Users

## Sample JSON

```json
[
  {
    "id": 101,
    "name": "Alex"
  },
  {
    "id": 102,
    "name": "John"
  }
]
```

---

## Deserialize to List<User>

```java
Response response =
        given()
        .when()
            .get("/users");

List<User> users =
        response.jsonPath()
                .getList("", User.class);

assertEquals(2, users.size());

assertEquals(
        "Alex",
        users.get(0).getName()
);

assertEquals(
        "John",
        users.get(1).getName()
);
```

---

# Complex Nested Validation Example

## Sample JSON

```json
{
  "name": "Alex",
  "address": {
    "city": "Noida"
  },
  "contacts": [
    {
      "type": "Mobile",
      "number": "9999999999"
    }
  ]
}
```

## Validation

```java
User user = response.as(User.class);

assertEquals(
        "Alex",
        user.getName()
);

assertEquals(
        "Noida",
        user.getAddress().getCity()
);

assertEquals(
        "Mobile",
        user.getContacts().get(0).getType()
);

assertEquals(
        "9999999999",
        user.getContacts().get(0).getNumber()
);
```

---

# Interview-Oriented Example (JUnit 5)

Using `assertAll()` to validate multiple fields in a single assertion block:

```java
import static org.junit.jupiter.api.Assertions.*;

User user = response.as(User.class);

assertAll(
    () -> assertEquals(101, user.getId()),
    () -> assertEquals("Alex", user.getName()),
    () -> assertEquals(
            "Noida",
            user.getAddress().getCity()
    ),
    () -> assertEquals(
            2,
            user.getContacts().size()
    ),
    () -> assertEquals(
            "Mobile",
            user.getContacts().get(0).getType()
    )
);
```

---

# Alternative Validation Using TestNG

```java
Assert.assertEquals(
        user.getId(),
        101
);

Assert.assertEquals(
        user.getName(),
        "Alex"
);

Assert.assertEquals(
        user.getAddress().getCity(),
        "Noida"
);

Assert.assertEquals(
        user.getContacts().size(),
        2
);
```

---

# Most Common POJO Validation Patterns

## Single Object Deserialization

```java
User user = response.as(User.class);
```

---

## Nested Object Validation

```java
user.getAddress().getCity();
```

---

## Array/List Validation

```java
user.getContacts().get(0);
```

---

## List of POJOs

```java
response.jsonPath()
        .getList("", User.class);
```

---

## Get Nested List of POJOs

```java
response.jsonPath()
        .getList("data", User.class);
```

---

## Stream-Based Search

```java
Contact contact =
        user.getContacts()
            .stream()
            .filter(c -> "Mobile".equals(c.getType()))
            .findFirst()
            .orElse(null);
```

---

# Real-World Validation Flow

```text
JSON Response
      |
      v
response.as(User.class)
      |
      v
User POJO
      |
      +--> Primitive Field Validation
      |
      +--> Nested Object Validation
      |
      +--> Array/List Validation
      |
      +--> Stream-Based Search Validation
```

---

# Key Takeaways

This example demonstrates the most common **Rest Assured POJO Validation** scenarios:

1. `response.as(User.class)` → Single Object Deserialization
2. Nested Object Validation → `user.getAddress().getCity()`
3. Array/List Validation → `user.getContacts().get(0)`
4. List of POJOs → `getList("", User.class)`
5. Stream-Based Lookup → Find and validate specific objects within collections
6. Combined Assertions using `assertAll()`
7. Validation of Primitive Fields, Nested Objects, and Collections

These patterns are widely used in enterprise API automation frameworks, microservices testing, and Rest Assured interview scenarios.
