# Bug Bounty Essentials

## Program Types
- **Public**: Open to all, listed on HackerOne/Bugcrowd
- **Private**: Invite-only, smaller scope
- **VDP**: Vulnerability Disclosure Program (no reward)
- **P4P**: Pay-for-Play (entry fee + rewards)

## Scope Types
- *.target.com (all subdomains)
- Specific subdomains (api.target.com, app.target.com)
- Wildcard with exclusions (not *.admin.target.com)
- Mobile apps (Android APK, iOS IPA)
- Source code on GitHub

## Banned/Out-of-Scope
- DoS/DDoS attacks
- Physical attacks
- Social engineering
- Spam/phishing
- Automated scanning without permission

## Disclosure Timeline
- Immediate: Report confirmed
- Triage: Reviewed by SOC (1-5 days)
- Validation: Reproduced by team (1-14 days)
- Reward: Bounty paid
- Fix: Remediation deployed
- Public Disclosure: Usually 30-90 days after fix

## Response Types
| Label | Meaning | Action |
|-------|---------|--------|
| Triaged | Valid issue | Awaiting reward |
| N/A | Not applicable | Not a security bug |
| Informative | Acknowledged | No action/reward |
| Duplicate | Already reported | No bounty (first reporter gets it) |
| Out of Scope | Outside program scope | Report elsewhere |
| Spam | Low quality | Account may be banned |

## Rate Limits & Safety
- Burp Intruder: 1 req/sec
- ffuf: -rate 50-100
- Always check program's rate limit policy
- Use -pause and -delay flags
- Stop on 429/503 responses