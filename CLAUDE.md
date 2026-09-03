# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Vanilla-JS Tetris rendered on HTML5 Canvas. No build step, no dependencies, no
`package.json`, no tests. Three source files: `index.html`, `style.css`, `game.js`.

## Running

```bash
open index.html            # macOS — works directly, no server needed
python3 -m http.server 8000 # or serve statically, then open http://localhost:8000
```

There is nothing to build, lint, or test. Verify changes by opening the game in a
browser and playing.

## Architecture (`game.js`)

Single-file, module-scoped script (`'use strict'`, no exports). All state lives in
the top-level `let board, current, next, score, ...` declarations, reset by `init()`.

- **Board model**: `board` is a `ROWS × COLS` array of ints. `0` = empty; `1–7` index
  into `COLORS` and identify the piece that filled the cell.
- **Pieces**: `PIECES[1..7]` are square matrices. `current`/`next` are
  `{ type, shape, x, y }`. Rotation (`rotateCW`) transposes + reverses rows into a
  fresh matrix; `tryRotate` applies basic wall kicks (`[0,-1,1,-2,2]` column offsets).
- **Collision**: `collide(shape, x, y)` is the single gatekeeper — every move,
  rotation, drop, and spawn checks it. Out-of-bounds and overlap with a non-zero
  board cell both count as collision; cells above the board (`ny < 0`) are allowed.
- **Game loop**: `loop(ts)` via `requestAnimationFrame`. Accumulates `dt` into
  `dropAccum`; when it exceeds `dropInterval` the piece drops one row or locks.
  `animId` holds the frame handle; pause/game-over cancel it.
- **Piece lifecycle**: `lockPiece` → `merge` (stamp shape into `board`) →
  `clearLines` → `spawn` (promote `next` to `current`, roll a new `next`, and call
  `endGame()` if the fresh piece already collides).
- **Scoring / level**: `LINE_SCORES` table × `level`; soft/hard drop add per-cell
  points. `level = floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Rendering**: `draw()` clears and repaints grid → locked board → ghost piece
  (`ghostY()` projected, alpha 0.2) → current piece, every frame. `drawNext()` runs
  only on spawn. `drawBlock` is shared by both canvases (board and next-preview).
- **Input**: one `keydown` listener. `P` toggles pause always; other keys are
  ignored while paused or game-over. Movement mutates `current.x/y` directly after a
  `collide` check.

## Gotchas

- Canvas pixel dimensions are hard-coded in `index.html` (`board` 300×600,
  `next-canvas` 120×120). Changing `COLS`, `ROWS`, or `BLOCK` in `game.js` requires
  matching edits there.
- UI strings mix English (HUD labels) and Spanish (overlay text, control captions).
  Match the surrounding language when editing.
- No hold piece, no 7-bag randomizer (`randomPiece` is uniform random), no SRS —
  keep additions consistent with this deliberately minimal scope unless asked.
