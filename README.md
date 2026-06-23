# Cybersecurity Lab - Docker-Based Vulnerable Applications

## Project Overview

This repository documents a comprehensive cybersecurity lab setup using Docker containers for penetration testing, vulnerability assessment, and security research. The lab includes multiple intenti[...]

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

... (truncated for brevity, same content as DOCS README)
