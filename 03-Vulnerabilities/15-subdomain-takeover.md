# Subdomain Takeover

Subdomain takeover occurs when a DNS CNAME record points to a service that is no longer active, allowing an attacker to claim it.

## How It Works
1. Target has sub.domain.com CNAME -> app.herokuapp.com
2. The Heroku app is deleted
3. Attacker claims app.herokuapp.com on Heroku
4. Now sub.domain.com serves attacker's content

## Vulnerable Services
| Service | CNAME Pattern |
|---------|---------------|
| AWS S3 | *.s3.amazonaws.com |
| AWS CloudFront | *.cloudfront.net |
| Azure | *.azurewebsites.net |
| GitHub Pages | *.github.io |
| Heroku | *.herokuapp.com |
| Shopify | *.myshopify.com |
| Fastly | *.fastly.net |
| WordPress | *.wordpress.com |
| Zendesk | *.zendesk.com |

## Detection
```bash
# Find CNAME records pointing to external services
dig sub.target.com CNAME

# Automated detection
nuclei -l subs.txt -t ~/nuclei-templates/takeovers/
subzy run --targets subs.txt
subjack -w subs.txt -t 100 -ssl -v
```

## Key Indicators
- CNAME to known cloud service
- HTTP response with service-specific error
  - "NoSuchBucket" (AWS S3)
  - "There is no app configured at that hostname" (Heroku)
  - "404 - File Not Found" (GitHub Pages)

## Prevention
1. Remove unused DNS records
2. Create placeholder resources for CNAME targets
3. Monitor certificate transparency for unexpected certs
4. Regularly audit DNS records

---

> **Next**: [Prototype Pollution](16-prototype-pollution.md)
