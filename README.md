# README

## Purpose

This repository follows a specification-first development process.

Documentation is the source of truth.

Always read the documentation before implementing or modifying any feature.

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

- Single Next.js fullstack monolith.
- PostgreSQL is the source of truth.
- Redis stores runtime state only.
- BullMQ is used only for background jobs.
- GitHub integration starts in v2.
- AI integration starts in v3.

---

## Claude Workflow Files

Use these files when working with Claude or another implementation AI:

- `claude/CLAUDE.md` — general behavior rules for implementation.
- `claude/TASK_TEMPLATE.md` — template for scoped implementation tasks.
- `claude/EXISTING_APP_INTEGRATION_AUDIT_PROMPT.md` — use before integrating Daily Tool into an existing Next.js/Supabase application.

---

## Goal

Build a simple, maintainable and autonomous internal tool.

Avoid unnecessary complexity.

Optimize for readability, maintainability and fast iteration.
