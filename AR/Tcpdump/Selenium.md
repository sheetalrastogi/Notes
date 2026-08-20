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
