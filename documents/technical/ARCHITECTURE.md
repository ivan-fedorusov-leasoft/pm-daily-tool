# Technical Design

---

# ARCHITECTURE

## Purpose

Defines the high-level architecture and implementation principles of the project.

---

# Core Principles

- Documentation is the source of truth.
- PostgreSQL is the source of truth.
- Redis stores runtime state only.
- Every feature starts as a manual workflow before automation.
- AI augments existing workflows, never defines them.
- The system prioritizes autonomy over centralization.
- Approval is required only for protected project data.
- Every significant architectural decision must have an ADR.

---

# Architecture Style

Use a single Next.js fullstack monolith.

Frontend and backend live in one repository.

Do not split the project into separate frontend/backend services.

---

# Technology Stack

## v1

- Next.js
- TypeScript
- PostgreSQL
- Drizzle ORM
- Authentication
- Server Actions / API Routes

## v1.5

Add:

- Redis
- Presence
- Edit Locks
- Pub/Sub
- Cache
- Rate Limiting
- Realtime Daily synchronization

## v2

Add:

- BullMQ
- GitHub Webhooks
- Background Workers

## v3

Add:

- AI Providers
- Transcript Processing
- AI Summaries
- Embeddings
- Semantic Search

---

# Suggested Project Structure

```text
src/

  app/
    today/
    radar/
    games/
    api/

  server/
    auth/
    db/
      client.ts
      schema.ts
    repositories/
    services/
    permissions/
    realtime/
    jobs/

  features/
    today/
    radar/
    game-page/
    daily/

  shared/
    types/
    constants/
    utils/
```

---

# Layers

## UI

Responsibilities

- Render UI
- Collect user input
- Display validation
- Call Server Actions / API

Must not

- Contain business logic
- Access database directly

---

## Services

Responsibilities

- Business logic
- Permission checks
- Workflow orchestration
- Coordination between repositories

Examples

- Create Daily Note
- Approve Change Request
- Start Daily Session
- Change Stage

---

## Repositories

Responsibilities

- Database queries
- Mapping database objects

Must not

- Contain business logic

---

# Permission Model

Permissions are role-based.

Responsibilities are assignment-based.

Daily session control is session-based.

These concepts must never be mixed.

## Roles

### Developer

Can:

- Edit Daily Notes for assigned games.
- Link Pull Requests to assigned games.
- Create Change Requests.

Cannot:

- Edit protected project data directly.

### Manager

Includes all Developer permissions.

Additionally can:

- Edit protected project data.
- Approve Change Requests.
- Reject Change Requests.

Managers may also be assigned to games as developers.

### Admin

Includes all Manager permissions.

Additionally can:

- Manage users.
- Manage system settings.

Admins may also be assigned to games as developers.

---

# Daily Host

Daily Host is not a role.

Daily Host is assigned automatically when a user starts a Daily session.

Any authenticated user may become Daily Host.

Daily Host may:

- Navigate Daily Mode.
- Move between games.
- Finish Daily.
- Edit Daily Notes during the session.

Daily Host does not receive Manager/Admin permissions.

---

# Assignment Model

A game has one primary assigned developer.

Assignment determines ownership.

Role determines permissions.

---

# Project Lifecycle

Every game has two independent properties.

## Status

Possible values:

- planned
- active
- paused
- cancelled
- completed

## Stage

Possible values:

- start_date
- playable
- alpha
- beta
- gold
- master
- eta_release_exclusive
- eta_release_com

Rules

- Planned games may not have a current stage.
- Active games must have a current stage.

---

# Screen Rules

## Today's Work

Displays assigned active games only.

## Simple Radar

Displays all games.

Supports expanding a game to view:

- Current stage
- Planned stage dates
- General project information

## Game Page

Displays:

- Full project information
- Stage dates
- Daily Notes
- Change Requests
- Pull Requests

## Daily Mode

Displays active games only.

All connected clients stay synchronized.

---

# Protected Project Data

Protected fields include:

- Game Title
- Client
- Assigned Developer
- Math Owner
- Current Stage
- Stage Dates

Developers cannot modify protected fields directly.

Protected changes require a Change Request.

Managers/Admins may edit protected fields directly.

---

# Data Ownership

## Persistent Data

Stored in PostgreSQL.

Examples:

- Users
- Games
- Stage Dates
- Daily Notes
- Change Requests
- Pull Requests
- History

## Runtime Data

Stored in Redis.

Examples:

- Active Daily Session
- Current Daily Game
- Presence
- Edit Locks
- Cache
- Pub/Sub Events

Redis is never the source of truth.

---

# Redis Responsibilities

Redis is introduced in v1.5.

Used for:

- Daily synchronization
- Presence
- Edit Locks
- Pub/Sub
- Cache
- Rate Limiting

Example keys:

```text
daily:session
daily:current_game
presence:game:{id}
lock:game:{id}
cache:radar
```

---

# BullMQ Responsibilities

BullMQ is introduced in v2.

Use BullMQ only for long-running or retryable jobs.

GitHub flow:

```text
Webhook
    ↓
 BullMQ
    ↓
Database
```

AI flow:

```text
Transcript
     ↓
  BullMQ
     ↓
 AI Summary
     ↓
Database
```

---

# GitHub Integration

GitHub starts in v2.

GitHub is a block inside Game Page.

Developers may manually link Pull Requests to games they are assigned to.

Managers/Admins may link Pull Requests to any game.

GitHub Webhooks are processed asynchronously using BullMQ.

---

# AI Integration

AI starts in v3.

Workflow:

```text
Manual Workflow
       ↓
Validated Workflow
       ↓
AI Augmentation
```

AI may provide:

- Transcript parsing
- Daily summaries
- Blocker extraction
- Semantic Search
- Suggested actions

The product must remain fully usable without AI.

---

# MVP Constraints

Do not implement before their roadmap stage:

- Redis
- BullMQ
- GitHub Webhooks
- AI
- Timeline
- Semantic Search
- Slack Integration
- Calendar Integration
