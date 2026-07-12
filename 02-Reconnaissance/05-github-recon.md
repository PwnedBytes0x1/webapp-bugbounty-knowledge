# GitHub Recon: Dorking & Leak Discovery

## Repository Discovery

```bash
# Organization repositories
gh repo list target-org --limit 100 --json name,visibility
# Search for target mentions in code
gh search code "target.com" --limit 100 --owner:target-org
```

## Advanced Git Dorking

### High-Value Dorks
```bash
# API keys and secrets
org:target "api_key" OR "api_secret" OR "api-key"
org:target "password" OR "passwd" OR "pwd" language:yaml
org:target "-----BEGIN RSA PRIVATE KEY-----"
org:target filename:.npmrc OR filename:.env OR filename:.s3cfg
org:target "AWS_ACCESS_KEY_ID" OR "AWS_SECRET_ACCESS_KEY"
org:target "smtp" OR "smtp_password" in:file

# Configuration exposure
org:target filename:config.js OR filename:config.php
org:target filename:database.yml extension:yml
org:target filename:.htaccess OR filename:web.config
org:target "connectionString" OR "connection_string"

# Endpoint exposure
org:target "staging" OR "dev" OR "test" in:file
org:target "localhost" OR "127.0.0.1" OR "0.0.0.0" in:file
org:target "internal" AND "api" in:file
```

### Commit History Leaks

```bash
# Search commit messages for leaked creds
gh search commits "password" --repo target-org/target-repo
# Check commit diffs for removed secrets
gh api repos/target-org/target-repo/commits --paginate | jq -r '.[].sha' |   xargs -I{} sh -c 'gh api repos/target-org/target-repo/commits/{}/files | jq -r ".[].filename"'
```

## Fork Analysis

Forks often have their own secrets not present in the parent:

```bash
# List all forks
gh api repos/target-org/target-repo/forks --paginate | jq -r '.[].full_name'
# Check each fork for sensitive data in branches not in the main repo
```

## Automated Monitoring

```bash
# Set up git-hound for continuous scanning
git-hound --subdomain-file subs.txt --dig-files --dig-commits --regex-file patterns.yml
# Or use truffleHog on cloned repos
trufflehog --regex --entropy=False https://github.com/target-org/target-repo.git
```

## Tungsten — Commit-by-commit analysis

For cloned repos, walk each commit for diff-introduced secrets:

```bash
git log --all --pretty=format:"%H %an %s" | while read hash author subject; do
  git diff-tree --no-commit-id -r --diff-filter=A $hash | grep -E 'api|secret|password|key|token' && echo "COMMIT: $hash - $author - $subject"
done
```

## Gist Discovery

```bash
# Search GitHub Gists for target.com
gh api /search/code?q=target.com+extension:txt+gist=true
# Gists are often excluded from org-level scanning but still leak data
```
