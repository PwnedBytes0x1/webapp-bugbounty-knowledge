# SQL Injection

## Types

| Type | Description |
|------|-------------|
| Error-based | Extract info via database error messages |
| Boolean-based | True/false response differences |
| Time-based | Response delay (SLEEP(5)) |
| UNION-based | Direct data extraction |
| Out-of-Band | DNS/HTTP callbacks for data exfil |
| Second-order | Stored then triggered |

## Detection Payloads

Basic test characters: ' " ) ') ") ` )`

Boolean tests:
' OR 1=1 --
' AND 1=1 --
' AND 1=2 --

Time-based:
; WAITFOR DELAY '0:0:5' --
OR SLEEP(5) --
AND SLEEP(5) --

## Tools
- SQLMap - Full automation
- Burp Suite Scanner
- jSQL Injection

## Prevention
- Parameterized queries (prepared statements)
- ORM frameworks (with proper use)
- Input validation
- Least privilege database accounts
