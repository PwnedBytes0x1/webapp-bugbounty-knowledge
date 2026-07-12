# URL & Content Discovery: Complete Methodology

## Historical URL Analysis

### Gau (GetAllUrls)
```bash
cat live_hosts.txt | gau --threads 5 > historical_urls.txt
# Sources: Wayback Machine, CommonCrawl, AlienVault OTX, URLScan

# Filter for interesting parameters
grep "?" historical_urls.txt | sort -u > parameter_urls.txt
grep -E "\.js($|\?)" historical_urls.txt | sort -u > historical_js.txt
grep -iE "admin|api|dev|staging|internal|backup|config" historical_urls.txt > interesting_urls.txt
```

### Waybackurls
```bash
# Tomnomnom's tool
waybackurls example.com > wayback_urls.txt

# Extract specific file types
cat wayback_urls.txt | grep -E '\.(json|xml|yaml|yml|env|config|sql|bak|old|txt|php~|swp)$' > sensitive_files.txt
```

## Active Crawling

### Katana (Modern Crawler)
```bash
# Standard crawl
katana -u https://target.com -d 5 -jc -kf all -silent -o crawl_urls.txt

# Subdomain-aware crawl
katana -list live_hosts.txt -d 3 -jc -kf all -ef woff,css,png,svg,jpg -silent -o cross_crawl.txt

# Known files endpoint discovery
katana -u https://target.com -d 3 -kf knownfiles -silent > known_files.txt
```

### Gospider (Depth Crawl)
```bash
gospider -s https://target.com -d 3 -c 50 -t 30 --js -o spider_output
gospider -S live_hosts.txt -d 2 --subs -o cross_spider
```

## Content Discovery (Directory Brute Force)

### ffuf Configuration
```bash
# Standard directory discovery
ffuf -u https://target.com/FUZZ -w wordlist.txt -c -t 100 -fc 403,404 -o ffuf_std.json

# Extension filtering
ffuf -u https://target.com/FUZZ -w wordlist.txt -e .php,.asp,.aspx,.jsp,.do,.action

# Recursive (depth 3)
ffuf -u https://target.com/FUZZ -w big.txt -recursion -recursion-depth 3 -of json -o recursive.json

# POST body fuzzing
ffuf -u https://target.com/api/login -w users.txt:USER -w passwords.txt:PASS -X POST -d "username=USER&password=PASS" -fc 401,403 -t 50

# Host header fuzzing
ffuf -u https://target.com -w vhosts.txt -H "Host: FUZZ.target.com" -fc 200,302

# Method fuzzing
ffuf -u https://target.com/api/resource -w methods.txt -X FUZZ -fc 400,404 -mc 200,201,403,405,500

# Content-Type switching
ffuf -u https://target.com/api/endpoint -X POST -d '{"data":"test"}' -w content_types.txt -H "Content-Type: FUZZ" -fc 415
```

### Wordlist Selection
```bash
# Directory wordlists (ordered by size)
/usr/share/seclists/Discovery/Web-Content/common.txt           # 4,700
/usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt  # ~63k
/usr/share/seclists/Discovery/Web-Content/raft-large-words.txt   # ~140k
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt  # ~220k
/usr/share/seclists/Discovery/Web-Content/api/actions.txt       # API-specific
SecLists/Discovery/Web-Content/burp-parameter-names.txt          # Parameters

# Technology-specific wordlists
# Laravel: /api, /sanctum/csrf-cookie, /_debugbar, /telescope
# Django: /admin, /api, /graphql
# Rails: /rails/info/routes, /assets, /packs
# Spring: /actuator, /actuator/health, /swagger-ui, /v2/api-docs
# Wordpress: /wp-admin, /wp-content, /wp-json, /xmlrpc.php
```

## Parameter Discovery

### Arjun
```bash
arjun -u https://target.com/api/endpoint -m POST --stable -oArjun /tmp/
arjun -u https://target.com/api/endpoint --headers -o arjun_params.txt
```

### ParamSpider
```bash
python3 paramspider.py --domain target.com --exclude woff,css,js,png,svg,jpg
# Output: results/target.com.txt
```

### GF (Pattern Matching)
```bash
# Install gf patterns
git clone https://github.com/1ndianl33t/Gf-Patterns
cp ~/Gf-Patterns/*.json ~/.gf/

# Classify URLs
cat all_urls.txt | gf xss > potential_xss_urls.txt
cat all_urls.txt | gf sqli > potential_sqli_urls.txt
cat all_urls.txt | gf ssrf > potential_ssrf_urls.txt
cat all_urls.txt | gf idor > potential_idor_urls.txt
cat all_urls.txt | gf redirect > potential_open_redirect.txt
cat all_urls.txt | gf debug_logic > potential_debug.txt
cat all_urls.txt | gf interestingparams > interesting_params.txt

# Custom GF patterns
# Write custom patterns in ~/.gf/*.json
```

## API Endpoint Discovery

### Kiterunner (API-Specific)
```bash
# API route discovery
kr scan https://target.com -w routes-large.kite -o api_routes.json

# JWT / token analysis
kr scan https://target.com/api -w api-routes-small.kite -A jwt
```

### API Gateway Testing
```bash
# Common API paths
ffuf -u https://api.target.com/v1/FUZZ -w api_paths.txt -fc 403,404
ffuf -u https://api.target.com/v2/FUZZ -w api_paths.txt -fc 403,404
ffuf -u https://api.target.com/internal/FUZZ -w api_paths.txt
```

## Technology Fingerprinting

```bash
# whatweb
whatweb -a 3 https://target.com -v

# wappalyzer CLI
wappalyzer https://target.com

# httpx tech detection
echo "https://target.com" | httpx -tech-detect -silent

# nuclei tech detection
echo "https://target.com" | nuclei -t http/technologies/ -silent

# BuiltWith (API)
# https://api.builtwith.com/v19/api.html
```
