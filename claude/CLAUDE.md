# Foundation

---

# CLAUDE

## Mission

Implement the project according to the specifications.

Do not redesign the product.

Do not over-engineer.

---

## Before Every Task

Read the relevant documentation before implementing or modifying any feature.

Minimum required read order:

1. README.md
2. documents/foundation/AI_CONTEXT.md
3. documents/foundation/PRODUCT.md
4. documents/foundation/GLOSSARY.md
5. documents/product/ROADMAP.md
6. documents/product/USER_FLOWS.md
7. documents/product/UI_UX.md
8. documents/technical/DATABASE.yaml
9. documents/technical/ARCHITECTURE.md
10. documents/technical/API.yaml
11. adr/*

Only read additional documents if the task requires them.

---

## General Rules

- Documentation is the source of truth.
- Follow ADR decisions.
- Keep naming consistent with GLOSSARY.md.
- Reuse existing patterns.
- Prefer consistency over novelty.
- Prefer simplicity over flexibility.
- Do not introduce unnecessary abstractions.
- Do not rename existing concepts.

---

## Permission Model

Respect the permission model.

- Permissions are role-based.
- Responsibilities are assignment-based.
- Daily session control is session-based.

Do not mix these concepts.

Important rules:

- Developer can own games.
- Manager can also own games.
- Admin can also own games.
- Daily Host is not a role.
- Any authenticated user may start a Daily session and become Daily Host.
- Starting Daily does not grant Manager/Admin permissions.

---

## Development Rules

- Implement only the requested task.
- Do not refactor unrelated code.
- Do not rename existing entities.
- Do not modify documentation unless explicitly requested.
- Do not implement postponed roadmap items.
- Do not introduce Redis before v1.5.
- Do not introduce BullMQ before v2.
- Do not introduce GitHub integration before v2.
- Do not introduce AI features before v3.

---

## MVP Rules

Focus on the current roadmap stage only.

Ignore future features unless explicitly requested.

If a feature is listed in FUTURE_IDEAS.md, do not implement it unless explicitly requested.

---

## Architecture Rules

- Single Next.js fullstack monolith.
- PostgreSQL is the source of truth.
- Redis stores runtime state only.
- BullMQ is only for background jobs.
- GitHub starts in v2.
- AI starts in v3.

---

## Data Rules

- Use DATABASE.yaml as the database schema reference.
- Use API.yaml as the operation contract reference.
- Do not invent new tables, fields, enums, or operations without explicit approval.
- If implementation requires a schema change, stop and explain why.

---

## UI Rules

- Keep UI simple.
- Prefer progressive disclosure.
- Avoid modal-heavy workflows.
- Today's Work is the default landing page.
- Game Page is the source of truth for a single game.
- Simple Radar is a lightweight overview.
- Daily Mode is a synchronized presentation mode.

---

## If Requirements Are Unclear

Do not guess.

Explain the ambiguity.

Propose one or more implementation options.

Wait for clarification before changing architecture, schema, permissions, or core terminology.

---

## Success Criteria

A task is complete when:

- The implementation matches the specifications.
- Existing functionality is preserved.
- Permission rules are enforced.
- Code follows the project architecture.
- No unnecessary complexity was introduced.
- No unrelated files were modified.
- No future roadmap items were implemented accidentally.
