## Selenium Java Screenshot for entire desktop, not just webview:

Use Case:
- Browser + Popup
- Native Applications
- PDF Viewers

```java

import java.awt.*;
import java.awt.image.BufferedImage;
import javax.imageio.ImageIO;

Robot robot = new Robot();

BufferedImage image = robot.createScreenCapture(new Rectangle(Toolkit.getDefaultToolkit().getScreenSize()));

ImageIO.write(image, "png", new File("desktop.png"));

```


## CDP for entire page Screenshot:

```java
import org.openqa.selenium.devtools.DevTools;
import org.openqa.selenium.devtools.v139.page.Page;

DevTools devTools = ((ChromeDriver) driver).getDevTools();

devTools.createSession();

devTools.send(Page.enable());

String base64 = devTools.send(Page.captureScreenshot());

System.out.println(base64);

```

Benefits
- Full page support
- Fast
- Chrome/Edge


## Screenshot Using AShot Library

Maven
```xml
<dependency>
    <groupId>ru.yandex.qatools.ashot</groupId>
    <artifactId>ashot</artifactId>
    <version>1.5.4</version>
</dependency>
```

Full Page Screenshot

```java
import ru.yandex.qatools.ashot.AShot;
import ru.yandex.qatools.ashot.Screenshot;
import ru.yandex.qatools.ashot.shooting.ShootingStrategies;

Screenshot screenshot = new AShot().shootingStrategy(ShootingStrategies.viewportPasting(100))
				.takeScreenshot(driver);

ImageIO.write(screenshot.getImage(), "PNG", new File("fullpage.png"));

```

## Selenium Grid Screenshots
--------------------------------------


During Selenium Grid execution, screenshots are typically captured on the remote node machine where the browser session is running. The challenge is consolidating screenshots from multiple nodes (Chrome, Firefox, Edge, Windows, Linux, Docker containers, etc.) into a single centralized location or report.


## Option 1: Store Screenshots as Base64 (Recommended)

Instead of saving screenshots on node file systems, capture screenshots as Base64 and attach them directly to your reporting tool.

Advantages
- No dependency on node file paths
- Works across local, Grid, Docker, Cloud
- Ideal for Allure, Extent, ReportPortal

Example:

```java
public static String captureScreenshot(WebDriver driver) {
    return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BASE64);
}

```

Allure:
```java
@Attachment(value = "Failure Screenshot", type = "image/png")
public static byte[] attachScreenshot(WebDriver driver) {
    return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
}
```


Extent Report:

```java

String base64Image = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BASE64);

test.fail("Screenshot", MediaEntityBuilder.createScreenCaptureFromBase64String(base64Image).build());

```


## Option 2: Central Shared Screenshot Repository

Configure all nodes to write screenshots to a common network location.

```text
+------------+
| Test Runner|
+-----+------+
      |
      v
+-------------+
| Selenium Hub|
+------+------+ 
       |
       +-------------------------+
       |                         |
       v                         v
+-------------+         +-------------+
| Chrome Node |         | Firefox Node|
+------+------+         +------+------+
       |                       |
       +-----------+-----------+
                   |
                   v
         Shared Network Drive
         \\fileserver\screenshots

```

Screenshot Utility:

```java
public class ScreenshotUtil {

    private static final String BASE_DIR = "\\\\fileserver\\screenshots\\";

    public static String capture(WebDriver driver, String testName) throws IOException {

        File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);

        String timestamp = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss"));

        String filename = testName + "_" + timestamp + ".png";

        File dest = new File(BASE_DIR + filename);

        FileUtils.copyFile(src, dest);

        return dest.getAbsolutePath();
    }
}

```





