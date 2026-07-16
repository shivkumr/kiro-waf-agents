---
name: waf-security
description: AWS Well-Architected Security pillar review criteria. Use when evaluating architecture for IAM, network isolation, encryption, threat detection, and secrets management.
---

# Security — Review Criteria

## Checklist

### Identity & Access Management
- [ ] Least privilege IAM policies (no `*` actions or resources)
- [ ] No long-lived access keys; use IAM roles and instance profiles
- [ ] MFA enforced for human users
- [ ] Service-to-service auth via IAM roles (not shared secrets)
- [ ] SCPs applied at Organization level for guardrails
- [ ] Regular access reviews and credential rotation

### Network Security
- [ ] VPC with private and public subnet separation
- [ ] Security groups: deny by default, minimal ingress rules
- [ ] NACLs for subnet-level defense-in-depth
- [ ] No direct internet access for backend services
- [ ] VPC endpoints for AWS service access (no internet path)
- [ ] Network Firewall or third-party IDS/IPS for inspection

### Data Protection
- [ ] Encryption at rest (KMS-managed keys for S3, RDS, EBS, DynamoDB)
- [ ] Encryption in transit (TLS 1.2+ everywhere)
- [ ] S3 Block Public Access enabled at account level
- [ ] Database connections via TLS, not plaintext
- [ ] Backup encryption enabled

### Secrets Management
- [ ] Secrets in AWS Secrets Manager or Parameter Store (SecureString)
- [ ] No hardcoded credentials in code, configs, or environment variables
- [ ] Automatic rotation configured
- [ ] Secrets referenced by ARN, not value

### Detection & Response
- [ ] GuardDuty enabled in all regions
- [ ] Security Hub with compliance standards (CIS, PCI-DSS)
- [ ] CloudTrail enabled with log file validation
- [ ] VPC Flow Logs enabled
- [ ] AWS Config rules for continuous compliance
- [ ] Incident response plan documented

### Edge Protection
- [ ] WAF on ALB/CloudFront with rate limiting
- [ ] Shield Advanced for DDoS protection (if critical)
- [ ] Bot control rules for public-facing APIs
- [ ] Geo-restriction if applicable

## Common Anti-Patterns

| Anti-Pattern | Risk | Fix |
|---|---|---|
| `Action: "*", Resource: "*"` in IAM | Full account compromise if creds leak | Scope to specific actions and resources |
| Security groups with 0.0.0.0/0 ingress | Attack surface exposure | Restrict to known CIDR ranges or VPC |
| S3 buckets without Block Public Access | Data breach | Account-level S3 Block Public Access |
| Hardcoded DB password in Lambda env vars | Credential exposure in console/logs | Secrets Manager with rotation |
| No GuardDuty or Security Hub | Blind to threats | Enable in all regions, aggregate to security account |
| No WAF on public ALB | Vulnerable to OWASP Top 10 | AWS WAF with managed rule groups |

## Key AWS Services

- **IAM**: Roles, Policies, Identity Center (SSO)
- **KMS**: Key management, automatic rotation
- **Secrets Manager**: Secret storage, auto-rotation
- **GuardDuty**: Threat detection
- **Security Hub**: Posture management, compliance
- **WAF**: Web application firewall
- **Shield**: DDoS protection
- **CloudTrail**: API audit logging
- **VPC**: Network isolation, endpoints, flow logs
- **Config**: Compliance rules, remediation

## Example Findings

**Good**: "RDS instance in private subnet, encrypted at rest with CMK, connections enforced via TLS, credentials in Secrets Manager with 30-day rotation"

**Bad**: "EC2 instance with public IP, security group allowing 0.0.0.0/0 on port 22, IAM role with AdministratorAccess policy attached"
