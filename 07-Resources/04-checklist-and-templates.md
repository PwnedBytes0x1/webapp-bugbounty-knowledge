# Bug Bounty Checklist & Report Templates

## Pre-Hunt Checklist

- [ ] Read program scope, rules, and in-scope/out-of-scope
- [ ] Read past disclosed reports (HackerOne Hacktivity, Bugcrowd disclosures)
- [ ] Identify target technologies (Wappalyzer, BuiltWith)
- [ ] Set up Burp project + scope highlighting
- [ ] Configure Collaborator client / Interact.sh
- [ ] Ensure proxy is running (Burp, Caido, ZAP)
- [ ] Check for authentication: guest + low-priv + admin
- [ ] Take baseline screenshot of target (for Hall of Fame)
- [ ] Review common vulns for this tech stack

## Target Reconnaissance

- [ ] Passive recon (subfinder, amass, Shodan, Censys)
- [ ] Active recon (httpx, naabu, gospider, katana)
- [ ] Technology fingerprinting (wappalyzer, whatweb, nuclei tech-detect)
- [ ] JS analysis (linkfinder, SecretFinder, jsleak)
- [ ] Endpoint discovery (ffuf, katana, gospider)
- [ ] Parameter discovery (Param Miner, ffuf)
- [ ] Third-party integrations (SSO, analytics, CDN)

## Vulnerability Testing Flow

1. **Information Disclosure** -> Check exposed .git, .env, backup files
2. **IDOR** -> Enumerate IDs, UUIDs, check horizontal + vertical
3. **XSS** -> Check all input fields, URL params, headers
4. **SQLi** -> Check all DB-interacting endpoints
5. **SSRF** -> Check URL/redirect parameters, file imports
6. **SSTI** -> Template engines, error messages
7. **XXE** -> XML endpoints, file upload parsers
8. **Authentication** -> Rate limiting, JWT attacks, OAuth misconfig
9. **Authorization** -> Role/privilege escalation checks
10. **Business Logic** -> Workflow bypass, race conditions, edge cases

## Report Template

```markdown
## Summary
[Brief description]

## Target
[URL/Endpoint]

## Vulnerability Type
[OWASP category]

## Severity
[CVSS 3.1 vector] -> [Score]

## Description
[Detailed explanation]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Proof of Concept
[Code/Request/Screenshot]

## Impact
[What attacker can achieve]

## Remediation Suggestion
[Specific fix]

## References
[Links to similar vulnerabilities / writeups]
```

## Post-Submission Checklist

- [ ] Added to personal bug tracker
- [ ] Screenshots organized in per-program folder
- [ ] Notes saved for future reference
- [ ] Retested after fix
- [ ] Updated personal checklist with new findings
