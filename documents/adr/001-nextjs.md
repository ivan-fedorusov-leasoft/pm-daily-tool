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
