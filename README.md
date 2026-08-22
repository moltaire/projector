# 🟦🟨🟩 projector

[![GitHub Pages](https://img.shields.io/badge/GitHub-Pages-blue?logo=github)](https://moltaire.github.io/projector/)

Browser-based time tracker that prepares Projectile timesheets, for my personal use. Single file, no build step, data stays in the browser.
Vibe coded with [Claude](https://claude.ai).

The name is the same joke as *Projectile* (project + suffix), minus the ordnance.

## what it does

- The day is one gapless track of blocks, edited like a video timeline: split at a marker, merge at a seam, drag blocks and boundaries
- Live tracking (start / switch / break / stop, several sessions per day) or rebuild a forgotten day from scratch
- Positions nest two levels (project → work package) with an orthogonal Factura / Intern flag and a colour, both inherited and overridable
- A booking code can be marked **Themenarbeit**: its description exports as `Hub | PEG | Text`, and its sub-positions become Hub and PEG rows — pick one by hotkey and both tags come with it, suggested from the tree and from what you have typed before
- Enforces the German break rule properly: 30 min *by* 6 h of work, 45 min by 9 h, in chunks of ≥15 min, checked chronologically
- Week view with a 7-day strip, hours per project, per-day totals and the Factura / Intern split
- Month view: calendar grid with per-day hours and flags, plus month totals, per-week bars, leave tally and a completeness check
- Days can be marked *Feiertag / Urlaub / Krank*; they drop out of the day, week and month targets
- Transfer to Projectile merges consecutive blocks into rows — optional, with a configurable separator between the merged descriptions — then walks you through cell by cell (paste → Tab → ↵)
- Calendar export (`.ics`), JSON backup, German / English, themeable, installable to the home screen

## how to use it

| key | action |
|-----|--------|
| `n` | next block without a position |
| `1`–`9` | assign that position |
| `0` | clear position |
| `↵` / `l` | edit description |
| `⌘↵` | commit description, jump to next open block |
| `⇥` / `⇧⇥` | next / previous block |
| `⇥` | inside a Themenarbeit block: Hub → PEG → description |
| `p` | position quick-pick |
| `s` | split block at the marker |
| `m` | merge with neighbour |
| `del` | delete block |
| `↑` / `↓` | move the marker |
| `←` / `→` | previous / next day (week, month) |
| `t` | jump to today |
| `w` | cycle day / week / month view |
| `?` | all shortcuts |

## try it

Append `?demo` to the URL — [live](https://moltaire.github.io/projector/?demo), or `index.html?demo` locally. That runs the app against a **separate storage key** with three weeks of sample data: a Themenarbeit code with a full Hub/PEG tree, a Themenarbeit code without sub-positions, two factura projects, a per-block PEG override, a missing Hub, an unlabeled block, a late break and the three kinds of day off. Your real days live under a different key and are never read or written in demo mode. The **Demo** badge in the header resets the sample data.

## notes

Data lives in `localStorage` per browser, so it does not sync between devices — move it with the JSON backup in *Einstellungen*. Clearing site data erases it.

Installing it to an iOS home screen gives the app its **own storage bucket**: it starts empty even though the same URL in Safari is full of days. The empty state offers the JSON import to seed it.

[SPEC.md](SPEC.md) has the reasoning behind the design decisions.
