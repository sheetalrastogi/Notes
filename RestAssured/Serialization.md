## Common approaches for creating a RestAssured request payload from the given JSON:
---

# Sample JSON:
```text
{
  "employeeId": "EMP1001",
  "name": "Sheetal Rastogi",
  "address": {
    "city": "Noida",
    "state": "UP",
    "country": "India"
  },
  "skills": [
    {
      "name": "Selenium",
      "experience": 10
    },
    {
      "name": "Playwright",
      "experience": 2
    },
    {
      "name": "RestAssured",
      "experience": 5
    }
  ]
}
```

# 1. Standard Java POJO Classes 

Employee class

```java
import java.util.List;

public class Employee {

    private String employeeId;
    private String name;
    private Address address;
    private List<Skill> skills;

    public String getEmployeeId() {
        return employeeId;
    }

    public void setEmployeeId(String employeeId) {
        this.employeeId = employeeId;
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

    public List<Skill> getSkills() {
        return skills;
    }

    public void setSkills(List<Skill> skills) {
        this.skills = skills;
    }
}
```

Address Class:

```java
public class Address {

    private String city;
    private String state;
    private String country;

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

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
    }
}
```

Skills class

```java
public class Skill {

    private String name;
    private int experience;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getExperience() {
        return experience;
    }

    public void setExperience(int experience) {
        this.experience = experience;
    }
}
```

**Create Request Payload using POJO**

```java
import java.util.Arrays;

public class EmployeePayloadBuilder {

    public static Employee createEmployee() {

        Address address = new Address();
        address.setCity("Noida");
        address.setState("UP");
        address.setCountry("India");

        Skill skill1 = new Skill();
        skill1.setName("Selenium");
        skill1.setExperience(10);

        Skill skill2 = new Skill();
        skill2.setName("Playwright");
        skill2.setExperience(2);

        Skill skill3 = new Skill();
        skill3.setName("RestAssured");
        skill3.setExperience(5);

        Employee employee = new Employee();
        employee.setEmployeeId("EMP1001");
        employee.setName("Sheetal Rastogi");
        employee.setAddress(address);
        employee.setSkills(
                Arrays.asList(skill1, skill2, skill3));

        return employee;
    }
}
```

**Use POJO in RestAssured Request**

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;

public class EmployeeAPITest {

    public static void main(String[] args) {

        Employee employee =
                EmployeePayloadBuilder.createEmployee();

        Response response =
                RestAssured
                        .given()
                            .baseUri("https://api.company.com")
                            .contentType("application/json")
                            .body(employee)
                        .when()
                            .post("/employees");

        System.out.println(response.getStatusCode());
        System.out.println(response.asPrettyString());
    }
}
```

## 2. Lombok POJO Classes (Most Compact)
---

Employee

```java
import lombok.Data;
import java.util.List;

@Data
public class Employee {

    private String employeeId;
    private String name;
    private Address address;
    private List<Skill> skills;
}
```

Address

```java
import lombok.Data;

@Data
public class Address {

    private String city;
    private String state;
    private String country;
}
```

Skills

```java
import lombok.Data;

@Data
public class Skill {

    private String name;
    private int experience;
}
```

**Payload creation with Lombok**

```java
import java.util.Arrays;

public class EmployeePayloadBuilder {

    public static Employee buildEmployee() {

        Address address = new Address();
        address.setCity("Noida");
        address.setState("UP");
        address.setCountry("India");

        Skill selenium = new Skill();
        selenium.setName("Selenium");
        selenium.setExperience(10);

        Skill playwright = new Skill();
        playwright.setName("Playwright");
        playwright.setExperience(2);

        Skill restAssured = new Skill();
        restAssured.setName("RestAssured");
        restAssured.setExperience(5);

        Employee employee = new Employee();
        employee.setEmployeeId("EMP1001");
        employee.setName("Sheetal Rastogi");
        employee.setAddress(address);
        employee.setSkills(
                Arrays.asList(
                        selenium,
                        playwright,
                        restAssured));

        return employee;
    }
}
```

**Use in RestAssured Request**

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;

public class EmployeeAPITest {

    public static void main(String[] args) {

        Employee employee =
                EmployeePayloadBuilder.buildEmployee();

        Response response =
                RestAssured
                        .given()
                            .baseUri("https://api.company.com")
                            .contentType("application/json")
                            .body(employee)
                        .when()
                            .post("/employees");

        System.out.println(response.asPrettyString());
    }
}
```

## 3. Single POJO with Inner Classes (Very Popular in Automation Frameworks)
---

Employee

```java
import java.util.List;

public class Employee {

    private String employeeId;
    private String name;
    private Address address;
    private List<Skill> skills;

    public String getEmployeeId() {
        return employeeId;
    }

    public void setEmployeeId(String employeeId) {
        this.employeeId = employeeId;
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

    public List<Skill> getSkills() {
        return skills;
    }

    public void setSkills(List<Skill> skills) {
        this.skills = skills;
    }

    public static class Address {

        private String city;
        private String state;
        private String country;

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

        public String getCountry() {
            return country;
        }

        public void setCountry(String country) {
            this.country = country;
        }
    }

    public static class Skill {

        private String name;
        private int experience;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getExperience() {
            return experience;
        }

        public void setExperience(int experience) {
            this.experience = experience;
        }
    }
}
```


## RestAssured Serialization Example (Inner Class)
------

```java
import java.util.Arrays;

public class EmployeePayload {

    public static Employee createEmployeePayload() {

        // Address
        Employee.Address address = new Employee.Address();
        address.setCity("Noida");
        address.setState("UP");
        address.setCountry("India");

        // Skill 1
        Employee.Skill selenium = new Employee.Skill();
        selenium.setName("Selenium");
        selenium.setExperience(10);

        // Skill 2
        Employee.Skill playwright = new Employee.Skill();
        playwright.setName("Playwright");
        playwright.setExperience(2);

        // Skill 3
        Employee.Skill restAssured = new Employee.Skill();
        restAssured.setName("RestAssured");
        restAssured.setExperience(5);

        // Employee Object
        Employee employee = new Employee();
        employee.setEmployeeId("EMP1001");
        employee.setName("Sheetal Rastogi");
        employee.setAddress(address);
        employee.setSkills(
                Arrays.asList(selenium, playwright, restAssured));

        return employee;
    }
}

```
**RestAssured Usage (for InnerClass)**

```java
import io.restassured.RestAssured;

public class EmployeeTest {

    public static void main(String[] args) {

        Employee employee =
                EmployeePayload.createEmployeePayload();

        RestAssured
                .given()
                    .baseUri("https://api.company.com")
                    .contentType("application/json")
                    .body(employee)
                .when()
                    .post("/employees")
                .then()
                    .statusCode(201);
    }
}
```


Output:  Serialized JSON Generated by RestAssured
```text
{
  "employeeId": "EMP1001",
  "name": "Sheetal Rastogi",
  "address": {
    "city": "Noida",
    "state": "UP",
    "country": "India"
  },
  "skills": [
    {
      "name": "Selenium",
      "experience": 10
    },
    {
      "name": "Playwright",
      "experience": 2
    },
    {
      "name": "RestAssured",
      "experience": 5
    }
  ]
}
```
