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

- Use a single Next.js fullstack monolith.
- Never split frontend and backend into separate repositories.
- PostgreSQL is the source of truth.
- Drizzle ORM is the ORM.
- Redis is introduced only in v1.5.
- BullMQ is introduced only in v2.
- GitHub integration is NOT part of MVP.
- AI integration is NOT part of MVP.

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
