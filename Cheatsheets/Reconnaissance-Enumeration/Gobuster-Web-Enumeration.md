# Gobuster Web Directory Enumeration Cheatsheet

**Knowledge Score:** 90/100

## Overview
Automated directory and file discovery for web applications. Essential for identifying hidden attack surfaces.

## Basic Syntax

```bash
gobuster dir -u <URL> -w <wordlist> -x <extensions> [options]
```

## Parameters Explained

- **dir**: Directory/file brute-force mode
- **-u**: Target URL (with protocol: http://target.thm/)
- **-w**: Wordlist path (directory names to test)
- **-x**: File extensions to append (php,js,txt,html)
- **--exclude-length**: Exclude responses of specific length (filters false positives)

## Common Command Template

```bash
gobuster dir -u http://target.thm/ -w /usr/share/wordlists/dirb/common.txt \
  -x php,js,txt,html --exclude-length 42131
```

## Wordlist Selection

| Wordlist | Use Case | Speed |
|----------|----------|-------|
| /usr/share/wordlists/dirb/common.txt | General directories | Fast |
| /usr/share/wordlists/dirb/big.txt | Comprehensive discovery | Slow |
| Custom wordlists | Technology-specific (CMS, framework) | Varies |

## Exclude-Length Strategy

1. Run initial scan to identify default/error page response sizes
2. Extract the length of false positive responses
3. Use --exclude-length to filter out those responses
4. Example: Apache default page returns 42131 bytes

## Examples from Practice

**Agent T:**
- Command: `gobuster dir -u http://agentt.thm/ -w /usr/share/wordlists/dirb/common.txt -x php,js,txt,html --exclude-length 42131`
- Found: blank.html, 404.html
- Assessment: Limited web attack surface, no hidden directories

## Best Practices

1. Identify and exclude error pages first (reduces noise)
2. Target relevant extensions for the technology stack
3. Use appropriate wordlist size (balance speed vs coverage)
4. Parse results carefully for false positives
5. Consider recursive scanning for found directories

## See Also

- [[Service-Fingerprinting-Vulnerability-Matching]]
- [[Manual-Validation-Before-Exploit]]

## References

- [Gobuster GitHub](https://github.com/OJ/gobuster)
- OWASP Directory Traversal Testing
