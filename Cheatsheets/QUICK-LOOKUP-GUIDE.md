# CHEATSHEET LOOKUP GUIDE - Quick Navigation

**All files organized by category subfolder**

Location: ~/Documentos/Obsidian/Obsidian-home/Cyber_Career/Knowledge/Cheatsheets/

---

## FOLDER STRUCTURE

```
Cheatsheets/
├── Linux-Privilege-Escalation/          [5 files]
├── Windows-Privilege-Escalation/        [2 files]
├── Web-Exploitation/                    [9 files]
├── Reconnaissance-Enumeration/          [8 files]
├── Technology-Stacks/                   [4 files]
└── System-Administration/               [1 file]
```

---

## 🔍 FIND WHAT YOU NEED

### Need SQL INJECTION Exploits?
**Folder:** Web-Exploitation/
- SQL-Injection-Fundamentals.md ← START HERE (payloads)
- SQLi-Burp-Suite.md (tool automation)
- SQLi-Python-Automation.md (script-based)
- SQLi-Bash-Exploitation.md (one-liners)

### Need IDOR (Insecure Direct Object Reference) Testing?
**Folder:** Web-Exploitation/
- IDOR-Burp-Suite.md ← START HERE
- IDOR-Bash-Enumeration.md
- IDOR-Python-Automation.md

### Need WEB SHELL / PHP RCE?
**Folder:** Web-Exploitation/
- PHP-Web-Shell-Techniques.md ← START HERE

### Need IIS Exploitation?
**Folder:** Web-Exploitation/
- IIS-8.3-Shortname-Vulnerability.md ← START HERE

---

### Need to ENUMERATE / DISCOVER?
**Folder:** Reconnaissance-Enumeration/
- Nmap-Reconnaissance.md ← Port scanning
- Nikto-Web-Scanning.md ← Web vulns
- Gobuster-Web-Enumeration.md ← Directory bruteforce
- Ffuf-Fuzzing.md ← Fuzzing (dirs/params/subs)
- Curl-Web-Requests.md ← HTTP requests
- Linux-Enumeration-Find.md ← Linux file discovery
- TCPdump-Network-Capture.md ← Network capture
- Redis-CLI-Enumeration.md ← Redis commands

### Need LINUX PRIVILEGE ESCALATION?
**Folder:** Linux-Privilege-Escalation/
- Linux-SUID-Enumeration.md
- Linux-Sudo-Abuse.md
- Linux-Cron-Jobs-Exploitation.md
- Linux-PATH-Hijacking.md
- Linux-NFS-Exploitation.md

### Need WINDOWS PRIVILEGE ESCALATION?
**Folder:** Windows-Privilege-Escalation/
- Windows-Credential-Harvesting.md
- IIS-WebDAV-Exploitation.md

---

### Need to IDENTIFY TECHNOLOGY STACK?
**Folder:** Technology-Stacks/
- LAMP-Stack-Identification.md (Apache/PHP/MySQL)
- MERN-Stack-Identification.md (Node.js/Express/React/MongoDB)
- Django-Framework-Identification.md (Python Django)
- Next.js-Framework-Identification.md (React/Next.js)

### Need SYSTEM/PORT MANAGEMENT?
**Folder:** System-Administration/
- Port-Management-Utility.md (kill process, find port)

---

## 📋 BY TOOL/COMMAND

| Tool | Cheatsheet | Folder |
|------|-----------|--------|
| Nmap | Nmap-Reconnaissance.md | Reconnaissance-Enumeration/ |
| Nikto | Nikto-Web-Scanning.md | Reconnaissance-Enumeration/ |
| Curl | Curl-Web-Requests.md | Reconnaissance-Enumeration/ |
| Ffuf | Ffuf-Fuzzing.md | Reconnaissance-Enumeration/ |
| Gobuster | Gobuster-Web-Enumeration.md | Reconnaissance-Enumeration/ |
| TCPdump | TCPdump-Network-Capture.md | Reconnaissance-Enumeration/ |
| Burp Suite | SQLi-Burp-Suite.md, IDOR-Burp-Suite.md | Web-Exploitation/ |
| Bash | SQLi-Bash-Exploitation.md, IDOR-Bash-Enumeration.md | Web-Exploitation/ |
| Python | SQLi-Python-Automation.md, IDOR-Python-Automation.md | Web-Exploitation/ |
| Find | Linux-Enumeration-Find.md | Reconnaissance-Enumeration/ |
| ss/lsof | Port-Management-Utility.md | System-Administration/ |

---

## 🎯 BY VULNERABILITY TYPE

| Vulnerability | Cheatsheets | Folder |
|---------------|------------|--------|
| SQL Injection | SQL-Injection-Fundamentals.md, SQLi-* (4 total) | Web-Exploitation/ |
| IDOR | IDOR-Burp-Suite.md, IDOR-Bash-Enumeration.md, IDOR-Python-Automation.md | Web-Exploitation/ |
| IIS 8.3 Shortname | IIS-8.3-Shortname-Vulnerability.md | Web-Exploitation/ |
| PHP RCE/Shell | PHP-Web-Shell-Techniques.md | Web-Exploitation/ |
| Linux PrivEsc | Linux-SUID-Enumeration.md, Linux-Sudo-Abuse.md, Linux-Cron-Jobs-Exploitation.md, Linux-PATH-Hijacking.md, Linux-NFS-Exploitation.md | Linux-Privilege-Escalation/ |
| Windows PrivEsc | Windows-Credential-Harvesting.md, IIS-WebDAV-Exploitation.md | Windows-Privilege-Escalation/ |

---

## 🔧 QUICK COMMAND REFERENCE

### SQL Injection
```bash
# Go to: Web-Exploitation/SQL-Injection-Fundamentals.md
# First 20 lines = copy-paste payloads
1' UNION SELECT NULL,NULL,NULL -- 
1' UNION SELECT database(),user(),version() -- 
```

### Find SUID Binaries
```bash
# Go to: Linux-Privilege-Escalation/Linux-SUID-Enumeration.md
# OR: Reconnaissance-Enumeration/Linux-Enumeration-Find.md
find / -perm /4000 -type f 2>/dev/null
```

### Web Scanning
```bash
# Go to: Reconnaissance-Enumeration/Nikto-Web-Scanning.md
nikto -h http://target
```

### Directory Fuzzing
```bash
# Go to: Reconnaissance-Enumeration/Ffuf-Fuzzing.md
ffuf -u http://target/FUZZ -w wordlist.txt
```

### Find Process on Port
```bash
# Go to: System-Administration/Port-Management-Utility.md
lsof -i :PORT
kill -9 PID
```

---

## 📂 COMPLETE FOLDER LIST

### Linux-Privilege-Escalation/
```
1. Linux-SUID-Enumeration.md (95/100)
2. Linux-Sudo-Abuse.md (95/100)
3. Linux-Cron-Jobs-Exploitation.md (90/100)
4. Linux-PATH-Hijacking.md (90/100)
5. Linux-NFS-Exploitation.md (85/100)
```

### Windows-Privilege-Escalation/
```
1. Windows-Credential-Harvesting.md (92/100)
2. IIS-WebDAV-Exploitation.md (88/100)
```

### Web-Exploitation/
```
1. SQL-Injection-Fundamentals.md (95/100)
2. IIS-8.3-Shortname-Vulnerability.md (85/100)
3. PHP-Web-Shell-Techniques.md (88/100)
4. SQLi-Burp-Suite.md
5. SQLi-Python-Automation.md
6. SQLi-Bash-Exploitation.md
7. IDOR-Burp-Suite.md
8. IDOR-Bash-Enumeration.md
9. IDOR-Python-Automation.md
```

### Reconnaissance-Enumeration/
```
1. Nmap-Reconnaissance.md
2. Nikto-Web-Scanning.md (88/100)
3. Gobuster-Web-Enumeration.md
4. Ffuf-Fuzzing.md (88/100)
5. Curl-Web-Requests.md (90/100)
6. TCPdump-Network-Capture.md (87/100)
7. Linux-Enumeration-Find.md (92/100)
8. Redis-CLI-Enumeration.md
```

### Technology-Stacks/
```
1. LAMP-Stack-Identification.md (85/100)
2. MERN-Stack-Identification.md (82/100)
3. Django-Framework-Identification.md (83/100)
4. Next.js-Framework-Identification.md (80/100)
```

### System-Administration/
```
1. Port-Management-Utility.md (85/100)
```

---

## ⚡ HOW TO USE (3 STEPS)

1. **Identify your need** (from sections above)
2. **Go to folder/file**
3. **Jump to "QUICK COMMANDS"** section (first 30 lines = copy-paste ready)

**Example:**
- Need SQL injection? → Web-Exploitation/SQL-Injection-Fundamentals.md
- Open file → "QUICK COMMANDS" section
- Copy payload, modify target
- Execute

---

## 📊 TOTAL STATS

- **Total Cheatsheets:** 33
- **Organized Folders:** 6 categories
- **Commands:** 500+
- **Scenarios:** 80+
- **Size:** ~290 KB

---

## 🚀 ALL READY TO USE

Navigate to any folder, open the cheatsheet, and find your command in the first 30 lines!

