---
name: waf-operational-excellence
description: AWS Well-Architected Operational Excellence pillar review criteria. Use when evaluating architecture for observability, automation, IaC, CI/CD, runbooks, and deployment strategies.
---

# Operational Excellence — Review Criteria

## Checklist

### Observability
- [ ] Centralized logging (CloudWatch Logs, OpenSearch)
- [ ] Distributed tracing (X-Ray, OpenTelemetry)
- [ ] Custom metrics and dashboards (CloudWatch Metrics, Grafana)
- [ ] Alerting with actionable thresholds (CloudWatch Alarms → SNS → PagerDuty)
- [ ] Structured logging with correlation IDs

### Infrastructure as Code
- [ ] All infrastructure defined in code (Terraform, CDK, CloudFormation)
- [ ] State management (S3 backend, locking via DynamoDB)
- [ ] Module reuse and versioning
- [ ] Drift detection enabled

### CI/CD Pipeline
- [ ] Automated build, test, deploy pipeline
- [ ] Separate stages (dev → staging → prod)
- [ ] Approval gates before production
- [ ] Automated rollback on failure
- [ ] Pipeline-as-code (not ClickOps)

### Deployment Strategy
- [ ] Blue/green or canary deployments
- [ ] Feature flags for gradual rollout
- [ ] Automated health checks post-deploy
- [ ] Deployment frequency tracked

### Runbooks & Incident Response
- [ ] Documented runbooks for common failures
- [ ] On-call rotation defined
- [ ] Incident communication channels established
- [ ] Post-incident review process (blameless retrospectives)

## Common Anti-Patterns

| Anti-Pattern | Risk | Fix |
|---|---|---|
| No centralized logging | Can't troubleshoot cross-service issues | CloudWatch Logs + Log Insights |
| Manual deployments (SSH + scripts) | Human error, no audit trail | CodePipeline / GitHub Actions |
| No rollback plan | Extended outages on bad deploy | Blue/green with auto-rollback |
| Alerts on everything | Alert fatigue, missed real issues | Actionable alerts with runbook links |
| No IaC, resources created in console | Drift, unreproducible environments | Terraform import + prevent ClickOps |

## Key AWS Services

- **CloudWatch**: Logs, Metrics, Alarms, Dashboards, Synthetics
- **X-Ray**: Distributed tracing, service maps
- **Systems Manager**: Automation runbooks, Parameter Store
- **CodePipeline / CodeDeploy**: CI/CD with deployment strategies
- **CloudFormation / CDK**: Infrastructure as Code
- **Config**: Drift detection, compliance rules

## Example Findings

**Good**: "ECS services use CodeDeploy blue/green deployments with automatic rollback on CloudWatch alarm trigger"

**Bad**: "Lambda functions deployed manually via console with no version aliases or traffic shifting"
