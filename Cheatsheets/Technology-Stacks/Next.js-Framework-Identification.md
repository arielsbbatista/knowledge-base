# Next.js Framework Identification

**Knowledge Score:** 80/100  
**Last Updated:** June 21, 2026

---

## QUICK COMMANDS

### Identify Next.js
```bash
curl -sI http://target:3001 | grep -E "Server|X-Powered"
curl -s http://target | grep -E "Next|__NEXT|_next"

# Check for Next.js static files
curl http://target/_next/
curl http://target/_next/static/
curl http://target/.next/
```

### Check Next.js Routes
```bash
curl http://target
# Look for: _next/static in HTML source

curl http://target/api/              # API routes
curl http://target/api/health        # Health check
curl http://target/api/ping

# Dynamic routes
curl http://target/[id]
curl http://target/[slug]
```

### Identify API Routes
```bash
curl http://target/api/             # Try base API
ffuf -u http://target/api/FUZZ -w api-endpoints.txt

curl http://target/api/config
curl http://target/api/users
curl http://target/api/admin
```

---

## ENUMERATION PAYLOADS

### Next.js Signatures
```
Port 3000-3001 (common)
_next/static/ in HTML
/__NEXT_DATA__ (NextJS data)
/_next/ routes
/api/ endpoints
```

### Static Asset Discovery
```bash
http://target/_next/static/
http://target/_next/image/
http://target/public/

# Source maps
http://target/_next/static/chunks/*.js.map
http://target/.next/static/chunks/*.js.map
```

### API Endpoint Discovery
```bash
http://target/api/
http://target/api/v1/
http://target/api/auth
http://target/api/auth/login
http://target/api/auth/logout
http://target/api/config
http://target/api/users
```

### Config Files
```bash
http://target/next.config.js
http://target/package.json
http://target/.env
http://target/.env.local
```

---

## EXPLOITATION SCENARIOS

### Scenario 1: Source Map Extraction
```bash
curl http://target/_next/static/chunks/main-*.js.map
# Contains unminified code, variable names, logic
# Reveals potential vulnerabilities
```

### Scenario 2: API Route Enumeration
```bash
curl http://target/api/
# Discover all available API endpoints
ffuf -u http://target/api/FUZZ -w routes.txt
```

### Scenario 3: Configuration Leakage
```bash
curl http://target/next.config.js
curl http://target/package.json
# May reveal: Dependencies, exposed configs
```

### Scenario 4: Authentication Bypass
```bash
# Test API routes for missing auth
curl http://target/api/admin/users
curl http://target/api/admin/settings
curl http://target/api/user/profile
```

### Scenario 5: Server-Side Template Injection
```bash
# Dynamic routes
curl "http://target/api/getdata?template=./../../etc/passwd"
curl "http://target/search/?q=<script>alert(1)</script>"
```

---

## DETECTION & VERIFICATION

### Confirm Next.js
```bash
curl -s http://target | grep "_next"
curl -sI http://target | grep -i "next\|vercel"
```

### Test API Accessibility
```bash
curl http://target/api/
# 405 Method Not Allowed = endpoint exists
# 404 Not Found = doesn't exist
```

### Check for Debug Info
```bash
curl http://target/_next/
curl http://target/__NEXT_DATA__
# Reveals page structure, data
```

---

## COMMON VULNERABILITIES

- **Exposed source maps** (unminified code leak)
- **Unprotected API routes** (no authentication)
- **Environment variable leaks** (.env file accessible)
- **Server-side template injection** (dynamic routes)
- **SSRF via image API** (/_next/image/?url=...)
- **Path traversal** (dynamic imports)

---

## TECHNOLOGY DETECTION

| Indicator | Meaning |
|-----------|---------|
| `_next/static` | Next.js static files |
| `3000/3001` port | Next.js default |
| `/api/[route]` | API endpoint |
| `.js.map` | Source maps present |
| `__NEXT_DATA__` | Page data exposure |

---

## QUICK REFERENCE

| Path | Purpose |
|------|---------|
| /_next/static/ | Static assets |
| /api/ | API routes |
| /public/ | Public files |
| /[slug] | Dynamic routes |

---

## RELATED CHEATSHEETS

- LAMP-Stack-Identification.md
- MERN-Stack-Identification.md
- Django-Framework-Identification.md

---

## REFERENCES

- Next.js Documentation: https://nextjs.org/
- Next.js Security: https://nextjs.org/docs/basic-features/security

---

## CHANGELOG

**v1.0 (2026-06-21)** - Initial creation from Scripts & Notes
- Next.js framework identification
- API route enumeration
- Static asset discovery
- Source map extraction patterns
