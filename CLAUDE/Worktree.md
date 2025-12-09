# Worktree Workflow

Git worktree workflow for isolated, safe refactoring and development.

```bash
# ✅ CORRECT ORDER:
cd /workspace/untracked/worktrees/worktree-plan-XXX
git merge master --no-edit              # 1. Sync worktree with main FIRST
# Run your build/test command           # 2. Verify it still builds
cd /workspace
git merge worktree-plan-XXX             # 3. ONLY THEN merge to main

# ❌ WRONG - Will cause conflicts and lost work:
cd /workspace
git merge worktree-plan-XXX             # DON'T DO THIS WITHOUT STEP 1!
```

## Overview

Git worktrees allow multiple working directories from a single repository, enabling parallel development without branch-switching disruption. This is **essential** for running parallel agents on different tasks.

### Hierarchical Structure

Worktrees support **parent-child relationships** for complex plans:

- **Parent (Plan) Worktrees**: Top-level worktree for a plan (e.g., Plan 004)
- **Child (Task) Worktrees**: Individual tasks within the plan

**Merge Rules:**
- ✅ **ALLOWED**: Child → Parent worktree (automatic, no approval needed)
- ❌ **NOT ALLOWED**: Parent → Main project (requires human approval)

## Critical Rules

### 1. Worktree Location

**ALL worktree folders MUST be in:**
```
./untracked/worktrees/<branch-name>/
```

✅ **Correct**: `./untracked/worktrees/worktree-plan-004/`
❌ **Wrong**: `../plan-004/`, `/tmp/worktree/`, etc.

**Why**:
- Keeps workspace organised
- Prevents git confusion
- Easy cleanup (just delete `untracked/worktrees/`)
- Excluded from main repo operations

### 2. Branch Naming

**Parent (Plan) Worktrees:**
- Prefix: `worktree-`
- Format: `worktree-<plan-name>`
- ✅ Examples: `worktree-plan-004`, `worktree-header-refactor`

**Child (Task) Worktrees:**
- Prefix: `worktree-child-`
- Format: `worktree-child-<parent-name>-<task-name>`
- ✅ Examples: `worktree-child-plan-004-component-1`, `worktree-child-plan-004-refactor-about`

❌ **Wrong**: `plan-004`, `feature/headers`, `temp-work`, `child-component-1`

**Why**:
- Clear identification of worktree hierarchy
- Easy filtering in `git branch` output
- Shows parent-child relationships
- Prevents accidental merges to main
- Signals temporary nature

### 3. Agent Containment

**Agents working in worktrees MUST stay in their worktree**

When launching sub-agents for worktree tasks:
- Set working directory explicitly
- Verify agent is in correct worktree
- Never `cd` back to main workspace
- All file operations relative to worktree root

**Example agent prompt:**
```
You are working in a git worktree at /workspace/untracked/worktrees/worktree-plan-004/
DO NOT work in /workspace - only work in YOUR worktree directory.
All file paths should be relative to /workspace/untracked/worktrees/worktree-plan-004/
```

### 4. Merge Protocol

**Two types of merges with different rules:**

#### Child → Parent Worktree (ALLOWED)

✅ **Can merge automatically** - no human approval needed

```bash
# From parent worktree
cd untracked/worktrees/worktree-plan-004
git merge worktree-child-plan-004-component-1
```

**Why allowed:**
- Isolated to plan worktree
- Doesn't affect main project
- Part of plan execution workflow
- Easy to rollback if needed

#### Parent → Main Project (REQUIRES APPROVAL)

❌ **MUST ask human approval first**

Before merging parent to main:
1. ✋ **STOP** - Ask human for approval
2. Verify no other agents working in main workspace
3. Verify no conflicts with main branch
4. Get explicit "yes" from human
5. Only then proceed with merge

**Why requires approval:**
- Multiple agents may be working simultaneously
- Main workspace might have uncommitted changes
- Conflicts need human resolution
- Risk of losing work

### 5. Cleanup Protocol

**On merge completion, ALL worktree branches and folders MUST be cleaned up:**

```bash
# After child merges to parent, delete child immediately
git worktree remove untracked/worktrees/worktree-child-plan-004-component-1
git branch -d worktree-child-plan-004-component-1

# After parent merges to main, delete parent immediately
git worktree remove untracked/worktrees/worktree-plan-004
git branch -d worktree-plan-004
```

**Why mandatory:**
- Prevents worktree clutter
- Reduces confusion about active work
- Frees disk space
- Keeps git branch list clean

## Standard Worktree Workflow

### Creating a Parent (Plan) Worktree

```bash
# 1. Ensure untracked/worktrees directory exists
mkdir -p untracked/worktrees

# 2. Create parent worktree from main branch
cd /workspace
git worktree add untracked/worktrees/worktree-plan-004 -b worktree-plan-004

# 3. Verify creation
git worktree list
```

### Creating a Child (Task) Worktree

```bash
# 1. Create child from parent worktree branch
cd /workspace
git worktree add untracked/worktrees/worktree-child-plan-004-component-1 \
  -b worktree-child-plan-004-component-1 worktree-plan-004

# 2. Verify it's based on parent
git worktree list
git log --oneline -5
```

**Note**: Child worktree is branched FROM the parent worktree branch, not from main.

### Working in a Worktree

```bash
# Navigate to worktree
cd untracked/worktrees/worktree-plan-004

# Work normally - commits, edits, builds
git status
# ... do your work ...
git add .
git commit -m "Implement feature"

# Return to main workspace
cd /workspace
```

### Merging Child → Parent Worktree

```bash
# From parent worktree directory
cd untracked/worktrees/worktree-plan-004

# 1. Review child changes
git log worktree-child-plan-004-component-1

# 2. Merge child into parent (no approval needed)
git merge worktree-child-plan-004-component-1

# 3. Immediately cleanup child
cd /workspace
git worktree remove untracked/worktrees/worktree-child-plan-004-component-1
git branch -d worktree-child-plan-004-component-1
```

### Merging Parent → Main Project

**CRITICAL**: This is a multi-step process that requires careful verification at each stage.

⚠️ **MERGE ORDER IS CRITICAL** ⚠️
**ALWAYS merge master → worktree FIRST, then worktree → main**
**NEVER merge worktree → main directly!**

```bash
# ===================================================================
# STEP 1: ALWAYS MERGE MASTER INTO WORKTREE FIRST!
# ===================================================================
# This is THE MOST IMPORTANT STEP - sync worktree with main BEFORE merging back
# Prevents conflicts and ensures worktree has all latest changes from main
cd /workspace/untracked/worktrees/worktree-plan-004
git fetch origin
git merge master --no-edit
# ⚠️ If there are conflicts, resolve them HERE in the worktree
# ⚠️ Test thoroughly after merge - the worktree must build/test cleanly
# Run your project's build/test commands here

# STEP 2: Verify main workspace is clean
cd /workspace
git checkout master
git pull
git status  # MUST show "nothing to commit, working tree clean"

# ✋ STOP - If main workspace has uncommitted changes:
#   - Commit them first OR
#   - Stash them: git stash
#   - DO NOT proceed until main is clean

# STEP 3: ✋ STOP - Ask human for final approval!
# Confirm with human:
#   - Is main branch clean? (no uncommitted changes)
#   - Are all other agents/processes stopped?
#   - Is it safe to merge now?

# STEP 4: Review parent worktree changes
git log worktree-plan-004 --oneline

# STEP 5: Merge parent to main (only after ALL approvals above!)
git merge worktree-plan-004 --no-edit

# STEP 6: Verify merge succeeded
git status  # Should show clean state
# Run your build/test commands to verify

# STEP 7: Push to origin
git push

# STEP 8: ONLY NOW cleanup parent worktree
# (Not before - we need it in case merge fails)
git worktree remove untracked/worktrees/worktree-plan-004
git branch -D worktree-plan-004

# STEP 9: Final verification
git status  # Confirm everything clean
```

**Why this order matters:**
1. **ALWAYS sync worktree first** (`master → worktree`):
   - Prevents conflicts by updating worktree with main's latest changes
   - Lets you resolve conflicts IN THE WORKTREE (isolated, safe)
   - Ensures your changes work with the current state of main
   - **If you skip this, the merge WILL conflict and you WILL lose work**
2. **Clean main workspace**: Uncommitted changes in main cause merge failures
3. **Human verification**: Ensures no other work is in progress
4. **Keep worktree until success**: Don't delete until merge is confirmed working
5. **Cleanup last**: Only remove worktree after everything pushed successfully

**Remember**: The worktree is your isolated workspace. ALWAYS bring main's changes INTO your workspace BEFORE you merge your workspace back to main!

### Cleaning Up

Cleanup is **mandatory and immediate** after merging:

```bash
# After merging child to parent
git worktree remove untracked/worktrees/worktree-child-plan-004-component-1
git branch -d worktree-child-plan-004-component-1

# After merging parent to main
git worktree remove untracked/worktrees/worktree-plan-004
git branch -d worktree-plan-004
```

**Never leave merged worktrees around** - cleanup prevents confusion and keeps workspace tidy.

## Parallel Agent Strategy

### Hierarchical Approach: Plan 004 (4 components, 4 refactors)

**Step 1: Create Parent (Plan) Worktree**

```bash
# Main orchestrator creates parent worktree for the plan
cd /workspace
git worktree add untracked/worktrees/worktree-plan-004 -b worktree-plan-004
```

**Step 2: Wave 1 - Create Components (4 child agents in parallel)**

```bash
# Create 4 child worktrees from parent
cd /workspace
git worktree add untracked/worktrees/worktree-child-plan-004-statusbadge \
  -b worktree-child-plan-004-statusbadge worktree-plan-004

git worktree add untracked/worktrees/worktree-child-plan-004-cardtitle \
  -b worktree-child-plan-004-cardtitle worktree-plan-004

git worktree add untracked/worktrees/worktree-child-plan-004-sectionheading \
  -b worktree-child-plan-004-sectionheading worktree-plan-004

git worktree add untracked/worktrees/worktree-child-plan-004-subheading \
  -b worktree-child-plan-004-subheading worktree-plan-004

# Launch 4 agents, each in their child worktree
Agent 1 → StatusBadge.tsx in worktree-child-plan-004-statusbadge
Agent 2 → CardTitle.tsx in worktree-child-plan-004-cardtitle
Agent 3 → SectionHeading.tsx in worktree-child-plan-004-sectionheading
Agent 4 → SubHeading.tsx in worktree-child-plan-004-subheading
```

**Step 3: Merge Children into Parent (sequential, in parent worktree)**

```bash
# From parent worktree, merge all children
cd untracked/worktrees/worktree-plan-004

git merge worktree-child-plan-004-statusbadge
git worktree remove ../worktree-child-plan-004-statusbadge
git branch -d worktree-child-plan-004-statusbadge

git merge worktree-child-plan-004-cardtitle
git worktree remove ../worktree-child-plan-004-cardtitle
git branch -d worktree-child-plan-004-cardtitle

# ... repeat for other children
```

**Step 4: Wave 2 - Refactor Sections (4 child agents in parallel)**

```bash
# Create 4 new child worktrees from updated parent
cd /workspace
git worktree add untracked/worktrees/worktree-child-plan-004-about \
  -b worktree-child-plan-004-about worktree-plan-004

git worktree add untracked/worktrees/worktree-child-plan-004-services \
  -b worktree-child-plan-004-services worktree-plan-004

# ... etc

# Launch 4 agents for refactoring
Agent 1 → About.tsx in worktree-child-plan-004-about
Agent 2 → ServicesDetailed.tsx in worktree-child-plan-004-services
# ... etc
```

**Step 5: Merge Children into Parent (sequential)**

```bash
# Same process as Step 3
cd untracked/worktrees/worktree-plan-004
git merge worktree-child-plan-004-about
# ... cleanup and repeat
```

**Step 6: Test in Parent Worktree**

```bash
# From parent worktree
cd untracked/worktrees/worktree-plan-004
# Run your build/lint/test commands
# ... verify everything works
```

**Step 7: Merge Parent into Main (REQUIRES APPROVAL)**

```bash
# ✋ STOP - Ask human for approval!
cd /workspace
git checkout master
git merge worktree-plan-004
git push

# Cleanup parent
git worktree remove untracked/worktrees/worktree-plan-004
git branch -d worktree-plan-004
```

### Benefits of Hierarchical Approach

1. **All plan work isolated**: Parent worktree contains entire plan
2. **Clean main workspace**: Main repo unaffected until final approval
3. **Easy rollback**: Can abandon entire plan without affecting main
4. **Parallel within plan**: Multiple agents work on tasks simultaneously
5. **Sequential integration**: Tasks merge to parent, then parent merges to main
6. **Clear hierarchy**: Easy to see which tasks belong to which plan

## Common Pitfalls

### ❌ Working in Wrong Directory

```bash
# Agent creates component in main workspace instead of worktree
cd /workspace  # WRONG
touch src/components/ui/StatusBadge.tsx  # WRONG LOCATION
```

✅ **Solution**: Always verify `pwd` before file operations

### ❌ Branch Name Confusion

```bash
# Creating branch without worktree- prefix
git worktree add untracked/worktrees/plan-004 -b plan-004  # WRONG
```

✅ **Solution**: Always use `worktree-` prefix

### ❌ Merging Without Approval

```bash
# Agent automatically merges after completing task
git merge worktree-plan-004  # WRONG - no human approval
```

✅ **Solution**: Always ask human before merging

### ❌ Forgetting Cleanup

```bash
# Leaving old worktrees around
$ git worktree list
/workspace                              abc1234 [master]
/workspace/untracked/worktrees/old-1    def5678 [worktree-old-1]
/workspace/untracked/worktrees/old-2    ghi9012 [worktree-old-2]
```

✅ **Solution**: Clean up after merging

## Benefits

1. **Parallel Work**: Multiple agents working simultaneously
2. **Isolation**: Changes don't interfere with each other
3. **Safety**: Main workspace remains stable
4. **Speed**: No branch switching overhead
5. **Clarity**: Clear separation of tasks

## When to Use Worktrees

✅ **Use worktrees for:**
- Multi-task plans with parallel phases
- Component creation that can be done independently
- Refactoring multiple files simultaneously
- Any work requiring 2+ parallel agents

❌ **Don't use worktrees for:**
- Single-file edits
- Quick fixes
- Sequential work where context matters
- Exploratory work (use main workspace)

## Directory Structure

```
/workspace/
├── .git/                                           # Main git directory
├── untracked/                                      # Not tracked by git
│   └── worktrees/                                  # All worktrees here
│       ├── worktree-plan-004/                      # Parent (Plan) worktree
│       │   ├── src/
│       │   ├── package.json
│       │   └── ... (full copy of repo)
│       ├── worktree-child-plan-004-statusbadge/    # Child (Task) worktree
│       │   └── ... (full copy, branched from parent)
│       ├── worktree-child-plan-004-cardtitle/      # Child (Task) worktree
│       │   └── ... (full copy, branched from parent)
│       └── worktree-child-plan-004-about/          # Child (Task) worktree
│           └── ... (full copy, branched from parent)
├── src/                                            # Main workspace files
├── package.json
└── ...
```

**Hierarchy:**
- Main project (`/workspace/`) ← Parent worktrees merge here (with approval)
- Parent worktrees (`worktree-plan-004`) ← Child worktrees merge here (automatic)
- Child worktrees (`worktree-child-*`) ← Individual tasks worked on here

## Verification Checklist

### Before Creating Parent Worktree:
- [ ] Directory will be `untracked/worktrees/worktree-<plan-name>`
- [ ] Branch name starts with `worktree-` (not `worktree-child-`)
- [ ] Branching from main/master

### Before Creating Child Worktree:
- [ ] Directory will be `untracked/worktrees/worktree-child-<parent>-<task>`
- [ ] Branch name starts with `worktree-child-`
- [ ] Branch name includes parent plan name
- [ ] Branching from parent worktree branch (not main!)
- [ ] Agent knows to stay in their child worktree

### Before Merging Child → Parent:
- [ ] Working in parent worktree directory
- [ ] Reviewed child changes
- [ ] No conflicts expected
- [ ] Ready to cleanup child immediately after merge

### Before Merging Parent → Main:
- [ ] ✋ **STEP 1**: ⚠️ **MERGED MASTER INTO WORKTREE FIRST** ⚠️ (`cd worktree && git merge master`)
- [ ] ✋ **STEP 2**: Resolved any conflicts in parent worktree (NOT in main!)
- [ ] ✋ **STEP 3**: Tested in parent worktree after sync (run build/test commands)
- [ ] ✋ **STEP 4**: Verified main workspace is clean (`git status` shows clean)
- [ ] ✋ **STEP 5**: Committed or stashed any uncommitted changes in main workspace
- [ ] ✋ **STEP 6**: Asked human for final approval
- [ ] ✋ **STEP 7**: Got explicit "yes" from human
- [ ] ✋ **STEP 8**: Confirmed no other agents/processes working in main workspace
- [ ] ✋ **STEP 9**: Reviewed changes one last time (`git log worktree-plan-xxx --oneline`)

**REMINDER**: The merge order is ALWAYS: `master → worktree` FIRST, then `worktree → main`

### After Merging Child → Parent:
- [ ] Removed child worktree folder immediately
- [ ] Deleted child branch immediately
- [ ] Verified parent worktree still works

### After Merging Parent → Main:
- [ ] ✋ **STEP 10**: Verified merge succeeded (`git status` shows clean)
- [ ] ✋ **STEP 11**: Verified build still works (run your build command)
- [ ] ✋ **STEP 12**: Pushed to origin successfully (`git push`)
- [ ] ✋ **STEP 13**: ONLY NOW remove parent worktree folder
- [ ] ✋ **STEP 14**: ONLY NOW delete parent branch
- [ ] ✋ **STEP 15**: Final verification (`git status` clean)
- [ ] ✋ **STEP 16**: Updated plan status to completed

**CRITICAL**: Never remove worktree/branch before merge is pushed successfully!

## Troubleshooting

### "Fatal: invalid reference: worktree-plan-004"

**Cause**: Branch doesn't exist yet
**Solution**: Use `-b` flag when creating worktree

### "Lock file exists"

**Cause**: Previous worktree operation was interrupted
**Solution**: `git worktree prune` to clean up

### "Already exists"

**Cause**: Worktree folder wasn't properly removed
**Solution**: Manual cleanup:
```bash
rm -rf untracked/worktrees/worktree-plan-004
git worktree prune
```

### Agent Working in Wrong Place

**Symptoms**: Files appearing in main workspace instead of worktree
**Solution**:
1. Stop agent immediately
2. Verify agent's working directory
3. Move files to correct worktree
4. Remind agent of worktree location

### Child Worktree Created from Main Instead of Parent

**Symptoms**: Child worktree doesn't have parent's changes
**Solution**:
1. Remove incorrect child worktree
2. Recreate child from parent branch:
   ```bash
   git worktree remove untracked/worktrees/worktree-child-plan-004-task
   git branch -D worktree-child-plan-004-task
   git worktree add untracked/worktrees/worktree-child-plan-004-task \
     -b worktree-child-plan-004-task worktree-plan-004
   ```

### Trying to Merge Parent to Main Without Approval

**Symptoms**: Agent attempts `git merge worktree-plan-004` from main workspace
**Solution**:
1. Stop immediately
2. Undo merge if it happened: `git merge --abort`
3. Ask human for approval
4. Only proceed after explicit "yes"

## Quick Reference

### Worktree Hierarchy

```
Main Project (/workspace/)
    ↑
    │ (merge with human approval)
    │
Parent Worktree (worktree-plan-004)
    ↑
    │ (merge automatically)
    │
Child Worktrees (worktree-child-plan-004-*)
```

### Key Rules Summary

| Action | Approval Required | Cleanup |
|--------|------------------|---------|
| Create parent worktree | No | After merge to main |
| Create child worktree | No | After merge to parent |
| Merge child → parent | **NO** | Immediate |
| Merge parent → main | **YES** | Immediate |

### Naming Cheat Sheet

```bash
# Parent (Plan) Worktree
worktree-plan-004
worktree-header-refactor

# Child (Task) Worktree - must include parent name!
worktree-child-plan-004-statusbadge
worktree-child-plan-004-refactor-about
worktree-child-header-refactor-component-1
```

### Common Commands

```bash
# Create parent from main
git worktree add untracked/worktrees/worktree-plan-004 -b worktree-plan-004

# Create child from parent
git worktree add untracked/worktrees/worktree-child-plan-004-task \
  -b worktree-child-plan-004-task worktree-plan-004

# Merge child to parent (in parent directory)
cd untracked/worktrees/worktree-plan-004
git merge worktree-child-plan-004-task

# Cleanup child immediately
cd /workspace
git worktree remove untracked/worktrees/worktree-child-plan-004-task
git branch -d worktree-child-plan-004-task

# Merge parent to main (ASK HUMAN FIRST!)
cd /workspace
git merge worktree-plan-004

# Cleanup parent immediately
git worktree remove untracked/worktrees/worktree-plan-004
git branch -d worktree-plan-004
```

## References

- Git worktree docs: `git help worktree`
- Main project docs: `CLAUDE.md`
- Plan workflow: `CLAUDE/PlanWorkflow.md`

---

**Remember**:
- Worktrees are for **parallel execution** on complex plans
- **Parent worktrees** isolate entire plans from main project
- **Child worktrees** allow parallel work within a plan
- **Always cleanup** immediately after merging
- **Always ask human** before merging parent to main
