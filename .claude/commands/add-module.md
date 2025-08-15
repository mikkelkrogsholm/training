---
description: Scaffold a new module with proper structure
argument-hint: [module-name]
allowed-tools: Task
model: claude-3-5-sonnet-20241022
---

## Context
- Module system docs: @docs/PRD.md#4-module-system
- Available agents: @docs/agents.md
- Best practices: @docs/BEST-PRACTICES.md

## Your Task

Create a new module called "$ARGUMENTS" using the module-builder agent.

The module should follow these requirements:

1. **Module Structure**:
   - Create under `/src/modules/$ARGUMENTS/`
   - Include proper module.json manifest
   - Setup routes, widgets, db, and jobs directories
   - Implement Zod validation for manifest

2. **Module Manifest Requirements**:
   - Unique module ID
   - Version following semver
   - Proper dependencies (core version)
   - UI slot registrations if needed
   - Event subscriptions if applicable
   - Database migrations if required

3. **Module Isolation**:
   - No cross-module imports
   - Only import from core
   - Use events for inter-module communication
   - Register in extension points properly

4. **Integration Points**:
   - Available slots: `today.tiles`, `movement.detail.tabs`, `routine.header.actions`, `profile.panels`
   - Events: `onSessionLogged`, `onRoutinePublished`, `onUserCreated`, `onMediaUploaded`

Use the Task tool to invoke the module-builder agent with these specifications.

After creation, provide:
- Module structure overview
- Key files created
- Integration points configured
- Next steps for implementation