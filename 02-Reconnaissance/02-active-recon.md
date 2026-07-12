# Active Reconnaissance: 2026 Complete Reference

## Port Scanning Tool Comparison

| Tool | Speed | Features | Best For |
|------|-------|----------|----------|
| masscan | Fastest | Full port, large ranges | Internet-scale scanning |
| naabu | Fast | SYN/CONNECT/UDP, PD stack | Pipeline integration |
| rustscan | Medium | Wraps nmap | Quick targeted scans |
| nmap | Slowest | -sV, -sC, scripts | Detailed service detection |

## Naabu (ProjectDiscovery) — v2.6.0

### Key Features
- SYN scan (requires root), CONNECT scan (no root), UDP scan
- Service version detection with nmap-service-probes (-sV)
- UDP service probes (-uP) with protocol-specific payloads
- Predictive smart scan (--smart-scan) using port correlation model (~95x faster)
- CDN/WAF exclusion (--exclude-cdn)
- SOCKS5 proxy support including DNS resolution
- Configurable DNS resolution order (--dns-order)

### Standard Pipeline
subfinder -d target.com | naabu -top-ports 1000 -json | jq -r '.ip + ":" + (.port|tostring)' | httpx -silent -title -status-code -tech-detect -o hosts.txt

### Performance Optimization
- -rate 5000 on solid uplinks (default 1000)
- -c 50 concurrent threads
- -scan-type s SYN scan (faster, needs sudo)
- -stats shows progress + ETA
- -exclude-cdn skips known CDN ranges (Cloudflare/Akamai always return 80/443)

### Service Version Detection
- -sV: Full service version detection using nmap probes
- -sV-fast: Only probe port-hinted services (skips fallback)
- -sV-probes: Custom probe file path
- -sD: Service discovery by port number only (no probing)

### UDP Scanning
- -p u:53 for UDP ports
- -uP sends protocol-specific payloads (DNS query for 53, NTP for 123, SNMP for 161)
- Without -uP, UDP scans send empty datagrams (most services stay silent)

### Predictive Scanning (v2.6.0, 2026)
- --smart-scan reorders ports based on correlation model
- Ports most likely to be open scanned first
- Found open port dynamically boosts correlated ports
- 95x faster in benchmarks
- --prediction-threshold 0-100% (default 20)

### Common Profiles
- Quick: naabu -host target.com -top-ports 1000 -json -o ports.json
- Full: naabu -host target.com -p - -json -o full.json
- Service: naabu -host target.com -p 80,443,8080 -sV -json
- UDP: naabu -host target.com -p u:53,u:161,u:123 -uP
- Evasion: naabu -host target.com -rate 200 -proxy socks5://127.0.0.1:9050

## Evasion Techniques
- Reduce -rate to 50-200 on protected scopes
- Use -scan-type connect when SYN fails (no root needed)
- Rotate source IPs if available
- Add -warm-up-time to stagger scan start
- Route through SOCKS5 proxy for IP rotation

## Port -> Service -> Vulnerability Mapping

| Port | Service | Test For |
|------|---------|----------|
| 21 | FTP | Anonymous login, weak creds |
| 22 | SSH | Weak keys, configured vulns |
| 25 | SMTP | Open relay |
| 53 | DNS | Zone transfer |
| 445 | SMB | EternalBlue |
| 6379 | Redis | No auth, RCE via cron |
| 27017 | MongoDB | No auth, data exposure |
| 9200 | Elasticsearch | No auth |
| 11211 | Memcached | UDP amplification |
| 3389 | RDP | BlueKeep, weak creds |
| 5432 | PostgreSQL | Default creds |
| 3306 | MySQL | Default creds |
| 8080/8443 | HTTP alt | Unprotected admin panels |
| 9090/9443 | HTTP alt | JMX, management consoles |