# Frontend Spec — Games Screen

> Source: `games (radar).png`  
> App: **BE Radar** (inside gc-games-dashboard)  
> Route: `/radar/games`  
> Navigation: radar layout → "Games" (active)  
> Design system: see [design-system.md](../design-system.md) — tokens from `gc-games-dashboard/src/app/globals.css`, raw Tailwind

---

## Naming note

The screen was previously called **"Simple Radar"** in the product docs. The visualisation and this spec use the name **"Games"**. Route: `/radar/games`.

---

## Layout

```
┌──────────────┬──────────────────────────────────────────────────────────┐
│   SIDEBAR    │  Games                                    [+ New Game]   │
│  (AppShell)  │                                                           │
│              │  [🔍 Search games...]  [All Statuses ▾]  [Sort ▾]  [≡][⊞]│
│              │                                                           │
│              │  ┌─ Game Row ──────────────────────────────────────────┐ │
│              │  │[img] Name    │ Stages timeline │ Note │ PRs │ CRs │> │ │
│              │  └──────────────────────────────────────────────────────┘ │
│              │  ┌─ Game Row ──────────────────────────────────────────┐ │
│              │  │ ...                                                  │ │
│              │  └──────────────────────────────────────────────────────┘ │
└──────────────┴───────────────────────────────────────────────────────────┘
```

Content area: `mx-auto max-w-5xl px-5 py-8 md:px-10 md:py-10`

---

## Page Header

```
Games                                              [+ New Game]
```
- `<PageTitle title="Games" />` — `text-2xl font-semibold tracking-tight`
- `<Button variant="primary">+ New Game</Button>` — right side, only visible to manager/admin

---

## Toolbar

**Layout:** flex row, gap-3, mb-4, align-center

### Search Input
- Placeholder: "Search games..."
- `rounded-xl border border-[var(--color-border)] bg-[var(--color-surface-2)] px-3 py-2 text-sm`
- Focus: `border-[var(--color-accent)]`
- Icon: search SVG `text-[var(--color-muted)]` left inside

### Status Filter Dropdown
- Label: "All Statuses"
- Options: All, Active, Planned, In Progress, On Hold, Paused, Completed
- `rounded-xl border border-[var(--color-border)] bg-[var(--color-surface-2)] text-sm`

### Sort Dropdown
- Label: "Sort by: Recently Updated"
- Same style

### View Toggle (list / grid)
```
[≡]  [⊞]
```
- `<Button variant="ghost" size="sm">` for each
- Active view: `border-[var(--color-accent)] text-[var(--color-accent)]` or `bg-[var(--color-accent)]/12`
- Gap between: `gap-1`

---

## Game Row (List View)

Wraps `<Card>`:
```tsx
"rounded-[var(--radius-card)] border border-[var(--color-border)] bg-[var(--color-surface)]
 shadow-[...] mb-3 cursor-pointer hover:border-[var(--color-border-strong)] transition-colors"
```
- **Border-radius: 16px**
- Hover: `border-[var(--color-border-strong)]`
- Padding: `p-5`
- Layout: CSS grid with 6 named columns

### Grid columns:
```
[Identity ~240px] | [Stages ~360px] | [Note ~180px] | [PRs ~110px] | [CRs ~150px] | [> 20px]
```

---

#### Cell: Game Identity

```
[img 64×64  rounded-2xl]  Cooking Diary        ← font-medium text-base
                           ● Active             ← <Badge>
                           🌐 Degen             ← text-xs text-muted
                           👤 Alex Smith        ← text-xs text-accent
                           f(x) Maria Gomez     ← text-xs text-accent
Updated 2h ago                                  ← text-xs text-faint
```

- Image: `64×64px rounded-2xl object-cover`
- Name: `font-medium text-[var(--color-text)]`
- Status: `<Badge tone="...">` (see design-system.md badge mapping)
- Meta icons: inline SVG 14px `text-[var(--color-muted)]`
- Person names: `text-xs text-[var(--color-accent)] hover:underline`
- "Updated X ago": `text-xs text-[var(--color-faint)] mt-1`

---

#### Cell: Stages Timeline

```
STAGES
Start   Playable   Alpha    Beta     Gold   Master  Exclusive
01.04   15.04      29.04    12.05    01.06  16.06   30.06

[✓]─────[✓]────────[✓]────[●]────────[○]────[○]────[○]
```

- "STAGES": `text-xs text-[var(--color-muted)] uppercase tracking-wider mb-1`
- Stage names: `text-[10px] text-[var(--color-muted)]` evenly spaced
- Dates: `text-[10px] text-[var(--color-faint)]`
- Connector line: `h-0.5 bg-[var(--color-border)]`, completed = `bg-[var(--color-ok)]`

**Node states:**

| State | Visual |
|---|---|
| Completed | `18px` circle, `bg-[var(--color-ok)]`, white `✓` |
| Current | `18px` circle, `bg-[var(--color-accent)]`, white `■` (or game-status color) |
| Future | `18px` circle, `border border-[var(--color-border)]` bg-surface, gray dot |

Current node color follows game status:
- Active → `--color-accent` (purple)
- Planned → `--color-accent` (purple, lighter)
- In Progress → `--color-warn` (amber)
- On Hold → `--color-err` (red)

---

#### Cell: Today's Note

```
TODAY'S NOTE
📋  Adjusted tutorial step for
    better retention. We've
    simplified step 3...

View all
```
- Label: `text-xs text-[var(--color-muted)] uppercase tracking-wider`
- Icon: note SVG `16px text-[var(--color-muted)]`
- Text: `text-xs text-[var(--color-muted)] line-clamp-3`
- "View all": `text-xs text-[var(--color-accent)]`
- If empty: `text-xs text-[var(--color-faint)]` — "Empty"

---

#### Cell: Pull Requests

```
PULL REQUESTS
  ⎇  3
  1 In Review
  2 Merged

View all
```
- Label: `text-xs text-[var(--color-muted)] uppercase tracking-wider`
- Count: `text-base font-semibold text-[var(--color-text)]`
- Breakdown: count `text-xs font-medium` (colored by status) + label `text-xs text-[var(--color-muted)]`
  - In Review → `text-[var(--color-accent)]`
  - Merged → `text-[var(--color-ok)]`
- "View all": `text-xs text-[var(--color-accent)]`

---

#### Cell: Change Requests

```
CHANGE REQUESTS
  📋  3
  1 Pending
  1 Approved
  1 Declined

  >
```
- Same label style
- Breakdown colors:
  - Pending → `text-[var(--color-warn)]`
  - Approved → `text-[var(--color-ok)]`
  - Declined → `text-[var(--color-err)]`
- `>` chevron: `text-[var(--color-muted)]`, right edge

---

## Component List

```
// src/app/radar/games/page.tsx
<RadarLayout user={user}>
  <div className="flex items-start justify-between mb-4">
    <PageTitle title="Games" />
    <Button variant="primary">+ New Game</Button>
  </div>

  <Toolbar>
    <SearchInput />
    <Dropdown label="All Statuses" />
    <SortDropdown />
    <ViewToggle view="list" />
  </Toolbar>

  <div className="grid gap-3">
    {games.map(g => (
      <Card key={g.id} className="animate-in cursor-pointer hover:border-[var(--color-border-strong)]"
            onClick={() => navigate(`/radar/games/${g.id}`)}>
        <GameIdentityCell />
        <StagesTimelineCell />
        <TodaysNoteCell />
        <PullRequestsCell />
        <ChangeRequestsCell />
        <ChevronCell />
      </Card>
    ))}
  </div>
</RadarLayout>
```

---

## Empty State

```tsx
<EmptyState title="No games yet" hint="Create your first game to get started." />
```

---

## Open Questions / TBD

- [ ] Grid view `⊞` — card grid layout TBD
- [ ] Stage names — fixed enum (`radar_game_stage`) or per-game configurable?
- [ ] "Updated X ago" — based on `radar_games.updated_at`?
- [ ] Clicking PR/CR cell — navigates to game page filtered to that tab, or separate route?
- [ ] Search — client-side filter or server-side?
- [ ] "+ New Game" — only manager/admin (`isRadarManager()`)?
- [ ] On Hold current-node color: `--color-err` or `--color-warn`?
