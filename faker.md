Using Faker Library in Selenium Java Projects

The Java Faker library is commonly used in Selenium automation frameworks to generate realistic test data dynamically instead of hard-coding values.

# Maven Dependency
```xml
Java Faker
<dependency>
    <groupId>com.github.javafaker</groupId>
    <artifactId>javafaker</artifactId>
    <version>1.0.2</version>
</dependency>
```

**Create a Utility Class**

```java

FakerUtil.java
import com.github.javafaker.Faker;

public class FakerUtil {

    private static final Faker faker = new Faker();

    public static Faker getFaker() {
        return faker;
    }
}
```

# Common Test Data Examples

**Person Name**
```java
String firstName = FakerUtil.getFaker().name().firstName();
String lastName = FakerUtil.getFaker().name().lastName();
String fullName = FakerUtil.getFaker().name().fullName();

System.out.println(fullName);
```

Sample Output
Alex Smith
John Miller
Sophia Johnson

**Email Address**
```java
String email = FakerUtil.getFaker()
                        .internet()
                        .emailAddress();

System.out.println(email);
```

Sample Output
john.smith@gmail.com
alex123@yahoo.com

**Unique Email**
```java
String email =
        "user"
        + System.currentTimeMillis()
        + "@gmail.com";
```

**Mobile Number**: Indian 10 Digit Mobile
```java
String mobile =
        "9" +
        FakerUtil.getFaker()
                 .number()
                 .digits(9);

System.out.println(mobile);
```

Sample
9876543210
9123456789

**Address**

```java
String street =
        FakerUtil.getFaker()
                 .address()
                 .streetAddress();

String city =
        FakerUtil.getFaker()
                 .address()
                 .city();

String state =
        FakerUtil.getFaker()
                 .address()
                 .state();

String zip =
        FakerUtil.getFaker()
                 .address()
                 .zipCode();
```

Sample output:

Street : 123 Main Street
City   : Noida
State  : Uttar Pradesh
Zip    : 201301

**Company Name**
```java
String company =
        FakerUtil.getFaker()
                 .company()
                 .name();
```

output:
ABC Solutions Inc.
Global Digital Services

**Username**
```java
String username =
        FakerUtil.getFaker()
                 .name()
                 .username();
```

**Password**
```java
String password =
        FakerUtil.getFaker()
                 .internet()
                 .password(
                        8,
                        15,
                        true,
                        true,
                        true);
```

Sample
Abc@12345
Pass#9876

**Date of Birth**
```java
Date dob =
        FakerUtil.getFaker()
                 .date()
                 .birthday();
```

**Credit Card Number**
```java
String card =
        FakerUtil.getFaker()
                 .business()
                 .creditCardNumber();
```

Insurance Domain Examples
**Policy Number**
```java
String policyNumber =
        "POL"
        + FakerUtil.getFaker()
                   .number()
                   .digits(8);
```

Output
POL12345678

**Claim Number**
```java
String claimNumber = "CLM"
        + FakerUtil.getFaker()
                   .number()
                   .digits(10);
```

Output
CLM9876543210

**VIN Number**
```java
String vin =
        FakerUtil.getFaker()
                 .regexify("[A-HJ-NPR-Z0-9]{17}");
```

Output
1HGCM82633A123456

**Driver License Number**
US Style
```java
String dlNumber =
        FakerUtil.getFaker()
                 .regexify("[A-Z]{2}[0-9]{10}");
```

Output
AB1234567890

**India Style Vehicle License Number**
```java
String licenseNo =
        FakerUtil.getFaker()
                 .regexify("[A-Z]{2}[0-9]{2}[0-9]{11}");
```

Output
UP142024123456789

**Vehicle Registration Number** India Format
```java
String vehicleNumber =
        FakerUtil.getFaker()
                 .regexify("[A-Z]{2}[0-9]{2}[A-Z]{2}[0-9]{4}");
```

Output
DL01AB1234
MH12XY5678

**Aadhaar Number**
```java
String aadhaar =
        FakerUtil.getFaker()
                 .number()
                 .digits(12);
```

Output
123456789123

**PAN Number**
```java
String pan = FakerUtil.getFaker()
                 .regexify("[A-Z]{5}[0-9]{4}[A-Z]");
```

Output
ABCDE1234F

**SSN**
```java
String ssn = FakerUtil.getFaker()
                 .idNumber()
                 .ssnValid();
```

**Complete User Object Example**

```java

Faker faker = new Faker();

String firstName = faker.name().firstName();
String lastName = faker.name().lastName();
String email = faker.internet().emailAddress();
String mobile = "9" + faker.number().digits(9);
String address = faker.address().streetAddress();
String city = faker.address().city();
String licenseNo = faker.regexify("[A-Z]{2}[0-9]{2}[0-9]{11}");

System.out.println(firstName);
System.out.println(lastName);
System.out.println(email);
System.out.println(mobile);
System.out.println(address);
System.out.println(city);
System.out.println(licenseNo);
```

Using Faker with Selenium Page Objects

```java
@Test
public void createUser() {
    Faker faker = new Faker();

    RegistrationPage page = new RegistrationPage(driver);
    page.enterFirstName(faker.name().firstName());
    page.enterLastName(faker.name().lastName());
    page.enterEmail(faker.internet().emailAddress());
    page.enterMobile("9" + faker.number().digits(9));
    page.submit();
}
```

# Enterprise Best Practice

Create a centralized TestDataFactory instead of calling Faker directly from test classes.

```java
public class TestDataFactory {

    private static Faker faker =
            new Faker();

    public static String getEmail() {
        return faker.internet()
                    .emailAddress();
    }

    public static String getMobile() {
        return "9" +
               faker.number()
                    .digits(9);
    }

    public static String getLicenseNumber() {
        return faker.regexify(
                "[A-Z]{2}[0-9]{2}[0-9]{11}");
    }
}
```

Usage:
```java
String email = TestDataFactory.getEmail();
String mobile = TestDataFactory.getMobile();
String license = TestDataFactory.getLicenseNumber();
```

This approach keeps your Selenium POM framework maintainable, reusable, and generates realistic test data for registration forms, insurance applications, policy creation, claims processing, and API automation.
