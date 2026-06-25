# Cybersecurity Lab - Docker-Based Vulnerable Applications

## Project Overview

This repository documents a comprehensive cybersecurity lab setup using Docker containers for penetration testing, vulnerability assessment, and security research. The lab includes multiple intenti...

## 📋 Lab Architecture

### Docker Compose Services (Multi-Container Environment)

#### **Screenshot 1: Vulnerable Web Applications**

##### WordPress Container
- **Image**: `wordpress:latest`
- **Container Name**: `wp_lab`
- **Port Mapping**: `8083:80`
- **Database**: MySQL (host: mysql:3306)
- **Credentials**: 
  - User: `labuser`
  - Password: `labpass`
  - Database: `labdb`
- **Vulnerabilities**: SQL injection, XSS, plugin exploits, authentication bypass
- **Purpose**: Test common CMS vulnerabilities

##### Log4Shell Vulnerable Application
- **Image**: `ghcr.io/christophetd/log4shell-vulnerable-app:latest`
- **Container Name**: `log4shell_app`
- **Port Mapping**: `8082:8080`
- **Vulnerability**: CVE-2021-44228 (Apache Log4j Remote Code Execution)
- **CVSS Score**: Critical (9.8+)
- **Purpose**: Demonstrate Java application exploitation
- **Note**: Real-world attack scenario on vulnerable logging framework

##### OWASP Juice Shop
- **Image**: `bkimminich/juice-shop`
- **Container Name**: `juice_shop`
- **Port Mapping**: `8085:3000`
- **Vulnerabilities**: OWASP Top 10 (Injection, Broken Auth, Sensitive Data Exposure, etc.)
- **Purpose**: Comprehensive web application security testing
- **Tools**: Compatible with BurpSuite, OWASP ZAP, Metasploit

#### **Screenshot 2: Infrastructure Services**

##### Nginx Web Server
- **Image**: `nginx:latest`
- **Container Name**: `web_nginx`
- **Port Mapping**: `8080:80`
- **Volume**: `/nginx_html:/usr/share/nginx/html`
- **Purpose**: Reverse proxy and static content server
- **Note**: Basic reference web server

##### MySQL Database Server
- **Image**: `mysql:5.7`
- **Container Name**: `db_mysql`
- **Port Mapping**: `3306:3306`
- **Root Password**: `rootpassword`
- **User Credentials**:
  - Username: `labuser`
  - Password: `labpass`
  - Database: `labdb`
- **Purpose**: Central database backend for WordPress and DVWA
- **Port**: 3306 (TCP)

##### DVWA (Damn Vulnerable Web Application)
- **Image**: `vulnerabilities/web-dvwa`
- **Container Name**: `web_dvwa`
- **Port Mapping**: `8081:80`
- **Database Dependency**: MySQL
- **Vulnerabilities**: OWASP Top 10 classic vulnerabilities
- **Purpose**: Beginner-friendly vulnerable web app for security learning
- **Difficulty Levels**: Low, Medium, High

#### Network Configuration
- **Network**: `labnet` (bridge driver)
- **Restart Policy**: `unless-stopped`
- **Isolation**: Containers connected via internal Docker network

---

## 🔍 Network Reconnaissance Results

### Screenshots 3-6: Nmap Scanning and Service Discovery

#### **Initial Network Scan (Screenshot 3)**
```
Target: 192.168.255.128
Scan Type: SYN Stealth Scan
Scan Duration: 56.85 seconds
Initial Response: 4 packets received, 0 dropped
Latency: ~3 ms (125.37 hosts/sec)

Initial Open Ports Detected:
- Port 22/tcp: SSH
- Port 80/tcp: HTTP
- Port 3306/tcp: MySQL
- Port 445/tcp: SMB
```

#### **Service Detection Scan (Screenshot 4)**
```
Command: nmap -sV -p 8080,8081,8082,8083,8085,3306,2222,445,139,8084 198.168.255.128

DISCOVERED SERVICES:
┌─────────────────────────────────────────────────────────┐
│ PORT     │ STATE    │ SERVICE              │ VERSION    │
├─────────────────────────────────────────────────────────┤
│ 3306/tcp │ filtered │ MySQL                │            │
│ 8080/tcp │ filtered │ http-proxy           │            │
│ 8081/tcp │ filtered │ blackice-icecap      │            │
│ 8082/tcp │ filtered │ blackice-alerts      │            │
│ 8083/tcp │ filtered │ us-srv               │            │
│ 8085/tcp │ filtered │ unknown              │            │
└─────────────────────────────────────────────────────────┘

Scan Summary:
- Service Detection: Performed
- Filtered Ports: 6 (firewall protection detected)
- Host Status: Up (0.00040s latency)
- Total Scan Time: 2.04 seconds
```

#### **Comprehensive Port Scan (Screenshot 5)**
```
Command: sudo nmap -sV -p- 8080,8081,8082,8083,8085,3306,2222,445,139,8084 198.168.255.128

FULL PORT ENUMERATION:
┌──────────────┬──────────────┬──────────────────────┬───────────[...]
│ PORT         │ STATE        │ SERVICE              │ VERSION     │
├──────────────┼──────────────┼──────────────────────┼───────────[...]
│ 139/tcp      │ filtered     │ netbios-ssn          │             │
│ 445/tcp      │ filtered     │ microsoft-ds         │             │
│ 2222/tcp     │ filtered     │ EtherNetIP-1         │             │
│ 3306/tcp     │ filtered     │ MySQL                │             │
│ 8080/tcp     │ filtered     │ http-proxy           │             │
│ 8081/tcp     │ filtered     │ blackice-icecap      │             │
│ 8082/tcp     │ filtered     │ blackice-alerts      │             │
│ 8083/tcp     │ filtered     │ us-srv               │             │
│ 8084/tcp     │ filtered     │ websnp               │             │
│ 8085/tcp     │ filtered     │ unknown              │             │
└──────────────┴──────────────┴──────────────────────┴───────────[...]

Results:
- Total Hosts Scanned: 1 IP address
- Closed Ports: 994 (reset)
- Host Latency: 0.00091s
- Scan Time: 1.96 seconds
- Note: 994 closed TCP ports indicate active firewall filtering
```

#### **Final Detailed Scan (Screenshot 6)**
```
Command: sudo nmap 192.168.255.128

FINAL ENUMERATION RESULTS:
┌──────────────┬────────┬──────────────────────┬──────────────┐
│ PORT         │ STATE  │ SERVICE              │ VERSION      │
├──────────────┼────────┼──────────────────────┼──────────────┤
│ 139/tcp      │ open   │ netbios-ssn          │              │
│ 445/tcp      │ open   │ microsoft-ds         │              │
│ 2222/tcp     │ open   │ EtherNetIP-1         │              │
│ 3306/tcp     │ open   │ MySQL                │              │
│ 8080/tcp     │ open   │ http-proxy           │              │
│ 8081/tcp     │ open   │ blackice-icecap      │              │
│ 8082/tcp     │ open   │ blackice-alerts      │              │
│ 8083/tcp     │ open   │ us-srv               │              │
│ 8084/tcp     │ open   │ websnp               │              │
│ 8085/tcp     │ open   │ unknown              │              │
└──────────────┴────────┴──────────────────────┴──────────────┘

MAC Address: 00:0C:29:38:8D:53 (VMware, Inc.)
Host Discovery:
- Host is up (0.00030s latency)
- Closed ports reset by host
- VMware virtual machine detected
- Scan completed in 0.76 seconds
```

---

## 🎯 Service Port Mapping Reference

| Port | Service | Container | Purpose |
|------|---------|-----------|---------|
| 8080 | http-proxy | Nginx | Web reverse proxy |
| 8081 | blackice-icecap | DVWA | Vulnerable web app |
| 8082 | blackice-alerts | Log4Shell | Java RCE vulnerability |
| 8083 | us-srv | WordPress | CMS testing |
| 8085 | unknown | Juice Shop | OWASP vulnerabilities |
| 3306 | MySQL | Database | Backend data storage |
| 2222 | EtherNetIP-1 | Metasploitable2 | SSH access |
| 445 | microsoft-ds | Metasploitable2 | SMB/file sharing |
| 139 | netbios-ssn | Metasploitable2 | NetBIOS sessions |
| 8084 | websnp | Metasploitable2 | Web server |

---

## 🚀 Quick Start Guide

### Prerequisites
- Docker & Docker Compose installed
- Nmap installed for reconnaissance
- Pentesting tools (BurpSuite, Metasploit, OWASP ZAP recommended)
- Minimum 4GB RAM available
- Network access on 192.168.255.x subnet

### Deployment Steps

#### 1. **Clone the Repository**
```bash
git clone https://github.com/xTMrLiuTx/cybersecurity-lab.git
cd cybersecurity-lab
```

#### 2. **Start Docker Compose Services**
```bash
docker-compose up -d
```

#### 3. **Deploy Metasploitable2**
```bash
docker rm -f metasploitable2
docker run -d --name metasploitable2 \
  -p 2222:22 \
  -p 8084:80 \
  -p 2121:21 \
  -p 2323:23 \
  -p 2525:25 \
  -p 3307:3306 \
  -p 139:139 \
  -p 445:445 \
  tleemcjr/metasploitable2 tail -f /dev/null
```

#### 4. **Start Metasploitable2 Services**
```bash
docker exec -it metasploitable2 bash
```

Inside the container:
```bash
service apache2 start
service ssh start
service mysql start
service proftpd start
service vsftpd start
service postgresql start
service tomcat5.5 start
service samba start
```

#### 5. **Verify All Services**
```bash
docker ps -a
nmap -sV localhost
```

---

## 📊 Network Reconnaissance Workflow

### Phase 1: Host Discovery
```bash
nmap -sn 192.168.255.0/24  # Identify active hosts
```

### Phase 2: Port Enumeration
```bash
nmap 192.168.255.128       # Standard scan
nmap -sV 192.168.255.128   # Service version detection
nmap -p- 192.168.255.128   # All 65535 ports
```

### Phase 3: Service Enumeration
From the scans above, services detected:
- **Port 139**: NetBIOS Session Service (Windows file sharing)
- **Port 445**: Microsoft-DS (SMB protocol - file sharing & RPC)
- **Port 2222**: SSH alternative port (Metasploitable2)
- **Port 3306**: MySQL database server
- **Port 8080-8085**: Multiple web services
- **Port 8084**: Metasploitable2 web server

### Phase 4: Vulnerability Assessment
```bash
# Using Nmap NSE scripts
nmap -sV --script vuln 192.168.255.128

# Using Metasploit
msfconsole
> db_nmap -sV 192.168.255.128

# Using OWASP ZAP
zaproxy -cmd -quickurl http://192.168.255.128:8083
```

---

## 🔐 Metasploitable2 Services Reference

| Service | Port | Protocol | Vulnerability | Exploitation Tool |
|---------|------|----------|----------------|-------------------|
| Apache2 | 80/8084 | HTTP | Outdated web server | Metasploit, manual |
| SSH | 22/2222 | TCP | Default credentials | SSH client, Metasploit |
| MySQL | 3306/3307 | TCP | No password | MySQL client, Metasploit |
| ProFTPD | 21/2121 | FTP | Version exploits | Metasploit |
| Telnet | 23/2323 | TCP | Unencrypted login | Telnet client, Metasploit |
| SMTP | 25/2525 | TCP | No auth required | Mail clients |
| PostgreSQL | 5432 | TCP | Default creds | psql client |
| Samba | 139/445 | SMB | Version vulnerabilities | Metasploit, enum4linux |
| Tomcat | 8080/8180 | HTTP | Manager bypass | Metasploit |

---

## 🎓 Penetration Testing Exercises

### Exercise 1: Web Application Testing
1. Access WordPress at `http://localhost:8083`
2. Use BurpSuite to intercept requests
3. Test for SQL injection in login form
4. Identify and exploit XSS vulnerabilities
5. Document findings

### Exercise 2: Log4Shell Exploitation
1. Target: `http://localhost:8082`
2. Craft JNDI injection payload
3. Set up reverse shell listener
4. Achieve RCE (Remote Code Execution)
5. Document exploitation steps

### Exercise 3: DVWA Security Testing
1. Access DVWA at `http://localhost:8081`
2. Set difficulty level to "Low"
3. Test each vulnerability module:
   - SQL Injection
   - Broken Authentication
   - File Inclusion
   - Command Injection
4. Progress to "Medium" and "High" levels

### Exercise 4: Metasploitable2 Exploitation
1. Run Nmap enumeration
2. Identify vulnerable services
3. Use Metasploit modules:
   ```bash
   use exploit/unix/ftp/vsftpd_234_backdoor
   use exploit/linux/ssh/openssh_pubkey_overflow
   ```
4. Achieve reverse shell access
5. Post-exploitation (privilege escalation, data exfiltration)

### Exercise 5: Network Traffic Analysis
1. Capture traffic with Wireshark
2. Analyze plaintext protocols (Telnet, FTP)
3. Identify credentials in captured packets
4. Document insecure transmission risks

---

## 📁 Repository Structure

```
cybersecurity-lab/
├── README.md                          # Main documentation
├── docker-compose.yml                 # Multi-container configuration
├── metasploitable2-setup.sh          # Metasploitable2 deployment script
├── NMAP_SCAN_RESULTS.md              # Network reconnaissance results
├── PENETRATION_TESTING_GUIDE.md      # Step-by-step testing guide
├── VULNERABILITY_ANALYSIS.md         # Detailed vulnerability breakdown
├── SCREENSHOTS/
│   ├── 1-docker-compose-part1.png    # WordPress, Log4Shell, Juice Shop
│   ├── 2-docker-compose-part2.png    # Nginx, MySQL, DVWA
│   ├── 3-nmap-initial-scan.png       # First reconnaissance
│   ├── 4-nmap-service-detection.png  # Service versions
│   ├── 5-nmap-comprehensive.png      # Full port enumeration
│   └── 6-nmap-final-results.png      # Detailed port status
└── TOOLS/
    ├── build-lab.sh                   # Automated lab setup
    ├── cleanup-lab.sh                 # Lab teardown
    └── health-check.sh                # Service verification
```

---

## 🛠️ Tools & Technologies

### Required Tools
- **Docker**: Container platform
- **Docker Compose**: Multi-container orchestration
- **Nmap**: Network mapping and reconnaissance
- **Metasploit Framework**: Exploitation framework

### Recommended Tools
- **BurpSuite Community**: Web app testing
- **OWASP ZAP**: Vulnerability scanning
- **Wireshark**: Network packet analysis
- **Ghidra**: Binary analysis
- **SQLmap**: SQL injection automation
- **enum4linux**: Windows enumeration

---

## 📈 Learning Objectives

After completing this lab, you will understand:

✅ Docker containerization and orchestration
✅ Network reconnaissance with Nmap
✅ Web application vulnerability assessment
✅ SQL injection and XSS exploitation
✅ Java RCE exploitation (Log4Shell)
✅ SMB and NetBIOS enumeration
✅ SSH and FTP protocol vulnerabilities
✅ Database security weaknesses
✅ Metasploit framework usage
✅ Penetration testing methodology
✅ Report writing and documentation

---

## ⚖️ Legal & Ethical Notice

### ⚠️ Important Disclaimer

This lab is strictly for:
- ✅ Educational purposes
- ✅ Authorized security research
- ✅ Personal skill development
- ✅ Controlled training environments
- ✅ Ethical hacking practice

### ❌ Prohibited Uses
- Testing systems without explicit authorization
- Malicious hacking attempts
- Unauthorized access attempts
- Commercial exploitation
- Illegal activities

**By using this lab, you agree to conduct all testing ethically and legally.**

---

## 📞 Support & Resources

### Official Documentation
- [Docker Docs](https://docs.docker.com/)
- [Nmap Manual](https://nmap.org/book/)
- [Metasploit Documentation](https://docs.metasploit.com/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)

### Vulnerable Applications
- [DVWA Project](http://dvwa.co.uk/)
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
- [Log4Shell Information](https://logging.apache.org/log4j/2.x/security.html)
- [Metasploitable2](https://docs.rapid7.com/metasploit/metasploitable-2/)

### Learning Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [SANS Cyber Academy](https://www.sans.org/)
- [OffSec PWK](https://www.offsec.com/pwk/)

---

## ✅ Lab Setup Checklist

- [ ] Docker and Docker Compose installed
- [ ] Repository cloned locally
- [ ] docker-compose.yml reviewed
- [ ] All containers deployed successfully
- [ ] Port mappings verified (`docker ps`)
- [ ] Web applications accessible
- [ ] Metasploitable2 running
- [ ] Services started inside Metasploitable2
- [ ] Nmap installed and functioning
- [ ] Initial network scan completed
- [ ] Service detection successful
- [ ] All 10 ports responding
- [ ] VMware/Virtual machine confirmed
- [ ] Documentation reviewed
- [ ] Ready for exploitation exercises

---

## 📝 Lab Session Log

**Lab Created**: May 23, 2026
**Configuration**: Docker Compose + Metasploitable2
**Total Services**: 9 containers
**Total Exposed Ports**: 10+ ports
**Host IP**: 192.168.255.128
**Status**: ✅ Fully Operational

---

## 👤 Lab Owner

**Username**: xTMrLiuTx
**Repository**: https://github.com/xTMrLiuTx/cybersecurity-lab
**Last Updated**: May 23, 2026

---

## 📄 License

Educational Use Only - For authorized security testing and learning purposes.

**Happy Hacking! 🔐**
