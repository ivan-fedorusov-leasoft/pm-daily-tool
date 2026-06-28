# Daily Tool Integration Audit

Produced per `claude/EXISTING_APP_INTEGRATION_AUDIT_PROMPT.md`. Audit only — no code, schema, or config was modified while producing this document.

---

## Summary

Integrate Daily Tool as an isolated `app/daily/*` module inside the existing `gc-pm-automation` Next.js/Supabase app. Reuse its Supabase Auth, `profiles` table, Supabase clients, and `AppShell` layout as-is. Add Daily Tool's own `daily_*` tables with their own RLS, driven by a **single new nullable column** (`profiles.daily_role`, override-only) plus a SQL helper that derives a default Daily Tool role from the existing `profiles.role` when no override is set. This requires zero changes to the existing `user_role` enum, the existing `handle_new_user` trigger, existing RLS policies, or `middleware.ts`. The only touch to existing files is one new entry in `AppShell.tsx`'s `NAV` array.

---

## Existing App Findings

(Full detail in vault: `gc-pm-automation/Overview`, `Architecture`, `Database Schema`, `Auth & Roles`, `Routes & Pages`.)

### Framework
- Next.js 15, App Router, no `src/` — `app/` and `lib/` live at repo root.
- No nested `layout.tsx` per section. Every page is a Server Component that calls a guard (`requireProfile`/`requirePM`/`requireAdmin`) and manually wraps its JSX in `<AppShell user={profile}>`.
- `middleware.ts` → `lib/supabase/middleware.ts`: matches everything except `_next`/static assets; refreshes the Supabase session cookie; redirects unauthenticated → `/login`; redirects authenticated-but-not-onboarded → `/onboarding` for any path except `/login`, `/auth/*`, `/api/*`. **New routes are covered by this gate automatically, no middleware change needed.**

### Auth
- Supabase Auth, Google OAuth, restricted to `gamingcorps.com` / `leasoft.org` domains (enforced in a DB trigger, not just UI).
- `profiles` table is 1:1 with `auth.users` (`profiles.id` = `auth.users.id`, cascade delete). This **is** the users/profile table — no separate user store exists or should be created.
- Single flat `user_role` enum: `super_admin | pm | product_owner | backender | frontender | qa_qc`. No assignment-vs-role separation today except `team_leads` (a narrow per-template-discipline exception).
- No session-based concept anywhere (nothing like Daily Host).
- Guards live in `lib/auth.ts`; pages call them directly, no central middleware-level permission layer beyond the auth/onboarding gate.

### Database
- Supabase Postgres, 4 migrations so far, RLS enabled on every table, helper functions in a non-exposed `private` schema (`SECURITY DEFINER`, `search_path=''`).
- Naming convention: plural snake_case table names, no prefix (`templates`, `runs`, `team_leads`...). No existing `daily_*`-prefixed anything, so that prefix is free and unambiguous.
- Service role is not used anywhere — all access goes through the anon key + RLS, server-side via `lib/supabase/server.ts` (cookie-based session) or client-side via `lib/supabase/client.ts`. No raw service-role client exists in the codebase today.

### UI
- Sidebar nav defined as a flat array in `app/ui/AppShell.tsx` (`NAV` const): `{ href, label, icon, pmOnly }`. Filtered by `isPM(user.role)`.
- Shared primitives: `app/ui/primitives.tsx` (Card, PageTitle, Badge, EmptyState), `app/ui/form.tsx`. No component library — custom Tailwind v4 components, dark theme via CSS vars, per-stream accent color injected at runtime.
- No router-level layout nesting to hook into — a new top-level section must (a) add a `NAV` entry and (b) have its own pages individually wrap themselves in `<AppShell>`, exactly like every existing page does.

### Deployment
- Vercel. `.env`: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `APP_ENCRYPTION_KEY`, `APP_BASE_URL`. `GET /api/health` reports which are configured.
- No Redis, no BullMQ, no background workers anywhere in the current app — fully synchronous.

---

## Recommended Integration Strategy

1. **Don't build a separate auth system.** Daily Tool runs as `app/daily/*` inside the same Next.js project, behind the same Supabase session/middleware gate that already protects every other route.
2. **Don't create a `daily_users` table.** Every Daily Tool FK that would point at "users" points at `public.profiles(id)` instead.
3. **Map Daily Tool's role model (`developer`/`manager`/`admin`) onto a derived value, not the existing `user_role` enum.** Add one new nullable column, `profiles.daily_role`, of a new enum `daily_role` (`developer`/`manager`/`admin`). It is an **override only** — `null` means "derive from existing `role`". A SQL helper computes the effective Daily Tool role:
   - `super_admin` → `admin`
   - `pm` → `manager`
   - everything else (`product_owner`, `backender`, `frontender`, `qa_qc`) → `developer`
   - if `profiles.daily_role` is explicitly set, it wins over the derived mapping (lets a future admin hand someone Daily Tool `manager` without touching their `gc-pm-automation` role).

   This means **zero migration backfill, zero change to the existing `handle_new_user` trigger, zero change to the `user_role` enum or its RLS policies.** New users get a sane Daily Tool role for free the moment they're onboarded into `gc-pm-automation`.
4. **Game ownership (assignment) is fully independent of role**, exactly as pm-daily-tool's spec requires — `daily_games.developer_id` references `profiles.id` with no constraint on that profile's role, so a `pm`/`super_admin` (mapped to manager/admin) can still be assigned as the owning developer of a game, matching `documents/foundation/AI_CONTEXT.md`'s permission model verbatim.
5. **Daily Host stays purely session-based**, stored in `daily_sessions.host_user_id` (any authenticated profile) plus Redis from v1.5 — no role or assignment implication, nothing to integrate with the existing role system at all.
6. **Reuse, don't rebuild, the UI shell.** New pages import `AppShell` exactly like `app/templates/page.tsx` does; one new line in `NAV`.
7. **Match the existing app's actual code layout, not Daily Tool's originally-planned standalone layout.** `gc-pm-automation` has no `server/services/repositories/permissions` split — it's `app/<feature>/actions.ts` + flat `lib/<feature>.ts`. Recommend Daily Tool follow that same flat pattern (`lib/daily/*.ts` + `app/daily/<route>/actions.ts`) instead of introducing a new layered architecture that nothing else in the repo uses. This is **not** a change to any pm-daily-tool ADR (ADR-001 is about monolith-vs-separate-repos, not internal folder layout) — it's a "reuse existing patterns" call per pm-daily-tool's own `CLAUDE.md` rule, applied to *which* existing patterns are now in scope. Flagged as an open question below in case you'd rather keep the originally-planned `server/` structure for clearer Daily Tool boundaries.

---

## Database Strategy

All new tables use the `daily_` prefix, live in `public`, and follow the existing migration-file convention (`supabase/migrations/000N_*.sql`).

### New enums
```
daily_role            (developer, manager, admin)        -- override column on profiles
daily_game_stage      (start_date, playable, alpha, beta, gold, master, eta_release_exclusive, eta_release_com)
daily_note_source     (manual, transcript)
daily_change_request_status (pending, approved, rejected)
daily_game_status     (planned, active, paused, cancelled, completed)
daily_session_status  (active, finished, cancelled)        -- v1.5
```
(Enum names are prefixed too, to avoid any future collision with `gc-pm-automation`'s own `task_type`/`run_status` etc.)

### New tables (v1)
| Table | Notes |
|---|---|
| `daily_math_owners` | id, name, timestamps. Not users — no FK to profiles. |
| `daily_games` | id, title, client, developer_id → profiles.id, math_owner_id → daily_math_owners.id (nullable), current_stage (daily_game_stage, nullable), status (daily_game_status), timestamps. Protected fields per spec: title, client, developer_id, math_owner_id, current_stage, status. |
| `daily_game_stage_dates` | id, game_id → daily_games.id, stage, target_date (nullable), timestamps. |
| `daily_notes` | id, game_id, author_id → profiles.id, date, summary, needs_lead_attention (bool), source, timestamps. |
| `daily_change_requests` | id, game_id, requested_by → profiles.id, field_name, old_value, new_value, status, reviewed_by → profiles.id (nullable), reviewed_at, timestamps. |
| `daily_stage_history` | id, game_id, from_stage (nullable), to_stage, changed_by → profiles.id, created_at. |

### New tables (v1.5+, not built now — listed for schema-naming continuity only)
`daily_sessions`, `daily_audit_logs`, `daily_pull_requests` (v2).

### Reused (no new table)
`profiles` (users), `profiles.daily_role` (new nullable column, additive — see above).

### RLS helper additions (new functions only, nothing existing touched)
```
private.daily_role()        -- coalesce(profiles.daily_role override, mapped default from profiles.role)
private.is_daily_admin()    -- daily_role() = 'admin'
private.is_daily_manager()  -- daily_role() in ('admin','manager')
```
Mirrors the existing `private.app_role()` / `private.is_admin()` pattern exactly — same schema, same `SECURITY DEFINER`/`search_path=''` convention, same grant-to-`authenticated` pattern.

### RLS policy shape (per table, mirroring existing `templates`/`runs` patterns)
- `select`: any authenticated profile (Daily Tool has no stream-style data partition — it's one shared workspace, unlike `gc-pm-automation`'s GC/Degen split).
- `daily_games` / `daily_game_stage_dates` update on protected fields: `is_daily_manager() OR is_daily_admin()`.
- `daily_notes` write: `author_id = auth.uid() OR is_daily_manager()` (covers "developer edits own note" + "manager/admin edits any").
- `daily_change_requests` insert: any authenticated profile; approve/reject: `is_daily_manager()`.
- Daily Host elevation never appears in RLS — it's enforced in the service/action layer reading `daily_sessions`, not via a Postgres policy, since "session-based" is inherently request-time and time-boxed (matches pm-daily-tool's own rule that a Lock/session "does not grant permissions" at the data layer).

---

## Auth & Permissions Strategy

- **Users**: `profiles` table, unchanged, reused as-is.
- **Roles**: derived `daily_role` per the mapping above; stored override is optional and additive.
- **Assignments**: `daily_games.developer_id` — independent of role, exactly like pm-daily-tool's spec (managers/admins can be assigned as developers without affecting their Daily Tool permissions).
- **Daily Host**: pure runtime concept, lives in `daily_sessions.host_user_id` (+ Redis from v1.5); any authenticated profile can become host by starting a session; zero permission elevation, enforced at the service layer.
- **Guards**: add `lib/daily/auth.ts` with `requireDailyProfile()` (= existing `requireProfile()`, Daily Tool has no extra onboarding step), `requireDailyManager()`, `requireDailyAdmin()`, mirroring `lib/auth.ts`'s existing `requirePM()`/`requireAdmin()` shape so the pattern stays consistent across the whole repo.

---

## Routing Strategy

New top-level segment `app/daily/`, matching the existing one-segment-per-feature convention (`app/templates`, `app/deploy`, `app/cleanup`, `app/settings`):

```
app/daily/
  page.tsx                 # redirects to /daily/today (Today's Work is the default landing experience for the module)
  today/page.tsx
  radar/page.tsx
  games/[id]/page.tsx
  actions.ts                # daily notes upsert, change request create/approve/reject, stage change — colocated like app/deploy/actions.ts
```
`app/daily/today` is the natural "default landing page" *within the module*; whether the Daily Tool becomes the app's actual root landing page is an open product decision (see below), not an integration mechanic — the existing `/` dashboard is untouched either way.

No middleware changes needed — the existing auth/onboarding gate already covers any path outside `/login`, `/auth/*`, `/api/*`.

---

## UI Integration Strategy

- Add one entry to `AppShell.tsx`'s `NAV` array: `{ href: "/daily", label: "Daily", icon: "📋", pmOnly: false }` — visible to every authenticated user regardless of their `gc-pm-automation` role, matching "any authenticated user may start a Daily session."
- Daily Tool pages import `AppShell` and `app/ui/primitives.tsx` exactly like existing pages — no new design system, no new shell.
- Per-stream accent theming (`STREAM_ACCENT`) is irrelevant to Daily Tool (it has no GC/Degen split) — Daily Tool pages render inside the same shell but don't need to set a custom accent; default indigo is fine.

---

## Risks

| Risk | Mitigation |
|---|---|
| Adding `daily_role` column could be perceived as touching the core permission model | It's additive/nullable with no default write path from existing triggers — `gc-pm-automation`'s own role logic (`lib/roles.ts`, all existing RLS) never reads it, so it's inert until Daily Tool code queries it. |
| New enum/table names collide with future `gc-pm-automation` additions | `daily_` prefix on every enum and table avoids this; confirmed no existing `daily_*` name in use. |
| `AppShell.tsx` edit could regress existing nav | Single additive array entry; existing entries and `pmOnly` filter logic untouched. |
| RLS bugs in new policies leak Daily Tool data | New tables are fully isolated (no joins into `gc-pm-automation` tables except read-only FK to `profiles.id`); test policies independently before wiring UI. |
| Scope creep: Daily Mode realtime (v1.5) tempts pulling in Redis/Vercel infra changes | Stay v1-only per `documents/product/ROADMAP.md`; do not provision Redis until v1.5 is explicitly requested. |
| Two unrelated products sharing one Vercel deploy/build pipeline | No isolation today (single Next.js build) — a Daily Tool bug could break `gc-pm-automation`'s build. Acceptable for an internal tool; flag if that changes. |

---

## Files Likely To Change

- `app/ui/AppShell.tsx` — one new `NAV` entry (additive).
- `supabase/migrations/0005_daily_tool_init.sql` (new file) — new enums/tables/RLS/helpers + the one `profiles.daily_role` column addition.
- `lib/database.types.ts` — regenerated after migration (adds new table/enum types; existing types should remain unchanged since nothing existing is altered, only appended).
- New files only, otherwise: `app/daily/**`, `lib/daily/**`.

## Files That Should Not Be Touched

- `lib/auth.ts`, `lib/roles.ts`, `lib/jira.ts`, `lib/deploy.ts`, `lib/crypto.ts`, `lib/jira-creds.ts` — no Daily Tool logic belongs here.
- `private.handle_new_user()` trigger and the `user_role` enum — left exactly as-is; the derived-role helper reads from them but never writes to or alters them.
- All existing RLS policies on `streams`, `templates`, `template_epics`, `template_tasks`, `runs`, `run_issues`, `team_leads`, `provisioned_users`.
- `middleware.ts` / `lib/supabase/middleware.ts` — already covers new routes for free.
- `app/templates/**`, `app/deploy/**`, `app/cleanup/**`, `app/settings/**`, `app/onboarding/**` and their `actions.ts` files.

---

## Open Questions

1. **Folder layout**: flat `lib/daily/*.ts` (matches existing app convention) vs. the originally-planned `server/{services,repositories,permissions}/daily-tool/` (matches pm-daily-tool's own `ARCHITECTURE.md`)? Recommendation above is flat, for consistency with the host app — confirm before implementation.
2. **Landing page**: should `/daily/today` become the app's actual root (`/`) eventually, or stay a secondary module reachable from nav, with `/` remaining the existing Templates/Deploy dashboard? Spec says "Today's Work is the default landing page" for Daily Tool itself, which doesn't necessarily mean for the whole combined app.
3. **`daily_role` override UI**: not required for v1 (default mapping is enough), but if you want admins to hand out Daily Tool `manager`/`admin` independent of `gc-pm-automation` role from day one, that's a small Settings-page addition worth scoping explicitly.
4. **Team Lead overlap**: `gc-pm-automation`'s `team_leads` concept (cross-stream template managers) and Daily Tool's `manager`/`admin` are unrelated — confirm there's no expectation of cross-wiring (e.g. a Team Lead automatically becoming a Daily Tool manager). Current recommendation: no link.
5. **v1.5 Redis**: when Daily Mode lands, will it use the same Vercel project (likely needs e.g. Upstash Redis) — infra decision, not urgent now.

---

## First Implementation Step

Smallest safe slice, fully reversible, touches nothing existing except one additive column:

1. Write `supabase/migrations/0005_daily_tool_init.sql`: the `daily_role` enum + `profiles.daily_role` nullable column + `private.daily_role()`/`is_daily_manager()`/`is_daily_admin()` helpers + the `daily_games` and `daily_math_owners` tables with RLS (read-only feature slice — no notes/change-requests yet).
2. Add `app/daily/page.tsx` behind `requireDailyProfile()` rendering inside `<AppShell>`, listing games from `daily_games` (read-only, seeded manually via SQL for now).
3. Add the single `NAV` entry in `AppShell.tsx`.
4. Verify: existing app still builds, existing role/RLS tests (manual smoke: login as each known account) still behave identically, new `/daily` page loads for any authenticated user and shows an empty/seeded list.

Everything else (Daily Notes, Change Requests, Stage Management, Today's Work filtering, Simple Radar) follows as separate scoped tasks once this slice is confirmed safe.
