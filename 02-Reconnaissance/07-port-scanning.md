# Port Scanning: 2026 Complete Reference

## Tool Selection Guide

| Tool | Speed | Features | Best For |
|------|-------|----------|----------|
| naabu | Fast | SYN/CONNECT/UDP, -sV, CDN exclusion | Pipeline integration |
| masscan | Fastest | Full port, large ranges | Internet-scale |
| rustscan | Medium | Wraps nmap | Quick targeted |
| nmap | Slowest | Scripts, full version detection | Detailed service analysis |

## Naabu Deep Dive (v2.6.0)

### Scan Types
- SYN scan (-scan-type s): Fastest, requires root/sudo
- CONNECT scan (-scan-type c): TCP connect, no root needed
- UDP scan (-p u:53,u:161): UDP services, use -uP for protocol probes

### Service Version Detection (-sV)
Built-in nmap-service-probes integration:
- -sD: Service discovery by port (no probing)
- -sV: Full version detection
- -sV-fast: Port-hinted only (skips fallback probes)
- -sV-timeout: Probe timeout (default 5s)
- -sV-workers: Concurrent workers (default 25)

### UDP Service Probes (-uP, new in v2.6.0)
Sends protocol-specific payloads:
- DNS query for port 53
- NTP request for port 123
- SNMPv1 GetRequest for port 161
Without -uP, sends empty datagram (most UDP services stay silent).

### Predictive Scanning (--smart-scan, new in v2.6.0)
Port correlation model reorders scan order:
- Prioritizes ports most likely to be open
- Found open port dynamically boosts correlated ports
- 95x faster in benchmarks
- --prediction-threshold: 0-100% confidence (default 20)

### Performance Optimization
- -rate 5000 (solid uplinks), 1000 (default)
- -c 50 concurrent threads
- --exclude-cdn (skips Cloudflare/Akamai ranges)
- -retries 3 (default)

### Evasion
- -rate 50-200 on protected scopes
- -scan-type connect (no SYN needed)
- SOCKS5 proxy: -proxy socks5://127.0.0.1:9050
- -interface eth0 -source-ip 10.0.0.5
- -warm-up-time: stagger scan start

### Output Formats
- JSON: -json -o results.json (preferred for pipelines)
- TXT: -o results.txt

## Host Discovery (naabu -wn)
- ARP ping (-arp)
- TCP SYN ping (-ps 80)
- TCP ACK ping (-pa 443)
- ICMP echo (-pe)
- ICMP timestamp (-pp)
- ICMP address mask (-pm)
- IPv6 neighbor discovery (-nd)

## Complete Pipeline
subfinder -d target.com -silent | naabu -silent -top-ports 1000 -json -o ports.json | jq -r '.ip + ":" + (.port|tostring)' | httpx -silent -title -status-code -tech-detect -mc 200,301,400 -o live.txt

## Ports to Prioritize

### Web (scan always)
- 80,443 (HTTP/HTTPS)
- 8080,8443 (alternate)
- 3000 (Node.js/React dev)
- 5000 (Flask/FastAPI)
- 8000,8888 (Django/Dev)
- 9000,9090,9443 (admin/management)

### Database (high value)
- 3306 (MySQL)
- 5432 (PostgreSQL)
- 27017 (MongoDB)
- 6379 (Redis)
- 9200 (Elasticsearch)
- 11211 (Memcached)

### Other High-Value Internal
- 21 (FTP)
- 22 (SSH)
- 25 (SMTP)
- 53 (DNS)
- 389 (LDAP)
- 445 (SMB)
- 1433 (MSSQL)
- 1521 (Oracle)
- 3389 (RDP)
- 5900 (VNC)
- 8086 (InfluxDB)
- 9092 (Kafka)

## Output Analysis
- Pipe JSON through jq to extract specific fields
- Filter by status code, response size, tech detect
- Chain into nuclei for vulnerability validation