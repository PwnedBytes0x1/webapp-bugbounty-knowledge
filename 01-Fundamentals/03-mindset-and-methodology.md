# Mindset & Methodology

## The Bug Hunter's Mindset
1. **Think Like an Adversary** - Assume everything can be manipulated
2. **Embrace the Process** - Go deep on features rather than shallow across everything
3. **Handle Rejection** - Learn from triage feedback; re-examine closed reports

## Structured Methodology

### Phase 1: Broad Reconnaissance
Target -> Subdomains -> IPs -> Open Ports -> Web Tech -> Endpoints
80% of time should be on recon - more recon = more attack surface = more bugs

### Phase 2: Scope Analysis
- Map authentication boundaries
- Identify user roles (guest, user, admin)
- Note which endpoints accept user input

### Phase 3: Targeted Testing
Create a hypothesis-driven approach for each endpoint and parameter.

### Phase 4: Exploit Chaining
XSS + CSRF -> ATO, IDOR + Race condition -> Data manipulation

### Phase 5: Documentation
Take screenshots at every step. Record exact request/response pairs.

## Daily Workflow
| Time | Activity |
|------|----------|
| Morning | Check new programs, recon changes (30min) |
| Deep work | Manual testing on 1-2 targets (3hr) |
| Afternoon | Automated scanning, new subdomains (2hr) |
| Review | Triage feedback, research bugs (1hr) |
| Wind down | Read writeups, learn techniques (30min) |

---

> **Next**: [Legal & Ethics](04-legal-and-ethics.md)
