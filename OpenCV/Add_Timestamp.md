## Example to add a timestamp to Screenshot in Selneium Java 

If you want to capture Selenium screenshots and add a timestamp before saving them to reports (Extent, Allure, etc.), OpenCV provides the **Imgproc.putText()** method


Maven Dependency

```xml
<dependency>
    <groupId>org.openpnp</groupId>
    <artifactId>opencv</artifactId>
    <version>4.9.0-0</version>
</dependency>
```


Utility Method

```java
import nu.pattern.OpenCV;
import org.opencv.core.*;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class ImageTimestampUtil {

	static {
		OpenCV.loadLocally();
	}

	public static void addTimestamp(String inputImage, String outputImage) {

		Mat image = Imgcodecs.imread(inputImage);

		String timestamp = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));

		Imgproc.putText(image, timestamp, new Point(20, 40), // x,y position
				Imgproc.FONT_HERSHEY_SIMPLEX, 1.0, // font size
				new Scalar(0, 255, 0), // green color
				2 // thickness
		);

		Imgcodecs.imwrite(outputImage, image);

		System.out.println("Timestamp added successfully");
	}
}

```

Usage:

ImageTimestampUtil.addTimestamp("screenshot.png", "timestamped_screenshot.png");


# Usage with Selenium Screenshot:

```java

		File screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);

		String screenshotPath = "target/screenshots/homepage.png";

		Files.copy(screenshot.toPath(), Paths.get(screenshotPath), StandardCopyOption.REPLACE_EXISTING);

		ImageTimestampUtil.addTimestamp(screenshotPath, screenshotPath);


```

Result:

```text
----------------------------------------------------
|                                                  |
| 2026-08-03 15:35:20                              |
|                                                  |
|        Selenium Screenshot                       |
|                                                  |
----------------------------------------------------
```


# Add Browser and Test Information

A common framework enhancement is to stamp additional execution details.

```java
		String watermark = String.format("Executed: %s | Browser: %s | Environment: %s", timestamp, "Chrome", "QA");

		Imgproc.putText(image, watermark, new Point(20, 40), Imgproc.FONT_HERSHEY_SIMPLEX, 0.7, new Scalar(255, 0, 0),
				2);

```

Output:

Executed: 2026-08-03 15:35:20 |
Browser: Chrome |
Environment: QA




