# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-player Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS — no build step, no package manager, no external dependencies. The entire game logic lives in `game.js` (~300 lines).

## Running the game

There is no build/install/lint/test tooling (no `package.json`). To run:

```bash
start index.html       # Windows — open directly in the browser
```

Or serve it locally (recommended so canvas/asset behavior matches a real deployment):

```bash
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

Then open `http://localhost:8000`. There is no test suite; verify changes by playing the game in the browser.

## Architecture

Three files, each with one responsibility:

- `index.html` — DOM shell: the main `#board` canvas (300×600), a side panel (`#score`, `#lines`, `#level`, `#next-canvas` preview), and an `#overlay` used for both the pause and game-over states.
- `style.css` — dark/retro arcade visual theme (flexbox layout, monospace HUD, `backdrop-filter` overlay).
- `game.js` — all game logic, structured around a global mutable state block (`board, current, next, score, lines, level, paused, gameOver, ...`) rather than a class/module system.

### Core model

- **Board**: `ROWS × COLS` matrix (`createBoard`); each cell is `0` (empty) or an integer 1–7 identifying which piece color occupies it (`COLORS` / `PIECES` are parallel arrays indexed by piece type).
- **Pieces**: defined as square matrices in `PIECES`. Rotation (`rotateCW`) is a transpose + row-reverse, not a lookup table of rotation states.
- **Collision** (`collide`): checks a shape against board bounds and already-locked cells at a given offset.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` until a non-colliding position is found.
- **Locking/clearing**: `lockPiece` → `merge` (bakes the current piece into `board`) → `clearLines` (bottom-up scan, `splice`/`unshift` to remove full rows) → `spawn` (promotes `next` to `current`, generates a new `next`; if the new piece immediately collides, calls `endGame`).
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop 1 pt/row. Level increments every 10 cleared lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Ghost piece**: `ghostY` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2`.

### Game loop

`init()` seeds state and starts `requestAnimationFrame(loop)`. `loop(ts)` accumulates elapsed time in `dropAccum`; once it exceeds `dropInterval`, the piece drops one row (or locks if it can't). Every frame calls `draw()` (grid → locked board → ghost → current piece). Input is handled by a single `keydown` listener (arrow keys, `X` to rotate, `Space` for hard drop, `P` to pause) that mutates `current`/game state directly — there's no separate input queue.

## Tunable constants (all in `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval` (initial). If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
