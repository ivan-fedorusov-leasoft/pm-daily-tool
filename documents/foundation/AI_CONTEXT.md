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

- Daily Tool is built inside the existing `gc-pm-automation` Next.js app, as an isolated `app/daily/*` module. It is not a standalone repository. See ADR-006.
- Never split frontend and backend into separate repositories.
- PostgreSQL is the source of truth — the same Supabase project already used by `gc-pm-automation`.
- No ORM. Use `@supabase/supabase-js` + generated `lib/database.types.ts`, matching the host app's existing pattern. Access control is enforced via Postgres RLS, not application-level ORM checks.
- Redis is introduced only in v1.5.
- BullMQ is introduced only in v2.
- GitHub integration is NOT part of MVP.
- AI integration is NOT part of MVP.

## Integration Rules

- Reuse `gc-pm-automation`'s Supabase Auth and `profiles` table for identity. Never create a separate `users` table.
- Every new table and enum uses a `daily_` prefix to avoid collisions in the shared database.
- Daily Tool's `developer` / `manager` / `admin` role is a derived value (`profiles.daily_role`, overridable), not a rename or replacement of `gc-pm-automation`'s existing `user_role` enum. Never modify that enum or its existing RLS policies.
- Reuse the host app's `AppShell` layout and add one nav entry — do not introduce a second design system or a second layout shell.
- Follow the host app's existing code layout: flat `lib/daily/*.ts` + colocated `app/daily/<route>/actions.ts`, not a separate `server/{services,repositories,permissions}` tree.
- Full integration rationale: `claude/INTEGRATION_AUDIT.md`.

---

## Product Rules

- Today's Work is the default landing page.
- Daily Mode is the primary feature.
- Daily meetings should become optional.
- Developers update status asynchronously.
- Managers/Admins review and discuss only what requires attention.
- Any authenticated user can start and host a Daily session.
- Game Page is the source of truth for a single project.
- Radar is a lightweight overview only.
- Timeline is postponed.
- Search is postponed.

---

## Naming Rules

- Never rename existing concepts.
- Always use terminology from GLOSSARY.md.
- Reuse existing entities before creating new ones.
- Keep naming consistent across the project.

---

## Architecture Rules

- Reuse existing patterns before introducing new ones.
- Do not introduce new abstractions without a clear benefit.
- Prefer extending existing modules over creating new ones.

---

## Permission Model

- Permissions are role-based.
- Responsibilities are assignment-based.
- Daily session control is session-based.
- Managers may also be assigned as developers to games.
- Admins may also be assigned as developers to games.
- Being assigned to a game does not grant management permissions.
- Being a Daily Host does not grant Manager/Admin permissions.
- Any authenticated user can start a Daily session.
- The user who starts Daily automatically becomes Daily Host.
- Starting Daily does not grant Manager/Admin permissions.

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
