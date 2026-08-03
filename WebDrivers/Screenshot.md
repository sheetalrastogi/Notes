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





