# JavaScript Analysis: Complete Guide

## JS File Collection

```bash
# From historical URLs
cat historical_urls.txt | grep -E '\.js($|\?)' | sort -u > js_files.txt

# From crawling
cat live_hosts.txt | katana -jc -kf all -silent | grep -E '\.js($|\?)' | sort -u >> js_files.txt

# From source map (sourcemap.js)
# /static/js/main.abc123.js.map
curl -s https://target.com/static/js/main.abc123.js.map | jq -r '.sources[]'

# JS files in Wayback
cat historical_urls.txt | grep -E '\.js($|\?)' | sort -u > wayback_js.txt
```

## Endpoint Extraction

```bash
# LinkFinder (comprehensive)
python3 linkfinder.py -i https://target.com/app.js -o cli

# xnLinkFinder (fast, multi-threaded)
python3 xnLinkFinder.py -i https://target.com/app.js -d 2 -sp 'example.com'

# JSLuice (Go, very fast)
jsluice extract https://target.com/app.js
jsluice secrets https://target.com/app.js

# Manual grep for endpoints
cat all_js_files.txt | grep -ohP '"[a-zA-Z0-9_\/.-]{3,}"' | sort -u
cat all_js_files.txt | grep -ohP "'[a-zA-Z0-9_\/.-]{3,}'" | sort -u
cat all_js_files.txt | grep -ohP '`/[a-zA-Z0-9_\/.-]+`' | sort -u

# API endpoint patterns
cat all_js_files.txt | grep -E 'api|v1|v2|graphql|rest' | grep -oP '("|\'|`)(/[a-zA-Z0-9_\/.-]{3,})("|\'|`)' | sort -u
```

## Secret Discovery

```bash
# SecretFinder
python3 SecretFinder.py -i https://target.com/app.js -o cli

# Detect Secrets (extensive rules)
detect-secrets scan --all-files --no-verify js_files/

# TruffleHog (for JS from web)
trufflehog filesystem --directory=js_files/ --no-verification

# Manual secret patterns to grep
cat all_js.txt | grep -E '"AKIA[0-9A-Z]{16}"'  # AWS Access Key
cat all_js.txt | grep -E 'sk_live_|pk_live_'  # Stripe keys
cat all_js.txt | grep -E 'ghp_[a-zA-Z0-9]{36}'  # GitHub tokens
cat all_js.txt | grep -E 'AIza[0-9A-Za-z_-]{35}'  # Google API keys
cat all_js.txt | grep -E 'xox[baprs]-[0-9]{12}-[0-9]{12}'  # Slack tokens
cat all_js.txt | grep -E 'eyJ[a-zA-Z0-9_-]{10,}\.eyJ[a-zA-Z0-9_-]{10,}'  # JWT tokens
cat all_js.txt | grep -E 'api[Kk]ey|apikey|api_key|secret|token|password|auth'  # Generic
```

## API Route Reconstruction

```javascript
// Build URL templates from JS
// Pattern: const API_BASE = '/api/v2/'
// Then: fetch(API_BASE + 'users/' + userId)
// = /api/v2/users/{userId}

// GraphQL introspection responses
// If app.js contains gql`query` or `buildSchema`
// Server likely supports introspection

// i18n / translation files
// Usually contain all route names the app knows
// /assets/i18n/en.json -> map of all frontend routes
```

## SPA Route Analysis

```bash
# Extract React router paths
cat app.js | grep -oP 'path: ["\'`]([\w\/:-]+)["\'`]' | sort -u
cat app.js | grep -oP 'route="[^"]+"' | sort -u
cat app.js | grep -oP "route='[^']+'" | sort -u

# Vue router
cat app.js | grep -oP "path: ['\"`].+?['\"`]" | sort -u

# Angular routes
cat app.js | grep -oP "path: ['\"`].+?['\"`]" | sort -u

# SPA routes often include:
# - Admin panels: /admin, /dashboard, /settings
# - Hidden features: /beta, /experimental, /internal
# - Debug endpoints: /debug, /test, /sandbox
# - Configuration: /config, /env, /feature-flags
```

## Source Map Analysis

```bash
# Source maps reveal minified code in original form
curl -s https://target.com/static/js/main.js.map > main.js.map
# Check for .map files (often present in production by accident)

# Extract from source map
jq -r '.sources[]' main.js.map  # List all original files
jq -r '.sourcesContent[]' main.js.map > source_content.txt

# Tools
node-easy-sourcemap/main.py -d https://target.com/static/js/main.js
# Source map can leak original code, internal names, dev comments
```

## React DevTools & Component Analysis

```bash
# Extract React component names
cat app.js | grep -oP "displayName: ['\"`][\w]+['\"`]" | sort -u

# Redux actions (state management side effects)
cat app.js | grep -oP "type: ['\"`][\w]+['\"`]" | sort -u

# Feature flags
cat app.js | grep -E 'featureFlag|feature_flag|featureToggle|isEnabled' | sort -u
```
