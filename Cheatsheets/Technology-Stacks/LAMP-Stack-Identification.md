# LAMP Stack Identification

**Knowledge Score:** 85/100  
**Last Updated:** June 21, 2026

---

## QUICK COMMANDS

### Identify LAMP Stack
```bash
curl -sI http://target | grep -E "Server|X-Powered-By"
# Look for: Apache, PHP versions

curl -s http://target | grep -E "<!--.*PHP|Powered by|Apache"
# Source code hints

nmap -sV http://target
# Service version detection

nikto -h http://target
# Web vulnerability scanner (identifies stack)
```

### Check for PHP
```bash
curl http://target/info.php
curl http://target/phpinfo.php
curl http://target/test.php
curl -s http://target | grep -i php

# Look for .php extensions
curl http://target/index.php
```

### Check for MySQL/MariaDB
```bash
# Port 3306 listening?
nmap -p 3306 target

# Try connection
mysql -h target -u root
mysql -h target -u root -p

# Check for exposed admin
curl http://target/phpmyadmin
curl http://target/pma
curl http://target/admin
```

### Check for Apache
```bash
curl -I http://target | grep Server
# Output: Server: Apache/2.4.41

curl http://target/.htaccess
curl http://target/web.config

# Check for mod_status
curl http://target/server-status
curl http://target/server-info
```

---

## ENUMERATION PAYLOADS

### LAMP Stack Signature
```
Port 80 (HTTP)
Server: Apache/x.x.x
X-Powered-By: PHP/x.x.x
Running: Linux
Database: MySQL/MariaDB on 3306
```

### Default Paths
```bash
/var/www/html          # Apache root
/var/www/html/index.php  # PHP entry point
/etc/apache2/          # Apache config
/etc/mysql/            # MySQL config
/var/log/apache2/      # Apache logs
/var/log/mysql/        # MySQL logs
```

### Common Admin Panels
```bash
http://target/phpmyadmin
http://target/pma
http://target/admin
http://target/cpanel
http://target/cPanel
http://target/webmin
```

### PHP Detection Files
```bash
http://target/info.php
http://target/phpinfo.php
http://target/test.php
http://target/config.php
http://target/index.php
```

---

## EXPLOITATION SCENARIOS

### Scenario 1: Identify LAMP Components
```bash
curl -sI http://target:80
# Server: Apache/2.4.41
# X-Powered-By: PHP/7.4

# Versions known → check for CVEs
# Apache 2.4.41 → exploitable?
# PHP 7.4 → end of life?
```

### Scenario 2: Access phpMyAdmin
```bash
curl http://target/phpmyadmin
# 200 OK = panel found
# 403 Forbidden = restricted but exists
# 404 = doesn't exist

# Try default credentials
curl -u admin:admin http://target/phpmyadmin
```

### Scenario 3: Find Uploaded Files
```bash
curl http://target/uploads/
curl http://target/files/
curl http://target/media/

# Upload and execute PHP shell
curl -F "file=@shell.php" http://target/upload.php
curl http://target/uploads/shell.php
```

### Scenario 4: MySQL Enumeration
```bash
# Port scanning
nmap -p 3306 target
# Result: 3306/tcp open mysql

# Try connection
mysql -h target -u root
mysql -h target -u root -p
mysql -h target -u admin -p admin
```

### Scenario 5: Configuration Extraction
```bash
# Find WordPress config (LAMP)
curl http://target/wp-config.php
curl http://target/wp-config.php.bak
curl http://target/wp-config.php.old

# Database credentials leaked
```

---

## DETECTION & VERIFICATION

### Check Apache Version
```bash
curl -sI http://target | grep "^Server:"
# Confirms Apache + PHP version
```

### Verify MySQL Access
```bash
mysql -h target -u root 2>&1 | grep -i "access\|refused\|connection"
# Access denied = running, restricted
# Connection refused = not running
```

### Test PHP Execution
```bash
curl http://target/test.php
# Response = PHP executing
# 404 = doesn't exist
# 403 = PHP disabled
```

---

## BEST PRACTICES

- Check headers first (`curl -sI`)
- Enumerate default admin paths
- Look for version information (CVE research)
- Test for weak credentials on admin panels
- Check config file exposure
- Monitor access logs for activity
- Document all findings

---

## COMMON VULNERABILITIES

- **Weak MySQL password** (no password or 'root')
- **Exposed phpMyAdmin** (unauthenticated access)
- **Old Apache/PHP versions** (known exploits)
- **Config file exposure** (database credentials)
- **Remote file inclusion** (PHP misconfiguration)

---

## RELATED CHEATSHEETS

- MERN-Stack-Identification.md
- Django-Framework-Identification.md
- Next.js-Framework-Identification.md

---

## REFERENCES

- Apache Documentation: https://httpd.apache.org/
- PHP Documentation: https://www.php.net/
- MySQL Documentation: https://dev.mysql.com/

---

## CHANGELOG

**v1.0 (2026-06-21)** - Initial creation from Scripts & Notes
- LAMP stack identification quick commands
- Common default paths and admin panels
- Exploitation scenarios for testing
