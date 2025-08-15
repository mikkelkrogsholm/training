---
description: Start a new development phase from PLAN.md
argument-hint: [phase-number]
allowed-tools: Read, TodoWrite, Task
---

## Context
- Development plan: @docs/PLAN.md
- Available agents: @docs/agents.md
- Best practices: @docs/BEST-PRACTICES.md
- Claude guidance: @CLAUDE.md

## Your Task

Start working on Phase $ARGUMENTS from the PLAN.md file. Follow these steps:

1. **Read the phase details** from PLAN.md for Phase $ARGUMENTS
2. **Create a todo list** using TodoWrite with all tasks from that phase
3. **Identify the first task** and determine which agent should handle it
4. **Begin implementation** by either:
   - Using the appropriate agent via Task tool if complex
   - Implementing directly if straightforward

Important guidelines:
- Follow all **IMPORTANT** markers in CLAUDE.md
- Ensure TypeScript strict mode compliance
- Use mobile-first design (≤390px priority)
- Validate all inputs with Zod
- Keep business logic in $lib/domain as pure functions
- Write tests for new functionality

Output a brief summary of:
- Phase objectives
- Number of tasks to complete
- First task you'll work on
- Agent assignment strategy