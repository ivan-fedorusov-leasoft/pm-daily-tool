# Architecture Decisions

---

# ADR-001: Next.js Fullstack Monolith

## Decision

Use a single Next.js fullstack monolith.

## Reason

- The product is a small internal tool.
- One repository is easier for AI-assisted development.
- Shared types are easier to maintain.
- Frontend and backend do not need separate deployment pipelines.
- Splitting frontend/backend would add unnecessary complexity.

## Alternatives

- Separate React frontend + NestJS backend.
- Separate frontend/backend repositories.
- Microservices.

## Consequences

- UI, backend logic, auth, and database access live in one codebase.
- The project must keep clear internal boundaries between UI, services, repositories, and database logic.
- Do not introduce microservices unless the product becomes significantly larger.

## Amendment (see ADR-006, then ADR-007)

Daily Tool no longer ships as its own repository. Per ADR-007 it is built inside **gc-games-dashboard** as an isolated `src/app/radar/*` module (ADR-006 had planned gc-pm-automation, since superseded). The core principle — one fullstack monolith, no split frontend/backend, no microservices — still holds; the monolith changed from "a new one" to "the existing gc-games-dashboard one."
