# Report Writing

A well-written report is as important as the vulnerability itself. Clear, professional reports get accepted faster, pay higher bounties, and build your reputation.

## Report Structure

### 1. Title
Clear, descriptive, and includes the vulnerability type.
- **Bad**: "Bug on login page"
- **Good**: "IDOR on GET /api/v1/users/{id}/profile Allows Unauthorized Access to Any User's Personal Data"

### 2. Summary (1-2 sentences)
What the vulnerability is and where it occurs.

### 3. Steps to Reproduce
Numbered, sequential steps that anyone can follow.

```
1. Log in as user A
2. Navigate to /api/v1/users/1234/profile
3. Change the ID to 5678
4. Observe that user A now sees user B's profile data
```

### 4. Proof of Concept (PoC)
- HTTP request/response pairs
- Screenshots (annotated)
- Video (for complex bugs)
- Curl commands or Python scripts

### 5. Impact
Explain the real-world consequences.
- **Low**: Information disclosure (non-sensitive)
- **Medium**: Access to user data
- **High**: Account takeover, data modification
- **Critical**: RCE, full system compromise

### 6. Affected Endpoints
List all URLs, parameters, and methods affected.

### 7. Remediation Suggestions
- "Implement server-side access control checks"
- "Use parameterized queries"
- "Validate file types server-side"

### 8. Technical Details (optional)
For complex vulnerabilities, include code snippets, full exploit scripts, or detailed technical analysis.

## Writing Tips

- **Be concise** — Program triagers read hundreds of reports
- **Be specific** — Include exact URLs, parameters, request bodies
- **Be professional** — No demands, threats, or emotional language
- **Redact PII** — Blur personal data in screenshots
- **Provide working PoCs** — A report with a clear PoC is accepted faster

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Vague title | Include vulnerability type + location |
| No screenshot | Add annotated screenshots |
| Wall of text | Use bullet points, short paragraphs |
| Missing steps | Write numbered reproduce steps |
| No impact | Explain what an attacker could do |
| Poor English | Use Grammarly or ask for review |

## Report Templates

### Critical/High Severity Template
```markdown
**Title:** [Vulnerability Type] in [Endpoint]

**Severity:** Critical/High

**Summary:**
[1-2 sentence description]

**Steps to Reproduce:**
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Proof of Concept:**
[Request/Response or Screenshot]

**Impact:**
[What an attacker can achieve]

**Remediation:**
[Suggested fix]
```

## Tools for Reporting

| Tool | Purpose |
|------|---------|
| **Hackmd/Markdown** | Draft and format reports |
| **Flameshot/ShareX** | Annotated screenshots |
| **Peek/ScreenFlow** | Screen recording (macOS/Linux) |
| **Burp Suite** | Export requests/responses |
| **Obsidian/Notion** | Track findings during testing |

---

> **Next**: [CVSS Scoring](02-cvss-scoring.md)
