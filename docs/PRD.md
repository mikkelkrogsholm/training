# Product Requirements Document (PRD)

**Project:** Modular Fitness Web App (“kernel + modules”)
**Owner:** Mikkel
**Primary users:** Mikkel, his son, small circle of family/friends
**Deployment:** Single-container app via Dokploy from a GitHub repo
**DB:** SQLite (mounted volume)
**Design:** Mobile-first, clean, high-contrast, touch-friendly
**Modularity principle:** “Ink & paper” — start with a base app (kernel), then add optional modules like Calendar, Analytics, AI Trainer, Routine Editor, Friends, Admin, etc.

---

## 1) Goals

* A private, lightweight fitness platform where “routines” act like self-contained mini sub-apps with guidance (text, images, video) and a flexible schedule.
* Simple deployment and ops: one container, SQLite file, one Dokploy service.
* Modular growth: enable/disable modules at runtime without changing the deploy topology.
* Multi-user with profiles, optional friends/leaderboards, and privacy controls.
* First-class progress tracking with normalized comparisons (E1RM, volume, relative strength).
* Optional AI “personal trainer” and AI “routine editor” that operate on the app’s own content.

### Non-Goals (initially)

* Public signup / open social network
* Heavy streaming video infrastructure or live coaching
* Multi-tenant SaaS or multi-region scaling

---

## 2) Personas & Use Cases

* **Mikkel (power user):** Creates routines, invites son and friends, wants graphs and normalized comparisons, wants an AI editor to quickly draft routines.
* **Son (casual user):** Executes routine, logs sets quickly on mobile, likes seeing progress and light competition.
* **Friend (optional):** Joins, follows a routine, shares selective stats.
* **Admin (often Mikkel):** Manages users, resets passwords, edits data, triggers restore.

Key flows:

* Onboard user → pick a routine → “Today” view → log a session quickly → see progress → optionally ask AI a question → see calendar and friends’ activity.

---

## 3) High-Level Architecture

* **Single-container monolith** built with **SvelteKit** (`adapter-node`) for UI + server routes.
* **SQLite** at `/data/db.sqlite` (Dokploy volume).
* **Drizzle ORM** for schema and migrations.
* **Tailwind CSS** for styling.
* **Charting:** Chart.js or ApexCharts (mobile-friendly; no seaborn etc.).
* **Images/Video:** stored under `/data/media` with simple upload; optional transcoding.
* **Module system inside the monolith**:

  * Kernel provides shell, auth, DB, settings, events, slot rendering.
  * Modules ship their own routes, widgets, migrations, and jobs via a manifest.
  * Kernel loads manifests at boot, applies migrations, registers widgets and routes.

---

## 4) Module System

### 4.1 Folder layout

```
/src
  /core                  # kernel: shell, auth, db client, settings, events, slots
  /modules
    /analytics
      module.json
      /routes
      /widgets
      /db
      /jobs
    /calendar
    /ai-trainer
    /routine-editor
    /friends
    /admin
  /lib
    module-registry.ts
    extension-points.ts
/drizzle                 # core migrations
/static                  # public assets
/scripts                 # backups, utilities
/data                    # runtime volume mount (db, media, backups)
```

### 4.2 Module manifest (validated with Zod)

```json
{
  "id": "calendar",
  "name": "Calendar",
  "version": "0.1.0",
  "requires": ["core>=0.1.0"],
  "routes": [
    { "path": "/calendar", "file": "routes/calendar/+page.svelte" }
  ],
  "widgets": [
    { "slot": "today.tiles", "file": "widgets/TodayCalendarTile.svelte" }
  ],
  "permissions": ["calendar:view"],
  "settings": [
    { "key": "calendar.showFriends", "type": "boolean", "default": true }
  ],
  "migrations": ["db/0001_init.sql"],
  "jobs": [
    { "cron": "0 2 * * *", "file": "jobs/nightly-summary.ts" }
  ]
}
```

### 4.3 Extension points

* **UI slots:** `today.tiles`, `movement.detail.tabs`, `routine.header.actions`, `profile.panels`.
* **Events (pub/sub):** `onSessionLogged`, `onRoutinePublished`, `onUserCreated`, `onMediaUploaded`.
* **DB migrations:** module migrations applied on boot; tracked in `_migrations`.
* **Jobs:** cron scheduled by the kernel using `node-cron`.

---

## 5) Data Model (SQLite via Drizzle)

### 5.1 Entities (fields are required unless noted)

**users**

* id (pk, uuid)
* username (unique)
* display\_name
* email (unique)
* password\_hash (bcrypt)
* role (“admin” | “user”)
* unit\_preferences (json: { mass: "kg"|"lb", distance: "km"|"mi" }) default {kg, km}
* time\_zone (string) default Europe/Copenhagen
* created\_at (datetime)

**friends**

* id (pk)
* user\_id (fk users)
* friend\_id (fk users)
* status (“pending” | “accepted” | “blocked”)

**movements**

* id (pk)
* name (unique per owner scope)
* category (string)
* unit (“kg” | “lb” | “reps” | “sec” | “m”)
* created\_by (fk users)
* aliases (json string\[]) optional
* variation\_of (fk movements.id) optional

**movement\_guides**

* id (pk)
* movement\_id (fk movements) nullable for general articles
* title
* body\_md (markdown)
* media (json: array of {type:"image"|"video"|"text"|"link", url, caption?})
* updated\_at

**routines**

* id (pk)
* user\_id (owner, fk users)
* name
* description
* color\_theme (hex)
* category (“strength” | “cardio” | “mobility” | “dance” | “hybrid”)
* schema\_version (string, e.g., “1.0.0”)
* routine\_json (json)  // entire routine definition (see §6)
* duration\_weeks (int)
* published (boolean)
* created\_at

**routine\_movements** (optional helper for indexing/search)

* id (pk)
* routine\_id (fk routines)
* movement\_id (fk movements)
* order\_index (int)

**plans**  // materialized “planned sessions” for calendar

* id (pk)
* user\_id (fk users)
* routine\_id (fk routines)
* date (date)
* state (“planned” | “completed” | “skipped”)
* notes (string?)

**sessions**

* id (pk)
* user\_id (fk users)
* routine\_id (fk routines) nullable (ad-hoc sessions)
* plan\_id (fk plans) nullable
* date (datetime)
* notes (string?)

**session\_movements**

* id (pk)
* session\_id (fk sessions)
* movement\_id (fk movements)
* sets\_completed (int)
* reps\_per\_set (json number\[])
* weight\_per\_set (json number\[])  // in base unit (kg)
* time\_sec (int?)  // for cardio/timed
* distance\_m (int?)
* avg\_hr\_bpm (int?)
* rpe\_per\_set (json number\[]?)
* notes (string?)

**bodyweight\_logs**

* id (pk)
* user\_id (fk users)
* date (date)
* weight\_kg (real)

**media\_assets**

* id (pk)
* kind (“image” | “video” | “doc”)
* url
* caption (string?)
* attribution (string?)
* uploaded\_by (fk users)
* created\_at

**settings** (key-value store)

* key (pk)
* value (json)

**\_migrations**

* id (pk) // “moduleId\:filePath” applied marker

---

## 6) Routine JSON Schema (versioned)

* Keep routines self-contained but don’t duplicate long-form guidance; link to `movement_guides` where possible.
* Validate with Zod on create/update; include `schema_version`.

```json
{
  "schema_version": "1.0.0",
  "id": "uuid-or-slug",
  "name": "GtG + PNF + Cluster Set – 8-week cycle",
  "description": "Focused plan combining heavy strength, Grease the Groove and mobility.",
  "category": "strength",
  "color_theme": "#22c55e",
  "duration_weeks": 8,
  "start_mode": "weekday", 
  "schedule": [
    {
      "label": "Monday", 
      "type": "heavy", 
      "blocks": [
        {
          "kind": "movement",
          "title": "Back Squat – Cluster Set",
          "movement_ref": "back_squat", 
          "prescription": {
            "sets": 1,
            "reps": "5x1",
            "intensity": "88–92% 1RM",
            "rest": "15s between reps",
            "tempo": null
          },
          "guidance": "Brace, stay tight, control descent.",
          "media": [
            { "type": "guide_ref", "id": "guide_back_squat_basics" },
            { "type": "video", "url": "https://..." }
          ]
        },
        {
          "kind": "movement",
          "title": "Bench Press – Cluster Set",
          "movement_ref": "bench_press",
          "prescription": { "sets": 1, "reps": "5x1", "intensity": "88–92% 1RM", "rest": "15s" },
          "media": [{ "type": "guide_ref", "id": "guide_bench_setup" }]
        }
      ]
    },
    {
      "label": "Tuesday",
      "type": "light",
      "blocks": [
        {
          "kind": "circuit",
          "title": "GtG – Technique Focus",
          "movements": ["pull_up", "push_up", "hinge_drill"],
          "instructions": "30–60% effort, never to failure.",
          "media": [{ "type": "text", "content": "Grease the Groove explained..." }]
        },
        {
          "kind": "mobility",
          "title": "PNF Stretching – 10 min",
          "movements": ["hip_flexor_pnf", "hamstring_pnf"],
          "media": [{ "type": "image", "url": "/media/hip_flexor_demo.jpg" }]
        }
      ]
    }
  ],
  "resources": [
    { "type": "guide_ref", "id": "guide_cluster_sets" },
    { "type": "video", "title": "Proper Squat Form", "url": "https://..." }
  ],
  "progression_rules": [
    {
      "movement_ref": "back_squat",
      "rule": "e1rm_rpe_increment",
      "params": { "threshold_rpe": 8.0, "inc_kg": 2.5 }
    }
  ],
  "created_by": "user-id"
}
```

---

## 7) Normalization & Analytics

* **Estimated 1RM (E1RM)**
  Prefer **Epley**: `E1RM = weight × (1 + reps/30)`
  Optionally support Brzycki: `E1RM = weight × (36 / (37 - reps))`
* **Relative strength**
  `relative_strength = E1RM / bodyweight_kg_at_date`
* **Volume load**
  `volume = Σ (weight × reps)`

Charts to provide:

* E1RM over time per movement (per user; compare to friends via toggle).
* Weekly volume per movement and total.
* Frequency heatmap (training days).
* PR markers.
* Leaderboards: best E1RM per movement; most sessions this month.

Example queries:

```sql
-- daily best E1RM for a movement for user
SELECT s.date as day,
       MAX( (json_extract(sm.weight_per_set, '$['||i||']')) * (1 + (json_extract(sm.reps_per_set, '$['||i||']'))/30.0) ) AS best_e1rm
FROM session_movements sm
JOIN sessions s ON sm.session_id = s.id
JOIN generate_series(0, json_array_length(sm.reps_per_set)-1) AS i
WHERE sm.movement_id = ? AND s.user_id = ?
GROUP BY day ORDER BY day ASC;

-- leaderboard: best E1RM per user
SELECT s.user_id,
       MAX( (json_extract(sm.weight_per_set, '$['||i||']')) * (1 + (json_extract(sm.reps_per_set, '$['||i||']'))/30.0) ) AS best_e1rm
FROM session_movements sm
JOIN sessions s ON sm.session_id = s.id
JOIN generate_series(0, json_array_length(sm.reps_per_set)-1) AS i
WHERE sm.movement_id = ?
GROUP BY s.user_id
ORDER BY best_e1rm DESC;
```

---

## 8) Calendar & Plans

* **plans** materializes the schedule: created when a routine is assigned or started.
* States: `planned`, `completed`, `skipped`.
* Month view and weekly agenda.
* Toggle to overlay friends’ completed sessions.

Auto-reschedule rule (MVP): if a heavy planned day is missed, mark `skipped` and push the next heavy block forward by 1–2 days if there’s no conflict (simple heuristic).

---

## 9) AI Integrations (optional modules)

### 9.1 AI Trainer (RAG Q\&A + gentle adaptation)

* **Retrieval-first** over `movement_guides` and routine `resources`.
* Adds user context: last 10 sessions, last 30 days of E1RM, today’s plan.
* Safety rails: never override medical red flags; provide conservative recommendations.
* Output formats: short tip, expanded explanation, linked guide references.

**System prompt (core idea):**

```
You are a concise, evidence-aware fitness assistant for a private app.
Use ONLY the provided docs and routine JSON for definitions and instructions.
If something isn't in docs, say what is known and state uncertainty.
Never prescribe medical advice. Emphasize safety.
Respond in short, clear sentences suitable for mobile.
```

**Retriever context keys:**

* movement\_guides for mentioned movements
* routine.resources
* recent sessions (structured summary)
* normalized metrics (latest E1RM, trend)

### 9.2 AI Routine Editor (dialogue → JSON)

* Clarifies goals, schedule, equipment, constraints.
* Produces routine JSON conforming to schema with `schema_version`.
* Must include guidance links; if missing, create placeholder guides.

**Output contract:** exactly one JSON object; on backend, validate with Zod; reject or repair with model if invalid.

---

## 10) Media Management

* Upload endpoint saves files under `/data/media/{yyyy}/{mm}/uuid.ext`.
* Allowed types: images (png, jpg, webp), video (mp4, webm).
* Optional thumbnailer (sharp) and optional ffmpeg transcode to H.264 + webm.
* `media_assets` record created; `movement_guides.media` or routine `media` reference these URLs.

---

## 11) Privacy & Social

* Friends must be accepted (no auto-follow).
* Visibility toggles per user:

  * Show profile to friends
  * Share PRs
  * Share session notes
  * Share bodyweight (off by default)
* Kid-friendly defaults: minimal sharing; no public discoverability.

---

## 12) Auth, Security, Admin

* Password login; bcrypt hashes; rate limit login attempts.
* Optional magic-link email login (MVP can skip if SMTP is not configured).
* Roles: `admin`, `user`.
* `/admin` features:

  * Create/edit users, reset passwords.
  * Publish/unpublish routines.
  * Edit or delete sessions and plans.
  * Backup/restore controls (select .sqlite snapshot).
  * Feature toggles for modules.

---

## 13) PWA & Offline (optional, but recommended)

* Service Worker caches shell, current routine, movement guides, recent charts data, and media thumbnails.
* Local queue for session logs when offline; sync on reconnect (last-write-wins).
* “Offline” banner and optimistic UI for set logging.

---

## 14) Observability & Backups

* Structured server logs (JSON).
* Nightly SQLite backup to `/data/backups/backup-YYYY-MM-DD-HHMM.sqlite`.
* Button in admin to download latest backup.
* Error boundary UI and basic error tracking (e.g., Sentry) if env set.

Backup script:

```sh
#!/bin/sh
set -e
mkdir -p "$BACKUP_DIR"
sqlite3 /data/db.sqlite ".backup '$BACKUP_DIR/backup-$(date +%F-%H%M).sqlite'"
```

---

## 15) API Surface (SvelteKit endpoints)

**Auth**

* POST `/api/auth/login` { username, password } → { token }
* POST `/api/auth/logout`
* POST `/api/admin/users` (admin) create user
* POST `/api/admin/users/:id/reset` (admin)

**Routines**

* GET `/api/routines` (mine + shared)
* POST `/api/routines` (create from JSON)
* GET `/api/routines/:id`
* POST `/api/routines/:id/publish` (admin or owner)

**Plans & Sessions**

* POST `/api/plans/generate` { routine\_id, user\_id, start\_date }
* GET `/api/plans?from=...&to=...`
* PATCH `/api/plans/:id` { state, notes }
* POST `/api/sessions` { routine\_id?, plan\_id?, date, notes, movements\[...] }
* PATCH `/api/sessions/:id`
* GET `/api/sessions?movement_id=...&user_id=...`

**Movements & Guides**

* GET `/api/movements`
* POST `/api/movements`
* GET `/api/guides/:id`
* POST `/api/guides`
* POST `/api/upload` multipart

**Friends**

* POST `/api/friends/request` { user\_id }
* POST `/api/friends/:id/accept`
* GET `/api/friends`

**Analytics**

* GET `/api/analytics/movement/:id/e1rm?user_id=...`
* GET `/api/analytics/volume?user_id=...&range=...`

**AI (if enabled)**

* POST `/api/ai/trainer` { message }
* POST `/api/ai/routine-editor` { transcript or answers } → routine JSON

---

## 16) UI/UX Specifications

**Global**

* Mobile-first; large tap targets; dark mode by default; accent color per routine.
* Bottom nav: Today, Movements, Calendar (if module), Friends (if module), Profile.

**Today Screen (kernel)**

* Header: date, routine picker.
* Tiles (slot `today.tiles`):

  * “Start Session” tile (kernel)
  * Calendar mini tile (calendar module)
  * AI Trainer tile (ai-trainer module)
  * Volume trend tile (analytics module)
* Start Session flow: show blocks; quick-set logging with one-hand controls; swipe to complete; long-press to edit.

**Routine Viewer**

* Routine summary (color theme), days list or week layout.
* Tap day → blocks with guidance previews and inline media.

**Movement Detail**

* Tabs slot `movement.detail.tabs`: Overview | E1RM | Volume | History | Guides
* PR badges and quick filters for time ranges.

**Calendar (module)**

* Monthly grid; planned vs completed badges; friend overlay toggle.
* Weekly agenda list optimized for mobile.

**Friends (module)**

* Requests & accepted.
* Leaderboard mini-cards.
* Privacy settings per user.

**Admin (module)**

* Users list, create/reset.
* Routines publish/unpublish.
* Backups list, download/restore.
* Module toggles and settings.

Accessibility:

* Focus states, semantic landmarks, ARIA on interactive pieces, caption fields for videos.

---

## 17) Settings & Feature Flags

* Stored in DB (`settings`) and read at boot; env overrides for secrets.
* Example flags: `modules.calendar.enabled`, `modules.ai_trainer.enabled`, `modules.analytics.enabled`.
* Per-module settings seeded from `module.json`.

---

## 18) Environment & Deployment

**Env vars**

```
DATABASE_URL=file:/data/db.sqlite
MEDIA_DIR=/data/media
BACKUP_DIR=/data/backups
SESSION_SECRET=change-me
OPENAI_API_KEY=optional
BASE_URL=https://your-domain.tld
```

**Dockerfile (single container)**

```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
RUN apk add --no-cache sqlite
ENV NODE_ENV=production
COPY --from=build /app/build ./build
COPY --from=build /app/package*.json ./
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/scripts ./scripts
VOLUME ["/data"]
ENV DATABASE_URL="file:/data/db.sqlite"
ENV MEDIA_DIR="/data/media"
ENV BACKUP_DIR="/data/backups"
EXPOSE 3000
CMD ["node", "build/index.js"]
```

**Compose (Dokploy)**

```yaml
services:
  app:
    build: .
    ports: ["3000:3000"]
    restart: unless-stopped
    environment:
      - DATABASE_URL=file:/data/db.sqlite
      - MEDIA_DIR=/data/media
      - BACKUP_DIR=/data/backups
      - SESSION_SECRET=${SESSION_SECRET}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    volumes:
      - appdata:/data
volumes:
  appdata:
```

DNS + TLS: use your own domain in Dokploy with Let’s Encrypt (avoid rate-limited shared domains).

---

## 19) Testing & Seed

* Seed script: creates admin user, 2 normal users, 1 strength routine, some plans, and 2 weeks of sample sessions including PRs.
* Unit tests for validators (routine JSON, uploads), analytics calculators (E1RM, volume).
* E2E smoke: login, select routine, start session, log set, see updated chart.

---

## 20) Acceptance Criteria (MVP)

* Users can log in and see a **Today** screen with their assigned routine.
* Can start a session, log sets/reps/weight/RPE, and save.
* Movement graphs show E1RM over time and volume per week.
* Calendar module (if enabled) displays planned vs completed sessions.
* AI Trainer (if enabled) answers with references to app guides, not hallucinations.
* Admin can create users, reset passwords, publish/unpublish routines, and trigger a backup.
* Nightly backup files appear in `/data/backups`.
* App runs in a single container via Dokploy with a persistent volume.

---

## 21) Roadmap (Ink → Paper)

**Phase 1 (Kernel)**

* Auth, profiles, movements, guides
* Routine viewer from JSON + validator
* Session logging + Today screen
* Basic analytics (E1RM per movement), movement detail
* Backups (cron) + Admin basics

**Phase 2 (Modules)**

* Analytics module: more charts, leaderboards
* Calendar module: plans materialization + views
* Friends module: requests + privacy
* AI Trainer module (RAG Q\&A)
* AI Routine Editor module (dialogue → JSON)

**Phase 3 (Polish)**

* PWA/offline sync
* Media transcoding
* Notifications (email digests; push later)
* Visual routine editor (drag/drop)

---

## 22) Out of Scope (for now)

* Public registration and discovery
* Payments, subscriptions
* Wearables integration and live telemetry
* Full-blown HIPAA/GDPR workflows beyond common-sense privacy

---

## 23) Known Risks & Mitigations

* **AI outputs invalid JSON:** enforce Zod validation; repair loop; reject if unrecoverable.
* **SQLite concurrency:** fine for small group; keep writes short; consider WAL mode.
* **Media bloat:** store thumbnails; link out to originals; set upload size limits.
* **Normalization misunderstandings:** UI clearly labels E1RM/relative strength; tooltips.
* **Child privacy:** defaults private; explicit opt-in for sharing.

---

## 24) Developer Notes (Claude Code pointers)

* Use **SvelteKit** + Tailwind; keep components small and accessible.
* Implement **ModuleRegistry** to load `module.json`, register routes/widgets, and apply migrations on boot.
* Define **Zod schemas** for routine JSON, module manifest, and API payloads.
* Write **Drizzle** schema for DB tables; seed script for dev.
* Add **E1RM/volume calculators** in a shared util module; unit test them.
* Build “Today” with **slot injection** so modules can add tiles.
* Keep all paths and file names **stable** so the manifest references work.
* Provide a minimal **/admin** for critical ops early (users, backups).

---

This PRD is designed so you can stand up a lovable MVP quickly, then layer in modules like Lego bricks without changing the deploy story. Start with the kernel, ship it, and then let the app grow along with your training.
