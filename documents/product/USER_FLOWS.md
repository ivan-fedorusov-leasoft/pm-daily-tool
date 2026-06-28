# Product Design

---

# USER FLOWS

## Developer

### Daily Update

Today's Work shows only active games assigned to the current user.

Flow:

Login → Today's Work → Write Daily Notes → Mark Team Lead Attention optional → Save → Done

### Suggest Change

Flow:

Game Page → Suggest Change → Submit → Wait for approval

### Link Pull Request

Flow:

Game Page → Pull Requests block → Add PR Link → Save

Rules:

- Developer may link Pull Requests only to games they are assigned to.
- Linking a Pull Request does not require approval.

---

## Manager

### Async Review

Flow:

Today's Work → Review notes → Review change requests → Approve / Reject

### Protected Data Edit

Flow:

Game Page → Edit protected field → Save

Rules:

- Manager may edit protected project data directly.
- Manager may also be assigned to games as developer.

---

## Admin

### System Management

Flow:

Admin area → Manage users / roles / system settings

Rules:

- Admin includes all Manager permissions.
- Admin may also be assigned to games as developer.

---

## Shared Daily

Daily Mode includes only active games.

Rules:

- Any authenticated user can start Daily Mode.
- The user who starts Daily automatically becomes Daily Host.
- Daily Host controls navigation.
- All connected users see the same game.
- Developers may edit only their own notes.
- Managers/Admins may edit protected project data.
- Daily Host does not receive Manager/Admin permissions.

---

## Daily Host

Can:

- start Daily Mode;
- control Previous / Next / Finish;
- edit Daily Notes during Daily;
- create Change Requests.

Cannot:

- directly edit protected project data unless also Manager/Admin.
