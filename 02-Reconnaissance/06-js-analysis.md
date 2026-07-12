# JavaScript Analysis

## Endpoint Extraction
```bash
# LinkFinder
python3 linkfinder.py -i https://target.com/app.js -o cli

# JSParser
python3 JSParser.py -u https://target.com/

# Custom extraction
cat app.js | grep -Eo "https?://[^"'\s]+" | sort -u
cat app.js | grep -oP '/api/[^"'\s)]+' | sort -u
```

## Secret Detection
```bash
# TruffleHog
trufflehog filesystem app.js --json | jq

# Custom patterns
grep -E '(api[_-]?key|secret|token|password|access[_-]?key)' app.js
grep -E '(ghp_|gho_|github_pat_)' app.js  # GitHub tokens
```

## Source Map Analysis
```bash
curl https://target.com/app.js.map
node -e "m=require('source-map');f=require('fs');c=f.readFileSync('app.js.map');m.SourceMapConsumer.with(c,null,function(c){c.eachMapping(function(s){console.log(s.originalLine,s.originalColumn,s.name)})})"
```

## SPA Route Discovery
```bash
# Next.js
curl https://target.com/_next/static/chunks/pages/

# React
grep -r "path:" app.js | sort -u

# Vue
grep -r "path:" app.js | grep -v node_modules
```

## API Key Patterns
```
google_api_key: AIza[0-9A-Za-z\-_]{35}
aws_key: AKIA[0-9A-Z]{16}
github_token: ghp_[0-9a-zA-Z]{36}
slack_token: xox[baprs]-[0-9a-zA-Z]{10,}
firebase_url: https://[a-z0-9-]+.firebaseio.com
```
