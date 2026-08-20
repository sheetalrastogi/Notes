**CIDR** stands for Classless Inter-Domain Routing.
In your Masscan examples, CIDR specifies the network range (IP address block) that you are authorised to scan.

**CIDR Format**
IP_ADDRESS/PREFIX_LENGTH

**Example**
| CIDR Block       | IP Range Size | Example Usage                |
|------------------|--------------|------------------------------|
| `10.0.0.0/24`    | 256 IPs      | Small subnet                 |
| `10.0.0.0/16`    | 65,536 IPs   | Large network segment        |
| `172.16.0.0/12`  | ~1 Million IPs | Enterprise private network |
| `192.168.1.0/24` | 256 IPs      | Common LAN                   |
| `10.10.10.5/32`  | Single IP    | Scan one specific host       |

**CIDR Cheatsheet**

| CIDR | Number of IPs | Typical Usage |
|------|--------------|---------------|
| `/32` | 1 | Single host |
| `/30` | 4 | Point-to-point links |
| `/29` | 8 | Small network |
| `/28` | 16 | Small subnet |
| `/27` | 32 | Department subnet |
| `/26` | 64 | Medium subnet |
| `/25` | 128 | Large subnet |
| `/24` | 256 | Standard LAN |
| `/23` | 512 | Large LAN |
| `/22` | 1,024 | Campus network |
| `/16` | 65,536 | Large enterprise segment |
| `/12` | 1,048,576 | Enterprise private network |
| `/8` | 16,777,216 | Very large enterprise network |

**Example Masscan Targets**
| CIDR Block       | Description                  |
|------------------|------------------------------|
| `10.10.10.5/32`  | Single host                  |
| `192.168.1.0/24` | Small LAN                    |
| `10.0.0.0/16`    | Large network segment        |
| `172.16.0.0/12`  | Enterprise private network   |
| `10.0.0.0/8`     | Large corporate network      |
