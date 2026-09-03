# AGENTS.md

Vanilla HTML/CSS/JS Pac-Man clone. No bundler, no npm, no tests, no linter. Open `src/index.html` in a browser (`file://` works).

## Spec-driven workflow

This repo exists to practice spec-driven development. For new features:

1. `/spec` → write `specs/NN-slug.md` as Draft. Do not write application code in that session.
2. A human sets status to Approved / Aprobado.
3. `/spec-impl NN-slug` → implement only that spec, one plan step at a time.

Do not add a toolchain, ES modules, or tests unless a spec says so.

Specs live in `specs/` (folder may not exist yet; first file is `01-…`). Once specs exist, match their language and status words. README, UI copy, and existing comments are Spanish.

## Architecture

Scripts load as classic globals. Order in `src/index.html` is required:

`maze.js` → `game.js` → `render.js` → `main.js`

Each file exports via `window`. Do not switch to `type="module"` without a spec.

- `src/js/maze.js` — immutable `MAZE` (copy per game into `game.grid`). Tiles: `0` empty, `1` wall, `2` dot, `3` pen door. Also `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`.
- `src/js/game.js` — `createGame` / `update`. Mutate `game.grid`, never `MAZE`.
- `src/js/render.js` — draw `game.grid` (not `MAZE`). `TILE = 20`.
- `src/js/main.js` — rAF loop, keyboard, overlay.

Canvas is 560×620 = 28×31 cells × 20px. Changing maze size or `TILE` requires updating the `<canvas>` attributes and `#game-wrap` in `src/css/style.css` together.

## Easy to break

- Pac-Man cannot pass tile `3` (pen door); ghosts can.
- Tunnel wrap only on `TUNNEL_ROW` (14).
- Speeds (`PACMAN_SPEED = 0.125`, `GHOST_SPEED = 0.1`) must land on integer cells (`aligned()`). Arbitrary speeds desync movement and turns.
- Ghost `kind`: `hunter` (greedy Manhattan) vs `random`. No 180° reverse except dead ends.
- `showOverlay` replaces overlay `innerHTML`; the Start/Restart click handler must be re-bound after that.

## Style

Spaces inside parentheses, matching existing files: `if ( !dir )`, `getElementById( 'game' )`.
