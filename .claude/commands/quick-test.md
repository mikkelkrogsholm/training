---
description: Run tests and checks for the current code
argument-hint: [unit|e2e|all]
allowed-tools: Bash, Read
---

## Context
- Test type requested: $ARGUMENTS (defaults to 'all' if not specified)
- Project test setup: !`[ -f package.json ] && grep -E '"(test|vitest|playwright)"' package.json || echo "No test config found"`
- TypeScript config: !`[ -f tsconfig.json ] && grep '"strict"' tsconfig.json || echo "No tsconfig found"`

## Your Task

Run appropriate tests based on the argument provided ($ARGUMENTS):

### If argument is 'unit' or empty:
1. Run unit tests with Vitest: `npm test`
2. Check domain logic test coverage
3. Verify Zod validators are tested

### If argument is 'e2e':
1. Run Playwright E2E tests: `npm run test:e2e`
2. Focus on critical user flows
3. Check mobile viewport tests

### If argument is 'all' or no argument:
1. Run linter: `npm run lint`
2. Run type checking: `npm run check`
3. Run unit tests: `npm test`
4. Run E2E tests if available: `npm run test:e2e`
5. Check for uncommitted changes

### Additional Checks:
- Verify E1RM calculations if present
- Check session logging flow if implemented
- Validate module isolation if modules exist
- Ensure security best practices

Report results with:
- ✅ Passing tests
- ❌ Failing tests with details
- ⚠️ Warnings or issues
- 📊 Coverage summary if available
- 🔧 Fixes needed