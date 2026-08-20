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

