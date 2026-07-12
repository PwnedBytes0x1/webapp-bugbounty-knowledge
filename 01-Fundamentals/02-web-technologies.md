# Web Technologies Reference

## Backend Frameworks
| Framework | Language | Common Headers | Cookiejar |
|-----------|----------|----------------|-----------|
| Django | Python | X-Frame-Options: DENY | csrftoken, sessionid |
| Flask | Python | - | session |
| Express | Node.js | X-Powered-By: Express | connect.sid |
| Next.js | Node.js | x-nextjs-page | next-auth.session-token |
| Spring Boot | Java | X-Application-Context | JSESSIONID |
| Rails | Ruby | X-Runtime, X-Request-Id | _session_id |
| Laravel | PHP | X-Powered-By: PHP/8.x | laravel_session |
| ASP.NET | C# | X-AspNet-Version | .ASPXAUTH |

## CDN Detection
- Cloudflare: `cf-ray`, `cf-cache-status`, `server: cloudflare`
- Akamai: `x-akamai-transformed`, `server: AkamaiGHost`
- Fastly: `x-served-by`, `x-cache`, `x-timer`
- CloudFront: `x-amz-cf-id`, `x-amz-cf-pop`

## Load Balancers
- HAProxy: `x-forwarded-for`, `x-forwarded-proto`
- Nginx: `server: nginx/1.x.x`
- AWS ALB: `x-amzn-trace-id`, `server: awselb/2.0`
- F5: `x-request-id`, `server: BigIP`

## WAF Detection (wafw00f)
- Cloudflare: 403 with cf-* headers
- ModSecurity: 406 Not Acceptable
- AWS WAF: 403 with x-amzn-ErrorType
- Imperva: 509 Bandwidth Limit Exceeded