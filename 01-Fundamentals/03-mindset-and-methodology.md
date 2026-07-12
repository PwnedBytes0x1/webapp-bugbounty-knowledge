# Hunter Mindset & Testing Methodology

## The Hypothesis-Driven Approach

Top hunters do not randomly fuzz parameters. They form hypotheses about the application architecture and test them:

1. **What framework is this?** → fingerprints the vulnerability classes likely present
2. **What does the auth model look like?** → identifies privilege boundaries to test
3. **Where does user input travel?** → maps injection surfaces systematically
4. **What is the business logic?** → uncovers the assumptions attackers break

## Reconnaissance-First

Spend 60% of your time on recon, 30% on exploitation, 10% on reporting. The recon phase is where you discover the attack surface that competitors miss.

### Daily Routine (when actively hunting)
1. Run automated recon pipeline (subfinder + httpx + nuclei) against fresh targets
2. Review results for anomalies: unexpected status codes, unusual technologies, exposed endpoints
3. Manually explore 3-5 interesting findings from automation output
4. Deep-dive on one target: parameter discovery, logic mapping, auth boundary testing
5. Write reports for any confirmed findings

## Burp Suite Power User Setup

- **Project options → TLS → Use custom CA certificate** — avoid fingerprinting
- **Session handling rules** — auto-reauthenticate when sessions expire
- **Macros** — automate multi-step authentication flows
- **Extensions** — Autorize, AuthMatrix, Turbo Intruder, Collaborator Everywhere
- **Scope configuration** — exclude out-of-scope domains to prevent accidental testing

## Note-Taking Methodology

Maintain a hunt log with:
- Target name and scope
- Endpoints tested and results (positive and negative)
- Payloads attempted
- Screenshots organized by finding
- Time spent per target

This serves as evidence for dispute resolution and as a knowledge base for future engagements.

## Burnout Prevention

- Set a daily time limit (4-6 hours of focused testing max)
- Rotate between vulnerability classes to maintain fresh perspective
- Take breaks after large payouts — the pressure to repeat success causes sloppy testing
- Collaborate with other hunters on discord/irc — shared methodology catches more bugs
