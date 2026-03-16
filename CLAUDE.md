# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file browser game: `monster_shooter_v6.html`. Open it directly in any browser — no build step, no dependencies, no server required.

## Architecture

Everything lives inside one `<script>` tag. The file is ~1965 lines. Sections are delimited by `// ── SECTION NAME ──` comments.

### State machine
`state` (string) drives all rendering and input. Valid values are in the `S` constant (`menu`, `load`, `playing`, `paused`, `upgrade`, `saveslot`, `gameover`, `shop`, `options`, `updates`).

### Game loop
`loop(ts)` → `update(dt)` + `render()` via `requestAnimationFrame`. `dt` is capped at 50ms. All simulation uses `dt`-scaled movement.

### Key data structures
- **`player`** — created by `makePlayer()`, extended by `applyShop(p)` at game start. All player stats live here.
- **`EDEFS`** — mob definitions (`hp`, `spd`, `dmg`, `xp`, `sz`, `hitR`, `coins`, `unlock`, `isBoss`).
- **`SHOP` / `SPECIAL_SHOP`** — permanent between-run upgrades. Levels stored in `shopLvl` (localStorage). Applied via `applyShop()`.
- **`UPGRADES`** (implicit in `pickUpgradeCard()`) — per-run level-up cards with rarity weights.
- **World arrays**: `mobs[]`, `bullets[]`, `parts[]`, `coins[]`, `pickups[]`, `boostPickups[]`, `dmgNums[]`, `rockets[]`.

### Persistence (localStorage keys, all prefixed `harlow_`)
| Key | Content |
|---|---|
| `harlow_s0/1/2` | Save slots — `{player, waveN, waveTimer, ts}` |
| `harlow_shop` | `shopLvl` object |
| `harlow_gc` | Global coin balance |
| `harlow_hi` | High score |
| `harlow_cfg` | Settings (`screenShake`, `volume`, `difficulty`, `showDmgNums`) |

### Wave system
`waveN` and `waveTimer` (counts down from 45s). `updateWaveParams()` recomputes spawn rate/interval from wave number. Called on wave advance and on save load.

### Draw functions
Each screen state has a corresponding `draw*()` function called from `render()`. The shared helper `txt(str, x, y, col, size, align)` wraps canvas text with the "Press Start 2P" font. `glassPanel()` draws the standard dark glass UI card.

### Audio
Procedural — `beep(freq, dur, type, vol, freqEnd)` via Web Audio API. Named wrappers: `sndHit`, `sndHurt`, `sndDie`, `sndLevel`, etc.

## Git workflow

After every meaningful change, commit and push to GitHub so work is never lost:

```bash
git add monster_shooter_v6.html   # or CLAUDE.md if updated
git commit -m "short description of what changed"
git push
```

Commit message conventions:
- `feat: add <thing>` — new feature or mechanic
- `fix: <bug>` — bug fix
- `balance: <change>` — tuning numbers, wave params, etc.
- `ui: <change>` — visual/layout changes
- `docs: <change>` — CLAUDE.md or changelog only

Push after every patch, not just at the end of a session.

## Editing conventions

When writing patch plans, always include the **exact** `old_string` and `new_string` for every change (copy-pasted verbatim from the file). This lets edits be applied with zero file reads during implementation.

## Versioning

Add a new entry at the **top** of the `CHANGELOG` array (line ~49) for every patch. Format:
```js
{ ver:'vX.Y', col:'#HEX', date:'Mon DD, YYYY', added:[], fixed:[] }
```
