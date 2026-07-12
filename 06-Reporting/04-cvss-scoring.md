# CVSS Scoring: Quick Reference

## CVSS 3.1 Calculator
https://www.first.org/cvss/calculator/3.1

### Base Score Ranges
| Severity | Score Range |
|----------|-------------|
| Critical | 9.0 - 10.0 |
| High | 7.0 - 8.9 |
| Medium | 4.0 - 6.9 |
| Low | 0.1 - 3.9 |
| None | 0.0 |

### Bug Bounty Mapping
- Critical: RCE, auth bypass, mass data exposure
- High: SSRF with metadata, stored XSS, SQLi w/ data
- Medium: Reflected XSS, blind SQLi, open redirect
- Low: Missing headers, INFO disclosure
- None: Config/best practice observations (usually)

### Temporal & Environmental
Bug bounty typically uses Base Score only (no temporal/environmental modifiers). Only modify for specific program guidance.