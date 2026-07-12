# GitHub Reconnaissance: Complete Methodology

## Organization Enumeration

```bash
# Find organization's repos
github-subdomains -d target.com -t $GITHUB_TOKEN > github_subdomains.txt

# Use GitHub's search API
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/search/users?q=target.com+type:org" | jq '.items[].login'

# Search for org with employee count
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/search/users?q=target+company+location&type:users" | jq '.items[].login'
```

## Secret Scanning Dorks

```bash
# Medium severity: API keys, tokens, passwords
org:target "api_key" NOT "example"
org:target "password" NOT "example"
org:target "secret" NOT "example"
org:target "-----BEGIN RSA PRIVATE KEY-----"
org:target "-----BEGIN OPENSSH PRIVATE KEY-----"
org:target "jdbc:mysql://"
org:target "mongodb://"
org:target "postgresql://"
org:target "connection_string"
org:target "JWT_SECRET"
org:target "SECRET_KEY"
org:target "AWS_SECRET_ACCESS_KEY"

# High severity: tokens embedded in code
org:target "ghp_"  # GitHub personal access tokens
org:target "ghs_"  # GitHub OAuth tokens
org:target "xox[pborsa]-[0-9]"  # Slack tokens
org:target "sk_live_"  # Stripe live keys
org:target "pk_live_"  # Stripe publishable live keys
org:target "AKIA[0-9A-Z]{16}"  # AWS access keys
org:target "AIza[0-9A-Za-z_-]{35}"  # Google API keys
```

## CI/CD Pipeline Analysis

```bash
# GitHub Actions config
org:target .github/workflows
org:target filename:.github/workflows

# Jenkinsfile
org:target filename:Jenkinsfile
org:target "jenkins" "password" "credentials"

# CircleCI
org:target filename:.circleci/config.yml
org:target "CIRCLE_TOKEN"

# Travis CI
org:target filename:.travis.yml
org:target "secure:" "api_key"  # Encrypted env vars

# Dockerfiles
org:target filename:Dockerfile
org:target filename:Dockerfile ENV password

# Kubernetes manifests
org:target filename:k8s-deployment.yaml
org:target filename:deployment.yaml "image:"
```

## Git History Mining

```bash
# Search commit messages
curl -s -H "Authorization: token $TOKEN" \
  "https://api.github.com/search/commits?q=org:target+password" | jq '.items[].html_url'

# Search commit diffs for removed secrets
# Secrets are often pushed then later removed (still in git history)
git log -p --all -S "password" --source

# Common patterns found in history
# - Hardcoded credentials that were later env-var-ized
# - Debug endpoints / dev URLs
# - TODO comments that reference security issues
# - Internal IP addresses / hostnames
# - Old API keys that still work because rotation was missed
```

## Personal Account Analysis

```bash
# Developer personal repos with work references
# Search: "target.com" "security" author:developer
# Search: "target" in:readme author:developer
# Search: filename:.npmrc author:developer

# Dotfile repos (often contain credentials)
org:target filename:.env
org:target filename:.bashrc
org:target filename:.bash_profile
org:target filename:.gitconfig

# .gitignore files may reveal what secrets devs are trying to protect
org:target filename:.gitignore
```

## Automated Tools

```bash
# gitLeaks (local scan)
gitleaks detect --source=/path/to/repo --verbose

# truffleHog (git history scan)
trufflehog git https://github.com/target/repo.git --only-verified

# GitDorker (automated GitHub dorking)
python3 GitDorker.py -t $TOKEN -q target.com -d Dorks/alldorksv3

# GitHub-Dork (python)
python3 github-dork.py -d target -t $TOKEN -o results/

# SecretFinder extended
# Test every discovered secret against public APIs
# AWS keys -> validate with STS GetCallerIdentity
# Stripe keys -> validate with Stripe API
```

## Post-Discovery Actions

Once secrets are found:
1. Verify they are still active (don't report expired keys)
2. Document exact files and line numbers
3. Screenshot with URL visible
4. Report to program security team
5. NEVER reuse found credentials
6. Revocation is the responsibility of the program
