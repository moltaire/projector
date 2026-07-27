# Projector — Build Plan

Phased plan. See [SPEC.md](SPEC.md) for the spec, [DESIGN.md](DESIGN.md) for look/feel.

**Status 2026-07-26:** Phase 1 is built and working. Phase 0 came back with a hard answer:
**Projectile does not support pasting a whole table.** That kills the original one-paste
export, but not the project — real-world use has shown the value is in *reconstructing the
day*, with Projectile entry as the last mile. Phase 2 is therefore redesigned around
**assisted cell-by-cell entry** (see below).

---

## Phase 0 — Verify Projectile ingestion — ✅ DONE (partially)
- [x] Whole-table paste: **not supported.** Ruled out.
- [x] Break rules confirmed: 30 min **by** 6h of work, 45 min **by** 9h, ≥15-min chunks.
- [ ] Still unknown: exact per-cell entry form (columns, tab order, which fields
      auto-fill), and whether bookmarklets / keyboard macros are permitted on the work
      machine. **These now gate Phase 2.**

---

## Phase 1 — v1 app — ✅ DONE
Single self-contained `index.html`, vanilla JS, localStorage, DESIGN.md tokens.

- [x] Data model + storage layer (Block, Position, settings) — SPEC §3.
- [x] Live loop: Start day / Switch / Break / Stop; running block ticks; multi-session.
- [x] Day track render: blocks, now-line, day navigation.
- [x] Editing grammar: split-at-marker, merge-on-seam (longer-wins), seam drag (ripple),
      block drag, edge resize, draw-on-empty, snapping, precise start/end fields.
- [x] Labeling: sorted position picker + description autocomplete; unlabeled-block flagging.
- [x] Totals + target + break-compliance engine (chronological ArbZG check).
- [x] Positions manager (CRUD; "Pause" reserved/locked) + positions JSON import/export.
- [x] Backup: full-state JSON export/import.
- [x] **Calendar (.ics) export.**
- [x] Overnight-running safety prompt.
- [x] Keyboard shortcuts: `s` split · `m` merge · `p` position · `l` label · `Del` · `↑/↓`.
- [x] German localisation (default) + English option; 24h time throughout.
- [x] Projectile export: visible **placeholder** only.

**Exit:** ✅ can track/rebuild a real day, see compliant totals, and export an .ics.

---

## Phase 2 — Projectile last mile (REDESIGNED — needs a decision)
Goal is no longer "one paste." It is **minimum cells, fastest handoff per cell.**

**2a. Reduce the work (app-side) — ✅ DONE**
- [x] Merge consecutive same-position blocks; de-duplicate joined descriptions.
- [x] Configurable columns (date/start/end/duration/position/description), date format and
      duration format (HH:MM or decimal) — the field set is still being learned.
- [x] Export preview table showing exactly the rows/cells to be entered.
- [x] Gate: refuses while a block is unlabeled; warns on break-rule violation.

**2b. Speed up the handoff — guided cell queue ✅ DONE; automation still open**
- [x] **Guided cell queue** — click a cell (or Enter) to copy and advance; arrows navigate;
      `Zeile kopieren` / `Alles kopieren` as TSV fallbacks. Copy failure never advances.
- [ ] **Bookmarklet** — a plain bookmark that fills Projectile's fields via the DOM.
      No install. Fastest option *if* allowed and the form is scriptable. **Needs recon.**
- [ ] **Keyboard macro payload** — emit a value-Tab-value-Tab script for a macro runner
      (macOS Shortcuts/AppleScript or Windows PowerShell SendKeys — both built in, no install).

**Exit:** a day gets into Projectile with materially less effort than typing it from memory.

---

## Phase 2.5 — Week view & stats — ✅ DONE
- [x] Day/Week toggle (`w`); ISO weeks (Mon–Sun, KW numbers); DST-safe date arithmetic.
- [x] Read-only 7-day strip; click a day to open it in the day view.
- [x] Stats rail: week total vs configurable weekly target, by-position shares, per-day
      work/Pause bars, per-day break-compliance flags, unlabeled warning.
- [x] Marker discoverability: gutter is a visible scrub rail; marker grabbable along its line.
- [x] Responsive fixes (no page-level horizontal scroll).

---

## Phase 2.6 — Naming, labels & help — ✅ DONE
- [x] Renamed **Projectile Buddy → Projector** (same project+suffix joke, no ordnance);
      storage migrated from `projectile-buddy-v1` with the legacy copy kept.
- [x] Header relabelled by action: Übertragen · Kalender-Export │ Positionen · Einstellungen · ?
- [x] `?` Help overlay: "How tracking works" (Start/Switch/Pause/Stop explained) + shortcuts.
- [x] Start/Pause/Stop explained in-context via tooltips and a Now-card hint.
- [x] User-adjustable theme (bg/text/accent + 5 presets), all derived tokens WCAG AA.

---

## Phase 2.7 — Keyboard-first labeling — ✅ DONE
- [x] Stable per-Position `hotkey` (1–9), unique, auto-assigned (projects before Pause),
      editable in the Positions manager.
- [x] Visible numbered chips + "n left without a position" counter in the editor card.
- [x] Loop: `n` next unlabeled · `1`–`9` assign · `0` clear · `↵` description ·
      `⌘↵` commit+next · `⇥`/`⇧⇥` next/previous block.
- [x] Quick-pick overlay (`p`) replacing the keystroke-trapping native select.
- [x] Help panel gains a "Fast labeling" section.

---

## Phase 2.8 — Position hierarchy & billability — ✅ DONE
- [x] Two-level tree on Position (`parentId`): projects contain work packages; bookable =
      leaf. Third level refused; containers hold no hotkey and never appear in pickers.
- [x] Orthogonal `billable` flag, inherited from the project, overridable per package;
      Pause counts as neither. Unset resolves to internal (safe direction).
- [x] Factura / Intern split bar in day totals and the week rail.
- [x] Week "by position" rolls up by project with packages listed beneath.
- [x] Positions manager: tree rendering, parent + billing selectors, colour inheritance.
- [x] Optional `Projekt` export column (off by default — position code is enough).
- [x] Today highlight in week view; Now-card caption reduced to a Help link; editor hint trimmed.
- [x] Dropped the `favorite` flag (hotkeys supersede it); picker order now follows hotkey slots.
- [x] Positions manager rebuilt as a labelled grid in a wider modal — usable name field.

---

## Phase 3 — Durability & polish
- [ ] IndexedDB (behind the storage interface) + File System Access autosave where available.
- [ ] Optional: recurring day templates, optional periodic ping.

---

## Deferred (v2+, SPEC §6)
Label inference · ambient capture · weekly summaries · multi-user.
