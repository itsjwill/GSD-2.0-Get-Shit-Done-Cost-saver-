---
description: Rollback to a previous checkpoint when things go wrong
argument-hint: "[phase-plan] or 'last'"
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
---

<objective>
Safely rollback to a previous GSD checkpoint, undoing changes from failed or unwanted plan executions.

GSD 2.0 creates git tags before and after each plan execution, enabling clean rollback to any checkpoint.
</objective>

<process>

<step name="list_checkpoints">
## List Available Checkpoints

```bash
# Find all GSD checkpoints
git tag | grep "gsd-checkpoint" | sort -V | tail -20
```

Display format:
```
Available checkpoints:
  gsd-checkpoint-01-01-pre    (before Phase 1, Plan 1)
  gsd-checkpoint-01-01-post   (after Phase 1, Plan 1)
  gsd-checkpoint-01-02-pre    (before Phase 1, Plan 2)
  gsd-checkpoint-01-02-post   (after Phase 1, Plan 2)
  gsd-checkpoint-02-01-pre    (before Phase 2, Plan 1) ← most recent
```

If no checkpoints exist:
```
No GSD checkpoints found.
Checkpoints are created automatically during /gsd:execute-plan in GSD 2.0.
```
</step>

<step name="parse_target">
## Determine Rollback Target

**If argument is "last":**
- Find most recent `-pre` tag
- This undoes the last plan execution

**If argument is "{phase}-{plan}":**
- Target: `gsd-checkpoint-{phase}-{plan}-pre`
- This undoes that specific plan and all subsequent work

**If no argument:**
Use AskUserQuestion:
- header: "Rollback Target"
- question: "Which checkpoint do you want to rollback to?"
- options: [Last 3 checkpoints] + "Show all checkpoints"
</step>

<step name="show_impact">
## Show What Will Be Undone

```bash
# Show commits that will be undone
git log {target-tag}..HEAD --oneline
```

```bash
# Show files that will change
git diff {target-tag} HEAD --stat
```

Present to user:
```
Rollback to: gsd-checkpoint-02-01-pre

This will undo:
  • 3 commits
  • 8 files changed

Commits to be undone:
  abc1234 feat(02-01): implement user authentication
  def5678 feat(02-01): add password hashing
  ghi9012 docs(02-01): complete auth plan

Files affected:
  src/auth/login.ts       | 45 +++++
  src/auth/middleware.ts  | 32 +++++
  .planning/phases/02-auth/02-01-SUMMARY.md | 80 +++++
```
</step>

<step name="confirm">
## Confirm Rollback

Use AskUserQuestion:
- header: "Confirm Rollback"
- question: "This will permanently undo the changes above. Proceed?"
- options:
  - "Yes, rollback" — Execute the rollback
  - "No, cancel" — Abort and keep current state
  - "Show more details" — Display full diff
</step>

<step name="execute_rollback">
## Execute Rollback

**If confirmed:**

1. Create safety backup:
   ```bash
   git tag gsd-rollback-backup-$(date +%Y%m%d%H%M%S)
   ```

2. Reset to checkpoint:
   ```bash
   git reset --hard {target-tag}
   ```

3. Clean up post-rollback artifacts:
   - Delete SUMMARY.md files for rolled-back plans
   - Delete ISSUES.md entries from rolled-back plans (if any)
   - Delete .continue-here.md if exists

4. Update STATE.md:
   ```markdown
   ## Current Position
   Phase: {rolled-back-to-phase}
   Plan: Ready to execute
   Status: Rolled back from {original-position}
   Last activity: {date} - Rolled back to checkpoint {target}

   ## Rollback History
   - {date}: Rolled back from {phase}-{plan} to {target}
     Reason: User-initiated rollback
     Backup tag: gsd-rollback-backup-{timestamp}
   ```

5. Report success:
   ```
   ✓ Rolled back to gsd-checkpoint-{target}

   Backup created: gsd-rollback-backup-{timestamp}
   (Use this to undo the rollback if needed)

   Current position: Phase {X}, ready to execute Plan {Y}

   Next: /gsd:execute-plan or /gsd:progress
   ```
</step>

<step name="handle_conflicts">
## Handle Edge Cases

**If uncommitted changes exist:**
```
⚠️ You have uncommitted changes that would be lost.

Options:
1. Stash changes (git stash)
2. Commit changes first
3. Discard changes and rollback anyway
```

**If working in the middle of a plan:**
```
⚠️ It looks like a plan is currently executing.

The current agent may still be running.
Consider using /gsd:recover instead.
```

**If target checkpoint doesn't exist:**
```
❌ Checkpoint gsd-checkpoint-{target} not found.

Available checkpoints:
  [list checkpoints]

Hint: Checkpoints are created during /gsd:execute-plan
```
</step>

</process>

<undo_rollback>
## Undoing a Rollback

If you rollback and want to undo:

```bash
# Find your backup tag
git tag | grep gsd-rollback-backup

# Reset to the backup
git reset --hard gsd-rollback-backup-{timestamp}
```

Or use this command:
```
/gsd:rollback undo
```
</undo_rollback>

<integration>
## How Checkpoints Are Created

During `/gsd:execute-plan`:

**Before execution:**
```bash
git tag gsd-checkpoint-{phase}-{plan}-pre
```

**After successful execution:**
```bash
git tag gsd-checkpoint-{phase}-{plan}-post
```

This ensures every plan execution has a clean rollback point.
</integration>

<success_criteria>
- [ ] Target checkpoint identified
- [ ] Impact clearly shown to user
- [ ] User confirmed rollback
- [ ] Backup tag created
- [ ] Git reset executed successfully
- [ ] STATE.md updated with rollback history
- [ ] Rolled-back artifacts cleaned up
- [ ] User informed of next steps
</success_criteria>
