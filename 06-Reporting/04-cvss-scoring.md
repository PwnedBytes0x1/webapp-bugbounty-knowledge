# CVSS v3.1 Scoring Guide

## Base Metrics

### Attack Vector (AV)
- **Network** (N): Remotely exploitable
- **Adjacent** (A): Same network segment
- **Local** (L): Requires local access
- **Physical** (P): Physical interaction

### Attack Complexity (AC)
- **Low** (L): No special conditions
- **High** (H): Requires specific conditions

### Privileges Required (PR)
- **None** (N): Unauthenticated
- **Low** (L): Regular user
- **High** (H): Admin privileges

### User Interaction (UI)
- **None** (N): No victim action
- **Required** (R): Victim must perform action

### Scope (S)
- **Unchanged** (U): Component stays same
- **Changed** (C): Impacts different component

### Confidentiality / Integrity / Availability (C/I/A)
- **None** (N): No impact
- **Low** (L): Limited disclosure
- **High** (H): Full compromise

## Example Vectors

### Critical RCE
`CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` (9.8)

### Stored XSS (admin panel)
`CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N` (5.4)

### Reflected XSS
`CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N` (6.1)

### SQLi
`CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` (9.8)

### SSRF (cloud metadata)
`CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:N/A:N` (8.8)