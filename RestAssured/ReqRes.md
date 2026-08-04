## RestAssured request processing:
---

# Route Request Through Burp Suite

Useful during API security testing.

```java
given()
    .proxy("localhost", 8080)
    .when()
    .get("https://api.company.com");
```

Burp will capture:

- Headers
- Cookies
- JWT Tokens
- Request Body
- Response Body


# Send requests through an HTTP/HTTPS proxy server

Add proxy-related headers such as:
- X-Forwarded-For
- X-Forwarded-Host
- X-Forwarded-Proto
- Forwarded

These headers are commonly used for **API security testing**, **gateway testing**, **load balancer testing**, and **microservices validation**.


**Add X-Forwarded-xxx Headers to simulate the client IP address**
```java
given()
    .header("X-Forwarded-For", "192.168.1.10")
    .header("X-Forwarded-Host", "qa.mycompany.com")
    .header("X-Forwarded-Proto", "https")
    .when()
    .get("/api/orders")
    .then()
    .statusCode(200);
```


**Using Header Map**

Enterprise frameworks often use a map.

```java

Map<String, String> proxyHeaders = new HashMap<>();
 
 proxyHeaders.put("X-Forwarded-For", "10.1.1.200");
 proxyHeaders.put("X-Forwarded-Host", "uat.company.com");
 proxyHeaders.put("X-Forwarded-Proto", "https");

given()
    .headers(proxyHeaders)
    .when()
    .get("/api/users")
    .then()
    .statusCode(200);

```


## Request processing via Client Certificate
---

When an API requires **Mutual TLS (mTLS)** or **client certificate authentication**, RestAssured can send a request signed with a local certificate stored in:

- PKCS12 (.p12, .pfx)
- Java Keystore (.jks)
- TrustStore



# Option 1: Using PKCS12 (.p12/.pfx) Certificate
---

Certificate:  client-cert.p12

RestAssured Example

```java
import io.restassured.RestAssured;

import java.io.File;

public class MtlsApiTest {

    public static void main(String[] args) {

        RestAssured
            .given()
                .keystore(
                        new File("certs/client-cert.p12"),
                        "certificatePassword")
            .when()
                .get("https://api.company.com/customers")
            .then()
                .statusCode(200);
    }
}
```


# Option 2: Using Java KeyStore (JKS)
Files
client-keystore.jks
client-truststore.jks

Step 1: Configure SSL
```java
import io.restassured.RestAssured;
import io.restassured.config.SSLConfig;

RestAssured.config =
        RestAssured.config()
                .sslConfig(
                    SSLConfig.sslConfig()
                        .with()
                        .keyStore(
                             "certs/client-keystore.jks",
                             "keystorePassword")
                        .trustStore(
                             "certs/client-truststore.jks",
                             "truststorePassword"));
```
Sample Request:
```java
given()
.when()
    .get("https://secured-api.company.com")
.then()
    .statusCode(200);

```

# Option 3: KeyStore Loaded Programmatically

Useful when certificates come from configuration.

```java
KeyStore keyStore = KeyStore.getInstance("PKCS12");

try(FileInputStream fis = new FileInputStream("client.p12")) {

    keyStore.load(fis, "password".toCharArray());
}

KeyStore trustStore = KeyStore.getInstance("JKS");

try (FileInputStream fis = new FileInputStream(trustStorePath)) {
	trustStore.load(fis, trustStorePassword.toCharArray());
}


// Configure SSL Context:

SSLContext sslContext = SSLContexts.custom()
                .loadKeyMaterial(keyStore, "password".toCharArray())
                .build();


// Execute API Request
        Response response = RestAssured
                .given()
                .relaxedHTTPSValidation()
                .config(
                        RestAssured.config()
                                .sslConfig(
                                        io.restassured.config.SSLConfig
                                                .sslConfig()
                                                .sslSocketFactory(
                                                        sslContext.getSocketFactory())
                                )
                )
                .header("Content-Type", "application/json")
                .when()
                .get("https://secured-api.company.com/customers");

        System.out.println("Status Code : " + response.getStatusCode());

        System.out.println(response.asPrettyString());

```


# Option 4: If you don't need to build SSLContext yourself:

```java
RestAssured
    .given()
    .keystore(
            "certs/client.p12",
            "password")
    .when()
    .get("https://secured-api.company.com/customers")
    .then()
    .statusCode(200);

```


**Notes**:   

# 1. Debug Certificate Aliases
-----------------------------------------

KeyStore keyStore = KeyStore.getInstance("PKCS12");

keyStore.load(new FileInputStream("certs/client.p12"),"password".toCharArray());

while (keyStore.aliases().hasMoreElements()) {

    System.out.println(keyStore.aliases().nextElement());

}


# 2. Convert PFX to JKS
-----------------------------------

```text

Certificate received:	client.pfx

Convert to JKS:

keytool -importkeystore \
-srcstoretype PKCS12 \
-srckeystore client.pfx \
-destkeystore client.jks

```

# 3. Common SSL/mTLS Errors

- PKIX Path Building Failed

Cause: Server certificate not trusted
Fix:   .trustStore(...)


- Bad Certificate
```text
SSLHandshakeException: bad_certificate

Cause:	Wrong client certificate
	Certificate expired
```






