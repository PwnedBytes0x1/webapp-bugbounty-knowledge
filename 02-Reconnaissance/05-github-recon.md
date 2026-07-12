# GitHub Recon

GitHub recon searches for sensitive data in repositories, commit history, and issues.

## Search Techniques
```
org:target "api_key"
org:target "password"
org:target "-----BEGIN PRIVATE KEY-----"
org:target ".env" "DB_PASSWORD"
org:target "secret"
```

## Tools
| Tool | Description |
|------|-------------|
| gitleaks | Git repo secret scanning |
| trufflehog | Deep commit history scanning |
| gitrob | Organization file analysis |
| git-hound | Pattern-based GitHub search |

## Commit History Analysis
```bash
git log --all --oneline | grep -i 'password|secret|key|token'
git log -p --all -S "password" --pretty=format:"%h %s"
```

## Considerations
- GitHub API limits: 60/hr unauthenticated, 5000/hr authenticated
- Use a dedicated PAT (read-only for public repos)
- Public repo scanning is generally acceptable

---

> **Next**: [Cloud Asset Discovery](06-cloud-asset-discovery.md)
