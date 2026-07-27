# Major Selenium 4 Enhancements Using Chrome DevTools Protocol (CDP)

Selenium 4 introduced native integration with the **Chrome DevTools Protocol (CDP)**, enabling testers to interact directly with browser internals that were not possible with traditional WebDriver APIs. 【1-c86ba0】

---

# 1. Network Monitoring & Interception

Capture all network requests and responses.

## Use Cases

- Validate API calls
- Monitor request headers
- Verify response codes
- Capture payloads
- Analyze REST API traffic

## Example

```java
DevTools devTools = ((ChromeDriver) driver).getDevTools();
devTools.createSession();

devTools.send(Network.enable(
        Optional.empty(),
        Optional.empty(),
        Optional.empty()));

devTools.addListener(
        Network.responseReceived(),
        response -> System.out.println(
                response.getResponse().getUrl()));
```

## Benefits

- ✅ API validation without external tools
- ✅ Request/Response inspection
- ✅ Backend troubleshooting

---

# 2. Request Blocking

Block specific URLs, domains, or resource types.

## Example

```java
devTools.send(Network.setBlockedURLs(
        List.of("*.css", "*.png")));
```

## Use Cases

- Negative testing
- Verify broken UI behavior
- Simulate unavailable services
- Security testing

---

# 3. Network Throttling

Simulate slow internet conditions.

## Example

```java
devTools.send(Network.emulateNetworkConditions(
        false,
        100,
        20000,
        10000,
        Optional.empty()));
```

## Simulate

- 2G
- 3G
- 4G
- High latency
- Packet loss

## Useful For

- Performance testing
- Mobile simulation
- Resilience testing

---

# 4. Override Geolocation

Change browser location without VPN.

## Example

```java
devTools.send(Emulation.setGeolocationOverride(
        Optional.of(19.0760),
        Optional.of(72.8777),
        Optional.of(1)));
```

## Use Cases

- Location-based pricing
- Geo-restriction testing
- Maps testing
- Insurance quoting applications

---

# 5. Device Emulation

Simulate mobile devices.

## Example

```java
devTools.send(Emulation.setDeviceMetricsOverride(
        390,
        844,
        100,
        true,
        Optional.empty(),
        Optional.empty(),
        Optional.empty(),
        Optional.empty(),
        Optional.empty(),
        Optional.empty(),
        Optional.empty(),
        Optional.empty(),
        Optional.empty()));
```

## Simulate

- iPhone
- Android
- Tablets

## Benefits

- No physical device required

---

# 6. Capture Console Logs

Read browser console messages.

## Example

```java
devTools.send(Log.enable());

devTools.addListener(
        Log.entryAdded(),
        log -> System.out.println(log.getText()));
```

## Useful For

- JavaScript exceptions
- Browser errors
- Runtime failures

---

# 7. Performance Metrics Collection

Retrieve browser performance metrics.

## Example

```java
devTools.send(Performance.enable());

List<Metric> metrics =
        devTools.send(Performance.getMetrics());
```

## Metrics

- CPU usage
- JS heap size
- DOM nodes
- Layout count
- Reflow count

---

# 8. Capture Browser Events

Listen to browser-level events.

## Events

- Navigation start
- Page load
- DOM ready
- Request sent
- Response received

## Use Cases

- Debug synchronization issues
- Measure page performance

---

# 9. Execute CDP Commands Directly

Selenium 4 allows raw CDP command execution.

## Example

```java
Map<String, Object> params = new HashMap<>();

driver.executeCdpCommand(
        "Network.enable",
        params);
```

## Useful For

- Unsupported Selenium features
- Chrome-specific capabilities

---

# 10. Modify User Agent

Spoof browser information.

## Example

```java
devTools.send(Network.setUserAgentOverride(
        "MyCustomAgent",
        Optional.empty(),
        Optional.empty(),
        Optional.empty()));
```

## Use Cases

- Browser compatibility testing
- Bot detection testing
- Mobile browser simulation

---

# 11. Emulate Offline Mode

Simulate internet disconnection.

## Example

```java
devTools.send(Network.emulateNetworkConditions(
        true,
        0,
        0,
        0,
        Optional.empty()));
```

## Useful For

- Offline applications
- Progressive Web Apps (PWA)
- Synchronization testing

---

# 12. Download File Handling

Control download behavior.

## Example

```java
driver.executeCdpCommand(
        "Page.setDownloadBehavior",
        Map.of(
                "behavior", "allow",
                "downloadPath", "C:/Downloads"));
```

## Benefits

- Automated file download validation
- Eliminate download popups

---

# 13. Inject HTTP Headers

Add custom request headers.

## Example

```java
Map<String, Object> headers = new HashMap<>();

headers.put("Authorization", "Bearer token");

devTools.send(Network.setExtraHTTPHeaders(
        new Headers(headers)));
```

## Use Cases

- API authentication
- Security testing
- Header validation

---

# 14. Basic Authentication Without Popup

Handle browser authentication directly.

## Example

```java
Map<String, Object> auth = new HashMap<>();

auth.put("username", "admin");
auth.put("password", "admin");
```

## Benefits

- Avoid authentication popups
- CI/CD friendly

---

# 15. Capture JavaScript Coverage

Identify unused JavaScript.

## Useful For

- Performance optimization
- Frontend quality analysis
- Technical debt identification

---

# 16. Simulate CPU Throttling

Emulate slow devices.

## Example

```java
devTools.send(
        Emulation.setCPUThrottlingRate(4));
```

## Use Cases

- Performance testing
- Low-end device simulation

---

# 17. Security Certificate Validation

Monitor SSL/TLS issues.

## Verify

- Invalid certificates
- Expired certificates
- TLS errors

## Useful For

- Security testing
- Environment validation

---

# 18. Take Full-Page Screenshots

Capture the entire page instead of only the visible viewport.

## Example

```java
String screenshot =
        devTools.send(Page.captureScreenshot(
                Optional.empty(),
                Optional.empty(),
                Optional.empty(),
                Optional.of(true),
                Optional.empty()));
```

## Benefits

- Visual testing
- Evidence collection

---

# 19. Browser Cache Management

Clear cache programmatically.

## Example

```java
devTools.send(Network.clearBrowserCache());
```

## Use Cases

- Fresh-session testing
- Cache-related defect validation

---

# 20. Cookie Management

Manipulate browser cookies.

## Example

```java
devTools.send(Network.clearBrowserCookies());
```

## Use Cases

- Login testing
- Session validation
- Security testing

---

# Selenium 4 CDP Features Most Valuable for Test Architects

| Testing Area | CDP Capability |
|-------------|----------------|
| API Testing | Network Monitoring |
| Performance Testing | CPU & Network Throttling |
| Security Testing | Header Injection, SSL Validation |
| Mobile Testing | Device Emulation |
| Geo Testing | Geolocation Override |
| Debugging | Console Logs |
| Resilience Testing | Offline Mode |
| Visual Testing | Full Page Screenshots |
| Authentication Testing | Basic Auth Handling |
| Monitoring | Browser Performance Metrics |

---

# High-Value Enterprise Use Cases

For enterprise applications such as:

- Insurance
- Banking
- Healthcare
- Retail

The most commonly adopted Selenium 4 CDP capabilities are:

- Network Request/Response Validation
- API Mocking and Monitoring
- Performance Metrics Collection
- Mobile Device Emulation
- Geolocation Testing
- Custom Header Injection
- Offline/Network Failure Testing
- Download Automation
- Console Error Monitoring
- Security & Authentication Validation

---

# Summary

Selenium 4 CDP integration significantly extends Selenium beyond traditional UI automation by enabling:

- UI Testing
- API Validation
- Performance Testing
- Security Testing
- Network Monitoring
- Device Emulation

This allows organizations to perform **UI + API + Performance + Security validations within a single automation framework**, making CDP one of the most valuable Selenium 4 enhancements for modern Quality Engineering teams.
