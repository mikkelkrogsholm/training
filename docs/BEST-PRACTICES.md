Here’s a tight, opinionated set of coding best practices tailored to this app (SvelteKit + TypeScript + Tailwind + Drizzle + SQLite, single-container, modular “kernel + modules”).

## 1) Language, config, and project hygiene

* **TypeScript strict**: `"strict": true`, `"noImplicitAny": true`, `"noUncheckedIndexedAccess": true`, `"exactOptionalPropertyTypes": true`.
* **ESLint + Prettier**: one source of formatting truth; block merges if `lint` or `format:check` fails.
* **Path aliases**: use `$lib/*`, `$core/*`, `$modules/*` to avoid brittle relative imports.
* **Small files, small components**: UI ≤ 200–300 lines; extract helpers early.
* **Conventional Commits**: `feat:`, `fix:`, `chore:`, etc. + CI checks.
* **Trunk-based**: short-lived branches; PRs under \~500 lines net diff where possible.

## 2) Architecture patterns

* **Kernel + modules**: modules depend on **core only**; never import across modules.
* **Extension points**: implement “slots” (`today.tiles`, `movement.detail.tabs`) and a simple event bus (`onSessionLogged`, …).
* **Domain-first code**: keep business logic (e.g., E1RM calc, schedule materialization) in `$lib/domain/*`, not in components or endpoints.
* **Pure functions**: analytics and normalization live in side-effect-free utilities with unit tests.
* **Adapters**: storage, AI, and settings behind small interfaces so you can swap implementations later (e.g., local vs. S3 media, local vs. API LLM).

## 3) Data & DB (Drizzle + SQLite)

* **WAL mode** for SQLite; create necessary indices (by `user_id`, `movement_id`, `date`) up front.
* **Migrations**: one direction, append-only; tracked in `_migrations`. Apply core first, then module migrations.
* **Validation in → out**: all API inputs validated with Zod; all DB reads mapped to typed DTOs (don’t leak raw DB shapes directly to UI).
* **Timestamps**: store UTC; convert to user TZ at the edge.
* **Units**: persist **canonical units** (kg, meters, seconds). Convert for display using user prefs.
* **Idempotency**: when materializing plans or importing sessions, use deterministic keys to avoid duplicates on retries.

## 4) API & server routes (SvelteKit)

* **Thin endpoints**: parse/validate input (Zod), call domain service, map to response.
* **Errors**: domain errors → typed HTTP responses; never leak stack traces. Map to humane toasts on the client.
* **Rate limiting**: login, password reset, AI endpoints. Keep per-IP and per-user guards.
* **CORS & CSRF**: use same-site secure cookies for sessions; CSRF token for mutating requests if you use non-cookie auth.

## 5) Auth, sessions, permissions

* **Sessions**: HTTP-only, `Secure`, `SameSite=Lax`, short-lived with rotation on privilege escalation.
* **Password hashing**: bcrypt with sane cost; constant-time comparisons.
* **RBAC**: `admin`, `user` + per-module permissions from the manifest. Enforce on server, not just UI.
* **Privacy toggles**: enforce on read queries (e.g., friends’ leaderboards honor visibility flags).

## 6) Validation & schema versioning

* **Zod everywhere**: routine JSON schema, module manifest, uploads, AI outputs.
* **`schema_version`** on routine JSON and a tiny migration layer. Reject or auto-migrate old versions with logs.

## 7) Frontend patterns (SvelteKit + Tailwind)

* **Mobile-first**: build for ≤390px first; progressive enhancements for larger screens.
* **State**: prefer local component state; use Svelte stores **only** for cross-route state (auth, settings).
* **Accessibility**: semantic HTML, keyboard focus management, visible focus rings, `prefers-reduced-motion` respected, captions for videos.
* **UI performance**: virtualize long lists if needed; debounce typeahead; lazy-load heavy charts and module routes.
* **Forms**: optimistic UI for set logging with rollback on failure.

## 8) Charts & analytics

* **Single source of truth**: compute E1RM/volume on the server (or at least in a shared util) so charts and leaderboards agree.
* **Explain units**: axis labels always show unit; tooltips show raw set and normalized value.
* **Sampling**: downsample for long time ranges to keep mobile smooth.

## 9) Media handling

* **Validations**: MIME sniff + extension check; file size limits; strip EXIF from images by default.
* **File naming**: `/data/media/{yyyy}/{mm}/{uuid}.{ext}`.
* **Thumbnails**: generate web-sized versions; serve responsive `<img srcset>`.
* **Unsafe content**: never render user-supplied HTML; markdown sanitized and linkified safely.

## 10) AI integration (trainer & editor)

* **Retrieval first**: ground answers in your guides/resources; include citations/links inside the app UI.
* **Deterministic contracts**: model returns **only JSON** for structured tasks; validate with Zod; retry with “repair” prompt if invalid.
* **Guardrails**: red-flag phrases (pain, dizziness) trigger conservative advice and “stop/seek professional help” language.
* **Telemetries**: never log sensitive prompt content verbatim; scrub PII; store minimal traces for debugging.

## 11) Offline & syncing (PWA)

* **Service worker**: cache app shell + current routine + movement guides + recent charts.
* **Write queue**: store pending logs in IndexedDB; on reconnect, sync with idempotency keys.
* **Conflict policy**: last-write-wins with user-facing warning if a merge happened.

## 12) Performance & scalability

* **N+1 queries**: batch reads (e.g., for movement stats) with `IN` queries; precompute weekly aggregates if needed.
* **Indices**: `sessions(user_id, date)`, `session_movements(movement_id)`, `plans(user_id, date)`.
* **Memory**: avoid loading large media lists into memory; stream downloads.
* **Chunky jobs**: run backups/transcodes off the request path (cron in same process is fine).

## 13) Observability & stability

* **Structured logs**: JSON logs with request id; log at boundaries (API in/out, DB failures, AI failures).
* **Error boundaries**: user-friendly fallbacks; keep the “Start Session” path resilient.
* **Backups**: nightly `.backup`; verify last backup on boot and expose “download latest” in admin.
* **Feature flags**: typed getter; flags stored in DB with env override; modules read flags at render.

## 14) Security basics

* **CSP**: strict default-src/self; allowlist for media/CDN if used; upgrade-insecure-requests.
* **Headers**: `X-Content-Type-Options: nosniff`, `Referrer-Policy: same-origin`, `Permissions-Policy` minimal.
* **Uploads**: never serve from the same hostname without a whitelist path; no user-executable files.
* **Dependency hygiene**: lockfile committed; periodic `npm audit` in CI; pin major versions.

## 15) Testing

* **Unit**: E1RM/volume calculators, schedule materialization, validators (routine, manifest, API payloads).
* **API**: contract tests with supertest (or SvelteKit’s test utils).
* **E2E**: Playwright for core flows (login → start session → log set → see chart update).
* **Seeds**: deterministic seed data for repeatable tests and dev.

## 16) Docs & dev ergonomics

* **README**: quickstart, env vars, scripts, architecture map, module system overview.
* **ADR notes**: brief markdown when you make a key decision (e.g., “Why single-container”).
* **API docs**: describe endpoints briefly with request/response Zod schemas.
* **Developer CLI**: small `npm run` scripts (`dev`, `build`, `db:migrate`, `db:studio`, `seed`, `backup:now`).

## 17) Deployment (Dokploy) niceties

* **Immutable image, mutable data**: only `/data` is a volume. Never write elsewhere.
* **Healthcheck**: simple `/api/health` that checks DB RW & returns module list.
* **Config via env**: no secrets in repo; fail fast if required env is missing.
* **Rollback**: keep previous image tagged; DB backups allow quick restore.

## 18) UI polish that pays off

* **One-handed logging**: large buttons, swipe-to-complete, “repeat last set”.
* **Latency**: prefetch next route; lazy-load heavy widgets; skeletons for charts.
* **Consistency**: shared button/input components; consistent spacing scale; dark mode first; motion subtle.

## 19) Code review checklist (use on every PR)

* Does this touch **validation**, **auth**, or **privacy**? If yes, tested paths?
* Any **new DB queries**: indices added? Explain complexity.
* UI: keyboard accessible? Mobile tested? High-contrast verified?
* Errors: friendly messages? Logging without PII?
* Tests: unit for logic, at least one integration for the new path.
* Docs: updated README/ADR if architecture changed.

## 20) Kill switches & safety

* Feature flags to disable modules (AI, calendar) if they misbehave.
* Circuit breaker around AI calls (timeouts, fallbacks).
* Hard limits on upload sizes, AI request frequency, and session length (server-side).

This keeps your codebase boring in the right places and expressive where it matters. Next sensible move is to scaffold the kernel with strict TypeScript, Zod validators, Drizzle schema, the ModuleRegistry, and a bare “Today” screen with a single slot—then grow outward in modules like Lego.
