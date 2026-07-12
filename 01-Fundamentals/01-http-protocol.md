# HTTP Protocol Deep Dive

## Request Structure
```
METHOD /path HTTP/1.1
Host: target.com
User-Agent: Mozilla/5.0
Accept: */*
Cookie: session=abc123

body=content
```

## Methods
- GET: Retrieve resource
- POST: Create/submit data
- PUT: Replace resource
- PATCH: Partial update
- DELETE: Remove resource
- OPTIONS: List allowed methods
- HEAD: Headers only (no body)

## Response Structure
```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234
Set-Cookie: session=xyz

<html>...</html>
```

## Status Codes
- 1xx: Informational
- 2xx: Success (200 OK, 201 Created, 204 No Content)
- 3xx: Redirect (301 Moved, 302 Found, 307 Temporary)
- 4xx: Client Error (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found)
- 5xx: Server Error (500 Internal, 502 Bad Gateway, 503 Service Unavailable)

## HTTP/2 Differences
- Binary protocol (not text)
- Multiplexed streams
- Server push
- Header compression (HPACK)
- Single connection per origin

## HTTP Request Smuggling Prerequisites
- Frontend and backend disagree on Content-Length vs Transfer-Encoding
- CL.TE, TE.CL, TE.TE variants