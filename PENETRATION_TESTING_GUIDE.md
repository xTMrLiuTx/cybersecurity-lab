# Penetration Testing Guide: DVWA Juice Shop Lab

## Overview

This guide documents a complete penetration testing scenario using the DVWA Juice Shop vulnerable application. The lab demonstrates network reconnaissance, service enumeration, and vulnerability analysis using industry-standard security tools.

---

## Lab Environment Setup

### Target System Information
- **IP Address**: 192.168.255.128
- **Operating System**: Linux 4.15-5.19 (Debian-based)
- **Target Application**: OWASP Juice Shop
- **Architecture**: Docker-based vulnerable application container

### Tools Used
- **Nmap**: Network mapping and service enumeration
- **SMB Discovery**: Windows/Samba service detection
- **XSLT Processing**: XML transformation testing
- **Firefox Browser**: Web application analysis

---

## Phase 1: Network Reconnaissance

### Initial SMB Discovery (Screenshot 1 - Terminal Output)

**Command Executed:**
```bash
smb-os-discovery and smb-security-mode
```

**Results:**
- **OS Detected**: Unix (Samba 3.0.20-Debian)
- **Computer Name**: f959e45232c8
- **NetBIOS Configuration**: Active
- **Domain**: Not configured (FQDN: f959e45232c8)
- **System Time**: 2026-05-23T04:25:50-04:00
- **Account Status**: Guest user with user-level authentication
- **Security Mode**: Message signing disabled (security risk)

**Key Findings:**
```
✓ SMB is exposed and running
✓ Guest access is enabled
✓ Message signing is DISABLED (vulnerability)
✓ Authentication is set to user level
✓ Challenge response is supported
```

### Traceroute Analysis
```
TRACEROUTE
HOP RTT       ADDRESS
1   0.38 ms  192.168.255.128
```
- Single hop indicates local network proximity
- Minimal latency confirms direct accessibility

---

## Phase 2: Nmap Service Enumeration

### Multi-Level NSE Scanning (Screenshot 1 - Continued)

**Scan Command:**
```bash
nmap -sV -sC -O -p80,81,8082,8083,8084,8085,3306,139,445,2222,3307 -sA puerto 192.168.255.128
```

**Scan Parameters:**
- `-sV`: Service version detection
- `-sC`: Standard NSE script scanning
- `-O`: OS detection
- `-p`: Specific port targeting (comprehensive service ports)

**NSE Script Post-Scanning Results:**
```
Starting runlevel 1 (of 3) scan
Completed NSE at 04:25, 0.00s elapsed

Starting runlevel 2 (of 3) scan
Completed NSE at 04:25, 0.00s elapsed

Starting runlevel 3 (of 3) scan
Completed NSE at 04:25, 0.00s elapsed

Data files: /usr/share/nmap
```

**Key Results:**
- 1 IP address identified (1 host up)
- Scan completed in 19.20 seconds
- Packets: 34 sent, 26 received (1.762kB)

---

## Phase 3: Web Application Discovery (Screenshot 2)

### Nmap Report - Service Detection Summary

**Target**: 192.168.255.128

**Address Information:**
- **IPv4**: 192.168.255.128
- **MAC Address**: 00:0C:29:38:8D:53 (VMware)

### Open Ports Identified

#### Port 139 (NetBIOS)
- **State**: Open
- **Service**: SMB/NetBIOS
- **Risk Level**: HIGH

#### Port 445 (SMB)
- **State**: Open
- **Service**: Microsoft-ds
- **Risk Level**: HIGH

#### Port 2222 (SSH)
- **State**: Open
- **Service**: SSH
- **Risk Level**: MEDIUM

#### Port 3306 (MySQL)
- **State**: Open
- **Service**: MySQL Server 5.7.44
- **Certificate**: Auto-generated certificate detected
- **Risk Level**: CRITICAL

#### Port 80 (HTTP)
- **State**: Open
- **Server**: OWASP Juice Shop
- **Content-Type**: text/html; charset=UTF-8
- **Content-Length**: 9903
- **Last Modified**: Sat, 23 May 2026 08:25:43 GMT
- **License**: MIT (Bjoern Kimminich & OWASP Juice Shop contributors)
- **Description**: "Probably the most modern and sophisticated insecure web application"
- **Risk Level**: CRITICAL

### Operating System Fingerprint Detection

**OS Detection Results:**
```
OS SCAN (V=7.99E=45D=5/23<0T=139<CT=<CU=41946\PV=Y\DS=\1<DC=D\G=N\M=000C29NT
```

**Fingerprint Analysis:**
- Linux kernel detected
- Debian-based distribution (64-bit)
- Multiple TCP/IP stack signatures captured
- OS confidence: High

### Host Script Output

#### SMB2 Security Mode
```
Script: smb2-security-mode
Status: COULDN'T ESTABLISH SMBv2 CONNECTION
```

---

## Phase 4: Nmap HTML Report Analysis (Screenshot 3)

### Complete Scan Report - 192.168.255.128
**Scanned**: Sat May 23 03:46:48 2026
**Duration**: 23.64 seconds

### Detailed Port State Analysis

| Port | Protocol | State | Service | Details |
|------|----------|-------|---------|---------|
| 139 | TCP | Open | NetBIOS-ssn | SMB over NetBIOS |
| 445 | TCP | Open | Microsoft-ds | Direct SMB |
| 2222 | TCP | Open | SSH | SSH alternative port |
| 3306 | TCP | Open | MySQL | Database service |

### SSH Certificate Details (Port 2222)

**Key Exchange Algorithm:**
```
1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
ssh-rsa AAAAB3NzaC1yc2EAAAADAIzL7XIYXQBX
[Additional RSA key material...]
2048 56:56:24:8f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
```

**SSH Configuration:**
- RSA authentication available
- DSA key pairs detected
- Custom SSH fingerprints identified

### MySQL Certificate Analysis (Port 3306)

**TLS Certificate Information:**
```
Subject: commonName=MySQL Server 5.7.44 Auto Generated Server Certificate
Issuer: commonName=MySQL Server 5.7.44 Auto Generated CA Certificate
Public Key Type: RSA
```

**Database Exposure Risks:**
- Default auto-generated certificate (self-signed)
- No certificate authority validation
- Plaintext password transmission possible
- SQL injection potential through web interface

---

## Phase 5: Web Application Access & XPath Processing (Screenshot 4)

### Juice Shop Web Interface

**Target**: OWASP Juice Shop on Port 80
**URL**: http://192.168.255.128/

### File Extraction via XSLT

**Commands Executed:**
```bash
xsltproc sbyy.xml -o sbyy.html
firefox puertas.html
```

**XSLT Processing Results:**
- Successfully converted XML to HTML
- Generated interactive web report
- Opened in Firefox for visual analysis

### Generated Report Files

**Output Artifacts:**
1. `puertas.gnmap` - Grepable Nmap format
2. `puertas.html` - HTML report (Primary)
3. `puertas.nmap` - Standard Nmap format
4. `puertas.xml` - XML data export
5. `sbyy.gnmap` - Grepable XSLT output
6. `sbyy.html` - XSLT processed report
7. `sbyy.xml` - Raw XSLT data

---

## Vulnerability Summary

### Critical Vulnerabilities Identified

#### 1. **Exposed SQL Database (Port 3306)**
- **Severity**: CRITICAL
- **Impact**: Direct database access possible
- **Remediation**: Firewall port 3306 from external access

#### 2. **SMB/NetBIOS Exposure (Ports 139, 445)**
- **Severity**: CRITICAL
- **Impact**: File share enumeration, share mounting possible
- **Remediation**: Disable SMB or restrict to trusted networks

#### 3. **Vulnerable Web Application (Port 80)**
- **Severity**: CRITICAL
- **Description**: OWASP Juice Shop is intentionally vulnerable
- **Vulnerabilities Include**:
  - SQL Injection
  - Cross-Site Scripting (XSS)
  - Authentication bypass
  - Insecure Direct Object References (IDOR)
  - Broken cryptography
  - Sensitive data exposure

#### 4. **Message Signing Disabled (SMB)**
- **Severity**: HIGH
- **Impact**: Potential for man-in-the-middle attacks
- **Remediation**: Enable message signing in Samba config

#### 5. **Weak SSH Configuration (Port 2222)**
- **Severity**: MEDIUM
- **Impact**: DSA keys deprecated, weak algorithms
- **Remediation**: Update to ED25519/RSA-4096 keys

---

## Exploitation Walkthrough

### Step 1: Service Enumeration
```bash
# Comprehensive port scanning
nmap -sV -sC -O -p- 192.168.255.128

# SMB enumeration
enum4linux -a 192.168.255.128
smbclient -L //192.168.255.128
```

### Step 2: Web Application Testing
```bash
# Access Juice Shop
firefox http://192.168.255.128/

# Common Juice Shop vulnerabilities to test:
# - Admin bypass (default credentials)
# - SQL injection on login form
# - XSS in user feedback
# - Directory traversal in files endpoint
```

### Step 3: Database Access (Post-Auth)
```bash
# Connect to MySQL
mysql -h 192.168.255.128 -u juice_shop -p

# Query database
SELECT * FROM users;
SELECT * FROM products;
```

### Step 4: Payload Crafting
```bash
# XPath Injection
' or '1'='1
admin' --
' UNION SELECT NULL --

# XSS Payloads
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
```

---

## Lab Exercises

### Exercise 1: Reconnaissance
1. Run Nmap scan against target
2. Identify all open ports
3. Determine OS and services
4. Document findings in report

### Exercise 2: SMB Enumeration
1. Enumerate SMB shares
2. Attempt null session connection
3. List available shares
4. Document permissions

### Exercise 3: Web Application Penetration Testing
1. Explore Juice Shop interface
2. Identify vulnerable endpoints
3. Exploit at least 3 vulnerabilities
4. Document exploitation steps

### Exercise 4: Database Security Testing
1. Attempt unauthorized database access
2. Test SQL injection vectors
3. Extract sensitive data
4. Analyze encryption methods

### Exercise 5: Reporting
1. Create comprehensive penetration test report
2. Include vulnerability classifications
3. Provide remediation recommendations
4. Calculate risk scores

---

## Security Hardening Recommendations

### For Database (MySQL 3306)
```
1. Enable SSL/TLS for all connections
2. Implement strong authentication
3. Restrict network access via firewall
4. Apply principle of least privilege
5. Regular security updates
```

### For SMB Services (139, 445)
```
1. Disable SMBv1 protocol
2. Enable message signing (SMB security mode)
3. Restrict to trusted networks only
4. Implement strong password policies
5. Enable audit logging
```

### For Web Application
```
1. Implement Web Application Firewall (WAF)
2. Enable HTTPS/TLS encryption
3. Implement input validation
4. Apply security headers (CSP, X-Frame-Options, etc.)
5. Regular security patching
```

### For SSH (Port 2222)
```
1. Use modern key algorithms (ED25519)
2. Disable deprecated algorithms (DSA)
3. Implement fail2ban for brute force protection
4. Change default port (already done - port 2222)
5. Disable root login
```

---

## Tools Reference

### Nmap
```bash
nmap -sV -sC -O -p- <target>
nmap --script smb-os-discovery <target>
nmap --script smb-security-mode <target>
```

### SMB Enumeration
```bash
enum4linux <target>
smbclient -L //<target>
rpcclient -U "" <target>
```

### Web Testing
```bash
burp suite
zaproxy
nikto -h <target>
sqlmap -u <url> --forms --batch
```

### Reporting
```bash
xsltproc <input.xml> -o <output.html>
nmap -sV -oX <output.xml> <target>
```

---

## Conclusion

This lab demonstrates a complete penetration testing workflow from reconnaissance through exploitation. The DVWA Juice Shop provides a safe environment to practice security testing techniques while understanding real-world vulnerabilities and attack vectors.

**Key Learnings:**
- Network reconnaissance identifies attack surface
- Service enumeration reveals potential entry points
- Web application testing discovers business logic flaws
- Defense-in-depth approach required for security
- Proper documentation essential for professional reporting

---

## References

- [OWASP Juice Shop](https://owasp-juice.shop)
- [Nmap Security Scanner](https://nmap.org)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [MITRE ATT&CK Framework](https://attack.mitre.org)

---

**Last Updated**: 2026-05-25  
**Lab Status**: Active  
**Difficulty Level**: Intermediate to Advanced
