# Backlog

Things to build some day. Not commitments, not ordered. [SPEC.md](SPEC.md) holds the
reasoning behind what already exists; this file holds what does not exist yet.

Keep each item to the idea plus whatever open question would otherwise have to be
re-derived from scratch.

---

## Pasting a block, not just its label

`c` / `v` carry a block's position, text and tags onto the block under the marker, which
covers the repetitive case without touching a single boundary. What is still missing is
pasting the block *itself* — its duration — and that is the part with the open question:
the day is gapless, so a paste always displaces recorded time.

**Open:** unchanged from before — insert-and-push (honest about the blocks, dishonest about
their times), overwrite (predictable, destructive), or fill-the-gap-only (safe, almost
always refuses). Worth revisiting only once the label paste has been lived with for a while;
it may turn out to cover everything the geometric version was wanted for.
