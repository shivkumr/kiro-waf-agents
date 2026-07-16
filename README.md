# WAF Architecture Agent Suite for Kiro CLI

A multi-agent system built on [Kiro CLI](https://kiro.dev) that helps teams design, build, validate, and audit AWS infrastructure following the **AWS Well-Architected Framework (WAF)** — all from natural language.

## What It Does

```
Natural Language → Architecture Diagram → IaC Code → Code Validation → Live Resource Audit
```

One orchestrator agent (`waf-ops`) routes your requests to 5 specialist agents, each focused on a specific phase of the infrastructure lifecycle:

| Phase | Agent | What It Does |
|-------|-------|--------------|
| **Design** | `waf-diagram-generator` | Creates draw.io architecture diagrams from text descriptions |
| **Review** | `waf-reviewer` | Reviews diagrams/architectures against all 6 WAF pillars |
| **Build** | `waf-iac-writer` | Generates WAF-compliant Terraform/CDK/CloudFormation |
| **Validate** | `waf-iac-validator` | Static analysis of IaC code against WAF best practices |
| **Audit** | `waf-resource-validator` | Checks live AWS resources against WAF best practices |

All agents share 6 WAF pillar skill files as a single source of truth — update a checklist once, it applies everywhere.

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│  waf-ops (Ctrl+Shift+W) — orchestrator                           │
│  MCP: AWS Docs | AWS API (read-only)                             │
│                                                                   │
│  ┌──────────────┬────────────┬────────────┬──────────┬─────────┐ │
│  │ waf-diagram  │ waf-       │ waf-iac-   │ waf-iac- │ waf-    │ │
│  │ -generator   │ reviewer   │ writer     │ validator│ resource│ │
│  │ Draw.io MCP  │ (no MCP)   │ Terraform  │Terraform │ AWS API │ │
│  │ + Docs MCP   │            │ + Docs MCP │+ Docs MCP│+ Docs   │ │
│  └──────────────┴────────────┴────────────┴──────────┴─────────┘ │
│                                                                   │
│  Shared: 6 WAF Skill Files (single source of truth)               │
└──────────────────────────────────────────────────────────────────┘
```

## Prerequisites

### Required

1. **Kiro CLI** installed and authenticated
   ```bash
   kiro-cli --version
   ```

2. **Node.js v18+** (for Draw.io MCP server)
   ```bash
   node --version
   ```

3. **Python 3.10+** with **uv** (for AWS Documentation and API MCP servers)
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   uv --version
   ```

4. **Docker** (for Terraform MCP server)
   ```bash
   docker --version
   docker pull hashicorp/terraform-mcp-server:latest
   ```

5. **AWS credentials** configured (for resource validator)
   ```bash
   aws sts get-caller-identity
   ```

### Enable Required Kiro CLI Features

```bash
kiro-cli settings chat.enableSubagent true
kiro-cli settings chat.enableTodoList true
```

## Installation

1. **Clone or copy** the `kiro-waf-agent/` folder into your project:
   ```bash
   cp -r kiro-waf-agent/.kiro /path/to/your/project/.kiro
   ```

   Or use it standalone:
   ```bash
   cd kiro-waf-agent
   kiro-cli chat
   ```

2. **Verify agents load:**
   ```
   /agent list
   ```
   You should see: `waf-ops`, `waf-diagram-generator`, `waf-reviewer`, `waf-iac-writer`, `waf-iac-validator`, `waf-resource-validator`

## Usage

### Quick Start

```bash
cd kiro-waf-agent
kiro-cli chat
```

Then:
```
/agent swap waf-ops
```

That's it. Talk naturally — the orchestrator routes to the right specialist.

### End-to-End Workflow Example

#### Step 1: Generate an Architecture Diagram

```
> Draw a serverless event-driven pipeline:
  - S3 bucket receives files
  - Lambda processes them
  - Results go to DynamoDB
  - Failures go to SQS dead letter queue
  - CloudWatch alarms on errors
```

→ `waf-diagram-generator` creates a `.drawio` file with official AWS icons, proper VPC boundaries, and WAF best practices baked in.

#### Step 2: Review the Architecture

```
> Review the diagram I just generated for WAF compliance
```

→ `waf-reviewer` analyzes against all 6 pillars and reports: ✅ Good / ⚠️ Needs Improvement / ❌ Critical Gap for each.

#### Step 3: Generate the Terraform

```
> Now generate the Terraform for this architecture
```

→ `waf-iac-writer` produces modular `.tf` files with:
- Encryption enabled everywhere
- Proper IAM roles (least privilege)
- Tags on all resources
- CloudWatch alarms configured
- Comments explaining each WAF decision

#### Step 4: Validate the Code

```
> Validate the Terraform in ./terraform/
```

→ `waf-iac-validator` reads the files and reports per-pillar findings:
```
❌ Security | main.tf:42 | S3 bucket missing Block Public Access
   Fix: Add `aws_s3_bucket_public_access_block` resource

⚠️ Reliability | rds.tf:18 | RDS instance not Multi-AZ
   Fix: Set `multi_az = true`

💡 Cost | lambda.tf:9 | Lambda memory at 1024MB but CPU-bound workload
   Fix: Consider 512MB with ARM architecture (Graviton)
```

#### Step 5: Audit Live Resources

```
> Check if the deployed Lambda and DynamoDB in us-east-1 match best practices
```

→ `waf-resource-validator` queries live AWS via API and reports:
```
❌ Security | arn:aws:lambda:us-east-1:123:function:processor
   Current: Runtime python3.9 (EOL)
   Expected: python3.12+
   Fix: aws lambda update-function-configuration --runtime python3.12

✅ Reliability | arn:aws:dynamodb:us-east-1:123:table/results
   Point-in-time recovery enabled, on-demand capacity
```

### Parallel Workflows

Run multiple specialists simultaneously:

```
> Draw the diagram AND generate the Terraform for a 3-tier web app
```
→ `waf-diagram-generator` + `waf-iac-writer` run in parallel

```
> Validate my Terraform AND check the live resources
```
→ `waf-iac-validator` + `waf-resource-validator` run in parallel

### Direct Access (Skip Orchestrator)

Use keyboard shortcuts to go directly to a specialist:

| Shortcut | Agent |
|----------|-------|
| `Ctrl+Shift+W` | waf-ops (orchestrator) |
| `Ctrl+Shift+D` | waf-diagram-generator |
| `Ctrl+Shift+A` | waf-reviewer |
| `Ctrl+Shift+I` | waf-iac-writer |
| `Ctrl+Shift+V` | waf-iac-validator |
| `Ctrl+Shift+R` | waf-resource-validator |

### AWS Documentation Lookups

The orchestrator handles documentation questions directly:

```
> What are the best practices for RDS encryption at rest?
> What S3 lifecycle policy options are available?
> How do I configure CloudFront with WAF?
```

## Project Structure

```
kiro-waf-agent/
└── .kiro/
    ├── agents/
    │   ├── waf-ops.json                    # Orchestrator (5 subagents)
    │   ├── waf-diagram-generator.json      # Draw.io diagram creation
    │   ├── waf-reviewer.json               # Architecture review
    │   ├── waf-iac-writer.json             # IaC code generation
    │   ├── waf-iac-validator.json          # IaC static analysis
    │   └── waf-resource-validator.json     # Live resource audit
    ├── steering/
    │   └── waf-review-format.md            # Output format template (always loaded)
    └── skills/
        └── waf-review/
            ├── operational-excellence/SKILL.md  # Pillar 1
            ├── security/SKILL.md               # Pillar 2
            ├── reliability/SKILL.md            # Pillar 3
            ├── performance/SKILL.md            # Pillar 4
            ├── cost-optimization/SKILL.md      # Pillar 5
            └── sustainability/SKILL.md         # Pillar 6
```

## MCP Servers Used

| MCP Server | Purpose | Required By |
|------------|---------|-------------|
| [AWS Documentation](https://awslabs.github.io/mcp/servers/aws-documentation-mcp-server) | Search/read AWS docs | All except reviewer |
| [AWS API](https://awslabs.github.io/mcp/servers/aws-api-mcp-server) | Query live AWS resources (read-only) | waf-ops, waf-resource-validator |
| [Draw.io](https://aws-samples.github.io/sample-drawio-mcp/) | Generate architecture diagrams | waf-diagram-generator |
| [Terraform](https://github.com/hashicorp/terraform-mcp-server) | Provider docs, resource schemas | waf-iac-writer, waf-iac-validator |

## Customization

### Update WAF Checklists

Edit any skill file in `.kiro/skills/waf-review/*/SKILL.md`. Changes apply to all agents automatically.

### Add a New Pillar or Custom Checklist

1. Create `.kiro/skills/waf-review/my-custom-pillar/SKILL.md`
2. Add YAML frontmatter:
   ```yaml
   ---
   name: my-custom-pillar
   description: When to load this skill
   ---
   ```
3. Add the skill URI to each agent's `resources` array:
   ```json
   "skill://.kiro/skills/waf-review/my-custom-pillar/SKILL.md"
   ```

### Change Default Region

Edit the `AWS_REGION` environment variable in `waf-ops.json` and `waf-resource-validator.json`:
```json
"env": {
  "AWS_REGION": "us-west-2"
}
```

### Restrict AWS API Access

The AWS API MCP server is already set to read-only:
```json
"READ_OPERATIONS_ONLY": "true"
```

To further restrict, configure AWS IAM credentials with a scoped-down policy.

## Kiro CLI Features Demonstrated

This project showcases these advanced Kiro CLI capabilities:

1. **Custom Agents** — Specialized AI personas with distinct tools and prompts
2. **Steering Files** — Always-loaded context for consistent output formatting
3. **Skills** — On-demand knowledge loaded only when relevant
4. **MCP Servers** — External tool integrations (AWS, Terraform, Draw.io)
5. **Subagents** — Parallel execution of multiple specialists
6. **Keyboard Shortcuts** — Quick agent switching

## Troubleshooting

### MCP server won't start

```bash
# AWS Documentation/API
uvx awslabs.aws-documentation-mcp-server@latest --help

# Terraform
docker run --rm hashicorp/terraform-mcp-server:latest --help

# Draw.io
npx -y https://github.com/aws-samples/sample-drawio-mcp/releases/latest/download/drawio-mcp-server-latest.tgz --help
```

### Agent not found

Ensure you're running `kiro-cli chat` from within the `kiro-waf-agent/` directory (or a directory containing the `.kiro/` folder).

### Subagent approval prompts

`waf-iac-writer` requires approval because it writes files. The other 4 specialists are auto-trusted (read-only). Press `y` when prompted.

### AWS API access denied

The resource validator needs valid AWS credentials. Run:
```bash
aws sts get-caller-identity
```

If expired, re-authenticate via SSO or refresh your credentials.

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.

## Credits

Built with [Kiro CLI](https://kiro.dev) using:
- [AWS MCP Servers](https://awslabs.github.io/mcp/) by AWS Labs
- [Draw.io MCP Server](https://aws-samples.github.io/sample-drawio-mcp/) by AWS Samples
- [Terraform MCP Server](https://github.com/hashicorp/terraform-mcp-server) by HashiCorp
