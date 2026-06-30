# AI_CONTEXT

## Purpose

This documentation is written primarily for AI coding assistants.

---

## Core Principles

- Prefer simplicity over flexibility.
- Never over-engineer.
- Optimize for maintainability, not theoretical scalability.
- Keep the codebase readable.

---

## Project Rules

- BE Radar is built inside the existing `gc-games-dashboard` Next.js app, as an isolated `src/app/radar/*` module. It is not a standalone repository. See ADR-007.
- Never split frontend and backend into separate repositories.
- PostgreSQL is the source of truth — the Supabase project backing `gc-games-dashboard` (`hstvuhqqbhzsgkzvgntm`).
- No ORM. Use `@supabase/supabase-js` with the service role key, matching the host app's existing pattern. Service role bypasses RLS — access control is enforced in Server Actions.
- Redis is introduced only in v1.5.
- BullMQ is introduced only in v2.
- GitHub integration is NOT part of MVP.
- AI integration is NOT part of MVP.

## Integration Rules (ADR-007: gc-games-dashboard host)

- Auth: NextAuth v5 (Google SSO). Use `getCurrentRadarUser()` to get the signed-in user.
- Identity: `radar_users` table keyed by email. Upserted on sign-in. No `profiles` table exists in the host app.
- Every new table uses a `radar_` prefix to avoid collisions with the host app's existing tables.
- Role is stored in `radar_users.role` (developer / manager / admin). Defaults to developer on first sign-in.
- No Supabase Auth, no RLS helper functions — service role bypasses RLS entirely.
- Follow the host app's code layout: flat `src/lib/radar/*.ts` + colocated `src/app/radar/<route>/actions.ts`.
- No shared component library in gc-games-dashboard — use raw Tailwind CSS, `.panel` class, and inline Tailwind patterns from existing components.

---

## Product Rules

- Today's Work is the default landing page (`/radar/today`).
- Daily Mode is the primary killer feature.
- Meetings should become optional — developers update status asynchronously.
- Managers/Admins review and discuss only what requires attention.
- Any authenticated user can start and host a Daily Mode session (Daily Host).
- Game Page is the source of truth for a single project.
- Games list is a lightweight overview.
- Timeline is postponed.
- Search is postponed.

---

## Naming Rules

- The product is called **BE Radar**. Never call it "Daily Tool".
- Routes are `/radar/*`. Never `/daily/*`.
- Table prefix is `radar_`. Never `daily_`.
- Lib folder is `src/lib/radar/`. Never `lib/daily/`.
- App folder is `src/app/radar/`. Never `app/daily/`.
- **Exception:** "Daily Mode" and "Daily Host" are feature names — keep as-is.
- Never rename existing concepts.
- Keep naming consistent across the project.

---

## Architecture Rules

- Reuse existing patterns before introducing new ones.
- Do not introduce new abstractions without a clear benefit.
- Prefer extending existing modules over creating new ones.

---

## Permission Model

- Permissions are role-based (`radar_users.role`).
- Responsibilities are assignment-based (`radar_games.developer_email`).
- Daily session control is session-based (Daily Host — whoever started Daily Mode).
- Managers may also be assigned as developers to games.
- Admins may also be assigned as developers to games.
- Being assigned to a game does not grant management permissions.
- Being a Daily Host does not grant Manager/Admin permissions.
- Any authenticated user can start a Daily Mode session.
- The user who starts Daily Mode automatically becomes Daily Host.

---

## UX Rules

- Show only necessary information.
- Prefer progressive disclosure over crowded screens.
- Every action should require as few clicks as possible.
- Avoid modal-heavy workflows.

---

## Coding Rules

- Write explicit code over magical abstractions.
- Keep implementation aligned with the documentation.
- If documentation and implementation conflict, update the implementation unless documentation is intentionally changed.
