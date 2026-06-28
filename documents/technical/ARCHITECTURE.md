# Technical Design

---

# ARCHITECTURE

## Purpose

Defines the high-level architecture and implementation principles of the project.

Daily Tool is built **inside the existing `gc-pm-automation` application**, not as a standalone product. See ADR-006 and `claude/INTEGRATION_AUDIT.md` for the full integration rationale.

---

# Core Principles

- Documentation is the source of truth.
- PostgreSQL is the source of truth — the Supabase project already backing `gc-pm-automation`.
- Redis stores runtime state only.
- Every feature starts as a manual workflow before automation.
- AI augments existing workflows, never defines them.
- The system prioritizes autonomy over centralization.
- Approval is required only for protected project data.
- Every significant architectural decision must have an ADR.
- Daily Tool reuses the host app's infrastructure (auth, hosting, database, layout) wherever it safely can, and isolates its own concerns behind a `daily_` naming prefix and a dedicated route segment.

---

# Architecture Style

Daily Tool is a module inside `gc-pm-automation`'s single Next.js fullstack monolith.

It does not have its own repository, its own deployment, or its own database.

Do not split Daily Tool into a separate frontend/backend service or a separate project.

---

# Host Application Integration

`gc-pm-automation` (Next.js 15, App Router, Supabase) is the host. Daily Tool is additive to it:

| Concern | Host app provides | Daily Tool reuses it by |
|---|---|---|
| Auth | Supabase Auth, Google OAuth, domain allowlist | No changes — Daily Tool requires no separate login |
| Identity | `profiles` table (1:1 with `auth.users`) | All `daily_*` tables reference `profiles.id`, no new users table |
| Roles | `user_role` enum + `profiles.role` | A new, additive, overridable `profiles.daily_role` column derives Developer/Manager/Admin (see Permission Model below); `user_role` itself is never modified |
| Layout | `app/ui/AppShell.tsx` sidebar shell | Daily Tool pages import and wrap themselves in `<AppShell>` exactly like existing pages (e.g. `app/templates/page.tsx`); one new nav entry is added |
| Hosting | Vercel project, Supabase project | Shared as-is, no new infra |
| Route gating | `middleware.ts` redirects unauthenticated → `/login`, not-onboarded → `/onboarding` for any path outside `/login`, `/auth/*`, `/api/*` | Applies to `/daily/*` automatically — no middleware changes needed |

---

# Technology Stack

## v1

- Next.js 15 (App Router) — the existing `gc-pm-automation` app
- TypeScript
- PostgreSQL via Supabase (existing project)
- `@supabase/supabase-js` + generated `lib/database.types.ts` — no ORM (see ADR-002 amendment)
- Auth: existing Supabase Auth / Google OAuth — no new auth code
- Next.js Server Actions, colocated as `actions.ts` per route (matches host convention)

## v1.5

Add:

- Redis
- Presence
- Edit Locks
- Pub/Sub
- Cache
- Rate Limiting
- Realtime Daily synchronization

## v2

Add:

- BullMQ
- GitHub Webhooks
- Background Workers

## v3

Add:

- AI Providers
- Transcript Processing
- AI Summaries
- Embeddings
- Semantic Search

---

# Project Structure

Matches `gc-pm-automation`'s actual flat layout (no `src/`, no layered `server/{services,repositories,permissions}` tree — that originally-planned structure is superseded, see ADR-006):

```text
app/
  daily/
    page.tsx              # redirects to /daily/today
    today/
      page.tsx
      actions.ts
    radar/
      page.tsx
    games/
      [id]/
        page.tsx
        actions.ts
  ui/
    AppShell.tsx           # existing — one new NAV entry added

lib/
  daily/
    auth.ts                # requireDailyProfile / requireDailyManager / requireDailyAdmin
    roles.ts                # daily_role derivation helpers (TS mirror of the SQL helper)
    games.ts                # daily_games + daily_game_stage_dates queries
    notes.ts                # daily_notes queries
    change-requests.ts       # daily_change_requests queries
  database.types.ts          # existing — regenerated after the Daily Tool migration, additive only
```

---

# Layers

## UI

Responsibilities

- Render UI
- Collect user input
- Display validation
- Call Server Actions

Must not

- Contain business logic
- Access database directly

---

## `lib/daily/*` (business logic + data access)

Responsibilities

- Business logic
- Permission checks (backed by Postgres RLS, double-checked in code where the host app's own `lib/auth.ts` pattern does the same)
- Database queries via the Supabase client
- Workflow orchestration

Examples

- Create Daily Note
- Approve Change Request
- Start Daily Session
- Change Stage

There is no separate repositories layer — `gc-pm-automation` doesn't have one either, and Daily Tool follows the host app's existing convention instead of introducing a new one.

---

# Permission Model

Permissions are role-based.

Responsibilities are assignment-based.

Daily session control is session-based.

These concepts must never be mixed.

## Roles (derived, not new accounts)

Every Daily Tool role is computed from the user's existing `gc-pm-automation` Profile — there are no separate Daily Tool accounts.

```text
profiles.daily_role override?
  yes -> use override (developer | manager | admin)
  no  -> derive from profiles.role:
           super_admin -> admin
           pm          -> manager
           (anything else: product_owner, backender, frontender, qa_qc) -> developer
```

### Developer

Can:

- Edit Daily Notes for assigned games.
- Link Pull Requests to assigned games.
- Create Change Requests.

Cannot:

- Edit protected project data directly.

### Manager

Includes all Developer permissions.

Additionally can:

- Edit protected project data.
- Approve Change Requests.
- Reject Change Requests.

Managers may also be assigned to games as developers.

### Admin

Includes all Manager permissions.

Additionally can:

- Manage users.
- Manage system settings.

Admins may also be assigned to games as developers.

Note: "manage users" here means Daily Tool–scoped settings (e.g. `daily_role` overrides), not `gc-pm-automation` user management, which remains under `gc-pm-automation`'s own `super_admin`/Settings page.

---

# Daily Host

Daily Host is not a role.

Daily Host is assigned automatically when a Profile starts a Daily session.

Any authenticated Profile may become Daily Host.

Daily Host may:

- Navigate Daily Mode.
- Move between games.
- Finish Daily.
- Edit Daily Notes during the session.

Daily Host does not receive Manager/Admin permissions, regardless of their derived `daily_role`.

---

# Assignment Model

A game has one primary assigned developer — a `profiles.id`.

Assignment determines ownership.

Derived `daily_role` determines permissions.

A Profile with derived role Manager or Admin may also be the assigned developer of a game; assignment and role are independent axes, exactly as in `gc-pm-automation`'s own `team_leads` concept (a narrower analog, scoped to templates only).

---

# Project Lifecycle

Every game has two independent properties.

## Status

Possible values:

- planned
- active
- paused
- cancelled
- completed

## Stage

Possible values:

- start_date
- playable
- alpha
- beta
- gold
- master
- eta_release_exclusive
- eta_release_com

Rules

- Planned games may not have a current stage.
- Active games must have a current stage.

---

# Screen Rules

## Today's Work

Displays assigned active games only.

## Simple Radar

Displays all games.

Supports expanding a game to view:

- Current stage
- Planned stage dates
- General project information

## Game Page

Displays:

- Full project information
- Stage dates
- Daily Notes
- Change Requests
- Pull Requests

## Daily Mode

Displays active games only.

All connected clients stay synchronized.

---

# Protected Project Data

Protected fields include:

- Game Title
- Client
- Assigned Developer
- Math Owner
- Current Stage
- Stage Dates

Developers cannot modify protected fields directly.

Protected changes require a Change Request.

Managers/Admins may edit protected fields directly.

---

# Data Ownership

## Persistent Data

Stored in PostgreSQL, in the shared Supabase project already backing `gc-pm-automation`, in `daily_`-prefixed tables.

Examples:

- Profiles (reused from `gc-pm-automation`, not owned by Daily Tool)
- daily_games
- daily_game_stage_dates
- daily_notes
- daily_change_requests
- daily_stage_history
- daily_pull_requests

## Runtime Data

Stored in Redis (v1.5+).

Examples:

- Active Daily Session
- Current Daily Game
- Presence
- Edit Locks
- Cache
- Pub/Sub Events

Redis is never the source of truth.

---

# Redis Responsibilities

Redis is introduced in v1.5.

Used for:

- Daily synchronization
- Presence
- Edit Locks
- Pub/Sub
- Cache
- Rate Limiting

Example keys:

```text
daily:session
daily:current_game
presence:game:{id}
lock:game:{id}
cache:radar
```

---

# BullMQ Responsibilities

BullMQ is introduced in v2.

Use BullMQ only for long-running or retryable jobs.

GitHub flow:

```text
Webhook
    ↓
 BullMQ
    ↓
Database
```

AI flow:

```text
Transcript
     ↓
  BullMQ
     ↓
 AI Summary
     ↓
Database
```

---

# GitHub Integration

GitHub starts in v2.

GitHub is a block inside Game Page.

Developers may manually link Pull Requests to games they are assigned to.

Managers/Admins may link Pull Requests to any game.

GitHub Webhooks are processed asynchronously using BullMQ.

---

# AI Integration

AI starts in v3.

Workflow:

```text
Manual Workflow
       ↓
Validated Workflow
       ↓
AI Augmentation
```

AI may provide:

- Transcript parsing
- Daily summaries
- Blocker extraction
- Semantic Search
- Suggested actions

The product must remain fully usable without AI.

---

# MVP Constraints

Do not implement before their roadmap stage:

- Redis
- BullMQ
- GitHub Webhooks
- AI
- Timeline
- Semantic Search
- Slack Integration
- Calendar Integration

Additionally, do not:

- Create a separate users/auth system.
- Modify `gc-pm-automation`'s `user_role` enum or its existing RLS policies.
- Introduce a new design system or layout shell.
- Introduce the originally-planned `server/{services,repositories,permissions}` layered structure — follow the host app's flat `lib/daily/*` + colocated `actions.ts` pattern instead.
