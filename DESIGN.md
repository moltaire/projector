# Projector — Design Language

Salvaged from the [pomodoro app](https://github.com/moltaire/pomodoro) (tokens extracted
from its source, kept verbatim where possible), adapted for a feature-richer tool.
See [SPEC.md](SPEC.md) for product scope.

**Thesis:** keep the pomodoro app's *visual* language (calm dark palette, olive accent,
Inter + tabular numerals, soft radii, restrained motion). Drop its *behavioral* minimalism
(invisible affordances, hover-only controls, single-screen everything) — those don't scale
to positions + timeline + export.

---

## 1. Design tokens (drop-in)

```css
:root {
  /* Surfaces (dark, muted) — verbatim from pomodoro */
  --bg:      #1c1b22;   /* app ground */
  --card:    #2b2a32;   /* raised panels, modals */
  --card2:   #27262e;   /* inset fields, nested surfaces */
  --line:    #3a3942;   /* hairline borders / timeline rules (new) */

  /* Text */
  --text:    #e5e5e5;   /* primary */
  --dim:     #7c7884;   /* secondary / labels / placeholders */

  /* Accent (olive/sage) */
  --accent:        #757c58;  /* borders, focus, checkboxes */
  --accent-bright: #a8b86a;  /* the ONE primary action (filled) */
  --accent-hover:  #bccf7a;  /* primary hover */
  --on-accent:     #1c1b22;  /* text on filled accent */

  /* Status (from the tomato glyphs) */
  --focus:   #CE3F32;  /* on-task / running */
  --break:   #F4BA40;  /* short break / warning */
  --rest:    #2A62B6;  /* long break / info */

  /* Shape & depth */
  --radius:    12px;   /* cards, modals */
  --radius-sm: 8px;    /* fields, buttons */
  --radius-pill: 999px;
  --shadow-modal: 0 8px 32px rgba(0,0,0,0.5);
  --scrim: rgba(0,0,0,0.55);

  /* Motion — restrained on purpose */
  --t: 0.15s;
}
```

### Position colors (new — pomodoro didn't need these)
The timesheet needs per-position block colors on the timeline. Muted set tuned for the dark
ground, harmonizing with the olive accent. Assign round-robin or let the user pick.
```css
/* --pos-1 … --pos-8 */
#5e7ce0  #4fa3a1  #8a9a5b  #d49a4a  #9b6a9e  #c2674f  #6b7a8f  #c07088
/* blue    teal     olive    amber    mauve    rust     slate    rose   */
```
Use at ~85% opacity for block fills so text stays legible; full strength for the left edge/dot.

---

## 2. Typography

- **Family:** `Inter, system-ui, sans-serif`. Pomodoro loads Inter from Google Fonts — the
  new app must not call out to a CDN (privacy/offline). **Decision needed:** self-host Inter
  (inline base64 woff2 subset) or fall back to the system UI stack. Default fine either way
  because `system-ui` is the fallback.
- **Tabular numerals everywhere time appears:** `font-variant-numeric: tabular-nums;`
  (running timer, totals, timeline labels). This is core to the look — don't skip it.

| Role | Size | Weight | Notes |
|---|---|---|---|
| Hero timer / countdown | 4.5rem | 600 | `letter-spacing: -3px`, tabular |
| Section value (totals) | 1.4–2rem | 600 | tabular |
| Body / inputs | 0.9rem | 400–500 | |
| Micro-label (UPPERCASE) | 0.65–0.72rem | 600 | `letter-spacing: 0.07em–0.1em`, `--dim` |
| Meta / hint | 0.65rem | 400 | `--dim` |

Tight negative tracking on big numerals, generous positive tracking on tiny uppercase labels —
that contrast *is* the pomodoro signature. Keep it.

---

## 3. Components

- **Cards:** `--card`, `--radius`, no border. Modals add `--shadow-modal` over a `--scrim`.
- **Fields:** `--card2` fill, `--radius-sm`, 1px transparent/`--line` border; focus →
  `border-color: var(--accent)`. (Pomodoro also used a bottom-border-only variant for inline
  edits — fine for inline rename, not for forms.)
- **Buttons:**
  - *Ghost (default):* transparent, `--dim` text, hover → `--text`. Use for nearly everything.
  - *Primary (exactly one per view):* filled `--accent-bright`, `--on-accent` text, hover
    `--accent-hover`. Reserve for the decisive action (Stop, Export, Save).
  - *Outline:* `--card2` fill, hover `border-color: var(--accent)`. Secondary modal actions.
- **Pills/badges:** `--radius-pill`, `--card`, `--dim` text. For pomodoro count, filters, tags.
- **Checkboxes:** `accent-color: var(--accent)`.
- **Icons:** thin inline SVG (`stroke-width ~1.3`, `currentColor`), like the trash glyph.

---

## 4. Motion

Only `transition: opacity var(--t), transform var(--t);`. No spring, no bounce, no long eases.
Timeline drag should feel direct (track the cursor 1:1), not animated. Calm > playful.

---

## 5. What to KEEP vs. RELAX vs. ADD

### Keep (the soul)
- The full color palette + olive accent, one-primary-action discipline.
- Inter + tabular numerals; the big-tight / small-tracked type contrast.
- 12px cards, soft modal shadow, 0.15s-only motion.
- Ghost-first buttons; dim secondary text; uppercase micro-labels.
- Status red/amber/blue for running / break / rest.

### Relax (won't scale to this app)
- **Invisible affordances** — pomodoro's near-invisible placeholder
  (`rgba(255,255,255,0.01)`) and hover-only controls. We have positions, export, settings,
  timeline editing: controls must be *discoverable*. Visible labels and placeholders.
- **Single-screen everything** — adopt the SPEC §5 two-pane layout (timeline | now+totals).
  Keep each pane calm, but persistent structure is allowed now.
- **Keyboard-only discovery** — keep the shortcuts (they're great), but every shortcut action
  also has a visible control. Don't hide function behind memorization.
- **No navigation** — add light day/week nav, a settings entry, an export button. Chrome is OK
  in small, quiet doses.

### Add (new needs)
- Position color spectrum (§1) for timeline blocks + legend dots.
- Timeline surface: hour rules in `--line`, "now" indicator in `--focus`, gaps = bare `--bg`
  (off-task reads as absence, matching SPEC principle 3).
- A real settings surface (rounding, positions, backup) — pomodoro hid config; we present it.
- (Later) light mode: all colors are already CSS vars, so a `:root` swap is enough. Defer.

---

## 6. Layout north star

Two calm panes on `--bg`, generous breathing room, content-forward. The timeline reads like a
quiet day planner; the right rail is the live timer + today's tally. One olive primary action
visible at a time. Nothing shouts. (Wireframe: SPEC §5.)
