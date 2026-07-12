# Recon One-Liners and Workflows

## Complete Recon Pipeline

```bash
# 1. Subdomain enumeration
subfinder -d target.com -all -o subs.txt
amass enum -passive -d target.com >> subs.txt

# 2. DNS resolution + live host probing
cat subs.txt | dnsx -a -resp -silent | tee resolved.txt
cat resolved.txt | httpx -silent -status-code -title -tech-detect | tee alive.txt

# 3. URL collection
cat alive.txt | gau --subs | tee all_urls.txt
cat alive.txt | waybackurls | tee -a all_urls.txt

# 4. JavaScript collection and analysis
cat all_urls.txt | grep -E \.js$ | sort -u > js_files.txt

# 5. Parameter filtering
cat all_urls.txt | grep -E \? | sort -u > param_urls.txt

# 6. Vulnerability scanning
cat alive.txt | nuclei -t cves/ -t exposures/ -severity critical,high
```

## One-Liners

### Subdomain Enumeration
```
subfinder -d target.com -all | anew subs.txt
amass enum -passive -d target.com | anew subs.txt
```

### Live Host Discovery
```
cat subs.txt | dnsx -a -resp | httpx -silent -status-code -title -tech-detect
```

### URL Discovery
```
cat subs.txt | gau --subs | sort -u | uro
cat subs.txt | waybackurls | sort -u
```

### Parameter Discovery
```
cat all_urls.txt | gf xss | sort -u > xss_params.txt
cat all_urls.txt | gf sqli | sort -u > sqli_params.txt
cat all_urls.txt | gf ssrf | sort -u > ssrf_params.txt
```

### Tech-Specific Fuzzing
```
# Spring Boot
ffuf -w /opt/seclists/Discovery/Web-Content/spring-boot.txt -u https://target.com/FUZZ

# API routes
ffuf -w /opt/seclists/Discovery/Web-Content/api.txt -u https://target.com/FUZZ
```
