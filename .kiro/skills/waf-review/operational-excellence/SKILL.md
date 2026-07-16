---
name: waf-operational-excellence
description: AWS Well-Architected Operational Excellence pillar. Use when reviewing, generating, or validating architecture for observability, automation, IaC, CI/CD, runbooks, and deployment strategies.
---

# Operational Excellence

## Checklist (for reviewers/validators)

### Observability
- [ ] Centralized logging (CloudWatch Logs, OpenSearch)
- [ ] Distributed tracing (X-Ray, OpenTelemetry)
- [ ] Custom metrics and dashboards (CloudWatch Metrics, Grafana)
- [ ] Alerting with actionable thresholds (CloudWatch Alarms → SNS)
- [ ] Structured logging with correlation IDs

### Infrastructure as Code
- [ ] All infrastructure defined in code (Terraform, CDK, CloudFormation)
- [ ] State management (S3 backend with locking)
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
- [ ] Post-incident review process

## Implementation Guide (for writers/generators)

### Observability

- Every compute resource (ECS, Lambda, EC2) MUST send logs to a CloudWatch Log Group
- Set log retention explicitly (30 days for dev, 90 for prod) — never use "never expire"
- Encrypt log groups with a dedicated KMS key
- Name log groups consistently: `/{compute-type}/{service-name}/{environment}`
- Enable X-Ray tracing on API Gateway, Lambda, and ECS tasks
- Create CloudWatch dashboards per service showing: error rate, latency p99, invocation count
- Every alarm MUST have an SNS topic action — no orphaned alarms
- Use structured JSON logging with fields: timestamp, level, correlationId, service, message

### Infrastructure as Code

- Use remote state backend (S3 + DynamoDB lock table) — never local state
- Encrypt the state bucket with a dedicated KMS key
- One state file per service per environment (not monolithic)
- Use module composition: separate modules for networking, compute, database, monitoring
- Pin module versions explicitly (no floating "latest")
- Include `terraform.tfvars.example` showing required variables without real values
- Add a `README.md` per module explaining inputs, outputs, and architecture decisions

### CI/CD & Deployment

- Define pipeline in code (GitHub Actions YAML, CodePipeline CDK, etc.)
- Minimum stages: lint → plan → apply-dev → test → approve → apply-prod
- Use blue/green for stateless services (ECS, Lambda)
- Use rolling updates for stateful services (RDS, ElastiCache) only via managed mechanisms
- Configure auto-rollback triggered by CloudWatch alarm breaches
- Set deployment circuit breaker on ECS services (min healthy 100%, max 200%)
- Track deployment frequency and lead time as operational metrics

### Diagram Components

When generating architecture diagrams:
- Show CloudWatch connected to all compute resources (logs + metrics)
- Show X-Ray trace lines between services
- Include CI/CD pipeline component (CodePipeline or GitHub Actions)
- Show SNS topic for alarm notifications
- Include Systems Manager if automation/patching is relevant

## Common Anti-Patterns

| Anti-Pattern | Risk | Fix |
|---|---|---|
| No centralized logging | Can't troubleshoot cross-service issues | CloudWatch Logs + Log Insights |
| Manual deployments | Human error, no audit trail | Pipeline-as-code with approval gates |
| No rollback plan | Extended outages on bad deploy | Blue/green with alarm-triggered rollback |
| Alerts on everything | Alert fatigue, missed real issues | Actionable alerts with runbook links |
| Console-created resources | Drift, unreproducible environments | IaC + drift detection via Config |

## Key AWS Services

- **CloudWatch**: Logs, Metrics, Alarms, Dashboards, Synthetics
- **X-Ray**: Distributed tracing, service maps
- **Systems Manager**: Automation runbooks, Parameter Store
- **CodePipeline / CodeDeploy**: CI/CD with deployment strategies
- **CloudFormation / CDK / Terraform**: Infrastructure as Code
- **Config**: Drift detection, compliance rules
