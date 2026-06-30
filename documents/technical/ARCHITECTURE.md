# Technical Design

---

# ARCHITECTURE

## Purpose

Defines the high-level architecture and implementation principles of the project.

> **ADR-007 (2026-06-30):** Integration target changed. Daily Tool is built **inside the existing `gc-games-dashboard` application**, not `gc-pm-automation` as planned in ADR-006. See `documents/adr/007-gc-games-dashboard-integration.md`.

---

# Core Principles

- Documentation is the source of truth.
- PostgreSQL is the source of truth — the Supabase project backing `gc-games-dashboard` (`hstvuhqqbhzsgkzvgntm`).
- Redis stores runtime state only.
- Every feature starts as a manual workflow before automation.
- AI augments existing workflows, never defines them.
- The system prioritizes autonomy over centralization.
- Approval is required only for protected project data.
- Every significant architectural decision must have an ADR.
- Daily Tool reuses the host app's infrastructure (auth, hosting, database) wherever it safely can, and isolates its own concerns behind a `daily_` naming prefix and a dedicated route segment.

---

# Architecture Style

Daily Tool is a module inside `gc-games-dashboard`'s single Next.js fullstack monolith.

It does not have its own repository, its own deployment, or its own database.

Do not split Daily Tool into a separate frontend/backend service or a separate project.

---

# Host Application Integration

`gc-games-dashboard` (Next.js 16, App Router, Supabase service role, NextAuth v5) is the host. Daily Tool is additive to it:

| Concern | Host app provides | Daily Tool reuses it by |
|---|---|---|
| Auth | NextAuth v5, Google OAuth, `AUTH_ALLOWED_DOMAIN` gate | No changes — Daily Tool requires no separate login |
| Identity | `getCurrentUserKey()` → email or `"local"` | New `daily_users` table keyed by email; upserted on sign-in |
| Roles | None (no role concept in gc-games-dashboard) | New `daily_users.role` column: developer / manager / admin |
| Layout | No shared layout currently | New `src/app/daily/layout.tsx` — shared nav for BE Radar section |
| DB access | Service role key, `getSupabase()` singleton | Same pattern — `supabase.from('daily_*').select(...)` |
| Hosting | Vercel + Supabase `hstvuhqqbhzsgkzvgntm` | Shared as-is, no new infra |
| Route gating | NextAuth middleware — unauthenticated → `/login` | Applies to `/daily/*` automatically if auth configured |

---

# Technology Stack

## v1

- Next.js 16 (App Router) — the existing `gc-games-dashboard` app
- TypeScript, React 19
- PostgreSQL via Supabase service role (project `hstvuhqqbhzsgkzvgntm`)
- `@supabase/supabase-js` — no ORM, raw `.from().select()` calls
- Auth: NextAuth v5 with Google provider — no new auth code
- Next.js Server Actions, colocated as `actions.ts` per route
- Types: defined locally in `src/lib/daily/types.ts` — no generated `database.types.ts` needed

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

`gc-games-dashboard` uses a `src/` directory. Daily Tool is additive inside it:

```text
src/
  app/
    daily/
      layout.tsx            # NEW — shared BE Radar nav (top bar or sidebar)
      page.tsx              # redirects to /daily/today
      today/
        page.tsx
        TodayWorkClient.tsx
        actions.ts
      games/
        page.tsx
        [id]/
          page.tsx
          GameDetailClient.tsx
          NoteButtonClient.tsx
          actions.ts
      change-requests/
        page.tsx
        CrActionsClient.tsx
        actions.ts
    api/daily/
      seed/route.ts         # POST — seed demo data

  lib/daily/
    types.ts                # All TS types, enums, labels, stageState() helper
    auth.ts                 # requireDailyProfile / requireDailyManager / requireDailyAdmin
    roles.ts                # getDailyRole(), isDailyManager(), isDailyAdmin()
    users.ts                # getCurrentDailyUser() — upserts daily_users on sign-in
    games.ts                # daily_games + daily_game_stage_dates queries
    notes.ts                # daily_notes queries
    change-requests.ts      # daily_change_requests queries

supabase/migrations/
  0003_daily_tool.sql       # daily_users + all daily_* tables (next after 0002)
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

## Roles (stored in `daily_users`)

Every Daily Tool role is stored in `daily_users.role`. There are no separate Daily Tool accounts — identity comes from the NextAuth Google session.

```text
daily_users.role:
  developer  (default for all new sign-ins)
  manager
  admin
```

Role is set by admin directly in the DB. On first sign-in, user is upserted into `daily_users` with `role = 'developer'`.

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

Stored in PostgreSQL, in the Supabase project backing `gc-games-dashboard` (`hstvuhqqbhzsgkzvgntm`), in `daily_`-prefixed tables.

Tables:

- daily_users (owned by Daily Tool — new)
- daily_games
- daily_game_stage_dates
- daily_notes
- daily_change_requests
- daily_stage_history
- daily_pull_requests (v2)

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

- Create a separate users/auth system — reuse NextAuth session from gc-games-dashboard.
- Modify gc-games-dashboard's existing tables (`games`, `check_runs`, etc.) or their RLS.
- Introduce a second design system — follow gc-games-dashboard's raw Tailwind pattern (no primitives.tsx).
- Use Supabase Auth — gc-games-dashboard uses NextAuth (service role key only, no anon key).
- Introduce the originally-planned `server/{services,repositories,permissions}` layered structure — use flat `lib/daily/*` + colocated `actions.ts` instead.
- Add RLS helper functions — service role bypasses RLS; access control is enforced in Server Actions.
