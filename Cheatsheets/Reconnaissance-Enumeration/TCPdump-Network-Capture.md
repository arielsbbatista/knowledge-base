# TCPdump Network Capture

**Knowledge Score:** 87/100  
**Last Updated:** June 21, 2026

---

## QUICK COMMANDS

### Basic Capture
```bash
sudo tcpdump -i eth0                       # Listen on interface
sudo tcpdump -i any                        # All interfaces
sudo tcpdump -i eth0 -w capture.pcap       # Save to file
sudo tcpdump -r capture.pcap               # Read saved file
```

### Filter by Port
```bash
sudo tcpdump -i eth0 port 80               # Port 80
sudo tcpdump -i eth0 port 443              # Port 443
sudo tcpdump -i eth0 dst port 22           # Destination port 22
sudo tcpdump -i eth0 src port 1234         # Source port 1234
sudo tcpdump -i eth0 portrange 8000-9000   # Port range
```

### Filter by Host
```bash
sudo tcpdump -i eth0 host 192.168.1.100
sudo tcpdump -i eth0 src host 192.168.1.100
sudo tcpdump -i eth0 dst host 10.0.0.5
sudo tcpdump -i eth0 host 192.168.1.0/24
```

### Filter by Protocol
```bash
sudo tcpdump -i eth0 tcp                   # TCP only
sudo tcpdump -i eth0 udp                   # UDP only
sudo tcpdump -i eth0 icmp                  # ICMP (ping)
sudo tcpdump -i eth0 arp                   # ARP requests
sudo tcpdump -i eth0 proto 6               # Protocol 6 (TCP)
```

### Display Options
```bash
sudo tcpdump -i eth0 -A                    # ASCII payload
sudo tcpdump -i eth0 -X                    # Hex and ASCII payload
sudo tcpdump -i eth0 -n                    # No DNS resolution
sudo tcpdump -i eth0 -nn                   # No DNS/port translation
sudo tcpdump -i eth0 -v                    # Verbose output
sudo tcpdump -i eth0 -vv                   # Very verbose
sudo tcpdump -i eth0 -c 10                 # Capture 10 packets
```

---

## ENUMERATION PAYLOADS

### Cleartext Protocol Capture
```bash
# FTP (Port 21)
sudo tcpdump -i eth0 port 21 -A

# Telnet (Port 23)
sudo tcpdump -i eth0 port 23 -A

# HTTP (Port 80)
sudo tcpdump -i eth0 port 80 -A

# SMTP (Port 25)
sudo tcpdump -i eth0 port 25 -A
```

### Session Detection
```bash
# Capture all traffic from specific host
sudo tcpdump -i eth0 host 192.168.1.100 -w session.pcap

# Filter for specific connection
sudo tcpdump -i eth0 "src 192.168.1.100 and dst 10.0.0.1 and port 3306" -A
```

### DNS Resolution
```bash
# Capture DNS queries
sudo tcpdump -i eth0 port 53 -A

# Capture DNS and see queries
sudo tcpdump -i eth0 port 53 -n
```

### Extract Credentials
```bash
# HTTP Basic Auth (Base64 in Authorization header)
sudo tcpdump -i eth0 port 80 -A | grep -E "Authorization|GET|POST"

# FTP credentials
sudo tcpdump -i eth0 port 21 -A | grep -E "USER|PASS"

# Telnet credentials
sudo tcpdump -i eth0 port 23 -A | strings
```

---

## EXPLOITATION SCENARIOS

### Scenario 1: Credential Harvesting via Telnet
```bash
sudo tcpdump -i eth0 port 23 -A
# Output: Username and password in plaintext
# Extract: User credentials for lateral movement
```

### Scenario 2: HTTP Traffic Inspection
```bash
sudo tcpdump -i eth0 port 80 -A | grep -E "GET|POST|Authorization"
# Capture: API requests, form submissions, auth tokens
```

### Scenario 3: DNS Enumeration
```bash
sudo tcpdump -i eth0 port 53 -A
# Discover: Internal hostnames, subdomain patterns
# Pattern: Which IPs resolve to which names
```

### Scenario 4: MITM Cookie Stealing
```bash
# Monitor on gateway interface
sudo tcpdump -i eth0 "port 80 or port 443" -A
# Extract: Set-Cookie headers, session tokens
```

### Scenario 5: Protocol Vulnerability Exploitation
```bash
# Capture SMTP for email interception
sudo tcpdump -i eth0 port 25 -A -w smtp.pcap

# Monitor SMB for session hijacking
sudo tcpdump -i eth0 port 445 -X
```

---

## DETECTION & VERIFICATION

### Verify Cleartext Protocols Active
```bash
# Check for unencrypted traffic
sudo tcpdump -i eth0 "port 21 or port 23 or port 80" | head -20

# If output shows traffic = vulnerable
```

### Identify Active Services
```bash
# Monitor all connections
sudo tcpdump -i eth0 -n | grep ">" | awk '{print $5}' | cut -d. -f5 | sort | uniq -c
# Shows: Which ports have active connections
```

### Extract File from Capture
```bash
# Use Wireshark (GUI)
wireshark capture.pcap

# Or via CLI
tcpflow -r capture.pcap  # Extract TCP streams
```

---

## ADVANCED FILTERS

### Complex Expressions
```bash
# TCP and not SSH
sudo tcpdump -i eth0 "tcp and not port 22"

# HTTP or HTTPS
sudo tcpdump -i eth0 "port 80 or port 443"

# Exclude local traffic
sudo tcpdump -i eth0 "not (src 192.168.1.0/24 and dst 192.168.1.0/24)"

# Outbound only
sudo tcpdump -i eth0 "src 192.168.1.100"
```

### Time-Based Filters
```bash
# Capture for 60 seconds
timeout 60 sudo tcpdump -i eth0 -w output.pcap

# Capture specific time window
sudo tcpdump -i eth0 -G 3600 -w "capture-%H:%M:%S.pcap"  # Hourly rotation
```

---

## BEST PRACTICES

- Always use `-w` to save captures for analysis
- Use `-A` to see payload content (slower, use filters first)
- Filter by port/protocol first to reduce noise
- Use `grep` on ASCII output to find keywords
- Be aware of SSL/TLS (encrypted, won't show content)
- Use BPF filters efficiently (less CPU usage)
- Monitor for suspicious patterns (multiple failed logins, etc.)

---

## COMMON MISTAKES

❌ Forgetting `sudo` (tcpdump needs root)
❌ Using `-A` on all traffic (creates huge output)
❌ Not filtering by port/protocol (captures everything)
❌ Forgetting `-w` (loses data after capture)
❌ Assuming HTTPS traffic is always secure (check cert pinning)

---

## QUICK REFERENCE

| Flag | Purpose | Example |
|------|---------|---------|
| -i | Interface | sudo tcpdump -i eth0 |
| -w | Write to file | sudo tcpdump -w capture.pcap |
| -r | Read from file | sudo tcpdump -r capture.pcap |
| -A | ASCII payload | sudo tcpdump -A |
| -X | Hex+ASCII | sudo tcpdump -X |
| -n | No DNS | sudo tcpdump -n |
| -c | Count packets | sudo tcpdump -c 10 |
| -v | Verbose | sudo tcpdump -v |

---

## RELATED CHEATSHEETS

- Nmap-Reconnaissance.md (network mapping)

---

## REFERENCES

- TCPdump Manual: https://www.tcpdump.org/papers/sniffing-faq.html
- BPF Filters: https://bpf.kerneldev.com/

---

## CHANGELOG

**v1.0 (2026-06-21)** - Initial creation from Scripts & Notes
- Quick command reference for network capture
- Port and protocol filtering techniques
- Credential harvesting patterns
- Advanced filter expressions
