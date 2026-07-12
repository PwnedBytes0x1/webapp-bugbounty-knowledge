# Content Discovery: 2026 Complete Reference

## Tool Comparison

| Tool | Best For | Language |
|------|----------|----------|
| ffuf | Surgical fuzzing (path/param/header/vhost) | Go |
| feroxbuster | Recursive directory discovery | Rust |
| gobuster | Quick directory/DNS | Go |
| dirsearch | Broad scanning | Python |

## ffuf Patterns

### Directory Discovery
ffuf -u https://target.com/FUZZ -w raft-large-directories.txt -ac
-ac = auto-calibrate (learns baseline, filters soft-404s)

### Extension Fuzzing
ffuf -u https://target.com/FUZZ -w raft-medium-files.txt -e .php,.bak,.old,.zip,.txt,.json

### Backup File Hunting (highest value hits)
ffuf -u https://target.com/FUZZ -w backup-extensions.txt -mc 200,301,302,401,403 -fc 404

### Parameter Fuzzing (GET)
ffuf -u 'https://target.com/api/endpoint?FUZZ=test' -w burp-parameter-names.txt -ac -mc 200,201,204

### Parameter Fuzzing (POST JSON)
ffuf -X POST -H 'Content-Type: application/json' -d '{"username":"admin","password":"FUZZ"}' -u https://target.com/api/login -fc 401,403

### Virtual Host Discovery
ffuf -u http://TARGET_IP/ -H 'Host: FUZZ.target.com' -w subdomains.txt -ac
- Requires baseline response size first, then filter with -fs
- Finds hosts not in DNS (internal-only vhosts)

### Recursive
ffuf -u https://target.com/FUZZ -w list.txt -recursion -recursion-depth 2

### Advanced Filtering
- -mc 200,301,302,401,403 (match codes - 401/403 = path exists but restricted)
- -fc 404 (filter 404)
- -fs 1234 (filter by size)
- -fw 50 (filter by word count)
- -fr 'error' (filter regex)
- -ac (auto-calibrate - essential for multi-host automation)

### Rate/Thread Control
- -rate 50-100 (default safe)
- -t 40-50 (threads)

## feroxbuster Patterns

### Basic Recursive
feroxbuster -u https://target.com -w raft-medium-directories.txt -d 3 --thorough

### With Extensions
feroxbuster -u https://target.com -w list.txt -x php,html,bak,zip,js,json

### Excluding Paths
feroxbuster -u https://target.com -w list.txt --dont-scan '/logout,/exit,/static'

### Status Code Filtering
feroxbuster -u https://target.com -w list.txt -C 404,400 -s 200,301,403 --filter-size 0

## Backup File Discovery (Always High Value)

Find config.php.bak, .env backup, .git/config:
ffuf -u https://target.com/FUZZ -w common-files.txt -e .bak,.old,.orig,.copy,.txt,.zip,.tar.gz,.sql,.dump

### .git Exposure
If .git/config returns 200, use git-dumper to reconstruct entire repo:
git-dumper https://target.com/.git/ git-repo/

### .env Exposure
Any 200 on .env or .env.backup = immediate critical finding.

## Wordlist Strategy

### Default
- SecLists/Discovery/Web-Content/raft-large-directories.txt
- SecLists/Discovery/Web-Content/raft-large-files.txt
- SecLists/Discovery/Web-Content/Common-DB-Backups.txt

### Technology-Specific
Match wordlist to detected stack (httpx -tech-detect). PHP stacks -> .php extensions. Node -> .js, .json.

### API Discovery
- common-api-endpoints-mazen160.txt
- objects.txt (for JSON APIs)
- OpenAPI/Swagger: parse spec -> generate ffuf commands

### Derived from Target
- Crawl site, extract paths from JS bundles
- waybackurls target.com | uro > urls.txt (historical URLs as wordlist)
- JS files often contain hidden endpoints

## File Hunting Priority
1. Backup files (.bak, .old, .orig, .copy) — plaintext source with secrets
2. Config files (.env, .config, .yml, .xml) — credentials, internal hosts
3. Source maps (.map) — full source reconstruction
4. .git — entire repo cloneable
5. Temp files (~, .swp, .swo) — editor artifacts
6. Archives (.zip, .tar.gz, .sql) — full dumps