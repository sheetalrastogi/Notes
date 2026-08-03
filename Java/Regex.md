## Regular Expressions:
---

# 1. Email validation:

```java
String EMAIL_REGEX =
    "^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$";
```

Usage:
```java
Pattern pattern = Pattern.compile(EMAIL_REGEX);
Matcher matcher = pattern.matcher("john.doe@gmail.com");
System.out.println(matcher.matches());
```

# 2. IPv4 Address Validation

```java
String IPV4_REGEX =
    "^((25[0-5]|2[0-4]\\d|[01]?\\d\\d?)\\.){3}" +
    "(25[0-5]|2[0-4]\\d|[01]?\\d\\d?)$";
```

# 3. PAN Card Validation (India)

```java
String PAN_REGEX =
    "^[A-Z]{5}[0-9]{4}[A-Z]{1}$";
```

# 4. Aadhaar Validation (India)

```java
String AADHAAR_REGEX =
    "^[2-9][0-9]{11}$";
```

# 5. Mobile Number Validation (India)

```java
String MOBILE_REGEX =
    "^[6-9]\\d{9}$";
```

# 6. Passport Number (India)

```java
String PASSPORT_REGEX =
    "^[A-Z]{1}[0-9]{7}$";
```

# 7. Credit Card Number

```java
String CREDIT_CARD_REGEX =
    "^[0-9]{13,19}$";
```

# 8. Social Security Number (US)

```java
String SSN_REGEX =
    "^\\d{3}-\\d{2}-\\d{4}$";
```

# 9. ZIP Code (US)

```java
String ZIP_REGEX =
    "^\\d{5}(?:-\\d{4})?$";
```

# 10. Username Validation

```java
String USERNAME_REGEX =
    "^[a-zA-Z0-9_]{5,20}$";
```


# 11. URL Validation

```java
String URL_REGEX =
    "^(https?|ftp)://[^\\s/$.?#].[^\\s]*$";
```

# 12. UUID Validation

```java
String UUID_REGEX =
    "^[0-9a-fA-F]{8}-" +
    "[0-9a-fA-F]{4}-" +
    "[0-9a-fA-F]{4}-" +
    "[0-9a-fA-F]{4}-" +
    "[0-9a-fA-F]{12}$";
```




## Generic Utility method:
----

```java
import java.util.regex.Pattern;

public class RegexUtils {

    private RegexUtils() {
        // Utility Class
    }

    // =========================
    // Common Validation Method
    // =========================
    private static boolean isValid(String value, String regex) {
        return value != null && Pattern.matches(regex, value);
    }

    // =========================
    // Regular Expressions
    // =========================

    private static final String EMAIL_REGEX =
            "^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$";

    private static final String IPV4_REGEX =
            "^((25[0-5]|2[0-4]\\d|[01]?\\d\\d?)\\.){3}"
            + "(25[0-5]|2[0-4]\\d|[01]?\\d\\d?)$";

    private static final String PAN_REGEX =
            "^[A-Z]{5}[0-9]{4}[A-Z]{1}$";

    private static final String AADHAAR_REGEX =
            "^[2-9][0-9]{11}$";

    private static final String MOBILE_REGEX =
            "^[6-9]\\d{9}$";

    private static final String PASSPORT_REGEX =
            "^[A-Z]{1}[0-9]{7}$";

    private static final String CREDIT_CARD_REGEX =
            "^[0-9]{13,19}$";

    private static final String SSN_REGEX =
            "^\\d{3}-\\d{2}-\\d{4}$";

    private static final String ZIP_CODE_REGEX =
            "^\\d{5}(?:-\\d{4})?$";

    private static final String USERNAME_REGEX =
            "^[a-zA-Z0-9_]{5,20}$";

    // =========================
    // Validation Methods
    // =========================

    public static boolean isValidEmail(String email) {
        return isValid(email, EMAIL_REGEX);
    }

    public static boolean isValidIPv4(String ipAddress) {
        return isValid(ipAddress, IPV4_REGEX);
    }

    public static boolean isValidPAN(String pan) {
        return isValid(pan, PAN_REGEX);
    }

    public static boolean isValidAadhaar(String aadhaar) {
        return isValid(aadhaar, AADHAAR_REGEX);
    }

    public static boolean isValidMobileNumber(String mobile) {
        return isValid(mobile, MOBILE_REGEX);
    }

    public static boolean isValidPassport(String passport) {
        return isValid(passport, PASSPORT_REGEX);
    }

    public static boolean isValidCreditCard(String cardNumber) {
        return isValid(cardNumber, CREDIT_CARD_REGEX);
    }

    public static boolean isValidSSN(String ssn) {
        return isValid(ssn, SSN_REGEX);
    }

    public static boolean isValidZipCode(String zipCode) {
        return isValid(zipCode, ZIP_CODE_REGEX);
    }

    public static boolean isValidUsername(String username) {
        return isValid(username, USERNAME_REGEX);
    }

    // =========================
    // Sample Usage
    // =========================

    public static void main(String[] args) {

        System.out.println("Email      : "
                + isValidEmail("john.doe@gmail.com"));

        System.out.println("IPv4       : "
                + isValidIPv4("192.168.1.100"));

        System.out.println("PAN        : "
                + isValidPAN("ABCDE1234F"));

        System.out.println("Aadhaar    : "
                + isValidAadhaar("234567890123"));

        System.out.println("Mobile     : "
                + isValidMobileNumber("9876543210"));

        System.out.println("Passport   : "
                + isValidPassport("A1234567"));

        System.out.println("CreditCard : "
                + isValidCreditCard("4111111111111111"));

        System.out.println("SSN        : "
                + isValidSSN("123-45-6789"));

        System.out.println("ZIP Code   : "
                + isValidZipCode("12345-6789"));

        System.out.println("Username   : "
                + isValidUsername("sheetal_123"));
    }
}
```


# Example Usage in Selenium/TestNG Assertion
---

```java
String email = driver.findElement(By.id("email")).getText();

Assert.assertTrue(
        RegexUtils.isValidEmail(email),
        "Invalid Email Format");

```

# Example Usage in API Testing (RestAssured)
---

```java
String panNumber =
        response.jsonPath().getString("customer.pan");

Assert.assertTrue(
        RegexUtils.isValidPAN(panNumber),
        "Invalid PAN Number");

```

