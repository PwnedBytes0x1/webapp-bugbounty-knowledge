# Writing Effective Bug Reports: 2026 Guide

## Core Principles

### 1. Clear Impact Statement (First Line)
Bad: "Found XSS in search"
Good: "Stored XSS in search feature — when reported content is viewed by the victim, their session cookie is exfiltrated via DNS to attacker.com"

### 2. Replicable Steps
- Numbered steps, chronological order
- Exact payloads in code blocks
- Request/response pairs when relevant
- Prereqs stated upfront ("requires logged-in user")

### 3. Impact Without Inflation
- SSRF on internal API: "Can hit internal services up to the metadata endpoint"
- NOT: "Can take over AWS account" (unlikely from single SSRF)
- Let triager assess severity — your job is to demonstrate the chain, not sell

### 4. Professional Tone
- No demands, no threats, no "I could have stolen all your data"
- Describe what's possible, not what you did
- Be constructive: suggest fix or mitigation

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Vague steps | Include exact URLs, payloads, parameters |
| No impact | "XSS in search" vs "Search XSS → session theft via document.cookie exfiltration" |
| Wall of text | Use headers, bullet points, code blocks |
| No reproduction | Provide curl command or Burp saved request |
| Inflated severity | Be honest about preconditions |
| Missing environment | State browser, device, app version |

## Report Structure
1. Title (Vulnerability Type: Location)
2. Severity + CVSS
3. Summary (1-2 sentences)
4. Steps to Reproduce (numbered)
5. Impact (specific, scoped)
6. Technical Details (optional — for complex chains)
7. Proof of Concept (curl / screenshot / video)
8. Remediation Suggestion
9. References (CWE, OWASP, research)