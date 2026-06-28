# Foundation

---

# PRODUCT

## Vision

Create a lightweight internal platform for managing game development Daily meetings.

---

## Problem

Current workflow relies on Excel:

- difficult to maintain;
- poor history;
- limited context;
- manual communication.

---

## Goal

Create a single source of truth where users can:

- update project status;
- review project progress;
- conduct Daily meetings;
- review project history;
- later integrate GitHub and AI.

---

## Non Goals

MVP is NOT intended to:

- replace GitHub;
- become a project management suite;
- become a task tracker;
- support microservices;
- optimize for thousands of projects.

---

## Success Criteria

- Developers spend less than 2 minutes updating their status.
- Managers/Admins spend less time collecting information.
- Daily meetings become shorter or unnecessary.
- Daily meetings can be conducted by any authenticated user when needed.

---

## Users

### Developer

Responsible for one or more games.

Can:

- write Daily Notes;
- submit Change Requests;
- link Pull Requests to assigned games;
- start and host a Daily session.

Cannot:

- directly edit protected project data.

---

### Manager

May also be assigned as developer to one or more games.

In addition to regular developer responsibilities, can:

- edit protected project data;
- approve or reject Change Requests;
- link Pull Requests to any game.

---

### Admin

May also be assigned as developer to one or more games.

Includes all Manager permissions.

Additionally can:

- manage users;
- manage system settings.

---

## Daily Host

Daily Host is not a permanent role.

Any authenticated user can start a Daily session.

The user who starts Daily automatically becomes Daily Host.

Daily Host controls the meeting, but does not receive Manager/Admin permissions.
