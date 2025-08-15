---
description: Review completion status of a development phase
argument-hint: [phase-number]
allowed-tools: Read, Bash, Grep, TodoWrite
---

## Context
- Development plan: @docs/PLAN.md
- Project structure: !`find . -type f -name "*.ts" -o -name "*.svelte" -o -name "*.json" | head -20`
- Current git status: !`git status --short`
- Package.json (if exists): !`[ -f package.json ] && cat package.json | head -30`

## Your Task

Review the completion status of Phase $ARGUMENTS. Perform these checks:

1. **Phase Requirements**
   - Read Phase $ARGUMENTS tasks from PLAN.md
   - Check which tasks are marked as completed
   - Identify any remaining tasks

2. **Code Quality Checks**
   - Verify TypeScript strict mode is enabled
   - Check for proper path aliases usage
   - Ensure Zod validation is in place
   - Verify mobile-first CSS (if UI phase)

3. **Testing Coverage**
   - Check if tests exist for the phase features
   - Verify domain logic has unit tests
   - Check for E2E tests if applicable

4. **Best Practices Compliance**
   - Verify canonical units storage (kg, meters, seconds)
   - Check for proper DTO mapping
   - Ensure no raw DB shapes exposed
   - Verify security practices (bcrypt, HTTP-only cookies)

5. **Module System (if applicable)**
   - Check module isolation (no cross-module imports)
   - Verify module.json if module was created
   - Check slot/event registrations

Provide a status report with:
- ✅ Completed tasks
- ⚠️ Partially completed tasks
- ❌ Missing tasks
- 📝 Recommendations for completion
- 🎯 Next phase readiness assessment