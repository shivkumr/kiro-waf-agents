---
name: waf-reliability
description: AWS Well-Architected Reliability pillar review criteria. Use when evaluating architecture for Multi-AZ/Region, auto-scaling, DR, fault isolation, and self-healing.
---

# Reliability — Review Criteria

## Checklist

### Multi-AZ & Multi-Region
- [ ] Compute distributed across 2+ Availability Zones
- [ ] Database Multi-AZ enabled (RDS, ElastiCache, OpenSearch)
- [ ] Load balancer spans multiple AZs
- [ ] Critical workloads have multi-region DR strategy
- [ ] DNS failover configured (Route 53 health checks)

### Auto-Scaling & Self-Healing
- [ ] Auto Scaling Groups with appropriate min/max/desired
- [ ] Target tracking or step scaling policies defined
- [ ] Health checks configured (ELB + EC2 or custom)
- [ ] Unhealthy instances automatically replaced
- [ ] ECS/EKS service auto-scaling configured

### Backup & Disaster Recovery
- [ ] RPO and RTO defined and documented
- [ ] Automated backups enabled (RDS, DynamoDB, EBS snapshots)
- [ ] Cross-region backup replication for critical data
- [ ] Backup restoration tested regularly
- [ ] DR runbook documented and drilled

### Fault Isolation
- [ ] Services loosely coupled via queues/events (not direct calls)
- [ ] Bulkhead pattern: failure in one component doesn't cascade
- [ ] Circuit breakers for downstream dependencies
- [ ] Timeout and retry with exponential backoff configured
- [ ] Graceful degradation (serve stale cache, feature flags)

### Dependency Management
- [ ] External dependency failures handled (third-party APIs)
- [ ] Queue-based decoupling for async operations
- [ ] Dead letter queues for failed messages
- [ ] Idempotent operations for safe retries

### Change Management
- [ ] Canary deployments to limit blast radius
- [ ] Automated rollback on health check failure
- [ ] Infrastructure changes via IaC (not manual)
- [ ] Game days / chaos engineering practiced

## Common Anti-Patterns

| Anti-Pattern | Risk | Fix |
|---|---|---|
| Single-AZ deployment | One AZ outage = total downtime | Spread across 2-3 AZs |
| RDS Single Instance (no Multi-AZ) | DB failure = extended outage + data risk | Enable Multi-AZ |
| No health checks on ALB target group | Traffic sent to dead instances | Configure health check path + thresholds |
| Synchronous chain: A → B → C → D | One failure cascades to entire chain | Async with SQS, circuit breakers |
| No backups or untested backups | Data loss, extended recovery | Automated backups + quarterly restore tests |
| Tightly coupled microservices | Cannot deploy or fail independently | Event-driven with EventBridge/SQS |

## Key AWS Services

- **ELB**: Cross-AZ load balancing, health checks
- **Auto Scaling**: Compute elasticity, self-healing
- **RDS**: Multi-AZ, read replicas, automated backups
- **Route 53**: DNS failover, health checks
- **SQS / EventBridge**: Decoupling, async patterns
- **Backup**: Centralized backup management, cross-region
- **FIS (Fault Injection Simulator)**: Chaos engineering
- **DynamoDB**: Global tables for multi-region

## DR Strategy Tiers

| Strategy | RPO | RTO | Cost |
|----------|-----|-----|------|
| Backup & Restore | Hours | Hours | $ |
| Pilot Light | Minutes | 10-30 min | $$ |
| Warm Standby | Seconds-Minutes | Minutes | $$$ |
| Active-Active | Near-zero | Near-zero | $$$$ |

## Example Findings

**Good**: "ECS service runs across 3 AZs with ALB health checks, RDS Multi-AZ with automated backups (7-day retention), SQS between services with DLQ configured"

**Bad**: "Single EC2 instance in us-east-1a running the entire application, no auto-scaling group, EBS volume with no snapshots, database on the same instance"
