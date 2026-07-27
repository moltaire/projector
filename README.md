# 🟦🟨🟩 projector

[![GitHub Pages](https://img.shields.io/badge/GitHub-Pages-blue?logo=github)](https://moltaire.github.io/projector/)

Browser-based time tracker that prepares Projectile timesheets, for my personal use. Single file, no install, data stays in the browser.
Vibe coded with [Claude](https://claude.ai).

The name is the same joke as *Projectile* (project + suffix), minus the ordnance.

## what it does

- The day is one gapless track of blocks, edited like a video timeline: split at a marker, merge at a seam, drag blocks and boundaries
- Live tracking (start / switch / break / stop, several sessions per day) or rebuild a forgotten day from scratch
- Positions nest two levels (project → work package) with an orthogonal Factura / Intern flag, inherited and overridable
- Enforces the German break rule properly: 30 min *by* 6 h of work, 45 min by 9 h, in chunks of ≥15 min, checked chronologically
- Week view with a 7-day strip, hours per project, per-day totals and the Factura / Intern split
- Transfer to Projectile merges consecutive blocks into rows, then walks you through cell by cell (paste → Tab → ↵)
- Calendar export (`.ics`), JSON backup, German / English, themeable

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
| `←` / `→` | previous / next day (week) |
| `t` | jump to today |
| `w` | day / week view |
| `?` | all shortcuts |

## notes

Data lives in `localStorage` per browser, so it does not sync between devices — move it with the JSON backup in *Einstellungen*. Clearing site data erases it.

[SPEC.md](SPEC.md) has the reasoning behind the design decisions.
