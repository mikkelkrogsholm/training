---
description: Complete current phase and merge to main
argument-hint: [phase-number]
allowed-tools: Bash, Read, TodoWrite
---

## Context
- Current branch: !`git branch --show-current`
- Git status: !`git status`
- Unpushed commits: !`git log origin/main..HEAD --oneline`
- Development plan: @docs/PLAN.md
- Phase number: $ARGUMENTS

## Your Task

Complete Phase $ARGUMENTS by merging to main and tagging the completion.

1. **Verify phase completion**
   - Run `/review-phase $ARGUMENTS` to check all tasks
   - Ensure all tests pass with `/quick-test all`
   - Verify no uncommitted changes

2. **Review commits**
   - Check all commits follow conventional format
   - Ensure atomic, logical commits
   - Verify no debug code or console.logs

3. **Merge to main**
   - Switch to main branch
   - Pull latest changes (if any)
   - Merge with --no-ff to preserve history
   - Create descriptive merge commit

4. **Tag the completion**
   - Create tag: `phase-{number}-complete`
   - Add descriptive tag message
   - Include completed features summary

5. **Push to origin**
   - Push main branch
   - Push tags
   - Confirm successful push

6. **Clean up**
   - Delete local feature branch
   - Update todo list to mark phase complete
   - Suggest next phase to work on

Post-completion checklist:
- ✅ All phase tasks completed
- ✅ Tests passing
- ✅ Code follows best practices
- ✅ Documentation updated
- ✅ Merged to main
- ✅ Tagged appropriately
- ✅ Pushed to origin

Output:
- Merge confirmation
- Tag created
- Push successful
- Next phase recommendation