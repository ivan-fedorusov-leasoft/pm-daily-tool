# Frontend Spec — Write Note Modal

> Source: `today work - write note.png`  
> App: **BE Radar** (inside gc-pm-automation)  
> Trigger: "Write Status" button on any game card in Today's Work  
> Design system: see [design-system.md](../design-system.md)

---

## Behavior

- Opens as a **centered modal overlay** over the current page
- Background page dimmed with scrim + optional blur
- Closing: `✕` button, "Cancel" button, or `Escape` key
- Focus trap inside modal while open

---

## Overlay

```css
background: rgba(0, 0, 0, 0.6);
backdrop-filter: blur(4px);
position: fixed;
inset: 0;
z-index: 50;
display: flex;
align-items: center;
justify-content: center;
```

---

## Modal Container

```tsx
<div className="w-full max-w-lg rounded-[var(--radius-card)] border border-[var(--color-border)]
  bg-[var(--color-surface)] shadow-[0_24px_64px_-12px_rgba(0,0,0,0.8)] p-6 animate-in" />
```

- **Width:** `max-w-lg` (512px)
- **Max-height:** `90vh`, `overflow-y-auto`
- **Border-radius:** `16px` (`--radius-card`)
- **Background:** `--color-surface`
- **Shadow:** deep drop shadow

---

## Modal Header

```
Write Note                                    [✕]
```

- Title: `text-lg font-semibold text-[var(--color-text)]`
- `✕` close: `<Button variant="subtle" size="sm">` — top-right, 24×24px, hover `text-[var(--color-err)]`
- `mb-5`

---

## Game Context Card

```
┌──────────────────────────────────────────────────────┐
│  [img 48px]  Cooking Diary   ● Active                │
│              Current stage                           │
│              Beta (May 12 – May 26)                  │
└──────────────────────────────────────────────────────┘
```

```tsx
<div className="rounded-xl border border-[var(--color-border)] bg-[var(--color-surface-2)] p-4 mb-5 flex items-center gap-3" />
```

- Image: `h-12 w-12 rounded-xl object-cover`
- Game name: `text-base font-medium text-[var(--color-text)]`
- Status: `<Badge tone="ok">Active</Badge>` — inline, ml-2
- "Current stage" label: `text-xs text-[var(--color-muted)] uppercase tracking-wider mt-1.5`
- Stage value: `text-sm font-medium text-[var(--color-text)]` — "Beta (May 12 – May 26)"

---

## Note Input Section

### Label
```
Note
```
`text-xs font-medium text-[var(--color-muted)] uppercase tracking-wider mb-1.5`

### Textarea

```tsx
<textarea
  className="w-full rounded-xl border border-[var(--color-border)] bg-[var(--color-surface-2)]
    px-3 py-2.5 text-sm text-[var(--color-text)] placeholder:text-[var(--color-muted)]
    focus:border-[var(--color-accent)] focus:outline-none resize-y min-h-40"
  placeholder="Write your note here..."
  maxLength={2000}
/>
```

- Min-height: `160px` (`min-h-40`)
- Resize: vertical only
- Focus: border color → `--color-accent`

### Character Counter
```
0/2000
```
- `text-xs text-[var(--color-faint)] text-right mt-1`
- Color shifts to `text-[var(--color-warn)]` when > 1800 chars

---

## Flag Checkbox

```
□  Need team lead attention
```

```tsx
<label className="flex items-center gap-2.5 mt-4 text-sm text-[var(--color-text)] cursor-pointer">
  <input
    type="checkbox"
    className="h-4 w-4 rounded border-[var(--color-border)] bg-[var(--color-surface-2)]
      accent-[var(--color-warn)]"
  />
  Need team lead attention
</label>
```

- Unchecked: `border border-[var(--color-border)] bg-[var(--color-surface-2)]`
- Checked: `accent-[var(--color-warn)]` — browser-native amber checkbox
- Label text color: `text-[var(--color-text)]`
- As seen in screenshot: border becomes amber when checked — use CSS custom checkbox or `accent` property

---

## Action Buttons

```
[     Cancel     ]       [   Save Note   ]
```

```tsx
<div className="flex justify-end gap-3 mt-6">
  <Button variant="ghost" onClick={onClose}>Cancel</Button>
  <Button variant="primary" onClick={handleSave} disabled={!noteText.trim()}>
    Save Note
  </Button>
</div>
```

- Cancel: `<Button variant="ghost">` — border, transparent bg, hover surface-2
- Save Note: `<Button variant="primary">` — accent bg, disabled when textarea empty
- Disabled: `opacity-50 pointer-events-none`
- Loading state: spinner replaces text, disabled

---

## Full Modal Anatomy

```
┌─────────────────────────────────────────────────────┐  ← rounded-[16px] border surface shadow
│  Write Note                                      [✕] │
│                                                       │
│  ┌─────────────────────────────────────────────────┐ │  ← rounded-xl border surface-2
│  │ [img]  Cooking Diary  ● Active                  │ │
│  │        Current stage                            │ │
│  │        Beta (May 12 – May 26)                  │ │
│  └─────────────────────────────────────────────────┘ │
│                                                       │
│  NOTE                                                 │
│  ┌─────────────────────────────────────────────────┐ │  ← rounded-xl border surface-2
│  │  Write your note here...                        │ │
│  │                                                 │ │
│  │                                              ↘  │ │
│  └─────────────────────────────────────────────────┘ │
│                                             0/2000    │
│                                                       │
│  □  Need team lead attention                          │
│                                                       │
│                         [Cancel]  [Save Note]         │
└─────────────────────────────────────────────────────┘
```

---

## Component

```tsx
<Modal open={isOpen} onClose={onClose} title="Write Note">
  <GameContextCard
    image={game.image}
    name={game.title}
    status={game.status}
    currentStage={game.currentStage}
    stageDateRange={stageDateRange}
  />

  <FormField label="Note">
    <Textarea
      value={noteText}
      onChange={setNoteText}
      placeholder="Write your note here..."
      maxLength={2000}
      showCounter
    />
  </FormField>

  <Checkbox
    label="Need team lead attention"
    checked={flagTeamLead}
    onChange={setFlagTeamLead}
  />

  <div className="flex justify-end gap-3 mt-6">
    <Button variant="ghost" onClick={onClose}>Cancel</Button>
    <Button
      variant="primary"
      onClick={() => handleSave({ noteText, flagTeamLead })}
      disabled={!noteText.trim()}
    >
      Save Note
    </Button>
  </div>
</Modal>
```

**On save:** creates a new `daily_notes` row:
```
{
  game_id: game.id,
  author_id: currentUser.id,
  date: today,
  summary: noteText,
  needs_lead_attention: flagTeamLead,
  source: 'manual'
}
```

---

## Open Questions / TBD

- [ ] After save — optimistic update of "Last Note" on the card without page reload?
- [ ] After save with flag — does warning badge on card update immediately?
- [ ] Are notes append-only or editable after saving?
- [ ] One note per day per game, or unlimited?
- [ ] 2000 char limit — hard block or just counter warning?
- [ ] Should modal be a `<dialog>` element or portal `<div>`?
