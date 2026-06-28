# Foundation

---

# EXISTING APP INTEGRATION AUDIT PROMPT

## Mission

Analyze the existing application and determine the safest way to integrate the Daily Tool into it.

Do not start implementation immediately.

Do not modify files during the audit phase.

First produce an integration plan.

---

## Context

The Daily Tool was originally designed as a standalone Next.js fullstack monolith.

Now it may be integrated into an existing application that already has:

- Next.js
- Vercel deployment
- Supabase
- PostgreSQL through Supabase
- existing authentication
- existing users/profiles
- existing routes and UI

The goal is to integrate Daily Tool as an isolated internal module without breaking the existing app.

---

## Primary Objective

Find the best integration strategy that:

- reuses existing infrastructure where appropriate;
- avoids duplicating auth unnecessarily;
- avoids breaking existing routes, tables, policies, and UI;
- keeps Daily Tool isolated enough to be maintainable;
- preserves the Daily Tool specifications;
- minimizes risk to the existing product.

---

## Required Reading

Before making recommendations, read:

1. README.md
2. CLAUDE.md
3. foundation/AI_CONTEXT.md
4. foundation/PRODUCT.md
5. foundation/GLOSSARY.md
6. product/ROADMAP.md
7. product/USER_FLOWS.md
8. product/UI_UX.md
9. technical/DATABASE.yaml
10. technical/ARCHITECTURE.md
11. technical/API.yaml
12. adr/*

Then inspect the existing application structure.

---

## Existing App Analysis

Analyze and report:

### Framework

- Next.js version
- App Router or Pages Router
- existing route structure
- layout structure
- middleware usage

### Auth

- current auth provider
- Supabase Auth usage
- user/profile tables
- role/permission model if any
- session access pattern

### Database

- existing Supabase tables
- existing migrations
- existing naming conventions
- existing RLS policies
- whether service role is used server-side
- whether client-side Supabase access exists

### UI

- existing navigation
- existing layout/sidebar/header
- existing design system/components
- where a Daily Tool tab/page should live

### Deployment

- Vercel setup
- environment variables
- Supabase environment usage
- existing build constraints

---

## Integration Questions To Answer

Before implementation, answer these questions:

1. Should Daily Tool use the existing auth system?
2. Should Daily Tool reuse an existing profiles/users table?
3. Should Daily Tool add separate daily_* tables?
4. Should Daily Tool use Supabase RLS or server-side permission checks?
5. Where should Daily Tool routes live?
6. How should Daily Tool be added to navigation?
7. What existing code must not be touched?
8. What risks exist?
9. What should be implemented in the first safe integration step?

---

## Preferred Integration Direction

Prefer integrating Daily Tool as an isolated module:

```text
app/
  daily-tool/
    today/
    radar/
    games/[id]/

src/
  features/
    daily-tool/

  server/
    daily-tool/
      services/
      repositories/
      permissions/
```

Prefer table names with a `daily_` prefix unless the existing schema strongly suggests another convention.

Examples:

- daily_games
- daily_math_owners
- daily_game_stage_dates
- daily_notes
- daily_change_requests
- daily_stage_history
- daily_sessions
- daily_audit_logs
- daily_pull_requests

Do not rename product concepts unless explicitly approved.

---

## Permission Rules

Preserve the Daily Tool permission model.

- Permissions are role-based.
- Responsibilities are assignment-based.
- Daily session control is session-based.

Important:

- Developer can own games.
- Manager can own games.
- Admin can own games.
- Any authenticated user may start a Daily session.
- The user who starts Daily becomes Daily Host.
- Daily Host does not receive Manager/Admin permissions.

If existing app roles conflict with Daily Tool roles, propose a mapping instead of silently changing the model.

---

## Supabase Rules

If Supabase is used:

- PostgreSQL remains the source of truth.
- Do not duplicate users if existing auth/profile tables can be reused safely.
- Do not weaken existing RLS policies.
- Do not expose protected writes directly from the client.
- Prefer server-side permission checks for protected project data.
- If RLS is required, propose policies explicitly before implementation.

---

## Output Required

Produce an integration audit report with this structure:

```md
# Daily Tool Integration Audit

## Summary

Short recommendation.

## Existing App Findings

What was found in the current app.

## Recommended Integration Strategy

How Daily Tool should be integrated.

## Database Strategy

Which tables to create or reuse.

## Auth & Permissions Strategy

How users, roles, assignments, and Daily Host should work.

## Routing Strategy

Where Daily Tool pages should live.

## UI Integration Strategy

How Daily Tool should appear in existing navigation/layout.

## Risks

List risks and how to avoid them.

## Files Likely To Change

List files/directories that will likely be modified.

## Files That Should Not Be Touched

List sensitive areas to avoid.

## Open Questions

Questions that must be answered before implementation.

## First Implementation Step

Smallest safe first step.
```

---

## Strict Rules

- Do not implement before producing the audit.
- Do not modify existing app behavior during audit.
- Do not introduce Redis unless the current roadmap stage requires v1.5.
- Do not introduce BullMQ unless the current roadmap stage requires v2.
- Do not introduce AI features.
- Do not implement future ideas.
- Do not refactor unrelated code.
- Do not rename existing Daily Tool concepts.
- Do not change architecture decisions without explaining which ADR must change.

---

## Success Criteria

The audit is successful when:

- the safest integration path is clear;
- risks are identified;
- existing app boundaries are respected;
- Daily Tool specs remain valid;
- the first implementation step is small and low-risk;
- no code has been changed yet.
