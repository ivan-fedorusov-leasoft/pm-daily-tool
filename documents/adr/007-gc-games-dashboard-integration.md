# Architecture Decisions

---

# ADR-007: Move Integration Target to gc-games-dashboard

**Date:** 2026-06-30  
**Supersedes:** ADR-006

---

## Decision

Daily Tool (BE Radar) is built as a module (`src/app/radar/*`) inside the existing **`gc-games-dashboard`** Next.js application, not `gc-pm-automation` as planned in ADR-006.

---

## Reason

- `gc-pm-automation`'s Supabase project (`xhreggwwzjwoiuclegzi`) has no service role key available and no dashboard access — applying migration `0003_radar.sql` was blocked indefinitely.
- `gc-games-dashboard` already has full Supabase access: `SUPABASE_SERVICE_ROLE_KEY` is in `.env`, project `hstvuhqqbhzsgkzvgntm` has the schema applied, and 59 real game records exist.
- Both tools serve the same internal company users (leasoft.org / gamingcorps.com domains via Google SSO).
- `gc-games-dashboard` is a simpler codebase — no existing complex auth wiring, no conflicting data models. Easier to extend cleanly.
- BE Radar is conceptually closer to game-health monitoring than to Jira/template automation — it belongs in the game-monitoring context.

---

## Alternatives

- Keep integration in `gc-pm-automation` and wait for dashboard access (rejected — blocked, no timeline).
- Build Daily Tool as a standalone deployment (rejected — duplicates infrastructure, same users, no benefit).

---

## Consequences

### Identity model changes
`gc-games-dashboard` has no `profiles` table. Instead:

```sql
create table public.radar_users (
  email        text primary key,  -- from NextAuth session.user.email
  name         text not null,
  avatar_url   text,
  role         text not null default 'developer'
                 check (role in ('developer', 'manager', 'admin')),
  created_at   timestamptz not null default now(),
  updated_at   timestamptz not null default now()
);
```

- Upserted on every sign-in from NextAuth Google profile.
- Role defaults to `developer`. Admin sets manager/admin roles directly in DB.
- All `radar_*` FKs reference `radar_users.email` (not `profiles.id`).
- No `profiles.radar_role` override column — the model is simpler: the user's role IS their `radar_users.role`.

### Auth
- NextAuth v5 (Auth.js) with Google provider — already in gc-games-dashboard.
- `getCurrentUserKey()` returns `session.user.email` or `"local"` (open/dev mode).
- No Supabase Auth JWT — service role bypasses RLS entirely.

### Design system
- No `primitives.tsx`, no `AppShell` component — gc-games-dashboard uses raw Tailwind CSS.
- CSS vars: `--background #07090d`, `--panel #0d1117`, `--panel-2 #11161f`, `--border #1d2430`, `--foreground #e6edf3`, `--muted #8b97a8`.
- A new `src/app/radar/layout.tsx` must be created to provide a shared nav for the BE Radar section.
- Full design system reference: `visualization/design-system.md`.

### Database access
- Service role key only. Application-layer access control (Server Actions check `radar_users.role`).
- No RLS helper functions — service role bypasses RLS entirely.
- All queries follow gc-games-dashboard's pattern: raw `supabase.from('radar_games').select(...)`.

### `radar_` prefix maintained
All new tables keep the `radar_` prefix to avoid collisions with gc-games-dashboard's existing tables (`games`, `check_runs`, `status_events`, `scheduled_checks`, `starred_games`).

### Migration numbering
New migration file: `supabase/migrations/0003_radar_tool.sql` (next after `0002_status_since_and_starred.sql`).

### gc-pm-automation v1 code
The v1 code written in gc-pm-automation (`app/radar/*`, `lib/radar/*`, `supabase/migrations/0003_radar.sql`) is paused — not deleted, not active. Migration was never applied.

### Roadmap unaffected
v1 / v1.5 / v2 / v3 scope and gating rules (Redis, BullMQ, GitHub, AI) are unchanged.
