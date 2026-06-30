# Design System Reference — BE Radar

> Authoritative source: `gc-pm-automation/app/globals.css` + `app/ui/primitives.tsx` + `app/ui/AppShell.tsx`
>
> BE Radar (pm-daily-tool) builds inside the existing gc-pm-automation app and **inherits its design system entirely**. No second design system, no new Tailwind theme. All tokens below are defined in `@theme` in `globals.css`.

---

## CSS Variables (Tailwind v4 `@theme`)

| CSS Var | Hex | Semantic role |
|---|---|---|
| `--color-bg` | `#07080c` | Page background |
| `--color-surface` | `#0d0f17` | Card / panel background |
| `--color-surface-2` | `#131725` | Card section / nav active / input bg |
| `--color-border` | `#1f2435` | Card borders, dividers, input borders |
| `--color-border-strong` | `#2c3350` | Stronger borders (e.g. active item outline) |
| `--color-text` | `#e7e9f3` | Primary text |
| `--color-muted` | `#8a90a8` | Secondary text, labels, placeholders |
| `--color-faint` | `#565d78` | Very muted — hints, timestamps, counters |
| `--color-accent` | `#6366f1` | Interactive — links, primary buttons, active states |
| `--color-accent-soft` | `#312e81` | Accent background (subdued) |
| `--color-ok` | `#34d399` | Success, active status, approved, merged |
| `--color-warn` | `#fbbf24` | Warning, pending, needs-attention |
| `--color-err` | `#f87171` | Error, declined, danger |
| `--radius-card` | `16px` | Card border-radius |
| `--accent` | runtime | Per-stream accent, injected by AppShell on `<body>`. Falls back to `#6366f1` |

---

## Background

```css
body {
  background:
    radial-gradient(1100px 600px at 80% -10%, color-mix(in oklab, var(--accent) 16%, transparent), transparent 60%),
    radial-gradient(900px 500px at 0% 0%,  color-mix(in oklab, var(--accent)  8%, transparent), transparent 55%),
    var(--color-bg);
}
```

Subtle gradient glow in top-right, softer glow top-left, deep dark base. The sidebar uses the `glass` utility:
```css
.glass {
  background: color-mix(in oklab, var(--color-surface) 80%, transparent);
  backdrop-filter: blur(12px);
  border: 1px solid var(--color-border);
}
```

---

## Core Components (from `primitives.tsx`)

### Card
```tsx
<div className="rounded-[var(--radius-card)] border border-[var(--color-border)] bg-[var(--color-surface)]
  shadow-[0_1px_0_rgba(255,255,255,0.03)_inset,0_20px_50px_-30px_rgba(0,0,0,0.8)]" />
```
- Border-radius: **16px**
- Background: `--color-surface`
- Border: `--color-border`
- Inset top highlight + deep drop shadow

### Button
```tsx
// Base: rounded-xl h-10 px-4 text-sm font-medium + transitions
// Size sm: h-8 px-3 text-xs
```

| Variant | Style |
|---|---|
| `primary` | `bg-[var(--accent)] text-white hover:brightness-110 shadow-accent-glow` |
| `ghost` | `bg-transparent border border-[var(--color-border)] hover:bg-[var(--color-surface-2)]` |
| `subtle` | `text-[var(--color-muted)] hover:text-text hover:bg-[var(--color-surface-2)]` |
| `danger` | `text-[var(--color-err)] border border-[var(--color-err)]/30 hover:bg-[var(--color-err)]/10` |

- Border-radius: `rounded-xl` = **12px**
- All buttons: `disabled:opacity-50`, `active:scale-[0.98]`, focus ring accent/50

### Badge
```tsx
// rounded-full border px-2.5 py-0.5 text-xs font-medium
```

| Tone | Background | Text | Border |
|---|---|---|---|
| `default` | `--color-surface-2` | `--color-muted` | `--color-border` |
| `accent` | `--color-accent`/15 | `--color-accent` | `--color-accent`/30 |
| `ok` | `--color-ok`/12 | `--color-ok` | `--color-ok`/25 |
| `warn` | `--color-warn`/12 | `--color-warn` | `--color-warn`/25 |

> **No `err` tone in primitives** — for "Declined" / "On Hold" use a custom class:  
> `bg-[var(--color-err)]/12 text-[var(--color-err)] border-[var(--color-err)]/25`

### PageTitle
```tsx
<h1 className="text-2xl font-semibold tracking-tight" />
<p className="mt-1 text-sm text-[var(--color-muted)]" />  // subtitle
```
- Title: **24px / 600 / tracking-tight**
- Subtitle: **14px / muted**

### EmptyState
```tsx
<div className="rounded-[var(--radius-card)] border border-dashed border-[var(--color-border)] py-16 text-center" />
```

---

## AppShell (from `AppShell.tsx`)

### Sidebar
```tsx
<aside className="sticky top-0 h-dvh w-64 glass px-4 py-5" />
```
- Width: **256px** (`w-64`)
- Position: sticky, full viewport height
- Style: `glass` (backdrop-blur + semi-transparent surface)
- Right border: `1px solid var(--color-border)`

### Logo block
```tsx
<span className="h-9 w-9 rounded-xl bg-[var(--accent)]/15 text-lg grid place-items-center">🎛️</span>
// App name: text-sm font-semibold
// Subtitle (stream): text-xs text-muted
```
> For BE Radar: icon changes to radar SVG, name changes to "BE Radar". Same dimensions and structure.

### Nav items
```tsx
// Active:   bg-[var(--accent)]/12 text-[var(--color-text)]   + icon text-accent
// Inactive: text-[var(--color-muted)] hover:bg-surface-2 hover:text-text
// Shape:    rounded-xl px-3 py-2 text-sm
```

### UserCard (bottom)
```tsx
<div className="rounded-xl border border-[var(--color-border)] bg-[var(--color-surface-2)] p-3" />
// Avatar: h-9 w-9 rounded-full
// Fallback: accent/20 bg, initial letter
// Role: Badge tone="accent"
// Sign out: text-faint hover:text-err
```

### Main content area
```tsx
<div className="mx-auto max-w-5xl px-5 py-8 md:px-10 md:py-10" />
```
- Max width: **1024px** (max-w-5xl)
- Padding: 20px mobile, 40px desktop

---

## Game Status → Badge Tone Mapping

| Status | Tone | CSS |
|---|---|---|
| Active | `ok` | `--color-ok` green |
| Planned | `accent` | `--color-accent` purple |
| In Progress | `warn` | `--color-warn` amber |
| On Hold | custom err | `--color-err` red |
| Paused | `default` | muted |
| Completed | `default` | muted |

## Change Request Status → Badge Tone

| Status | Tone |
|---|---|
| Pending | `warn` |
| Approved | `ok` |
| Declined/Rejected | custom err |

## PR Status → Badge Tone

| Status | Tone |
|---|---|
| In Review | `accent` |
| Merged | `ok` |
| Closed | custom err |

---

## Typography Scale (reference)

| Use | Tailwind | Size |
|---|---|---|
| Page title (`<h1>`) | `text-2xl font-semibold tracking-tight` | 24px/600 |
| Section heading | `text-lg font-semibold` | 18px/600 |
| Card title | `font-medium` | 16px/500 |
| Body | `text-sm` | 14px/400 |
| Labels / metadata | `text-xs text-[var(--color-muted)]` | 12px/muted |
| Faint hints | `text-xs text-[var(--color-faint)]` | 12px/faint |

Font family: `ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, sans-serif`  
Anti-aliasing: `-webkit-font-smoothing: antialiased`

---

## Animate-in utility

```css
@keyframes fade-up {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}
.animate-in { animation: fade-up 0.4s cubic-bezier(0.22, 1, 0.36, 1) both; }
```

Use `animate-in` on cards/panels that appear after navigation.

---

## Scrollbar

```css
scrollbar-width: thin;
scrollbar-color: var(--color-border-strong) transparent;
```
