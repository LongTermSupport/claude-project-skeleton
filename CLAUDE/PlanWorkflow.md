# Claude Code Planning Workflow

**Version 1.0** | Effective: 24 November 2025

This document defines the standard planning workflow for Claude Code projects. All developers and AI agents should follow this workflow to ensure efficient, trackable, and high-quality work.

---

## 🎯 Core Principles

1. **Plan Before Execute** - Never start implementation without a documented plan
2. **Break Down Complexity** - Decompose large work into manageable tasks
3. **Track Everything** - Every task has a status and owner
4. **Document Decisions** - Capture rationale for major decisions
5. **Iterate Rapidly** - Plans are living documents, update as you learn

---

## 📁 Plan Structure

### Directory Layout

```
CLAUDE/
└── Plan/
    ├── 001-gemini-design-implementation/
    │   ├── PLAN.md                    # Main plan document - declarative for what needs doing and current status - no bloat
    │   ├── {supporint-docs*}.md          # Supporting analysis - extra info docs as required named as you see fit
    │   └── assets/                     # Screenshots, mockups, etc.
    ├── 002-folder-based-routing/
    │   ├── PLAN.md
    │   └── routing-spec.md
    └── README.md                       # Index of all plans
```

### Plan Numbering

- Plans are numbered sequentially: `001-`, `002-`, `003-`, etc.
- Use kebab-case for plan folder names
- Plan numbers never change even if plan is cancelled

---

## 📋 Plan Document Structure

Every `PLAN.md` must follow this structure:

```markdown
# Plan XXX: [Plan Title]

**Status**: 🟡 In Progress | 🟢 Complete | 🔴 Blocked | ⚫ Cancelled
**Created**: YYYY-MM-DD
**Owner**: [Name/Agent]
**Priority**: High | Medium | Low
**Estimated Effort**: [Hours/Days]

## Overview

[2-3 paragraphs describing what this plan aims to achieve and why]

## Goals

- Clear, measurable goal 1
- Clear, measurable goal 2
- Clear, measurable goal 3

## Non-Goals

- Explicitly what this plan will NOT do
- Helps scope creep management

## Context & Background

[Summary relevant background, previous decisions, or context needed]
Refer to detailed info in supporting docs as required

## Tasks

### Phase 1: [Phase Name]

- [ ] ⬜ **Task 1.1**: Description of task
  - [ ] ⬜ Subtask 1.1.1: More specific work
  - [ ] ⬜ Subtask 1.1.2: More specific work
- [ ] ⬜ **Task 1.2**: Description of task

### Phase 2: [Phase Name]

- [ ] ⬜ **Task 2.1**: Description of task
- [ ] ⬜ **Task 2.2**: Description of task

## Dependencies

- Depends on: Plan 001 (Complete)
- Blocks: Plan 003 (Not Started)
- Related: Plan 002

## Technical Decisions

### Decision 1: [Title]
**Context**: Why this decision is needed
**Options Considered**:
1. Option A - pros/cons
2. Option B - pros/cons

**Decision**: We chose Option A because [rationale]
**Date**: YYYY-MM-DD

## Success Criteria

- [ ] Criterion 1 that must be met
- [ ] Criterion 2 that must be met
- [ ] Criterion 3 that must be met

## Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Risk description | High/Med/Low | High/Med/Low | How we'll handle it |

## Timeline

- Phase 1: [Date range]
- Phase 2: [Date range]
- Target Completion: YYYY-MM-DD

## Notes & Updates

### YYYY-MM-DD
- Update or note about progress/changes

### YYYY-MM-DD
- Another update
```

---

## ✅ Task Status System

### Status Icons

Use these Unicode icons for task status:

| Status | Icon | Markdown | Meaning |
|--------|------|----------|---------|
| **Not Started** | ⬜ | `⬜` | Task not yet begun |
| **In Progress** | 🔄 | `🔄` | Currently being worked on |
| **Completed** | ✅ | `✅` | Task finished and verified |
| **Blocked** | 🚫 | `🚫` | Cannot proceed (dependency/issue) |
| **Cancelled** | ❌ | `❌` | Task no longer needed |
| **On Hold** | ⏸️ | `⏸️` | Paused temporarily |
| **Needs Review** | 👁️ | `👁️` | Work done, awaiting review |

### Task Formatting

```markdown
- [ ] ⬜ **Task Title**: Clear description of what needs to be done
  - [ ] ⬜ Subtask 1: Specific action
  - [ ] 🔄 Subtask 2: Another action (currently working)
  - [ ] ✅ Subtask 3: Already completed
```

### Rules for Status Updates

1. **One Task In Progress** - Limit to 1-2 tasks marked 🔄 at a time
2. **Update Immediately** - Change status as soon as state changes
3. **Document Blocks** - If marking 🚫, add note explaining why
4. **Verify Completion** - Only mark ✅ after testing/verification

---

## 🔄 Planning Workflow Steps

### Step 1: Identify Work

When new work is identified:

1. Check if it fits in an existing plan
2. If not, determine if a new plan is needed
3. For small tasks (< 1 hour), use TodoWrite instead

**New Plan Threshold**: If work will take > 2 hours or involves multiple phases

### Step 2: Create Plan

1. Create new folder: `CLAUDE/Plan/XXX-descriptive-name/`
2. Copy plan template to `PLAN.md`
3. Fill in overview, goals, and initial task breakdown
4. Update `CLAUDE/Plan/README.md` with plan entry

### Step 3: Break Down Tasks

1. Decompose work into phases (if needed)
2. Break each phase into concrete tasks
3. Break tasks into subtasks if task > 30 minutes
4. Ensure tasks are actionable and testable

**Good Task**: "Create Hero component with typewriter effect matching Gemini design"
**Bad Task**: "Work on homepage"

### Step 4: Review & Approve

1. Review plan completeness
2. Verify tasks are well-defined
3. Check for missing dependencies
4. Get stakeholder approval (if needed)

### Step 5: Execute

1. Mark plan status as 🔄 In Progress
2. Work through tasks sequentially
3. Update task status in real-time
4. Document any blockers or changes
5. Commit work with reference to plan: `Plan 001: Implement Hero component`

### Step 6: Complete

1. Verify all success criteria met
2. Mark all tasks as ✅
3. Mark plan status as 🟢 Complete
4. Add completion date to plan
5. Document any lessons learned

---

## 📝 Using TodoWrite vs Plans

### Use TodoWrite For:
- Very small Tasks and LOW RISK tasks
- Single-session work
- No major architectural decisions
- Temporary tracking during active work

### Use Plans For:
- Medium sized+
- Any risk
- Work with multiple phases
- Architectural or design decisions
- Work that may need to be resumed later
- Work that others need to understand

### Converting TodoWrite to Plan

If TodoWrite list grows beyond 5 items or becomes multi-session:
1. Create proper plan
2. Migrate tasks to plan
3. Clear TodoWrite
4. Reference plan in work

---

## 🎨 Plan Templates

### Feature Implementation Plan Template

```markdown
# Plan XXX: [Feature Name]

**Status**: ⬜ Not Started
**Type**: Feature Implementation
**Estimated Effort**: [X hours/days]

## Overview
[What feature, why needed]

## Tasks
- [ ] ⬜ Design component structure
- [ ] ⬜ Implement core functionality
- [ ] ⬜ Add styling
- [ ] ⬜ Write tests
- [ ] ⬜ Update documentation

## Success Criteria
- [ ] Feature works as specified
- [ ] Tests passing
- [ ] Documentation updated
```

### Bug Fix Plan Template

```markdown
# Plan XXX: Fix [Bug Description]

**Status**: ⬜ Not Started
**Type**: Bug Fix
**Severity**: Critical | High | Medium | Low

## Bug Description
[What's broken, how to reproduce]

## Tasks
- [ ] ⬜ Reproduce bug locally
- [ ] ⬜ Identify root cause
- [ ] ⬜ Implement fix
- [ ] ⬜ Add regression test
- [ ] ⬜ Verify fix works

## Success Criteria
- [ ] Bug no longer reproducible
- [ ] Test added to prevent regression
```

### Design Implementation Plan Template

```markdown
# Plan XXX: Implement [Design Name]

**Status**: ⬜ Not Started
**Type**: Design Implementation
**Reference**: [Link to design/mockup]

## Overview
[What design, design goals]

## Design Analysis
- Key visual elements
- Animations/interactions
- Responsive behavior

## Tasks
### Phase 1: Structure
- [ ] ⬜ Analyze reference design
- [ ] ⬜ Map components needed
- [ ] ⬜ Plan component hierarchy

### Phase 2: Implementation
- [ ] ⬜ Create base components
- [ ] ⬜ Implement styling
- [ ] ⬜ Add animations/effects
- [ ] ⬜ Responsive adjustments

### Phase 3: Polish
- [ ] ⬜ Fine-tune animations
- [ ] ⬜ Cross-browser testing
- [ ] ⬜ Performance optimization

## Success Criteria
- [ ] Visual match to reference design
- [ ] Animations smooth (60fps)
- [ ] Responsive on all breakpoints
```

---

## 🚀 Best Practices

### Task Writing

✅ **Good Tasks**:
- "Create Navigation component with mobile menu toggle"
- "Add typewriter animation to Hero heading (40ms/char)"
- "Implement blur-in effect for stats section"

❌ **Bad Tasks**:
- "Fix the website"
- "Make it look better"
- "Work on components"

### Task Granularity

- **Task**: 15-60 minutes of focused work
- **Subtask**: 5-15 minutes of specific action
- **Phase**: Group of related tasks (hours/days)

### Status Update Discipline

1. **Before starting work**: Review plan, mark task 🔄
2. **During work**: Update status if blocked
3. **After completing**: Mark ✅, commit with reference
4. **Daily**: Review plan, update progress notes

### Handling Changes

When requirements change mid-plan:

1. **Document Change**: Add note to plan with date
2. **Update Tasks**: Revise task list as needed
3. **Assess Impact**: Update estimates, dependencies
4. **Communicate**: Ensure stakeholders aware

---

## 🔍 Plan Reviews

### Daily Review (If actively working on plan)

- Are tasks up to date?
- Any blockers need attention?
- Is plan still on track?

### Weekly Review (For active plans)

- Progress vs timeline?
- Any scope changes needed?
- Dependencies still valid?

### Completion Review

- All success criteria met?
- Lessons learned documented?
- Follow-up work identified?

---

## 📊 Plan Metrics

Track these for each plan:

- **Planned vs Actual Effort**: Improve estimation
- **Blocker Count**: Identify process issues
- **Scope Changes**: Track requirements stability
- **Completion Rate**: % of tasks completed

---

## 🎯 Integration with Git

### Commit Messages

Reference plans in commits:

```
Plan 001: Implement Hero typewriter animation

- Add Typewriter component with configurable speed
- Integrate into Hero section
- Add delay/trigger props for sequencing

Refs: CLAUDE/Plan/001-gemini-design-implementation
```

### Branch Naming

For larger plans, use feature branches:

```
plan/001-gemini-design-implementation
plan/002-folder-routing-system
```

---

## 📚 Plan Index

Maintain `CLAUDE/Plan/README.md`:

```markdown
# Plans Index

## Active Plans
- [001: Gemini Design Implementation](001-gemini-design-implementation/PLAN.md) - 🔄 In Progress

## Completed Plans
- None yet

## Blocked Plans
- None

## Cancelled Plans
- None
```

---

## 🤖 AI Agent Guidelines

When Claude Code (or other AI agents) work on this project:

1. **Always check for existing plans** before starting work
2. **Create plan if none exists** for work > 2 hours
3. **Update task status in real-time** as you work
4. **Document blockers immediately** if you get stuck
5. **Ask user for approval** before marking plan complete
6. **Reference plans in all commits** for traceability

### Agent Workflow Example

```
User: "Implement the Gemini design"

Agent:
1. Checks CLAUDE/Plan/ for existing plan
2. If none, creates Plan 001
3. Breaks down into tasks
4. Shows plan to user for approval
5. Begins execution, updating status
6. Commits with "Plan 001: [work done]"
7. Updates plan progress notes
8. Marks complete when done
```

---

## ✨ Summary

**Remember**:
- 📋 Plan before you code
- ✅ Update status religiously
- 🎯 Keep tasks concrete and testable
- 📝 Document decisions and changes
- 🔄 Plans are living documents

**Questions?** See examples in `CLAUDE/Plan/` or ask in conversation.

---

**Last Updated**: December 2025
