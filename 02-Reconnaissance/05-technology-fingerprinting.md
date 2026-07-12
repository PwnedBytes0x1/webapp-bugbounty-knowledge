# Technology Fingerprinting: 2026 Reference

## Framework Detection from HTTP Headers

X-Powered-By: Express -> Node.js/Express
X-Runtime: 0.01234 -> Ruby on Rails
X-AspNet-Version: 4.0.30319 -> ASP.NET
X-Application-Context: app:prod:8080 -> Spring Boot
Server: uvicorn -> Python ASGI (FastAPI)

## WAF Detection (15+ Vendors)

### Detection Vectors
1. HTTP response headers (160+ vendor rules)
2. Block page content (distinctive per vendor)
3. Cookies (naming conventions)
4. DNS CNAME chain
5. Behavioral patterns (rate limiting style)
6. TLS fingerprint (JA3/JA4)

### Header Detection
| WAF | Headers |
|-----|---------|
| Cloudflare | CF-Ray, cf-cache-status |
| Akamai | x-akamai-transformed, akamai-grn |
| CloudFront | x-amz-cf-id, x-amz-cf-pop |
| AWS WAF | x-amzn-ErrorType |
| Imperva | x-iinfo, x-cdn |
| Sucuri | x-sucuri-id |
| F5 BIG-IP | TS cookie (TS+hex) |
| Barracuda | barra_counter_session |

### Tools
- wafw00f: Passive detection (vendor ID only)
- wafprobe: Per-axis mutation probes (identifies WHY blocked)
- WafRift: 160+ WAF rule catalog, JA3 rotation
- BypassBurrito: LLM-powered bypass generation

### Advanced WAF Fingerprinting (WafRift)
Detects on 4 axes:
1. HTTP response headers + body
2. DNS CNAME chain
3. Reverse-DNS (PTR) on leaf IP
4. BGP origin-ASN lookup

### Layer Tagging
- sensor: Profiling but not blocking
- challenge: JS interstitial (Cloudflare Managed Challenge)
- http: Block page
- rate-limit: 429/503

## JA3/JA4 TLS Fingerprinting

JA3: MD5 of TLS ClientHello (ciphers + extensions + curves)
JA4: Granular version with protocol, SNI, ALPN

Tools: uTLS (Go), tls-client (Node.js), curl-impersonate

## WAF Bypass Techniques

### Encoding-Based
- URL encoding (double, triple, overlong UTF-8)
- Unicode normalization bypass
- Mixed encoding per character

### Protocol-Level
- HTTP/2 downgrade desync (Kettle 2021)
- Transfer-Encoding obfuscation
- Content-Type confusion (JSON vs XML vs form)

### Body Padding
Cloud WAFs inspect only leading bytes (Cloudflare Pro 8KB, AWS WAF 16KB). Pad past the inspection window.

### Header Smuggling
X-Forwarded-For: 127.0.0.1 (IP-based ACL bypass)
X-Original-URL: /admin (path-based bypass)
X-Rewrite-URL: /admin

### TLS Fingerprint Rotation
Rotate JA3/JA4 per request with fresh TCP source port.

### Automated Bypass Tools
- BypassBurrito: LLM-generates mutations, evolves payloads
- WafRift: Grammar + encoding mutation, evolutionary search, per-WAF gene bank
- wafprobe: Axis mutation probes

## CDN Detection via DNS

Tool: unearth (17 parallel techniques)
Finds origin IPs behind CDNs using:
- CT fingerprint pivots
- DNS history
- SPF/MX analysis
- Censys IPv6 (catches dual-stack origin leaks)
- Host-header bypass
- Favicon hash pivot
- ASN sweep

Keyless passives: ct_fingerprint, crtsh, spf_mx, subdomain_enum, split_dns
Keyed passives: censys_cert, shodan_cert, dns_history
Active: host_header, banner_grab, asn_sweep