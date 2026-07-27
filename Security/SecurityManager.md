## Import certificate to Java Key Store (JKS)
----------------------------------------------------

Java Keystore (JKS), you can import an SSL certificate (.cer, .crt, .pem) into a JKS truststore using the Java keytool utility.

## 1. Import SSL Certificate into JKS

Command

```text

keytool -importcert \
-alias company-cert \
-file company-cert.cer \
-keystore truststore.jks \
-storepass changeit

```

Paramaters:

```text

-alias       Unique certificate name
-file        Certificate file
-keystore    JKS file
-storepass   Keystore password

```

## 2. Verify Certificate Added Successfully

```text

keytool -list \
-v \
-keystore truststore.jks \
-storepass changeit

```

You should see:

```text
Alias name: company-cert
Owner: CN=company.com
Issuer: CN=Company Root CA
Valid from: ...
Valid until: …
```


## 3. Convert PEM Certificate to JKS

in case you have server.pem

Import it to JKS as follow:

```text

keytool -importcert \
-alias server-cert \
-file server.pem \
-keystore truststore.jks

```

## 4. Import PKCS12 (.p12/.pfx) Into JKS

in case you have client-cert.p12

```text

keytool -importkeystore \
-srckeystore client-cert.p12 \
-srcstoretype PKCS12 \
-destkeystore client-cert.jks \
-deststoretype JKS

```
example usage:

```text

keytool -importkeystore \
-srckeystore client.p12 \
-srcstorepass password123 \
-srcstoretype PKCS12 \
-destkeystore client.jks \
-deststorepass changeit \
-deststoretype JKS

```


## 5. Use JKS in Selenium Automation
Java System Properties

```java
System.setProperty(
    "javax.net.ssl.trustStore",
    "certs/truststore.jks");

System.setProperty(
    "javax.net.ssl.trustStorePassword",
    "changeit");

```


For Mutual LTS:

```java

System.setProperty("javax.net.ssl.keyStore", "certs/client.jks");

System.setProperty("javax.net.ssl.keyStorePassword", "changeit");

```


## 6. Verify Certificate Programmatically


```java

KeyStore keyStore = KeyStore.getInstance("JKS");

try (FileInputStream fis =
         new FileInputStream("truststore.jks")) {

    keyStore.load(fis, "changeit".toCharArray());
}

Certificate cert =
        keyStore.getCertificate("company-cert");

System.out.println(cert);

```





## 7. Enterprise Example Folder Structure

```text
project
|
+-- certs
|    |
|    +-- company-cert.cer
|    +-- truststore.jks
|    +-- client.jks
|
+-- src
|
+-- selenium-tests
```

**Common Keytool Commands**

# List certificates
keytool -list -keystore truststore.jks

# Delete certificate
keytool -delete -alias company-cert -keystore truststore.jks

# Export certificate
keytool -exportcert -alias company-cert -keystore truststore.jks -file export.cer

# Import certificate
keytool -importcert -alias company-cert -file company-cert.cer -keystore truststore.jks

