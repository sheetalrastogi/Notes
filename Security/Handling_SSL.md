## Programmatically loading a client certificate (PFX/P12/JKS) into the browser/session
------------------------------------------------------------------------------------------

To enable Selenium can authenticate against a certificate-protected website


Use Case

Many enterprise applications require:

- Mutual TLS (mTLS)
- Client Certificate Authentication
- Smart Card Authentication
- PKI-based Login
- Internal CA Certificates

Flow:
```text
Selenium
    |
    ├── Chrome Browser
    |
    ├── SSLContext
    |
    ├── KeyStore (.p12/.pfx/.jks)
    |
    └── TrustManager
              |
              └── Client Certificate Authentication

```


## Example 1: Load Client Certificate Using Java KeyStore

```text
Assume:

client-cert.p12
password = changeit

```

SSL Security Manager Utility:

```java

import java.io.FileInputStream;
import java.security.KeyStore;

import javax.net.ssl.KeyManagerFactory;
import javax.net.ssl.SSLContext;
import javax.net.ssl.TrustManagerFactory;

public class SecurityManagerUtil {

    public static SSLContext createSSLContext(
            String certificatePath,
            String password) throws Exception {

        KeyStore keyStore = KeyStore.getInstance("PKCS12");

        try (FileInputStream fis =
                     new FileInputStream(certificatePath)) {

            keyStore.load(fis, password.toCharArray());
        }

        KeyManagerFactory kmf =
                KeyManagerFactory.getInstance(
                        KeyManagerFactory.getDefaultAlgorithm());

        kmf.init(keyStore, password.toCharArray());

        TrustManagerFactory tmf =
                TrustManagerFactory.getInstance(
                        TrustManagerFactory.getDefaultAlgorithm());

        tmf.init(keyStore);

        SSLContext sslContext =
                SSLContext.getInstance("TLS");

        sslContext.init(
                kmf.getKeyManagers(),
                tmf.getTrustManagers(),
                null);

        return sslContext;
    }
}

```


## Example 2: Configure Security Properties Before Launching Selenium


```java

public class CertificateConfigurator {

    public static void configure() {

        System.setProperty(
                "javax.net.ssl.keyStore",
                "certs/client-cert.p12");

        System.setProperty(
                "javax.net.ssl.keyStorePassword",
                "changeit");

        System.setProperty(
                "javax.net.ssl.keyStoreType",
                "PKCS12");

        System.setProperty(
                "javax.net.ssl.trustStore",
                "certs/truststore.jks");

        System.setProperty(
                "javax.net.ssl.trustStorePassword",
                "changeit");
    }
}

```


## Example 3: Selenium Driver Factory


```java

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

public class DriverFactory {

    public static WebDriver getDriver() {

        CertificateConfigurator.configure();

        ChromeOptions options = new ChromeOptions();

        options.setAcceptInsecureCerts(true);

        return new ChromeDriver(options);
    }
}

```

Usage:

```java

WebDriver driver = DriverFactory.getDriver();

driver.get("https://secure-app.company.com");

```


## Example 4: Automatically Install Certificate Into Chrome Profile

Create a Chrome profile that already contains the certificate.

```java

ChromeOptions options = new ChromeOptions();

options.addArguments("--user-data-dir=C:\\ChromeProfile");

WebDriver driver = new ChromeDriver(options);

```

This is a very common enterprise approach because:

- Certificate already installed
- No certificate popup
- Easy CI/CD execution



## Advanced Framework Example

```java

public class DriverFactory {

    public static WebDriver getDriver() {

        SecurityManager.initializeSecurity();

        ChromeOptions options =
                new ChromeOptions();

        options.setAcceptInsecureCerts(true);

        options.addArguments("--start-maximized");

        options.addArguments("--disable-web-security");

        return new ChromeDriver(options);
    }
}
```

Real Enterprise Test Cases

```java

SSL Certificate Validation
Assert.assertTrue(
        driver.getCurrentUrl()
              .startsWith("https"));
```

Verify Certificate Expiry

```java

X509Certificate cert = (X509Certificate) keyStore.getCertificate("client");

Date expiryDate = cert.getNotAfter();

Assert.assertTrue(expiryDate.after(new Date()));


// Verify Certificate Subject

String subject = cert.getSubjectX500Principal().getName();

Assert.assertTrue(subject.contains("CompanyName"));


// Verify TLS Protocol
SSLContext context = SSLContext.getDefault();

System.out.println(context.getProtocol());




**Recommended Enterprise Automation framework Architecture**

```text
src/test/java
|
+-- security
|    |
|    +-- SecurityManager.java
|    +-- CertificateLoader.java
|    +-- SSLContextFactory.java
|
+-- driver
|    |
|    +-- DriverFactory.java
|
+-- tests
|
+-- resources
     |
     +-- client-cert.p12
     +-- truststore.jks
```
