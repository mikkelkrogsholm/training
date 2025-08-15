# Development Plan - Modular Fitness Web App

This plan breaks down the development into standalone phases. Each phase delivers working functionality that can be deployed and tested independently.

## Git Workflow for Phases

Each phase follows this git workflow:

1. **Start Phase**: Create branch `phase-{number}-{description}`
2. **During Development**: Commit with conventional format `feat(scope): message`
3. **Before Commits**: Run `/quick-test` to ensure quality
4. **Complete Phase**: Merge with `--no-ff` and tag `phase-{number}-complete`

See [GIT-STRATEGY.md](./GIT-STRATEGY.md) for detailed git practices.

## Phase 0: Project Setup & Foundation
**Goal:** Initialize project with all necessary tooling and configurations
**Branch:** `phase-0-project-setup`

### Tasks:
- [ ] Initialize SvelteKit project with TypeScript strict mode - **ui-component-agent**
- [ ] Configure Tailwind CSS with mobile-first utilities - **ui-component-agent**
- [ ] Setup Drizzle ORM with SQLite configuration - **db-schema-manager**
- [ ] Configure ESLint, Prettier, and path aliases - **ui-component-agent**
- [ ] Create Dockerfile and docker-compose.yml - **api-endpoint-agent**
- [ ] Setup environment variables and .env.example - **api-endpoint-agent**
- [ ] Initialize Git repository with .gitignore - **ui-component-agent**
- [ ] Configure Vitest and Playwright for testing - **testing-agent**

## Phase 1: Kernel Core - Database & Auth
**Goal:** Establish database foundation and authentication system
**Branch:** `phase-1-auth-system`

### Tasks:
- [ ] Create core database schema (users, settings, _migrations) - **db-schema-manager**
- [ ] Implement SQLite WAL mode configuration - **db-schema-manager**
- [ ] Create initial migration system - **db-schema-manager**
- [ ] Implement bcrypt password hashing utilities - **domain-logic-agent**
- [ ] Create session management with HTTP-only cookies - **api-endpoint-agent**
- [ ] Implement /api/auth/login endpoint with rate limiting - **api-endpoint-agent**
- [ ] Implement /api/auth/logout endpoint - **api-endpoint-agent**
- [ ] Create auth middleware for protected routes - **api-endpoint-agent**
- [ ] Build login page with mobile-first design - **ui-component-agent**
- [ ] Add auth context/store for client-side state - **ui-component-agent**
- [ ] Write auth integration tests - **testing-agent**

## Phase 2: Kernel Core - Module System
**Goal:** Implement the module registry and extension points
**Branch:** `phase-2-module-system`

### Tasks:
- [ ] Create ModuleRegistry class in $lib - **domain-logic-agent**
- [ ] Implement module.json Zod validator - **domain-logic-agent**
- [ ] Create extension points system (slots, events) - **domain-logic-agent**
- [ ] Build module loader with migration support - **domain-logic-agent**
- [ ] Implement UI slot renderer component - **ui-component-agent**
- [ ] Create event bus for module communication - **domain-logic-agent**
- [ ] Setup module-specific route registration - **api-endpoint-agent**
- [ ] Add module settings to database - **db-schema-manager**
- [ ] Create /api/admin/modules endpoint - **api-endpoint-agent**
- [ ] Write module system unit tests - **testing-agent**

## Phase 3: Movements & Guides Foundation
**Goal:** Enable movement definitions and guidance content
**Branch:** `phase-3-movements`

### Tasks:
- [ ] Create movements table schema - **db-schema-manager**
- [ ] Create movement_guides table schema - **db-schema-manager**
- [ ] Create media_assets table schema - **db-schema-manager**
- [ ] Implement /api/movements CRUD endpoints - **api-endpoint-agent**
- [ ] Implement /api/guides CRUD endpoints - **api-endpoint-agent**
- [ ] Create movement DTO mappers - **domain-logic-agent**
- [ ] Build movements list page - **ui-component-agent**
- [ ] Build movement detail page with tabs slot - **ui-component-agent**
- [ ] Implement markdown renderer for guides - **ui-component-agent**
- [ ] Create movement search/filter component - **ui-component-agent**
- [ ] Add movement seed data script - **testing-agent**
- [ ] Write movement API tests - **testing-agent**

## Phase 4: Routines & Session Logging
**Goal:** Core fitness functionality - routines and workout logging
**Branch:** `phase-4-session-logging`

### Tasks:
- [ ] Create routines table schema - **db-schema-manager**
- [ ] Create sessions & session_movements schemas - **db-schema-manager**
- [ ] Create routine JSON Zod validator (v1.0.0) - **domain-logic-agent**
- [ ] Implement routine JSON processor - **domain-logic-agent**
- [ ] Create /api/routines endpoints - **api-endpoint-agent**
- [ ] Create /api/sessions endpoints - **api-endpoint-agent**
- [ ] Build Today screen with slot system - **ui-component-agent**
- [ ] Implement Start Session tile (kernel) - **ui-component-agent**
- [ ] Create session logging UI (one-handed) - **ui-component-agent**
- [ ] Build routine viewer with day/week views - **ui-component-agent**
- [ ] Implement set/rep/weight input components - **ui-component-agent**
- [ ] Add optimistic UI for logging - **ui-component-agent**
- [ ] Create sample routine seed data - **testing-agent**
- [ ] Write E2E test for session logging flow - **testing-agent**

## Phase 5: Analytics & Normalization
**Goal:** Progress tracking with E1RM calculations and charts
**Branch:** `phase-5-analytics`

### Tasks:
- [ ] Implement E1RM calculator (Epley formula) - **domain-logic-agent**
- [ ] Create volume calculator functions - **domain-logic-agent**
- [ ] Implement relative strength calculator - **domain-logic-agent**
- [ ] Create /api/analytics endpoints - **api-endpoint-agent**
- [ ] Add bodyweight_logs table - **db-schema-manager**
- [ ] Setup Chart.js or ApexCharts - **ui-component-agent**
- [ ] Build E1RM chart component - **ui-component-agent**
- [ ] Build volume chart component - **ui-component-agent**
- [ ] Create PR detection logic - **domain-logic-agent**
- [ ] Add movement history tab content - **ui-component-agent**
- [ ] Write calculator unit tests - **testing-agent**
- [ ] Test chart rendering on mobile - **testing-agent**

## Phase 6: Admin Module
**Goal:** Administrative controls for users and system
**Branch:** `module-admin`

### Tasks:
- [ ] Scaffold admin module structure - **module-builder**
- [ ] Create admin-only middleware - **api-endpoint-agent**
- [ ] Implement /api/admin/users endpoints - **api-endpoint-agent**
- [ ] Create user management UI - **ui-component-agent**
- [ ] Implement password reset functionality - **api-endpoint-agent**
- [ ] Create backup/restore endpoints - **api-endpoint-agent**
- [ ] Build backup management UI - **ui-component-agent**
- [ ] Add routine publish/unpublish controls - **api-endpoint-agent**
- [ ] Create module toggle interface - **ui-component-agent**
- [ ] Setup nightly backup cron job - **domain-logic-agent**
- [ ] Write admin access tests - **testing-agent**

## Phase 7: Calendar Module
**Goal:** Planning and scheduling workouts
**Branch:** `module-calendar`

### Tasks:
- [ ] Scaffold calendar module - **module-builder**
- [ ] Create plans table schema - **db-schema-manager**
- [ ] Implement schedule materialization - **domain-logic-agent**
- [ ] Create /api/plans endpoints - **api-endpoint-agent**
- [ ] Build month view calendar - **ui-component-agent**
- [ ] Build week agenda view - **ui-component-agent**
- [ ] Create calendar tile for Today screen - **ui-component-agent**
- [ ] Implement plan state management - **domain-logic-agent**
- [ ] Add auto-reschedule logic - **domain-logic-agent**
- [ ] Create calendar module migration - **db-schema-manager**
- [ ] Write materialization tests - **testing-agent**

## Phase 8: Media Upload & Management
**Goal:** Support for images and videos in guides
**Branch:** `phase-8-media`

### Tasks:
- [ ] Create /api/upload endpoint - **api-endpoint-agent**
- [ ] Implement file validation (MIME, size) - **domain-logic-agent**
- [ ] Setup media storage structure - **api-endpoint-agent**
- [ ] Create thumbnail generator (sharp) - **domain-logic-agent**
- [ ] Build media upload component - **ui-component-agent**
- [ ] Implement media gallery viewer - **ui-component-agent**
- [ ] Add media to movement guides - **ui-component-agent**
- [ ] Setup media cleanup job - **domain-logic-agent**
- [ ] Write upload security tests - **testing-agent**

## Phase 9: Friends Module
**Goal:** Social features with privacy controls
**Branch:** `module-friends`

### Tasks:
- [ ] Scaffold friends module - **module-builder**
- [ ] Create friends table schema - **db-schema-manager**
- [ ] Implement friend request system - **api-endpoint-agent**
- [ ] Create privacy settings in user profile - **api-endpoint-agent**
- [ ] Build friends list UI - **ui-component-agent**
- [ ] Create leaderboard components - **ui-component-agent**
- [ ] Implement visibility filters in queries - **domain-logic-agent**
- [ ] Add friend activity to calendar - **ui-component-agent**
- [ ] Create friend module widgets - **ui-component-agent**
- [ ] Write privacy enforcement tests - **testing-agent**

## Phase 10: Analytics Module (Enhanced)
**Goal:** Advanced charts and insights
**Branch:** `module-analytics`

### Tasks:
- [ ] Scaffold analytics module - **module-builder**
- [ ] Create frequency heatmap component - **ui-component-agent**
- [ ] Build comparative charts (user vs friends) - **ui-component-agent**
- [ ] Implement volume trend analysis - **domain-logic-agent**
- [ ] Create analytics dashboard page - **ui-component-agent**
- [ ] Add analytics tile to Today screen - **ui-component-agent**
- [ ] Build leaderboard queries - **domain-logic-agent**
- [ ] Implement data downsampling - **domain-logic-agent**
- [ ] Create export functionality - **api-endpoint-agent**
- [ ] Test chart performance on mobile - **testing-agent**

## Phase 11: Production Deployment
**Goal:** Deploy to Dokploy with proper configuration
**Branch:** `phase-11-production`

### Tasks:
- [ ] Configure health check endpoint - **api-endpoint-agent**
- [ ] Setup security headers and CSP - **api-endpoint-agent**
- [ ] Implement error boundaries - **ui-component-agent**
- [ ] Configure structured logging - **api-endpoint-agent**
- [ ] Setup environment validation - **domain-logic-agent**
- [ ] Create production seed data - **testing-agent**
- [ ] Test backup/restore process - **testing-agent**
- [ ] Configure Dokploy deployment - **api-endpoint-agent**
- [ ] Setup domain and SSL - **api-endpoint-agent**
- [ ] Run production smoke tests - **testing-agent**

## Phase 12: PWA & Offline (Optional)
**Goal:** Progressive Web App with offline support
**Branch:** `phase-12-pwa`

### Tasks:
- [ ] Create service worker - **ui-component-agent**
- [ ] Implement cache strategies - **ui-component-agent**
- [ ] Setup IndexedDB for offline queue - **ui-component-agent**
- [ ] Add offline indicators - **ui-component-agent**
- [ ] Implement sync on reconnect - **domain-logic-agent**
- [ ] Create PWA manifest - **ui-component-agent**
- [ ] Test offline scenarios - **testing-agent**

## Phase 13: AI Trainer Module (Optional)
**Goal:** AI-powered fitness assistant
**Branch:** `module-ai-trainer`

### Tasks:
- [ ] Scaffold AI trainer module - **module-builder**
- [ ] Implement RAG retrieval system - **domain-logic-agent**
- [ ] Create context builder (sessions, metrics) - **domain-logic-agent**
- [ ] Setup OpenAI integration - **api-endpoint-agent**
- [ ] Implement safety guardrails - **domain-logic-agent**
- [ ] Build chat interface - **ui-component-agent**
- [ ] Create AI trainer tile - **ui-component-agent**
- [ ] Add response caching - **api-endpoint-agent**
- [ ] Test safety responses - **testing-agent**

## Phase 14: AI Routine Editor Module (Optional)
**Goal:** Generate routines through dialogue
**Branch:** `module-ai-editor`

### Tasks:
- [ ] Scaffold routine editor module - **module-builder**
- [ ] Create dialogue flow system - **domain-logic-agent**
- [ ] Implement JSON generator with Zod - **domain-logic-agent**
- [ ] Build conversation UI - **ui-component-agent**
- [ ] Create validation/repair loop - **domain-logic-agent**
- [ ] Add routine preview component - **ui-component-agent**
- [ ] Implement save/publish flow - **api-endpoint-agent**
- [ ] Test JSON generation accuracy - **testing-agent**

## Success Metrics per Phase

- **Phase 0-2**: Project runs locally, auth works, modules can be registered
- **Phase 3-4**: Can create movements, routines, and log sessions
- **Phase 5**: Can see progress charts and analytics
- **Phase 6**: Admin can manage users and backups
- **Phase 7-10**: Modules enhance functionality without breaking core
- **Phase 11**: App runs in production on Dokploy
- **Phase 12-14**: Optional enhancements work seamlessly

## Notes

1. Each phase produces a deployable increment
2. Phases 1-6 are the critical MVP path
3. Phases 7-10 add modular features
4. Phases 12-14 are optional enhancements
5. Always run tests after each phase completion
6. Document any deviations from the plan in CHANGELOG.md

## Getting Started

1. Complete Phase 0 to establish the foundation
2. Work through phases sequentially for MVP (1-6)
3. Add modules (7-10) based on user priorities
4. Deploy to production (11) when MVP is ready
5. Consider optional phases (12-14) after gathering feedback