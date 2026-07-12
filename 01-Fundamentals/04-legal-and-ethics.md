# Legal & Ethical Framework

## The Authorization Chain

Bug bounty authorization flows from:
1. Program policy (explicit scope, rules of engagement)
2. Applicable law (CFAA, GDPR, Computer Misuse Act, etc.)
3. Platform terms of service (HackerOne, Bugcrowd, Immunefi)

If any of these contradict, the most restrictive applies. When in doubt, do not test.

## Scope Interpretation

### What "in scope" actually means:
- `*.target.com` does _not_ include `target.com` third-party subdomains (e.g., `target.slack.com`, `target.okta.com`)
- APIs listed in scope often include undocumented endpoints on the same base path
- Acquisitions: the acquired company's assets are typically NOT in scope unless explicitly added
- Cloud infrastructure: the target's AWS account is not in scope just because their API runs on AWS

### Rate of fire limitations:
- Most programs prohibit automated scanning without prior approval
- Some programs allow directory brute-forcing but prohibit port scanning
- Web Application scanning is generally permitted; infrastructure scanning is not

## Vulnerability Disclosure: The Practitioner's Framework

### Reporting
1. Report through the program's designated channel only
2. Include clear reproduction steps, impact analysis, and remediation suggestions
3. Never demand payment or threaten disclosure

### Response timeline expectations
- Triage: 1-5 business days (public programs) or 1-2 days (private/priority programs)
- Initial fix: 30-90 days depending on severity
- Bounty payment: Upon fix confirmation or within the program's stated timeline

### Disclosure policy
- Coordinated disclosure: typically 90-120 days after report submission
- Some programs never permit public disclosure
- Check the program policy before publishing any technical details

## GDPR & Privacy Implications

- If you access EU user data during testing, you potentially create a data processing obligation
- Never export or store real user PII
- Screenshot anonymization: blur names, email addresses, and any identifying information
- Report accidental data access to the program immediately

## International Considerations

- US programs: CFAA jurisdiction, state-level computer crime laws apply
- EU programs: GDPR breach notification requirements may apply
- Programs in Russia, China, or the Middle East: local laws may criminalize all unauthorized access regardless of program policy
- Cross-border testing: you are subject to your local laws AND the target's local laws
