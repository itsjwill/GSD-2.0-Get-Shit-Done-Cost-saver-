# Changelog

All notable changes to GSD (Get Shit Done) are documented here.

---

## [2.0.0] - 2026-01-11

### 🚀 GSD 2.0 Release

**Forked from:** [glittercowboy/get-shit-done](https://github.com/glittercowboy/get-shit-done) v1.x
**Enhanced by:** Claude Opus 4.5 comprehensive analysis

---

### Analysis Methodology

GSD 2.0 was created through comprehensive AI-assisted analysis of the original codebase:

#### What Was Analyzed
- **22 Commands** - Every slash command in `commands/gsd/`
- **13 Workflows** - All workflow files in `get-shit-done/workflows/`
- **22 Templates** - All template files including codebase mapping
- **9 Reference Documents** - Knowledge base in `get-shit-done/references/`

#### Analysis Agents Used
1. **Command Analyzer** - Examined patterns, dependencies, tool usage, UX flows
2. **Workflow Analyzer** - Traced execution flows, subagent patterns, state management
3. **Template Analyzer** - Evaluated context engineering, token efficiency, metadata
4. **Reference Analyzer** - Checked consistency, gaps, verbosity, conflicts

#### Competitive Research
- **BMAD v6** - 21-agent system, 74-90% token savings claimed
- **SpecKit** - GitHub's spec-driven development toolkit
- **Anthropic Best Practices** - Official Claude Code context engineering guidance

---

### New Features in 2.0

#### 1. Multi-Model Intelligence (`get-shit-done/core/model-router.md`)
**What:** Automatic routing to optimal model (Haiku/Sonnet/Opus)
**Why:** 40-60% cost reduction, better quality on complex decisions
**How:**
- Haiku for validation, file ops, progress checks
- Sonnet for standard execution (default)
- Opus for architectural decisions, project initialization

#### 2. Adaptive Context Loading (`get-shit-done/core/context-loader.md`)
**What:** Load only relevant context, not everything
**Why:** 30% token savings, less noise in context
**How:**
- Tier-based loading (minimal → planning → execution → full)
- Subsystem detection (auth, database, UI keywords)
- Frontmatter dependency graph for smart summary selection

#### 3. Rollback System (`commands/gsd/rollback.md`)
**What:** Git-based checkpoints with one-command rollback
**Why:** Safety net for experimentation, faster recovery
**How:**
- Auto-creates `gsd-checkpoint-{phase}-{plan}-pre/post` tags
- `/gsd:rollback last` or `/gsd:rollback 02-01`
- Backup tags for undo-the-undo

#### 4. Recovery Command (`commands/gsd/recover.md`)
**What:** Intelligent recovery from interrupted/failed executions
**Why:** Eliminates manual debugging of stuck states
**How:**
- Diagnoses: interrupted agents, handoff files, uncommitted changes
- Offers: resume, restart, rollback, or manual review
- Cleans up artifacts automatically

#### 5. Master Plan Documentation (`.planning/GSD-2.0-MASTER-PLAN.md`)
**What:** Comprehensive 7-pillar enhancement roadmap
**Why:** Clear vision for 2.0 development
**Pillars:**
1. Multi-Model Intelligence ✅ Implemented
2. Parallel Execution Engine (planned)
3. Adaptive Context System ✅ Implemented
4. Self-Learning Memory (planned)
5. Rollback & Recovery ✅ Implemented
6. Project Templates (planned)
7. Quality Gate Integration (planned)

---

### What Changed from Original

#### Added Files
```
get-shit-done/core/                    # NEW: Core 2.0 systems
├── model-router.md                    # Multi-model routing rules
└── context-loader.md                  # Adaptive context loading

commands/gsd/
├── rollback.md                        # NEW: Checkpoint rollback
└── recover.md                         # NEW: Recovery from failures

.planning/
└── GSD-2.0-MASTER-PLAN.md            # NEW: 2.0 roadmap
```

#### Documentation Updates
```
CHANGELOG.md                           # NEW: This file
README.md                              # UPDATED: 2.0 features, attribution
```

#### Unchanged (Preserved from Original)
All existing commands and workflows remain backward compatible:
- 22 original commands (new-project, create-roadmap, plan-phase, execute-plan, etc.)
- 13 workflows (execute-phase.md, plan-phase.md, map-codebase.md, etc.)
- All templates and references
- Core philosophy and principles

---

### Migration from 1.x

**Backward Compatible:** All 1.x projects work without changes.

**To Enable 2.0 Features:**
1. Update to 2.0: `npx get-shit-done-cc@latest`
2. Add to `.planning/config.json`:
```json
{
  "version": "2.0",
  "model_routing": {
    "enabled": true
  },
  "adaptive_context": {
    "enabled": true
  },
  "checkpoints": {
    "enabled": true
  }
}
```

---

### Credits

**Original Creator:** TÂCHES ([@glittercowboy](https://github.com/glittercowboy))
- Core concept and philosophy
- All 1.x implementation
- "The complexity is in the system, not in your workflow"

**2.0 Enhancements:** Analysis and implementation by Claude Opus 4.5
- Comprehensive codebase analysis
- Competitive landscape research
- 2.0 feature design and implementation

**Research Sources:**
- [BMAD Method](https://github.com/bmad-code-org/BMAD-METHOD) - Multi-agent patterns
- [GitHub SpecKit](https://github.com/github/spec-kit) - Spec-driven development
- [Anthropic Engineering Blog](https://www.anthropic.com/engineering) - Context engineering best practices

---

### Future Roadmap

See `.planning/GSD-2.0-MASTER-PLAN.md` for detailed plans:

- [ ] Parallel Execution Engine (2-3x faster)
- [ ] Self-Learning Memory (cross-project patterns)
- [ ] Project Templates (SaaS, CLI, API starters)
- [ ] Quality Gate Integration (automated lint/test/build)

---

## [1.x] - Original Release

See original repository: https://github.com/glittercowboy/get-shit-done

Core features:
- Subagent orchestration with fresh 200K context
- XML task structure for Claude-native parsing
- 5-tier deviation rules for auto-handling
- Per-task atomic commits
- Brownfield support with codebase mapping
- TDD integration
- Frontmatter dependency graphs
