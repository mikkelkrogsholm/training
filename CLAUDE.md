# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a modular fitness web application ("kernel + modules") designed for private use by a small group of family and friends. The application follows a unique architecture where a kernel provides core functionality and modules can be dynamically enabled/disabled at runtime.

## Tech Stack & Architecture

- **Framework**: SvelteKit with adapter-node
- **Database**: SQLite with Drizzle ORM
- **Styling**: Tailwind CSS
- **Deployment**: Single container via Dokploy
- **Data Visualization**: Chart.js or ApexCharts (mobile-friendly)

## Critical Best Practices

**IMPORTANT: These practices MUST be followed in all code:**

### TypeScript & Code Quality
- **IMPORTANT**: Use TypeScript strict mode with `"strict": true`, `"noImplicitAny": true`, `"noUncheckedIndexedAccess": true`
- **IMPORTANT**: Run ESLint and Prettier before committing - code must pass `npm run lint` and `npm run format:check`
- **IMPORTANT**: Keep files small (≤200-300 lines for UI components) - extract helpers early

### Architecture Rules
- **IMPORTANT**: Modules depend on core ONLY - NEVER import across modules
- **IMPORTANT**: Keep business logic in `$lib/domain/*` as pure functions, NOT in components or endpoints
- **IMPORTANT**: Use path aliases (`$lib/*`, `$core/*`, `$modules/*`) - avoid relative imports

### Data & Validation
- **IMPORTANT**: ALL inputs must be validated with Zod - API inputs, routine JSON, module manifests, uploads
- **IMPORTANT**: Store canonical units (kg, meters, seconds) - convert only for display
- **IMPORTANT**: Use WAL mode for SQLite; create indices for `user_id`, `movement_id`, `date` up front
- **IMPORTANT**: Never leak raw DB shapes to UI - map to typed DTOs

### Security & Privacy
- **IMPORTANT**: NEVER store secrets in code or commits
- **IMPORTANT**: Validate file uploads with MIME sniff + extension check + size limits
- **IMPORTANT**: Privacy flags must be enforced on server queries, not just UI
- **IMPORTANT**: Use bcrypt for passwords with constant-time comparisons
- **IMPORTANT**: Sessions must be HTTP-only, Secure, SameSite=Lax cookies

### API & Error Handling
- **IMPORTANT**: Thin endpoints pattern - parse/validate (Zod) → call domain service → map response
- **IMPORTANT**: NEVER leak stack traces - map to user-friendly error messages
- **IMPORTANT**: Rate limit auth endpoints (login, password reset, AI endpoints)

## Key Commands

### Development
```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build

# Run production build
npm run preview

# Run tests
npm test

# Run linter
npm run lint

# Run type checking
npm run check
```

### Database Management
```bash
# Run migrations
npm run db:migrate

# Generate migration
npm run db:generate

# Push schema changes
npm run db:push

# Seed database
npm run db:seed
```

## Architecture & Module System

### Core Structure
The application uses a kernel + modules architecture where:
- **Kernel** (`/src/core`): Authentication, database, settings, events, slot rendering
- **Modules** (`/src/modules/*`): Self-contained features with own routes, widgets, migrations, and jobs

### Module Manifest
Each module must include a `module.json` with:
- Routes, widgets, permissions, settings
- Database migrations
- Scheduled jobs
- Dependencies on other modules

### Extension Points
- **UI Slots**: `today.tiles`, `movement.detail.tabs`, `routine.header.actions`, `profile.panels`
- **Events**: `onSessionLogged`, `onRoutinePublished`, `onUserCreated`, `onMediaUploaded`
- **Jobs**: Cron-scheduled tasks using node-cron

## Database Schema

Key entities and their relationships:
- **users**: Core user accounts with preferences
- **movements**: Exercise definitions with units
- **routines**: Training programs with JSON schema (versioned)
- **sessions**: Logged workouts
- **session_movements**: Individual exercise data within sessions
- **plans**: Materialized calendar entries
- **friends**: Social connections

### E1RM Calculation
Use Epley formula: `E1RM = weight × (1 + reps/30)`

## Development Guidelines

### File Organization
```
/src
  /core         # Kernel components
  /modules      # Feature modules
    /[module]
      module.json
      /routes   # SvelteKit routes
      /widgets  # UI components
      /db       # Migrations
      /jobs     # Scheduled tasks
  /lib          # Shared utilities
    /domain     # Business logic (pure functions)
```

### Module Development
1. Create module folder under `/src/modules`
2. Add `module.json` manifest with Zod validation
3. Implement routes as SvelteKit pages
4. Add widgets for slot injection
5. Include migrations in `/db` subfolder
6. Register cron jobs if needed

### API Patterns
- Auth endpoints: `/api/auth/*`
- Resource endpoints: `/api/[resource]/*`
- Module endpoints: `/api/modules/[module]/*`
- Admin endpoints: `/api/admin/*`

### Testing Requirements
- **IMPORTANT**: Unit test ALL domain logic (E1RM/volume calculators, schedule materialization)
- **IMPORTANT**: Test all Zod validators for routine JSON and module manifests
- **IMPORTANT**: E2E test core flows: login → start session → log set → see chart update
- Use deterministic seed data for repeatable tests

### UI/UX Requirements
- **IMPORTANT**: Mobile-first design - build for ≤390px first
- **IMPORTANT**: Accessibility - semantic HTML, keyboard navigation, visible focus rings
- **IMPORTANT**: One-handed logging - large buttons, swipe-to-complete
- Respect `prefers-reduced-motion`
- Include captions for videos

### Performance Guidelines
- **IMPORTANT**: Avoid N+1 queries - batch reads with IN queries
- Virtualize long lists if needed
- Debounce typeahead searches
- Lazy-load heavy charts and module routes
- Use optimistic UI for set logging with rollback on failure

### Code Review Checklist
Before any PR:
- Does code touch validation, auth, or privacy? → Fully tested?
- New DB queries? → Indices added? Complexity explained?
- UI accessible? → Keyboard tested? Mobile tested?
- Errors handled? → User-friendly messages? No PII in logs?
- Tests included? → Unit tests for logic? Integration test for new paths?

## Important Constraints

1. **Single Container**: Everything runs in one container - no microservices
2. **SQLite Only**: No PostgreSQL/MySQL - use SQLite with proper WAL mode
3. **Module Isolation**: Modules should be self-contained and not directly depend on other modules
4. **Mobile First**: All UI must be touch-friendly with large tap targets
5. **Privacy First**: Default to minimal sharing, explicit opt-in for social features
6. **Immutable Image**: Only `/data` is mutable volume - never write elsewhere
7. **Feature Flags**: Typed getters, DB storage with env override for safety

## Common Tasks

### Adding a New Module
1. Create module directory structure
2. Define module.json manifest
3. Implement initial migration
4. Add routes and widgets
5. Register in ModuleRegistry
6. Test module loading/unloading

### Creating a Routine
- **IMPORTANT**: Validate against JSON schema with Zod (currently v1.0.0)
- Include `schema_version` field for migrations
- Include movement references and guidance
- Support blocks: movement, circuit, mobility
- Define progression rules

### Implementing Analytics
- **IMPORTANT**: Calculate E1RM server-side using Epley formula: `weight × (1 + reps/30)`
- Track volume as Σ(weight × reps)
- Support relative strength (E1RM/bodyweight)
- Provide time-series charts with Chart.js/ApexCharts
- Downsample for long time ranges to keep mobile smooth

### AI Integration Guidelines
- **IMPORTANT**: Retrieval-first - ground answers in guides/resources with citations
- **IMPORTANT**: AI outputs must return JSON only - validate with Zod, retry if invalid
- **IMPORTANT**: Safety guardrails - red-flag medical terms trigger conservative advice
- Never log sensitive prompt content - scrub PII
- Use circuit breakers with timeouts and fallbacks

## Environment Variables
```
DATABASE_URL=file:/data/db.sqlite
MEDIA_DIR=/data/media
BACKUP_DIR=/data/backups
SESSION_SECRET=[required]
OPENAI_API_KEY=[optional for AI modules]
BASE_URL=https://your-domain.tld
```

## Deployment Notes
- Build with Node 20 Alpine
- Volume mount at `/data` for persistence
- SQLite database at `/data/db.sqlite`
- Media storage at `/data/media`
- Backups at `/data/backups`
- Exposed on port 3000
- **IMPORTANT**: Implement health check at `/api/health` that verifies DB R/W
- **IMPORTANT**: Fail fast if required env vars are missing
- Nightly SQLite backups with `.backup` command
- Keep previous image tagged for rollback capability

## Security Headers & CSP
**IMPORTANT: Configure these headers in production:**
- Content Security Policy: `default-src 'self'`
- `X-Content-Type-Options: nosniff`
- `Referrer-Policy: same-origin`
- `Permissions-Policy` with minimal permissions
- Force HTTPS with `upgrade-insecure-requests`

## Dependency Management
- **IMPORTANT**: Commit lockfile and run `npm audit` in CI
- Pin major versions
- Regular security updates