# CTF Writeup Template

**Version:** 1.0.0  
**Created:** 2026-06-19  
**Based on:** Corridor CTF (TryHackMe)  
**Purpose:** Standardized template for CTF challenge documentation

---

## Metadata Section

```
Challenge: [Challenge Name]
Platform: [TryHackMe / HackTheBox / CTFtime / etc]
Difficulty: [Easy / Medium / Hard / Insane]
Category: [Web / Cryptography / Reverse Engineering / etc]
Time to Complete: [X minutes]
Date Completed: [YYYY-MM-DD]
Status: ✅ Rooted / 🚩 Partial / 📝 In Progress
```

---

## Structure & Sections

### 1. General Information

**Purpose:** Quick reference about the challenge.

```
### Objective

[1-2 sentences describing what the challenge asks you to do]
[What does success look like?]
[What vulnerability/concept is being tested?]
```

---

### 2. Recon

**Purpose:** Initial reconnaissance and information gathering.

**What to include:**
- Port scanning results (nmap)
- Service versions discovered
- Web application enumeration (URLs, endpoints)
- Technologies identified
- Initial observations

**Example structure:**
```
### Recon

#### Network Scanning
```bash
nmap -T4 -sV -p- <target>
```

Results:
- Port 80: HTTP (Apache 2.4.x)
- Port 443: HTTPS
- Other services

#### Web Application Analysis
- URL: http://target.com
- Technologies: Python, Flask, etc
- Initial page: [description]
```

---

### 3. Enumeration

**Purpose:** Detailed exploration and mapping.

**What to include:**
- Web directory enumeration (ffuf, gobuster)
- Endpoint mapping
- Parameter discovery
- API endpoints
- Configuration files found
- Technology-specific enumeration

**Example structure:**
```
### Enumeration

#### Web Directory Enumeration
```bash
ffuf -u http://target/FUZZ -w wordlist.txt
```

Results:
- /admin - HTTP 200
- /api - HTTP 200
- /uploads - HTTP 403

#### Endpoint Analysis
- GET /dashboard → Requires auth
- POST /upload → Accepts files
- /config.php → Sensitive data
```

---

### 4. Exploitation

**Purpose:** Vulnerability discovery and exploitation.

**What to include:**
- Vulnerability identification
- Root cause analysis
- Step-by-step exploitation
- Actual commands used (not generics)
- Reasoning for each step
- Alternative methods (if any)

**Example structure:**
```
### Exploitation

#### Vulnerability: [Type]

**Description:** [What is the vulnerability?]

**Root Cause:** [Why does it exist?]

**Impact:** [What can an attacker do?]

#### Discovery Method
1. [First step and result]
2. [Second step and result]
3. [Pattern recognition]

#### Exploitation Steps

**Step 1: [Description]**
```bash
command here
Output: [what you see]
```

**Step 2: [Description]**
```bash
command here
Output: [what you see]
```

**Key Insight:** [Why this works / what the vulnerability is]
```

---

### 5. Post Exploitation

**Purpose:** Actions after initial compromise.

**What to include:**
- Privilege escalation attempts
- Additional access gained
- Data exfiltration
- Persistence mechanisms (if applicable)
- Lateral movement
- Further enumeration as privileged user

**Example structure:**
```
### Post Exploitation

#### Privilege Escalation

[Methods tried and results]

#### Flag Discovery

[Where flags are located]
[How to extract them]
```

---

### 6. Flags

**Purpose:** Document captured flags (with security).

**IMPORTANT:** Never include flags in plain text in public repositories.

**Methods for hiding flags:**
- ROT13 encoding
- Base64 encoding
- Hex encoding
- Refer to external key file (not in repo)

**Example structure:**
```
### Flags

**User Flag (Encoded - ROT13):**
```
[encoded_flag_here]
```

**To decode:**
```bash
echo "[encoded_flag]" | tr 'a-zA-Z' 'n-za-mN-ZA-M'
```

**Root Flag (Encoded - ROT13):**
```
[encoded_flag_here]
```

**Decoding Method:** [Explain which encoding was used and why]
```

---

### 7. Tools Used

**Purpose:** Document tools with rationale.

**Format for each tool:**
- Name & Version
- Purpose
- Command used in this challenge
- Why it was useful
- When to use similar tools

**Example structure:**
```
### Tools Used

#### nmap (Network Mapping)
- **Version:** 7.x
- **Purpose:** Port and service enumeration
- **Command:** `nmap -T4 -sV -p- <target>`
- **Why Used:** Quick network reconnaissance
- **Key Finding:** Identified HTTP on port 80

#### ffuf (Web Fuzzing)
- **Version:** 2.1+
- **Purpose:** Web directory and parameter discovery
- **Command:** `ffuf -u http://target/FUZZ -w /wordlists/common.txt`
- **Why Used:** Found hidden endpoints
- **Key Finding:** Discovered /admin and /api endpoints

[Continue for each tool...]
```

---

### 8. Methodology

**Purpose:** Repeatable process for similar challenges.

**What to include:**
- General attack steps
- Decision points
- When to pivot approaches
- Tools selection rationale

**Example structure:**
```
### Methodology

#### Phase 1: Reconnaissance (5 minutes)
1. Network scan (nmap)
2. Service version identification
3. Initial web page analysis

#### Phase 2: Enumeration (10 minutes)
1. Web directory enumeration
2. Endpoint mapping
3. Parameter testing

#### Phase 3: Exploitation (X minutes)
1. Vulnerability identification
2. Exploitation attempt
3. Validation

#### Phase 4: Post-Exploitation
1. Privilege escalation
2. Flag retrieval
```

---

### 9. Lessons Learned

**Purpose:** Knowledge and improvements.

**What to include:**
- What you learned
- Misconceptions corrected
- Techniques for future use
- Tools you discovered
- Common mistakes to avoid

**Example structure:**
```
### Lessons Learned

#### Vulnerability Understanding
- IDOR works because developers assume IDs are non-sequential
- Always test boundary cases (0, -1, negative numbers)
- Response size differences indicate accessible resources

#### Testing Best Practices
- Manual testing reveals patterns automated tools might miss
- Boundary case testing is critical for ID-based vulnerabilities
- Response comparison (size, timing) reveals data leakage

#### Tools & Techniques
- MD5 hash pattern recognition speeds vulnerability discovery
- Manual curl testing provides more control than scripts
- Python automation useful for extended testing ranges
```

---

### 10. References

**Purpose:** Links to learning resources.

**What to include:**
- OWASP references
- CWE references
- CVE information (if applicable)
- Tool documentation
- Articles used during learning
- Official vulnerability databases

**Example structure:**
```
### References

#### Vulnerability Information
- **OWASP:** [Link to OWASP page]
- **CWE-639:** [Link to CWE]
- **CVSS Score:** [Score if applicable]

#### Learning Resources
- [Article about vulnerability type]
- [Tool documentation]
- [Research paper or blog]

#### Official Sources
- [Vendor advisory]
- [Exploit database]
```

---

## Quality Standards

### Content Standards
- ✅ Technical accuracy verified
- ✅ Commands are actual (not generic)
- ✅ Examples include real output
- ✅ Methodology is repeatable
- ✅ Professional tone maintained
- ✅ Markdown consistency

### Length Guidelines
- **Main Writeup:** 250-400 lines (covers key points)
- **Full Analysis:** 400-500+ lines (extended deep-dive)
- **Flags Section:** Hidden format (ROT13 or Base64)
- **Tools Section:** 1-2 lines per tool minimum

### File Structure

```
CTF-Name/
├── README.md              (Main writeup, 250-400 lines)
├── analysis/
│   └── FULL_ANALYSIS.md   (Extended analysis, 400+ lines)
├── scripts/
│   ├── exploitation.py    (Main exploit script)
│   └── automation.sh      (Testing automation)
└── references/
    └── README.md          (Tools, links, resources)
```

---

## Checklist Before Publishing

- [ ] All sections completed
- [ ] Flag hidden (ROT13/Base64)
- [ ] No credentials in any file
- [ ] Commands are verified/tested
- [ ] Methodology is clear and repeatable
- [ ] Tools documented with rationale
- [ ] Markdown syntax correct
- [ ] Links work and point to correct resources
- [ ] References section complete
- [ ] Professional tone throughout
- [ ] Git commit message descriptive
- [ ] No sensitive information exposed

---

**Last Updated:** 2026-06-20  
**Status:** Synced to Obsidian from Hermes repo
