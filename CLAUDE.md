# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris in vanilla JavaScript + HTML5 Canvas + CSS. No dependencies, no package.json, no build step, no linter, no test suite. UI strings and the README are in Spanish; keep user-facing text in Spanish.

## Running

Open `index.html` directly, or serve the folder statically (preferred):

    python -m http.server 8000   # then http://localhost:8000

There are no tests; verify changes by playing in the browser and checking the devtools console.

## Architecture

All logic is in `game.js`: one classic script (no modules) with `'use strict'` and module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, …). `init()` resets all of it and is also the restart button handler.

- **Board model**: a `ROWS × COLS` matrix. `0` means empty; `1–7` is the piece type, which indexes both `COLORS` and `PIECES`. Each piece's shape matrix stores its own type number, so merged cells keep their color.
- **Piece lifecycle**: `spawn()` promotes `next` to `current` and checks for game over by testing for a collision at the spawn point → gravity in `loop()` / `softDrop()` / `hardDrop()` → `lockPiece()` = `merge()` + `clearLines()` + `spawn()`.
- **Collision**: everything goes through `collide(shape, x, y)`. Cells above the board (`y < 0`) are allowed.
- **Rotation**: `rotateCW()` then `tryRotate()` tries horizontal kicks `[0, -1, 1, -2, 2]`. This is not SRS.
- **Loop**: `requestAnimationFrame` adds elapsed time to `dropAccum` and compares it with `dropInterval`. Pause and game over stop the loop with `cancelAnimationFrame(animId)`. Resuming resets `lastTime` and calls `loop` again.
- **Scoring/level**: `LINE_SCORES[cleared] * level`, plus 1 point per soft-drop row and 2 per hard-drop cell. Level is `floor(lines/10)+1`. `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Rendering**: `draw()` redraws the full frame (grid, locked cells, ghost at alpha 0.2, then the current piece). `drawNext()` only redraws when a piece spawns, into a 4×4 grid of 30px cells.
- **HUD**: DOM elements are looked up by id at load time. The `keydown` handler calls `updateHUD()` after every key press.

## Coupled values to keep in sync

- `COLS × BLOCK` and `ROWS × BLOCK` must match the `<canvas id="board">` width and height in `index.html` (300×600).
- `#next-canvas` (120×120) assumes the hard-coded 4×4 grid × 30px in `drawNext()`.
- Element ids in `index.html` (`score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`, `next-canvas`) are referenced directly from `game.js`.
- When you change controls or mechanics, update the controls list in `index.html` and the README tables.
