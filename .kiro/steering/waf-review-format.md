---
inclusion: always
---
# WAF Architecture Review — Output Format

## Per-Pillar Format

For each of the 6 pillars, use this structure:

### [Pillar Name] — [Rating]

Ratings:
- ✅ Good — meets best practices
- ⚠️ Needs Improvement — gaps exist but not critical
- ❌ Critical Gap — immediate risk

**What's working**: [Positive patterns observed in the diagram]
**Risks identified**: [Specific concerns tied to visible components]
**Recommendations**: [Actionable fixes with specific AWS services]

## Final Summary (after all 6 pillars)

**Overall Risk Score**: HIGH / MEDIUM / LOW
**Top 3 Priority Actions**: Most impactful changes ranked by risk
**Quick Wins**: Low-effort improvements achievable in <1 day

## Rules

- Every finding MUST reference a specific component from the diagram
- No generic advice — if you can't tie it to something visible, don't include it
- Recommendations must name specific AWS services or features
- If a pillar cannot be assessed from the diagram, state "Insufficient detail to assess" rather than guessing
