# Git Strategy for Modular Fitness App

This document defines the git workflow and version control practices for the project. All development must follow these guidelines to maintain a clean, traceable history.

## Branch Strategy

### Main Branches

- **`main`**: Production-ready code, always deployable
- **`develop`**: Integration branch for features (created when needed)

### Feature Branches

**Naming Convention:**
```
phase-{number}-{short-description}
module-{name}
fix-{issue-description}
docs-{description}
```

**Examples:**
- `phase-0-project-setup`
- `phase-1-auth-system`
- `module-calendar`
- `fix-e1rm-calculation`
- `docs-api-endpoints`

## Commit Strategy

### Commit Message Format

Follow Conventional Commits specification:

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code style changes (formatting, semicolons, etc.)
- **refactor**: Code refactoring without changing functionality
- **test**: Adding or updating tests
- **chore**: Maintenance tasks (build, dependencies, etc.)
- **perf**: Performance improvements
- **ci**: CI/CD changes

### Scopes (aligned with architecture)

- **core**: Kernel functionality
- **auth**: Authentication system
- **db**: Database and migrations
- **api**: API endpoints
- **ui**: UI components
- **module-{name}**: Specific module
- **domain**: Business logic
- **config**: Configuration files

### Examples

```bash
feat(core): implement module registry system
fix(auth): resolve session timeout issue
docs(api): add endpoint documentation
test(domain): add E1RM calculator tests
feat(module-calendar): add monthly view component
```

## Workflow Rules

### 1. Phase-Based Development

Each phase from PLAN.md gets its own branch:

```bash
# Start a new phase
git checkout -b phase-0-project-setup

# Work on the phase...
git add .
git commit -m "feat(config): initialize SvelteKit project with TypeScript"

# Complete the phase
git checkout main
git merge --no-ff phase-0-project-setup
git tag -a "phase-0-complete" -m "Phase 0: Project Setup completed"
git push origin main --tags
```

### 2. Module Development

Each module gets its own branch:

```bash
# Create module branch
git checkout -b module-calendar

# Scaffold module
git commit -m "feat(module-calendar): scaffold calendar module structure"

# Implement features
git commit -m "feat(module-calendar): add month view component"
git commit -m "feat(module-calendar): implement plan materialization"

# Merge when complete
git checkout main
git merge --no-ff module-calendar
git tag -a "module-calendar-v0.1.0" -m "Calendar module v0.1.0"
```

### 3. Commit Frequency

- **Atomic commits**: Each commit should be a single logical change
- **Commit per task**: Generally one commit per task from PLAN.md
- **Test before commit**: Run `/quick-test unit` before committing
- **Working code only**: Never commit broken code to main

### 4. Merge Strategy

- **No fast-forward**: Use `--no-ff` for feature branches to maintain history
- **Squash optional**: Squash only for cleanup commits (typos, formatting)
- **Rebase carefully**: Only rebase local branches, never shared branches

## Tags and Releases

### Tag Format

```
phase-{number}-complete     # Phase completion
module-{name}-v{version}    # Module releases
v{version}                  # Overall app releases
```

### Version Numbering

Follow Semantic Versioning:
- **MAJOR**: Breaking changes
- **MINOR**: New features (backwards compatible)
- **PATCH**: Bug fixes

**Examples:**
- `v0.1.0` - Initial MVP
- `v0.2.0` - Calendar module added
- `v1.0.0` - Production release

## Pull Request Guidelines

When working with others:

1. **PR Title**: Use conventional commit format
2. **PR Description**: 
   - Reference PLAN.md phase
   - List completed tasks
   - Include test results
   - Note any deviations from plan

3. **PR Template**:
```markdown
## Phase/Module
Phase X: [Name] / Module: [Name]

## Completed Tasks
- [ ] Task 1 from PLAN.md
- [ ] Task 2 from PLAN.md

## Testing
- [ ] Unit tests pass
- [ ] E2E tests pass (if applicable)
- [ ] Linting passes
- [ ] Type checking passes

## Checklist
- [ ] Follows BEST-PRACTICES.md
- [ ] Updates documentation if needed
- [ ] No console.logs or debug code
- [ ] Canonical units (kg, meters, seconds)
- [ ] Mobile-first design (≤390px)
```

## Git Hooks (Optional)

### Pre-commit Hook

```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run tests
npm run lint
npm run check

# Check for console.logs
if grep -r "console.log" --include="*.ts" --include="*.svelte" src/; then
  echo "Error: console.log found in code"
  exit 1
fi
```

### Commit Message Hook

```bash
#!/bin/sh
# .git/hooks/commit-msg

# Verify conventional commit format
commit_regex='^(feat|fix|docs|style|refactor|test|chore|perf|ci)(\([a-z0-9-]+\))?: .{1,50}'

if ! grep -qE "$commit_regex" "$1"; then
  echo "Invalid commit message format!"
  echo "Format: <type>(<scope>): <subject>"
  exit 1
fi
```

## Common Scenarios

### Starting Work on a Phase

```bash
/start-phase 1                                    # Use slash command
git checkout -b phase-1-auth-system               # Create branch
# ... work on tasks ...
git add -A                                         # Stage changes
git commit -m "feat(auth): implement login API"   # Commit with message
/quick-test all                                    # Test everything
```

### Completing a Phase

```bash
/review-phase 1                                    # Review completion
git log --oneline                                  # Review commits
git checkout main                                   # Switch to main
git merge --no-ff phase-1-auth-system             # Merge branch
git tag -a "phase-1-complete" -m "Auth complete"  # Tag completion
git push origin main --tags                        # Push with tags
```

### Quick Fix

```bash
git checkout -b fix-login-rate-limit              # Create fix branch
# ... fix the issue ...
git commit -m "fix(auth): add rate limiting"      # Commit fix
git checkout main                                  # Back to main
git merge --no-ff fix-login-rate-limit           # Merge fix
git push origin main                              # Push fix
```

### Adding a Module

```bash
/add-module analytics                             # Scaffold module
git checkout -b module-analytics                  # Create branch
git add -A                                         # Stage scaffolding
git commit -m "feat(module-analytics): scaffold"  # Commit
# ... implement module ...
git commit -m "feat(module-analytics): add charts"
```

## Important Rules

1. **NEVER force push** to main
2. **ALWAYS test** before committing (`/quick-test`)
3. **ALWAYS use conventional commits** format
4. **NEVER commit secrets** or .env files
5. **ALWAYS update docs** when changing APIs
6. **TAG phase completions** for easy rollback
7. **KEEP commits atomic** and logical
8. **REVIEW changes** with `git diff` before committing

## Integration with Development Workflow

1. Each phase in PLAN.md corresponds to a branch
2. Each completed phase gets tagged
3. Modules are developed in separate branches
4. All commits follow conventional format
5. Use slash commands to verify before merging

## Git Aliases (Recommended)

Add to your `.gitconfig`:

```ini
[alias]
  phase = checkout -b
  complete = merge --no-ff
  lastphase = describe --tags --abbrev=0
  phaselog = log --oneline --graph --decorate
```

## See Also

- [PLAN.md](./PLAN.md) - Development phases
- [BEST-PRACTICES.md](./BEST-PRACTICES.md) - Coding standards
- [commands.md](./commands.md) - Slash commands for workflow