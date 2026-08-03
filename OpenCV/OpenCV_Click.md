## Examples to click on an image with OpenCV

In Selenium Java 4, OpenCV can be used to locate an image on the screen and then Selenium can perform a click at the detected coordinates.

Useful when:

- Elements are inside a <canvas>
- Elements do not have stable locators
- Image-based validation is required
- Third-party controls hide DOM elements
- Appium image recognition scenarios

# Example 1: Click an Icon Using Template Matching

Step 1: Save Reference Image

```text
project
│
├── target.png      <-- image to find
├── screenshot.png  <-- captured page

```

# Step 2: Locate Image

```java

import nu.pattern.OpenCV;
import org.opencv.core.*;
import org.opencv.imgcodecs.Imgcodecs;
import org.opencv.imgproc.Imgproc;

public class ImageLocator {

	static {
		OpenCV.loadLocally();
	}

	public static Point findImage(String screenshot, String template) {

		Mat source = Imgcodecs.imread(screenshot);
		Mat target = Imgcodecs.imread(template);

		Mat result = new Mat();

		Imgproc.matchTemplate(source, target, result, Imgproc.TM_CCOEFF_NORMED);

		Core.MinMaxLocResult match = Core.minMaxLoc(result);

		return match.maxLoc;
	}
}
```

# Step 3: Click Using Selenium Actions


```java

		Point location = ImageLocator.findImage("screenshot.png", "target.png");

		Actions actions = new Actions(driver);

		actions.moveByOffset((int) location.x, (int) location.y).click().perform();

```


## Alternatively find centre of image & click:

```java

		Mat source = Imgcodecs.imread("screen.png");
		Mat target = Imgcodecs.imread("button.png");

		Mat result = new Mat();

		Imgproc.matchTemplate(source, target, result, Imgproc.TM_CCOEFF_NORMED);

		Core.MinMaxLocResult mmr = Core.minMaxLoc(result);

		int centerX = (int) (mmr.maxLoc.x + target.cols() / 2);

		int centerY = (int) (mmr.maxLoc.y + target.rows() / 2);

		new Actions(driver).moveByOffset(centerX, centerY).click().perform();

```


## Another approach - Use Selenium Screenshot and then OpenCV to perform Click

```java
		File file = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);

		Files.copy(file.toPath(), Paths.get("screen.png"));

```

Locate image:

```java

		Point point = findImage("screen.png", "loginButton.png");

```

Click action

```java

		new Actions(driver).moveByOffset((int) point.x, (int) point.y).click().perform();

```


