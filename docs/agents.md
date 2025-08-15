# Available Agents for Fitness App Development

This document describes the specialized agents available for developing the modular fitness web application. Each agent follows separation of concerns and focuses on a specific aspect of the codebase.

## db-schema-manager

**Focus:** Database schema, migrations, and Drizzle ORM operations

**Responsibilities:**
- Creates and modifies Drizzle schema files in `/src/core/db` and `/src/modules/*/db`
- Generates database migrations for both core and module components
- Implements proper table relationships and foreign keys
- Configures SQLite with WAL mode for better concurrency
- Ensures canonical units (kg, meters, seconds) are stored correctly
- Creates appropriate indices for `user_id`, `movement_id`, and `date` columns

**When to use:** When adding new tables, modifying schema, creating migrations, or optimizing database performance.

## module-builder

**Focus:** Creating new modules following the kernel+modules architectural pattern

**Responsibilities:**
- Scaffolds complete module directory structure under `/src/modules`
- Creates valid `module.json` manifests with Zod validation schemas
- Implements UI slot injections (e.g., `today.tiles`, `movement.detail.tabs`)
- Sets up event subscriptions (e.g., `onSessionLogged`, `onRoutinePublished`)
- Configures module-specific routes, widgets, migrations, and cron jobs
- Ensures strict module isolation (no cross-module imports)

**When to use:** When adding new features as modules (e.g., Calendar, Analytics, AI Trainer, Friends).

## api-endpoint-agent

**Focus:** SvelteKit API routes and server-side endpoint logic

**Responsibilities:**
- Implements thin endpoints following the pattern: validate → service → response
- Creates Zod schemas for request/response validation
- Handles authentication and authorization checks
- Implements rate limiting for sensitive endpoints
- Maps database entities to DTOs (never exposes raw DB shapes)
- Provides proper error responses without leaking stack traces

**When to use:** When creating or modifying API endpoints in `/src/routes/api`.

## ui-component-agent

**Focus:** SvelteKit components and mobile-first user interface

**Responsibilities:**
- Creates accessible, touch-friendly UI components
- Implements mobile-first design (≤390px viewport priority)
- Applies Tailwind CSS with dark mode support
- Ensures proper keyboard navigation and focus management
- Implements one-handed logging interfaces with large tap targets
- Handles form validation and optimistic UI updates

**When to use:** When building user-facing components, pages, or improving mobile UX.

## domain-logic-agent

**Focus:** Pure business logic functions in `$lib/domain`

**Responsibilities:**
- Implements E1RM calculations using Epley formula
- Creates volume tracking and analytics functions
- Handles schedule materialization from routine templates
- Processes routine JSON with version migrations
- Writes side-effect-free, unit-testable pure functions
- Implements normalization logic for movements and metrics

**When to use:** When implementing business rules, calculations, or data transformations.

## testing-agent

**Focus:** Unit, integration, and E2E tests

**Responsibilities:**
- Writes Vitest unit tests for domain logic
- Creates API contract tests with supertest
- Implements Playwright E2E tests for critical user flows
- Generates deterministic seed data for testing
- Tests Zod validators and error handling
- Ensures core flows are tested (login → session → logging → charts)

**When to use:** When writing tests for new features or improving test coverage.

## Usage Guidelines

1. **Choose the right agent:** Select based on the primary concern of your task
2. **Provide context:** Include relevant file paths and requirements
3. **Follow conventions:** Each agent adheres to the project's best practices
4. **Test coverage:** Use testing-agent after implementing features
5. **Module boundaries:** Respect the kernel+modules architecture

## Example Workflow

1. Use `db-schema-manager` to create database tables
2. Use `module-builder` to scaffold a new feature module
3. Use `domain-logic-agent` for business logic
4. Use `api-endpoint-agent` for API routes
5. Use `ui-component-agent` for the interface
6. Use `testing-agent` to ensure quality

Each agent is optimized for its specific domain and will follow the project's established patterns and best practices.