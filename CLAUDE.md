# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Vanilla JavaScript Tetris implementation using HTML5 Canvas. No build system, no package manager, no dependencies, no tests — just `index.html`, `style.css`, and `game.js`.

## Running the game

Open `index.html` directly in a browser, or serve it statically:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no build, lint, or test step for this project.

## Architecture

All game logic lives in `game.js` (~300 lines), structured around a single `requestAnimationFrame` loop with module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc. — declared once at the top, reset in `init()`).

Key pieces:

- **Board model**: `board` is a `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: `PIECES` defines the 7 tetrominoes as square matrices. `rotateCW` rotates via transpose + row reversal (no matrix library).
- **Collision** (`collide`): checks board bounds and overlap against locked cells; used for movement, rotation, and spawn checks.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until one doesn't collide, else the rotation is discarded.
- **Locking and lines** (`lockPiece` → `merge` + `clearLines` + `spawn`): `clearLines` scans bottom-to-top, splicing out full rows and unshifting empty ones at the top; re-checks the same row index after a splice since rows shift down.
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row, soft drop adds 1 pt/row. Level increments every 10 cleared lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece** (`ghostY`): projects the current piece straight down until collision, drawn at `globalAlpha = 0.2`.
- **Game over**: detected in `spawn()` when a freshly spawned piece immediately collides.

Rendering (`draw`, `drawNext`, `drawBlock`, `drawGrid`) is plain Canvas 2D — no sprites/images, blocks are filled rects with a lighter highlight strip.

Input is a single `keydown` listener dispatching on `e.code` (arrows, `KeyX` for rotate, `Space` for hard drop, `KeyP` for pause); pause/game-over short-circuit all input except unpause/restart.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell size px), `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).

## Notes

- README.md is written in Spanish and is the authoritative player-facing doc (controls, mechanics, customization table) — keep it in sync with any gameplay-affecting change.
- `index.html` and `game.js` are tightly coupled via DOM element IDs (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`); renaming one requires updating the other.
