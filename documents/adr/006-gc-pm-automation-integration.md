# Architecture Decisions

---

# ADR-006: Integrate Into gc-pm-automation Instead Of Standalone

## Decision

Daily Tool is built as a module (`app/daily/*`) inside the existing `gc-pm-automation` Next.js/Supabase application, instead of as its own standalone repository and deployment.

## Reason

- `gc-pm-automation` already runs a working Next.js 15 + Supabase (Postgres + Auth + RLS) stack on Vercel.
- It already has Google OAuth restricted to the company's email domains (`gamingcorps.com`, `leasoft.org`) and a `profiles` table 1:1 with `auth.users`.
- Both tools serve the same internal company users — a second monolith would duplicate auth, hosting, and database infrastructure for no product benefit.
- Full audit and rationale: `claude/INTEGRATION_AUDIT.md`.

## Alternatives

- Standalone Daily Tool repo/deployment, as originally planned in ADR-001 (rejected — unnecessary infrastructure duplication).
- Migrate `gc-pm-automation`'s users into a new auth system (rejected — migration risk for zero benefit).

## Consequences

- Amends ADR-001: Daily Tool is still a single Next.js fullstack monolith — it is now the *existing* `gc-pm-automation` monolith, not a new one. No separate frontend/backend, no microservices, no second repository.
- Amends ADR-002: no Drizzle ORM. Daily Tool uses the same pattern already used throughout `gc-pm-automation`: `@supabase/supabase-js` clients + types generated into `lib/database.types.ts`, with access control enforced through Postgres RLS policies. PostgreSQL remains the source of truth — now the same Supabase project already backing `gc-pm-automation`.
- No new `users` table. Daily Tool reuses `gc-pm-automation`'s `profiles` table for identity.
- Daily Tool's `developer` / `manager` / `admin` role model is derived from `gc-pm-automation`'s existing `user_role` enum via a new, additive, overridable `profiles.daily_role` column. See `documents/technical/DATABASE.yaml` and `documents/technical/ARCHITECTURE.md`.
- All new database objects (tables, enums) use a `daily_` prefix to avoid name collisions in the shared database.
- Routing, layout (`AppShell`), and code conventions follow `gc-pm-automation`'s existing patterns — flat `app/daily/<route>` + `lib/daily/*.ts`, Server Actions colocated as `actions.ts` per route — instead of the originally planned `server/{services,repositories,permissions}` layered structure.
- Roadmap stages (v1 / v1.5 / v2 / v3) and the Redis/BullMQ/AI gating rules are unaffected by this decision.
