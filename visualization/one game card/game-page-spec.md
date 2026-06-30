# Frontend Spec — Game Page (Individual Game)

> Source: `game page.png`  
> App: **BE Radar** (inside gc-games-dashboard)  
> Route: `/radar/games/:id`  
> Design system: see [design-system.md](../design-system.md) — tokens from `gc-games-dashboard/src/app/globals.css`, raw Tailwind

---

## Layout

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  ← Today  >  Cooking Diary                              (breadcrumb nav)    │
├──────────────────────────────────────────────────────────────────────────────┤
│  [img 100px]  Cooking Diary   ● Active       [✏ Suggest Changes]  [···]    │
│               Client     Developer    Math Owner    Created   Daily Mode     │
│               Degen      Alex Smith   Maria Gomez   Apr 1,25   On           │
├────────────────┬─────────────────────────┬─────────────────────────────────┤
│   STAGES       │   DAILY NOTES           │   PULL REQUESTS (v2)            │
│  (~240px)      │   (center feed)         │   (~300px)                      │
│                │                         ├─────────────────────────────────┤
│                │                         │   CHANGE REQUESTS               │
└────────────────┴─────────────────────────┴─────────────────────────────────┘
```

Content: `mx-auto max-w-5xl px-5 py-8 md:px-10 md:py-10`  
Main grid: `grid grid-cols-[240px_1fr_300px] gap-5`

---

## Breadcrumb

```
← Today  >  Cooking Diary
```
- `text-sm text-[var(--color-muted)]` for "← Today" (clickable, `hover:text-text`)
- `>` separator: `text-[var(--color-faint)]`
- "Cooking Diary": `text-sm font-medium text-[var(--color-text)]` (non-clickable)
- Height: `40px`, `mb-4`

---

## Game Hero Header

`<Card className="p-6 mb-5">`

### Row 1 — Identity + Actions

```
[img 100px]  Cooking Diary  ● Active            [✏ Suggest Changes]  [···]
```

- Image: `100×100px rounded-2xl object-cover`
- Name: `text-2xl font-semibold text-[var(--color-text)]` — ml-4
- Status: `<Badge tone="ok">Active</Badge>` — ml-2, inline
- `✏ Suggest Changes`: `<Button variant="primary" size="md">` — right side
- `···`: `<Button variant="ghost" size="sm">` — ml-2, three-dots icon

### Row 2 — Meta Bar

```
Client          Developer        Math Owner       Created        Daily Mode
Degen           Alex Smith       Maria Gomez      Apr 1, 2025    On
```

- Layout: `flex gap-8 mt-4 pt-4 border-t border-[var(--color-border)]`
- Label: `text-xs text-[var(--color-muted)] uppercase tracking-wider`
- Value: `text-sm font-medium mt-0.5`
  - Client: `text-[var(--color-text)]`
  - Developer: `text-[var(--color-accent)] cursor-pointer`
  - Math Owner: `text-[var(--color-accent)] cursor-pointer`
  - Created: `text-[var(--color-text)]`
  - Daily Mode: `text-[var(--color-ok)]` for "On", `text-[var(--color-muted)]` for "Off"

---

## Column 1 — Stages Sidebar

`<Card className="p-5">`

### Header
```
Stages                    ✏
```
- `text-base font-semibold text-[var(--color-text)]`
- Edit pencil: `<Button variant="subtle" size="sm">` right side

### Stage List (vertical timeline)

```
[●/✓/○]  Stage Name
│         Apr 1 – Apr 14
│
[●/✓/○]  Stage Name
          ...
```

**Left edge line:** `border-l-2 border-[var(--color-border)] ml-[9px]`, completed = `border-[var(--color-ok)]`

**Node variants:**

| State | Classes |
|---|---|
| Completed | `20px circle bg-[var(--color-ok)] text-white flex items-center justify-center` — `✓` |
| Current | `20px circle bg-[var(--color-accent)] text-white` — `■` or dot, name `text-[var(--color-accent)] font-semibold` |
| Future | `20px circle border-2 border-[var(--color-border)] bg-[var(--color-surface-2)]` |

**Stage name:** `text-sm font-medium text-[var(--color-text)]` (current: accent color + semibold)  
**Date range:** `text-xs text-[var(--color-muted)]`

**Stages list (from `radar_game_stage` enum):**
1. Start Date
2. Playable
3. Alpha
4. Beta
5. Gold
6. Master
7. ETA Release Exclusive
8. ETA Release Com

---

## Column 2 — Daily Notes Feed

`<Card className="p-5">`

### Header
```
Daily Notes                          View all
```
- `text-base font-semibold` + `text-sm text-[var(--color-accent)]` right

### Note Items

```
[Avatar 32px]  May 12, 10:32 AM  by Alex Smith    [⚠ flag]
               ────────────────────────────────────────────
               Adjusted tutorial step for better retention.

               We've simplified step 3 in the tutorial flow...
```

- Avatar: `h-8 w-8 rounded-full`
- Timestamp: `text-xs text-[var(--color-faint)]`
- "by Name": `text-xs text-[var(--color-accent)]`
- Warning flag `⚠`: `text-[var(--color-warn)]` — shown if `needs_lead_attention = true`
- Headline: `text-base font-medium text-[var(--color-text)] mt-2`
- Body: `text-sm text-[var(--color-muted)] leading-relaxed mt-1`
- Separator between notes: `border-t border-[var(--color-border)] my-4`

### Add Note Button
```tsx
<Button variant="ghost" className="w-full mt-2">+ Add Note</Button>
```

---

## Column 3 — Right Panels (stacked)

### Panel A — Pull Requests (v2 only)

`<Card className="p-5 mb-4">`

```
Pull Requests (v2 only)              View all
────────────────────────────────────────────────
v2.3.1  Balance changes    [In Review]  #142  >
v2.2.4  Refill rate        [Merged]     #138  >
v2.2.1  UI improvements    [Merged]     #133  >
```

- Header: `text-base font-semibold` + "View all" `text-sm text-[var(--color-accent)]`
- PR row: `flex items-center gap-2 py-2.5 border-b border-[var(--color-border)] hover:bg-[var(--color-surface-2)]/40 cursor-pointer`
  - Version: `text-xs text-[var(--color-faint)] font-mono w-12`
  - Title: `text-sm text-[var(--color-text)] flex-1 truncate`
  - Status badge:
    - In Review → `<Badge tone="accent">In Review</Badge>`
    - Merged → `<Badge tone="ok">Merged</Badge>`
  - PR number: `text-xs text-[var(--color-faint)]`
  - `>` chevron: `text-[var(--color-muted)]`

### Panel B — Change Requests

`<Card className="p-5">`

```
Change Requests                      View all
────────────────────────────────────────────────
Current stage date        [Pending]           >
Refill Energy Amount      [Approved]          >
Ad Free Reward Gems       [Declined]          >
```

- Header: same as PR panel
- Row: `flex items-center justify-between py-2.5 border-b border-[var(--color-border)] hover:bg-[var(--color-surface-2)]/40 cursor-pointer`
  - Name: `text-sm text-[var(--color-text)] flex-1`
  - Status badge (same as Change Requests screen: rounded-lg, not pill)
  - `>` chevron: `text-[var(--color-muted)]`

---

## Component List

```
// src/app/radar/games/[id]/page.tsx
<RadarLayout user={user}>
  <Breadcrumb items={[{label:"Today", href:"/radar/today"}, {label:game.title}]} />

  <Card className="p-6 mb-5 animate-in">
    <div className="flex gap-4 items-start">
      <GameImage src={game.image} />
      <div className="flex-1">
        <div className="flex items-center gap-2">
          <GameTitle>{game.title}</GameTitle>
          <StatusBadge status={game.status} />
        </div>
        <MetaBar fields={[client, developer, mathOwner, created, dailyMode]} />
      </div>
      <div className="flex items-center gap-2 shrink-0">
        <Button variant="primary">✏ Suggest Changes</Button>
        <Button variant="ghost" size="sm">···</Button>
      </div>
    </div>
  </Card>

  <div className="grid grid-cols-[240px_1fr_300px] gap-5">
    <StagesSidebar stages={game.stages} currentStage={game.currentStage} />

    <Card className="p-5">
      <DailyNotesFeed notes={game.notes} onAddNote={...} />
    </Card>

    <div className="grid gap-4 content-start">
      <PullRequestsPanel prs={game.pullRequests} />
      <ChangeRequestsPanel requests={game.changeRequests} />
    </div>
  </div>
</RadarLayout>
```

---

## Open Questions / TBD

- [ ] `✏ Suggest Changes` — opens Change Request creation modal or `/radar/change-requests/new?game=id`?
- [ ] `···` menu — edit game, archive, manage stage dates?
- [ ] Stages edit `✏` — date picker per stage or bulk edit modal?
- [ ] Warning flag on note `⚠` — does clicking navigate to Team Lead view?
- [ ] "View all" on notes — infinite scroll in panel or dedicated notes page?
- [ ] Daily Mode "On/Off" — toggleable by manager/admin?
- [ ] PR numbers `#142` — link to GitHub? (v2 feature)
- [ ] Panel visibility — PR panel hidden until v2?
