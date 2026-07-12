# Bug Bounty Report Template

---
**Title**: [Short descriptive title: Vulnerability Type in Feature]

**Severity**: [Critical / High / Medium / Low / Informational]
**CVSS Score**: [e.g. 7.5 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N)]
**Bounty**: [Optional — disclosed bounty amount]
---

## Summary
[1-2 sentence description of the vulnerability]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Impact
[What an attacker can achieve. Be specific but don't inflate.]

## Technical Details
[Deep explanation — root cause, attack chain, affected endpoints]

## Proof of Concept
[curl command, request/response block, or screenshot]

```http
POST /api/endpoint HTTP/1.1
Host: target.com
...

(payload)
```

## Remediation Suggestion
[Actionable fix recommendation]

## References
- [OWASP / CWE / Relevant research]
