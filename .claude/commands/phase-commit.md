---
description: Create a properly formatted commit for current changes
argument-hint: [type] [scope] [message]
allowed-tools: Bash, Grep
---

## Context
- Git strategy: @docs/GIT-STRATEGY.md
- Current changes: !`git status --short`
- Current diff: !`git diff --stat`
- Recent commits: !`git log --oneline -5`
- Current branch: !`git branch --show-current`

## Your Task

Create a properly formatted git commit following Conventional Commits specification.

Arguments provided: $ARGUMENTS

1. **Analyze changes**
   - Review modified files
   - Determine appropriate type and scope
   - Ensure changes are related and atomic

2. **Validate commit format**
   - Type: feat, fix, docs, test, refactor, chore, style, perf
   - Scope: core, auth, db, api, ui, module-{name}, domain, config
   - Message: clear, concise description

3. **Pre-commit checks**
   - Run `/quick-test unit` if code changes
   - Verify no console.logs in code
   - Check for secrets or .env files

4. **Create commit**
   - Stage appropriate files
   - Format: `<type>(<scope>): <message>`
   - Ensure message is under 50 characters

If no arguments provided:
- Analyze changes and suggest appropriate commit message
- Ask for confirmation before committing

Examples:
- feat(core): implement module registry
- fix(auth): resolve session timeout issue
- test(domain): add E1RM calculator tests
- docs(api): update endpoint documentation

Output:
- Files staged
- Commit created with proper format
- Commit hash and message confirmation
- Next suggested action