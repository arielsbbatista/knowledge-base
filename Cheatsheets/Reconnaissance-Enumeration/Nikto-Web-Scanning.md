# Nikto Web Scanning

**Knowledge Score:** 88/100  
**Last Updated:** June 21, 2026

---

## QUICK COMMANDS

### Basic Scan
```bash
nikto -h http://target.com
nikto -h 192.168.1.100:8080
nikto -h http://target.com -p 80,443,8080
```

### Output Options
```bash
nikto -h http://target -o scan.txt           # Text output
nikto -h http://target -o scan.html -Format html  # HTML report
nikto -h http://target -o scan.xml -Format xml    # XML output
```

### Scan Specific Paths
```bash
nikto -h http://target -C all        # Check for default files
nikto -h http://target -root /app    # Scan specific directory
```

### Full Enumeration
```bash
nikto -h http://target -C all -Plugins tests -o full_scan.html -Format html
nikto -H http://target -Tuning x,y,z # Custom tuning
```

---

## ENUMERATION PAYLOADS

### Check for Default Installations
```bash
nikto -h http://target -C all
# Detects: phpmyadmin, WordPress, Joomla, Drupal, etc.
```

### Identify Stack & Vulnerabilities
```bash
nikto -h http://target
# Reports: Server type, plugins, potential vulns
```

### Scan for Specific Vulnerabilities
```bash
nikto -h http://target -Tuning 1,2,3,4,5,6,7,8,9
# 1=Interesting files  2=Interesting dirs  3=File upload forms
# 4=Authentication bypass  5=Software ID  6=Doc  7=General  8=All
```

### Check for Outdated Software
```bash
nikto -h http://target -output report.html -Format html
# Report shows outdated server/app versions
```

---

## EXPLOITATION SCENARIOS

### Scenario 1: Discover Vulnerable Installation
```bash
nikto -h http://target:8080
# Output: "Server: Apache/2.4.1 - outdated, vulnerable to X"
# Find: Default paths like /phpmyadmin, /wp-admin, /cpanel
```

### Scenario 2: Detect Enabled Dangerous Methods
```bash
nikto -h http://target
# Check: PUT/DELETE/TRACE methods enabled?
# Check: WebDAV enabled?
# Check: OPTIONS available?
```

### Scenario 3: Find Sensitive Directories
```bash
nikto -h http://target -C all
# Detects: /admin, /backup, /config, /wp-content, /uploads
# Identifies: robots.txt, .htaccess, web.config
```

### Scenario 4: Enumerate API Endpoints
```bash
nikto -h http://target/api/v1/
# Finds: Unprotected endpoints, default credentials, exposed configs
```

### Scenario 5: Check for Clickjacking Protection
```bash
nikto -h http://target
# Reports: X-Frame-Options missing (clickjacking vulnerable)
# Reports: X-Content-Type-Options missing (MIME sniffing)
# Reports: HSTS missing (SSL stripping possible)
```

---

## DETECTION & VERIFICATION

### Identify Vulnerable Headers
```bash
nikto -h http://target 2>&1 | grep -E "X-Frame|X-Content|HSTS|Server"
```

### Check for Authentication Bypass
```bash
nikto -h http://target | grep -i "authentication\|bypass\|default"
```

### Verify Findings
```bash
curl -I http://target  # Check response headers
curl -X OPTIONS http://target  # Check allowed methods
curl -X PROPFIND http://target  # Check WebDAV
```

---

## TUNING OPTIONS

### Tuning Values Reference
```
1 = Interesting files
2 = Interesting directories  
3 = File upload forms
4 = Authentication bypass
5 = Software identification
6 = Doc/Info pages
7 = Generic/General vulnerabilities
8 = Administrative pages
9 = Interesting parameter handling
0 = Filename extensions
a = Apache specific checks
b = Interesting backdoors
c = CGI Directory
d = WebDAV/Misconfiguration
e = Error handling
x = Reverse proxy checks
```

### Combinations
```bash
nikto -h target -Tuning 1,2,3,4,5,6,7,8,9  # All checks
nikto -h target -Tuning a,b,d,e            # Specific focus
nikto -h target -Tuning x                  # Reverse proxy only
```

---

## BEST PRACTICES

- Run baseline scan first, then targeted scans
- Use output formats for documentation (HTML/XML)
- Combine with manual inspection of findings
- Cross-reference with vulnerability databases
- Update nikto plugins regularly: `nikto -update`
- Scan during maintenance windows (can cause server stress)
- Use proxy mode to intercept results: `nikto -h target -useproxy http://localhost:8080`
- Document false positives for this target

---

## COMMON MISTAKES

❌ Not checking output headers for important info
❌ Ignoring "Server" header clues about version
❌ Not following up on "Interesting files" found
❌ Running without knowing tuning options (runs all by default)
❌ Not combining with manual header inspection
❌ Forgetting to check for backdoor detection

---

## QUICK REFERENCE

| Flag | Purpose | Example |
|------|---------|---------|
| -h | Host | nikto -h http://target |
| -p | Port | nikto -h target -p 8080 |
| -C | Scan plugins (all) | nikto -h target -C all |
| -o | Output file | nikto -h target -o report.txt |
| -Format | Output format | nikto -h target -Format html |
| -Tuning | Specific checks | nikto -h target -Tuning 1,2,3 |

---

## RELATED CHEATSHEETS

- Gobuster-Web-Enumeration.md (directory enumeration)
- Nmap-Reconnaissance.md (port scanning)

---

## REFERENCES

- Nikto Official: https://www.cirt.net/nikto2
- GitHub: https://github.com/sullo/nikto

---

## CHANGELOG

**v1.0 (2026-06-21)** - Initial creation from Scripts & Notes
- Quick command reference
- Web vulnerability scanning techniques
- Tuning options for targeted scans
- Common vulnerability detection patterns
