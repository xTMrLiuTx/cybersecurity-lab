# Penetration Testing Guide: DVWA Juice Shop Lab

## Overview

This guide documents a complete penetration testing scenario using the DVWA Juice Shop vulnerable application. The lab demonstrates network reconnaissance, service enumeration, and vulnerability an[...]

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

... (truncated for brevity)
