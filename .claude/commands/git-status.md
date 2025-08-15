---
description: Comprehensive git repository status
allowed-tools: Bash
---

## Context
- Git strategy: @docs/GIT-STRATEGY.md
- Current branch: !`git branch --show-current`

## Your Task

Provide a comprehensive overview of the git repository status.

1. **Current State**
   - Current branch and its tracking status
   - Uncommitted changes (staged and unstaged)
   - Unpushed commits
   - Stashed changes count

2. **Branch Overview**
   - List all local branches
   - Show merged and unmerged branches
   - Identify stale branches

3. **Commit History**
   - Last 10 commits with graph
   - Check commit message format compliance
   - Identify any fixup or WIP commits

4. **Tags and Phases**
   - List all phase completion tags
   - Show latest tag
   - Identify current phase from branch name

5. **Remote Status**
   - Check connection to origin
   - Compare local vs remote
   - Show any divergence

6. **Health Checks**
   - Check for large files
   - Verify .gitignore is working
   - Check for accidentally committed secrets
   - Verify no console.logs in staged files

Format output as:
```
📍 Current Branch: [branch-name]
📝 Working Directory: [clean/dirty]
📤 Unpushed Commits: [count]
🏷️ Latest Phase Tag: [tag]
✅ Conventional Commits: [yes/no]
⚠️ Issues Found: [list if any]

Recommendations:
- [Next actions based on status]
```

Provide guidance for:
- Cleaning up branches
- Resolving uncommitted changes
- Following git workflow
- Next development steps