# Product Design

---

# ROADMAP

## v1 (MVP) — 🔄 Re-targeting to gc-games-dashboard (ADR-007, 2026-06-30)

> v1 code was built in gc-pm-automation (paused, migration never applied). Now porting to gc-games-dashboard. See `documents/adr/007-gc-games-dashboard-integration.md`.

### Features

- Today's Work (`/radar/today`)
- Game Page (`/radar/games/:id`)
- Games list (`/radar/games`)
- Manual Daily Notes
- Change Requests
- Stage Management
- Authentication (NextAuth v5 reused from gc-games-dashboard)

### v1 integration tasks

1. Write `supabase/migrations/0003_radar_tool.sql`
2. Port `src/lib/radar/*` — email-based identity (`radar_users`)
3. Add `src/app/radar/layout.tsx` — shared nav
4. Port all page components
5. Apply migration to Supabase `hstvuhqqbhzsgkzvgntm`
6. Seed demo data

### References

- `documents/product/USER_FLOWS.md`
- `documents/product/UI_UX.md`
- `documents/technical/DATABASE.yaml`
- `documents/technical/API.yaml`
- `documents/adr/007-gc-games-dashboard-integration.md`

---

## v1.5

### Features

- Daily Mode
- Redis
- Presence
- Locks
- Live synchronization
- Cache
- Rate limiting
- Timeline

### References

- `documents/product/USER_FLOWS.md`
- `documents/product/UI_UX.md`
- `documents/technical/DATABASE.yaml`
- `documents/technical/API.yaml`
- `documents/technical/ARCHITECTURE.md`

---

## v2

### Features

- GitHub integration
- Pull Request history
- BullMQ
- GitHub webhooks

### References

- `documents/technical/DATABASE.yaml`
- `documents/technical/API.yaml`
- `documents/adr/004-github.md`

---

## v3

### Features

- AI transcript parsing
- AI summaries
- Semantic search
- Embeddings

### References

- `documents/adr/005-ai.md`
