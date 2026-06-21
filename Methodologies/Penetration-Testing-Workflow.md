# Methodology: Penetration Testing Workflow

**Knowledge Score:** 93/100

## Overview

Standard framework for conducting systematic penetration tests and CTF challenges. Ensures comprehensive approach and prevents missed attack vectors.

## Five-Phase Model

### Phase 1: Reconnaissance
**Objective:** Map target infrastructure without direct interaction

**Activities:**
- DNS enumeration (dig, nslookup, whois)
- Network mapping (traceroute)
- Passive information gathering
- Public source research

**Tools:** dig, whois, host, nslookup

**Output:** IP ranges, DNS records, approximate infrastructure

---

### Phase 2: Enumeration
**Objective:** Detailed service analysis and vulnerability discovery

**Activities:**
- Port scanning (identify services)
- Service fingerprinting (determine versions)
- Application analysis (functionality testing)
- Configuration discovery (misconfigurations)
- User/group enumeration

**Tools:** nmap, gobuster, redis-cli, curl, ssh-keyscan, nikto

**Output:** Service list, versions, potential vulnerabilities

---

### Phase 3: Hypothesis
**Objective:** Identify exploitable weaknesses

**Activities:**
- Match findings to known vulnerabilities (CVE database)
- Assess risk and impact
- Prioritize attack vectors
- Rank by likelihood of success

**Example Decision Logic:**
```
Service: PHP 8.1.0-dev identified
CVE Search: CVE-2021-21224 (User-Agent RCE)
Assessment: Highly relevant (exact version match)
Priority: HIGH (simple exploitation, immediate RCE)
```

---

### Phase 4: Validation
**Objective:** Confirm vulnerability before automated exploitation

**Activities:**
- Manual payload crafting
- Test in isolation
- Reproduce conditions
- Understand mechanics

**Example:**
```bash
curl -H "User-Agentt: zerodiumsystem('id');" http://target.thm
# Validates RCE without using automated tools
```

**Benefits:**
- Confirms actual exploitability (not false positive)
- Avoids tool incompatibilities
- Builds understanding (OSCP-critical)
- Adaptable to variations

---

### Phase 5: Exploitation
**Objective:** Execute attack and achieve objectives

**Activities:**
- Payload refinement (based on validation)
- Privilege escalation (if needed)
- Flag/objective capture
- Access consolidation

**Continuation:**
- Post-Exploitation activities
- Persistence (if applicable)
- Lateral movement (if multi-user/multi-system)

---

## Workflow Diagram

```
┌─────────────────────┐
│  Reconnaissance     │
│  (Passive gathering)│
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   Enumeration       │
│  (Active scanning)  │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│    Hypothesis       │
│ (Vulnerability ID)  │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   Validation        │
│ (Manual testing)    │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   Exploitation      │
│  (Gain access)      │
└─────────────────────┘
```

## Application to CTFs

Every CTF challenge follows this workflow:

1. **Recon:** Learn basic target info (IP, DNS)
2. **Enumeration:** Discover open ports and services
3. **Hypothesis:** Identify vulnerable services
4. **Validation:** Confirm exploitability manually
5. **Exploitation:** Retrieve flags

Missing any phase increases likelihood of missing attack vectors.

## Real-World Example: Agent T (TryHackMe)

| Phase | Activity | Result |
|-------|----------|--------|
| Recon | Basic target info | agentt.thm identified |
| Enum | Nmap standard → Redis found on expanded scan | Ports 22, 80, 6379 |
| Hypothesis | PHP 8.1.0-dev + CVE-2021-21224 | RCE vulnerability suspected |
| Validation | Manual curl test with `id` command | RCE confirmed (uid output) |
| Exploit | `cat /flag.txt` via User-Agent header | Flag retrieved, machine compromised |

## Time Allocation (Guidance)

- Reconnaissance: 10-20% (passive, less time-critical)
- Enumeration: 30-40% (foundation for success)
- Hypothesis: 10-15% (research and planning)
- Validation: 15-25% (prevents wasted exploitation)
- Exploitation: 10-20% (often quickest with proper validation)

## Key Principles

1. **Systematic → Don't assume, enumerate**
2. **Thorough → Expand scope if initial results seem sparse**
3. **Validated → Test before automated exploitation**
4. **Documented → Record findings and methodology**
5. **Iterative → Loop if vectors fail, revisit enumeration**

## Related Techniques

- [[Iterative-Enumeration]] (expanding scope in enumeration phase)
- [[Service-Fingerprinting-Vulnerability-Matching]] (hypothesis phase)
- [[Manual-Validation-Before-Exploit]] (validation phase)

## See Also

- CTF writeups documenting phase application
- OSCP examination framework

## References

- [Penetration Testing Execution Standard (PTES)](http://www.pentest-standard.org/)
- OSCP Course Material
- Professional Penetration Testing Standards
