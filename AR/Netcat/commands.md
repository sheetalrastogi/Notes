**Netcat (nc)** is often called the **Swiss Army knife of networking** because it can create TCP/UDP connections, listen on ports, transfer files, test services, and troubleshoot network issues.

# 1. Check if a Port is Open
nc -vz 192.168.1.10 22

# 2. Scan Multiple Ports
nc -zv 192.168.1.10 20-100

# 3. Connect to a Web Server
Step 1: Manually send an HTTP request.
  nc example.com 80
Step 2: Type following in Netcat
  GET / HTTP/1.1
  Host: example.com

# 4. Create a TCP Listener
Listen for incoming connections on port 9000.
 nc -lvp 9000

# 5. Send a Message Between Hosts
Receiver:  nc -lvp 9000
Sender:    echo "Hello from Host A" | nc 192.168.1.20 9000

# 6. Transfer a File
Receiver:  nc -lvp 9000 > received_file.txt
Sender:    nc 192.168.1.20 9000 < file.txt

# 7. Test SMTP Connectivity
nc -vz mail.example.com 25

# 8. Test HTTPS Port Availability
nc -vz example.com 443

# 9. UDP Port Testing
nc -uvz 192.168.1.10 53

# 10. Banner Grabbing
nc 192.168.1.10 21
  example output:  220 FTP Server Ready

# Security Testing Examples (Authorised Environments)
As a Test Architect, you may use Netcat for validation:

ScenarioCommandValidate exposed SSH servicenc -vz 10.0.0.10 22Verify Kubernetes API accessibilitync -vz 10.0.0.50 6443Check exposed database portnc -vz 10.0.0.30 3306Verify Redis exposurenc -vz 10.0.0.40 6379Test message broker portnc -vz 10.0.0.60 5672

