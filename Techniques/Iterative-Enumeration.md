# Technique: Iterative Enumeration (Expanding Scope)

**Knowledge Score:** 88/100

## Principle
Initial enumeration is a starting point, not the final result. Continuously expand scope when preliminary findings seem incomplete or don't present clear attack vectors.

## Mental Model

```
Initial Recon: ✓ Complete
Review Results: Sparse or Uninteresting?
  ├─ YES → Expand Scope
  │   └─ Re-enumerate with larger scope
  └─ NO → Proceed to Hypothesis
```

## When to Expand Scope

- Default port scan reveals only common services (SSH, HTTP, DNS)
- No clear vulnerability or attack surface identified
- Service versions don't match known public vulnerabilities
- Unusual ports might host critical services
- Time and resources permit further enumeration
- Target complexity suggests hidden services

## Port Range Expansion Strategy

### Step 1: Initial Scan (Default Ports)
```bash
sudo nmap -sC -sS -oA scan1 target.thm
```

**Typical Results:** SSH (22), HTTP (80), maybe FTP (21)

### Step 2: Observation & Assessment
- Do results seem complete?
- Is there an obvious attack vector?
- Are service versions exploitable?

**If sparse or uninteresting → Proceed to Step 3**

### Step 3: Expanded Scan
```bash
sudo nmap -sC -sS -p1-9000 -oA scan2 target.thm
```

**Common Discoveries:**
- Database services: Redis (6379), MongoDB (27017), MySQL (3306)
- Alternative HTTP ports: 8080, 8888, 3000
- Custom application services
- Development/debugging services

### Step 4: Compare & Analyze
- Which new services appeared?
- Do any have known vulnerabilities?
- Does this change attack strategy?

### Step 5: Decision
- Continue with focused enumeration of new services
- Further port expansion if justified (time permitting)
- Or proceed to hypothesis with expanded knowledge

## Real-World Example: Agent T

| Phase | Scan | Result | Decision |
|-------|------|--------|----------|
| Initial | Default ports | SSH, HTTP only | "Too simple" |
| Expand | -p1-9000 | Redis discovered on 6379 | Attack vector identified |
| Outcome | | RCE vulnerability found | Machine compromised |

## Avoiding Over-Enumeration

- Set time limits (don't scan indefinitely)
- Use port-specific scanning (1000 ports → 1-9000 → full if needed)
- Prioritize services based on knowledge (database ports more interesting than random high ports)
- Full scan (1-65535) only if justified by initial results

## Complementary Techniques

Works well with:
- [[Service-Fingerprinting-Vulnerability-Matching]] (once services found)
- [[Nmap-Reconnaissance]] (tool-specific knowledge)
- [[Penetration-Testing-Workflow]] (positioned in Enumeration phase)

## See Also

- [[Manual-Validation-Before-Exploit]]
- [[Nmap-Reconnaissance]]

## References

- OSCP Enumeration Methodology
- Penetration Testing Execution Standard (PTES)
