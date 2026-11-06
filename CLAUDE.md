# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris (HTML5 Canvas + CSS). No dependencies, no build step, no tests, no linter. UI text and README are in Spanish.

Run it by opening `index.html` in a browser, or serve it locally (e.g. `python3 -m http.server`) and visit `http://localhost:8000`.

## Architecture

Three files: `index.html` (markup, canvas elements, HUD, overlay), `style.css`, and `game.js` (all logic, loaded as a plain `<script>`).

`game.js` is a single global-state module (`'use strict'`) with no classes:

- **State** lives in top-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, ...), reset by `init()`. `init()` is also the restart handler.
- **Board** is a `ROWS × COLS` (20×10) matrix of color indices; `0` is empty, `1–7` index into both `COLORS` and `PIECES` (I, O, T, S, Z, J, L). Piece shapes embed their own color index as the cell value, so `merge()` writes shape cells straight into the board.
- **Game loop**: `loop(ts)` runs on `requestAnimationFrame`, accumulates `dropAccum` against `dropInterval` for gravity, then calls `draw()`. Pause/game over cancel the frame via `cancelAnimationFrame(animId)`; unpausing restarts `loop`.
- **Piece lifecycle**: `lockPiece()` → `merge()` → `clearLines()` (updates score/level/`dropInterval`) → `spawn()` (promotes `next`, game over if the new piece collides).
- **Input**: one `keydown` listener (arrows, `X` rotate, `Space` hard drop, `P` pause). Rotation is `rotateCW` plus horizontal wall-kick offsets `[0, -1, 1, -2, 2]` in `tryRotate()`. All movement validity goes through `collide(shape, ox, oy)`.
- **Rendering** is split between the main board canvas (`draw()`, including the ghost piece via `ghostY()`) and `next-canvas` (`drawNext()`); both use `drawBlock`. Canvas pixel size (300×600) is hardcoded in `index.html` and must match `COLS/ROWS × BLOCK` (30).

**Theming**: colors in `style.css` are CSS variables (`:root` = dark, `[data-theme="light"]` overrides). Canvas colors come from the `THEMES` object in `game.js` (piece palette, grid, highlight). `applyTheme()` sets `data-theme`, persists to `localStorage` and redraws manually (the loop may be cancelled). The toggle button calls `blur()` so `Space` doesn't re-trigger it.

Scoring/speed constants: `LINE_SCORES` × `level`; level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`; soft drop +1/cell, hard drop +2/cell.
