---
name: waf-reliability
description: AWS Well-Architected Reliability pillar. Use when reviewing, generating, or validating architecture for Multi-AZ/Region, auto-scaling, DR, fault isolation, and self-healing.
---

# Reliability

## Checklist (for reviewers/validators)

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
- [ ] Services loosely coupled via queues/events
- [ ] Bulkhead pattern: failure in one component doesn't cascade
- [ ] Circuit breakers for downstream dependencies
- [ ] Timeout and retry with exponential backoff
- [ ] Graceful degradation (serve stale cache, feature flags)

### Dependency Management
- [ ] External dependency failures handled
- [ ] Queue-based decoupling for async operations
- [ ] Dead letter queues for failed messages
- [ ] Idempotent operations for safe retries

## Implementation Guide (for writers/generators)

### Multi-AZ Design

- Always deploy to a minimum of 2 AZs (3 preferred for production)
- Place subnets in separate AZs and pass all AZ subnets to compute resources
- RDS: always enable Multi-AZ (`multi_az = true`) for production
- ElastiCache: use replication group with `automatic_failover_enabled` and nodes in separate AZs
- ECS/EKS: spread tasks/pods across AZ subnets via service configuration
- ALB: register targets in all AZs, enable cross-zone load balancing
- NAT Gateway: deploy one per AZ for AZ-independent resilience (cost trade-off: document this)

### Auto-Scaling

- ECS services: use target tracking on CPU (70%) and memory (80%) as starting point
- EC2 ASG: set min = baseline steady-state, max = 2-3x peak, desired = current need
- Use target tracking over step scaling (simpler, self-adjusting)
- Add scaling cooldown to prevent flapping (300s default)
- For Lambda: configure reserved concurrency for critical functions, provisioned for latency-sensitive
- Always set ECS deployment circuit breaker with rollback enabled

### Health Checks

- ALB health check: use a dedicated `/health` endpoint that verifies downstream dependencies
- Set health check interval to 10s, threshold to 3 unhealthy counts
- ECS: configure both ALB health check AND container health check (belt and suspenders)
- For non-HTTP services: use TCP health checks or custom CloudWatch-based checks
- Route 53: add health checks for DNS failover to DR region

### Backup & DR

- Enable automated backups on all databases (minimum 7-day retention for dev, 35 for prod)
- Enable point-in-time recovery on DynamoDB tables
- Use AWS Backup for centralized, cross-service backup management
- For critical data: replicate backups to a second region
- Document RPO/RTO per service and validate with quarterly restore tests
- Tag backups with retention policy and owning service

### Fault Isolation & Decoupling

- Between microservices: use SQS queues for async operations (not direct HTTP chains)
- Configure dead letter queues (DLQ) with `maxReceiveCount` of 3-5 before DLQ redirect
- Set reasonable timeouts on all HTTP calls (5s default, adjust per dependency)
- For critical paths: implement circuit breaker pattern (use Lambda destinations or Step Functions catch)
- Use EventBridge for event-driven decoupling (fan-out without direct coupling)
- Design all operations to be idempotent (use idempotency keys for writes)

### Diagram Components

When generating architecture diagrams:
- Show resources spanning 2+ AZ boxes
- Draw AZ boundaries clearly labeled (us-east-1a, us-east-1b)
- Show Auto Scaling indicators on compute resources
- Include SQS/EventBridge between services (not direct arrows for async flows)
- Show DLQ connected to each queue
- If DR: show second region with Route 53 failover connecting them
- Include backup arrows to S3/Backup vault

## Common Anti-Patterns

| Anti-Pattern | Risk | Fix |
|---|---|---|
| Single-AZ deployment | One AZ outage = total downtime | Spread across 2-3 AZs |
| RDS Single Instance (no Multi-AZ) | DB failure = extended outage | Enable Multi-AZ |
| No health checks on ALB targets | Traffic sent to dead instances | Configure health check path |
| Synchronous chain A → B → C → D | One failure cascades entire chain | Async with SQS, circuit breakers |
| No backups or untested backups | Data loss, extended recovery | Automated backups + quarterly tests |
| Tightly coupled microservices | Cannot deploy or fail independently | Event-driven with EventBridge/SQS |

## DR Strategy Tiers

| Strategy | RPO | RTO | Cost | When to Use |
|----------|-----|-----|------|-------------|
| Backup & Restore | Hours | Hours | $ | Non-critical, cost-sensitive |
| Pilot Light | Minutes | 10-30 min | $$ | Important but not real-time |
| Warm Standby | Seconds-Minutes | Minutes | $$$ | Business-critical apps |
| Active-Active | Near-zero | Near-zero | $$$$ | Zero-tolerance for downtime |

## Key AWS Services

- **ELB**: Cross-AZ load balancing, health checks
- **Auto Scaling**: Compute elasticity, self-healing
- **RDS**: Multi-AZ, read replicas, automated backups
- **Route 53**: DNS failover, health checks
- **SQS / EventBridge**: Decoupling, async patterns
- **Backup**: Centralized backup management, cross-region
- **DynamoDB**: Global tables for multi-region
- **FIS**: Fault injection for chaos engineering
