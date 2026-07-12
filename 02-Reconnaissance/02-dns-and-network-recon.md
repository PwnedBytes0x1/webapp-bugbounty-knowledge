# DNS & Network Recon

## DNS Record Types
| Record | Recon Value |
|--------|-------------|
| A/AAAA | Origin server IPs, CDN detection |
| CNAME | Cloud services, takeover candidates |
| MX | Mail server identification |
| TXT | SPF/DMARC, verification tokens |
| NS | DNS provider, zone transfer testing |

## Key DNS Tools
- dig, nslookup, dnsrecon, dnsdumpster, dnsx

## Port Scanning
| Tool | Speed | Features |
|------|-------|----------|
| nmap | Moderate | NSE scripts |
| masscan | Extremely fast | Internet-scale |
| naabu | Fast | Built for bug bounty |
| rustscan | Fast | Rust + nmap integration |

## CDN / WAF Detection and Origin IP Discovery
- Historical DNS (SecurityTrails, CriminalIP)
- Mail server IPs (often not behind CDN)
- Shodan favicon hash search
- SPF records

## Cloud Provider Detection
| Provider | DNS Pattern |
|----------|-------------|
| AWS | *.cloudfront.net, *.elb.amazonaws.com |
| GCP | *.appspot.com, *.cloudfunctions.net |
| Azure | *.azurewebsites.net, *.azureedge.net |

---

> **Next**: [URL & Content Discovery](03-url-and-content-discovery.md)
