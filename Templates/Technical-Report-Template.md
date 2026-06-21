# Technical Report Template

**Version:** 1.0.0  
**Created:** 2026-06-19  
**Based on:** Corridor CTF Full Analysis  
**Purpose:** Standardized deep-dive technical report for CTF challenges

---

## Overview

This template is used for **detailed technical analysis** beyond the main writeup. It provides extensive documentation of the vulnerability, methodology, and findings.

**Typical use case:** `analysis/FULL_ANALYSIS.md` in CTF repository

---

## Standard Sections

### 1. Executive Summary

**Purpose:** High-level overview for quick review.

**What to include:**
- Vulnerability type
- Severity rating (High/Medium/Low)
- Time to exploit
- Prerequisites
- Impact assessment

**Example structure:**
```
### Executive Summary

**Challenge:** Corridor  
**Vulnerability:** Insecure Direct Object Reference (IDOR)  
**Severity:** HIGH  
**Time to Exploit:** 26 minutes  
**Prerequisites:** Web browser, basic curl knowledge  

**Impact:** An unauthenticated attacker can access protected resources through predictable IDs, leading to information disclosure.

**Key Finding:** Boundary case (ID=0) was not protected, exposing sensitive content despite sequential ID scheme being predictable.
```

---

### 2. Vulnerability Classification

**Purpose:** Formal classification and context.

**What to include:**
- OWASP classification
- CWE reference
- CVSS score (if available)
- Attack vector
- Complexity level
- Required privileges

**Example structure:**
```
### Vulnerability Classification

**OWASP:** A01:2021 – Broken Access Control  
**CWE:** CWE-639 (Authorization through User-Controlled Key)  
**Type:** Insecure Direct Object Reference (IDOR)  

**CVSS v3.1:** 7.5 (High)
- Attack Vector: Network
- Attack Complexity: Low
- Privileges Required: None
- User Interaction: None
- Scope: Unchanged
- Confidentiality Impact: High

**Key Characteristics:**
- No authentication required
- Application logic flaw
- Predictable resource identifiers
```

---

### 3. Technical Details

**Purpose:** Deep-dive into the system configuration and vulnerability.

**What to include:**
- Service versions
- Technology stack
- Configuration details
- Security mechanisms in place (or absent)
- Enumeration results (detailed)

**Example structure:**
```
### Technical Details

#### Service Configuration

**Target:** 10.67.168.239  
**Port:** 80/tcp  
**Service:** HTTP  
**Server:** Werkzeug 2.0.3 (Python WSGI)  
**Python Version:** 3.10.2  

**Technologies Identified:**
- Flask (implied by Werkzeug)
- Custom web application
- Dynamic content generation

#### Enumeration Results

**Port Scan Results:**
```
PORT   STATE SERVICE VERSION
80/tcp open  http    Werkzeug 2.0.3 Python 3.10.2
```

**Web Directory Enumeration:**
```
[200 status] /
[No other directories found]
```

**HTTP Methods Tested:**
- GET: Functional
- POST: No endpoints found
- PUT: Not implemented
- DELETE: Not implemented

#### Application Structure

**Root Page Analysis:**
- Custom HTML generated
- Image map with 13 linked rooms
- Links to MD5-hashed endpoints
- Rooms numbered 1-13
- No visible authentication mechanism
```

---

### 4. Vulnerability Analysis

**Purpose:** Detailed explanation of why the vulnerability exists.

**What to include:**
- Root cause analysis
- Design flaw explanation
- Security assumptions that failed
- Code-level issues (if visible)

**Example structure:**
```
### Vulnerability Analysis

#### Root Cause

The application implements access control based on URL path prediction:
- IDs are MD5 hashes of sequential numbers (1-13)
- Server assumes attackers cannot guess IDs
- No server-side validation of user authorization
- No access control lists (ACLs) implemented

#### Why It's Exploitable

1. **Predictable IDs**
   - MD5 hashing sequential numbers is deterministic
   - Any attacker can compute: MD5(1), MD5(2), ..., MD5(N)
   - Not actually secret information

2. **No Access Control**
   - Server never checks: "Does this user own this room?"
   - All endpoints respond with HTTP 200
   - No authentication verification

3. **Boundary Case Oversight**
   - Developer assumed rooms numbered 1-13
   - Forgot to protect ID=0 (MD5 of "0")
   - ID=0 contains sensitive data with different content size

#### Security Assumptions That Failed

**Assumption:** "Obscurity through hashing provides security"  
**Reality:** Hashing sequential numbers is not security

**Assumption:** "If users can't discover the URL, they can't access it"  
**Reality:** URLs can be enumerated/predicted

**Assumption:** "Room 0 doesn't exist, so it won't be accessed"  
**Reality:** Boundary cases must be explicitly protected
```

---

### 5. Exploitation Methodology

**Purpose:** Complete step-by-step walkthrough with technical reasoning.

**What to include:**
- Phase breakdown
- Commands with actual output
- Decision points and reasoning
- Dead ends and lessons learned

**Example structure:**
```
### Exploitation Methodology

#### Phase 1: Pattern Identification (5 minutes)

**Objective:** Recognize the ID scheme

**Steps:**
1. Examine HTML source of root page
2. Identify MD5 hashes in image map links
3. Verify pattern with manual calculation

**Commands:**
```bash
# Verify MD5(1) matches the first endpoint
echo -n "1" | md5sum
# Output: c4ca4238a0b923820dcc509a6f75849b

# Verify in browser: hash matches first room link ✓
```

**Finding:** IDs are MD5(sequential_numbers)

**Time:** 5 minutes

---

#### Phase 2: Boundary Case Testing (10 minutes)

**Objective:** Test beyond documented ID range

**Reasoning:** Developers often forget boundary cases (0, -1, negative)

**Commands:**
```bash
# Generate MD5(0)
hash=$(echo -n "0" | md5sum | cut -d' ' -f1)
echo $hash
# Output: cfcd208495d565ef66e7dff9f98764da

# Test the endpoint
curl -s http://10.67.168.239/$hash | wc -c
# Output: 797 bytes (different from other rooms: 632 bytes)

# This indicates content! Let's examine it
curl -s http://10.67.168.239/$hash | grep -i flag
```

**Finding:** Room 0 exists and contains hidden content

**Time:** 10 minutes

---

#### Phase 3: Response Analysis (5 minutes)

**Objective:** Extract the flag from room 0

**Commands:**
```bash
curl -s http://10.67.168.239/cfcd208495d565ef66e7dff9f98764da
```

**Output:**
[Full HTML response showing the flag]

**Finding:** Flag extracted: flag{2477ef02448ad9156661ac40a6b8862e}

**Time:** 1 minute
```

---

### 6. Extended Testing

**Purpose:** Additional validation and alternative approaches.

**What to include:**
- Testing beyond minimum exploitation
- Automation attempts
- Edge cases discovered
- Alternative exploitation paths
- Scale testing (how many IDs can be accessed)

**Example structure:**
```
### Extended Testing

#### Automated Enumeration

**Script:** Python IDOR tester (see scripts/idor_test.py)

**Testing Range:** IDs from -100 to +200

**Results:**
```bash
python3 idor_test.py -u http://10.67.168.239 -r -100:200

ID -100 to -1: HTTP 404
ID 0: HTTP 200 ✓ (Flag found)
ID 1-13: HTTP 200 (Standard rooms)
ID 14-200: HTTP 404
```

**Finding:** Only ID 0 is the exception; ID 1-13 are normal rooms

---

#### Alternative Exploitation Methods

**Method 1: Direct ID enumeration (What we did)**
- Time: 26 minutes
- Effectiveness: 100%
- Difficulty: Low

**Method 2: Automated scanning**
- Time: 5 minutes
- Effectiveness: 100%
- Difficulty: Very Low

**Method 3: Timing attack**
- Unlikely to be successful (HTTP is stateless)
- Not applicable here

---

#### Data Extraction Statistics

**Accessible Resources:** 14 total (IDs 0-13)
**Response Sizes:**
- ID 0: 797 bytes (sensitive content)
- ID 1-13: 632 bytes each (normal)
- IDs 14+: 404 error

**Information Disclosed:**
- Flag in ID 0
- Room content in ID 1-13
```

---

### 7. Security Assessment

**Purpose:** Professional evaluation of the vulnerability and system.

**What to include:**
- Risk assessment
- Affected assets
- Potential attacker scenarios
- Business impact

**Example structure:**
```
### Security Assessment

#### Risk Rating: HIGH

**Justification:**
- No authentication required
- Simple to exploit
- Direct access to protected content
- Affects all users

#### Affected Assets

1. **User Data** - If this pattern extends to user profiles
2. **Sensitive Rooms** - Room 0 contains flags/sensitive info
3. **Data Integrity** - Potential for data modification (if PUT/DELETE enabled)

#### Attacker Scenarios

**Scenario 1: Reconnaissance**
- Attacker discovers MD5 pattern
- Enumerates all accessible rooms
- Maps application architecture

**Scenario 2: Data Exfiltration**
- Attacker accesses sensitive data in room 0
- Extracts flags or credentials
- Gains system knowledge

**Scenario 3: Escalation**
- If user profiles follow same pattern
- Attacker can access arbitrary user data
- Potential for privilege escalation

#### Business Impact

- **Confidentiality:** HIGH (sensitive data exposed)
- **Integrity:** MEDIUM (no modification capability in this instance)
- **Availability:** LOW (no service disruption)
```

---

### 8. Remediation & Prevention

**Purpose:** Guide for developers to fix the vulnerability.

**What to include:**
- Code-level fixes
- Architectural improvements
- Detection methods
- Network-level controls
- Testing approaches

**Example structure:**
```
### Remediation & Prevention

#### For Developers

**Fix 1: Implement Proper Access Control**
```python
@app.route('/room/<room_id>')
def get_room(room_id):
    # Current (vulnerable):
    return get_room_content(room_id)
    
    # Fixed:
    if not is_user_authorized_for_room(user_id, room_id):
        return "Unauthorized", 403
    return get_room_content(room_id)
```

**Fix 2: Use Non-Predictable IDs**
- Replace MD5(numbers) with UUIDs or random tokens
- Generate once and store in database
- Never derive from sequential inputs

**Fix 3: Implement Database-Driven Access Control**
```
Table: room_access
├── user_id
├── room_id
├── permission_level
└── created_date
```

#### For Security Teams

**Detection Strategies:**
1. Monitor for sequential ID enumeration attempts
2. Alert on 404 → 200 transitions (boundary case exploitation)
3. Watch for rapid room access from same IP
4. Log all access to sensitive rooms

**Network Controls:**
1. Rate limiting on /room/* endpoints
2. IP-based whitelisting for admin/sensitive rooms
3. WAF rules to detect ID enumeration patterns

#### Testing Approaches

**Secure Code Review:**
- Audit all endpoint handlers for access control
- Verify authorization checks before data access
- Test with different user roles

**Penetration Testing:**
- Attempt IDOR on all endpoints
- Test boundary cases (0, -1, negative)
- Attempt parameter manipulation
```

---

### 9. Appendices

**Purpose:** Detailed reference material.

#### Appendix A: Full Tool Output

```
### Full nmap Output
[Complete nmap scan result]

### Full ffuf Output
[Complete directory enumeration]

### Full HTTP Response
[Complete HTML of room 0]
```

#### Appendix B: Timeline of Discovery

| Time | Action | Result |
|------|--------|--------|
| 00:00 | Initial reconnaissance | Werkzeug 2.0.3 identified |
| 05:00 | HTML analysis | MD5 pattern recognized |
| 10:00 | Manual hash verification | Pattern confirmed |
| 15:00 | Boundary testing (ID=0) | Room 0 found |
| 20:00 | Response analysis | Different content size noted |

---

**Last Updated:** 2026-06-20  
**Status:** Synced to Obsidian from Hermes repo
