# Cloud Security

## AWS Security Testing

### IAM Privilege Escalation
- **PassRole** — Attach admin role to EC2 instance
- **CreatePolicy** — Create admin policy
- **SetExistingDefaultPassword** — Change default password
- **UpdateLoginProfile** — Change IAM user password

### S3 Security
- Bucket policy misconfiguration (public read/write)
- Object ACL misconfiguration
- S3 access point misconfiguration
- Server access logging disabled
- Default encryption not enabled

### Lambda Security
- **Event source mapping injection** — Control event sources
- **Lambda function URL exposed** — Unauthenticated access
- **Environment variable secrets** — Plaintext in env vars
- **Overprivileged execution role** — Lambda with full admin

### EC2 Security
- **SSRF to IMDS** — Access instance metadata (credentials)
- **User data leakage** — Scripts in user data contain secrets
- **Security group misconfiguration** — Overly permissive inbound rules
- **Unencrypted EBS volumes** — Data at rest leakage

### AWS Testing Tools
| Tool | Purpose |
|------|---------|
| **Pacu** | AWS exploitation framework |
| **ScoutSuite** | Multi-cloud audit tool |
| **CloudSploit** | Cloud security scanning |
| **S3Scanner** | S3 bucket enumeration |
| **Principal Mapper** | IAM privilege escalation paths |

## GCP Security Testing

### GCP Enumeration
- **IAM policy misconfiguration** — Overly permissive roles
- **Cloud Storage buckets** — Public bucket access
- **Cloud Functions** — Unauthenticated invocation
- **KMS key management** — Weak key rotation policies
- **Firestore / Datastore** — Unsecured databases

### GCP Testing Tools
| Tool | Purpose |
|------|---------|
| **GCPBucketBrute** | GCP bucket enumeration |
| **gcp_scanner** | GCP asset discovery |
| **CloudSploit** | Multi-cloud scanning |

## Azure Security Testing

### Azure Enumeration
- **Azure Blob Storage** — Public blob containers
- **Azure VMs** — Open management ports
- **Azure AD** — Conditional access misconfiguration
- **Managed Identity** — SSRF via IMDS endpoint
- **Key Vault** — Permissive access policies

### Azure Testing Tools
| Tool | Purpose |
|------|---------|
| **MicroBurst** | Azure security testing |
| **Stormspotter** | Azure recon graph database |
| **AzureHound** | BloodHound for Azure AD |
| **ScoutSuite** | Multi-cloud audit |

## Cloud Metadata SSRF

AWS Instance Metadata Service (IMDSv1):
```bash
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl http://169.254.169.254/latest/user-data/
```

GCP Metadata:
```bash
curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/
curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
```

Azure IMDS:
```bash
curl -H "Metadata: true" "http://169.254.169.254/metadata/instance?api-version=2021-02-01"
curl -H "Metadata: true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"
```

## Cloud Security Best Practices
1. **Enable IMDSv2** (AWS) — Session-based, prevents SSRF attacks
2. **Least privilege IAM** — No wildcard permissions
3. **Bucket policies** — Block public access by default
4. **Encryption** — Enable at rest and in transit
5. **Monitor** — CloudTrail, GuardDuty, Security Hub
6. **Secure CI/CD** — No hardcoded credentials in pipelines
7. **Network segmentation** — Private subnets, security groups
8. **Backup and disaster recovery** — With encryption

---

> **Next**: [Mobile Application Testing](02-mobile-testing.md)
