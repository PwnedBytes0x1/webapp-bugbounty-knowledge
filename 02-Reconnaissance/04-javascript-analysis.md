# JavaScript Analysis

JS files contain API endpoints, auth logic, hardcoded secrets, and business logic.

## What to Look For
- API endpoints and routes (/api/v1/users, /graphql, /admin)
- Secrets: API keys, JWTs, OAuth tokens, Firebase config
- Application logic: auth checks, validation rules, feature flags

## Tools
| Tool | Purpose |
|------|---------|
| LinkFinder | Extract endpoints from JS |
| SecretFinder | API key and token detection |
| JSParser | Tornado-based JS URL parsing |
| xnLinkFinder | Burp-friendly link extraction |
| SourceMapper | Reverse engineer source maps |
| SubJS | Subdomain extraction from JS |

## Methodology
```bash
# Collect JS URLs
cat all_urls.txt | grep -E "\\.js(\\?|$)" | sort -u > js_files.txt

# Download and analyze
cat js_files.txt | xargs -I{} curl -s {} > all.js
python3 linkfinder.py -i all.js -o cli
python3 SecretFinder.py -i all.js -o cli

# Source map analysis
# Check if .map files exist for each JS file
```

## Key Findings
| Finding | Impact |
|---------|--------|
| Firebase URL + misconfigured rules | Full DB access (critical) |
| AWS keys with S3 permissions | Data breach (critical) |
| Hardcoded JWT / API keys | ATO (high) |
| Internal API endpoints | SSRF, IDOR (high) |
| Debug endpoints | Info disclosure (medium) |

---

> **Next**: [GitHub Recon](05-github-recon.md)
