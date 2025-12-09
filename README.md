# Claude Project Skeleton

A standard project skeleton for Claude Code projects, providing:

- **Hooks System**: Front controller pattern with pre-built safety and workflow handlers
- **Agents**: Specialised agents for common tasks
- **Workflow Documentation**: Plan management and workflow state persistence

## Quick Start

Copy the `.claude/` directory and `CLAUDE/` directory into your project root.

```bash
# Clone this skeleton
git clone https://github.com/LongTermSupport/claude-project-skeleton.git

# Copy to your project
cp -r claude-project-skeleton/.claude /path/to/your/project/
cp -r claude-project-skeleton/CLAUDE /path/to/your/project/
```

## Directory Structure

```
.claude/
├── hooks/                    # Claude Code hooks system
│   ├── CLAUDE.md            # Hooks documentation
│   ├── controller/          # Front controller implementation
│   │   ├── front_controller.py
│   │   ├── pre_tool_use.py  # PreToolUse entry point
│   │   ├── handlers/        # Handler implementations
│   │   └── tests/           # Test suite
│   └── [entry point scripts]
└── agents/
    └── hooks-specialist.md  # Agent for creating new hooks

CLAUDE/
├── PlanWorkflow.md          # Planning workflow documentation
├── Workflow.md              # Workflow state management
├── Worktree.md              # Git worktree workflow
└── Plan/                    # Plan documents directory
```

## Included Handlers

### Safety Handlers (Priority 10-20)
- **DestructiveGitHandler** - Blocks `git reset --hard`, `git clean -f`, etc.
- **SedBlockerHandler** - Blocks sed usage (use Edit tool instead)
- **AbsolutePathHandler** - Validates file paths
- **TddEnforcementHandler** - Enforces TDD for hook development
- **WorktreeFileCopyHandler** - Prevents copying between worktrees
- **GitStashHandler** - Discourages git stash usage

### Workflow Handlers (Priority 25-60)
- **OfficialPlanCommandHandler** - Enforces plan command structure
- **EslintDisableHandler** - Blocks eslint-disable comments (for JS/TS projects)
- **ValidatePlanNumberHandler** - Validates plan numbering
- **MarkdownOrganizationHandler** - Enforces markdown structure
- **PlanTimeEstimatesHandler** - Blocks time estimates in plans
- **PlanWorkflowHandler** - Provides plan creation guidance
- **WebSearchYearHandler** - Ensures correct year in web searches
- **BritishEnglishHandler** - Warns about American spellings

### Architecture Enforcement
- **EnforceControllerPatternHandler** - Ensures all hooks use front controller

## Creating New Handlers

Use the hooks-specialist agent or follow the TDD workflow:

1. Write tests in `.claude/hooks/controller/tests/`
2. Create handler in `.claude/hooks/controller/handlers/pre_tool_use/`
3. Register in `__init__.py` and entry point
4. Run tests: `python3 .claude/hooks/controller/run_tests.py`

See `.claude/hooks/CLAUDE.md` for full documentation.

## Workflow Documentation

### Plan Workflow
Plans are structured documents for tracking multi-step work. See `CLAUDE/PlanWorkflow.md`.

### Workflow State Persistence
Workflow state survives conversation compaction. See `CLAUDE/Workflow.md`.

### Git Worktree Workflow
For parallel development work. See `CLAUDE/Worktree.md`.

## Customisation

### Removing Handlers
Edit `.claude/hooks/controller/handlers/pre_tool_use/__init__.py` and `.claude/hooks/controller/pre_tool_use.py` to remove unwanted handlers.

### Adding Project-Specific Handlers
Follow the TDD workflow with the hooks-specialist agent.

### British vs American English
The BritishEnglishHandler warns about American spellings. Remove it if not needed.

## Requirements

- Python 3.8+
- Claude Code CLI

## License

MIT
