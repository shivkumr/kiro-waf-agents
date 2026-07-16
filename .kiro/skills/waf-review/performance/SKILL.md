---
name: waf-performance
description: AWS Well-Architected Performance Efficiency pillar. Use when reviewing, generating, or validating architecture for right-sizing, caching, database selection, async processing, and CDN.
---

# Performance Efficiency

## Checklist (for reviewers/validators)

### Compute Right-Sizing
- [ ] Instance types match workload profile (compute/memory/storage optimized)
- [ ] Graviton instances used where compatible
- [ ] Serverless considered for variable/bursty workloads
- [ ] Spot instances for fault-tolerant batch workloads
- [ ] Compute Optimizer recommendations reviewed

### Caching
- [ ] Application-level cache (ElastiCache Redis/Memcached)
- [ ] Database query cache or read replicas for read-heavy loads
- [ ] API response caching (API Gateway cache, CloudFront)
- [ ] Session store externalized to cache (not local disk)
- [ ] Cache invalidation strategy defined

### Database Selection
- [ ] Purpose-built database for workload pattern
- [ ] Read replicas for read-heavy workloads
- [ ] Connection pooling (RDS Proxy) for serverless/many connections
- [ ] Query optimization and indexing reviewed

### Async & Event-Driven
- [ ] Long-running tasks offloaded to queues
- [ ] Event-driven processing via EventBridge
- [ ] Webhooks/callbacks instead of polling
- [ ] Batch processing for non-time-sensitive operations

### Content Delivery & Edge
- [ ] CloudFront for static assets and API acceleration
- [ ] Lambda@Edge or CloudFront Functions for edge logic
- [ ] Regional edge caches for dynamic content

### Monitoring & Benchmarking
- [ ] Load testing performed (baseline established)
- [ ] P99 latency tracked (not just averages)
- [ ] Auto-scaling triggers based on actual metrics

## Implementation Guide (for writers/generators)

### Compute Selection

- Start with Graviton (arm64) for all new workloads — 20% better price-performance
- Use Fargate for containers unless sustained high-CPU (then EC2 with ASG)
- Lambda for event-driven, sub-15-min tasks with variable traffic
- Size based on actual metrics, not guesses — deploy small, then right-size with Compute Optimizer
- For batch/CI workloads: use Spot instances with graceful interruption handling
- Set Lambda memory based on profiling (memory also scales CPU proportionally)

### Caching Layers

- Add ElastiCache Redis between application and database for frequently-read data
- Cache TTL: short for volatile data (60s), longer for reference data (3600s)
- Use Redis cluster mode for horizontal scaling, single-node for simplicity
- Externalize session storage to Redis (never store sessions on local disk)
- For API responses: use CloudFront with cache-control headers (avoids hitting origin)
- DynamoDB + DAX for microsecond read latency on hot keys

### Database Selection

- Relational (Aurora/RDS): ACID transactions, complex joins, structured schema
- DynamoDB: key-value, single-digit ms latency, massive scale, flexible schema
- ElastiCache Redis: session store, leaderboards, real-time counters
- OpenSearch: full-text search, log analytics
- Timestream: IoT metrics, time-series data
- Neptune: graph relationships (social networks, fraud detection)
- Use RDS Proxy for Lambda→RDS connections (prevents connection exhaustion)
- Add read replicas when read:write ratio exceeds 5:1

### Async Processing

- Any operation >1 second that isn't user-blocking: put it on a queue
- Use SQS for point-to-point, EventBridge for fan-out/filtering
- Step Functions for multi-step workflows with error handling
- Set visibility timeout to 6x the expected processing time
- Use batch processing (SQS batch size 10) to improve throughput

### Content Delivery

- Put CloudFront in front of ALL static assets (S3 origins)
- For APIs: use CloudFront with caching on GET requests where possible
- Set appropriate Cache-Control headers at origin (don't rely on CDN defaults)
- Use Origin Shield to reduce origin load for popular content
- For global users: ensure CloudFront distribution covers needed edge locations

### Diagram Components

When generating architecture diagrams:
- Show CloudFront as the entry point for static content and optionally APIs
- Show ElastiCache between app tier and database tier
- Show read replicas as separate nodes from primary database
- Indicate async flows with SQS/EventBridge icons (not direct arrows)
- Label instance types or sizing tier on compute resources
- Show Graviton badge on arm64 resources

## Common Anti-Patterns

| Anti-Pattern | Risk | Fix |
|---|---|---|
| Over-provisioned EC2 (running at 5% CPU) | Wasted spend, false capacity confidence | Right-size with Compute Optimizer |
| No caching — every request hits DB | High latency under load, DB bottleneck | ElastiCache + read replicas |
| Synchronous calls to slow dependencies | Cascading latency | SQS + async processing |
| Serving static files from app server | Wasted compute, high latency | S3 + CloudFront |
| Single DB for everything | Contention, scaling ceiling | Purpose-built databases per workload |
| No CDN for global users | High latency for distant users | CloudFront with regional edge caches |

## Key AWS Services

- **CloudFront**: CDN, edge caching, TLS termination
- **ElastiCache**: Redis/Memcached in-memory caching
- **RDS Proxy**: Connection pooling for serverless
- **DynamoDB + DAX**: Microsecond read caching
- **Auto Scaling**: Dynamic compute adjustment
- **Compute Optimizer**: Right-sizing recommendations
- **Lambda**: Event-driven serverless compute
- **Step Functions**: Workflow orchestration
- **Global Accelerator**: Network-layer acceleration
