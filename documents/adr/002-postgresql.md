# Architecture Decisions

---

# ADR-002: PostgreSQL + Drizzle ORM

## Decision

Use PostgreSQL as the main database and Drizzle ORM as the database layer.

## Reason

- Project data is relational.
- Games, users, daily notes, stage dates, change requests, and pull requests have clear relationships.
- PostgreSQL is mature and reliable.
- PostgreSQL supports JSONB for flexible future metadata.
- PostgreSQL can support future full-text or vector search if needed.
- Drizzle is lightweight and explicit.

## Alternatives

- SQLite.
- MySQL.
- MongoDB.
- Prisma ORM.

## Consequences

- PostgreSQL is the source of truth.
- Redis must never become the source of truth.
- Database schema is defined explicitly.
- Prefer simple SQL-friendly models over document-style storage.
