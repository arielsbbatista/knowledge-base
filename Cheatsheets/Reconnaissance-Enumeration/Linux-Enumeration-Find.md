# Linux Enumeration - Find Commands

**Knowledge Score:** 92/100  
**Last Updated:** June 21, 2026

---

## QUICK COMMANDS

### Basic Find Syntax
```bash
find / -name "target"                      # Find by name
find / -name "target" -type f              # Files only
find / -name "target" -type d              # Directories only
find / -name "*pattern*"                   # Wildcard search
find / -iname "target"                     # Case insensitive
```

### Find by Permissions
```bash
find / -perm /4000                         # SUID binaries
find / -perm /2000                         # SGID binaries
find / -perm /1000                         # Sticky bit
find / -perm 0777                          # World writable
find / -perm -u=s                          # SUID files
find / -perm -g=s                          # SGID files
```

### Find Writable Directories
```bash
find / -writable -type d 2>/dev/null       # All writable dirs
find / -writable -type f 2>/dev/null       # Writable files
find /tmp -type f 2>/dev/null              # Files in /tmp
find /home -writable -type d 2>/dev/null   # Home dirs
```

### Find by Owner
```bash
find / -user root                          # Root-owned files
find / -user USERNAME                      # User's files
find / -group GROUP                        # Group-owned files
find / -uid 0                              # UID 0 (root)
```

### Find by Time
```bash
find / -mtime 0                            # Modified today
find / -mtime -7                           # Modified in last 7 days
find / -mmin -60                           # Modified in last 60 minutes
find / -atime 0                            # Accessed today
find / -newer /etc/passwd                  # Modified after file
```

---

## ENUMERATION PAYLOADS

### SUID Hunting
```bash
find / -perm /4000 -type f 2>/dev/null
# Output: /usr/bin/sudo, /usr/bin/passwd, /usr/bin/at, custom binaries
```

### SGID Enumeration
```bash
find / -perm /2000 -type f 2>/dev/null
# Output: Group-ID set files
```

### Development Tools
```bash
find / -name "*.php" -o -name "*.py" -o -name "*.js" -o -name "*.java"
# Source code files for potential vulnerabilities
```

### Config Files
```bash
find / -name "*.conf" -o -name "*.config" -o -name "*.cfg"
# Application configurations (may have passwords)

find / -name "web.config" -o -name "config.php" -o -name "settings.py"
# Framework-specific configs
```

### Database Files
```bash
find / -name "*.sql" -o -name "*.db" -o -name "*.sqlite"
# Database files

find / -name "*password*" -o -name "*cred*" -o -name "*secret*"
# Sensitive files
```

### Backup Files
```bash
find / -name "*.bak" -o -name "*.backup" -o -name "*.old"
find / -name "*~" -o -name "*.tmp"
```

### SSH Keys
```bash
find / -name "id_rsa" -o -name "id_ecdsa" -o -name "authorized_keys"
find /home -name ".ssh" -type d 2>/dev/null
```

### World Writable
```bash
find / -perm 0777 -type f 2>/dev/null
# Dangerous: World can read and write
```

---

## EXPLOITATION SCENARIOS

### Scenario 1: SUID Binary Exploitation
```bash
find / -perm /4000 -type f 2>/dev/null | head -20
# Identify: /usr/bin/custom_app (SUID)
# Test: custom_app is GTFOBin or vulnerable
# Exploit: Escalate privileges
```

### Scenario 2: Writable /etc Files
```bash
find /etc -writable -type f 2>/dev/null
# Result: /etc/shadow writable = root escalation
# Result: /etc/sudoers writable = add sudo privilege
```

### Scenario 3: Enumerate Configuration Leaks
```bash
find / -name "config*" -type f 2>/dev/null | xargs grep -l "password\|username\|api_key" 2>/dev/null
# Extract: Database passwords, API tokens
```

### Scenario 4: Find Cron Scripts
```bash
find / -name "*cron*" -type f 2>/dev/null
find /etc/cron* -type f 2>/dev/null
find /var/spool/cron -type f 2>/dev/null
# Identify: User-created cron scripts
# Test: Can we modify cron files?
```

### Scenario 5: Development Artifact Discovery
```bash
find / -name ".git" -o -name ".svn" -o -name ".env"
# Potential: Source code, environment variables, credentials
```

---

## ADVANCED FIND SYNTAX

### Complex Expressions
```bash
# SUID binaries NOT in system directories
find / -perm /4000 -type f ! -path "/usr/bin/*" ! -path "/usr/sbin/*" 2>/dev/null

# Writable files in /opt (custom applications)
find /opt -writable -type f ! -path "*/.*" 2>/dev/null

# Recently modified files in /home
find /home -type f -mtime -30 2>/dev/null | sort
```

### Executing Commands on Results
```bash
find / -perm /4000 -type f 2>/dev/null -exec ls -la {} \;
find / -name "*.php" 2>/dev/null -exec grep -l "eval\|exec\|system" {} \;
```

### Size-Based Enumeration
```bash
find / -size 0                              # Empty files (logs?)
find / -size +100M                         # Large files (database dumps?)
find / -size -1k                           # Small files (scripts?)
```

---

## DETECTION & VERIFICATION

### Confirm SUID Permissions
```bash
ls -la /usr/bin/BINARY
# Output: -rwsr-xr-x = SUID set
# Owner: root = owned by root
```

### Test File Writability
```bash
touch /path/to/file.test 2>&1
# Success = writable
# Permission denied = not writable
```

### Check Cron Execution
```bash
find /var/spool/cron -type f 2>/dev/null
ls -la /etc/cron*
# See scheduled tasks
```

---

## BEST PRACTICES

- Always redirect stderr: `2>/dev/null`
- Start enumeration from common locations
- Combine multiple find conditions
- Check permissions on found files
- Document all interesting findings
- Test file accessibility before exploitation
- Check if you own the file (can modify if owner)

---

## COMMON MISTAKES

❌ Forgetting `2>/dev/null` (clutters output)
❌ Not checking if you own writable file
❌ Assuming SUID = exploitable (check permissions)
❌ Not searching in /opt, /srv, /var (custom locations)
❌ Missing hidden files (use `-o -name ".*"`)

---

## QUICK REFERENCE

| Search | Command | Purpose |
|--------|---------|---------|
| SUID | find / -perm /4000 | Privilege escalation |
| SGID | find / -perm /2000 | Group exploitation |
| Writable | find / -writable -type f | File modification |
| Config | find / -name "*.conf" | Sensitive info |
| Keys | find / -name "id_rsa" | SSH access |
| Backups | find / -name "*.bak" | Old versions |

---

## DIRECTORY REFERENCE

| Path | Purpose | Worth Checking |
|------|---------|---------|
| /usr/bin | System binaries | SUID, privilege escalation |
| /usr/sbin | Root binaries | SUID, privilege escalation |
| /opt | Custom apps | SUID, writable, configs |
| /srv | Services | Web apps, databases |
| /var/www | Web content | Source code, configs |
| /home | User files | SSH keys, backups |
| /tmp | Temporary | Writable, scripts |
| /root | Root home | SSH keys (if accessible) |

---

## RELATED CHEATSHEETS

- Linux-Privilege-Escalation/Linux-SUID-Enumeration.md
- Linux-Privilege-Escalation/Linux-Sudo-Abuse.md

---

## REFERENCES

- Find Manual: `man find`
- Privilege Escalation Guide: https://gtfobins.github.io/

---

## CHANGELOG

**v1.0 (2026-06-21)** - Initial creation
- Quick find commands for file enumeration
- Permission-based searching (SUID, SGID, writable)
- Configuration and credential discovery
- Advanced find expressions
