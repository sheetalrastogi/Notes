## Common Java KeyStore Commands and Operations

# 1. Create an Empty KeyStore

```java 
KeyStore keyStore = KeyStore.getInstance("JKS");
keyStore.load(null, null);
```

# 2. Load an Existing KeyStore
```java 
KeyStore keyStore = KeyStore.getInstance("PKCS12");

try (FileInputStream fis = new FileInputStream("client.p12")) {
    keyStore.load(fis, "password".toCharArray());
}
```

# 3. Get All Aliases
```java 
Enumeration<String> aliases = keyStore.aliases();

while (aliases.hasMoreElements()) {
    System.out.println(aliases.nextElement());
}
```

# 4. Check Whether Alias Exists
```java 
boolean exists = keyStore.containsAlias("mycert");

System.out.println(exists);
```


# 5. Get Certificate by Alias
```java 
Certificate cert = keyStore.getCertificate("mycert");

System.out.println(cert);
```

# 6. Get Certificate Expiry Date
```java 
X509Certificate cert = (X509Certificate) keyStore.getCertificate("mycert");

System.out.println("Expires On: " + cert.getNotAfter());
```


# 7. Get Certificate Subject
```java 
X509Certificate cert = (X509Certificate) keyStore.getCertificate("mycert");

System.out.println(cert.getSubjectX500Principal());
```
Example Output:
```java 
CN=api.company.com, OU=IT, O=Company, C=IN
```


# 8. Get Certificate Issuer
```java 
X509Certificate cert = (X509Certificate) keyStore.getCertificate("mycert");

System.out.println(cert.getIssuerX500Principal());
```

#9. Check Certificate Expiration
```java 
X509Certificate cert = (X509Certificate) keyStore.getCertificate("mycert");
try {
    cert.checkValidity();
    System.out.println("Certificate is valid");
} catch (CertificateExpiredException ex) {
    System.out.println("Certificate expired");
} catch (CertificateNotYetValidException ex) {
    System.out.println("Certificate not yet valid");
}
```

# 10. Retrieve Private Key

```java 
PrivateKey privateKey = (PrivateKey)keyStore.getKey("client-cert", "password".toCharArray());

System.out.println(privateKey.getAlgorithm());
```

# 11. Add Certificate to KeyStore
```java 
Certificate certificate = cert;

keyStore.setCertificateEntry("newAlias", certificate);
```

# 12. Delete Certificate
```java 
keyStore.deleteEntry("oldAlias");
```


# 13. Save Updated KeyStore
```java
try (FileOutputStream fos = new FileOutputStream("updated.jks")) {
    keyStore.store(fos, "changeit".toCharArray());
}
```

# 14. Count Entries in KeyStore
```java
System.out.println("Total Entries = " + keyStore.size());
```


# 15. Check Entry Type
```java
if (keyStore.isKeyEntry("client-cert")) {
    System.out.println("Private Key Entry");
}

if (keyStore.isCertificateEntry("ca-cert")) {
    System.out.println("Certificate Entry");
}
```

16. Extract Certificate Chain
```java
Certificate[] chain = keyStore.getCertificateChain("client-cert");

for (Certificate cert : chain) {
    System.out.println(cert);
}
```

## Useful KeyStore Types

- KeyStore.getInstance("JKS");      // Traditional Java Keystore
- KeyStore.getInstance("PKCS12");   // .p12 / .pfx
- KeyStore.getInstance("JCEKS");    // Supports secret keys
- KeyStore.getInstance("PKCS11");   // HSM / Smart Card





