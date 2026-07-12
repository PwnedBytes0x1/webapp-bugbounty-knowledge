# Subdomain Enumeration

## Passive Tools
```bash
subfinder -d target.com -all -o subs.txt
amass enum -d target.com -passive -o amass.txt
assetfinder --subs-only target.com
chaos -d target.com -key API_KEY -o chaos.txt
```

## Active Tools
```bash
gobuster dns -d target.com -w subdomains.txt -t 50
dnsx -d target.com -w subdomains.txt -r resolvers.txt
puredns bruteforce subdomains.txt target.com -r resolvers.txt
```

## Recursive Enumeration
```bash
# Found subdomain: dev.target.com
# Enumerate deeper: *.dev.target.com
subfinder -d dev.target.com
amass enum -d dev.target.com
```

## Permutation Generation
```bash
# Based on discovered subdomains
altdns -i subs.txt -o data.csv -w words.txt -r -s subs_perm.txt
dnsgen subs.txt -w /usr/share/seclists/Discovery/DNS/deepmagic.txt
```

## Resolution & Validation
```bash
puredns resolve subs_final.txt -r resolvers.txt | tee resolved.txt
dnsx -l subs_final.txt -resp-only -o resolved.txt
```

## Common Subdomains to Check
```
admin, api, dev, staging, test, uat, qa, beta, preprod,
vpn, mail, webmail, portal, dashboard, console, cdn, static,
assets, upload, files, backup, db, mysql, redis, jenkins, git,
jira, confluence, wiki, docs, support, help, status, monitor
```
