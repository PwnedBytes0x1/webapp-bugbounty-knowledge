# Scanning Tools

## Vulnerability Scanners
| Tool | Type | Description |
|------|------|-------------|
| Nuclei | Template | Fast YAML-based scanner |
| Nikto | Web server | Comprehensive web scanner |
| WPScan | CMS | WordPress vulnerability scanner |
| Wapiti | Web app | Black-box web scanner |

## Port & Service Scanners
| Tool | Description |
|------|-------------|
| Nmap | Industry standard network scanner |
| Masscan | Internet-scale port scanner |
| Naabu | Fast port scanner by ProjectDiscovery |
| Rustscan | Rust-based port scanner |

## API Scanners
| Tool | Description |
|------|-------------|
| Kiterunner | API endpoint discovery |
| Arjun | HTTP parameter discovery |
| Graphw00f | GraphQL fingerprinting |
| Inql | GraphQL security testing |

## CMS & Framework Scanners
| Tool | Target |
|------|--------|
| WPScan | WordPress |
| Joomscan | Joomla |
| Droopescan | Drupal |
| CMSmap | WP, Joomla, Drupal |

## TLS/SSL Scanners
| Tool | Description |
|------|-------------|
| testssl.sh | Comprehensive TLS testing |
| SSLyze | Fast TLS scanner |

## Installation
```bash
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
go install -v github.com/ffuf/ffuf/v2@latest
pip install arjun wapiti3
apt install nmap nikto wpscan
```

---

> **Next**: [Exploitation Tools](03-exploitation-tools.md)
