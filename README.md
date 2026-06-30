# README

## Purpose

This repository follows a specification-first development process.

Documentation is the source of truth.

Always read the documentation before implementing or modifying any feature.

Daily Tool is built **inside the existing `gc-games-dashboard` application** (`c:\Users\Ivan\Desktop\it\IGAMING\tools\gc-games-dashboard`), not as a standalone repository or deployment.

> **ADR-007 (2026-06-30):** Integration target changed from `gc-pm-automation` to `gc-games-dashboard`. See `documents/adr/007-gc-games-dashboard-integration.md` before implementing anything.

---

## Read Order

Read the documents in the following order:

1. `documents/foundation/AI_CONTEXT.md`
2. `documents/foundation/PRODUCT.md`
3. `documents/foundation/GLOSSARY.md`

4. `documents/product/ROADMAP.md`
5. `documents/product/USER_FLOWS.md`
6. `documents/product/UI_UX.md`

7. `documents/technical/DATABASE.yaml`
8. `documents/technical/ARCHITECTURE.md`
9. `documents/technical/API.yaml`

10. `documents/adr/*`

11. `documents/product/FUTURE_IDEAS.md`

---

## Development Rules

- Follow the specifications.
- Follow ADR decisions.
- Reuse existing patterns.
- Do not introduce unnecessary abstractions.
- Prefer consistency over novelty.
- Prefer simplicity over flexibility.
- Keep naming consistent with `documents/foundation/GLOSSARY.md`.

---

## Source of Truth

Documentation is authoritative.

If implementation conflicts with documentation, update the implementation unless documentation was intentionally changed.

---

## MVP First

Implement only the current roadmap stage.

Do not implement future roadmap items early.

Ignore postponed features unless explicitly requested.

---

## Permission Model

Permissions are role-based.

Responsibilities are assignment-based.

Daily session control is session-based.

Do not mix these concepts.

---

## Architecture

- Single Next.js fullstack monolith — the existing `gc-games-dashboard` app (Next.js 16, `src/` layout).
- PostgreSQL is the source of truth — Supabase project `hstvuhqqbhzsgkzvgntm` backing `gc-games-dashboard`.
- No ORM. Use `@supabase/supabase-js` service role, raw `.from().select()`, matching the host app's pattern.
- Identity: new `daily_users` table keyed by email (from NextAuth session) — no `profiles` table in host.
- Auth: NextAuth v5 (Google SSO) — no Supabase Auth.
- Every new table uses a `daily_` prefix.
- Redis stores runtime state only (v1.5+).
- BullMQ is used only for background jobs (v2+).
- GitHub integration starts in v2.
- AI integration starts in v3.

---

## Claude Workflow Files

Use these files when working with Claude or another implementation AI:

- `claude/CLAUDE.md` — general behavior rules for implementation.
- `claude/TASK_TEMPLATE.md` — template for scoped implementation tasks.
- `claude/EXISTING_APP_INTEGRATION_AUDIT_PROMPT.md` — the audit prompt that was used to plan the integration.
- `claude/INTEGRATION_AUDIT.md` — the resulting integration plan. Canonical reference for how Daily Tool fits inside `gc-pm-automation`.

---

## Goal

Build a simple, maintainable and autonomous internal tool.

Avoid unnecessary complexity.

Optimize for readability, maintainability and fast iteration.
