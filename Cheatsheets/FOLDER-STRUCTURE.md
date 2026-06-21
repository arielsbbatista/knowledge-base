# CHEATSHEET LIBRARY - ORGANIZED STRUCTURE

**Final Organization Complete** ✓  
**Date:** June 21, 2026

---

## 📁 FOLDER STRUCTURE

```
Knowledge/Cheatsheets/
│
├── QUICK-LOOKUP-GUIDE.md ← START HERE (Navigation Hub)
│
├── 📂 Linux-Privilege-Escalation/ [5 cheatsheets]
│   ├── Linux-SUID-Enumeration.md (95/100)
│   ├── Linux-Sudo-Abuse.md (95/100)
│   ├── Linux-Cron-Jobs-Exploitation.md (90/100)
│   ├── Linux-PATH-Hijacking.md (90/100)
│   └── Linux-NFS-Exploitation.md (85/100)
│
├── 📂 Windows-Privilege-Escalation/ [2 cheatsheets]
│   ├── Windows-Credential-Harvesting.md (92/100)
│   └── IIS-WebDAV-Exploitation.md (88/100)
│
├── 📂 Web-Exploitation/ [9 cheatsheets]
│   ├── SQL-Injection-Fundamentals.md (95/100) ⭐
│   ├── SQLi-Burp-Suite.md
│   ├── SQLi-Python-Automation.md
│   ├── SQLi-Bash-Exploitation.md
│   ├── IDOR-Burp-Suite.md
│   ├── IDOR-Bash-Enumeration.md
│   ├── IDOR-Python-Automation.md
│   ├── PHP-Web-Shell-Techniques.md (88/100)
│   └── IIS-8.3-Shortname-Vulnerability.md (85/100)
│
├── 📂 Reconnaissance-Enumeration/ [8 cheatsheets]
│   ├── Nmap-Reconnaissance.md
│   ├── Nikto-Web-Scanning.md (88/100) ⭐
│   ├── Gobuster-Web-Enumeration.md
│   ├── Ffuf-Fuzzing.md (88/100) ⭐
│   ├── Curl-Web-Requests.md (90/100) ⭐
│   ├── TCPdump-Network-Capture.md (87/100)
│   ├── Linux-Enumeration-Find.md (92/100) ⭐
│   └── Redis-CLI-Enumeration.md
│
├── 📂 Technology-Stacks/ [4 cheatsheets]
│   ├── LAMP-Stack-Identification.md (85/100)
│   ├── MERN-Stack-Identification.md (82/100)
│   ├── Django-Framework-Identification.md (83/100)
│   └── Next.js-Framework-Identification.md (80/100)
│
├── 📂 System-Administration/ [1 cheatsheet]
│   └── Port-Management-Utility.md (85/100)
│
└── 📂 Vulnerability-Exploitation/ [5 reference docs]
    ├── PHASE-1-COMPLETION-SUMMARY.md
    ├── AVAILABLE-CONTENT-ANALYSIS.md
    ├── COMPLETE-REPORT.md
    ├── KNOWLEDGE-BASE-SUMMARY.md
    └── DECISION-POINT.md

⭐ = Highest value, most used commands
```

---

## 🎯 QUICK NAVIGATION

### I need to EXPLOIT something
- **SQL Injection?** → Web-Exploitation/SQL-Injection-Fundamentals.md
- **PHP Shell?** → Web-Exploitation/PHP-Web-Shell-Techniques.md
- **IDOR Bug?** → Web-Exploitation/IDOR-Burp-Suite.md
- **Linux PrivEsc?** → Linux-Privilege-Escalation/
- **Windows PrivEsc?** → Windows-Privilege-Escalation/
- **IIS 8.3?** → Web-Exploitation/IIS-8.3-Shortname-Vulnerability.md

### I need to ENUMERATE/SCAN
- **Port scanning?** → Reconnaissance-Enumeration/Nmap-Reconnaissance.md
- **Web scanning?** → Reconnaissance-Enumeration/Nikto-Web-Scanning.md
- **Directory fuzzing?** → Reconnaissance-Enumeration/Ffuf-Fuzzing.md
- **HTTP requests?** → Reconnaissance-Enumeration/Curl-Web-Requests.md
- **Network traffic?** → Reconnaissance-Enumeration/TCPdump-Network-Capture.md
- **Linux files?** → Reconnaissance-Enumeration/Linux-Enumeration-Find.md

### I need to IDENTIFY TECHNOLOGY
- **Apache/PHP/MySQL?** → Technology-Stacks/LAMP-Stack-Identification.md
- **Node.js/Express?** → Technology-Stacks/MERN-Stack-Identification.md
- **Django?** → Technology-Stacks/Django-Framework-Identification.md
- **Next.js/React?** → Technology-Stacks/Next.js-Framework-Identification.md

### I need SYSTEM MANAGEMENT
- **Kill process on port?** → System-Administration/Port-Management-Utility.md

---

## 📊 STATISTICS

| Category | Count | Knowledge Score |
|----------|-------|-----------------|
| Linux-Privilege-Escalation | 5 | 85-95/100 |
| Windows-Privilege-Escalation | 2 | 88-92/100 |
| Web-Exploitation | 9 | 85-95/100 |
| Reconnaissance-Enumeration | 8 | 87-92/100 |
| Technology-Stacks | 4 | 80-85/100 |
| System-Administration | 1 | 85/100 |
| **TOTAL** | **29 active** | **~88/100** |

**+ 5 reference docs in Vulnerability-Exploitation/**

**Total: 35 files, ~290 KB, 500+ commands, 80+ scenarios**

---

## 🚀 HOW TO USE

### Step 1: Open QUICK-LOOKUP-GUIDE.md
- Shows all folders and categories
- Explains what's in each

### Step 2: Navigate to relevant folder
- Find what you need by category

### Step 3: Open the cheatsheet
- Go to "QUICK COMMANDS" section
- First 30 lines = copy-paste ready

### Step 4: Copy, modify, execute
- Only change TARGET/command variables
- Execute immediately

---

## 💡 PRO TIPS

✓ **QUICK-LOOKUP-GUIDE.md** = Your navigation hub  
✓ Each cheatsheet has commands ready to copy-paste  
✓ First 30 lines = most common usage  
✓ Exploitation Scenarios = understand context  
✓ Quick Reference Table = lookup flags/options  

---

## 🔍 FIND BY PURPOSE

**Exploitation:**
- Web-Exploitation/ (SQL, IDOR, PHP, IIS)
- Linux-Privilege-Escalation/ (SUID, Sudo, Cron, PATH, NFS)
- Windows-Privilege-Escalation/ (Creds, WebDAV)

**Enumeration:**
- Reconnaissance-Enumeration/ (Nmap, Nikto, Ffuf, Curl, etc)

**Identification:**
- Technology-Stacks/ (LAMP, MERN, Django, Next.js)

**Administration:**
- System-Administration/ (Port management)

---

## 📍 DIRECT FILE PATHS

```bash
# Linux PrivEsc
~/Documentos/Obsidian/Obsidian-home/Cyber_Career/Knowledge/Cheatsheets/Linux-Privilege-Escalation/

# Windows PrivEsc
~/Documentos/Obsidian/Obsidian-home/Cyber_Career/Knowledge/Cheatsheets/Windows-Privilege-Escalation/

# Web Exploitation
~/Documentos/Obsidian/Obsidian-home/Cyber_Career/Knowledge/Cheatsheets/Web-Exploitation/

# Reconnaissance
~/Documentos/Obsidian/Obsidian-home/Cyber_Career/Knowledge/Cheatsheets/Reconnaissance-Enumeration/

# Tech Stacks
~/Documentos/Obsidian/Obsidian-home/Cyber_Career/Knowledge/Cheatsheets/Technology-Stacks/

# System Admin
~/Documentos/Obsidian/Obsidian-home/Cyber_Career/Knowledge/Cheatsheets/System-Administration/
```

---

## ✅ ORGANIZATION COMPLETE

All 33 cheatsheets are now organized in 6 logical categories with a single QUICK-LOOKUP-GUIDE for navigation!

**Start with:** QUICK-LOOKUP-GUIDE.md

