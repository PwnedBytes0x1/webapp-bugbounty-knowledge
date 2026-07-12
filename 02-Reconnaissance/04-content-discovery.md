# Content Discovery

## Directory Bruteforce
```bash
feroxbuster -u https://target.com -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt -t 50 -o dirs.txt
gobuster dir -u https://target.com -w common.txt -t 50
dirsearch -u https://target.com -e php,asp,aspx,jsp,html,txt
```

## File Discovery
```bash
ffuf -u https://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt
ffuf -u https://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt
```

## Parameter Discovery
```bash
arjun -u https://target.com/api/endpoint
x8 -u https://target.com/api/endpoint -w params.txt
```

## Hidden Parameters
```bash
# Common hidden params
debug, test, source, action, method, mode, type, format, callback
```

## GraphQL Discovery
```bash
# Endpoints to fuzz
/graphql, /graphiql, /playground, /v1/graphql, /gql, /query
# Field suggestion
{"query": "{__schema{types{name}}}"}
```

## API Discovery
```bash
# Common API paths
ffuf -u https://target.com/api/FUZZ -w api-endpoints.txt
ffuf -u https://target.com/FUZZ -w api-versions.txt -mr "swagger|openapi|redoc"
```
