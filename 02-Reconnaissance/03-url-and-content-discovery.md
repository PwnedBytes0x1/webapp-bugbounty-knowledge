# URL & Content Discovery: Directories, Parameters, Endpoints

## Directory Brute-Forcing at Scale

### Wordlist Strategy
```bash
# Tiered approach:
# 1. Small wordlist for speed (SecLists common.txt)
ffuf -u https://target.com/FUZZ -w ~/wordlists/common.txt -c -t 100
# 2. Medium for depth (SecLists directory-list-2.3-medium.txt)
ffuf -u https://target.com/FUZZ -w big.txt -recursion -recursion-depth 3
# 3. Targeted for specific technologies (WordPress, Laravel, Spring-specific lists)
ffuf -u https://target.com/FUZZ -w raft-large-files.txt -e .php,.asp,.aspx,.jsp,.json
```

### Recursive Discovery
```bash
# FFUF recursion with depth limit
ffuf -u https://target.com/FUZZ -w ~/wordlists/raft-large-directories.txt   -recursion -recursion-depth 3 -c -t 80 -o results.json
```

## Parameter Discovery

Hidden parameters often unlock hidden functionality:

### From JavaScript
```bash
# Extract parameter names from JS
grep -roP '(?<=\?|&)[a-zA-Z_][a-zA-Z0-9_]*=' ~/js-files/ | sort -u
# Use LinkFinder or relative-url-extractor
python3 linkfinder.py -i https://target.com/app.js -o cli
```

### From Known Endpoints
```bash
# Use param miner (Burp extension) for automated discovery
# Manual: Test common hidden params on every endpoint
```

### Fuzzing Unknown Parameters
```bash
# Add common params to requests
ffuf -u https://target.com/api/resource?FUZZ=test -w params.txt -fc 400,404,500
```

Look for:
- `debug`, `dev`, `test`, `env` — often enable debug output
- `admin`, `internal` — access to administrative functions
- `source`, `view`, `template` — SSTI paths
- `callback`, `url`, `redirect` — SSRF vectors
- `id`, `user_id`, `document_id` — IDOR potential

## Wayback Machine & Historical Data

```bash
# Get all historical URLs
gau --subs target.com | sort -u
waybackurls target.com | sort -u
# Filter for interesting file types
cat urls.txt | grep -E '\.js$|\.json$|\.xml$|\.yaml$|\.conf$|\.sql$'
# Find endpoints that no longer exist (potential subdomain takeover)
cat urls.txt | grep -v "$(curl -sI https://target.com | grep 'location:' | cut -d' ' -f2)"
```

## Technology-Specific Endpoints

### GraphQL
```bash
ffuf -u https://target.com/FUZZ -w graphql-endpoints.txt
# Check: /graphql, /v1/graphql, /graph, /gql, /query
# Also probe /graphql?query={__schema{types{name}}}
```

### API Gateways
```bash
ffuf -u https://target.com/api/vFUZZ -w numbers.txt
ffuf -u https://api.target.com/vFUZZ -w numbers.txt
```

### AWS S3/CloudFront
```bash
ffuf -u https://target.com.s3.amazonaws.com -w common.txt -fc 404
# Or bucket name permutations
```

### Swagger/OpenAPI
```bash
ffuf -u https://target.com/FUZZ -w swagger-paths.txt
# /swagger.json, /api-docs, /swagger-ui.html, /openapi.json
```

## Mobile API Endpoints

Mobile apps often call different API endpoints than the web app:

```bash
# Extract from decompiled APK
apktool d app.apk
grep -rohP 'https?://[^"'"'"' ]+' app/ | sort -u
# Or through mitmproxy during runtime
