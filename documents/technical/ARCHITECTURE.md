# Technical Design

------------------------------------------------------------------------

# Architecture Style

Use a single fullstack monolith.

Frontend and backend live in one repository.

Do not split into separate frontend/backend services.

------------------------------------------------------------------------

# Stack

## v1

-   Next.js
-   TypeScript
-   PostgreSQL
-   Drizzle ORM
-   Auth
-   Server Actions / API Routes

## v1.5

Add: - Redis - Realtime transport - Presence - Locks - Cache - Rate
limiting

## v2

Add: - BullMQ - GitHub webhooks - Background workers

## v3

Add: - AI providers - Transcript processing - Embeddings - Semantic
search

------------------------------------------------------------------------

# Suggested Project Structure

``` text
src/
  app/
    today/
    games/
    radar/
    api/

  server/
    db/
      schema.ts
      client.ts

    services/
      games.service.ts
      daily-notes.service.ts
      change-requests.service.ts
      stages.service.ts

    repositories/
      games.repo.ts
      users.repo.ts
      daily-notes.repo.ts
      change-requests.repo.ts

    auth/
      auth.config.ts
      permissions.ts

  features/
    today/
    game-page/
    simple-radar/
    daily-mode/

  shared/
    types/
    constants/
    utils/
```

------------------------------------------------------------------------

# Layers

## UI Layer

Responsibilities: - render screens - collect user input - display
validation errors - call Server Actions / API

Must not: - contain business permission logic - write directly to
database

------------------------------------------------------------------------

## Service Layer

Responsibilities: - business rules - permission checks - orchestration -
calling repositories

Example: - create daily note - approve change request - update stage
date - start daily session

------------------------------------------------------------------------

## Repository Layer

Responsibilities: - database queries only - no business rules

------------------------------------------------------------------------

# Permission Model

Permissions are role-based.

Responsibilities are assignment-based.

Daily session control is session-based.

## Roles

### developer

-   can create/edit own Daily Notes for assigned games
-   can submit Change Requests

### manager

-   all developer capabilities
-   can edit protected project data
-   can approve/reject Change Requests
-   can assign Daily Host
-   can also be assigned as developer to games

### admin

-   all manager capabilities
-   can manage users
-   can manage system settings
-   can also be assigned as developer to games

## Daily Host

Daily Host is not a role.

Daily Host is assigned to one Daily session.

Daily Host can: - control Daily Mode navigation - start/continue/finish
a session - edit Daily Notes during the session

Daily Host cannot: - edit protected project data unless also
manager/admin

------------------------------------------------------------------------

# Redis Usage

Redis is introduced in v1.5.

Redis is used for runtime data only.

PostgreSQL remains the source of truth.

Use Redis for: - active Daily session state - current game in Daily
Mode - presence - edit locks - cache - rate limiting - pub/sub events

Examples: - `daily:active_session_id` -
`daily:{sessionId}:current_game_id` -
`presence:game:{gameId}:user:{userId}` - `lock:game:{gameId}` -
`cache:radar:list`

------------------------------------------------------------------------

# BullMQ Usage

BullMQ is introduced in v2.

Use BullMQ for background tasks that: - call external APIs - may take
several seconds - need retries - should not block HTTP requests

Examples: - process GitHub webhook - sync Pull Request details -
generate PR summary - parse transcript in v3 - generate embeddings in v3

------------------------------------------------------------------------

# Realtime

Realtime starts in v1.5.

Use realtime for: - Daily Mode synchronization - presence - lock
updates - project changes - timeline events

Daily Mode must keep all connected clients synchronized to the same
current game.

------------------------------------------------------------------------

# Protected Data

Protected project data includes: - game title - client - assigned
developer - math owner - current stage - stage dates

Only manager/admin can edit protected data directly.

Developers submit Change Requests.

------------------------------------------------------------------------

# MVP Constraints

Do not implement in v1: - Redis - BullMQ - GitHub webhooks - AI -
Timeline - semantic search - Slack integration - calendar integration
