# Test automation flow

```text
@BeforeEach
  Start Tcpdump
@Test
  Execute Web Selenium Workflow
  Execute API Calls
  Upload Sensitive File

@AfterEach
- Stop Tcpdump
- Analyze Pcap
- Generate JSON Report

```

Selenium 4 + Java Example

Use Case
Validate that a login page sends traffic only via HTTPS.

```java
import org.junit.jupiter.api.*;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

import java.io.IOException;

public class TcpdumpSecurityTest {

	private WebDriver driver;
	private Process tcpdumpProcess;

	@BeforeEach
	public void startCapture() throws IOException {
		ProcessBuilder pb = new ProcessBuilder("sudo", "tcpdump", "-i", "eth0", "-w", "/tmp/login_capture.pcap");
		tcpdumpProcess = pb.start();
		driver = new ChromeDriver();
	}

	@Test
	public void testSecureLogin() {
		driver.get("https://testapp.company.com/login");
		driver.findElement(org.openqa.selenium.By.id("username")).sendKeys("testuser");
		driver.findElement(org.openqa.selenium.By.id("password")).sendKeys("Password123");
		driver.findElement(org.openqa.selenium.By.id("loginButton")).click();
	}

	@AfterEach
	public void stopCapture() {
		tcpdumpProcess.destroy();
		if (driver != null) {
			driver.quit();
		}
	}
}
```


# 2. PCAP Validation Example

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;

public class PcapValidator {

	public static boolean verifyHttpsOnly() throws Exception {

		ProcessBuilder pb = new ProcessBuilder("tcpdump", "-nn", "-r", "/tmp/login_capture.pcap");
		Process process = pb.start();
		BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));

		String line;
		while ((line = reader.readLine()) != null) {
			if (line.contains(".80 >")) {
				return false;
			}
		}
		return true;
	}
}

```

Usage:

```java
Assertions.assertTrue(PcapValidator.verifyHttpsOnly(), "Unencrypted HTTP traffic detected" );

```

# Few more QE Security Scenarios

- 1. Session Cookie Security Validation

tcpdump -i eth0 port 443 -w cookies.pcap

**Validates**:
- Secure flag enabled
- HttpOnly enabled
- Cookies not visible over HTTP

- 2. Kubernetes Service Exposure Validation

tcpdump -i eth0 port 6443

**Selenium**:
	Access application deployed in cluster.
**Validate**:
	No direct browser communication to K8s API.
	Traffic only through ingress/load balancer.

- 3. Database Leakage Detection

tcpdump -i eth0 port 3306

**Selenium**:
	Trigger UI search/export operations.
**Validate**:
	Browser never communicates directly with database.
	Only application tier accesses DB.

- 4. Third-Party Data Exfiltration Detection

tcpdump -nn -i eth0

**Selenium**:
	Execute full Selenium regression suite.
**Validate**:
	No unexpected outbound destinations.
	Approved SaaS endpoints only.

- 5. Security Header Validation + Packet Capture

String csp = ((HasAuthentication) driver).getPageSource();

Network validation:	
	tcpdump -i eth0 host testapp.company.com

**Validate**:
	CSP present
	HSTS enabled
	No mixed-content requests




