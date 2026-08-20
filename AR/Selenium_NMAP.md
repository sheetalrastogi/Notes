**Nmap** is a command-line security scanner, Selenium does not directly integrate with it, but as a security tester we can automate for vulnerability assessments

Typical approach:
```text
Selenium Functional Test
        ↓
Navigate Application
        ↓
Execute Nmap Scan (Java ProcessBuilder)
        ↓
Collect Scan Output
        ↓
Validate Security Baseline
        ↓
Pass / Fail TestNG Test
```

# Nmap Scanner Utility

```java
package security;

import java.io.BufferedReader;
import java.io.InputStreamReader;

public class NmapScanner {

	public String executeScan(String command) throws Exception {
		Process process = Runtime.getRuntime().exec(command);
		BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
		StringBuilder result = new StringBuilder();
		String line;
		while ((line = reader.readLine()) != null) {
			result.append(line).append(System.lineSeparator());
		}
		process.waitFor();
		return result.toString();
	}
}
```

# Validate only allowed ports are open

```java
package tests;

import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.Test;
import security.NmapScanner;

public class SecurityBaselineTest {

	@Test
	public void validateOpenPorts() throws Exception {

		WebDriverManager.chromedriver().setup();
		WebDriver driver = new ChromeDriver();

		try {
			driver.get("https://example.com");

			NmapScanner scanner = new NmapScanner();
			String result = scanner.executeScan("nmap -sV example.com");

			System.out.println(result);
			Assert.assertFalse(result.contains("21/tcp open"), "FTP port should not be exposed");
			Assert.assertFalse(result.contains("23/tcp open"), "Telnet port should not be exposed");

		} finally {
			driver.quit();
		}
	}
}
```

## Similarly following test be automated

- SSL/TLS Security Validation
  nmap --script ssl-enum-ciphers -p 443 example.com

```java
    NmapScanner scanner = new NmapScanner();
    String output = scanner.executeScan("nmap --script ssl-enum-ciphers -p 443 example.com");
    System.out.println(output);

    Assert.assertFalse(output.contains("TLSv1.0"), "TLS 1.0 detected");
    Assert.assertFalse(output.contains("TLSv1.1"), "TLS 1.1 detected");
```

## Security Headers Validation

```java
		NmapScanner scanner = new NmapScanner();
		String output = scanner.executeScan("nmap --script http-security-headers example.com");
		System.out.println(output);

		Assert.assertTrue(output.contains("Strict-Transport-Security"));
		Assert.assertTrue(output.contains("X-Frame-Options"));
```

## Vulnerability Scan

```java
		NmapScanner scanner = new NmapScanner();
		String output = scanner.executeScan("nmap --script vuln example.com");
		System.out.println(output);

		Assert.assertFalse(output.contains("VULNERABLE"), "Vulnerability detected");
```
