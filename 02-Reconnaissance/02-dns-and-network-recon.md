# DNS & Network Reconnaissance

## DNS Record Analysis

```bash
# Full DNS record retrieval
dnsx -d example.com -a -aaaa -cname -ns -mx -soa -txt -caa -resp -silent

# Zone Transfer Test
dig axfr @ns1.example.com example.com
# If successful, you have the entire DNS database
# Common misconfig: zone transfer allowed from ANY

# DNSSEC - zone enumeration via NSEC/NSEC3
ldns-walk @ns1.example.com example.com

# DNS cache snooping
dig +norecurse +timeout=1 @8.8.8.8 admin.example.com
# Non-recursive resolution from a resolver reveals cached entries
```

### DNS Record Insight
- **CNAME**: Can point to service that may be vulnerable to takeover
- **MX**: Email provider identification (Google Workspace, O365, self-hosted)
- **TXT**: SPF, DKIM, DMARC reveal email security posture; also verification strings
- **SOA**: Admin email, refresh/retry timing (older infrastructure indicator)
- **NS**: Nameserver hosting (Cloudflare, AWS Route53, self-hosted)
- **CAA**: Authorized CA issuers (which CAs can issue certs)
- **SRV**: Service discovery (_sip, _ldap, _xmpp)
- **PTR**: Reverse DNS (pivot from IP to hostname)

## SPF/DMARC Analysis

```bash
# SPF record
dig TXT example.com | grep -i spf
# Parse: include, ip4, ip6, a, mx, redirect
# Weak SPF: v=spf1 +all  # Anyone can send as @example.com
# Better: v=spf1 include:_spf.google.com ~all

# DMARC
dig TXT _dmarc.example.com
# p=none: no enforcement, p=quarantine: spam folder, p=reject: block
# rua/ruf: aggregate/forensic report emails

# DKIM selector enumeration
dig TXT default._domainkey.example.com
dig TXT selector1._domainkey.example.com
dig TXT google._domainkey.example.com
```

## Reverse DNS & PTR Analysis

```bash
# PTR records for discovered IPs
cat all_ips.txt | dnsx -ptr -resp-only -silent

# Often reveals:
# system hostnames that don't have forward DNS records
# infrastructure names (web-01.prod, db-02.staging)
# Cloud provider internal naming conventions
```

## DNS History & Changes

```bash
# SecurityTrails DNS history (API required)
curl -s "https://api.securitytrails.com/v1/history/example.com/dns/a" -H "APIKEY: $KEY"

# ViewDNS historical data
# https://viewdns.info/iphistory/?domain=example.com

# WhoisXML API domain history
# https://www.whoisxmlapi.com/
```

## Port Scanning

### naabu (Fast, ProjectDiscovery)
```bash
# Top 1000 ports
naabu -l resolved_hosts.txt -top-ports 1000 -silent -o ports.txt

# Full scan (1-65535, slow but thorough)
naabu -l resolved_hosts.txt -p - -rate 1000

# Service identification (nmap integration)
naabu -l hosts.txt -p 80,443,8080,8443 -nmap -o naabu_nmap.xml
```

### nmap (Deep)
```bash
# Service version detection
nmap -sV -p 80,443,8080,8443 -iL hosts.txt -oA service_scan

# Script scanning
nmap -sC -p 80,443 -iL hosts.txt -oA script_scan

# Full TCP scan (stealth U for UDP)
nmap -sT -sV -sC -p- -T4 -iL limited_hosts.txt -oA full_scan

# HTTP enumeration scripts
nmap --script http-enum,http-title,http-server-header,http-headers,http-methods,http-webdav-scan -p 80,443 -iL hosts.txt
```

### Masscan (Ultra-Fast)
```bash
# Scan entire subnets in seconds
masscan 10.0.0.0/8 -p80,443,8080 --rate=10000 -oL masscan.txt
```

## Port Analysis Strategy
- 22: SSH (check for weak auth, known CVEs)
- 25/465/587: SMTP/SMTPS (open relay, spoofing)
- 53: DNS (zone transfer, cache poisoning)
- 80/443: HTTP/HTTPS (primary web app)
- 389/636: LDAP/LDAPS (anonymous bind)
- 1433/3306/5432/6379: Database (exposed to internet)
- 8080/8443: Alternate web (admin panels, APIs)
- 9200: Elasticsearch (unauthenticated)
- 27017: MongoDB (unauthenticated)

## CDN Detection & Bypass

```bash
# Detect CDN
dig +short example.com
# If multiple IPs resolving to same hostname -> likely CDN

# Historical DNS for origin IP
# SecurityTrails, Censys, Shodan historical DNS
# Certificate transparency logs (subject IP != CDN IP)

# Origin IP bypass methods
# - MX records (mail server IP)
# - TXT records (old verification IP)
# - Subsidiary subdomains without CDN
# - Shodan search: org:example.com + http.title:"example"
# - Google dork: site:example.com -www
# - F5 BigIP cookie decoding
# - Censys search for SSL certs with same subject
```
