# Adaptive Context Loader

<purpose>
Load only what's needed, when it's needed.
Reduce token waste by 30% through intelligent context selection.
</purpose>

<loading_tiers>

## Tier 0: Always Load (~200 tokens)
Minimal context for any GSD operation.

```
.planning/STATE.md (Current Position section only)
.planning/config.json
```

**When:** Every GSD command invocation
**Why:** Position awareness and config needed universally

---

## Tier 1: Planning Context (~2,000 tokens)
Context needed for planning operations.

```
.planning/STATE.md (full)
.planning/ROADMAP.md
.planning/PROJECT.md
```

**When:** `/gsd:plan-phase`, `/gsd:create-roadmap`, `/gsd:discuss-phase`
**Why:** Planning needs vision + structure

---

## Tier 2: Execution Context (~3,000 tokens)
Context needed for task execution.

```
Tier 1 context +
.planning/phases/{phase}/{plan}-PLAN.md
Relevant prior SUMMARY.md (via frontmatter graph)
```

**When:** `/gsd:execute-plan`
**Why:** Execution needs task details + relevant history

---

## Tier 3: Brownfield Context (~5,000 tokens)
Additional context for existing codebases.

```
Tier 1 or 2 +
.planning/codebase/{relevant}.md (subsystem-specific)
```

**When:** Planning/execution for brownfield projects
**Why:** Need to understand existing patterns

---

## Tier 4: Full Context (~10,000+ tokens)
Complete context for complex operations.

```
All tiers +
All SUMMARY.md files
ISSUES.md
RESEARCH.md (if exists)
```

**When:** `/gsd:complete-milestone`, complex debugging
**Why:** Need complete project picture

</loading_tiers>

<subsystem_detection>

## Automatic Subsystem Detection

Parse current task/phase description for keywords:

| Keywords | Subsystem | Load |
|----------|-----------|------|
| auth, login, JWT, session, password | auth | ARCHITECTURE.md, auth summaries |
| database, schema, migration, model | database | STACK.md, ARCHITECTURE.md |
| API, endpoint, REST, GraphQL | api | ARCHITECTURE.md, CONVENTIONS.md |
| UI, component, style, layout | ui | CONVENTIONS.md, STRUCTURE.md |
| test, spec, coverage | testing | TESTING.md, CONVENTIONS.md |
| deploy, CI, pipeline | infra | STACK.md, INTEGRATIONS.md |
| payment, stripe, billing | payments | INTEGRATIONS.md, relevant summaries |

### Detection Algorithm

```python
def detect_subsystems(phase_description: str, task_names: list) -> list:
    keywords = extract_keywords(phase_description + ' '.join(task_names))

    subsystems = []
    for keyword in keywords:
        if keyword in AUTH_KEYWORDS:
            subsystems.append('auth')
        elif keyword in DB_KEYWORDS:
            subsystems.append('database')
        # ... etc

    return list(set(subsystems))
```

</subsystem_detection>

<dependency_graph>

## Frontmatter-Based Context Selection

### How It Works

1. **Scan Phase:** Read first 30 lines of all SUMMARY.md files (frontmatter only)
2. **Build Graph:** Extract `requires`, `provides`, `affects` fields
3. **Transitive Closure:** Find all phases that feed into current phase
4. **Selective Load:** Read full content only for relevant phases

### Example

Current phase: 05-api-endpoints

```yaml
# 05-01-SUMMARY.md frontmatter
requires:
  - phase: 02-auth
    provides: [JWT middleware, User model]
  - phase: 03-database
    provides: [Prisma client, Schema]
```

**Load:**
- 05-01-PLAN.md (current)
- 02-*-SUMMARY.md (provides auth)
- 03-*-SUMMARY.md (provides database)
- Skip: 01-*, 04-* (not required)

### Graph Building

```markdown
<step name="build_dependency_graph">
1. List all SUMMARY.md files:
   ```bash
   find .planning/phases -name "*-SUMMARY.md"
   ```

2. For each, extract frontmatter:
   ```bash
   head -30 {file} | grep -A 10 "requires:"
   ```

3. Build adjacency list:
   ```
   phase_02 → [phase_05]  # 02 feeds 05
   phase_03 → [phase_05]  # 03 feeds 05
   ```

4. For current phase, find all ancestors:
   ```
   ancestors(05) = {02, 03}
   ```

5. Load only ancestor summaries
</step>
```

</dependency_graph>

<context_budget>

## Budget Enforcement

### Limits
- **Hard limit:** 40% of context window for loaded files
- **Soft limit:** 30% (trigger warning if exceeded)
- **Target:** 20-25% for optimal quality

### When Over Budget

1. **Summarize older content:**
   ```
   Phases 1-3: Summarize to 500 tokens each
   Phase 4+: Keep full content
   ```

2. **Drop low-relevance context:**
   - If subsystem not detected, skip that codebase doc
   - If phase >5 back, use summary only

3. **Alert user:**
   ```
   ⚠️ Context usage: 45%
   Summarized phases 1-2 to maintain quality.
   Consider splitting plan into smaller scope.
   ```

### Budget Calculation

```
Total budget = 200,000 tokens (Claude context)
Reserved for output = 50,000 tokens
Available for context = 150,000 tokens
Target usage = 40% = 60,000 tokens

If loaded_tokens > 60,000:
  - Trigger compression
  - Alert user
```

</context_budget>

<context_manifest>

## Context Manifest File

Optional file to customize loading:

```json
// .planning/context-manifest.json
{
  "always_load": [
    "STATE.md"
  ],
  "planning_context": [
    "ROADMAP.md",
    "PROJECT.md"
  ],
  "subsystem_mapping": {
    "auth": [
      "codebase/ARCHITECTURE.md",
      "phases/02-auth/*-SUMMARY.md"
    ],
    "database": [
      "codebase/STACK.md",
      "codebase/ARCHITECTURE.md"
    ]
  },
  "exclude": [
    "phases/01-foundation/*"  // Skip if foundational work is stable
  ],
  "custom_rules": [
    {
      "if_phase_contains": "payment",
      "load": ["codebase/INTEGRATIONS.md", "notes/stripe-setup.md"]
    }
  ]
}
```

</context_manifest>

<integration>

## How Workflows Use This

### In plan-phase.md

```markdown
<step name="load_context" loader="adaptive">
## Load Planning Context

1. Load Tier 1 (always)
2. Detect subsystems from phase description
3. Load relevant codebase docs (Tier 3 if brownfield)
4. Build dependency graph from summaries
5. Load relevant prior summaries
6. Check budget, compress if needed
</step>
```

### In execute-phase.md

```markdown
<step name="load_context" loader="adaptive">
## Load Execution Context

1. Load Tier 0 (minimal)
2. Load PLAN.md
3. Parse task files[] to detect subsystems
4. Load only codebase docs for detected subsystems
5. Load summaries from frontmatter requires[]
</step>
```

</integration>

<expected_impact>

## Token Savings Analysis

### Before (Eager Loading)
| Context Type | Tokens |
|-------------|--------|
| STATE.md | 1,000 |
| ROADMAP.md | 2,000 |
| PROJECT.md | 1,500 |
| All codebase/*.md | 8,000 |
| All prior summaries | 12,000 |
| **Total** | **24,500** |

### After (Adaptive Loading)
| Context Type | Tokens |
|-------------|--------|
| STATE.md (position) | 200 |
| ROADMAP.md | 2,000 |
| PROJECT.md | 1,500 |
| Relevant codebase (2 files) | 2,000 |
| Relevant summaries (3 files) | 4,500 |
| **Total** | **10,200** |

**Savings: 58%**

### Per-Project Impact
- 10 plans × 14,000 tokens saved = 140,000 tokens
- At $3/1M tokens = **$0.42 saved per project**
- Plus: Better quality from less noise

</expected_impact>
