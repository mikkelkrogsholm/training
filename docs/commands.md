# Custom Slash Commands for Fitness App Development

This document provides an overview of the custom slash commands available for developing the modular fitness web application. These commands streamline development workflow and ensure consistency with project standards.

## Recommended Command Workflow Order

### Phase Development Workflow (Primary Path)
1. **`/start-phase [n]`** - Begin working on a phase with todo list
2. **`/phase-branch [n]`** - Create and switch to feature branch
3. **`/quick-test`** - Verify baseline before starting work
4. **Development work** - Implement features
5. **`/phase-commit`** - Make conventional commits during work
6. **`/quick-test all`** - Test after each significant change
7. **`/review-phase [n]`** - Check completion status
8. **`/phase-complete [n]`** - Merge to main and tag
9. **`/git-status`** - Verify clean state before next phase

### Module Development Workflow
1. **`/add-module [name]`** - Scaffold new module structure
2. **`/db-status`** - Verify database after migrations
3. **`/quick-test all`** - Ensure module integration works

### Daily Development Checkpoints
- **Morning**: `/git-status` → `/quick-test` → Continue phase work
- **Before commits**: `/quick-test unit` → `/phase-commit`
- **After features**: `/quick-test all` → `/review-phase`
- **End of day**: `/git-status` → Document progress

## Available Commands

### `/start-phase [phase-number]`

**Purpose:** Begin working on a specific development phase from the PLAN.md

**Usage:**
```bash
/start-phase 0   # Start Phase 0: Project Setup & Foundation
/start-phase 1   # Start Phase 1: Kernel Core - Database & Auth
/start-phase 7   # Start Phase 7: Calendar Module
```

**What it does:**
- Reads the specified phase details from PLAN.md
- Creates a comprehensive todo list with all phase tasks
- Identifies the appropriate agent for each task
- Begins implementation of the first task
- Ensures compliance with CLAUDE.md best practices

**Output:** Summary of phase objectives, task count, and agent assignments

---

### `/review-phase [phase-number]`

**Purpose:** Review the completion status of a development phase

**Usage:**
```bash
/review-phase 0   # Review Phase 0 completion
/review-phase 4   # Review Phase 4 (Routines & Session Logging)
```

**What it does:**
- Checks all tasks from the specified phase
- Verifies code quality and TypeScript strict mode
- Reviews test coverage for phase features
- Validates best practices compliance (Zod validation, DTO mapping, security)
- Checks module system integrity if applicable

**Output:** 
- ✅ Completed tasks
- ⚠️ Partially completed tasks  
- ❌ Missing tasks
- 📝 Recommendations for completion
- 🎯 Next phase readiness assessment

---

### `/quick-test [unit|e2e|all]`

**Purpose:** Run tests and quality checks for the current code

**Usage:**
```bash
/quick-test         # Run all tests (default)
/quick-test unit    # Run only unit tests
/quick-test e2e     # Run only E2E tests
/quick-test all     # Explicitly run all tests
```

**Test Types:**
- **unit:** Runs Vitest unit tests, checks domain logic coverage
- **e2e:** Runs Playwright E2E tests, focuses on critical user flows
- **all:** Runs linter, type checking, unit tests, and E2E tests

**Additional Checks:**
- E1RM calculation verification
- Session logging flow validation
- Module isolation checks
- Security best practices audit

**Output:**
- ✅ Passing tests
- ❌ Failing tests with details
- ⚠️ Warnings or issues
- 📊 Coverage summary
- 🔧 Required fixes

---

### `/add-module [module-name]`

**Purpose:** Scaffold a new module with proper structure and isolation

**Usage:**
```bash
/add-module calendar        # Create calendar module
/add-module analytics       # Create analytics module
/add-module ai-trainer      # Create AI trainer module
```

**What it does:**
- Creates module directory under `/src/modules/[module-name]/`
- Generates valid module.json manifest with Zod validation
- Sets up routes, widgets, db, and jobs directories
- Configures UI slot registrations
- Ensures module isolation (no cross-module imports)
- Registers extension points and event subscriptions

**Integration Points:**
- **UI Slots:** `today.tiles`, `movement.detail.tabs`, `routine.header.actions`, `profile.panels`
- **Events:** `onSessionLogged`, `onRoutinePublished`, `onUserCreated`, `onMediaUploaded`

**Output:** Module structure overview, created files, and next implementation steps

---

### `/db-status`

**Purpose:** Perform comprehensive database health and migration status check

**Usage:**
```bash
/db-status   # Check database health
```

**What it checks:**

1. **Database Status:**
   - SQLite existence and location
   - WAL mode verification
   - Database size and modification time
   - Table listing

2. **Schema Review:**
   - Core tables (users, settings, _migrations)
   - Feature tables based on completed phases
   - Index verification on key columns

3. **Migration Status:**
   - Applied migrations from _migrations table
   - Pending migrations
   - Module migrations
   - File naming consistency

4. **Data Integrity:**
   - Canonical units (kg, meters, seconds)
   - Foreign key relationships
   - Password hash verification
   - Orphaned records

5. **Performance:**
   - Table sizes and row counts
   - Index usage
   - N+1 query risks

**Output:**
- ✅ Healthy aspects
- ⚠️ Optimization opportunities
- ❌ Critical issues
- 📊 Database statistics
- 🔧 Recommended actions

---

## Workflow Examples

### Starting a New Phase
```bash
# Begin Phase 0 (Project Setup)
/start-phase 0

# After completing tasks, review progress
/review-phase 0

# Run tests to ensure everything works
/quick-test all

# If all good, move to next phase
/start-phase 1
```

### Adding a Module
```bash
# After completing Phase 6 (Admin Module), add Calendar
/add-module calendar

# Check database after module adds migrations
/db-status

# Test the new module
/quick-test all
```

### Daily Development Flow
```bash
# Check current phase status
/review-phase 3

# Run quick tests before starting
/quick-test unit

# Continue with phase tasks
/start-phase 3

# After implementing features, test again
/quick-test all

# Check database health after schema changes
/db-status
```

## Command Location

All custom commands are stored in `.claude/commands/` directory:

```
.claude/
└── commands/
    ├── start-phase.md
    ├── review-phase.md
    ├── quick-test.md
    ├── add-module.md
    └── db-status.md
```

## Best Practices

1. **Always start with `/start-phase`** when beginning a new development phase
2. **Use `/review-phase`** before moving to the next phase to ensure completion
3. **Run `/quick-test`** frequently during development to catch issues early
4. **Check `/db-status`** after schema changes or migrations
5. **Use `/add-module`** for consistent module scaffolding

## Integration with Agents

These commands work seamlessly with the specialized agents:

- `start-phase` coordinates with all agents based on task requirements
- `add-module` specifically uses the **module-builder** agent
- `review-phase` may trigger **testing-agent** for verification
- `db-status` relates to **db-schema-manager** work
- `quick-test` validates work from all agents

## Tips

- Commands support arguments for flexibility
- Commands reference project documentation automatically
- Each command follows the project's strict TypeScript and best practices
- Commands include bash execution for real-time status checks
- All commands respect the modular architecture principles

---

## Git Workflow Commands

### `/phase-branch [phase-number]`

**Purpose:** Create and switch to a phase development branch

**Usage:**
```bash
/phase-branch 0   # Create branch for Phase 0
/phase-branch 7   # Create branch for Phase 7 (Calendar Module)
```

**What it does:**
- Checks working directory is clean
- Creates branch following naming convention: `phase-{number}-{description}`
- Switches to the new branch
- Shows phase objectives and first task

**Output:** Branch creation confirmation, phase goals, git workflow reminders

---

### `/phase-commit [type] [scope] [message]`

**Purpose:** Create a properly formatted conventional commit

**Usage:**
```bash
/phase-commit feat core "implement module registry"
/phase-commit fix auth "resolve session timeout"
/phase-commit test domain "add E1RM tests"
/phase-commit   # Analyzes changes and suggests commit
```

**What it does:**
- Validates commit format (type, scope, message)
- Runs pre-commit checks (tests, console.logs, secrets)
- Stages appropriate files
- Creates conventional commit

**Commit Types:** feat, fix, docs, test, refactor, chore, style, perf
**Scopes:** core, auth, db, api, ui, module-{name}, domain, config

---

### `/phase-complete [phase-number]`

**Purpose:** Complete a phase by merging to main and tagging

**Usage:**
```bash
/phase-complete 0   # Complete Phase 0
/phase-complete 4   # Complete Phase 4
```

**What it does:**
- Verifies all phase tasks completed
- Runs full test suite
- Merges to main with `--no-ff`
- Creates tag: `phase-{number}-complete`
- Pushes to origin with tags
- Cleans up feature branch

**Output:** Merge confirmation, tag details, next phase suggestion

---

### `/git-status`

**Purpose:** Comprehensive git repository status overview

**Usage:**
```bash
/git-status   # Full repository status check
```

**What it shows:**
- Current branch and tracking status
- Uncommitted changes
- Unpushed commits
- Branch overview (merged/unmerged)
- Commit history and format compliance
- Phase completion tags
- Repository health checks

**Output Format:**
```
📍 Current Branch: phase-1-auth-system
📝 Working Directory: clean
📤 Unpushed Commits: 3
🏷️ Latest Phase Tag: phase-0-complete
✅ Conventional Commits: yes
```

## Complete Phase Development Example

Following the recommended workflow order:

```bash
# 1. Start new phase - creates todo list
/start-phase 1          

# 2. Create feature branch
/phase-branch 1         

# 3. Baseline test
/quick-test            

# 4-5. Development with commits
# ... implement features ...
/phase-commit feat auth "implement login endpoint"
/quick-test unit        
/phase-commit test auth "add login tests"

# 6. Full test after features
/quick-test all        

# 7. Review completion
/review-phase 1         

# 8. Complete and merge
/phase-complete 1       

# 9. Verify clean state
/git-status            

# Ready for next phase
/start-phase 2         
```

## Quick Reference Card

### Essential Commands by Frequency
1. **Most Used (multiple times daily)**
   - `/quick-test` - Before and after changes
   - `/phase-commit` - For each logical change
   
2. **Phase Transitions (per phase)**
   - `/start-phase` - Begin phase
   - `/phase-branch` - Create branch
   - `/review-phase` - Check progress
   - `/phase-complete` - Finish phase
   
3. **Status Checks (as needed)**
   - `/git-status` - Repository overview
   - `/db-status` - Database health
   
4. **Module Work (when adding features)**
   - `/add-module` - New module scaffold

## See Also

- [Git Strategy](./GIT-STRATEGY.md) - Complete git workflow documentation
- [Development Plan](./PLAN.md) - Detailed phase breakdown
- [Available Agents](./agents.md) - Agent descriptions and usage
- [Best Practices](./BEST-PRACTICES.md) - Coding standards
- [Project Requirements](./PRD.md) - Complete project specification