# BE Radar — Full Specification

> **Status:** Ready for implementation. DB + backend work on owner's signal.  
> **Built inside:** `gc-pm-automation` repo — same Supabase project, same Vercel deployment, same design system.  
> **Canonical detail:** individual screen specs in `visualization/`, vault notes in `vault/pm-daily-tool/`.

---

## 1. What Is BE Radar

An internal PM tool replacing an Excel-based daily status workflow. Developers write daily notes on their assigned games; managers review and approve/reject change requests; anyone can host a Daily meeting. Eventually integrates GitHub (v2) and AI summaries (v3).

**Not:** a Jira replacement, a task tracker, a kanban board, a full PM suite.

---

## 2. App Name & Branding

| Item | Value |
|---|---|
| App display name | **BE Radar** |
| Logo icon | Radar/target SVG (replacing current PM Automation 🎛️ for the Daily section, or sidebar sub-label) |
| Location | Inside `gc-pm-automation`, under `/daily/*` routes |
| Sidebar entry | "Daily" nav item already added to `AppShell.tsx` `NAV` |

The host app (`gc-pm-automation`) keeps its own name and branding. BE Radar is the *feature name* visible on the Daily screens.

---

## 3. Naming Conventions

### Table / Enum Prefix

All new Postgres tables and enums use the **`daily_`** prefix.

**Rationale:** Describes the feature (Daily meeting workflow), not the app name. Semantic, stable, already decided in ADR-006. `ber_` (BE Radar abbreviation) is an alternative but would lose meaning if the app is ever renamed again. **Decision: keep `daily_`.**

### Screen Names (updated)

| Screen | Old name | New name | Route |
|---|---|---|---|
| Daily landing | — | Today's Work | `/daily/today` |
| Games overview | Simple Radar | **Games** | `/daily/games` |
| Individual game | Game Page | Game Page | `/daily/games/:id` |
| Meeting mode | Daily Mode | Daily Mode | `/daily/mode` |
| Change requests | Change Requests | Change Requests | `/daily/change-requests` |

### DB Enum Values (authoritative)

```
daily_game_status:        planned, active, paused, cancelled, completed
daily_game_stage:         start_date, playable, alpha, beta, gold, master,
                          eta_release_exclusive, eta_release_com
daily_change_request_status: pending, approved, rejected
daily_note_source:        manual, transcript
daily_session_status:     active, finished, cancelled
daily_role:               developer, manager, admin
```

---

## 4. Design System

All styles come from `gc-pm-automation/app/globals.css` and `primitives.tsx`. **No new Tailwind theme, no new design tokens, no component library.**

### Key tokens

| Token | Hex | Use |
|---|---|---|
| `--color-bg` | `#07080c` | Page bg |
| `--color-surface` | `#0d0f17` | Card bg |
| `--color-surface-2` | `#131725` | Card section, nav active, inputs |
| `--color-border` | `#1f2435` | All borders |
| `--color-border-strong` | `#2c3350` | Hover/active borders |
| `--color-text` | `#e7e9f3` | Primary text |
| `--color-muted` | `#8a90a8` | Labels, secondary text |
| `--color-faint` | `#565d78` | Timestamps, hints |
| `--color-accent` | `#6366f1` | Links, buttons, active state |
| `--color-ok` | `#34d399` | Active, approved, merged, success |
| `--color-warn` | `#fbbf24` | Pending, team-lead flag |
| `--color-err` | `#f87171` | Declined, rejected, error |
| `--radius-card` | `16px` | Card border-radius |

### Reused components

- `<Card>` — all panels and cards
- `<Button variant="primary|ghost|subtle|danger">` — all interactive buttons
- `<Badge tone="default|accent|ok|warn">` — status pills (add custom `err` class for Declined)
- `<PageTitle>` — `text-2xl font-semibold tracking-tight`
- `<EmptyState>` — empty list states
- `<AppShell>` — sidebar + main layout wrapper on every page

### Badge → Tone mapping

| Status | Tone |
|---|---|
| Active | `ok` |
| Planned | `accent` |
| In Progress | `warn` |
| On Hold | custom err |
| Pending CR | `warn` |
| Approved CR | `ok` |
| Declined/Rejected CR | custom err |
| In Review PR | `accent` |
| Merged PR | `ok` |

See full design details: [visualization/design-system.md](visualization/design-system.md)

---

## 5. Routes & Pages

| Route | File | Guard | Purpose |
|---|---|---|---|
| `/daily` | `app/daily/page.tsx` | `requireProfile()` | Redirect → `/daily/today` |
| `/daily/today` | `app/daily/today/page.tsx` + `actions.ts` | `requireProfile()` | Today's Work — write daily notes |
| `/daily/games` | `app/daily/games/page.tsx` | `requireProfile()` | Games list (was Radar) |
| `/daily/games/:id` | `app/daily/games/[id]/page.tsx` + `actions.ts` | `requireProfile()` | Individual game page |
| `/daily/change-requests` | `app/daily/change-requests/page.tsx` + `actions.ts` | `requireProfile()` | All change requests |
| `/daily/mode` | `app/daily/mode/page.tsx` | `requireProfile()` | Daily Mode (v1.5) |

**No new middleware changes** — existing `middleware.ts` applies `/daily/*` auth gate automatically.

---

## 6. Screen Specs (detail files)

| Screen | Spec file | Folder |
|---|---|---|
| Today's Work | [today-work-spec.md](visualization/today%20work/today-work-spec.md) | `visualization/today work/` |
| Write Note Modal | [write-note-modal-spec.md](visualization/today%20work%20-%20add%20note/write-note-modal-spec.md) | `visualization/today work - add note/` |
| Games (list) | [games-list-spec.md](visualization/games/games-list-spec.md) | `visualization/games/` |
| Game Page | [game-page-spec.md](visualization/one%20game%20card/game-page-spec.md) | `visualization/one game card/` |
| Change Requests | [change-requests-spec.md](visualization/change%20request/change-requests-spec.md) | `visualization/change request/` |
| Daily Mode | [daily-mode-spec.md](visualization/daily%20mode/daily-mode-spec.md) | `visualization/daily mode/` |

---

## 7. Database Schema

Runs in the **same Supabase project** as gc-pm-automation. No new Supabase project, no new auth.

### New migration file: `0005_daily_tool.sql`

#### New enums

```sql
CREATE TYPE daily_role AS ENUM ('developer', 'manager', 'admin');
CREATE TYPE daily_game_status AS ENUM ('planned', 'active', 'paused', 'cancelled', 'completed');
CREATE TYPE daily_game_stage AS ENUM (
  'start_date', 'playable', 'alpha', 'beta', 'gold',
  'master', 'eta_release_exclusive', 'eta_release_com'
);
CREATE TYPE daily_change_request_status AS ENUM ('pending', 'approved', 'rejected');
CREATE TYPE daily_note_source AS ENUM ('manual', 'transcript');
```

#### Additive column on existing `profiles` table

```sql
ALTER TABLE profiles ADD COLUMN daily_role daily_role;
-- nullable — NULL means "derive from profiles.role"
-- super_admin → admin, pm → manager, everything else → developer
```

#### New tables (v1)

```sql
-- Math contacts (not app users — no login)
CREATE TABLE daily_math_owners (
  id         uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name       varchar NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- Main game/project records
CREATE TABLE daily_games (
  id             uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  title          varchar NOT NULL,
  client         varchar,
  developer_id   uuid NOT NULL REFERENCES profiles(id),
  math_owner_id  uuid REFERENCES daily_math_owners(id),
  current_stage  daily_game_stage,
  status         daily_game_status NOT NULL DEFAULT 'planned',
  created_at     timestamptz NOT NULL DEFAULT now(),
  updated_at     timestamptz NOT NULL DEFAULT now()
);
-- Protected fields (manager/admin only direct edit):
-- title, client, developer_id, math_owner_id, current_stage, status

-- Stage target dates
CREATE TABLE daily_game_stage_dates (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  game_id     uuid NOT NULL REFERENCES daily_games(id) ON DELETE CASCADE,
  stage       daily_game_stage NOT NULL,
  target_date date,
  created_at  timestamptz NOT NULL DEFAULT now(),
  updated_at  timestamptz NOT NULL DEFAULT now(),
  UNIQUE (game_id, stage)
);

-- Daily progress notes
CREATE TABLE daily_notes (
  id                   uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  game_id              uuid NOT NULL REFERENCES daily_games(id) ON DELETE CASCADE,
  author_id            uuid NOT NULL REFERENCES profiles(id),
  date                 date NOT NULL,
  summary              text NOT NULL,
  needs_lead_attention boolean NOT NULL DEFAULT false,
  source               daily_note_source NOT NULL DEFAULT 'manual',
  created_at           timestamptz NOT NULL DEFAULT now(),
  updated_at           timestamptz NOT NULL DEFAULT now()
);

-- Proposed field changes (require approval)
CREATE TABLE daily_change_requests (
  id           uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  game_id      uuid NOT NULL REFERENCES daily_games(id) ON DELETE CASCADE,
  requested_by uuid NOT NULL REFERENCES profiles(id),
  field_name   varchar NOT NULL,
  old_value    text,
  new_value    text NOT NULL,
  status       daily_change_request_status NOT NULL DEFAULT 'pending',
  reviewed_by  uuid REFERENCES profiles(id),
  reviewed_at  timestamptz,
  created_at   timestamptz NOT NULL DEFAULT now(),
  updated_at   timestamptz NOT NULL DEFAULT now()
);

-- Stage transition audit trail
CREATE TABLE daily_stage_history (
  id         uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  game_id    uuid NOT NULL REFERENCES daily_games(id) ON DELETE CASCADE,
  from_stage daily_game_stage,
  to_stage   daily_game_stage NOT NULL,
  changed_by uuid NOT NULL REFERENCES profiles(id),
  created_at timestamptz NOT NULL DEFAULT now()
);
```

#### New tables (v1.5 — Daily Mode)

```sql
CREATE TYPE daily_session_status AS ENUM ('active', 'finished', 'cancelled');

CREATE TABLE daily_sessions (
  id           uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  host_user_id uuid NOT NULL REFERENCES profiles(id),
  date         date NOT NULL,
  status       daily_session_status NOT NULL DEFAULT 'active',
  started_at   timestamptz NOT NULL DEFAULT now(),
  finished_at  timestamptz,
  created_at   timestamptz NOT NULL DEFAULT now(),
  updated_at   timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE daily_audit_logs (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     uuid NOT NULL REFERENCES profiles(id),
  entity_type varchar NOT NULL,
  entity_id   uuid NOT NULL,
  action      varchar NOT NULL,
  before      jsonb,
  after       jsonb,
  created_at  timestamptz NOT NULL DEFAULT now()
);
```

#### New tables (v2 — GitHub)

```sql
CREATE TABLE daily_pull_requests (
  id         uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  game_id    uuid NOT NULL REFERENCES daily_games(id) ON DELETE CASCADE,
  stage      daily_game_stage,
  github_url text NOT NULL,
  repo       varchar,
  pr_number  integer,
  title      varchar,
  status     varchar,
  author     varchar,
  merged_at  timestamptz,
  summary    text,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);
```

#### RLS helper functions (`private` schema — same pattern as existing)

```sql
-- Derives daily_role for current user
CREATE OR REPLACE FUNCTION private.daily_role()
RETURNS daily_role LANGUAGE sql SECURITY DEFINER SET search_path = ''
AS $$
  SELECT COALESCE(
    p.daily_role,
    CASE p.role
      WHEN 'super_admin' THEN 'admin'::daily_role
      WHEN 'pm'          THEN 'manager'::daily_role
      ELSE                    'developer'::daily_role
    END
  )
  FROM public.profiles p WHERE p.id = auth.uid()
$$;
GRANT EXECUTE ON FUNCTION private.daily_role() TO authenticated;

CREATE OR REPLACE FUNCTION private.is_daily_manager()
RETURNS boolean LANGUAGE sql SECURITY DEFINER SET search_path = ''
AS $$ SELECT private.daily_role() IN ('admin', 'manager') $$;
GRANT EXECUTE ON FUNCTION private.is_daily_manager() TO authenticated;

CREATE OR REPLACE FUNCTION private.is_daily_admin()
RETURNS boolean LANGUAGE sql SECURITY DEFINER SET search_path = ''
AS $$ SELECT private.daily_role() = 'admin' $$;
GRANT EXECUTE ON FUNCTION private.is_daily_admin() TO authenticated;
```

---

## 8. Permissions Model

Three axes — **never conflate**:

| Axis | Source | Examples |
|---|---|---|
| **Role** | `private.daily_role()` — derived from `profiles.role` or override | developer / manager / admin |
| **Assignment** | `daily_games.developer_id = auth.uid()` | owns this specific game |
| **Session** | Daily Host — whoever called "Start Daily" | controls Daily Mode navigation |

### What each role can do

| Action | Developer | Manager | Admin |
|---|---|---|---|
| Write Daily Note (own games) | ✓ | ✓ | ✓ |
| Write Daily Note (any game) | ✗ | ✓ | ✓ |
| Create Change Request | ✓ | ✓ | ✓ |
| Approve / Reject Change Request | ✗ | ✓ | ✓ |
| Edit protected game fields directly | ✗ | ✓ | ✓ |
| Link PR to own game | ✓ | ✓ | ✓ |
| Link PR to any game | ✗ | ✓ | ✓ |
| Create / edit games | ✗ | ✓ | ✓ |
| Manage users / daily_role overrides | ✗ | ✗ | ✓ |
| Start Daily Mode session | ✓ (any) | ✓ | ✓ |
| Control Daily navigation | Host only | Host only | Host only |

**Daily Host** gets **zero** permission elevation — navigation control only.

### Protected game fields

Direct edit requires `is_daily_manager()`. Developers submit Change Requests instead:
- `title`, `client`, `developer_id`, `math_owner_id`, `current_stage`, `status`

---

## 9. Code Structure

Follows gc-pm-automation's **flat convention** (no `server/services/repositories/` layering):

```
app/
  daily/
    page.tsx                      # redirect → /daily/today
    today/
      page.tsx                    # Server Component: requireDailyProfile, fetch, render
      actions.ts                  # Server Actions: saveNote, flagAttention
    games/
      page.tsx                    # Games list (was radar)
      [id]/
        page.tsx                  # Game Page
        actions.ts                # suggestChange, editStageDates, addNote
    change-requests/
      page.tsx                    # CR list + approve/reject
      actions.ts
    mode/
      page.tsx                    # Daily Mode (v1.5)
      DailyModeClient.tsx         # client component for live state

lib/
  daily/
    auth.ts                       # requireDailyProfile, requireDailyManager, requireDailyAdmin
    roles.ts                      # daily_role derivation (TS mirror of SQL helper)
    games.ts                      # getGames, getGame, getActiveGames
    notes.ts                      # getNotes, createNote
    change-requests.ts            # getCRs, approveCR, rejectCR
```

All `daily_*` Server Actions follow the same `actions.ts` colocation pattern as existing gc-pm-automation pages.

---

## 10. v1 Implementation Checklist

> **Do not start until owner signals go.** This is the reference list.

### Database
- [ ] Migration `0005_daily_tool.sql`: enums, `profiles.daily_role` column, v1 tables, RLS helpers
- [ ] RLS policies on all new tables
- [ ] Regenerate `lib/database.types.ts`
- [ ] Seed test data (2-3 games, notes, CRs)

### Core library
- [ ] `lib/daily/auth.ts` — `requireDailyProfile`, `requireDailyManager`, `requireDailyAdmin`
- [ ] `lib/daily/roles.ts` — `getDailyRole(profile)` TS function
- [ ] `lib/daily/games.ts` — CRUD for `daily_games`, `daily_game_stage_dates`
- [ ] `lib/daily/notes.ts` — create/list notes
- [ ] `lib/daily/change-requests.ts` — create/list/approve/reject

### AppShell nav update
- [ ] Update `app/ui/AppShell.tsx` — nav item label/icon for Daily entry; consider sub-nav for Daily screens

### Screens (v1)
- [ ] `/daily/today` — Today's Work page
- [ ] Write Note modal component
- [ ] `/daily/games` — Games list
- [ ] `/daily/games/:id` — Game Page (without PR panel — v2)
- [ ] `/daily/change-requests` — CR list + approve/reject actions

### v1.5 (Daily Mode — separate milestone)
- [ ] Redis setup
- [ ] `daily_sessions` + `daily_audit_logs` migration
- [ ] `/daily/mode` — Daily Mode client component (real-time)

### v2 (GitHub — separate milestone)
- [ ] `daily_pull_requests` migration
- [ ] GitHub webhook handler
- [ ] PR block on Game Page

---

## 11. Open Questions (Pre-Implementation)

- [ ] **Table prefix final decision:** keep `daily_` or switch to `ber_`? → Recommend `daily_`
- [ ] **AppShell sidebar sub-nav:** does "Daily" expand to sub-items (Today, Games, Change Requests) or stay as a top-level entry that lands on `/daily/today`?
- [ ] **Change Request auto-apply:** when a CR is approved, does the system automatically update `daily_games` field, or does manager manually apply?
- [ ] **Approved CR → auto-apply:** prefer yes (simpler UX) — `approved` action runs UPDATE in same Server Action
- [ ] **Games screen visibility:** all games for all users, or only games user is assigned to (for Developer role)?
- [ ] **"+ New Game" access:** manager/admin only, or any user can create a game (developer would own it)?
- [ ] **Daily Mode v1 scope:** is a non-real-time simplified Daily Mode acceptable for v1, or is Redis required from day one?
- [ ] **PR panel on Game Page:** hide entirely until v2 or show empty state with "Coming soon"?
- [ ] **Notes:** one per day per game, or unlimited?
- [ ] **Math Owner:** is `daily_math_owners` a simple name-only table or should it eventually link to `profiles`?
