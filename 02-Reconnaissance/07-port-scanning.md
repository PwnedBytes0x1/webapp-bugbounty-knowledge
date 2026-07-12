# Port Scanning

## Masscan (Fast)
```bash
# Full port scan
masscan -p1-65535 --rate=10000 target.com -oJ masscan.json

# Common web ports
masscan -p80,443,8080,8443,3000,5000,8000,9000,9090,9443 --rate=10000
```

## Nmap (Detailed)
```bash
# Service detection
nmap -sV -p 80,443,8080 -T4 target.com

# Script scanning
nmap -sC -p 80,443 target.com

# Full scan
nmap -sV -sC -p- --min-rate=1000 target.com -oA target

# Vulnerability scripts
nmap --script vuln -p 80,443 target.com
```

## naabu (Go-based)
```bash
naabu -host target.com -p - -rate 1000 -o ports.txt
naabu -list hosts.txt -top-ports 1000
```

## Internal Port Mapping (via SSRF)
```bash
# Common internal ports
:443 (HTTPS)
:8080 (HTTP Proxy/Alt)
:8443 (HTTPS Alt)
:3000 (Node/Dev)
:5000 (Flask Dev)
:9000 (PHP-FPM)
:6379 (Redis)
:27017 (MongoDB)
:3306 (MySQL)
:5432 (PostgreSQL)
:11211 (Memcached)
:9200 (Elasticsearch)
```

## Port to Service Mapping
| Port | Service | Vulnerabilities |
|------|---------|----------------|
| 21 | FTP | Anonymous login, weak creds |
| 22 | SSH | Weak keys, configured vulns |
| 25 | SMTP | Open relay |
| 53 | DNS | Zone transfer |
| 445 | SMB | EternalBlue |
| 6379 | Redis | No auth, RCE via cron |
| 27017 | MongoDB | No auth, data exposure |
| 9200 | Elasticsearch | No auth, data exposure |
| 11211 | Memcached | UDP amplification |