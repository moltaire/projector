# 🟦🟨🟩 projector

[![GitHub Pages](https://img.shields.io/badge/GitHub-Pages-blue?logo=github)](https://moltaire.github.io/projector/)

Browser-based time tracker that prepares Projectile timesheets, for my personal use.
Single file, no build step, data stays in the browser. Vibe coded with [Claude](https://claude.ai).

## Quickstart

1. Open the [app](https://moltaire.github.io/projector/) — or `index.html` from anywhere you can serve it.
2. Set up your positions once (**Positionen** in the toolbar). See below.
3. Track the day: start, switch position, break, stop — or rebuild a forgotten day afterwards.
   The day is one gapless track of blocks, edited like a video timeline: split at the marker,
   merge at a seam, drag blocks and boundaries.
4. Press `n` to jump to the next block without a position, `1`–`9` to assign one, `↵` to describe it.
5. **Transfer to Projectile** walks you through the day cell by cell (paste → Tab → ↵).

`?` shows every shortcut. `w` cycles day / week / month.

## Positions

A position is what a block books on. They form two levels — **project → work package** — set
with the *Belongs to* column. A project that has packages stops being bookable itself; you book
on the packages. Colour and the **Factura / Intern** flag are inherited from the project and can
be overridden per package. Hotkeys `1`–`9` are assigned per bookable position.

**Themenarbeit** is the one non-obvious part. Tick it on a booking code whose description
Projectile parses as `Hub | PEG | Text` instead of free text. Its sub-positions then stop being
booking codes of their own and become **Hub and PEG rows**: they all book on the Themenarbeit
code, and picking one by hotkey fills both tags at once. The manager lays them out as a third
level — a Hub row, then the PEG rows repeating that Hub, indented. A Themenarbeit code *without*
sub-positions keeps the format, and you type the two tags on the block itself.

Positions can be archived rather than deleted, and exported / imported as their own JSON.

## Views

- **Day** — the timeline, per-block editing, and the Factura / Intern split.
- **Week** — 7-day strip, hours per project, per-day totals.
- **Month** — calendar grid with per-day hours and flags, month totals, per-week bars, leave
  tally, completeness check.

Days can be marked *Feiertag / Urlaub / Krank*, whole or half. A whole day drops out of the day,
week and month targets; a half day halves them.

The German break rule is enforced properly: 30 min *by* 6 h of work, 45 min by 9 h, in chunks
of ≥15 min, checked chronologically.

## Keyboard

| key | action |
|-----|--------|
| `n` | next block without a position (carries on into later days) |
| `1`–`9` | assign that position |
| `0` | clear position |
| `↵` / `l` | edit description |
| `⌘↵` | commit description, jump to next open block |
| `⇥` / `⇧⇥` | next / previous block |
| `⇥` | inside a Themenarbeit block: Hub → PEG → description |
| `p` | position quick-pick |
| `c` / `v` | copy the position, text and tags — paste them onto the selected block |
| `s` | split block at the marker |
| `m` | merge with neighbour |
| `del` | delete block |
| `↑` / `↓` | move the marker |
| `←` / `→` | previous / next day (week, month) |
| `t` | jump to today |
| `w` | cycle day / week / month view |
| `?` | all shortcuts |

## Export

Transfer to Projectile and the calendar export both act on **one day** — the one on screen — so
they are offered in day view only. The transfer merges consecutive blocks into rows (optional,
with a configurable separator between the merged descriptions), then walks you through cell by
cell. Also: `.ics` calendar export of that day, and a JSON backup of everything.

## Try it

Append `?demo` to the URL — [live](https://moltaire.github.io/projector/?demo), or
`index.html?demo` locally. It runs against a **separate storage key** with three weeks of sample
data covering the awkward cases: a Themenarbeit code with a full Hub/PEG tree, one without
sub-positions, two factura projects, a per-block PEG override, a missing Hub, an unlabeled block,
a late break and the three kinds of day off. The last seven days are always filled, so the demo
opens onto a day with something on it. Your real days live under a different key and are never
touched. The **Demo** badge resets the sample data.

## Notes

Data lives in `localStorage` per browser, so it does not sync between devices — move it with the
JSON backup in the toolbar. Clearing site data erases it. Installing to an iOS home screen gives
the app its **own storage bucket**: it starts empty even though the same URL in Safari is full of
days — the empty state offers the JSON import to seed it.

The timeline runs on pointer events, so blocks, seams and the marker drag under a finger as well
as a mouse. Dragging a block's *body* on a touch screen scrolls instead — move its boundaries on
the seams. Where there is no hover, the merge button shows on the seams around the selected
block, and the block's card carries its own *Verbinden / Merge*.

Open [tests.html](tests.html) from a served copy (not `file://`) to run the test suite — it
drives `index.html` in a hidden frame, so there is still no build step.

[SPEC.md](SPEC.md) has the reasoning behind the design decisions, [BACKLOG.md](BACKLOG.md) the
things not built yet.
