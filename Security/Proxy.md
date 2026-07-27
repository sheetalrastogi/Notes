## BrowserMob Proxy Integration

Useful for:
- Network monitoring
- HAR file creation
- Request/Response validation

**Maven Dependency**
```xml
<dependency>
    <groupId>net.lightbody.bmp</groupId>
    <artifactId>browsermob-core</artifactId>
    <version>2.1.5</version>
</dependency>
```

**Example**

```java

BrowserMobProxy proxy =
        new BrowserMobProxyServer();

proxy.start();

Proxy seleniumProxy =
        ClientUtil.createSeleniumProxy(proxy);

ChromeOptions options =
        new ChromeOptions();

options.setProxy(seleniumProxy);

WebDriver driver =
        new ChromeDriver(options);

proxy.newHar("GoogleTest");

driver.get("https://www.google.com");

Har har = proxy.getHar();

```


## Capture API Traffic Through Proxy

```java
for (HarEntry entry :
        har.getLog().getEntries()) {

    System.out.println(
        entry.getRequest().getUrl());

    System.out.println(
        entry.getResponse().getStatus());
}

```

Typical validate:
```java
Assert.assertEquals(
    entry.getResponse().getStatus(),
    200);
```

**Common QA Use Cases for Proxy Configuration**
- Network Validation
    Verify API calls pass through proxy

- Corporate Firewall Testing
    Verify application works behind organization proxy

- Geolocation Testing
    Route traffic through country-specific proxy

- Performance Testing
    Measure network latency and bandwidth

- Security Testing
   Capture traffic using OWASP ZAP

- API Verification
   Inspect request/response payloads

**Recommended Enterprise Framework Structure**

```text
framework
|
+-- config
|    +-- proxy.properties
|
+-- driver
|    +-- DriverFactory.java
|
+-- proxy
|    +-- ProxyManager.java
|    +-- BrowserMobManager.java
|    +-- ZapProxyManager.java
|
+-- tests
|
+-- reports
     +-- HAR Files
```
