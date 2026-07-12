# HTTP Protocol: Complete Reference for Bug Bounty

## HTTP/1.1 — The Desync Era

### Message Boundary Problem
HTTP/1.1 defines three ways to delimit a message body:
1. Content-Length (CL): Fixed byte count
2. Transfer-Encoding: chunked (TE): Self-delimiting chunks terminated by 0-length chunk
3. Connection: close: Server closes the socket

### CL.TE Smuggling
Frontend uses CL, backend uses TE:
POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

GPOST /admin HTTP/1.1
Frontend reads CL=13, considers request done. Backend reads TE, treats GPOST as next request.

### TE.CL Smuggling
Frontend uses TE, backend uses CL:
POST / HTTP/1.1
Host: target.com
Content-Length: 4
Transfer-Encoding: chunked

5c
GPOST / HTTP/1.1
Host: internal-admin
Content-Length: 15
x=1
0

Frontend (TE-aware) reads through the chunk. Backend (CL-aware) reads CL=4 only.

### CL.0 (PortSwigger 2024)
Some endpoints ignore CL for GET/HEAD/OPTIONS. Send CL:0 with body — backend treats body as next request.

### TE.0 (Dechunking Desync — 2025, Kettle)
Proxy removes TE but forgets to update CL.

### Expect-Based Desync (Kettle, 2026)
HTTP/1.1 responses have one header block unless you send Expect: 100-continue. Second header block bypasses front-end response removal.

### 0.CL Desync (Kettle, 2026)
Double-desync: Chain 0.CL and CL.0 in two requests.

## HTTP/2 — Binary Framing

### Key RFC Rules
- RFC 9113 Section 8.2.2: Transfer-Encoding MUST NOT appear in HTTP/2
- Binary length-prefixed frames eliminate body ambiguity
- Receiving TE = PROTOCOL_ERROR

### H2.CL / H2.TE (Kettle 2021)
Some proxies forward malformed TE/CL during H2 to H1.1 downgrade.

### H2.0 (Kettle 2022)
Stream declares CL=0 but carries trailing DATA frames that leak into next stream.

### Rapid Reset (CVE-2023-44487)
Open many streams, immediately RST_STREAM each. Exhausts server connection handling.

## HTTP/3 (QUIC) — New Vectors

### Key Differences
- QUIC over UDP (not TCP)
- TLS 1.3 mandatory
- QPACK replaces HPACK
- Connection migration across IPs
- No head-of-line blocking

### QUIC Downgrade Desync (2026)
Proxy accepts HTTP/3, forwards HTTP/1.1 — QUIC-to-TCP translation creates new parsing differentials.

### HAProxy H3 Smuggling (CVE-2026)
HAProxy didn't validate header names/values under HTTP/3. Malformed headers (CRLF, uppercase) passed through when downgraded to HTTP/1.1.

### Alt-Svc Fallback Attack
Blocking QUIC forces downgrade to HTTP/2 over TLS, moving attack surface rather than reducing it.

### WebTransport (RFC 9297)
Port 4433 UDP. Multiple bidirectional streams + unreliable datagrams. No WAF signatures exist. No pentesting tooling supports it.

### Connection Coalescing Abuse (2024-2026)
Browsers coalesce H2/H3 requests onto one TLS connection when cert/ALPN/IP match. If CDN only validates Host on first request, subsequent coalesced requests inherit authorization.

## Transfer-Encoding Obfuscation Variants
Transfer-Encoding: xchunked
Transfer-Encoding : chunked
Transfer-Encoding: chunked\\x0b
Transfer-Encoding: chunked\\x0c
Transfer-Encoding: chunked\\x20
Transfer-Encoding: chunked\\x00
Transfer-Encoding: chunked,
Transfer-Encoding: chunked, identity
Transfer-Encoding: chunked\\nTransfer-Encoding: chunked

## Detection Cheatsheet (2026)
1. Send two requests in same TCP/TLS with different Host
2. CL.TE probe: 400/403/422 = safe; 200 = vulnerable
3. TE.TE obfuscation: send 5 variants over H2, expect identical codes
4. Check Alt-Svc for h3; test H3 CL.TE with QUIC-capable client
5. TCP-probe WebTransport port: TCP open on UDP port = misconfig

## Mitigation: Upstream HTTP/2+
Kettle 2026: More desync attacks are always coming. Stop patching HTTP/1.1.
- Upstream H2: HAProxy, F5 Big-IP, Google Cloud, Imperva, Apache (experimental)
- H1.1-only: nginx, Akamai, CloudFront, Fastly
- WAFs cannot effectively thwart desync