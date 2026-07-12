# Writing Effective Bug Bounty Reports

## The Report Structure

### 1. Title
- Clear, specific, action-oriented
- Bad: "XSS vulnerability"
- Good: "Stored XSS in user profile display name field via unsanitized markdown input"

### 2. Vulnerability Type & Severity
- OWASP classification (e.g., A03:2021 - Injection)
- CVSS 3.1 score with vector string
- Example: `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N` (Reflected XSS)

### 3. Description
- What the vulnerability is
- Where it occurs (endpoint, parameter, component)
- Business impact (what an attacker can do)

### 4. Steps to Reproduce
Numbered, precise, copy/paste ready:
```
1. Navigate to https://target.com/profile
2. Set display name to: <img src=x onerror=alert(1)>
3. Click "Save"
4. Navigate to https://target.com/users/admin
5. Observe alert(1) execution
```

### 5. Proof of Concept (PoC)
- Minimal, clean, non-destructive
- Include HTTP request/response if relevant
- Collaborator/OOB proof (DNS/HTTP interaction)
- Screenshot (annotated, showing URL bar + payload)

### 6. Impact
- What attacker can achieve given full exploitation chain
- Realistic scenario, not worst-case fantasy
- Example: "Attacker can steal session cookies of all users viewing the profile page"

### 7. Remediation
- Specific fix (not generic advice)
- Bad: "Sanitize user input"
- Good: "Use `DOMPurify.sanitize()` on the display name field before rendering. Add Content-Type: application/json response header to prevent XSS via other vectors."

### 8. Weakness Enumeration
- CWE ID (e.g., CWE-79: Improper Neutralization of Input)
- Bounty programs use CWEs for triage categorization

## Common Mistakes

| Mistake | Better Approach |
|---------|----------------|
| Vague title | Specific endpoint + technique |
| No business impact | Explain what attacker actually gains |
| Complex PoC | Minimal, clean, replicable |
| No remediation suggestion | Specific, actionable fix |
| Wall of text | Bullet points, screenshots, code blocks |

## Severity Inflation Pitfalls
- Inflating severity (e.g., calling reflected self-XSS "High") damages credibility
- Be honest: if impact is low, say so
- Triage teams appreciate accuracy -> faster bounties
