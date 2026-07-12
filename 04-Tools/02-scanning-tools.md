# Vulnerability Scanning Tools

## Web Application Scanners

### Burp Suite
- **Active Scan**: Automated crawling + scanning
- **Passive Scan**: Background analysis without invasive payloads
- **Extensions**: Autorize, AuthMatrix, Collaborator, Sequencer

### OWASP ZAP
- **Automated Scan**: Quick scan with spider
- **Active Scan**: Targeted vulnerability detection
- **HUD**: Heads-up display for manual testing

## Specialized Scanners

### Nuclei (Template-based)
`nuclei -u https://target.com -t cves/ -t exposures/ -t misconfigurations/`

### Nikto
`nikto -h https://target.com`

### Skipfish
`skipfish -o output_dir https://target.com`

## Cloud Security Scanners

### AWS
- **ScoutSuite**: Multi-cloud auditing
  `ScoutSuite --provider aws`
- **Prowler**: AWS security assessment
  `prowler -M text`

### GCP
- **GCPBucketBrute**: GCP bucket enumeration
  `gcpbucketbrute -k target`

### Azure
- **Stormspotter**: Azure attack surface mapper

## API Scanners
- **Arjun**: HTTP parameter discovery
  `arjun -u https://target.com/api`
- **Kiterunner**: API path bruteforce
  `kr scan https://target.com -w api-routes.txt`