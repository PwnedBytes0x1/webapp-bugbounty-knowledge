# Report Writing Guide

## Principles

### Clarity
- Write for the triager, not other hackers
- Assume minimal context about the application
- Be specific: "The XSS occurs in the bio field at /profile/update when the value contains <script>alert(1)</script>"

### Reproducibility
- Every step must be exact and repeatable
- Include complete curl commands, not partial snippets
- Screenshots/videos should show the browser URL bar

### Impact
- Explain business impact, not just technical
- "An attacker can steal admin sessions, leading to account takeover" (not just "XSS fires")

## Structure

### Good Title Examples
- "Stored XSS in user profile bio via HTML injection"
- "IDOR in GET /api/orders allows reading any user's order details"
- "Authentication bypass via JWT none algorithm on login endpoint"

### Bad Title Examples
- "XSS bug"
- "Security issue"
- "IDOR"

## CVSS Justification
Always include CVSS v3.1 vector and score. Justify each metric:
- AV:N (network) - remotely exploitable
- AC:L (low complexity) - no special conditions
- PR:N (no privileges) - unauthenticated
- UI:N (no user interaction) - no victim action required
- S:U (unchanged) - or S:C if scope changed
- C:H/I:H/A:H - full impact

## Common Mistakes
- Including API keys/tokens in POC
- Not clearing cookies/session before retesting
- Assuming the triager has the same tools
- Overstating impact without evidence