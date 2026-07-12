# Reconnaissance Tools: Configuration & Advanced Usage

## Subdomain Discovery

### subfinder (Passive)
```bash
subfinder -d target.com -all -silent -o subs.txt
# -all enables ALL passive sources
# Key sources: crtsh, certspotter, dnsdb, hackertarget, securitytrails, virustotal
```

### amass
```bash
# Passive enumeration
amass enum -passive -d target.com -o amass.txt
# Active enumeration (queries NS, attempts zone transfer)
amass enum -active -d target.com -o active.txt
# Intel mode (related domains via ASN, reverse WHOIS)
amass intel -whois -d target.com
```

### dnsx (DNS Probing)
```bash
# Resolve A, AAAA, CNAME records
cat subs.txt | dnsx -a -aaaa -cname -resp -silent -o resolved.txt
# Wildcard filtering
dnsx -l subs.txt -wd -silent
```

## Service Discovery

### httpx (HTTP Probing)
```bash
cat subs.txt | httpx -title -status-code -tech-detect -follow-redirects \
  -web-server -ip -cname -cdn -silent -o metadata.json
```

### naabu (Port Scanning)
```bash
naabu -l hosts.txt -top-ports 1000 -silent -o ports.txt
naabu -l hosts.txt -p - -rate 1000  # Full scan
```

## Content Discovery

### ffuf
```bash
# Directory brute-force
ffuf -u https://target.com/FUZZ -w wordlist.txt -c -t 100 -fc 403,404
# Parameter fuzzing
ffuf -u https://target.com/api/FUZZ -w params.txt -mc all -fc 400
# Recursive with depth
ffuf -u https://target.com/FUZZ -w big.txt -recursion -recursion-depth 3
# VHOST discovery
ffuf -u https://target.com -H "Host: FUZZ.target.com" -w subs.txt -fc 200
```

### gospider (Crawler)
```bash
gospider -s https://target.com -d 3 -c 50 -t 30 --js -o spider_output
gospider -S subs.txt -d 2 --subs -o cross_spider
```

## Favicon/Hash Scanning
```bash
python3 favicon_hash.py -d target.com
# Search Shodan: http.favicon.hash:<hash>
```
