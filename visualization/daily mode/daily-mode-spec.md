# Frontend Spec — Daily Mode Screen

> Source: `daily mode.png`  
> App: **BE Radar** (inside gc-pm-automation)  
> Route: `/daily/mode`  
> Feature version: v1.5 (requires Redis for live sync; v1 can be a basic non-real-time version)  
> Design system: see [design-system.md](../design-system.md)

---

## Layout

```
┌──────────┬──────────────┬──────────────────────────────────────────────────────┐
│  SIDEBAR │  GAMES PANEL │                  MAIN CONTENT                        │
│ (AppShell│  (~240px)    │                                                      │
│  global) │              │  Daily Mode                   [Transfer Daily Host]  │
│          │  Games ← hdr │  Hosted by Alex Smith                                │
│          │  ─────────── │                                                      │
│          │  [Cooking D] │  ┌─ Game Header Card ───────────────────────────┐   │
│          │  [Solitaire] │  │ [img] Name  Status  │ Client  Dev  MathOwner │   │
│          │  [Pet Adv.]  │  │                     │          [Complete →]   │   │
│          │  [Galaxy S]  │  └────────────────────────────────────────────────┘  │
│          │              │                                                      │
│          │              │  ┌─ Stage Process ─────────────────────────────────┐ │
│          │              │  │ ✓ Start  ✓ Playable  ✓ Alpha  ● Beta  🔒 Gold  │ │
│          │              │  └─────────────────────────────────────────────────┘ │
│          │              │                                                      │
│          │              │  ┌─ Note ───────┐ ┌─ Change Reqs ┐ ┌─ Activity ──┐ │
│          │              │  │              │ │              │ │             │  │
│          │              │  └──────────────┘ └──────────────┘ └─────────────┘  │
│          │              │  ─────────────────────────────────────────────────  │
│          │              │  [← Prev Game]  [🔒 Finish Daily]  [Next Game →]    │
└──────────┴──────────────┴──────────────────────────────────────────────────────┘
```

Main area content: `mx-auto max-w-5xl px-5 py-8 md:px-10 md:py-10`  
**Note:** This screen adds a secondary left panel (Games list) that sits *between* AppShell sidebar and main content. Total layout is a three-column flex.

---

## Left Games Panel

**Width:** `w-60` (240px), fixed height `h-full`  
**Background:** `bg-[var(--color-surface)] border-r border-[var(--color-border)]`  
**Scroll:** `overflow-y-auto`

### Section Header
```
Games
```
- `text-base font-semibold text-[var(--color-text)] px-4 py-4`

### Game List Items

`<Card>` equivalent per item:
```tsx
// Active:   border-[var(--color-accent)]/30  bg-[var(--color-accent)]/8
// Inactive: border-[var(--color-border)]  bg-[var(--color-surface-2)]
// Shape:    rounded-xl p-3 mx-2 mb-2
```

```
[Game img 48×48  rounded-xl]  Cooking Diary  ← text-sm font-medium
                               ● Active       ← <Badge>
```

- Image: `h-12 w-12 rounded-xl object-cover`
- Name: `text-sm font-medium text-[var(--color-text)] ml-3`
- Status: `<Badge tone="ok/accent/warn/err">` — same mapping as global

---

## Page Header Area (main content top)

```
Daily Mode                              [Transfer Daily Host]
Hosted by Alex Smith
```

- "Daily Mode": `text-2xl font-semibold tracking-tight`
- "Hosted by": `text-sm text-[var(--color-muted)]` — name in `text-[var(--color-accent)]`
- `Transfer Daily Host`:
  ```tsx
  <Button variant="ghost">Transfer Daily Host</Button>
  ```
  Icon: person-swap SVG left

---

## Game Header Card

`<Card className="p-6 animate-in">`  
Layout: flex row, align-start, justify-between

### Left — Game Identity
```
[img 80px]   Cooking Diary   ● Active
```
- Image: `80×80 rounded-2xl`
- Name: `text-xl font-semibold`
- Status: `<Badge>`

### Middle — People (3 sub-columns)
```
Client          Developer        Math Owner
[icon] Degen    [Avatar] Alex    [Avatar] Maria Gomez
```
- Label: `text-xs text-[var(--color-muted)] uppercase tracking-wider`
- Icon: person/globe SVG 16px `text-[var(--color-muted)]`
- Value: `text-sm text-[var(--color-accent)]` (all three clickable)
- Avatar: `h-6 w-6 rounded-full` for Developer and Math Owner (they have profiles)

### Right — Complete Stage Button
```tsx
<Button variant="primary" className="shrink-0">Complete Stage →</Button>
<p className="text-xs text-[var(--color-muted)] mt-1 text-right max-w-36">
  Mark this stage as complete and move to the next one.
</p>
```

---

## Stage Process Section

`<Card className="p-5 mt-4">`

```
Stage Process
```
`text-base font-semibold mb-4`

### Timeline (horizontal)

Layout: `flex items-center justify-between relative`

Each stage node:
```
  Start          Playable        Alpha          Beta             Gold
  01.04.2026  ✏  15.04.2026  ✏  29.04.2026 ✏  12.05.2026  ✏  01.06.2026  ✏

  [✓]  ──────  [✓]  ──────  [✓]  ──────  [●]  ──────  [🔒]
```

**Stage label:** `text-xs text-[var(--color-muted)]` above  
**Stage date:** `text-xs text-[var(--color-faint)]` + `✏` pencil icon 12px inline (clickable by manager/admin)

**Connector line:** `h-0.5 flex-1`
- Completed: `bg-[var(--color-ok)]`
- Current/future: `bg-[var(--color-border)]`

**Node states (28px circles):**

| State | Style |
|---|---|
| Completed | `bg-[var(--color-ok)] text-white` — `✓` |
| Current (active) | `bg-[var(--color-accent)] text-white` — square `■` |
| Future | `border-2 border-[var(--color-border)] bg-[var(--color-surface-2)]` — dot |
| Future (locked) | `border-2 border-[var(--color-border)] bg-[var(--color-surface-2)]` — 🔒 icon |
| Terminal | smaller `20px`, faint dot |

---

## Bottom Info Cards (3 columns)

`<div className="grid grid-cols-3 gap-4 mt-4">`

### Card A — Today's Note (~38%)

`<Card className="p-5">`

```
📋  Today's Note                          ↗
────────────────────────────────────────────
Adjusted tutorial step for better retention.

We've simplified step 3 in the tutorial flow.
Early data shows that players were dropping
off before reaching the kitchen.
New version is live and early metrics look
promising.

Updated 10:32 AM by Alex Smith
```

- Header: `text-base font-semibold` + note icon + expand `↗` `<Button variant="subtle" size="sm">` right
- Headline: `text-lg font-medium text-[var(--color-text)] mt-4`
- Body: `text-sm text-[var(--color-muted)] leading-relaxed mt-1`
- Footer: `text-xs text-[var(--color-faint)] mt-4 pt-3 border-t border-[var(--color-border)]`

### Card B — Change Requests (~28%)

`<Card className="p-5">`

```
Change Requests  [3]
──────────────────────────────────────
current_stage          [Pending]
math_owner_id          [Approved]
status                 [Declined]
──────────────────────────────────────
View all requests →
```

- Header: `text-base font-semibold` + count `text-sm font-semibold text-[var(--color-text)]` right
- Row: `flex items-center justify-between py-2.5 border-b border-[var(--color-border)]`
  - Field: `text-xs text-[var(--color-muted)] font-mono`
  - Badge: same as Change Requests screen
- "View all requests →": `text-sm text-[var(--color-accent)] mt-3 block`

### Card C — Recent Activity (~33%)

`<Card className="p-5">`

```
Recent Activity
────────────────────────────────────────────────────────────
[Avatar] Alex Smith updated today's note        10:32 AM
[Avatar] Maria Gomez updated stage dates   Yesterday, 04:15 PM
[Avatar] John Doe completed "Balance review..."
                                           Yesterday, 11:20 AM
──────────────────────────────────────────────────────────
View all activity →
```

- Header: `text-base font-semibold`
- Row: flex gap-2, `py-2`
  - Avatar: `h-7 w-7 rounded-full`
  - Name: `text-sm font-medium text-[var(--color-accent)]` inline
  - Action: `text-sm text-[var(--color-muted)]` inline after name
  - Timestamp: `text-xs text-[var(--color-faint)]` right
  - Multi-line wraps under avatar
- "View all activity →": `text-sm text-[var(--color-accent)] mt-3 block`

---

## Bottom Navigation Bar

**Position:** sticky bottom-0 of main content area  
**Background:** `bg-[var(--color-surface)] border-t border-[var(--color-border)]`  
**Height:** `72px`  
**Layout:** `grid grid-cols-3 gap-3 px-6 items-center`

```
[← Previous Game]     [🔒 Finish Daily]     [Next Game →]
                    Available after the last game
```

### Previous Game
```tsx
<Button variant="ghost" disabled={isFirst}>← Previous Game</Button>
```

### Finish Daily (center)
```tsx
// Disabled state:
<Button variant="ghost" disabled className="flex-col gap-0">
  🔒 Finish Daily
  <span className="text-xs text-[var(--color-faint)]">Available after the last game</span>
</Button>

// Active state (last game reached):
<Button variant="primary">Finish Daily</Button>
```
- Disabled: lock icon, muted text, faint subtitle
- Active: `bg-[var(--color-accent)]` primary button

### Next Game
```tsx
<Button variant="ghost" disabled={isLast}>Next Game →</Button>
```

---

## Component List

```
<AppShell user={profile}>
  <div className="flex gap-0 -mx-5 md:-mx-10">   {/* break out of max-w-5xl padding */}

    <GamesSidePanel
      games={activeGames}
      currentId={currentGameId}
      onSelect={setCurrentGame}
    />

    <div className="flex-1 max-w-5xl px-5 py-8 md:px-10 md:py-10">
      <DailyPageHeader host={currentUser} onTransfer={...} />

      <GameHeaderCard game={currentGame}>
        <GameImage />
        <GamePeopleInfo client developer mathOwner />
        <CompleteStageButton onComplete={...} />
      </GameHeaderCard>

      <StageProcessCard stages={currentGame.stages} currentStage={...} onEditDate={...} />

      <div className="grid grid-cols-3 gap-4 mt-4">
        <TodaysNoteCard note={currentGame.todayNote} onExpand={...} />
        <ChangeRequestsCard requests={currentGame.changeRequests} />
        <RecentActivityCard activities={currentGame.recentActivity} />
      </div>

      <DailyNavBar
        onPrev={...}
        onNext={...}
        onFinish={...}
        canFinish={isLastGame}
        isFirst={isFirstGame}
        isLast={isLastGame}
      />
    </div>
  </div>
</AppShell>
```

---

## Open Questions / TBD

- [ ] "Transfer Daily Host" — any user or only current host?
- [ ] `✏` date edit on stage — inline date picker or modal? Manager/admin only
- [ ] "Complete Stage" — confirmation before moving? Writes to `daily_stage_history`
- [ ] `↗` expand on Today's Note — opens Write Note modal (`write-note-modal-spec.md`)?
- [ ] Activity feed — real-time via Redis (v1.5) or polled?
- [ ] v1 Daily Mode — non-real-time static version acceptable?
- [ ] "Finish Daily" — writes `daily_sessions.finished_at` + status = finished
- [ ] Game order in side panel — alphabetical, or by some priority?
