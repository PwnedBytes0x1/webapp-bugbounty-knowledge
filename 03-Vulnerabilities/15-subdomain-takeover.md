# Subdomain Takeover: 2026 Reference

## Vulnerable Services

| Service | CNAME Pattern | Detection |
|---------|--------------|-----------|
| AWS S3 | `<bucket>.s3.amazonaws.com` | 404 NoSuchBucket |
| AWS CloudFront | `<hash>.cloudfront.net` | 404, 403 |
| Azure | `*.azurewebsites.net`, `*.trafficmanager.net` | 404, NXDOMAIN |
| GitHub Pages | `<user>.github.io` | 404 |
| GitLab | `<group>.gitlab.io` | 404 |
| Heroku | `*.herokuapp.com` | No such app |
| Shopify | `*.myshopify.com` | 404 |
| Wordpress | `*.wordpress.com` | Domain not configured |
| Fastly | `*.fastly.net` | 404, 503 |
| Pantheon | `*.pantheon.io` | 404 |
| Readme.io | `*.readme.io` | Project not found |
| Zendesk | `*.zendesk.com` | Help center not found |
| DigitalOcean | `*.ondigitalocean.app` | 404 |
| Vercel | `*.vercel.app` | 404: Not Found |
| Netlify | `*.netlify.app` | Not Found |
| Firebase | `*.firebaseapp.com` | 404 |

## Detection Pipeline
```bash
subfinder -d target.com | dnsx -cname -resp-only | tee cnames.txt
```

CNAMEs pointing to unclaimed cloud services = takeover candidates.

## Validation
```bash
# DNS resolution confirms CNAME exists but target service returns 404
dig CNAME sub.target.com +short
# → bucket.s3.amazonaws.com
curl -I http://sub.target.com
# → 404 NoSuchBucket
```

## Exploitation
1. Create matching bucket/service in the cloud provider
2. Upload content (phishing page, JS file for XSS, etc.)
3. If target returns 404 and you can register that resource name → confirmed takeover
4. Impact depends on domain: `cdn.target.com` → XSS target-wide, `login.target.com` → credential harvesting

## Automation
- **nuclei -t ~/nuclei-templates/takeovers/** — automated takeover detection
- **SubOver** — subdomain takeover detection tool
- **Can I Takeover XYZ?** — reference for CNAME takeover fingerprints

## Defensive Checklist
1. Audit all external CNAME records regularly
2. Remove unused DNS records pointing to cloud services
3. Use `_acme-challenge` TXT records for provisioning only
4. Monitor certificate transparency logs for unexpected subdomain certs
5. Set up notifications for 4xx/5xx on subdomains