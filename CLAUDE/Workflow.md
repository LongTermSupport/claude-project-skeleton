# Workflow State Management

**CANONICAL DOCUMENTATION** - Single source of truth for workflow concepts and state management.

## What is a Workflow?

A **workflow** is structured sub-agent orchestration with:
- **Clearly defined steps** - Sequential phases with specific objectives
- **Gates** - Validation checkpoints between phases
- **Compaction resilience** - Must survive multiple conversation compaction cycles
- **State tracking** - Persistent state file tracking progress and context
- **Required reading** - Documentation that must be read at workflow start/resume

Workflows are NOT ad-hoc task sequences. They are formal, documented processes with explicit phase tracking.

## Workflow Types

| Type | Description | Phases | Example |
|------|-------------|--------|---------|
| `planning` | Planning workflow | 6 | Research → Design → Plan → Review |
| `qa-loop` | Quality assurance loop | 2-3 | Run QA → Fix Issues → Verify |
| `implementation` | Feature implementation | Variable | Design → Build → Test → Deploy |
| `custom` | Custom workflows | Variable | Any structured process |

## State File Lifecycle

### 1. Workflow START

**When**: Agent begins a formal workflow

**Action**: Create state file

**Location**: `./untracked/workflow-state/{workflow-name}/state-{workflow-name}-{start-time}.json`

**Example**:
```bash
./untracked/workflow-state/feature-implementation/state-feature-implementation-20251209_143052.json
```

**Agent Responsibility**: Create state file at workflow start with:
- Workflow name and type
- Initial phase (1/total)
- Required reading list (with @ syntax)
- Context variables (plan number, page name, etc.)
- Key reminders

### 2. Phase Transitions

**When**: Moving from one phase to another

**Action**: UPDATE existing state file (same file, preserve `created_at`)

**Agent Responsibility**: Update state file with:
- New phase number and name
- Updated status
- Any new context or reminders

### 3. PreCompact Hook

**When**: Before conversation compaction occurs

**Action**: PreCompact hook UPDATES existing state file

**How it works**:
1. Hook detects active workflow (checks CLAUDE.local.md or active plans)
2. Extracts current workflow state
3. Looks for existing state file matching workflow name
4. **If found**: UPDATES file with current state (preserves `created_at`)
5. **If not found**: CREATES new file with start_time timestamp

**No agent action needed** - hook handles this automatically.

### 4. Compaction

**When**: Conversation context window fills up

**What happens**: State file survives (filesystem-based, not in conversation)

### 5. SessionStart Hook (Post-Compaction)

**When**: Session resumes after compaction

**Action**: SessionStart hook READS state file (DOES NOT DELETE)

**How it works**:
1. Hook finds all state files in `./untracked/workflow-state/*/`
2. Reads most recently modified state file
3. Provides guidance to agent with:
   - Workflow name and current phase
   - **REQUIRED READING** with @ syntax (forces file reading)
   - Key reminders
   - Context variables
4. **DOES NOT DELETE** - file persists

**Agent sees guidance** with workflow context and must read required files.

### 6. Continue Workflow

**When**: Agent continues working through phases

**Action**: Agent continues updating state file at phase transitions

**Agent Responsibility**: Keep state file current.

### 7. Workflow COMPLETE

**When**: All phases complete, workflow finished

**Action**: DELETE state file

**Agent Responsibility**: Remove state file when workflow is done.

**Example**:
```bash
rm -rf ./untracked/workflow-state/page-orchestration/
```

## State File Format

### Required Fields

```json
{
  "workflow": "Feature Implementation (User Auth)",
  "workflow_type": "implementation",
  "phase": {
    "current": 2,
    "total": 5,
    "name": "Build Components",
    "status": "in_progress"
  },
  "required_reading": [
    "@CLAUDE/PlanWorkflow.md",
    "@CLAUDE/Plan/001-user-auth/PLAN.md"
  ],
  "context": {
    "plan_number": 1,
    "feature_name": "user-auth"
  },
  "key_reminders": [
    "Use existing auth patterns from codebase",
    "Run tests after each component"
  ],
  "created_at": "2025-12-09T14:30:52.123456Z"
}
```

### Field Descriptions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `workflow` | string | ✅ | Human-readable workflow name |
| `workflow_type` | enum | ✅ | Machine-readable type (see Workflow Types above) |
| `phase.current` | integer | ✅ | Current phase number (1-indexed) |
| `phase.total` | integer | ✅ | Total number of phases |
| `phase.name` | string | ✅ | Human-readable phase name |
| `phase.status` | enum | ✅ | `not_started` \| `in_progress` \| `completed` \| `blocked` |
| `required_reading` | array | ✅ | File paths with **@ prefix** to force reading |
| `context` | object | ⬜ | Flexible key-value pairs (workflow-specific) |
| `key_reminders` | array | ⬜ | Critical rules that must survive compaction |
| `created_at` | string | ✅ | ISO 8601 timestamp (preserved across updates) |

### @ Syntax for REQUIRED READING

**CRITICAL**: All file paths in `required_reading` must use **@ prefix**.

**Why**: The @ syntax forces agents to actually READ files, not just be contextually aware of them.

**Examples**:
- `@CLAUDE/PageOrchestration.md` - Forces reading
- `@CLAUDE/Sitemap/case-studies.md` - Forces reading
- `@.claude/skills/page-orchestration/SKILL.md` - Forces reading

**When agent resumes after compaction**: SessionStart hook provides these files with @ syntax in guidance, ensuring agent reads them.

## Directory Structure

```
./untracked/workflow-state/
├── feature-implementation/
│   └── state-feature-implementation-20251209_143052.json
├── qa-loop/
│   └── state-qa-loop-20251209_150000.json
└── plan-001/
    └── state-plan-001-20251209_160000.json
```

**Benefits**:
- One directory per workflow (clean organization)
- Multiple concurrent workflows supported
- Easy to find and manage workflow state
- Safe workflow name sanitization (spaces → hyphens)

## Workflow Detection

The PreCompact hook detects active workflows using:

### Method 1: CLAUDE.local.md Markers

If `CLAUDE.local.md` exists and contains:
- `"WORKFLOW STATE"` marker
- `"workflow:"` field
- `"Phase:"` field

**Example**:
```markdown
# Current Workflow State

Workflow: Page Orchestration - Case Studies
Phase: 4/10 - SEO Generation

@CLAUDE/PageOrchestration.md
@CLAUDE/Sitemap/case-studies.md
```

### Method 2: Active Plans

If `CLAUDE/Plan/*/PLAN.md` exists with:
- `🔄 In Progress` status
- `"phase"` or `"workflow"` keywords in content

**Generic Detection**: The question is "Are you in a formal workflow?" - NOT a hardcoded whitelist.

## Compaction Resilience

### The Problem

Conversation compaction loses:
- Procedural workflow details (phase tracking)
- Document references (which files to read)
- Conditional logic (e.g., "skip Phase 3" becomes vague)
- Context state (plan numbers, page names)

**Example failure**: Agent hallucinates incorrect workflow state when context is lost.

### The Solution

**State file + hooks** = Compaction resilience

1. **PreCompact hook** saves complete workflow state to file BEFORE compaction
2. **Compaction** happens (conversation cleared)
3. **State file survives** (filesystem, not conversation)
4. **SessionStart hook** reads state file and provides guidance with @ syntax
5. **Agent reads required docs** (forced by @ syntax)
6. **Workflow continues** without loss of context

### Multiple Compaction Cycles

State files persist across MULTIPLE compactions:
- PreCompact updates existing file (cycle 1)
- Compaction happens (cycle 1)
- SessionStart reads file (cycle 1)
- Work continues, PreCompact updates file again (cycle 2)
- Compaction happens (cycle 2)
- SessionStart reads file (cycle 2)
- etc.

**Lossless preservation** across unlimited compaction cycles.

## Agent Responsibilities

### At Workflow Start

1. **Create state file** in `./untracked/workflow-state/{workflow-name}/`
2. Use sanitized workflow name (lowercase, hyphens, no spaces)
3. Include start_time timestamp in filename
4. Populate all required fields
5. Use **@ syntax** for all `required_reading` paths

### During Workflow

1. **Update state file** at each phase transition
2. Keep `phase.current`, `phase.name`, `phase.status` current
3. Preserve original `created_at` timestamp
4. Add context or reminders as needed

### After Compaction (SessionStart Guidance)

1. **Acknowledge hook guidance**: "✅ Workflow restored: {workflow}, Phase {current}/{total}"
2. **Read ALL required files** using @ syntax
3. **Confirm understanding** of current phase and context
4. **Do NOT proceed** with assumptions or hallucinated logic

### At Workflow Completion

1. **Delete state file** and directory:
   ```bash
   rm -rf ./untracked/workflow-state/{workflow-name}/
   ```
2. Clean up after yourself

## Examples

### Example 1: Feature Implementation

**Start** (Phase 1 - Design):
```json
{
  "workflow": "Feature Implementation (User Auth)",
  "workflow_type": "implementation",
  "phase": {"current": 1, "total": 5, "name": "Design", "status": "in_progress"},
  "required_reading": [
    "@CLAUDE/PlanWorkflow.md",
    "@CLAUDE/Plan/001-user-auth/PLAN.md"
  ],
  "context": {"plan_number": 1, "feature_name": "user-auth"},
  "key_reminders": ["Follow existing auth patterns"],
  "created_at": "2025-12-09T14:30:52Z"
}
```

**Update** (Phase 3 - Testing):
```json
{
  "workflow": "Feature Implementation (User Auth)",
  "workflow_type": "implementation",
  "phase": {"current": 3, "total": 5, "name": "Testing", "status": "in_progress"},
  "required_reading": [
    "@CLAUDE/PlanWorkflow.md",
    "@CLAUDE/Plan/001-user-auth/PLAN.md"
  ],
  "context": {"plan_number": 1, "feature_name": "user-auth"},
  "key_reminders": [
    "Follow existing auth patterns",
    "Run integration tests after unit tests"
  ],
  "created_at": "2025-12-09T14:30:52Z"
}
```

### Example 2: QA Loop

```json
{
  "workflow": "QA Loop",
  "workflow_type": "qa-loop",
  "phase": {"current": 2, "total": 3, "name": "Fixing Errors", "status": "in_progress"},
  "required_reading": ["@CLAUDE/PlanWorkflow.md"],
  "context": {"iteration": 3},
  "key_reminders": [
    "Run tests between iterations",
    "Fix type errors before linting"
  ],
  "created_at": "2025-12-09T15:00:00Z"
}
```

### Example 3: Custom Workflow

```json
{
  "workflow": "Database Migration",
  "workflow_type": "custom",
  "phase": {"current": 3, "total": 5, "name": "Data Transformation", "status": "in_progress"},
  "required_reading": [
    "@docs/migration-guide.md",
    "@scripts/migrate.py"
  ],
  "context": {
    "source_db": "postgres",
    "target_db": "mysql",
    "records_migrated": 15000
  },
  "key_reminders": [
    "Backup database before each phase",
    "Validate data integrity after transformation"
  ],
  "created_at": "2025-12-09T16:00:00Z"
}
```

## Technical Implementation

### PreCompact Hook

**Handler**: `WorkflowStatePreCompactHandler`
**Location**: `.claude/hooks/controller/handlers/pre_compact/workflow_state_pre_compact_handler.py`
**Priority**: N/A (PreCompact doesn't use priorities)

**Behavior**:
1. Checks if workflow active (`_detect_workflow()`)
2. If NO: Returns allow, no file created
3. If YES: Extracts workflow state
4. Sanitizes workflow name for directory/filename
5. Looks for existing state file matching workflow name
6. If found: UPDATES existing file (preserves `created_at`)
7. If not found: CREATES new file with start_time timestamp
8. Always returns allow (never blocks compaction)

### SessionStart Hook

**Handler**: `WorkflowStateRestorationHandler`
**Location**: `.claude/hooks/controller/handlers/session_start/workflow_state_restoration_handler.py`
**Matcher**: `source: "compact"`

**Behavior**:
1. Finds all state files in `./untracked/workflow-state/*/`
2. If none: Returns allow, no guidance
3. If found: Sorts by modification time (most recent first)
4. Reads most recently modified state file
5. Builds guidance message with:
   - Workflow name and phase info
   - **REQUIRED READING** (@ syntax)
   - Key reminders
   - Context variables
   - ACTION REQUIRED section
6. **DOES NOT DELETE** state file
7. Returns allow with guidance context

### Tests

**Test Coverage**: 33 tests, all passing
- 12 PreCompact handler tests
- 14 SessionStart handler tests
- 7 Integration tests

**Test Scenarios**:
- Workflow detection (CLAUDE.local.md, active plans)
- State file creation/updates
- Directory structure and filenames
- @ syntax preservation
- Multiple compaction cycles
- No workflow active (defensive)
- Concurrent workflows

## Troubleshooting

### State file not created?

**Check**:
1. Is workflow formally documented?
2. Does CLAUDE.local.md contain workflow markers?
3. Is there an active plan with 🔄 In Progress status?
4. Check `.claude/hooks/controller/tests/` for test examples

### State file not updated after phase transition?

**Agent responsibility**: Update the file manually when moving between phases.

### Workflow lost after compaction?

**Check**:
1. Was state file created before compaction?
2. Check `./untracked/workflow-state/{workflow-name}/`
3. Did SessionStart hook provide guidance?
4. Did agent read @ prefixed files?

### Multiple state files accumulating?

**Normal if multiple workflows active**. Each workflow gets its own directory.

**Cleanup**: Delete state file when workflow completes.

## Related Documentation

- **Hooks Infrastructure**: `.claude/hooks/CLAUDE.md` - Hook system documentation
- **Plan Workflow**: `CLAUDE/PlanWorkflow.md` - Planning workflow documentation

---

**Document Version**: 1.0
**Last Updated**: December 2025
**Status**: Canonical / Production
