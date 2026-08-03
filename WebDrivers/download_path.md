## Identify download location during Selenium Script Execution

# 1. Store the Download Directory When Creating ChromeDriver

```java
String downloadDir = System.getProperty("user.dir") + "\\downloads";

Map<String, Object> prefs = new HashMap<>();
prefs.put("download.default_directory", downloadDir);

ChromeOptions options = new ChromeOptions();
options.setExperimentalOption("prefs", prefs);

ChromeDriver driver = new ChromeDriver(options);
System.out.println("Download Directory: " + downloadDir);

```

# 2. With Chrome Dev Protocol

```java
ChromeDriver driver = new ChromeDriver();

driver.executeCdpCommand(
        "Browser.setDownloadBehavior",
        Map.of(
            "behavior", "allow",
            "downloadPath", "C:\\Downloads"
        )
);

```
