# Wordlists & Tool Reference: 2026

## Essential Wordlists

### SecLists (Daniel Miessler)
https://github.com/danielmiessler/SecLists
- Discovery/Web-Content/: Directories, files, parameters
- Discovery/DNS/: Subdomain wordlists
- Usernames/, Passwords/: Credential lists
- Fuzzing/: Payload collections

### Other Wordlists
- **raft (large/medium/small)** — Web content discovery
- **commonspeak2** — Derived from BigQuery
- **Assetnote wordlists** — Best for directory brute-force
- **OneForAll wordlist** — Chinese subdomain wordlist
- **jhaddix subdomain list** — ~2M subdomains
- **nn9ed wordlist** — Focused API endpoints

## Tool Installation

### ProjectDiscovery Tools (pdtm)
```bash
curl -s https://raw.githubusercontent.com/projectdiscovery/pdtm/main/install.sh | bash
pdtm install subfinder httpx nuclei naabu katana uncover dnsx asnmap mapcidr chaos
```

### Other Essential
```bash
go install github.com/tomnomnom/waybackurls@latest
go install github.com/tomnomnom/assetfinder@latest
go install github.com/ffuf/ffuf/v2@latest
go install github.com/jaeles-project/gospider@latest
pip install sqlmap arjun uro
```

### Resolvers List
```bash
wget -O resolvers.txt https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt
```