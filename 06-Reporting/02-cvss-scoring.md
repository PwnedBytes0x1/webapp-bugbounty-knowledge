# CVSS Scoring

Common Vulnerability Scoring System (CVSS) provides a standardized way to rate vulnerability severity. Understanding CVSS helps you communicate the impact of your findings effectively.

## CVSS v3.1 Base Metrics

### Attack Vector (AV)
| Value | Description |
|-------|-------------|
| **Network (N)** | Exploitable remotely |
| **Adjacent (A)** | Same network segment |
| **Local (L)** | Requires local access |
| **Physical (P)** | Requires physical access |

### Attack Complexity (AC)
| Value | Description |
|-------|-------------|
| **Low (L)** | No special conditions |
| **High (H)** | Requires specific conditions |

### Privileges Required (PR)
| Value | Description |
|-------|-------------|
| **None (N)** | No authentication |
| **Low (L)** | Basic user |
| **High (H)** | Admin privileges |

### User Interaction (UI)
| Value | Description |
|-------|-------------|
| **None (N)** | No user action needed |
| **Required (R)** | User must perform action |

### Scope (S)
| Value | Description |
|-------|-------------|
| **Unchanged (U)** | Affects same component |
| **Changed (C)** | Affects different component |

### Confidentiality (C), Integrity (I), Availability (A)
| Value | Description |
|-------|-------------|
| **None (N)** | No impact |
| **Low (L)** | Limited impact |
| **High (H)** | Complete loss |

## Severity Ratings

| Score Range | Severity |
|-------------|----------|
| 0.0 | None |
| 0.1-3.9 | Low |
| 4.0-6.9 | Medium |
| 7.0-8.9 | High |
| 9.0-10.0 | Critical |

## Common Vulnerability Scoring Examples

### SQLi (typically 8.0-9.0)
```
AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H = 9.8 (Critical)
```

### Stored XSS (typically 5.0-7.0)
```
AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N = 5.4 (Medium)
```

### IDOR (typically 5.0-8.0)
```
AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N = 6.5 (Medium)
```

### SSRF (typically 6.0-9.0)
```
AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:N/A:N = 8.6 (High)
```

### RCE (typically 9.0-10.0)
```
AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H = 9.8 (Critical)
```

## CVSS Calculator

- **NVD CVSS Calculator**: https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator
- **FIRST CVSS Calculator**: https://www.first.org/cvss/calculator/3.1

## Bug Bounty vs Enterprise CVSS

Bug bounty programs often use modified severity scales:
- Many ignore Availability impact
- Scope (S) changes are weighted differently
- Some use a 4-tier system without decimals

## Pro Tips

1. **Be conservative** — Underscoring is better than overscoring
2. **Focus on business impact** — CVSS is a guideline, not absolute
3. **Include both CVSS and plain language** — "This is critical because..."
4. **Consider chaining** — If this bug enables other attacks, note that

---

> **Next**: [PoC Creation](03-poc-creation.md)
