# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris using the HTML5 Canvas 2D API. No dependencies, no
build step, no bundler, no `package.json`. UI text and comments are in Spanish.

## Running

There is nothing to build or install. Open `index.html` directly, or serve the
folder statically (needed only if you later add module scripts or fetch calls):

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

There are no tests, linter, or CI configured.

## Architecture

Three cooperating files: `index.html` (DOM + two `<canvas>` elements),
`style.css` (dark arcade theme), and `game.js` (all game logic, ~300 lines).
Everything runs in the module-less global scope of `game.js`; game state lives
in top-level `let` variables (`board`, `current`, `next`, `score`, etc.),
reset by `init()`.

Key model conventions in `game.js`:

- **Board** is a `ROWS × COLS` array of ints. `0` = empty; `1–7` = a color
  index that also identifies the piece type. `COLORS` and `PIECES` are indexed
  the same way (index `0` is `null`), so a cell value maps directly to both.
- **Pieces** are square matrices. Rotation is transpose + row-reverse
  (`rotateCW`); `tryRotate` applies basic wall kicks by testing horizontal
  offsets `[0, -1, 1, -2, 2]`.
- **`collide(shape, x, y)`** is the single source of truth for legality —
  bounds and overlap. Movement, rotation, dropping, and game-over all gate on
  it. It reads the module-global `board`.
- **Game loop** (`loop`) is `requestAnimationFrame`-driven, accumulating `dt`
  into `dropAccum` and stepping the piece down once it exceeds `dropInterval`.
  Pause/resume cancels and restarts the rAF via `animId`.
- **Lifecycle**: `spawn()` promotes `next` → `current` and rolls a new `next`;
  a collision on spawn triggers `endGame()`. `lockPiece()` = `merge()` +
  `clearLines()` + `spawn()`.
- **Scoring/level**: `LINE_SCORES` (× level) for clears, +2/cell hard drop,
  +1/row soft drop. Level = `floor(lines/10)+1`; speed =
  `max(100, 1000 − (level−1)×90)` ms.

## Gotchas

- Canvas pixel dimensions are hard-coded in `index.html`
  (`board` = 300×600, `next-canvas` = 120×120). If you change `COLS`, `ROWS`,
  or `BLOCK` in `game.js`, update the `<canvas width/height>` to match
  (`COLS×BLOCK` by `ROWS×BLOCK`), or rendering will be clipped or scaled.
- `game.js` references DOM elements by id at load time, so it must run after the
  markup — it's included as a plain (non-deferred) `<script>` at the end of
  `<body>`. Keep element ids in `index.html` and `game.js` in sync.
