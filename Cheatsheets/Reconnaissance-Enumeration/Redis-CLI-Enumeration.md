# Redis CLI Enumeration Cheatsheet

**Knowledge Score:** 85/100

## Overview
Common commands for enumerating and gathering intelligence from Redis instances.

## Connection

```bash
redis-cli -h <host> -p <port>
```

## Basic Commands

```
PING                     # Test connectivity (responds with PONG)
INFO                     # Server information (version, config, stats)
DBSIZE                   # Number of keys in current database
KEYS *                   # List all keys (WARNING: can be slow on large datasets)
GET <key>                # Retrieve value of key
TYPE <key>               # Show data type (string, list, hash, etc)
SELECT <db_number>       # Switch database (default is 0)
```

## Configuration Enumeration (CRITICAL)

```bash
CONFIG GET dir           # Storage directory for RDB dumps
CONFIG GET dbfilename    # Database filename (default: dump.rdb)
CONFIG GET bind          # Binding address (127.0.0.1 = safe, 0.0.0.0 = exposed)
CONFIG GET protected-mode # Authentication requirement (yes/no)
CONFIG GET requirepass   # Password requirement (none/set)
CONFIG GET save          # Persistence settings
```

## Dangerous Configurations

| Configuration | Risk |
|---------------|------|
| bind 0.0.0.0 | Exposed to network (no localhost protection) |
| protected-mode no | No authentication required |
| dir = "/" or other writable | RCE via persistence possible |
| requirepass not set | Unauthenticated access |

## Data Inspection Examples

```bash
# List all keys
KEYS *

# Get value by key
GET user:1

# Inspect hash data
HGETALL user:1

# Check list contents
LRANGE queue:pending 0 -1
```

## Example from Practice

**Agent T (6379/tcp open redis):**
```
CONFIG GET dir → "/"
CONFIG GET dbfilename → "dump.rdb"
Assessment: Stored in root directory (potentially dangerous)
Exploitation: Possible RCE via persistence if combined with other vulns
```

## Related Exploitation Techniques

- RCE via .rdb persistence (overwrite system files)
- Cron job injection via persistence
- SSH key injection (if ~/.ssh/ is writable)
- Data exfiltration (SAVE and download dump.rdb)

## Defensive Practices

- Bind to 127.0.0.1 only (no network exposure)
- Enable protected-mode
- Use strong requirepass authentication
- Run with minimal privileges (never as root)
- Disable dangerous commands: CONFIG, SAVE, BGSAVE

## See Also

- [[Redis-Enumeration-Exploitation]]
- [[Service-Fingerprinting-Vulnerability-Matching]]

## References

- [Redis CLI Documentation](https://redis.io/topics/cli)
- [Redis Security Guidelines](https://redis.io/topics/security)
- OSCP Redis Exploitation Patterns
