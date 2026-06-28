# Architecture Decisions

---

# ADR-003: Redis Starts in v1.5

## Decision

Do not include Redis in MVP. Introduce Redis in v1.5.

## Reason

Redis is not required for the first usable version.

Redis becomes useful when the product adds:
- Daily Mode synchronization.
- Presence.
- Edit locks.
- Cache.
- Rate limiting.
- Realtime events.

## Alternatives

- Add Redis in MVP.
- Store runtime state in PostgreSQL.
- Skip realtime coordination.

## Consequences

- MVP remains simpler.
- v1.5 introduces Redis as runtime storage.
- PostgreSQL remains the source of truth.
- Redis data may be temporary and disposable.
- Audit logs do not replace permission checks.
