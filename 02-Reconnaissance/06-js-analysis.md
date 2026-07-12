# JavaScript Analysis: 2026 Complete Reference

## Source Map Discovery (Highest-Value Target)

Source maps (.map files) contain the original source code, including API endpoints, secrets, and internal routes.

### Detection
- //# sourceMappingURL=xxx.js.map comment at end of bundles
- Probe: bundle.js.map, bundle.js.map (relative)
- Google dork: site:target.com filetype:map

### Next.js Specific
Bundles served from /_next/static/chunks/
Crawl pages, extract chunk URLs from HTML
But page crawling misses lazy-loaded/admin/feature-flagged chunks.
Better: Parse Webpack chunk registry from initial bundles (contains mapping of every chunk ID to hash).

### Tools
- source-map-sentinel (TreRB): Detects exposed maps, reconstructs sources, greps for secrets/routes
- JS Surgeon: Entropy-based scoring, 40+ secret patterns, Webpack chunk following
- unwebpack-sourcemap / Sourcemapper: Reconstructs directory structure from .map URL

### What Source Maps Reveal (Claude Code March 2026 leak = 59.8MB .map, 512K lines TS)
- Full frontend source (API clients, authentication flow)
- Internal routes (/api/internal/*, /admin/*, debug endpoints)
- Feature flags (BETA, EXPERIMENTAL, ENABLE)
- Developer comments (TODO, FIXME, HACK)
- Environment variables (process.env, import.meta.env)
- Hardcoded credentials and API keys

### Mining Source Maps
1. Discover .map files
2. Reconstruct with source-map library
3. Search for: secrets, internal routes, feature flags, PII, codenames
4. Diff across deployments (content-hashed chunks change when source changes)

## Secret Detection (35+ Patterns)

Patterns with CWE/CVSS mapping:
- AWS Keys (AKIA...), GCP API Keys (AIza...), GCP Service Accounts
- GitHub Tokens (ghp_, gho_, github_pat_)
- Stripe Keys (sk_live_/sk_test_)
- Twilio SID + Auth Token
- Slack Bot/User Tokens + Webhooks
- Firebase API Keys
- JWT Tokens
- OAuth Client Secrets
- RSA/EC Private Keys, Ethereum Private Keys + Mnemonics
- MongoDB/PostgreSQL/MySQL/Redis URIs
- S3/GCS/Azure Blob URLs

### Entropy Analysis
Shannon entropy detects high-entropy strings near sensitive variable names.
Score 0-100% confidence reduces false positives.

### Tools
- TruffleHog: Git history + filesystem scanning
- Gitleaks: Git-oriented secret scanning
- inspectJS: 35+ secret patterns, source maps, tech stack, DOM XSS
- JS Surgeon: Entropy scoring, 40+ patterns, Webpack chunk navigation

## Endpoint Extraction

### Static Analysis Patterns
Search minified JS for:
- Relative paths: /api/, /admin/, /internal/
- Full URLs: https://api...
- HTTP methods: GET, POST, PUT, DELETE
- Parameter keys: role, isAdmin, permissions
- Versioning: /v1/, /v2/, /beta/, /legacy/

### Tools
- LinkFinder / xnLinkFinder: Broad regex endpoint extraction
- inspectJS: fetch/axios/XHR/WebSocket/GraphQL extraction
- E2R (Burp Extension): Montoya API-based passive scanning, AI Workbench generates valid HTTP requests from JS context
- Burp Suite JS Miner: Passive proxy-based
- Jsmon: Continuous JS monitoring, change detection

### Dynamic Analysis (when routes are constructed at runtime)
- Chrome DevTools webpack:// namespace (if source maps loaded)
- Set breakpoints in API client code, step through URL assembly
- Burp DOM Invader: Trace request data flow through DOM

### WebSocket Discovery
Look for: ws://, wss://, new WebSocket(), socket.io
InspectJS detects WebSocket connections automatically.

### GraphQL Discovery
Look for: /graphql, /gql, /query, introspection fragments
InspectJS extracts named query/mutation/subscription operations.

## Tech Stack Fingerprinting (35+ Signatures)

### Tools
- inspectJS: Firebase, Supabase, Stripe, Sentry, Auth0, Clerk, OpenAI, Anthropic
- Wappalyzer: Browser extension
- WhatWeb: CLI

### Library Version Detection
React, Vue, Angular, jQuery, Axios, Socket.io, Next.js.

### Missing SRI Detection
External scripts loaded without integrity= (CWE-353) -> supply chain risk.

## Cloud Bucket Detection
Hardcoded AWS S3, GCS, Azure Blob, DigitalOcean Spaces, Backblaze URLs.

## Automation Pipeline
1. Subfinder -> naabu -> httpx -> nuclei (standard)
2. For each live host: crawl JS -> extract endpoints/secrets -> probe endpoints -> nuclei validation