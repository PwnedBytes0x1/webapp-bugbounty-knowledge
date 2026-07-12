# DNS & Network Recon: Beyond Subdomains

## DNS Record Analysis

Every DNS record type carries recon value beyond A/CNAME:

### NS Records — Zone Transfer & Delegation Attacks
```bash
# Test zone transfer
dig axfr @ns1.target.com target.com
# Find NS record subdomains
dig ns target.com +short
# Check for split-DNS / internal-only NS records
```

### MX Records — Email Infrastructure Mapping
```bash
# Identify email provider (Google, Microsoft, custom MX)
dig mx target.com +short
# Check for SPF/DKIM/DMARC misconfigurations
dig txt target.com +short | grep -i spf
dig txt _dmarc.target.com +short
dig txt selector1._domainkey.target.com +short
```

DMARC policy of `p=none` indicates no protection — useful for phishing attack surface mapping but out of scope for most programs.

### CNAME Chain Analysis
```bash
# Track full CNAME chain
dig +short cname sub.target.com | while read cname; do dig +short cname $cname; done
# Identify third-party services in the chain
```
Service identification from CNAMEs:
- `*.cloudfront.net` — AWS CloudFront
- `*.azureedge.net` — Azure CDN
- `*.fastly.net` — Fastly
- `*.s3-website-*.amazonaws.com` — AWS S3 static site

### SOA Records — DNS Provider Fingerprinting
```bash
dig soa target.com +short
# Identify DNS provider for vulnerability assessment
# Route53 → IAM misconfiguration potential
# CloudDNS → GCP project identification
# Azure DNS → subscription mapping
```

## ASN & IP Range Discovery

Identify the target's owned IP ranges for targeted scanning:

```bash
# ASN lookup
whois -h whois.radb.net -- '-i origin AS12345' | grep -E '^route:'
# Using bgp.he.net or ipinfo.io
curl "https://ipinfo.io/AS12345/json"
```

## Reverse DNS / Passive DNS

```bash
# Find all subdomains resolving to the same IP
# Shodan: hostname:target.com
# SecurityTrails: DNS history
# RiskIQ: passive DNS
```

## CDN Detection & Bypass

```bash
# Detect CDN
curl -sI https://target.com | grep -i 'x-cache\|cf-ray\|server\|akamai\|fastly'
# Find origin IP through:
# - DNS history (SecurityTrails)
# - SSL certificate SANs (crt.sh)
# - Subdomain with direct origin (cdn-origin.target.com)
# - Email header analysis (MX IP is often near origin)
```

## Favicon Hash for Asset Discovery

```bash
# Generate and search favicon hash
curl -s https://target.com/favicon.ico | python3 -c "import mmh3, requests, sys, codecs; f=sys.stdin.buffer.read(); print(mmh3.hash(codecs.encode(f,'base64')))"
# Search Shodan for hash
# http.favicon.hash:1921827345
```

This technique finds staging environments, dev instances, and internal tools sharing the same favicon but on different domains.
