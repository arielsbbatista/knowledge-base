# Port Management Utility

**Knowledge Score:** 85/100  
**Last Updated:** June 21, 2026

---

## QUICK COMMANDS

### Find Process Using Port
```bash
ss -ltnp | grep :PORT                      # Show listening on port
lsof -i :PORT                              # List open files on port
fuser PORT/tcp                             # Find process using port
netstat -tulnp | grep PORT                 # Old method (netstat deprecated)
```

### List All Listening Ports
```bash
ss -ltn                                    # Show all listening TCP
ss -lun                                    # Show all listening UDP
lsof -i -P -n                              # Detailed port information
netstat -tulpn                             # All listening ports
```

### Get Process Details
```bash
ss -ltnp | grep :8080                      # Show what's on 8080
lsof -i :3306                              # Find MySQL process
ps aux | grep -E "\\b[0-9]+\\b" | grep 5432  # Find PostgreSQL
```

### Kill Process on Port
```bash
kill -9 $(lsof -t -i :PORT)                # Force kill
kill $(ss -ltnp | grep :PORT | awk '{print $NF}' | cut -d= -f2)  # Via ss
fuser -k PORT/tcp                          # Kill with fuser
```

### Force Restart Service
```bash
kill -9 $(lsof -ti :PORT) && service apache2 restart
kill -9 $(lsof -ti :3306) && service mysql restart
```

---

## ENUMERATION PAYLOADS

### Port Scan Results
```bash
# List all ports with process names
ss -tulpn | grep LISTEN

# Format: Protocol | LocalAddress | State | Process
# TCP    | 0.0.0.0:80 | LISTEN | 1234/apache2
```

### Identify Service on Port
```bash
ss -ltnp | grep :PROTOCOL/tcp
# Port 80 → HTTP (Apache, Nginx)
# Port 443 → HTTPS
# Port 3306 → MySQL
# Port 5432 → PostgreSQL
# Port 27017 → MongoDB
# Port 6379 → Redis
# Port 8080 → Common app port
```

### Show All Connections
```bash
ss -tnp                                    # All TCP connections
ss -unp                                    # All UDP connections
ss -tnap                                   # TCP + established
```

---

## EXPLOITATION SCENARIOS

### Scenario 1: Port Already in Use (Common Error)
```bash
# Port 8080 in use
lsof -i :8080
# Output: java 2345 user TCP *:8080 (LISTEN)

# Kill and restart app
kill -9 2345
./myapp &  # Restart
```

### Scenario 2: Multiple Processes on Same Port (Debugging)
```bash
ss -ltnp | grep :5000
# Output: 2 processes listening?
# Kill the wrong one, keep correct service
```

### Scenario 3: Unresponsive Service (Force Kill)
```bash
# Service not responding
lsof -i :80
# Get PID: 1234

# Graceful kill first
kill 1234
sleep 2

# Force kill if needed
kill -9 1234

# Restart
service apache2 start
```

### Scenario 4: Find Zombies (Hung Processes)
```bash
# Show zombie processes
ps aux | grep Z

# Find which port it was using
lsof -p ZOMBIE_PID

# Clean up
kill -9 ZOMBIE_PID
```

### Scenario 5: Service Port Hijacking Detection
```bash
# Check if service is really listening
ss -ltnp | grep :3306

# Verify it's the right process
ps aux | grep 3306_PID

# If wrong process, kill and restart legitimate service
kill -9 WRONG_PID
service mysql start
```

---

## DETECTION & VERIFICATION

### Verify Port Released After Kill
```bash
lsof -i :PORT
# No output = port released

# Try to start service again
service servicename start
```

### Check Port Status Real-Time
```bash
watch -n 1 'ss -ltnp | grep :PORT'
# Refreshes every 1 second
```

### Confirm Correct Process
```bash
ps aux | grep PID
# Verify service name matches
```

---

## ADVANCED TECHNIQUES

### Kill by Pattern
```bash
# Kill all Java processes
killall java

# Kill all Python apps
pkill -f python

# Kill specific app name
pkill -f "node myapp.js"
```

### Port Forwarding (Redirect)
```bash
# Redirect traffic from 80 to 8080
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

### Monitoring Port Activity
```bash
# Watch connections in real-time
watch -n 1 'ss -ltnp'

# Log all connections
ss -ltnp > port-baseline.txt
```

### Check for Port Reuse
```bash
# Some services use multiple ports
ss -ltnp | grep SERVICE_NAME
# May show multiple ports
```

---

## BEST PRACTICES

- Always verify correct process before killing
- Use `lsof` or `ss` to confirm (not just port number)
- Graceful kill first (`kill PID`), force kill (`kill -9`) second
- Restart service after cleanup
- Document baseline ports/processes
- Monitor for unexpected services
- Check logs before killing processes

---

## COMMON MISTAKES

❌ Killing wrong process (check twice!)
❌ Not waiting for graceful shutdown
❌ Not restarting service after kill
❌ Using old `netstat` instead of `ss`
❌ Forgetting `sudo` for privileged ports (<1024)
❌ Not checking if port actually released

---

## QUICK REFERENCE

| Command | Purpose | Example |
|---------|---------|---------|
| ss -ltnp | List all listening | ss -ltnp \\| grep :80 |
| lsof -i | List open files | lsof -i :PORT |
| kill | Graceful kill | kill 1234 |
| kill -9 | Force kill | kill -9 1234 |
| fuser | Find process | fuser PORT/tcp |
| ps aux | Process details | ps aux \\| grep PID |

---

## PORT REFERENCE

| Port | Service | Kill & Restart |
|------|---------|---------|
| 22 | SSH | service ssh restart |
| 80 | HTTP | service apache2 restart |
| 443 | HTTPS | service nginx restart |
| 3306 | MySQL | service mysql restart |
| 5432 | PostgreSQL | service postgresql restart |
| 27017 | MongoDB | service mongodb restart |
| 6379 | Redis | service redis-server restart |
| 8080 | App Port | pkill -f myapp |

---

## RELATED CHEATSHEETS

- None (standalone utility reference)

---

## REFERENCES

- ss Manual: `man ss`
- lsof Manual: `man lsof`
- kill Manual: `man kill`

---

## CHANGELOG

**v1.0 (2026-06-21)** - Initial creation from Scripts & Notes
- Quick commands for port/process management
- Common port conflict resolution
- Service restart procedures
- Troubleshooting zombie processes
