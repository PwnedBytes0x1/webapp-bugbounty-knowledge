# JavaScript Analysis: Finding Hidden Attack Surface

## Automated JS Discovery

```bash
# Find all JS files linked from pages
gospider -s https://target.com -d 2 --js | grep -E '\.js' | sort -u
# Wayback JS
gau target.com | grep -E '\.js' | sort -u
# Subdomain-specific JS
subfinder -d target.com | httpx -silent | subjs | sort -u
```

## Static Analysis Techniques

### Extract Endpoints from JS
```bash
# Regex-based extraction
cat bundle.js | grep -oP 'https?://[^"'"'"' )]+' | sort -u
cat bundle.js | grep -oP '["'"'"'](/[a-zA-Z0-9_/.-]+)["'"'"']' | sort -u | grep -v '\.css\|\.png\|\.svg\|\.woff'
```

### Secrets in JS
```bash
# High-signal patterns
grep -roP '(?:api[Kk]ey|apikey|api_key|secret|token|password|credentials?)["'\s]*[:=]\s*["'"'"'][^"'"'"']+' *.js
grep -roP 'AIza[0-9A-Za-z_-]{35}' *.js  # Firebase API key
grep -roP 'AKIA[0-9A-Z]{16}' *.js       # AWS Access Key
grep -roP 'sk_live_[0-9a-z]+' *.js      # Stripe live key
grep -roP 'ghp_[0-9A-Za-z]{36}' *.js    # GitHub personal access token
```

Use `trufflehog`, `secretfinder`, or `mantra` for comprehensive scanning.

### Source Map Analysis
If source maps are deployed to production, the entire source is recoverable:

```bash
# Check for .map files
ffuf -u https://target.com/assets/app.jsFUZZ -w .map
# Use source-map-unpack or similar tools
```

## Runtime Analysis with Burp/DOM Invader

### DOM Invader for Client-Side Scanning
1. Open Burp browser with DOM Invader enabled
2. Navigate through the application triggering every feature
3. Monitor DOM Invader's "Prototype Pollution" and "DOM XSS" tabs
4. Investigate canary reports — green canaries without sinks are useful; with sinks = exploitable

### Sink Discovery
```js
// Monitor in console for data flow to dangerous sinks
monitorEvents(document.body, 'mouseover');
// Watch for innerHTML assignments, eval calls, document.write usage
```

## WebSocket Discovery

```js
// Intercept WebSocket connections from JS
new WebSocket('wss://target.com/ws');
// Look for /ws, /socket, /connect, /stream patterns in JS
```

Check WebSocket authentication: is the session cookie sent? Is a token in the URL? Do messages contain auth tokens?

## JS Bundler Fingerprinting

- **Webpack** — `webpackJsonp`, `__webpack_require__`
- **Rollup** — IIFE pattern with `(function () {` at top
- **Parcel** — `$parcel$` runtime identifiers
- **Vite** — ES module imports with `?t=` timestamps

Each has different vulnerability profiles — Webpack source maps are commonly accidentally deployed, Rollup bundles are harder to deobfuscate, Vite dev servers on production expose HMR endpoints.
