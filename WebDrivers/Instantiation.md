Supported WebDriver executions:

- Local Chrome / Firefox / Edge

```java
WebDriver driver = new ChromeDriver();

ChromeOptions options = new ChromeOptions();
	options.addArguments("--start-maximized");
	options.addArguments("--incognito");
	options.addArguments("--headless=new");
	options.addArguments("--window-size=1920,1080");
    options.addArguments("--disable-gpu");
    options.addArguments("--no-sandbox");
    options.addArguments("--disable-dev-shm-usage");
    options.addArguments("--window-size=1920,1080");


	WebDriver driver = new ChromeDriver(options);


WebDriver driver = new EdgeDriver();

EdgeOptions options = new EdgeOptions();
        options.addArguments("--start-maximized");
        options.addArguments("--inprivate");
	    options.addArguments("--headless=new");
		options.addArguments("--window-size=1920,1080");

        WebDriver driver = new EdgeDriver(options);


WebDriver driver = new FirefoxDriver();

FirefoxOptions options = new FirefoxOptions();
        options.addArguments("-private");
		options.addArguments("-headless");

        WebDriver driver = new FirefoxDriver(options);
```



##1. Launch Chrome with Existing User Profile

**Chrome profiles are useful when you want Selenium to launch Chrome with**:

- Existing bookmarks
- Saved cookies
- Extensions
- Corporate certificates
- SSO/MFA configuration
- Pre-authenticated sessions


Typical Windows Platform Profile Location

```text
	C:\Users\<username>\AppData\Local\Google\Chrome\User Data
```

**Common profiles**:
- Default
- Profile 1
- Profile 2
- Profile 3

**Example WebDriver Instantiation with Default profile**:
```java
        ChromeOptions options = new ChromeOptions();

        options.addArguments("--user-data-dir=C:\\Users\\xxx\\AppData\\Local\\Google\\Chrome\\User Data");
        options.addArguments("--profile-directory=Default");

        WebDriver driver = new ChromeDriver(options);
```


##2. Launch Specific Chrome Profile

```java
ChromeOptions options = new ChromeOptions();
options.addArguments("--user-data-dir=C:\\Users\\xxx\\AppData\\Local\\Google\\Chrome\\User Data");
options.addArguments("--profile-directory=Profile 2");

WebDriver driver = new ChromeDriver(options);
```

##3. Create Dedicated Automation Profile

Create a separate profile:
```text
	chrome.exe --profile-directory="AutomationProfile"
```
Then use:
```java
ChromeOptions options = new ChromeOptions();
options.addArguments("--user-data-dir=D:\\ChromeProfiles");
options.addArguments("--profile-directory=AutomationProfile");

WebDriver driver = new ChromeDriver(options);
```




- Selenium Grid 

```java
        ChromeOptions options = new ChromeOptions();

        options.addArguments("--start-maximized");

        WebDriver driver = new RemoteWebDriver(new URL("http://localhost:4444"), options);
        driver.get("https://www.google.com");
```


- Docker Selenium

**Start Selenium Docker Container**

Standalone Chrome

```text	
	docker run -d -p 4444:4444 -p 7900:7900 selenium/standalone-chrome:latest
```

Grid URL:

```text
	http://localhost:4444
```

```java
        ChromeOptions options = new ChromeOptions();

        WebDriver driver = new RemoteWebDriver(new URL("http://localhost:4444"), options);

```


Other Docker images:
----------------------------

docker run -d -p 4444:4444 selenium/standalone-firefox:latest

docker run -d -p 4444:4444 selenium/standalone-edge:latest



## Docker compose file for Selenium Grid 4

docker-compose.yml
```text
version: "3.8"

services:

  selenium-hub:
    image: selenium/hub:latest
    container_name: selenium-hub
    ports:
      - "4444:4444"

  chrome:
    image: selenium/node-chrome:latest
    shm_size: 2gb
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443

  firefox:
    image: selenium/node-firefox:latest
    shm_size: 2gb
    depends_on:
      - selenium-hub
```

**Start docker image**:
```text
	docker-compose up -d
```

**Verify Grid by opening following URL**:
```text
	http://localhost:4444/ui
```

**Hub URL**:
```text
http://localhost:4444
```


**VNC Access**
```text
	http://localhost:7900

Password: secret
```



## Docker Compose with Chrome + Video Recording

```text
version: '3.8'

services:

  chrome:
    image: selenium/standalone-chrome:latest

    ports:
      - "4444:4444"
      - "7900:7900"

    shm_size: "2gb"

    environment:
      - SE_VNC_NO_PASSWORD=1

    volumes:
      - ./downloads:/home/seluser/Downloads

```

**Useful Docker Commands**

- Pull Latest Chrome Image
```text
	docker pull selenium/standalone-chrome:latest
```

- Start
```text
	docker compose up -d
```

- Stop
```text
	docker compose down
```

- View Logs
```text
	docker logs selenium-chrome
```

- Running Containers
```text
	docker ps
```


-----------------------------


##Chrome Mobile Emulation:
--------------------------

```java
        ChromeOptions options = new ChromeOptions();
        options.setExperimentalOption("mobileEmulation", Map.of("deviceName", "iPhone 14 Pro"));

        WebDriver driver = new ChromeDriver(options);
```


##Edge Mobile Emulation:
-----------------------

```java
        EdgeOptions options = new EdgeOptions();
        options.setExperimentalOption("mobileEmulation", Map.of("deviceName", "iPhone 14 Pro"));

        WebDriver driver = new EdgeDriver(options);
```

##Custom Mobile Emulation:
----------------------------

Instead of using predefined devices, define your own device as follow:

```java
ChromeOptions options = new ChromeOptions();

Map<String, Object> mobileEmulation = new HashMap<>();
	mobileEmulation.put("width", 412);
	mobileEmulation.put("height", 915);
	mobileEmulation.put("pixelRatio", 3.0);

options.setExperimentalOption("mobileEmulation", mobileEmulation);

WebDriver driver = new ChromeDriver(options);
```


