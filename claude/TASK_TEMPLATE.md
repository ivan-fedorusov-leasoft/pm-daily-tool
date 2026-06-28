# Foundation

---

# TASK TEMPLATE

## Goal

Describe the exact feature, fix, or change to implement.

Keep the goal narrow.

---

## Current Roadmap Stage

Specify the current implementation stage.

Example:

- v1
- v1.5
- v2
- v3

Do not implement features from later stages unless explicitly requested.

---

## Context

Explain why this task exists.

Include important product or business context.

---

## Documentation To Read

List only the documents relevant to this task.

Recommended baseline:

- README.md
- foundation/AI_CONTEXT.md
- foundation/GLOSSARY.md
- product/ROADMAP.md
- product/USER_FLOWS.md
- product/UI_UX.md
- technical/DATABASE.yaml
- technical/ARCHITECTURE.md
- technical/API.yaml
- adr/*

---

## Requirements

List functional requirements.

Example:

- Create a new Daily Note.
- Allow editing an existing Daily Note.
- Respect role permissions.
- Validate input.
- Show validation errors in UI.

---

## Permission Rules

Describe who can perform this action.

Example:

- Developer can edit own Daily Notes for assigned active games.
- Manager/Admin can edit Daily Notes for any game.
- Daily Host can edit Daily Notes during the active Daily session.
- Daily Host does not receive Manager/Admin permissions.

---

## Constraints

Things that must NOT change.

Example:

- Do not modify database schema.
- Do not introduce Redis.
- Do not change existing API contracts.
- Do not rename entities.
- Do not implement future roadmap items.

---

## Files Allowed To Modify

Explicitly list allowed files.

Example:

- src/features/daily-notes/*
- src/server/services/daily-notes.service.ts
- src/server/repositories/daily-notes.repo.ts

Do not modify files outside this list unless explicitly requested.

---

## Expected Output

Describe what the final result should look like.

Example:

- User can create and update Daily Notes from Today's Work.
- Notes are saved to PostgreSQL.
- Notes are visible on Game Page.
- Permission rules are enforced.

---

## Verification

List how to verify the task.

Example:

- Run typecheck.
- Run lint.
- Run tests if available.
- Manually test the main flow.
- Confirm no unrelated files changed.

---

## Definition of Done

The task is complete when:

- Feature works.
- Permission rules are respected.
- Existing functionality is preserved.
- Code follows project architecture.
- Documentation remains valid.
- No unrelated files were modified.
- No future roadmap features were implemented accidentally.

---

## Notes

Any additional implementation details.

Optional.
