# Technology Fingerprinting

## HTTP Headers
```bash
curl -sI https://target.com | grep -iE "server|x-powered|x-aspnet|x-runtime|x-generator"
```

## Framework Detection
| Header | Technology |
|--------|------------|
| X-Powered-By: Express | Node.js/Express |
| X-Runtime: 0.01234 | Ruby on Rails |
| X-AspNet-Version | ASP.NET |
| Server: nginx/1.18.0 | Nginx |
| X-Frame-Options: DENY | Django |

## JS Framework Detection
- React: `data-reactroot`, `data-reactid`, `__NEXT_DATA__`
- Vue: `data-v-xxxxxx`, `__NUXT__`
- Angular: `ng-version`, `_nghost-xxx`

## CMS Detection
- WordPress: `/wp-content/`, `/wp-admin/`, `wp-json`
- Joomla: `/components/`, `/modules/`, `/media/system/`
- Drupal: `/sites/default/`, `/core/`, `/modules/`

## WAF Detection
```bash
wafw00f https://target.com
# Common WAF indicators
# Cloudflare: cf-ray, __cfduid
# ModSecurity: 406 Not Acceptable
# AWS WAF: Request blocked by AWS WAF
```

## Version Info
```bash
# Scan for version disclosure
curl -sk https://target.com/ | grep -iE "version|powered|generator"
```
