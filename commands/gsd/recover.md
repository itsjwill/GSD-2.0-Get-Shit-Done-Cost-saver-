---
description: Recover from interrupted or failed execution
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
  - Task
---

<objective>
Intelligent recovery from interrupted, failed, or stuck GSD executions.

Detects the problem state and offers appropriate recovery options:
- Resume interrupted subagents
- Restart failed plans
- Clean up partial state
- Rollback to last good checkpoint
</objective>

<process>

<step name="diagnose">
## Diagnose Current State

Run comprehensive state check:

```bash
# Check for incomplete plans (PLAN without SUMMARY)
find .planning/phases -name "*-PLAN.md" | while read plan; do
  summary="${plan/-PLAN.md/-SUMMARY.md}"
  [ ! -f "$summary" ] && echo "INCOMPLETE: $plan"
done
```

```bash
# Check for interrupted agents
cat .planning/current-agent-id.txt 2>/dev/null
cat .planning/agent-history.json 2>/dev/null | grep '"status": "spawned"'
```

```bash
# Check for handoff files
find .planning/phases -name ".continue-here.md"
```

```bash
# Check git state
git status --porcelain
git stash list
```

**Categorize the state:**

| Finding | State | Recovery Action |
|---------|-------|-----------------|
| PLAN exists, no SUMMARY, no agent | Plan not started | Offer to execute |
| PLAN exists, agent spawned | Agent interrupted | Offer to resume agent |
| .continue-here.md exists | Mid-task pause | Offer to resume from handoff |
| Uncommitted changes + incomplete | Partial execution | Offer to commit/stash/discard |
| SUMMARY exists but incomplete | Failed summary | Regenerate summary |
| Everything looks complete | No problem detected | Show /gsd:progress |
</step>

<step name="present_diagnosis">
## Present Diagnosis

```
🔍 Recovery Diagnosis

State: [Interrupted Agent / Incomplete Plan / Mid-Task Pause / etc.]

Details:
  • Phase: 03-authentication
  • Plan: 03-02-PLAN.md
  • Last known progress: Task 2 of 4 complete
  • Interrupted agent: agent_01HXYZ123
  • Handoff file: .planning/phases/03-auth/.continue-here.md

Recovery options available:
```
</step>

<step name="offer_recovery">
## Offer Recovery Options

Based on diagnosis, use AskUserQuestion:

**For Interrupted Agent:**
- header: "Recovery"
- question: "An agent was interrupted mid-execution. How would you like to proceed?"
- options:
  - "Resume agent" — Try to resume the interrupted agent (Recommended)
  - "Restart plan" — Start the plan fresh from task 1
  - "Rollback" — Undo all changes and go back to pre-plan state
  - "Manual review" — Show me what was done so far

**For Mid-Task Pause (.continue-here.md):**
- header: "Recovery"
- question: "Found a pause point from a previous session. How to proceed?"
- options:
  - "Resume from handoff" — Continue from where you left off (Recommended)
  - "Restart plan" — Start the plan fresh
  - "Discard handoff" — Delete .continue-here.md and start fresh

**For Uncommitted Changes:**
- header: "Recovery"
- question: "Found uncommitted changes. How to handle?"
- options:
  - "Commit changes" — Stage and commit what's there
  - "Stash changes" — Save changes for later
  - "Discard changes" — Throw away uncommitted work
  - "Show changes" — Let me review first

**For Incomplete Plan (no agent, no handoff):**
- header: "Recovery"
- question: "Found an incomplete plan with no execution state. How to proceed?"
- options:
  - "Execute plan" — Start executing the plan now (Recommended)
  - "Regenerate plan" — Delete and recreate the plan
  - "Skip plan" — Mark as skipped and move to next
</step>

<step name="execute_recovery">
## Execute Chosen Recovery

**Resume Agent:**
```markdown
1. Read agent ID from current-agent-id.txt or agent-history.json
2. Use Task tool with resume parameter:
   - resume: {agent_id}
   - prompt: "Continue execution from where you left off"
3. Monitor completion
4. Update agent-history.json
5. Continue normal flow
```

**Resume from Handoff:**
```markdown
1. Read .continue-here.md for context:
   - Current task number
   - Completed work
   - Remaining work
   - Mental context
2. Present summary to user
3. Continue execution from that task
4. Delete .continue-here.md after resumption
```

**Restart Plan:**
```markdown
1. Delete any partial artifacts:
   - SUMMARY.md (if exists and incomplete)
   - .continue-here.md
   - Clear current-agent-id.txt
2. Reset git if needed:
   - If partial commits: offer rollback to pre-plan
   - If no commits: just restart
3. Execute /gsd:execute-plan fresh
```

**Rollback:**
```markdown
1. Find pre-execution checkpoint:
   - git tag | grep "gsd-checkpoint-{phase}-{plan}-pre"
2. Execute /gsd:rollback {phase}-{plan}
3. Return to clean state
```

**Commit Partial Work:**
```markdown
1. Stage all changes: git add -A
2. Create WIP commit:
   ```bash
   git commit -m "wip({phase}-{plan}): partial execution - interrupted"
   ```
3. Update STATE.md with partial progress note
4. Offer to continue or pause
```
</step>

<step name="verify_recovery">
## Verify Recovery Success

After recovery action:

```bash
# Verify clean state
git status  # Should be clean or have only expected changes
```

```bash
# Verify no lingering artifacts
ls .planning/current-agent-id.txt 2>/dev/null  # Should not exist
find .planning/phases -name ".continue-here.md"  # Should be empty
```

```bash
# Verify STATE.md is accurate
cat .planning/STATE.md | grep "Current Position"
```

Report to user:
```
✓ Recovery complete

Current state:
  Phase: 03-authentication
  Plan: Ready to execute 03-02
  Git: Clean

Recovered from: [Interrupted agent / Mid-task pause / etc.]
Action taken: [Resume / Restart / Rollback / etc.]

Next: /gsd:execute-plan or /gsd:progress
```
</step>

</process>

<edge_cases>

## Special Scenarios

**Multiple Problems Detected:**
Address in priority order:
1. Uncommitted changes (git state)
2. Interrupted agents (might still be running)
3. Handoff files (user-initiated pause)
4. Incomplete plans (system state)

**Agent Can't Be Resumed:**
- If Task tool resume fails
- Offer: Restart plan or Rollback
- Log failure in STATE.md

**Corrupted State Files:**
- If STATE.md or ROADMAP.md is malformed
- Offer to reconstruct from git history
- Or restore from last known good state

**No Recovery Needed:**
```
✓ No issues detected

Your project is in a clean state.
Current position: Phase 3, Plan 2 complete

Use /gsd:progress to see next steps.
```

</edge_cases>

<success_criteria>
- [ ] State accurately diagnosed
- [ ] Recovery options clearly presented
- [ ] User choice executed successfully
- [ ] All lingering artifacts cleaned up
- [ ] Git state is clean or intentionally dirty
- [ ] STATE.md reflects current position
- [ ] User informed of next steps
</success_criteria>
