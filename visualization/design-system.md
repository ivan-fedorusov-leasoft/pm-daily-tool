# Design System Reference — BE Radar

> **ADR-007:** BE Radar builds inside **gc-games-dashboard**, not gc-pm-automation. This document reflects gc-games-dashboard's design system.
>
> Authoritative source: `gc-games-dashboard/src/app/globals.css` + raw Tailwind patterns in components.

---

## CSS Variables

Defined in `globals.css` under `:root` and exposed as Tailwind custom colors via `@theme inline`:

| CSS Var | Tailwind token | Hex | Semantic role |
|---|---|---|---|
| `--background` | `bg-background` | `#07090d` | Page bg |
| `--panel` | `bg-panel` | `#0d1117` | Card / panel bg (`.panel` class) |
| `--panel-2` | `bg-panel-2` | `#11161f` | Nested panel, input bg |
| `--border` | `border-border-soft` | `#1d2430` | All borders, dividers |
| `--foreground` | `text-foreground` | `#e6edf3` | Primary text |
| `--muted` | `text-muted` | `#8b97a8` | Secondary text, labels, metadata |

No accent/ok/warn/err vars — status colors are Tailwind semantic classes (emerald/amber/rose/violet).

---

## Background

```css
body {
  background:
    radial-gradient(1200px 600px at 80% -10%, rgba(56, 189, 248, 0.06), transparent),
    radial-gradient(900px 500px at -10% 10%, rgba(244, 114, 182, 0.05), transparent),
    var(--background);
  color: var(--foreground);
}
```

Dark, near-black with subtle sky-blue and pink radial glows at the corners.

---

## Utility classes

### `.panel`
```css
.panel { background: var(--panel); border: 1px solid var(--border); }
```
Used for all cards, tables, panels. Always pair with `rounded-xl` or `rounded-lg`.

### `.mono`
```css
.mono { font-family: var(--font-mono), ui-monospace, "SF Mono", monospace; }
```
Used for game codes, hash values, technical strings.

### `.pulse-down`
```css
animation: pulse-down 2.4s ease-in-out infinite;
/* rose glow pulse — used on "down" status indicators */
```

---

## Fonts

- **Sans:** Geist (loaded via `next/font/google`, var `--font-geist-sans`)
- **Mono:** Geist Mono (var `--font-geist-mono`)

```tsx
// In layout.tsx:
const geistSans = Geist({ variable: "--font-geist-sans", subsets: ["latin"] })
const geistMono = Geist_Mono({ variable: "--font-geist-mono", subsets: ["latin"] })
```

---

## Layout

gc-games-dashboard has **no sidebar, no persistent nav**. Pages are centered content blocks:
```tsx
<div className="mx-auto w-full max-w-6xl flex-1 px-4 py-8 sm:px-6 sm:py-12">
```

**For BE Radar integration:** a shared layout must be added — either a top nav bar or left sidebar — scoped to `/daily/*` routes via `src/app/daily/layout.tsx`.

Recommended approach: **top nav bar** (fits the existing aesthetic better than a sidebar — gc-games-dashboard is content-first, no chrome-heavy sidebar).

---

## Typography

| Use | Classes | Size |
|---|---|---|
| Page title (`<h1>`) | `text-xl font-semibold tracking-tight sm:text-2xl` | 20–24px |
| Section heading | `text-sm font-semibold uppercase tracking-wide text-muted` | 12px caps |
| Card title / game name | `font-semibold` | 16px |
| Body | `text-sm` | 14px |
| Metadata / labels | `text-xs text-muted` | 12px |
| Monospace code | `.mono text-sm` | 14px mono |

---

## Buttons (no component — raw Tailwind)

gc-games-dashboard has no `Button` component. Copy these patterns:

```tsx
// Default (ghost-style):
className="rounded-lg border border-border-soft bg-panel-2 px-3 py-2 text-sm 
           transition hover:border-slate-500/60 disabled:opacity-50"

// Primary (sky accent):
className="rounded-lg border border-sky-500/40 bg-sky-500/10 px-3 py-2 text-sm 
           font-medium text-sky-200 transition hover:bg-sky-500/20 disabled:opacity-50"

// Destructive:
className="rounded-lg border border-rose-500/30 bg-rose-500/10 px-3 py-2 text-sm 
           font-medium text-rose-300 transition hover:bg-rose-500/20 disabled:opacity-50"
```

---

## Inputs

```tsx
className="rounded-lg border border-border-soft bg-panel-2 px-3 py-2 text-sm 
           outline-none placeholder:text-muted focus:border-sky-500/50"
```

---

## Status colors

gc-games-dashboard uses these for game monitoring statuses. BE Radar will reuse the same color conventions for different semantic meanings.

| Status / Tone | Dot | Text | Ring/Border | Badge bg |
|---|---|---|---|---|
| `operatable` / success / ok | `bg-emerald-400` | `text-emerald-300` | `border-emerald-500/30` | `bg-emerald-500/10` |
| `merged_not_updated` / warning / pending | `bg-amber-400` | `text-amber-300` | `border-amber-500/30` | `bg-amber-500/10` |
| `down` / error / rejected | `bg-rose-400` | `text-rose-300` | `border-rose-500/30` | `bg-rose-500/10` |
| `not_setup` / info / accent | `bg-violet-400` | `text-violet-300` | `border-violet-500/30` | `bg-violet-500/10` |
| `unknown` / muted / default | `bg-slate-500` | `text-slate-400` | `border-slate-600/40` | `bg-slate-500/10` |
| UPDATED badge (bumped) | — | `text-sky-200` | — | `bg-sky-500/20` |

---

## BE Radar status → color mapping

| BE Radar concept | Color |
|---|---|
| Game `active` | emerald (ok) |
| Game `planned` | violet (info) |
| Game `paused` / `cancelled` | slate (muted) |
| Game `completed` | emerald (ok) |
| CR `pending` | amber (warning) |
| CR `approved` | emerald (ok) |
| CR `rejected` | rose (error) |
| Stage completed ✓ | emerald |
| Stage current ● | sky-200 |
| Stage future ○ | slate-600 |

---

## Panels / Cards

```tsx
// Standard panel:
<div className="panel rounded-xl px-4 py-4">

// Table panel:
<div className="panel overflow-hidden rounded-xl">
  <div className="overflow-x-auto">
    <table className="w-full text-sm">

// Section heading inside page:
<h2 className="mb-3 text-sm font-semibold uppercase tracking-wide text-muted">
```

---

## Badges (no component — inline spans)

```tsx
// emerald ok:
<span className="rounded bg-emerald-500/15 px-1.5 py-0.5 text-[10px] font-semibold text-emerald-300">OK</span>

// amber warning:
<span className="rounded bg-amber-500/15 px-1.5 py-0.5 text-[10px] font-semibold text-amber-300">PENDING</span>

// rose error:
<span className="rounded bg-rose-500/15 px-1.5 py-0.5 text-[10px] font-semibold text-rose-300">REJECTED</span>

// slate muted:
<span className="rounded bg-slate-500/15 px-1.5 py-0.5 text-[10px] font-medium text-slate-400">muted label</span>

// sky info/updated:
<span className="rounded bg-sky-500/20 px-1.5 py-0.5 text-[10px] font-semibold text-sky-200">UPDATED</span>
```

---

## Empty / loading states

```tsx
// Empty:
<div className="panel rounded-xl px-4 py-6 text-center text-sm text-muted">
  Nothing here yet.
</div>

// Error/degraded banner:
<div className="rounded-xl border border-amber-500/30 bg-amber-500/10 px-4 py-3 text-sm text-amber-200">
  Warning message
</div>
```

---

## Scrollbar

```css
::-webkit-scrollbar { width: 10px; height: 10px; }
::-webkit-scrollbar-thumb { background: #232b36; border-radius: 6px; }
::-webkit-scrollbar-track { background: transparent; }
```
