# Nmap Reconnaissance Cheatsheet

**Knowledge Score:** 95/100

## Overview
Structured port scanning workflow for initial target enumeration.

## Basic Syntax

```bash
sudo nmap -sC -sS -oA nmap_log.txt <target>
```

## Parameters Explained

- **-sC**: Script scanning (OS detection, version detection, service enumeration)
- **-sS**: SYN stealth scan (half-open connections, less detectable)
- **-oA**: Output all formats (normal, grep, XML for different tools)

## Extended Port Scanning

```bash
sudo nmap -sC -sS -p1-9000 -oA nmap_log.txt <target>
```

### When to Use Extended Range
- Default scan reveals only common services (SSH, HTTP)
- No clear attack surface identified in initial results
- Unusual ports might host critical services (Redis 6379, MongoDB 27017)
- Time permits extended scanning

## Common Port Ranges

| Range | Purpose |
|-------|---------|
| 1-1024 | Well-known ports (standard services) |
| 1-9000 | Extended range (catches alternative services) |
| 1-65535 | Full scan (time-intensive, use if justified) |

## Examples from Practice

**Agent T:**
- Initial scan (default ports): SSH (22), HTTP (80)
- Observation: Results seemed incomplete
- Expanded scan (-p1-9000): Discovered Redis on 6379
- Result: Alternative attack vector identified

## Best Practices

1. Start with standard scan
2. If results seem incomplete, expand port range
3. Use -oA for persistent logging (cross-compatible formats)
4. Cross-reference results with service vulnerability database
5. Don't assume top ports are the only services

## See Also

- [[Iterative-Enumeration]]
- [[Service-Fingerprinting-Vulnerability-Matching]]

## References

- [Nmap Official Documentation](https://nmap.org/book/man.html)
- OSCP Enumeration Methodology
