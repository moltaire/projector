# 🟦🟨🟩 projector

[![GitHub Pages](https://img.shields.io/badge/GitHub-Pages-blue?logo=github)](https://moltaire.github.io/projector/)

Browser-based time tracker that prepares Projectile timesheets, for my personal use. Single file, no build step, data stays in the browser.
Vibe coded with [Claude](https://claude.ai).

The name is the same joke as *Projectile* (project + suffix), minus the ordnance.

## what it does

- The day is one gapless track of blocks, edited like a video timeline: split at a marker, merge at a seam, drag blocks and boundaries
- Live tracking (start / switch / break / stop, several sessions per day) or rebuild a forgotten day from scratch
- Positions nest two levels (project → work package) with an orthogonal Factura / Intern flag and a colour, both inherited and overridable
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
| `p` | position quick-pick |
| `s` | split block at the marker |
| `m` | merge with neighbour |
| `del` | delete block |
| `↑` / `↓` | move the marker |
| `←` / `→` | previous / next day (week, month) |
| `t` | jump to today |
| `w` | cycle day / week / month view |
| `?` | all shortcuts |

## notes

Data lives in `localStorage` per browser, so it does not sync between devices — move it with the JSON backup in *Einstellungen*. Clearing site data erases it.

Installing it to an iOS home screen gives the app its **own storage bucket**: it starts empty even though the same URL in Safari is full of days. The empty state offers the JSON import to seed it.

[SPEC.md](SPEC.md) has the reasoning behind the design decisions.
