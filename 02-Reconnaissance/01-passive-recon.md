# Passive Reconnaissance: 2026 Complete Reference

## Data Source Hierarchy (by intelligence density)

1. Certificate Transparency logs (crt.sh, Censys, Google CT) — reveals 30-60% more subdomains than DNS alone
2. Passive DNS databases (SecurityTrails, VirusTotal, Farsight DNSDB)
3. WHOIS/RDAP registries (5 RIRs: ARIN, RIPE, APNIC, AFRINIC, LACNIC)
4. Internet-wide scan data (Shodan, Censys, FOFA, ZoomEye)
5. Web archives (Wayback Machine CDX API)
6. Search engines (Google dorks, Bing, publicwww)

## Certificate Transparency Deep Dive

CT logs are mandated by Google since 2018 - every publicly trusted cert must be submitted to >=2 logs. Every subdomain that ever got a cert is searchable forever.

### crt.sh Query
curl -s 'https://crt.sh/?q=%25.target.com&output=json' | jq -r '.[].name_value' | sort -u

### Alternative: Censys Certificates (richer filters)
censys search 'services.tls.certificates.leaf_data.subject.common_name: "target.com"'

### Alternative: Google CT API
https://transparencyreport.google.com/https/certificates

### CT Log Monitoring (continuous)
certstream-monitor --domain target.com --webhook https://your-webhook.com/ct-alerts

## ASN & Infrastructure Expansion

Stop at DNS - expand into IP space. Find the hosts nobody maps to DNS anymore.

### Find ASN
amass intel -asn 13335
curl -s 'https://api.bgpview.io/asn/AS13335/prefixes' | jq -r '.data.ipv4_prefixes[].prefix'

### Reverse DNS across range
prips 192.0.2.0/24 | dnsx -ptr -resp-only

### Shodan pivots
shodan search 'org:"Target Corp"'
shodan search 'ssl.cert.subject.cn:target.com'
shodan search 'http.favicon.hash:-123456789'  # mmh3(base64(favicon))

### Favicon Hash Pivot
Shodan uses mmh3 (MurmurHash3) of base64-encoded favicon, signed 32-bit.
FOFA uses same convention.
Grab favicon from target -> hash -> search all hosts with same favicon.

## Passive DNS (Historical)
- SecurityTrails: Historical DNS / WHOIS
- DomainTools Iris / Farsight DNSDB: Premium passive DNS
- Microsoft Defender TI: Replaces RiskIQ PassiveTotal
- VirusTotal: Relations / Resolutions tab
- Mnemonic PassiveDNS: Free academic tier
- DNSdumpster: Free, rate-limited

## Reverse WHOIS
Find other domains owned by same registrant:
amass intel -whois -d target.com
whoxy-reverse-whois (Whooxy API)
ViewDNS reverse WHOIS

## Keyless Techniques (no API keys needed)
- crt.sh (CT logs)
- BGPView (ASN data)
- DNSDumpster (DNS recon)
- Wayback CDX API (historical URLs)
- Censys free tier (limited)
- Shodan free tier (limited)

## API-Key-Required Techniques
- SecurityTrails (DNS history)
- Shodan (internet-wide scan data)
- Censys Platform (cert + host data)
- Chaos (ProjectDiscovery dataset)

## Common Pitfalls
- crt.sh returns 502 on bursty queries - add retry logic
- Hardcoding cloud provider IP ranges (they rotate - pull live JSON)
- Using MD5 of raw favicon for Shodan - it indexes mmh3(base64(bytes))
- dig ANY returns HINFO only per RFC 8482 - use per-type queries
- Missing SecurityTrails aggressive rate limiting (429s)
- Not caching results locally - repeat queries burn API quotas