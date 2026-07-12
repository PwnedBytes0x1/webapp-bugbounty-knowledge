# Scanning Tools: 2026 Reference

## Vulnerability Scanners

### Nuclei (ProjectDiscovery)
Template-based scanner. 8000+ templates in default install.
- **nuclei -u https://target.com -t ~/nuclei-templates/** — scan all templates
- **nuclei -l live.txt -t cves/** — CVE-only scan
- **nuclei -t exposures/configs/ -t exposures/tokens/** — secret/config scanning
- **-severity critical,high** — Prioritize impact
- **-headless** — Browser-based template execution
- **-stats** — Progress tracking
- Custom templates in YAML format with matchers (status, regex, dsl, word, binary)

### Katana (ProjectDiscovery)
Crawler for endpoint discovery.
- **katana -u https://target.com -d 3 -silent -o endpoints.txt**
- **-js-crawl**: Parse JavaScript for endpoints
- **-known-files all**: robots.txt, sitemap.xml, security.txt
- **-aff**: Automatic form filling
- **-headless**: Browser-based crawling (JS rendering)

### Uncover (ProjectDiscovery)
Shodan/Censys/Fofa/Publicwww search aggregator.
- **uncover -q 'ssl.cert.subject.cn:target.com' -e shodan,censys**

### Scanner Comparison

| Scanner | Type | Best For |
|---------|------|----------|
| Nuclei | Template-based | CVE, misconfig, exposure |
| Nikto | Signature-based | Legacy web servers |
| WPScan | CMS-specific | WordPress |
| OpenVAS | Full suite | Comprehensive |
| Nexpose | Commercial | Compliance-focused |