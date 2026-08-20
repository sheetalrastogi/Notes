Selenium itself cannot read Wireshark packets. A practical automation approach is:

- Start Wireshark/TShark capture.
- Execute Selenium login.
- Stop packet capture.
- Parse the capture file (.pcapng).
- Search for username/password strings.
- Fail the test if credentials are found in clear text.

For automation, **TShark (Wireshark command-line component)** is better to integrate than the Wireshark GUI.

# Automation workflow

```text
Start TShark Capture
        V
Execute Selenium Login
        V
Stop Capture
        V
Analyze PCAP
        |
        +--> username found?
        +--> password found?
        +--> API key exposed?
        +--> JWT token exposed?
        +--> HTTP used instead of HTTPS?
        V
Pass / Fail Security Test
```

# Selenium code for TShark integration

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.Test;

import java.io.BufferedReader;
import java.io.InputStreamReader;

public class ClearTextCredentialExposureTest {

	@Test
	public void verifyCredentialsNotExposed() throws Exception {

		String pcapFile = "capture.pcapng";

		// ---------------------------------------------------
		// Start TShark Capture
		// ---------------------------------------------------
		Process captureProcess = new ProcessBuilder("tshark", "-i", "Wi-Fi", "-w", pcapFile).start();

		Thread.sleep(5000);

		// ---------------------------------------------------
		// Selenium Login
		// ---------------------------------------------------
		WebDriver driver = new ChromeDriver();

		String username = "testuser";
		String password = "Password123";

		try {

			driver.get("https://myapplication.com/login");

			driver.findElement(By.id("username")).sendKeys(username);

			driver.findElement(By.id("password")).sendKeys(password);

			driver.findElement(By.id("loginBtn")).click();

			Thread.sleep(5000);

		} finally {
			driver.quit();
		}

		// ---------------------------------------------------
		// Stop Capture
		// ---------------------------------------------------
		captureProcess.destroy();

		Thread.sleep(3000);

		// ---------------------------------------------------
		// Search Capture For Clear Text Data
		// ---------------------------------------------------
		boolean credentialFound = containsSensitiveData(pcapFile, username, password);

		Assert.assertFalse(credentialFound, "SECURITY FAILURE : Credentials exposed in network traffic");
	}

	private boolean containsSensitiveData(String pcap, String username, String password) throws Exception {

		Process process = new ProcessBuilder("tshark", "-r", pcap, "-Y", "http || ftp || telnet", "-V").start();

		BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));

		String line;

		while ((line = reader.readLine()) != null) {

			if (line.contains(username) || line.contains(password)) {

				System.out.println("Potential Clear Text Exposure: " + line);

				return true;
			}
		}

		return false;
	}
}
```


