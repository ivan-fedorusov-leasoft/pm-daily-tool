# BE Radar — Full Specification

> **Status:** Re-targeting to gc-games-dashboard (ADR-007, 2026-06-30). Say "газ" to begin integration.
> **Built inside:** `gc-games-dashboard` repo — Supabase `hstvuhqqbhzsgkzvgntm`, NextAuth v5, raw Tailwind CSS.
> **ADR-007:** `documents/adr/007-gc-games-dashboard-integration.md` — replaces ADR-006.
> **Canonical detail:** screen specs in `visualization/`, vault notes in `vault/pm-daily-tool/`.

---

## 1. What Is BE Radar

An internal PM tool replacing an Excel-based status workflow. Developers write notes on their assigned games; managers review and approve/reject change requests; anyone can host a Daily meeting via Daily Mode. Eventually integrates GitHub (v2) and AI summaries (v3).

**Not:** a Jira replacement, a task tracker, a kanban board, a full PM suite.

---

## 2. App Name & Branding

| Item | Value |
|---|---|
| App display name | **BE Radar** |
| Logo icon | Radar/target SVG |
| Location | Inside `gc-games-dashboard`, under `/radar/*` routes |
| Nav | New `src/app/radar/layout.tsx` — shared nav for the BE Radar section |

`gc-games-dashboard` keeps its own name and branding. BE Radar is the *feature name* visible on the Radar screens.

---

## 3. Naming Conventions

### Table Prefix

All new Postgres tables use the **`radar_`** prefix to avoid collisions with `gc-games-dashboard`'s existing tables (`games`, `check_runs`, `status_events`, `scheduled_checks`, `starred_games`).

### Screen Names

| Screen | Route |
|---|---|
| Today's Work | `/radar/today` |
| Games | `/radar/games` |
| Game Page | `/radar/games/:id` |
| Daily Mode | `/radar/mode` |
| Change Requests | `/radar/change-requests` |

### DB Column/Check Values (authoritative)

```
role values (radar_users.role):   developer, manager, admin
radar_game_status:                planned, active, paused, cancelled, completed
radar_game_stage:                 start_date, playable, alpha, beta, gold, master,
                                  eta_release_exclusive, eta_release_com
radar_change_request_status:      pending, approved, rejected
radar_note_source:                manual, transcript
radar_session_status:             active, finished, cancelled
```

---

## 4. Design System

All styles come from `gc-games-dashboard/src/app/globals.css` — raw Tailwind CSS, no component library.

### Key tokens

| Token | Tailwind | Hex | Use |
|---|---|---|---|
| `--background` | `bg-background` | `#07090d` | Page bg |
| `--panel` | `bg-panel` | `#0d1117` | Card/panel bg |
| `--panel-2` | `bg-panel-2` | `#11161f` | Nested panel, input bg |
| `--border` | `border-border-soft` | `#1d2430` | Borders |
| `--foreground` | `text-foreground` | `#e6edf3` | Primary text |
| `--muted` | `text-muted` | `#8b97a8` | Secondary text, labels |

### `.panel` utility
```css
.panel { background: var(--panel); border: 1px solid var(--border); }
```
Always pair with `rounded-xl` or `rounded-lg`.

### Status colors (raw Tailwind)

| Status / Tone | Text | Dot | Badge bg |
|---|---|---|---|
| ok / active / approved | `text-emerald-300` | `bg-emerald-400` | `bg-emerald-500/10` |
| warn / pending | `text-amber-300` | `bg-amber-400` | `bg-amber-500/10` |
| error / rejected | `text-rose-300` | `bg-rose-400` | `bg-rose-500/10` |
| info / planned | `text-violet-300` | `bg-violet-400` | `bg-violet-500/10` |
| muted / default | `text-slate-400` | `bg-slate-500` | `bg-slate-500/10` |

See full details: [visualization/design-system.md](visualization/design-system.md)

---

## 5. Routes & Pages

All routes under `/radar/*`. A shared `src/app/radar/layout.tsx` provides the BE Radar nav.

| Route | File | Guard | Purpose |
|---|---|---|---|
| `/radar` | `src/app/radar/page.tsx` | `requireRadarProfile()` | Redirect → `/radar/today` |
| `/radar/today` | `src/app/radar/today/page.tsx` + `actions.ts` | `requireRadarProfile()` | Today's Work |
| `/radar/games` | `src/app/radar/games/page.tsx` | `requireRadarProfile()` | Games list |
| `/radar/games/:id` | `src/app/radar/games/[id]/page.tsx` + `actions.ts` | `requireRadarProfile()` | Game Page |
| `/radar/change-requests` | `src/app/radar/change-requests/page.tsx` + `actions.ts` | `requireRadarProfile()` | All change requests |
| `/radar/mode` | `src/app/radar/mode/page.tsx` | `requireRadarProfile()` | Daily Mode (v1.5) |

---

## 6. Screen Specs (detail files)

| Screen | Spec file |
|---|---|
| Today's Work | [today-work-spec.md](visualization/today%20work/today-work-spec.md) |
| Write Note Modal | [write-note-modal-spec.md](visualization/today%20work%20-%20add%20note/write-note-modal-spec.md) |
| Games (list) | [games-list-spec.md](visualization/games/games-list-spec.md) |
| Game Page | [game-page-spec.md](visualization/one%20game%20card/game-page-spec.md) |
| Change Requests | [change-requests-spec.md](visualization/change%20request/change-requests-spec.md) |
| Daily Mode | [daily-mode-spec.md](visualization/daily%20mode/daily-mode-spec.md) |

---

## 7. Database Schema

Runs in **gc-games-dashboard's Supabase project** (`hstvuhqqbhzsgkzvgntm`). Service role key only — no Supabase Auth, no RLS enforcement (service role bypasses). Access control is in Server Actions.

### Migration file: `supabase/migrations/0003_radar.sql`

No Postgres enums — text columns with CHECK constraints (easier to extend without migrations).

```sql
-- User profiles for BE Radar (keyed by NextAuth email)
CREATE TABLE IF NOT EXISTS public.radar_users (
  email        text PRIMARY KEY,
  name         text NOT NULL,
  avatar_url   text,
  role         text NOT NULL DEFAULT 'developer'
                 CHECK (role IN ('developer', 'manager', 'admin')),
  created_at   timestamptz NOT NULL DEFAULT now(),
  updated_at   timestamptz NOT NULL DEFAULT now()
);

-- Math contacts (not app users — no login)
CREATE TABLE IF NOT EXISTS public.radar_math_owners (
  id         uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name       text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- Main game/project records
CREATE TABLE IF NOT EXISTS public.radar_games (
  id              uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  title           text NOT NULL,
  client          text,
  developer_email text NOT NULL REFERENCES public.radar_users(email),
  math_owner_id   uuid REFERENCES public.radar_math_owners(id),
  current_stage   text CHECK (current_stage IN (
                    'start_date','playable','alpha','beta','gold',
                    'master','eta_release_exclusive','eta_release_com'
                  )),
  status          text NOT NULL DEFAULT 'planned'
                    CHECK (status IN ('planned','active','paused','cancelled','completed')),
  created_at      timestamptz NOT NULL DEFAULT now(),
  updated_at      timestamptz NOT NULL DEFAULT now()
);
-- Protected fields (manager/admin only direct edit):
-- title, client, developer_email, math_owner_id, current_stage, status

-- Stage target dates
CREATE TABLE IF NOT EXISTS public.radar_game_stage_dates (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  game_id     uuid NOT NULL REFERENCES public.radar_games(id) ON DELETE CASCADE,
  stage       text NOT NULL CHECK (stage IN (
                'start_date','playable','alpha','beta','gold',
                'master','eta_release_exclusive','eta_release_com'
              )),
  target_date date,
  created_at  timestamptz NOT NULL DEFAULT now(),
  updated_at  timestamptz NOT NULL DEFAULT now(),
  UNIQUE (game_id, stage)
);

-- Progress notes
CREATE TABLE IF NOT EXISTS public.radar_notes (
  id                   uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  game_id              uuid NOT NULL REFERENCES public.radar_games(id) ON DELETE CASCADE,
  author_email         text NOT NULL REFERENCES public.radar_users(email),
  date                 date NOT NULL,
  summary              text NOT NULL,
  needs_lead_attention boolean NOT NULL DEFAULT false,
  source               text NOT NULL DEFAULT 'manual'
                         CHECK (source IN ('manual','transcript')),
  created_at           timestamptz NOT NULL DEFAULT now()
);

-- Proposed field changes (require approval)
CREATE TABLE IF NOT EXISTS public.radar_change_requests (
  id             uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  game_id        uuid NOT NULL REFERENCES public.radar_games(id) ON DELETE CASCADE,
  requested_by   text NOT NULL REFERENCES public.radar_users(email),
  field_name     text NOT NULL,
  old_value      text,
  new_value      text NOT NULL,
  status         text NOT NULL DEFAULT 'pending'
                   CHECK (status IN ('pending','approved','rejected')),
  reviewed_by    text REFERENCES public.radar_users(email),
  reviewed_at    timestamptz,
  created_at     timestamptz NOT NULL DEFAULT now(),
  updated_at     timestamptz NOT NULL DEFAULT now()
);

-- Stage transition audit trail
CREATE TABLE IF NOT EXISTS public.radar_stage_history (
  id           uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  game_id      uuid NOT NULL REFERENCES public.radar_games(id) ON DELETE CASCADE,
  from_stage   text,
  to_stage     text NOT NULL,
  changed_by   text NOT NULL REFERENCES public.radar_users(email),
  created_at   timestamptz NOT NULL DEFAULT now()
);

-- Enable RLS (service role bypasses, anon gets nothing)
ALTER TABLE public.radar_users          ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.radar_math_owners    ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.radar_games          ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.radar_game_stage_dates ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.radar_notes          ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.radar_change_requests ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.radar_stage_history  ENABLE ROW LEVEL SECURITY;
```

#### v1.5 — Daily Mode tables

```sql
CREATE TABLE IF NOT EXISTS public.radar_sessions (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  host_email  text NOT NULL REFERENCES public.radar_users(email),
  date        date NOT NULL,
  status      text NOT NULL DEFAULT 'active'
                CHECK (status IN ('active','finished','cancelled')),
  started_at  timestamptz NOT NULL DEFAULT now(),
  finished_at timestamptz,
  created_at  timestamptz NOT NULL DEFAULT now(),
  updated_at  timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE IF NOT EXISTS public.radar_audit_logs (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_email  text NOT NULL REFERENCES public.radar_users(email),
  entity_type text NOT NULL,
  entity_id   uuid NOT NULL,
  action      text NOT NULL,
  before      jsonb,
  after       jsonb,
  created_at  timestamptz NOT NULL DEFAULT now()
);

ALTER TABLE public.radar_sessions   ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.radar_audit_logs ENABLE ROW LEVEL SECURITY;
```

#### v2 — GitHub

```sql
CREATE TABLE IF NOT EXISTS public.radar_pull_requests (
  id         uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  game_id    uuid NOT NULL REFERENCES public.radar_games(id) ON DELETE CASCADE,
  stage      text,
  github_url text NOT NULL,
  repo       text,
  pr_number  integer,
  title      text,
  status     text,
  author     text,
  merged_at  timestamptz,
  summary    text,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

ALTER TABLE public.radar_pull_requests ENABLE ROW LEVEL SECURITY;
```

---

## 8. Permissions Model

Three axes — **never conflate**:

| Axis | Source | Examples |
|---|---|---|
| **Role** | `radar_users.role` | developer / manager / admin |
| **Assignment** | `radar_games.developer_email = currentUserEmail` | owns this specific game |
| **Session** | Daily Host — whoever called "Start Daily" in Daily Mode | controls meeting navigation |

### What each role can do

| Action | Developer | Manager | Admin |
|---|---|---|---|
| Write Note (own games) | ✓ | ✓ | ✓ |
| Write Note (any game) | ✗ | ✓ | ✓ |
| Create Change Request | ✓ | ✓ | ✓ |
| Approve / Reject Change Request | ✗ | ✓ | ✓ |
| Edit protected game fields directly | ✗ | ✓ | ✓ |
| Link PR to own game | ✓ | ✓ | ✓ |
| Link PR to any game | ✗ | ✓ | ✓ |
| Create / edit games | ✗ | ✓ | ✓ |
| Manage users / roles | ✗ | ✗ | ✓ |
| Start Daily Mode session | ✓ (any) | ✓ | ✓ |
| Control Daily navigation | Daily Host only | Daily Host only | Daily Host only |

**Daily Host** gets **zero** permission elevation — navigation control only.

### Protected game fields

Direct edit requires `isRadarManager()`. Developers submit Change Requests instead:
- `title`, `client`, `developer_email`, `math_owner_id`, `current_stage`, `status`

---

## 9. Code Structure

`gc-games-dashboard` uses `src/`. All BE Radar files are additive:

```
src/
  app/
    radar/
      layout.tsx                    # Shared BE Radar nav
      page.tsx                      # redirect → /radar/today
      today/
        page.tsx                    # Server Component: requireRadarProfile, fetch, render
        TodayWorkClient.tsx
        actions.ts                  # saveNote, flagAttention
      games/
        page.tsx                    # Games list
        [id]/
          page.tsx                  # Game Page
          GameDetailClient.tsx
          NoteButtonClient.tsx
          actions.ts                # suggestChange, editStageDates, addNote
      change-requests/
        page.tsx                    # CR list + approve/reject
        CrActionsClient.tsx
        actions.ts
      mode/
        page.tsx                    # Daily Mode (v1.5)
        DailyModeClient.tsx
    api/radar/
      seed/route.ts                 # POST — seed demo data

  lib/radar/
    types.ts                        # All TS types, stage/status labels
    auth.ts                         # requireRadarProfile, requireRadarManager, requireRadarAdmin
    roles.ts                        # getRadarRole(), isRadarManager(), isRadarAdmin()
    users.ts                        # getCurrentRadarUser() — upserts radar_users on sign-in
    games.ts                        # getGames, getGame, getActiveGames, createGame
    notes.ts                        # getNotes, createNote
    change-requests.ts              # getCRs, approveCR, rejectCR

supabase/migrations/
  0003_radar.sql                    # radar_users + all radar_* tables
```

---

## 10. v1 Implementation Checklist

> Say "газ" to begin. This is the reference list.

### Database
- [ ] Migration `0003_radar.sql`: `radar_users` + all v1 tables + RLS enable
- [ ] Apply to Supabase `hstvuhqqbhzsgkzvgntm`
- [ ] Seed test data (2-3 games, notes, CRs)

### Core library (`src/lib/radar/`)
- [ ] `auth.ts` — `requireRadarProfile`, `requireRadarManager`, `requireRadarAdmin`
- [ ] `roles.ts` — `getRadarRole()`, `isRadarManager()`, `isRadarAdmin()`
- [ ] `users.ts` — `getCurrentRadarUser()` (upsert on sign-in)
- [ ] `games.ts` — CRUD for `radar_games`, `radar_game_stage_dates`
- [ ] `notes.ts` — create/list notes
- [ ] `change-requests.ts` — create/list/approve/reject

### Layout
- [ ] `src/app/radar/layout.tsx` — shared BE Radar nav (links to Today, Games, Change Requests, Daily Mode)

### Screens (v1)
- [ ] `/radar/today` — Today's Work
- [ ] Write Note modal
- [ ] `/radar/games` — Games list
- [ ] `/radar/games/:id` — Game Page (without PR panel — v2)
- [ ] `/radar/change-requests` — CR list + approve/reject actions

### v1.5 (Daily Mode — separate milestone)
- [ ] Redis setup
- [ ] `radar_sessions` + `radar_audit_logs` migration
- [ ] `/radar/mode` — Daily Mode client component (real-time)

### v2 (GitHub — separate milestone)
- [ ] `radar_pull_requests` migration
- [ ] GitHub webhook handler
- [ ] PR block on Game Page

---

## 11. Open Questions

- [ ] **Approved CR → auto-apply:** when a CR is approved, system automatically updates `radar_games` field in same Server Action (prefer yes — simpler UX)
- [ ] **Games screen visibility:** all games for all users, or only games user is assigned to (for Developer role)?
- [ ] **"+ New Game" access:** manager/admin only?
- [ ] **Daily Mode v1 scope:** is a non-real-time simplified Daily Mode acceptable for v1, or is Redis required from day one?
- [ ] **PR panel on Game Page:** hide entirely until v2 or show empty state?
- [ ] **Notes:** one per day per game, or unlimited?
- [ ] **Math Owner:** is `radar_math_owners` a simple name-only table or should it eventually link to `radar_users`?
