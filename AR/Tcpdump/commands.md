| Tool      | Description |
|-----------|-------------|
| `tcpdump` | A command-line packet analyzer used to capture, inspect, and troubleshoot network traffic in real time. Commonly used for network monitoring, security analysis, incident investigation, and protocol debugging. |

# Common Tcpdump Security Testing Commands

| Scenario | Command |
|----------|---------|
| Capture all packets | `tcpdump -i any` |
| Capture on specific interface | `tcpdump -i eth0` |
| Capture HTTP traffic | `tcpdump -i eth0 port 80` |
| Capture HTTPS traffic | `tcpdump -i eth0 port 443` |
| Capture SSH traffic | `tcpdump -i eth0 port 22` |
| Capture traffic from specific host | `tcpdump host 10.10.10.5` |
| Capture source host traffic | `tcpdump src host 10.10.10.5` |
| Capture destination host traffic | `tcpdump dst host 10.10.10.5` |
| Capture network subnet traffic | `tcpdump net 192.168.1.0/24` |
| Capture and save to file | `tcpdump -i eth0 -w capture.pcap` |
| Read packets from file | `tcpdump -r capture.pcap` |
| Capture first 100 packets | `tcpdump -c 100` |
| Capture DNS traffic | `tcpdump port 53` |
| Capture ICMP (Ping) traffic | `tcpdump icmp` |
| Capture TCP SYN packets | `tcpdump 'tcp[tcpflags] & tcp-syn != 0'` |


# Security Testing Usage
- Network Traffic Analysis
- Incident Investigation
- Intrusion Detection Support
- Malware Communication Analysis
- API Traffic Inspection
- DNS Monitoring
- Firewall Validation
- Network Segmentation Testing
- Data Exfiltration Detection
- Penetration Testing Verification
