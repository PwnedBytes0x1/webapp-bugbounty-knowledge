# Security Headers: Complete Reference with CSP Deep Dive

## Essential Headers

HSTS: max-age=31536000; includeSubDomains; preload. Bypass: first request before HSTS.
X-Frame-Options: DENY or SAMEORIGIN. Missing = clickjacking. CSP alt: frame-ancestors.
X-Content-Type-Options: nosniff. Without it: text/plain can render as HTML.
Referrer-Policy: strict-origin-when-cross-origin (modern default) or no-referrer (max privacy).
Permissions-Policy: camera=(), microphone=(), geolocation=(), interest-cohort=().

## CSP Level 3 (W3C CR 2026)

Recommended nonce-based strict CSP:
default-src none; script-src self nonce-RANDOM strict-dynamic; style-src self nonce-RANDOM; img-src self data:; font-src self; connect-src self https://api.target.com; frame-ancestors none; base-uri self; form-action self; upgrade-insecure-requests; require-trusted-types-for script; report-uri /csp-report

## CSP Bypass Techniques (2025-2026)

1. Nonce Leakage via CSS Injection: If style-src allows unsafe-inline, leak nonce via CSS attribute selectors char-by-char.
2. Nonce Reuse via Disk Cache (Jorian Woltjer 2025): bfcache snapshots nonce, CSRF changes payload, navigate back -> old nonce + new payload execute.
3. JSONP Endpoint Abuse: If script-src includes trusted origin with JSONP, inject script via callback parameter.
4. Strict-Dynamic Propagation: Trusted script creates script with src=attacker.com, trusted by strict-dynamic.
5. Data:/Blob: URL Injection: If not explicitly denied, data:text/javascript,alert(1) executes.
6. Base-URI Manipulation: base tag redirects relative script src to attacker.
7. DOM Clobbering + CSP Bypass (Intigriti CTF 2026): Inject data-component, ComponentManager creates script from trusted-origin path, bypasses self.

## CSP Pitfalls
unsafe-inline, script-src *, script-src https:, static nonce, JSONP on allowed domain, unset base-uri, no report-uri, CDN wildcards.

## Testing Checklist
1. Missing headers
2. CSP: unsafe-inline, unsafe-eval, wildcards
3. Nonce reuse
4. style-src inline allowance
5. base-uri set/missing
6. JSONP on allowed origins
7. data:/blob: in script-src
8. form-action redirect
9. frame-ancestors clickjacking
10. referrer-policy token leak