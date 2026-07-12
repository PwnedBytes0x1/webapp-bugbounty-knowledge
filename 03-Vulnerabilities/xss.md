# Cross-Site Scripting (XSS)

## Types

- **Reflected** - Payload in URL reflected in response
- **Stored** - Payload stored on server, served to all users
- **DOM-based** - Client-side JS renders untrusted data
- **Blind** - Triggered in admin panel (support tickets, etc.)

## Testing Methodology

1. Identify input reflection points
2. Test special characters: <>"'&
3. Break out of context (HTML, attribute, JS, URL)
4. Use context-specific payloads

## Context-Specific Payloads

**HTML Context:**
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<details open ontoggle=alert(1)>

**Attribute Context:**
" onfocus=alert(1) autofocus=
" onmouseover=alert(1) "

**JavaScript Context:**
';alert(1)//
</script><script>alert(1)</script>

## Tools
- Burp Suite (DOM Invader)
- XSStrike
- Dalfox

## Escalation Paths

- Session not HttpOnly: Steal cookie -> Session hijack -> ATO
- HttpOnly cookie: XHR to change email/password -> ATO
- Self-XSS: Combine with CSRF -> Trigger on victim

## Prevention
- Context-aware output encoding
- Content Security Policy (CSP)
- Input validation (positive allowlist)
