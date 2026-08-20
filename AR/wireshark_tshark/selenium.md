Selenium itself cannot read Wireshark packets. A practical automation approach is:

- Start Wireshark/TShark capture.
- Execute Selenium login.
- Stop packet capture.
- Parse the capture file (.pcapng).
- Search for username/password strings.
- Fail the test if credentials are found in clear text.

For automation, **TShark (Wireshark command-line component)** is better to integrate than the Wireshark GUI.

# Wireshark security testing scenarios
---

1. Clear Text Credential Exposure
2. Sensitive Data Leakage
3. TLS/SSL Validation
4. Weak Protocol Identification
5. Session Hijacking Testing
6. API Security Validation
7. DNS Security Assessment
8. ARP Spoofing Detection
9. Port Scanning Detection
10. Unauthorized Internal Traffic
11. Malware C2 Communication
12. Data Exfiltration Detection
13. Cloud Security Validation
14. Certificate Validation
15. DDoS Analysis
16. Kubernetes Security Validation
17. Database Traffic Security
18. Authentication & Login Security Testing


# what to look for for TShark captures
| Test Area | Can TShark Test? | Notes |
|------------|------------|------------|
| HTTP Basic Authentication | Yes | Authorization header visible |
| FTP credentials | Yes | USER/PASS commands visible |
| Telnet sessions | Yes | Plaintext traffic |
| API keys in headers | Yes | Visible in HTTP headers |
| Session tokens in URLs | Yes | Visible in requests |
| TLS version | Yes | TLS handshake analysis |
| Cipher suites | Yes | TLS negotiation visible |
| Certificate exchange | Yes | TLS certificates visible |
| Weak encryption | Yes | Detected from TLS handshake |
| Telnet | Yes | Protocol inspection |
| FTP | Yes | Protocol inspection |
| POP3 | Yes | Cleartext credential detection |
| IMAP | Yes | Protocol inspection |
| SNMP v1/v2 | Yes | Community strings visible |
| HTTP | Yes | HTTP packet inspection |
| Session cookies | Yes | Cookie headers visible |
| JWT tokens | Yes | JWT transmission visible |
| Authorization headers | Yes | HTTP headers visible |
| Bearer tokens | Yes | HTTP headers visible |
| API keys | Yes | Request inspection |
| JWT transmission | Yes | Token transport visible |
| Sensitive payloads | Yes | Payload inspection |
| DNS tunneling | Yes | DNS traffic analysis |
| Malicious domains | Yes | DNS queries visible |
| Clear text DNS | Yes | DNS packet analysis |
| Duplicate ARP responses | Yes | ARP analysis |
| MAC address changes | Yes | Layer-2 analysis |
| Multiple connection attempts | Yes | TCP session analysis |
| Sequential port access | Yes | Port scanning behavior |
| Application-to-database traffic | Yes | Network flow analysis |
| User access to server networks | Yes | Traffic flow analysis |
| Unexpected east-west traffic | Yes | Traffic monitoring |
| Beaconing patterns | Yes | Traffic timing analysis |
| Unknown external IPs | Yes | Connection analysis |
| Repeated DNS requests | Yes | DNS inspection |
| Large outbound transfers | Yes | Traffic volume analysis |
| Suspicious uploads | Yes | HTTP/FTP payload analysis |
| Metadata requests | Yes | Cloud metadata traffic detection |
| Expired certificates | Yes | Certificate inspection |
| Self-signed certificates | Yes | Certificate inspection |
| Incorrect CN/SAN entries | Yes | Certificate validation |
| SYN floods | Yes | TCP analysis |
| UDP floods | Yes | Traffic analysis |
| ICMP floods | Yes | Packet analysis |
| Exposed kube-api traffic | Yes | API traffic detection |
| Unencrypted service communication | Yes | Protocol analysis |
| Unauthorized pod communication | Partially | Traffic visible, authorization not |
| SQL queries transmitted in plain text | Yes | Database protocol inspection |
| Weak database authentication | Partially | Visible if transmitted over network |
| Unencrypted connections | Yes | Protocol inspection |
| Password transmission | Yes | Cleartext detection |
| MFA verification traffic | Partially | Traffic visible |
| Token issuance flow | Yes | API traffic inspection |
| Cloud API calls | Yes | API request inspection |

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

**Additional Security Test Assertions**

- Assert.assertFalse(packet.contains("Authorization: Basic"));
- Assert.assertFalse(packet.contains("password="));
- Assert.assertFalse(packet.contains("Bearer "));
- Assert.assertFalse(packet.contains("apiKey"));
- Assert.assertFalse(packet.contains("secret"));
- Assert.assertFalse(packet.contains("Set-Cookie"));

**Useful TShark Commands**

- Capture Traffic		tshark -i Wi-Fi -w capture.pcapng
- Read HTTP Requests	tshark -r capture.pcapng -Y "http"
- Find FTP Credentials	tshark -r capture.pcapng -Y ftp
- Find Telnet Traffic	tshark -r capture.pcapng -Y telnet
- Extract HTTP POST Data	tshark -r capture.pcapng -Y "http.request.method==POST" -V


