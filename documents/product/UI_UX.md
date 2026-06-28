# Product Design

---

# UI / UX

## Host Application Shell (reused)

Daily Tool renders inside `gc-pm-automation`'s existing `AppShell` (sidebar nav, header, design tokens) — it does not introduce a second layout or design system. A single new "Daily" entry is added to the existing sidebar, visible to every authenticated user (not gated to PM-only, since any Profile may use Today's Work and start a Daily session). All Daily Tool screens below live under the `/daily` route segment.

## Today's Work

Purpose:

Primary landing page.

Contains:

- My Games
- Today's Daily Notes
- Team Lead Attention flag
- Save action

Rules:

- Only active assigned games are shown.
- Planned, paused, cancelled and completed games are hidden.

---

## Game Page

Contains:

- General information
- Current stage
- Stage dates
- Daily Note history
- Change Requests
- Pull Requests block in v2
- AI block in v3

Rules:

- Game Page is the source of truth for a single game.
- Game Page can be opened for any game status.

---

## Simple Radar

Contains:

- Game
- Developer
- Current Stage
- Project Health
- Game Status

Expandable:

- Full stage dates
- General project information

Rules:

- Displays all games, including planned, active and paused games.

---

## Daily Mode

Presentation mode.

Contains:

- Current game
- Current stage
- Stage dates
- Today's Note
- Previous
- Next
- Finish

Rules:

- Controlled by Daily Host.
- Any authenticated user can become Daily Host by starting Daily.
- All connected users stay synchronized.
- Displays active games only.
