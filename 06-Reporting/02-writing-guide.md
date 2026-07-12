# Report Writing Guide

## Quick Reference

### Title Format
`[Vulnerability Type] in [Endpoint/Feature] leading to [Impact]`

Examples:
- "Stored XSS in profile comments leading to session theft"
- "IDOR in /api/orders/{id} exposing other users' payment data"
- "SSRF in PDF generator accessing EC2 metadata"

### Severity Guidelines
| Severity | Typical Impact |
|----------|---------------|
| Critical | RCE, auth bypass → account takeover, SSRF → metadata credentials |
| High | Stored XSS, SQLi with data exfiltration, IDOR with PII |
| Medium | Reflected XSS, Blind SQLi, open redirect |
| Low | Information disclosure, missing security headers |
| Informational | CSP misconfig, fingerprint exposure |

### What Triagers Look For
1. **Uniqueness**: Is this a known/resolved issue? (CVE check)
2. **Clarity**: Can they reproduce it in under 30 seconds?
3. **Impact**: Does this cross the program's minimum severity?
4. **Proof**: Is there a clear demonstration?
5. **Scope**: Is this in scope? (always double-check)