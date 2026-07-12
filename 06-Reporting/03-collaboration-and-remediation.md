# Collaboration & Remediation: After Finding Bugs

## During Triage

### Triage Response Time
- Most programs: 1-5 business days
- Critical vulns: triaged within 24 hours
- VDP-only programs: may take weeks

### Common Triage Questions
- "Can you provide more detail on the impact?"
- "Does this require user interaction?"
- "Can you reproduce on latest production?"
- "Is there an authentication requirement?"

Be responsive: delayed responses can lead to duplicate or N/A findings.

## When to Retest
- Wait for "Fixed" status from the team
- Retest the same endpoint with same technique
- Test for incomplete fixes (bypasses)
- Report bypass as a separate finding (or comment on original, depending on program rules)

## Collaboration with Dev Teams

### Effective Communication
- Provide clear reproduction steps
- Offer to hop on a call for complex exploits
- Be patient: devs may not have security background
- Explain "why" not just "what"

### When They Push Back
- "This is intended behavior" -> validate against similar programs
- "This requires authenticated access" -> explain if auth bypass/CSRF chains
- "This is out of scope" -> check scope.md, move on if confirmed

## Handling Duplicates
- Search thoroughly before reporting (use this repo's checklist)
- If marked duplicate, ask which field/bug it duplicates (if allowed)
- No point arguing duplicates - spend time on new findings

## Remediation Verification

### Check for Complete Fix
```bash
# Original payload
curl 'https://target.com/search?q=%3Cscript%3Ealert(1)%3C/script%3E'
# Look for successful exploitation
curl -X POST https://target.com/api/search -d '{"q":"<script>alert(1)</script>"}'
```

### Bypass Attempts
- Try different encodings (URL, Unicode, double encoding)
- Try alternate parameters with same function
- Try different HTTP methods
- Try different Content-Type headers
