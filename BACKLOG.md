# Backlog

Things to build some day. Not commitments, not ordered. [SPEC.md](SPEC.md) holds the
reasoning behind what already exists; this file holds what does not exist yet.

Keep each item to the idea plus whatever open question would otherwise have to be
re-derived from scratch.

---

## Copy / paste blocks with `c` and `v`

Copy the selected block with `c`, paste at the marker with `v`. Carries position,
description and — for a Themenarbeit block — both tags.

**Open:** what happens when the paste would land on occupied time. The day is one gapless
track, so a paste is never a free insertion. Candidates:

- **Insert and push** — everything after the marker shifts later by the pasted duration.
  Keeps every existing block intact, but moves times that were tracked, not chosen.
- **Overwrite** — the pasted block wins, whatever it lands on is trimmed or removed.
  Predictable, destructive.
- **Fill the gap only** — paste is refused unless there is room, clipped to the space
  available. Safest, most often refuses.
- **Paste the label, not the block** — `v` stamps the copied position/description/tags onto
  the block under the marker without touching any boundary. Sidesteps the question, and may
  be what is actually wanted most of the time.

The last one is worth prototyping first: the repetitive part of a day is usually re-labeling
similar work, not re-creating identical durations.

## Half days as Urlaub / Krank

Day types are currently whole-day (`holiday` / `vacation` / `sick`), and a marked day drops
out of the target entirely. Half a day off needs the target to drop by half instead.

**Open:** whether the fraction is fixed at ½ or free (a stored hour count), and how the
month view's leave tally counts it — `0.5` days, or a separate half-day column. The
completeness check and the week/month targets both read `dayTargetMs()`, so the change
lands in one place; the display decisions are the actual work.

## Find and fix the things a warning is warning about

Warnings currently tell you *how many* but rarely *where*. Only the break violation gets
per-day attribution — a `Pause!` flag in the month grid and in the week's per-day rows.
Everything else (blocks without a position, Themenarbeit blocks without a Hub) is an
aggregate count in the rail: you learn two blocks are wrong somewhere this month and then go
looking day by day.

Within a single day it is already solved — unlabeled blocks are hatched on the timeline and
`n` jumps to the next one. The gap is everything above the day.

Worth combining rather than choosing between:

- **Per-day flags in the month grid and week view for every warning**, not just the break —
  the `mo-flags` slot and the week's per-day row already exist and render one flag each.
- **Shading on a flagged day cell**, the way an unlabeled block is hatched on the timeline,
  so a bad day is visible without reading the flags.
- **Make the counts clickable** — clicking "2 Blöcke ohne Position" in the month rail jumps
  to the first offending day with that block selected.
- **Extend `n` across days** when the current day is clean, so the labeling flow keeps
  working past midnight instead of stopping at the day boundary.

**Open:** whether "quickly fix" means *navigate to it* (cheap, composes with the existing
keyboard flow) or *fix it in place* from the month view (a picker in a popover — powerful,
but a second editing surface to keep honest). Navigation first is the safer bet; the fix-in-
place version is only worth it if navigating still feels slow afterwards.

---

# Housekeeping

Code health rather than features. Neither is urgent; both were found in the review after the
Themenarbeit work landed.

## One stats function instead of three

`renderTotals`, `weekStats` and `monthStats` each contain the same loop: walk blocks,
accumulate `per[positionId]`, split work vs pause vs unlabeled. Three copies, so every
change of shape has to be made three times — which is exactly what the Themenarbeit warning
counts needed. A `statsFor(blocks)` returning `{per, work, pause, unlabeledCount, noWgCount}`
would collapse them: the day calls it once, the week seven times, the month once per day.
Perhaps 40 lines net removed.

**Open:** nothing conceptual — but all three views depend on it, so it wants care and a
careful check of each rail afterwards.

## Tests for the export path

`mergedRows → XCOLS → joinDescs / tagParts` is pure, DOM-free, and the one place a bug
produces a *wrong timesheet* rather than a visible glitch. Every check made while building
Themenarbeit was throwaway console code; keeping a dozen of them would have caught the
empty-string tag bug at the moment it was written.

**Open:** where they live, given the project deliberately has no build step. A second HTML
file that loads `index.html`'s functions and prints pass/fail keeps that property; a real
test runner does not.
