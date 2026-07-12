# Bug Bounty: The Practitioner's View

## Beyond the Hype

Bug bounty is not a get-rich-quick scheme. The hunters earning consistently treat it as a discipline: recon methodology, exploitation frameworks, and report craftsmanship. The market has matured — automated scanners hit the low-hanging fruit. Real payouts come from chain exploitation, logic flaws, and bypass techniques scanners cannot model.

## The Economics

- **Critical bugs (RCE, ATO, mass data exposure):** $5,000–$50,000+ on private programs
- **High severity (SSRF with impact, IDOR chains):** $1,000–$10,000
- **Medium/Low (reflected XSS, missing headers):** $100–$500
- **Duplicates earn $0** — speed and unique methodology are everything

The hard truth: 80% of hunters earn under $500 total. The top 1% earn 90% of rewards. The differentiator is depth — not width.

## Target Selection

- **New programs** on HackerOne/Bugcrowd (less competition, triage queues are empty)
- **VDP-to-paid transitions** — companies that recently launched bug bounties often have undiscovered criticals
- **Acquired companies** — integration debt means scope gaps and forgotten endpoints
- **Thick-client + API** targets where web-only hunters do not look

## The Stack

Modern targets are not LAMP stacks anymore. Know your target's infrastructure:
- Cloud provider (AWS/GCP/Azure) dictates SSRF impact potential
- Auth provider (Okta/Auth0/Firebase) dictates SSO attack surface
- Frontend framework (React/Vue/Angular) dictates client-side attack surface
- API protocol (REST/GraphQL/gRPC) dictates testing methodology

## Mindset Shifts

1. **Burp Repeater is not a methodology** — systematic testing requires hypothesis-driven approaches
2. **Automation finds bugs; manual chain exploitation gets paid** — automate recon, manually exploit
3. **No duplicate is ever truly novel** — if you found it, someone else can too. Speed matters
4. **The report is half the product** — a critical bug with a bad report pays less than a medium with an excellent report
