# Reconnaissance Tools: 2026 Reference

## Passive Tools

| Tool | Sources | API Key Required | Output |
|------|---------|-----------------|--------|
| subfinder | 30+ (PD ecosystem) | Some sources | Subdomains |
| amass | Deepest coverage | Free tier available | Subdomains + ASN |
| assetfinder | Minimal, fast | No | Subdomains |
| chaoss | PD curated dataset | Yes (free) | Subdomains |
| httpx | - | No | Live hosts, titles, tech |
| interactsh | PD | No | OOB collaborator |

## Active Tools

### DNS
- **puredns**: Wildcard-filtered resolution with DoH support
- **shuffledns**: Mass DNS resolver for brute-forced subs
- **dnsx**: Multi-type DNS query (A, AAAA, CNAME, TXT, MX, SOA)

### HTTP
- **httpx**: Probing, title, status, tech detect, CDN check
- **httprobe**: Simple host probing
- **aquatone**: Visual inspection, screenshots

### Infrastructure
- **bgpview.io API**: ASN to prefix mapping
- **ipinfo.io API**: IP -> org, ASN, location
- **browser-x**: Browser automation for recon

## Pipeline Integration

### Default PD Chain
subfinder -d target.com -silent | puredns resolve -r resolvers.txt | httpx -silent -title -status-code -tech-detect -o live.txt

### Full Recon Pipeline
subfinder -d target.com -all |   puredns resolve -r resolvers.txt |   naabu -top-ports 1000 -silent -json |   httpx -silent -title -status-code -tech-detect -mc 200 |   nuclei -t ~/nuclei-templates/ -o vulns.txt

### Cloud Asset Discovery
chaos -d target.com -silent | httpx -silent -tech-detect | grep -E 'S3|Cloud|Azure|GCP'