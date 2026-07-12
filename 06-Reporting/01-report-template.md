# Bug Bounty Report Template

## Summary
[One-line description of the vulnerability]

## Title
Brief, descriptive title (e.g., "Stored XSS in User Profile Bio")

## Severity
Critical / High / Medium / Low / Informational

## CVSS Score
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H (9.8 Critical)

## Steps to Reproduce
1. Navigate to [URL]
2. Input [payload] in [field]
3. Observe [result]
4. Proof of impact: [evidence]

## Proof of Concept
```http
[HTTP request/response demonstrating the issue]
```
[Or screenshot/video link]

## Impact
- [What an attacker can do with this vulnerability]
- [Business impact]

## Remediation
- [Specific fix recommendation]
- [Code example if applicable]

## Affected Assets
- https://target.com/some/endpoint
- API: POST /api/vulnerable-endpoint

## References
- [OWASP link]
- [CVE/Mitre link]
- [Related writeup]