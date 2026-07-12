# Active Reconnaissance

## DNS Bruteforce
```bash
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
dnsrecon -d target.com -t brt -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

## Port Scanning
```bash
# Quick scan
masscan -p1-65535 --rate=10000 target.com -oJ masscan.json

# Detailed scan
nmap -sV -sC -p- --min-rate=1000 target.com -oA nmap
```

## Live Host Probing
```bash
cat subs.txt | httpx -title -status-code -tech-detect -o live.txt
cat subs.txt | httpx -ports 80,443,8080,8443,3000,8000,9000
```

## Technology Fingerprinting
```bash
whatweb https://target.com -v
python3 wafw00f/bin/wafw00f https://target.com
```

## Screenshot Collection
```bash
cat live.txt | aquatone -out screenshots/
eyewitness --web -f live.txt -d eyewitness/
```
