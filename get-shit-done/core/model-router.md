# Model Router

<purpose>
Intelligent model selection for optimal cost, speed, and quality balance.
Route operations to the right model: Haiku for speed, Sonnet for balance, Opus for complexity.
</purpose>

<routing_rules>

## Haiku (Fast, Cost-Effective)
**Use for:** Operations requiring speed over reasoning depth

### Automatic Haiku Routing
- File existence checks (`[ -f file ]`)
- Simple grep/glob operations
- YAML/JSON/Markdown parsing
- Template instantiation
- Progress bar calculations
- Syntax validation (lint-like checks)
- STATE.md position extraction
- Config.json reading

### Characteristics
- **Speed:** ~2x faster than Sonnet
- **Cost:** ~$0.25/1M tokens (12x cheaper than Sonnet)
- **Quality:** Sufficient for structured, deterministic tasks

---

## Sonnet (Balanced) - DEFAULT
**Use for:** Standard development operations

### Automatic Sonnet Routing
- Task execution (code generation)
- Test writing
- Standard planning (2-3 task plans)
- Documentation generation
- Bug fixes (Rule 1-3 deviations)
- Verification checks
- SUMMARY.md creation

### Characteristics
- **Speed:** Baseline
- **Cost:** ~$3/1M tokens
- **Quality:** Excellent for most coding tasks

---

## Opus (Maximum Reasoning)
**Use for:** Complex decisions requiring deep analysis

### Automatic Opus Routing
- Architectural decisions (Rule 4 deviations)
- Initial project questioning (`/gsd:new-project`)
- Milestone planning (`/gsd:discuss-milestone`)
- Complex debugging (multi-file, multi-system)
- Research synthesis (`/gsd:research-phase`)
- checkpoint:decision analysis
- Code review for security/architecture

### Characteristics
- **Speed:** ~0.5x Sonnet (more thorough)
- **Cost:** ~$15/1M tokens
- **Quality:** Best for nuanced reasoning

</routing_rules>

<escalation_protocol>

## Automatic Escalation

### Haiku → Sonnet
**Trigger:** Task fails twice with Haiku
**Action:** Retry with Sonnet, log escalation

```
[Model Router] Haiku failed on {task}, escalating to Sonnet
```

### Sonnet → Opus
**Trigger:**
- Task involves >5 files
- User explicitly requests deep analysis
- Deviation detected that could be Rule 4
- checkpoint:decision with >3 options

**Action:** Switch to Opus for this operation

```
[Model Router] Complex task detected, using Opus for {operation}
```

### De-escalation
**Trigger:** After Opus decision, routine tasks resume
**Action:** Return to Sonnet/Haiku for subsequent work

</escalation_protocol>

<configuration>

## Config.json Schema

```json
{
  "model_routing": {
    "enabled": true,
    "default_model": "sonnet",
    "routing": {
      "validation": "haiku",
      "file_ops": "haiku",
      "execution": "sonnet",
      "planning": "sonnet",
      "complex_planning": "opus",
      "decisions": "opus",
      "research": "opus"
    },
    "escalation": {
      "haiku_retry_limit": 2,
      "auto_escalate": true
    },
    "overrides": {
      "always_opus": ["new-project", "discuss-milestone"],
      "always_sonnet": ["execute-plan"],
      "prefer_haiku": ["progress", "help"]
    }
  }
}
```

## User Override

Users can force a specific model:
```
/gsd:plan-phase 3 --model=opus
```

Or disable routing entirely:
```json
{
  "model_routing": {
    "enabled": false
  }
}
```

</configuration>

<integration>

## How Workflows Use This

### In Commands
```markdown
<model_preference>opus</model_preference>
```

### In Workflows
```markdown
<step name="analyze" model="opus">
Complex analysis requiring deep reasoning
</step>

<step name="write_files" model="haiku">
Simple file operations
</step>
```

### In Task Tool Calls
```
When spawning subagent:
- Check task complexity
- Route to appropriate model
- Log routing decision
```

</integration>

<cost_tracking>

## Token Accounting

After each operation, log:
```json
{
  "operation": "plan-phase",
  "model_used": "sonnet",
  "tokens": {
    "input": 15000,
    "output": 3000
  },
  "cost_estimate": "$0.054",
  "routing_reason": "standard_planning"
}
```

Aggregate in STATE.md:
```markdown
## Cost Metrics (2.0)
- Total tokens: 150,000
- Estimated cost: $2.50
- Savings vs single-model: ~40%
```

</cost_tracking>

<expected_impact>

## Typical Project Savings

| Operation | Count | Old Cost | New Cost | Savings |
|-----------|-------|----------|----------|---------|
| Progress checks | 20 | $0.60 | $0.05 | 92% |
| Validation | 50 | $1.50 | $0.12 | 92% |
| Planning | 10 | $3.00 | $3.00 | 0% |
| Execution | 15 | $7.50 | $7.50 | 0% |
| Decisions | 5 | $1.50 | $2.50 | -67% |
| **Total** | 100 | **$14.10** | **$13.17** | **7%** |

*Note: Opus decisions cost more but produce better outcomes. Net savings come from Haiku on routine ops.*

**With aggressive Haiku routing:**
- 40-60% reduction achievable on validation-heavy workflows

</expected_impact>
