# Frontend Spec — Today's Work Screen

> Source: `today work.png`  
> App: **BE Radar** (inside gc-games-dashboard)  
> Route: `/radar/today`  
> Design system: see [design-system.md](../design-system.md) — tokens from `gc-games-dashboard/src/app/globals.css`, raw Tailwind, no primitives.tsx

---

## Layout

```
┌──────────────────┬─────────────────────────────────────────────────┐
│   LEFT SIDEBAR   │              MAIN CONTENT AREA                  │
│    (256px fixed) │  mx-auto max-w-5xl px-5 py-8                   │
│                  │                                                  │
│  Logo            │  [Header: "Today's Work"  date  [+ New Game]]   │
│  ─────────────   │                                                  │
│  OVERVIEW        │  "Your Active Games" (i)                        │
│    Today ◀       │                                                  │
│    Games         │  ┌──────── Game Card ───────────────────────┐   │
│    Change Reqs   │  │ [img] Title  Status  |  Stages  |  Alert  │   │
│    Notes         │  │        Client        |          |         │   │
│  MANAGEMENT      │  │        Math Owner    |          |         │   │
│    Daily         │  ├──────────────────────────────────────────┤   │
│  SETTINGS        │  │ Last Note  |  Pull Requests  |  Change Reqs│  │
│    Settings      │  └──────────────────────────────────────────┘   │
│                  │                                                  │
│  ─────────────   │  ┌──────── Game Card ───────────────────────┐   │
│  [Avatar] Name   │  │  ...                                     │   │
│  Role       ∨    │  └──────────────────────────────────────────┘   │
└──────────────────┴─────────────────────────────────────────────────┘
```

---

## Design Tokens (real CSS vars from gc-games-dashboard)

| Role | CSS Var | Hex |
|---|---|---|
| Page background | `--color-bg` | `#07080c` |
| Card / panel | `--color-surface` | `#0d0f17` |
| Card section / nav active | `--color-surface-2` | `#131725` |
| Borders, dividers | `--color-border` | `#1f2435` |
| Strong border | `--color-border-strong` | `#2c3350` |
| Primary text | `--color-text` | `#e7e9f3` |
| Muted / labels | `--color-muted` | `#8a90a8` |
| Faint / hints | `--color-faint` | `#565d78` |
| Interactive / links | `--color-accent` | `#6366f1` |
| Accent bg (subdued) | `--color-accent-soft` | `#312e81` |
| Success / active / ok | `--color-ok` | `#34d399` |
| Warning / pending | `--color-warn` | `#fbbf24` |
| Error / declined | `--color-err` | `#f87171` |
| Card radius | `--radius-card` | `16px` |

**Typography:** `text-2xl font-semibold tracking-tight` for page title (`<PageTitle>`)  
**Font:** `ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, sans-serif`

---

## Left Sidebar

BE Radar navigation is in `src/app/radar/layout.tsx`. The host app has no shared sidebar — this layout wraps all `/radar/*` routes.

- **Width:** `w-64` = 256px, sticky, `h-dvh`
- **Style:** `glass` utility — `backdrop-filter: blur(12px)` + `bg-[var(--color-surface)]/80` + `border-r border-[var(--color-border)]`

### Logo block
```
[🎛️ / radar icon  36px rounded-xl accent/15]  BE Radar
                                                GamingCorps / Degen
```
- Icon box: `h-9 w-9 rounded-xl bg-[var(--accent)]/15 grid place-items-center text-lg`
- App name: `text-sm font-semibold text-[var(--color-text)]`
- Stream subtitle: `text-xs text-[var(--color-muted)]`

### Nav items
```tsx
// Active:   bg-[var(--accent)]/12  text-text   icon: text-accent
// Inactive: text-muted  hover:bg-surface-2  hover:text-text
// Shape:    rounded-xl px-3 py-2 text-sm
```

| Icon | Label | Route |
|---|---|---|
| 🏠 | Today | `/radar/today` |
| 🎮 | Games | `/radar/games` |
| 📋 | Change Requests | `/radar/change-requests` |
| 📝 | Notes | `/radar/notes` |
| 📅 | Daily | `/radar/mode` |
| ⚙️ | Settings | `/settings` |

### User block (bottom)
```tsx
<div className="rounded-xl border border-[var(--color-border)] bg-[var(--color-surface-2)] p-3" />
// Avatar: h-9 w-9 rounded-full (photo or accent/20 initial)
// Name: text-sm font-medium
// Role: <Badge tone="accent">
// Sign out: text-xs text-faint hover:text-err
```

---

## Page Header

```
Today's Work                          [+ New Game]
May 12, 2025
```

Uses `<PageTitle title="Today's Work" subtitle="May 12, 2025" />`:
- Title: `text-2xl font-semibold tracking-tight` (24px/600)
- Subtitle: `text-sm text-[var(--color-muted)]`
- `+ New Game` button: `<Button variant="primary">` — accent bg, `rounded-xl h-10 px-4 text-sm`

---

## Section Header: "Your Active Games"

```
Your Active Games  ⓘ
```
- `text-lg font-semibold` (18px/600)
- `ⓘ` info icon: `text-[var(--color-muted)]`, tooltip on hover

---

## Game Card

Uses `<Card>`:
```tsx
"rounded-[var(--radius-card)] border border-[var(--color-border)] bg-[var(--color-surface)]
 shadow-[0_1px_0_rgba(255,255,255,0.03)_inset,0_20px_50px_-30px_rgba(0,0,0,0.8)]"
```
- **Border-radius: 16px**
- Margin-bottom: `gap-4` or `mb-4`
- `animate-in` on initial render

### Upper Section (info row)
**Padding:** `p-6`, flex row, min-height ~140px

#### Column 1 — Game Identity
```
[img 80×80 rounded-2xl]  Cooking Diary     ← font-semibold text-lg
                         ● Active          ← status badge
                         Client
                         Degen
                         Math Owner
                         Maria Gomez       ← text-accent, clickable
```
- Image: `80×80px rounded-2xl object-cover`
- Name: `text-lg font-semibold text-[var(--color-text)]`
- Status: `<Badge tone="ok">Active</Badge>` — dot + label:
  - Active → `tone="ok"` green
  - Planned → `tone="accent"` purple
  - In Progress → `tone="warn"` amber
  - On Hold → custom err class
- "Client" / "Math Owner" label: `text-xs text-[var(--color-muted)] uppercase tracking-wider`
- Client value: `text-sm font-medium text-[var(--color-text)]`
- Math Owner value: `text-sm text-[var(--color-accent)] cursor-pointer`

#### Column 2 — Stage Timeline
```
Current Stage         Next Stage
──────────────        ──────────────
Beta                  Gold
May 12 – May 26       Jun 1 – Jun 15
```
- Labels: `text-xs text-[var(--color-muted)] uppercase tracking-wider`
- Stage name: `text-2xl font-semibold text-[var(--color-text)]`
- Date range: `text-sm text-[var(--color-muted)]`
- Separator: `border-l border-[var(--color-border)]` between columns

#### Column 3 — Alert Badge (conditional)
```
⚠  Need Team Lead
   Attention
```
- Border: `1.5px solid var(--color-warn)`
- Border-radius: `rounded-xl` (12px)
- Padding: `px-4 py-2.5`
- Icon + text: `text-[var(--color-warn)]`
- Background: `bg-[var(--color-warn)]/8` (very subtle amber tint)
- Only rendered when `needs_lead_attention = true`

---

### Lower Section (activity row)
**Background:** `bg-[var(--color-surface-2)]`  
**Border-top:** `border-t border-[var(--color-border)]`  
**Padding:** `px-6 py-5`  
**Layout:** 3 columns with `divide-x divide-[var(--color-border)]`

#### Sub-column A — Last Note
```
LAST NOTE
May 12, 10:32 AM
Adjusted tutorial step for better retention.

[✏ Write Status]
```
- "LAST NOTE": `text-xs text-[var(--color-muted)] uppercase tracking-wider`
- Timestamp: `text-xs text-[var(--color-faint)]`
- Note text: `text-sm text-[var(--color-text)]` — 2 lines max, `line-clamp-2`
- `Write Status` button: `<Button variant="ghost" size="sm">` — `rounded-xl border-[var(--color-border)]`

#### Sub-column B — Pull Requests (v2 only)
```
PULL REQUESTS (V2 ONLY)   [1]

v2.3.1  Balance changes   [In Review]

View all PRs →
```
- Label: `text-xs text-[var(--color-muted)] uppercase tracking-wider`
- Count badge: `bg-[var(--color-surface)] rounded-full text-xs font-semibold px-2` or simple `font-semibold text-[var(--color-text)]`
- PR row: version `text-xs text-[var(--color-faint)]`, title `text-sm`, status `<Badge tone="accent">In Review</Badge>` or `<Badge tone="ok">Merged</Badge>`
- "View all PRs →": `text-sm text-[var(--color-accent)] hover:underline`
- If 0 PRs: "No pull requests yet" `text-sm text-[var(--color-muted)]`

#### Sub-column C — Change Requests
```
CHANGE REQUESTS   [3]

Current stage date         [Pending] >
Refill Energy Amount       [Approved] >
Ad Free Reward Gems        [Declined] >

View all requests →
```
- Same label/count style
- Status badges:
  - Pending → `<Badge tone="warn">`
  - Approved → `<Badge tone="ok">`
  - Declined → custom: `bg-[var(--color-err)]/12 text-[var(--color-err)] border-[var(--color-err)]/25`
- Chevron `>`: `text-[var(--color-muted)]`
- Row hover: `hover:bg-[var(--color-border)]/30 cursor-pointer`
- Max 3 rows, then "View all requests →"

---

## Component List

```
// src/app/radar/today/page.tsx (Server Component)
// Layout wrapping: src/app/radar/layout.tsx
<RadarLayout user={user}>
  <PageHeader title="Today's Work" subtitle={date}>
    <button className="...">+ New Game</button>
  </PageHeader>

  <div className="mb-2 flex items-center justify-between">
    <SectionHeader label="Your Active Games" info />
  </div>

  <div className="grid gap-4">
    {games.map(g => (
      <div key={g.id} className="panel animate-in rounded-lg border border-[--border]">
        <GameCardTop>
          <GameIdentity image title status client mathOwner />
          <GameStages current next />
          {g.needs_lead_attention && <AlertBadge />}
        </GameCardTop>
        <GameCardBottom className="bg-[--panel-2] border-t border-[--border]">
          <LastNotePanel note={g.lastNote} onWriteStatus={...} />
          <PullRequestsPanel prs={g.pullRequests} />
          <ChangeRequestsPanel requests={g.changeRequests} />
        </GameCardBottom>
      </div>
    ))}
  </div>
</RadarLayout>
```

---

## Open Questions / TBD

- [ ] What triggers "Need Team Lead Attention" alert — `radar_notes.needs_lead_attention` flag
- [ ] Clicking Math Owner name — filter by math owner or profile page?
- [ ] `Write Status` — opens Write Note Modal (`write-note-modal-spec.md`)
- [ ] `View all PRs` / `View all requests` — side panel or navigate to sub-page?
- [ ] Max change request rows shown in card (currently 3)
- [ ] Empty state when no active games — `<EmptyState>` from primitives
- [ ] "+ New Game" — only visible to manager/admin role?
