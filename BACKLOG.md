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
