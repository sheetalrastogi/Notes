## Image comparison with OpenCV:

OpenCV provides multiple ways to compare images in Java depending on your objective:

- Pixel-by-Pixel Comparison → Exact match validation.
- Difference Detection → Highlight changed regions.
- Similarity Percentage → Measure how similar two images are.
- Template Matching → Find an image within another image.
- Feature Matching (SIFT/ORB) → Compare screenshots despite scaling or UI changes.


# 1. Exact Pixel Comparison

Useful when validating downloaded images, logos, generated reports, etc.

```xml
<dependency>
    <groupId>org.openpnp</groupId>
    <artifactId>opencv</artifactId>
    <version>4.9.0-0</version>
</dependency>
```


```java
import org.opencv.core.Core;
import org.opencv.core.Mat;
import org.opencv.imgcodecs.Imgcodecs;

public class ExactImageCompare {

    static {
        nu.pattern.OpenCV.loadLocally();
    }

    public static void main(String[] args) {

        Mat img1 = Imgcodecs.imread("image1.png");
        Mat img2 = Imgcodecs.imread("image2.png");

        if (img1.rows() != img2.rows() ||
            img1.cols() != img2.cols()) {

            System.out.println("Images are different sizes");
            return;
        }

        Mat diff = new Mat();
        Core.compare(img1, img2, diff, Core.CMP_NE);

        int nonZeroPixels = Core.countNonZero(diff.reshape(1));

        if (nonZeroPixels == 0) {
            System.out.println("Images are identical");
        } else {
            System.out.println("Images differ");
        }
    }
}
```


# 2. Percentage Similarity

Useful in Selenium/Appium visual testing.

```java

import org.opencv.core.*;
import org.opencv.imgcodecs.Imgcodecs;

public class SimilarityCheck {

    static {
        nu.pattern.OpenCV.loadLocally();
    }

    public static void main(String[] args) {

        Mat img1 = Imgcodecs.imread("expected.png");
        Mat img2 = Imgcodecs.imread("actual.png");

        Mat diff = new Mat();

        Core.absdiff(img1, img2, diff);

        int differentPixels =
                Core.countNonZero(diff.reshape(1));

        double totalPixels =
                img1.rows() * img1.cols() * img1.channels();

        double similarity =
                100 - ((differentPixels / totalPixels) * 100);

        System.out.printf("Similarity = %.2f%%", similarity);
    }
}


# 3. Generate Difference Image

Useful when reporting Selenium failures.


```java

Mat image1 = Imgcodecs.imread("expected.png");
Mat image2 = Imgcodecs.imread("actual.png");

Mat diff = new Mat();

Core.absdiff(image1, image2, diff);

Imgcodecs.imwrite("difference.png", diff);


```

output:  (Changed pixels appear highlighted.)

expected.png
actual.png
difference.png





# 4. Feature Matching using ORB (Recommended)

Best for Selenium/Appium visual validation when:

- Resolution changes
- Browser zoom changes
- Minor UI movement
- Mobile device screenshots

```java

import org.opencv.core.*;
import org.opencv.features2d.*;
import org.opencv.imgcodecs.Imgcodecs;

public class ORBComparison {

    static {
        nu.pattern.OpenCV.loadLocally();
    }

    public static void main(String[] args) {

        Mat img1 = Imgcodecs.imread("img1.png", 0);
        Mat img2 = Imgcodecs.imread("img2.png", 0);

        ORB orb = ORB.create();

        MatOfKeyPoint kp1 = new MatOfKeyPoint();
        MatOfKeyPoint kp2 = new MatOfKeyPoint();

        Mat desc1 = new Mat();
        Mat desc2 = new Mat();

        orb.detectAndCompute(img1, new Mat(), kp1, desc1);
        orb.detectAndCompute(img2, new Mat(), kp2, desc2);

        DescriptorMatcher matcher =
                BFMatcher.create(Core.NORM_HAMMING, true);

        MatOfDMatch matches = new MatOfDMatch();

        matcher.match(desc1, desc2, matches);

        System.out.println(
                "Matches Found : "
                        + matches.toArray().length);
    }
}

```

Output:

Matches Found : 845


# 5. Selenium Screenshot Validation Example

```java

File screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);

Files.copy(screenshot.toPath(), Paths.get("actual.png"));

double similarity = ImageComparator.compare("baseline.png", "actual.png");

Assert.assertTrue(similarity > 98);

```

Recommended Approach for Test Automation

| Scenario                      | Best Technique       |
|-------------------------------|----------------------|
| Exact image validation        | Pixel Comparison     |
| Downloaded image validation   | Pixel Comparison     |
| Selenium UI testing           | SSIM                 |
| Mobile visual testing         | ORB Feature Matching |
| Detect changed area           | AbsDiff              |
| Dynamic pages                 | ORB + SSIM           |

