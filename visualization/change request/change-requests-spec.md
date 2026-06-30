# Frontend Spec — Change Requests Screen

> Source: `change requests to game info.png`  
> App: **BE Radar** (inside gc-games-dashboard)  
> Route: `/radar/change-requests`  
> Navigation: radar layout → "Change Requests" (active)  
> Design system: see [design-system.md](../design-system.md) — tokens from `gc-games-dashboard/src/app/globals.css`, raw Tailwind

---

## Layout

```
┌──────────────┬──────────────────────────────────────────────────────────┐
│   SIDEBAR    │  Change Requests                     [+ New Request]     │
│  (AppShell)  │                                                           │
│              │  [All 18] [Pending 8] [Approved 6] [Declined 3]         │
│              │                                                           │
│              │  [🔍 Search...] [All Games ▾] [All Fields ▾]  [Sort ▾][≡]│
│              │                                                           │
│              │  ┌─ Table ──────────────────────────────────────────────┐│
│              │  │ Game │ Field │ Old │ New │ By │ At │ Status │ Actions ││
│              │  │ row...                                               │ │
│              │  └──────────────────────────────────────────────────────┘│
│              │                                                           │
│              │  Showing 1–8 of 18   < [1] [2] [3] >   Rows/page: 10 ▾  │
└──────────────┴───────────────────────────────────────────────────────────┘
```

Content: `mx-auto max-w-5xl px-5 py-8 md:px-10 md:py-10`

---

## Page Header

```
Change Requests                              [+ New Request]
```
- `<PageTitle title="Change Requests" />`
- `<Button variant="primary">+ New Request</Button>` — only for manager/admin?

---

## Status Filter Tabs

```
[All  18]  [Pending  8]  [Approved  6]  [Declined  3]
```

Each tab is a pill toggle button:

| State | Style |
|---|---|
| Active | `bg-[var(--color-accent)]/12 border-[var(--color-border-strong)] text-[var(--color-text)]` |
| Inactive | `bg-transparent border-[var(--color-border)] text-[var(--color-muted)] hover:bg-[var(--color-surface-2)]` |

- Shape: `rounded-xl border px-4 py-1.5 text-sm font-medium`
- Count badge inside tab:
  - Active tab: `text-[var(--color-text)]`
  - Pending count: `text-[var(--color-warn)]`
  - Approved count: `text-[var(--color-ok)]`
  - Declined count: `text-[var(--color-err)]`
- Row gap: `gap-2 mb-5`

---

## Toolbar

**Layout:** flex row, gap-3, mb-4

### Search Input
```
[🔍  Search change requests...]
```
- `rounded-xl border border-[var(--color-border)] bg-[var(--color-surface-2)] px-3 py-2 text-sm`
- Width: ~320px
- Focus: `border-[var(--color-accent)]`

### Dropdowns (All Games, All Fields, Sort)
- Same `rounded-xl border bg-[var(--color-surface-2)] text-sm` style
- Chevron `▾` right side: `text-[var(--color-muted)]`
- Hover: `border-[var(--color-border-strong)]`

### Filter Icon Button
- `<Button variant="ghost" size="sm">` with filter-lines icon

---

## Table

Wraps `<Card>`:
```tsx
"rounded-[var(--radius-card)] border border-[var(--color-border)] bg-[var(--color-surface)] overflow-hidden"
```

### Table Header Row
- `bg-[var(--color-surface-2)] border-b border-[var(--color-border)] h-11`
- Column headers: `text-xs font-medium text-[var(--color-muted)] uppercase tracking-wider px-4`

| Column | Width | Align |
|---|---|---|
| Game | ~210px | left |
| Field | ~130px | left |
| Old Value | ~170px | left |
| New Value | ~170px | left |
| Requested By | ~150px | left |
| Requested At | ~150px | left |
| Status | ~110px | left |
| Actions | ~90px | center |

### Table Data Rows

- Height: ~64px (two-line content)
- `border-b border-[var(--color-border)]`
- `hover:bg-[var(--color-surface-2)]/50 transition-colors`
- Padding: `px-4`

#### Column: Game
```
[img 36×36  rounded-lg]  Cooking Diary
```
- Image: `36×36 rounded-lg object-cover`
- Name: `text-sm font-medium text-[var(--color-text)]`
- Layout: flex row gap-2 align-center

#### Column: Field
```
current_stage
```
- `text-xs text-[var(--color-muted)] font-mono`

#### Column: Old Value / New Value
```
May 12 – May 26
(Alpha)
```
- Primary value: `text-sm text-[var(--color-text)]`
- Secondary (in parens): `text-xs text-[var(--color-muted)]`
- Status values (dot + label): dot `8px` circle + `text-sm`
  - "In Progress" dot: `bg-[var(--color-warn)]`
  - "Active" dot: `bg-[var(--color-ok)]`

#### Column: Requested By
```
[Avatar 28px]  John Doe
               Developer
```
- Avatar: `h-7 w-7 rounded-full`
- Name: `text-sm text-[var(--color-accent)]`
- Role: `text-xs text-[var(--color-muted)]`

#### Column: Requested At
- `text-sm text-[var(--color-muted)]`
- Format: "May 12, 10:15 AM"

#### Column: Status
Inline badge (not the `<Badge>` rounded-full — matches screenshot style):
```tsx
// Pending
"text-xs font-semibold px-2.5 py-1 rounded-lg
 bg-[var(--color-warn)]/12 text-[var(--color-warn)]"

// Approved
"bg-[var(--color-ok)]/12 text-[var(--color-ok)]"

// Declined / Rejected
"bg-[var(--color-err)]/12 text-[var(--color-err)]"
```

#### Column: Actions (Pending rows only)
```
[✓]  [✗]  [>]
```
- `✓` approve: `24px circle border border-[var(--color-ok)]/50 text-[var(--color-ok)] hover:bg-[var(--color-ok)]/10`
- `✗` decline: `border border-[var(--color-err)]/50 text-[var(--color-err)] hover:bg-[var(--color-err)]/10`
- `>` chevron: `text-[var(--color-muted)]`
- Approved / Declined rows: only `>` chevron (no action buttons)

---

## Pagination

`border-t border-[var(--color-border)] px-4 py-3`

```
Showing 1 to 8 of 18 requests    < [1] [2] [3] >    Rows per page: 10 ▾
```

- Text: `text-sm text-[var(--color-muted)]`
- Page buttons: `32×32px rounded-lg text-sm`
  - Active: `bg-[var(--color-accent)]/12 text-[var(--color-text)] border border-[var(--color-border-strong)]`
  - Default: `text-[var(--color-muted)] hover:bg-[var(--color-surface-2)]`
- `<` `>` arrows: same size, ghost style

---

## Component List

```
// src/app/radar/change-requests/page.tsx
<RadarLayout user={user}>
  <div className="flex items-start justify-between mb-5">
    <PageTitle title="Change Requests" />
    <Button variant="primary">+ New Request</Button>
  </div>

  <StatusFilterTabs value={filter} onChange={setFilter}>
    <Tab value="all" count={18}>All</Tab>
    <Tab value="pending" count={8}>Pending</Tab>
    <Tab value="approved" count={6}>Approved</Tab>
    <Tab value="declined" count={3}>Declined</Tab>
  </StatusFilterTabs>

  <Toolbar className="mb-4">
    <SearchInput />
    <Dropdown label="All Games" />
    <Dropdown label="All Fields" />
    <SortDropdown />
    <FilterButton />
  </Toolbar>

  <Card className="overflow-hidden">
    <Table>
      <TableHeader />
      <TableBody>
        {requests.map(r => <ChangeRequestRow key={r.id} request={r} />)}
      </TableBody>
    </Table>
    <Pagination total={18} page={1} perPage={10} />
  </Card>
</RadarLayout>
```

---

## Open Questions / TBD

- [ ] Clicking `>` chevron — opens side panel detail or navigates to `/radar/change-requests/:id`?
- [ ] `+ New Request` — who can create? Only developer for their own games, or anyone?
- [ ] "All Fields" filter options — derived from `radar_change_requests.field_name` distinct values?
- [ ] Inline approve/decline — confirmation tooltip/popover before committing?
- [ ] Approved requests — auto-applied to `radar_games` fields or requires manual step?
- [ ] `math_owner_id` field rows — should show resolved names in Old/New Value columns
