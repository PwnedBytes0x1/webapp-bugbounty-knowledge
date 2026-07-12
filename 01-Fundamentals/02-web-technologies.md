# Web Technologies: Deep Reference for Bug Bounty

## Framework Detection from Headers

X-Powered-By: Express -> Node.js/Express
X-Runtime: 0.01234 -> Ruby on Rails
X-AspNet-Version: 4.0.30319 -> ASP.NET
Server: gunicorn/20.1.0 -> Python WSGI
Server: uvicorn -> Python ASGI
X-Application-Context -> Spring Boot

## HTTP/2 Fingerprinting (2025-2026)

Traditional UA spoofing and JS fingerprinting dying. Protocol-level identification rising.
Identifies: SETTINGS frame order/values, frame priority ordering, WINDOW_UPDATE behavior, header ordering (case-sensitive), pseudoj-header usage, ALPN negotiation, TLS fingerprint (JA3/JA4).

Bypasses:
1. Full browser: Puppeteer/Playwright inherits native HTTP/2 stack
2. Custom client: undici, http2-wrapper, nghttp2 replicate Chrome frame structure
3. TLS proxy: mitmproxy converting to browser-identical frames
4. Session replay: Export NetLog, replay with identical frame sequence

## Browser Fingerprinting Surfaces

Canvas: Skia anti-aliasing produces GPU/OS-specific pixels. Brave farbling, Camoufox Skia patch.
WebGL: getParameter, getSupportedExtensions. Cross-check claimed GPU vs rendered pixels.
Audio: OfflineAudioContext + OscillatorNode + DynamicsCompressorNode = machine-specific float.
WebRTC: ICE candidates leak IP via STUN below JS. Fix must be native protocol level.

## Anti-Detect Browser Mechanics (2026)

Engine-level patches (Camoufox, Brave) modify native paths - no JS wrapper to stringify.
Kasada research: CDP detection via Runtime.enable side-channel (broken V8 May 2025).
Playwright: __playwright__binding__ and __pwInitScripts globals.
Selenium: cdc_ prefixed variables in window/document.
Consistency checks beat individual property patches - all signals must agree.

## Framework-Specific Attack Surfaces

Next.js: _next/image SSRF (CVE-2024-34351), __NEXT_DATA__ prop exposure, middleware bypasses, Server Actions.
Spring Boot: /actuator/* endpoints, whitelabel error pages, SpEL injection.
Django: /admin/ path, DEBUG=True errors, SECRET_KEY -> session forge, Pickle deserialization.
Laravel: APP_KEY -> cookie forge, .env exposure, Ignition RCE (CVE-2021-3129).
Rails: config/secrets.yml exposure, unsafe YAML deserialization.

## CDN/LB/WAF Fingerprinting

Cloudflare: cf-ray, cf-cache-status. Weakness: H2 downgrade desync.
CloudFront: x-amz-cf-id. Weakness: origin exposure via direct-to-origin.
Fastly: x-served-by, x-cache. Weakness: CRLF in :method desync (2026).
AWS ALB: x-amzn-trace-id. Weakness: connection-state reuse.
Nginx: server: nginx/1.x.x. Weakness: CL.0 on keep-alive.
