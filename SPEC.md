# Projector — Spec Sheet

> A local-first, no-install web app for filling **Projectile** timesheets without the friction.
> The day is a single gapless track of time blocks you label with project positions, edited
> like a video timeline (cut / merge / drag), then transferred into Projectile.
>
> **Name:** *Projector* — the same joke as *Projectile* (project + suffix), minus the
> ordnance: it projects your day onto a timeline before you aim it at Projectile.

Status: **v1 built** · Owner: Felix · Last updated: 2026-07-26

---

## 1. Goal & success criteria

**Goal:** Make daily Projectile entry fast and low-recall, without installing anything or
letting sensitive data leave the machine.

**Success looks like:**
- Filling a day takes ~1 minute of cleanup, not 10 minutes of recall.
- Export pastes into Projectile with zero reformatting.
- No data leaves the device; no install.
- Felix prefers it to typing into Projectile directly.

**Non-goals (v1):** team/sync, reporting dashboards, automated activity capture, label
inference, mobile-native app.

---

## 2. Core model — a single gapless track

The mental model is a **video-editing track**, not a calendar:

- The day is **one lane**. Blocks tile edge-to-edge; **no overlaps, and within a working
  session, no gaps** — every minute on the clock belongs to exactly one block.
- A block is `work-on-a-position` or a **break**, and a break is just the reserved position
  **"Pause"** — otherwise identical to any other block. There is *no* separate "gap" or
  "off-task" concept inside a session.
- **Off-clock time** = time *not covered by any block*, i.e. before a Start-day, after a
  Stop-day, or between a Stop and the next Start. This is the only empty time and it never
  exports. (This is how *time off* differs from a *break*.)

Because the lane is gapless, **every boundary edit ripples** (one block grows, its neighbour
shrinks). There is no gap-creating edit to design.

### Editing grammar (the whole thing reduces to two gestures)
- **Scissors on a block → split** it in two at the cut point.
- **Merge on a seam → join** the two blocks that seam divides.

Everything else is these two plus relabeling:
- *Switch task now* = split at the playhead, relabel the new piece (the live "Switch" button).
- *Fix a forgotten switch* = split at the past moment, relabel.
- *Undo an accidental cut* = merge the seam.
- *Delete a block* = merge its seam into a neighbour.

**Merge label resolution:** when two merged blocks differ, the **longer** block's
position + description wins; a quick two-option pick is offered when they differ. Same-label
merges are silent.

---

## 3. Data model

### Block (the primitive)
```jsonc
{
  "id": "uuid",
  "start": 1718607780000,   // epoch ms, local
  "end":   1718612100000,   // epoch ms; null while this is the running block
  "positionId": "pos_123",  // FK to Position; null = unlabeled (must resolve before export)
  "description": "Reviewed PR #42",
  "wg": null,               // Themenarbeit only: Hub tag, null = take the row's own
  "prj": null               // Themenarbeit only: PEG tag, null = take the row's own
}
```
- The **running block** is the one with `end === null`. At most one at a time. It visually
  extends to "now."
- Blocks within a session abut exactly: `block[i].end === block[i+1].start`.
- All the export needs is here: **begin / end / duration (any one inferable) / position /
  description.** Nothing else is required, so the data model is considered stable.

### Position (the bookable target)
```jsonc
{
  "id": "pos_123",
  "code": "PRJ-001-DEV",   // the identifier Projectile books on (exact format TBD, Phase 0)
  "label": "Project Alpha — Development",
  "color": "#5e7ce0",
  "archived": false,
  "reserved": false,        // true only for the built-in "Pause"
  "hotkey": 2,              // 1-9, unique, stable — the keyboard slot (§4.3)
  "parentId": "pos_099",    // null = top level (a project); set = a work package
  "billable": null,         // true | false | null = inherit from parent (§4.4a)
  "tagged": false,          // true = Themenarbeit; sub-positions become Hub / PEG rows
  "wg": null, "prj": null   // on a Hub / PEG row: the tags it gives every block booked on it
}
```
- **"Pause"** is a reserved, built-in position: cannot be deleted or renamed, has its own
  icon, and is what both the **Break** button and the **compliance engine** reference.

**Two orthogonal axes** (deliberately two fields, not one):
1. **Hierarchy** — `parentId`, exactly **two levels**: top-level entries are *projects*
   (containers), their children are *work packages*. **Bookable = has no children**, so a
   childless top-level position is itself bookable and needs no dummy child. Containers never
   appear in a picker and hold no hotkey. A third level is refused.
2. **Billability** — `billable`, inherited from the parent when `null`, overridable per
   package. They must stay separate because a *factura* project can legitimately contain a
   non-billable package (rework, warranty, goodwill) — a single combined field can't express
   that. Unset at the root resolves to **internal**: under-claiming billable time is the safe
   direction. *Pause* is neither factura nor internal.

**Themenarbeit (a third axis, opt-in per booking code).** Internal work is booked on a few
broad codes, and the unit's own system reads the *description* to tell that work apart. It
must arrive in a fixed shape:

```
<Hub> | <PEG or None> | free text
```

A position marked `tagged` is such a code. Its sub-positions then stop being booking codes
of their own and become **Hub rows and PEG rows**: they book on the parent and contribute
only the two tags. This is deliberate reuse rather than a parallel mechanism — a Hub row is
an ordinary position, so hotkeys, quick-pick, colour inheritance and the week rollup all
keep working, and the tag vocabulary *is* the tree instead of a second list to maintain.

Because there is no third level to hang a PEG under its Hub, a **PEG row carries both tags**
(`wg` *and* `prj`) as a flat sibling of the Hub row whose `wg` it repeats; the manager
merely draws it indented. Blocks inherit both tags from their row and may override either,
the same shape as colour and billability — so renaming a tag also fixes every block already
booked on it. On export, both position columns resolve **upward** to the nearest real
booking code, and two blocks merge into one row only when their tags agree as well, since a
row states its tags exactly once.

A Themenarbeit code with *no* sub-positions keeps the format and leaves both tags to be
typed on the block — the escape hatch for work that has no standing Hub row yet.

**Vocabulary.** `Themenarbeit`, `Hub` and `PEG` are the unit's own words, kept verbatim in
both languages rather than translated. The word *project* was deliberately not reused for
the second tag: it already names the hierarchy's top level and Projectile's own export
column, and a third meaning made the positions manager unreadable.

Work packages inherit a lighter shade of their project's colour, so a project reads as one
visual family on the timeline.
- Manually maintained; importable/exportable as JSON (seed once, reuse forever).

**Half days.** `dayTypes[iso]` holds either the bare type string (a whole day, the shape
every store written before half days used) or `{t,f}` once a fraction is involved.
`dayType()` and `dayOff()` hide which of the two a caller has, so the target arithmetic —
`dayTargetMs = target × (1 − dayOff)` — is the only place the fraction appears. A half day
is still *expected*: it owes half a target and should still have something tracked on it.

### Persistence envelope (backup / restore)
```jsonc
{ "version": 1, "positions": [...], "blocks": [...], "settings": {...} }
```

---

## 4. Features — v1

### 4.1 The live loop
The four verbs, and what they actually mean (surfaced in-app via the Help panel, button
tooltips and a hint under the Now card — this model is not self-evident and users need it):
- **Start day** → creates the running block at now. From here on, **every minute belongs to a
  block** until you Stop.
- **Switch** → closes the current block at now, opens a new running block (gapless).
- **Break** → same as Switch, but the new block uses the reserved position **Pause**. Breaks
  **are transferred to Projectile** like any other row, and count toward the legal break rule.
- **Stop** → closes the current block; the clock is now off. **Time after a Stop is never
  transferred** — that is precisely how *time off* differs from a *break*.
- **Start again** later → a new session the same day (e.g. a doctor's appointment); the time
  between Stop and the next Start stays off-clock.
- Running block ticks live and is visible across reloads (recomputed from `start`).
- **Overnight safety net:** on load, if a block has been running past a threshold (machine
  slept / app left open), prompt to trim its end rather than booking the gap. This is the
  *only* role presence/idle plays — it never writes a block, only asks.

### 4.2 The day track (view + edit)
- Vertical day timeline; blocks coloured by position; "now" line on today.
- **Scissors / split** at a cut point; **merge** on a seam (§2).
- **Drag a seam** to move a shared boundary (ripples both neighbours).
- **Drag a session edge** to adjust Start/Stop.
- **Draw on empty time** (click-drag) to create a block — for rebuilding past days.
- **Snap** edits to a grid (default 15 min, configurable).
- **Marker (playhead)** — the hour-label strip is a visible **scrub rail** (row-resize cursor,
  hover preview line); the marker can also be grabbed **anywhere along its own line**, and
  nudged with ↑/↓. `s` splits the block under it.
- Precise **start/end time fields** on the selected block as the dependable editor
  (plain 24 h `HH:MM` text, never locale-dependent `type=time`).
- **Unlabeled blocks are flagged** — they hold time but no position; export refuses until
  resolved.

### 4.3 Labeling — keyboard-first
Blocking out time stays a mouse job (dragging is good at it). **Labeling is keyboard-first**,
optimised as an inbox-zero loop over the day's unlabeled blocks.

- **Position hotkeys `1`–`9`** — `hotkey` is a **stable property of the Position**, never
  derived from sort order: a frequency-sorted mapping would silently renumber itself and turn
  muscle memory into mis-bookings. Auto-assigned to the first free slot (real projects before
  the reserved *Pause*), editable in the Positions manager, and **unique** — reassigning a
  taken slot releases it from the previous holder.
- **Visible legend** — numbered position chips in the editor card (clickable, for mouse
  parity) plus an *"n left without a position"* counter. No invisible affordances.
- There is **no "favourite" flag**: the hotkey *is* the favourite, and it is explicit,
  visible and stable. Picker order follows the hotkey slots, so the list matches the chips.
- **The loop:** `n` → next unlabeled block · `1`–`9` assign (**stays put**, so a description
  is still possible) · `0` clear · `Enter` edit description · `Enter` commit & exit ·
  `⌘`/`Alt`+`Enter` commit & jump to the next unlabeled · `Tab`/`Shift+Tab` next/previous
  block. Digits typed *inside* a description never assign a position.
- **Quick-pick (`p`)** replaces the native `<select>`, which traps keystrokes (the reason `l`
  used to stop working after `p`): a filter field with ↑/↓, Enter to pick, hotkeys, Esc.
- Description **autocomplete from past descriptions**. No inference, just sorting.

### 4.4 Totals, target & break compliance
- Per-position totals, **total work** (Σ non-Pause), **total Pause**, off-clock excluded.
- Configurable daily **target** (default 8h) with a progress hint.
- **Compliance engine** (German ArbZG). The break must be **completed by the time work
  reaches the threshold**, not summed at day's end: **30 min taken by 6h** of work, **45 min
  by 9h**; breaks count only in **chunks ≥15 min**. Checked **chronologically** — working 9h
  straight and booking a 45-min break at the end is a *violation*. Warn live and gate export.
- **Factura / Intern split** — a single glanceable bar plus totals and percentages, in both
  the day totals and the week rail. *Pause* is excluded from both sides.

### 4.5 Week view (read-only) + stats
- **Day / Week** toggle (`w`). Week uses **ISO weeks (Mon–Sun) with KW numbers**; all date
  arithmetic is DST-safe (never `+86400000`).
- **7-day strip** — read-only overview of the week's blocks in position colours, sharing one
  time window. Click any day (column or header) to open it in the day view for editing.
  Editing stays day-only by design: the day view already handles it and keeping the week
  read-only avoids threading column-awareness through every drag gesture.
- **Stats rail:**
  - *Week summary* — total work vs. a configurable weekly target (default 40 h), over/under,
    progress bar, total Pause, and a flag for unlabeled blocks.
  - *By position* — hours per position with share bars and percentages (Pause excluded from
    the work split; unlabeled time is excluded from work, as in the day totals).
  - *By position* — rolled up **by project**, with the project's work packages listed
    beneath it; share bars and percentages are computed on the project total.
  - *Per day* — work/Pause bar per weekday plus a per-day break-compliance flag.
- **Today** is highlighted (column + header); **weekends** are shaded slightly differently.

### 4.6 Help overlay, theming & narrow screens
- **`?` opens a Gmail-style Help panel** (also a `?` button in the header, and a link on the
  Now card). Two parts: **How tracking works** (the §4.1 verbs explained in plain language)
  and **Keyboard shortcuts**, grouped into General / Marker & cutting / Selected block /
  Export dialog. Suppressed while typing in a field or while another dialog is open.
- Navigation keys: `←`/`→` previous/next day (week in week view), `t` today, `w` day/week.
- **Theming** — the user sets three colours (**background, text, accent**); every other token
  (`--card`, `--card2`, `--line`, `--dim`, `--accent-bright/-hover/-on-accent`, tinted
  surfaces) is *derived* by mixing along the bg↔text axis, so a light background yields a
  light theme automatically and no combination can render text illegible.
  `--dim` uses an **asymmetric mix weight** (0.30 on light backgrounds, 0.40 on dark) because
  perceptual luminance is non-linear; all five presets clear **WCAG AA (≥4.5)** on
  text-on-bg, dim-on-card and on-accent. Presets: Olive (default), Slate, Plum, Ember, Paper
  (light). Reset restores the default. Status colours (focus/break/rest) stay fixed —
  they're semantic.

**Narrow screens (≤820 px)** — mobile is not first-class but must be usable:
- The two panes become **one at a time**, switched by a header toggle; tapping a block also
  reveals the rail, since the editor lives there. The timeline keeps its own scroll.
- The week grid drops to a 34 px gutter and flexible columns so all seven days fit with **no
  horizontal page scroll** anywhere.
- The mobile overrides are the **last** rules in the stylesheet: media queries add no
  specificity, so source order is what makes them win.

**Long names** — real project and work-package names are long, so a block's label is two
spans: the *project* prefix carries `flex-shrink: 9999` and the work package `1`, meaning the
project gives up all its width before the package loses a character. Hotkey chips show the
short `code` instead, with the full path in the tooltip.

### 4.7 Positions management
- A **grid with column headers** — key · code · name · project · billing · colour · archive —
  rather than a cramped single flex row. Work packages are indented under their project.
- Containers show no key slot; "Pause" is built-in and locked.
- Import/export positions as JSON.

### 4.8 Persistence & backup
- Primary store: **localStorage** for v1 (small data; swap to IndexedDB later behind the
  same storage interface).
- One-click **JSON backup export / import** (the §3 envelope).
- UI note that data lives in the browser; nudge periodic backup.

### 4.9 Export
- **Calendar (.ics)** — each block → a VEVENT (`summary = position label · description`,
  start/end). Reuses the pomodoro app's exporter. **In v1.**
- **Projectile** — uniform rows `(date, start, end, duration, position, description)`,
  **consecutive same-position blocks merged** (Pause merges like anything else), rounded per
  §4.10. Position is emitted as its `code`; merged descriptions are joined and de-duplicated.
  **Whole-table paste is not supported by Projectile** (verified 2026-07), so the export is a
  **guided cell queue**: a preview table where clicking a cell (or pressing Enter) copies it
  and advances, so the loop is *paste → Tab → Enter*. Row-TSV and all-TSV copy remain as
  fallbacks. Columns, date format (`dd.mm.yyyy` / `yyyy-mm-dd`) and duration format
  (`HH:MM` / decimal) are **configurable**, since the exact field set is still being learned.
  Export **refuses** while any block is unlabeled and **warns** on a break-rule violation.
  Copy failures never advance the cursor.

### 4.10 Rounding
- Export rounds block boundaries to a configurable increment (default 15 min); show raw-vs-
  rounded so drift is visible. Editing snaps (§4.2) independently of export rounding.

---

### 4.11 Naming & header
Buttons are named for **what they do**, not what they are. Actions first, then configuration,
separated by a divider:

`Übertragen` · `Kalender-Export` │ `Positionen` · `Einstellungen` · `?`

| Was | Now | Why |
|---|---|---|
| `Projectile` | **Übertragen** (Transfer) | It transfers cells; it doesn't open Projectile |
| `Kalender` | **Kalender-Export** | It's an export, not a calendar view |
| `Daten` | **Einstellungen** | It's settings + theme + backup |
| `Positionen` | unchanged | already accurate |

Storage key is `projector-v1`; the pre-rename `projectile-buddy-v1` store is **migrated on
first load and kept as a safety net** so the rename can never orphan data.

---

## 5. UX shape

```
┌──────────────────────────────────────────────────────────────┐
│ Projectile Buddy        ‹ Tue 17 Jun ›  [today]   ✂ ⚙ ⤓ 📅   │
├──────────────────────────────┬───────────────────────────────┤
│  DAY TRACK (vertical)        │  NOW                          │
│   08 ┌───────────────┐       │   [position ▾] [description…] │
│      │ Admin         │       │   ⏱ 00:42  [Switch][Break]    │
│   09 ├───────────────┤ ←seam │            [ Stop day ]       │
│      │ Alpha · standup│      ├───────────────────────────────┤
│   10 ├───────────────┤       │  TOTALS                       │
│      │ Alpha · dev   │       │   Alpha   3:15                │
│   12 ├───────────────┤       │   Gamma   0:30                │
│      │ Pause · lunch │       │   Pause   0:45                │
│   13 ├───────────────┤       │   ─────────────               │
│      │ Alpha · dev   │       │   Work 6:45 / 8:00 · break ✓  │
│   …  └───────────────┘       │  [ Export 📅 ]  [ Projectile ▸ (soon) ]
└──────────────────────────────┴───────────────────────────────┘
```
Keyboard-first; every shortcut also has a visible control (see DESIGN.md for look/feel).

---

## 6. Out of scope (v2+)
Label inference · ambient/active-window capture · periodic pings (optional nudge, deferred) ·
recurring day templates · weekly summaries · multi-user · IndexedDB/File-System-Access
durability hardening.

---

## 7. Open questions / Phase-0 (needs the work system)
1. **Projectile import format** — exact columns/order, date & duration format, how a position
   is identified, whether a pasted TSV populates the table (or CSV/API). Gates 4.7-Projectile.
2. **Exact break rules** as Projectile enforces them (confirm the ArbZG tiers + chunk rule).
3. Decimal-hours vs HH:MM in the Projectile export.

---

## 8. Risks
| Risk | Mitigation |
|---|---|
| Projectile import format unknown | Phase 0 before building that exporter; data model already sufficient |
| Browser storage wiped | JSON backup now; IndexedDB + file autosave later |
| Block left running overnight | On-load staleness check + trim prompt (§4.1) |
| Merge of differing labels surprises user | Longer-wins + quick pick when they differ (§2) |
