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

# Security Testing Automation Scenarios Using Tcpdump
| Scenario | Objective | Tcpdump Command | Validation |
|----------|-----------|----------------|------------|
| Unauthorized Port Exposure Validation | Verify traffic reaches only approved ports | `tcpdump -i eth0 host 10.0.0.10` | Only approved ports observed |
| TLS Enforcement Validation | Ensure HTTP traffic is redirected to HTTPS | `tcpdump -i eth0 port 80 or port 443` | HTTPS packets seen; no sensitive data on HTTP |
| Firewall Rule Verification | Validate blocked ports remain inaccessible | `tcpdump -i eth0 port 22` | No packets observed from unauthorized source |
| API Security Monitoring | Capture API traffic during tests | `tcpdump -i eth0 host api.company.internal` | Requests follow expected protocol |
| DDoS Detection Validation | Validate IDS/IPS detects traffic spikes | `tcpdump -nn -i eth0` | Alert generated in SIEM |
| Database Segmentation Testing | Confirm DB access only from app tier | `tcpdump -i eth0 port 3306` | Traffic only from approved subnet |
| Kubernetes Control Plane Protection | Monitor API Server access | `tcpdump -i eth0 port 6443` | Only authorized nodes connect |
| DNS Security Validation | Detect unauthorized DNS queries | `tcpdump -i eth0 port 53` | Queries go only to approved resolvers |
| DevSecOps Deployment Validation | Verify application does not call external IPs | `tcpdump -i eth0` | No unexpected outbound traffic |
| Secrets Leakage Detection | Capture packets during login/API tests | `tcpdump -A -i eth0 port 80` | No credentials visible in clear text |

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
