# Ffuf Fuzzing

**Knowledge Score:** 88/100  
**Last Updated:** June 21, 2026

---

## QUICK COMMANDS

### Directory Fuzzing
```bash
ffuf -u http://target/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
ffuf -u http://target/FUZZ -w wordlist.txt -o results.txt
ffuf -u http://target/FUZZ -w wordlist.txt -e .php,.html,.txt
```

### Subdomain Enumeration
```bash
ffuf -u http://FUZZ.target.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt
ffuf -u http://FUZZ.target.com -w wordlist.txt -v  # Verbose
ffuf -u http://FUZZ.target.com -w wordlist.txt -o subs.txt
```

### Parameter Fuzzing
```bash
ffuf -u "http://target/page?FUZZ=value" -w wordlist.txt
ffuf -u "http://target/page?id=FUZZ" -w numbers.txt
ffuf -u "http://target/page?FUZZ=FUZZ" -w params.txt -w values.txt
```

### Filter Results
```bash
ffuf -u http://target/FUZZ -w wordlist.txt -fc 404           # Filter by status code
ffuf -u http://target/FUZZ -w wordlist.txt -fs 300           # Filter by response size
ffuf -u http://target/FUZZ -w wordlist.txt -fw 50            # Filter by word count
ffuf -u http://target/FUZZ -w wordlist.txt -fl 5             # Filter by line count
```

---

## ENUMERATION PAYLOADS

### Directory Discovery
```bash
# Common directories
ffuf -u http://target/FUZZ -w /usr/share/wordlists/dirb/common.txt

# Admin paths
ffuf -u http://target/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/admin-panels.txt

# Backup files
ffuf -u http://target/FUZZ -w /usr/share/wordlists/seclists/Fuzzing/Backups-Interesting-Files.txt
```

### Extension Enumeration
```bash
ffuf -u http://target/FUZZ -w names.txt -e .php,.html,.txt,.bak,.sql,.zip
```

### Parameter Discovery
```bash
# Common parameters
ffuf -u "http://target/page?FUZZ=1" -w /usr/share/wordlists/seclists/Discovery/Web-Content/Burp-Params-Common.txt

# Check for vulnerabilities
ffuf -u "http://target/page?FUZZ=" -w /usr/share/wordlists/seclists/Fuzzing/Sensitive-Params.txt
```

### Subdomain Hunting
```bash
ffuf -u http://FUZZ.target.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt

# Filter 404s and CNAME records
ffuf -u http://FUZZ.target.com -w subs.txt -fc 404
```

---

## EXPLOITATION SCENARIOS

### Scenario 1: Discover Hidden Admin Panel
```bash
ffuf -u http://target/FUZZ -w common.txt -e .php
# Output: /admin-panel found (200 OK)
# Verify: http://target/admin-panel
```

### Scenario 2: Find Backup Files
```bash
ffuf -u http://target/FUZZ -w backup-wordlist.txt -e .bak,.sql,.zip
# Output: /config.sql.bak found
# Extract: Database credentials
```

### Scenario 3: Parameter Enumeration for SQLi
```bash
ffuf -u "http://target/product.php?FUZZ=1" -w param-list.txt
# Output: /product.php?id=1 exists
# Test: /product.php?id=1' OR '1'='1
```

### Scenario 4: Subdomain Takeover Detection
```bash
ffuf -u http://FUZZ.target.com -w subs.txt
# Identify: Subdomains with 404s but registered
# Test: CNAME points to unclaimed service
```

### Scenario 5: API Endpoint Discovery
```bash
ffuf -u http://target/api/FUZZ -w api-endpoints.txt
ffuf -u http://target/api/v1/FUZZ -w resources.txt
# Discover: /api/users, /api/admin, /api/config
```

---

## DETECTION & VERIFICATION

### Test for Valid Responses
```bash
ffuf -u http://target/FUZZ -w wordlist.txt -v | grep "200\|301\|302"
```

### Identify Size Patterns
```bash
# See response sizes to filter intelligently
ffuf -u http://target/FUZZ -w wordlist.txt
# Note: 404 responses are typically same size
# Filter: -fs 4096 (filter by size)
```

### Check Rate Limiting
```bash
ffuf -u http://target/FUZZ -w wordlist.txt -r  # Follow redirects
ffuf -u http://target/FUZZ -w wordlist.txt -H "X-Forwarded-For: 127.0.0.1"  # Bypass IP limit
```

---

## ADVANCED TECHNIQUES

### Multiple Wordlists
```bash
ffuf -u http://target/FUZZ -w wordlist1.txt:FUZZ -w wordlist2.txt:EXT -e FUZZ
```

### Request Modification
```bash
ffuf -u http://target/FUZZ -w wordlist.txt \
  -H "Authorization: Bearer TOKEN" \
  -H "User-Agent: CustomAgent" \
  -b "session=SESSIONID"
```

### JSON/POST Data Fuzzing
```bash
ffuf -u http://target/api -w wordlist.txt \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"parameter": "FUZZ"}'
```

### Proxy & Intercept
```bash
ffuf -u http://target/FUZZ -w wordlist.txt -x http://localhost:8080
# Route through Burp Suite for inspection
```

---

## BEST PRACTICES

- Start with small wordlists, expand if needed
- Filter by status code (remove 404s)
- Use `-e` to specify extensions (faster than trying all)
- Combine with `-v` to see full responses
- Use `-r` to follow redirects
- Monitor for rate limiting (add delays with `-rate`)
- Save results with `-o` for documentation
- Use appropriate wordlists (DNS vs Web-Content)

---

## COMMON MISTAKES

❌ Using default wordlist (too slow, many false positives)
❌ Not filtering 404 responses (-fc 404)
❌ Forgetting to try file extensions (-e .php)
❌ Running without filtering by size (-fs)
❌ Not checking status codes in output
❌ Using insufficient wordlist size

---

## QUICK REFERENCE

| Flag | Purpose | Example |
|------|---------|---------|
| -u | URL with FUZZ | ffuf -u http://target/FUZZ |
| -w | Wordlist | ffuf -w wordlist.txt |
| -e | Extensions | ffuf -e .php,.html |
| -fc | Filter status | ffuf -fc 404 |
| -fs | Filter size | ffuf -fs 1024 |
| -fw | Filter words | ffuf -fw 50 |
| -o | Output file | ffuf -o results.txt |
| -v | Verbose | ffuf -v |
| -H | Add header | ffuf -H "Auth: token" |
| -b | Add cookie | ffuf -b "session=id" |

---

## WORDLIST RESOURCES

- SecLists: https://github.com/danielmiessler/SecLists
- Common.txt: /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
- Dirb: /usr/share/wordlists/dirb/
- DNS Subdomains: /usr/share/wordlists/seclists/Discovery/DNS/

---

## RELATED CHEATSHEETS

- Gobuster-Web-Enumeration.md (similar tool)
- Nikto-Web-Scanning.md (vulnerability scanning)

---

## REFERENCES

- Ffuf GitHub: https://github.com/ffuf/ffuf
- Ffuf Documentation: https://github.com/ffuf/ffuf/wiki

---

## CHANGELOG

**v1.0 (2026-06-21)** - Initial creation from Scripts & Notes
- Quick command reference for fuzzing
- Directory and subdomain enumeration
- Parameter and API endpoint discovery
- Advanced filtering techniques
