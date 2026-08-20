## Comprehensive Masscan Security Testing Scenarios

Assumptions: All testing is authorised, scoped, and conducted against owned assets. Commands use placeholder CIDRs and conservative rates. Adjust rates and exclusions according to approved testing windows and monitoring capacity.

# Masscan Security Testing Scenarios

| Area | Scenario | Testing Objective | Security Risk Addressed | Masscan Command Example | Expected Outcome | Pass/Fail Criteria | Severity | Suggested Remediation |
|--------|----------|----------|----------|----------|----------|----------|----------|----------|
| Cloud | Public SSH Exposure Discovery | Identify internet-facing SSH services in cloud VPCs | Unauthorised administrative access surface | `masscan <CLOUD_VPC_CIDR> -p22 --rate 1000 --exclude-file exclude.txt -oL cloud-ssh.txt` | Only approved bastion hosts respond | Pass: Only approved hosts open. Fail: Any unauthorised SSH exposure | High | Restrict via Security Groups, NSGs, firewalls, bastion-only access |
| Cloud | Public RDP Exposure Validation | Detect exposed Windows administration endpoints | Credential attacks against RDP | `masscan <CLOUD_VPC_CIDR> -p3389 --rate 1000 -oL cloud-rdp.txt` | No direct internet-facing RDP | Pass: Approved jump hosts only | Critical | VPN-only access, JIT access, firewall restrictions |
| Cloud | Shadow Asset Discovery | Discover forgotten cloud workloads | Unmanaged attack surface | `masscan <CLOUD_VPC_CIDR> -p1-10000 --rate 1000 --exclude-file exclude.txt -oX shadow-assets.xml` | Inventory aligns with CMDB/cloud inventory | Fail if unknown hosts/services discovered | High | Asset governance, automated inventory reconciliation |
| Cloud | Load Balancer Exposure Review | Validate externally exposed services | Excessive internet-facing attack surface | `masscan <CLOUD_VPC_CIDR> -p80,443,8080,8443 --rate 1000 -oL lb-services.txt` | Only intended application endpoints exposed | Fail if administrative services discovered | High | Security group hardening, listener restrictions |
| Cloud | East-West Segmentation Validation | Verify internal segmentation controls | Lateral movement opportunities | `masscan <AUTHORIZED_CIDR> -p22,445,3389,5985,5986 --rate 500 -oL east-west.txt` | Restricted administrative connectivity | Fail if broad unrestricted access exists | High | Microsegmentation, firewall controls |
| Kubernetes | Kubernetes API Exposure | Detect exposed API server access | Cluster compromise risk | `masscan <K8S_NODE_CIDR> -p6443 --rate 500 -oL k8s-api.txt` | API reachable only from authorised management networks | Fail if broadly reachable | Critical | Private control plane, firewall rules |
| Kubernetes | etcd Exposure Assessment | Identify exposed etcd services | Exposure of cluster secrets and configuration | `masscan <K8S_NODE_CIDR> -p2379,2380 --rate 500 -oL etcd.txt` | No external exposure | Fail if etcd reachable outside control plane | Critical | Network isolation, TLS, firewall restrictions |
| Kubernetes | Kubelet Exposure Validation | Detect exposed Kubelet endpoints | Node compromise risk | `masscan <K8S_NODE_CIDR> -p10250,10255 --rate 500 -oL kubelet.txt` | Kubelet accessible only from approved cluster management networks | Fail if reachable broadly | Critical | Disable anonymous access, restrict network paths |
| Kubernetes | Ingress Controller Exposure Review | Validate intended ingress exposure | Misconfigured ingress services | `masscan <K8S_NODE_CIDR> -p80,443,8080,8443 --rate 1000 -oL ingress.txt` | Only approved ingress endpoints visible | Fail if admin interfaces exposed | High | Restrict ingress classes, firewall filtering |
| Kubernetes | NodePort Enumeration | Detect unintended NodePort services | Direct workload exposure bypassing ingress controls | `masscan <K8S_NODE_CIDR> -p30000-32767 --rate 1000 -oL nodeports.txt` | Only approved NodePorts active | Fail if undocumented NodePorts detected | High | Limit NodePort usage, enforce network policies |
| Databases | MySQL Exposure Discovery | Identify exposed MySQL listeners | Database compromise risk | `masscan <DATA_ZONE_CIDR> -p3306 --rate 1000 -oL mysql.txt` | Database accessible only from approved networks | Fail if exposed externally | Critical | Private endpoints, firewall restrictions |
| Databases | PostgreSQL Exposure Validation | Discover PostgreSQL listeners | Data theft and lateral movement | `masscan <DATA_ZONE_CIDR> -p5432 --rate 1000 -oL postgres.txt` | Access restricted to application tiers | Fail if broad accessibility exists | Critical | Network ACLs, segmentation |
| Databases | MSSQL Exposure Review | Detect SQL Server exposure | Enterprise data compromise | `masscan <DATA_ZONE_CIDR> -p1433,1434 --rate 1000 -oL mssql.txt` | Limited authorised exposure | Fail if internet-facing | Critical | Restrict access paths, private networking |
| Databases | MongoDB Exposure Review | Find exposed document databases | Sensitive data exposure | `masscan <DATA_ZONE_CIDR> -p27017 --rate 1000 -oL mongo.txt` | Only internal authorised connectivity | Fail if public access exists | Critical | Bind to private interfaces, firewall controls |
| Databases | Redis Exposure Validation | Identify exposed cache services | Data leakage and misuse risk | `masscan <DATA_ZONE_CIDR> -p6379 --rate 1000 -oL redis.txt` | No public access | Fail if reachable beyond intended scope | Critical | Private networking, ACL controls |
| APIs | API Gateway Exposure Inventory | Discover API gateway instances | Unmanaged API exposure | `masscan <APP_CIDR> -p443,8443,9443 --rate 1000 -oL api-gateways.txt` | Inventory matches approved APIs | Fail if unknown gateways found | High | API governance and registration process |
| APIs | Internal API Segmentation Check | Verify internal-only APIs remain internal | Data exposure risk | `masscan <AUTHORIZED_CIDR> -p8080,8081,9000,9443 --rate 1000 -oL internal-api.txt` | Internal APIs inaccessible externally | Fail if exposed outside approved networks | High | Network segmentation and gateway controls |
| APIs | Management Endpoint Discovery | Detect exposed API management interfaces | Administrative compromise | `masscan <APP_CIDR> -p9443,9444,8443 --rate 500 -oL api-admin.txt` | Admin endpoints restricted | Fail if broadly accessible | Critical | RBAC, firewall controls, VPN access |
| IAM-Adjacent Exposure | LDAP Exposure Discovery | Detect exposed LDAP infrastructure | Directory exposure | `masscan <CORP_CIDR> -p389,636 --rate 500 -oL ldap.txt` | Only authorised consumers can connect | Fail if broad exposure exists | High | Network restriction and segmentation |
| IAM-Adjacent Exposure | Kerberos Service Exposure | Validate authentication service boundaries | Identity infrastructure reconnaissance | `masscan <CORP_CIDR> -p88,464 --rate 500 -oL kerberos.txt` | Access limited to required zones | Fail if exposed publicly | High | Restrict network routes and firewall rules |
| IAM-Adjacent Exposure | AD Administrative Port Exposure | Find exposed Active Directory management services | Domain compromise risk | `masscan <CORP_CIDR> -p135,445,3268,3269 --rate 500 -oL ad-admin.txt` | Limited internal-only availability | Fail if exposed beyond approved networks | Critical | Segmentation and privileged access controls |
| IAM-Adjacent Exposure | WinRM Exposure Validation | Discover remote management interfaces | Unauthorised administration pathways | `masscan <CORP_CIDR> -p5985,5986 --rate 500 -oL winrm.txt` | Reachable only from management networks | Fail if broadly accessible | High | Restrict access via firewall and jump hosts |
| Logging & Monitoring | SIEM Detection Validation | Confirm Masscan-style activity triggers alerts | Blind spots in monitoring | `masscan <AUTHORIZED_CIDR> -p22,80,443,445 --rate 500 -oL siem-test.txt` | Security alert generated within SLA | Pass only if alert created and investigated | Medium | Tune IDS/SIEM and detection engineering |
| Logging & Monitoring | Network IDS Validation | Verify NIDS/NDR detects port scanning | Undetected reconnaissance activity | `masscan <AUTHORIZED_CIDR> -p1-1024 --rate 500 -oL ids-test.txt` | Detection generated with source attribution | Fail if no visibility exists | Medium | Improve signatures and telemetry collection |
| Logging & Monitoring | Critical Service Monitoring Review | Confirm monitoring covers sensitive services | Missing operational visibility | `masscan <AUTHORIZED_CIDR> -p22,3389,3306,5432,6443 --rate 500 -oL monitoring.txt` | All active services visible in monitoring inventory | Fail if services absent from monitoring | Medium | Expand monitoring coverage and integration |
| DevSecOps | Pre-Production Exposure Gate | Validate deployment does not expose unauthorised services | Configuration drift before release | `masscan <PREPROD_CIDR> -p1-10000 --rate 1000 -oL preprod.txt` | Exposure matches approved architecture | Fail if unexpected ports appear | High | CI/CD security gates and automated network validation |
| DevSecOps | IaC Drift Validation | Detect infrastructure exposure drift | Security control regression | `masscan <ENV_CIDR> -p22,80,443,445,3306,5432 --rate 1000 -oX drift.xml` | Results align with IaC-defined state | Fail on unapproved changes | High | Continuous compliance and IaC enforcement |
| DevSecOps | Container Platform Exposure Check | Verify orchestrator-related ports remain restricted | Container platform compromise | `masscan <CONTAINER_CIDR> -p2375,2376,6443 --rate 500 -oL containers.txt` | No unauthorised management service exposure | Fail if management services are reachable | Critical | Secure Docker/K8s APIs and enforce network controls |
| DevSecOps | Release Readiness Security Validation | Validate approved service baseline before go-live | Production exposure issues | `masscan <RELEASE_ENV_CIDR> -p1-65535 --rate 1000 -oL release-validation.txt` | Only approved services exposed | Fail if deviations exist | High | Baseline enforcement and deployment approval process |
| Red Team Validation | External Reconnaissance Simulation | Emulate attacker service discovery | Early-stage adversary reconnaissance | `masscan <EXTERNAL_RANGE> -p22,80,443,445,3389 --rate 1000 -oL external-recon.txt` | Exposure aligns with approved attack surface | Fail on unexpected services | High | Reduce public attack surface |
| Red Team Validation | Management Service Exposure Assessment | Identify exposed administrative services | Initial access pathways | `masscan <AUTHORIZED_CIDR> -p22,3389,5900,5985,5986 --rate 500 -oL mgmt-services.txt` | Only authorised management hosts visible | Fail if unmanaged exposure exists | Critical | Bastion-only administration model |
| Red Team Validation | Forgotten Environment Discovery | Detect legacy or abandoned environments | Unmanaged compromise risk | `masscan <AUTHORIZED_CIDR> -p1-10000 --rate 1000 -oX forgotten-assets.xml` | No undocumented systems discovered | Fail if unknown assets found | High | Environment lifecycle management |
| Red Team Validation | Multi-Tenant Boundary Validation | Verify tenant network isolation | Cross-tenant reconnaissance risk | `masscan <TENANT_TEST_CIDR> -p22,80,443,445 --rate 500 -oL tenancy.txt` | No visibility into other tenant services | Fail if tenant boundary breached | Critical | Segmentation, VPC isolation, network policies |
| Red Team Validation | Critical Infrastructure Exposure Review | Identify exposed infrastructure services | High-value target discovery | `masscan <AUTHORIZED_CIDR> -p53,123,161,389,636,6443 --rate 500 -oL infra-services.txt` | Exposure limited to approved networks | Fail if externally discoverable | Critical | Restrict infrastructure service access |



**Recommended Evidence Artifacts**
For audit, QE security validation following needs to be captured:
- Masscan output (-oL, -oX, -oJ)
- Approved port baseline documentation
- Firewall/Security Group/NSG rule exports
- Kubernetes NetworkPolicy manifests
- Cloud route table and private endpoint evidence
- SIEM alert screenshots and incident IDs
- Change tickets and approved exceptions
- IaC configuration diffs
- Service ownership mapping
- Risk acceptance records (where applicable)

**High-Risk Ports Worth Including in Baseline Reviews**
| Category                | Ports                                      |
|-------------------------|--------------------------------------------|
| Administration          | 22, 3389, 5900, 5985, 5986                 |
| Windows Infrastructure  | 135, 139, 445                              |
| Databases               | 1433, 1521, 27017, 3306, 5432, 6379        |
| Kubernetes              | 2379, 2380, 6443, 10250, 10255             |
| Containers              | 2375, 2376                                 |
| Directory Services      | 88, 389, 636, 3268, 3269                   |
| Web/API                 | 80, 443, 8080, 8443, 9443                  |
| Messaging / Search      | 5672, 9092, 9200, 9300                     |
