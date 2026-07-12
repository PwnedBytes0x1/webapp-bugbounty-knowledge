# Severity Matrix (CVSS 3.1)

| Vector | Abbreviation | Values |
|--------|--------------|--------|
| Attack Vector | AV | Network[N], Adjacent[A], Local[L], Physical[P] |
| Attack Complexity | AC | Low[L], High[H] |
| Privileges Required | PR | None[N], Low[L], High[H] |
| User Interaction | UI | None[N], Required[R] |
| Scope | S | Unchanged[U], Changed[C] |
| Confidentiality | C | None[N], Low[L], High[H] |
| Integrity | I | None[N], Low[L], High[H] |
| Availability | A | None[N], Low[L], High[H] |

### Bug Bounty Severity Examples

| Vuln | CVSS | Severity |
|------|------|----------|
| RCE on web server | 9.8 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H) | Critical |
| Stored XSS | 6.1 (AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N) | Medium/High |
| SSRF with metadata | 8.8 (AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H) | Critical/High |
| Reflected XSS | 6.1 (AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N) | Medium |
| Missing HSTS | 3.7 (AV:N/AC:H/PR:N/UI:R/S:U/C:L/I:N/A:N) | Low |