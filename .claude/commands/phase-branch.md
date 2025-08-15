---
description: Create and switch to a phase branch
argument-hint: [phase-number]
allowed-tools: Bash, Read
---

## Context
- Development plan: @docs/PLAN.md
- Git strategy: @docs/GIT-STRATEGY.md
- Current branch: !`git branch --show-current`
- Git status: !`git status --short`

## Your Task

Create and switch to the appropriate git branch for Phase $ARGUMENTS.

1. **Check current status**
   - Ensure working directory is clean
   - Verify we're on main branch or appropriate base

2. **Read phase details** from PLAN.md
   - Get the phase name and branch name

3. **Create the phase branch**
   - Follow naming convention: `phase-{number}-{description}`
   - Switch to the new branch

4. **Set up for development**
   - Confirm branch creation
   - Show the phase goals and tasks

If there are uncommitted changes, provide guidance on:
- Stashing changes
- Committing work in progress
- Or cleaning the working directory

Output:
- Branch created and switched
- Phase objectives reminder
- First task to work on
- Git workflow reminders (conventional commits, testing before commit)