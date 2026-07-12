# Screenshots & PoC: 2026 Guide

## Screenshot Best Practices
- Show both the request (Burp Repeater, curl output) AND the result
- Annotate key elements (red box around exfiltrated data)
- Include browser URL bar and dev tools for XSS
- For blind vulnerabilities: show collaborator callback
- Resolution: 1920x1080 window, readable font size
- Format: PNG is standard; GIF for demonstrations

## PoC Types

### Simple PoC (curl)
```bash
curl -s 'https://target.com/search?q=<script>alert(1)</script>' | grep -i error
```

### Full PoC (HTML file)
```html
<html><body><form action="https://target.com/forgot" method="POST">
<input name="email" value="victim@test.com">
</form><script>document.forms[0].submit();</script></body></html>
```

### Burp Export
Use "Copy as curl command" from Burp Repeater for exact recreation.

### Video PoC
- Use for multi-step chains (race conditions, business logic)
- Screen capture tool + audio explanation
- Keep under 2 minutes
- Upload as unlisted YouTube or attach to report

## Tools
- **Flameshot**: Screenshot + annotation (Linux)
- **ShareX**: Screenshot + GIF + video (Windows)
- **Kap**: Screen capture (macOS)
- **Screenity**: Chrome extension for video PoC