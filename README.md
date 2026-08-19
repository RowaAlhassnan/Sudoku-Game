# Sudoku

A colour-coded Sudoku game that lives in a single HTML file. No build step, no
dependencies, no bundler — open `index.html` in a browser and play.

Every digit owns a hue, so the board starts near-monochrome (clues are in ink)
and colours itself in as you solve it. Pencil marks, the keypad and the
highlight system all inherit the digit's colour, which makes the palette
functional rather than decorative.

## Running it

```
open index.html          # macOS
```

Or serve the directory if you prefer a real origin:

```
python3 -m http.server 8000    # then visit http://localhost:8000
```

Both work. On `file://` origins `localStorage` can throw, so the storage layer
falls back to an in-memory store for the session — statistics and saved games
just won't survive a reload.

## Features

**The puzzle**
- Puzzles are generated in the browser: a randomised backtracking solver builds
  a full grid, then clues are dug out one at a time, keeping a clue only when
  its removal would leave more than one solution. Every puzzle has exactly one
  answer.
- Three tiers, by clue count: Easy 45, Medium 34, Hard 27.

**Playing**
- Pencil marks (notes) with a 9-bit mask per cell.
- Undo / redo history.
- Auto-check, which flags entries that contradict the solution.
- Conflict highlighting for digits duplicated in a row, column or box.
- Peer highlighting for the selected cell.
- Keypad badges counting how many of each digit are still missing.
- Timer with pause (the board is hidden while paused).
- Reset board, new puzzle, and a win overlay with confetti.

**Persistence** — all in `localStorage`, all local to the browser:
- `sudoku.hue.game.v1` — the game in progress
- `sudoku.hue.stats.v1` — best time per difficulty, games won, current streak
- `sudoku.hue.key.v1` — the coach's API key, if you set one

## Keyboard

| Key | Action |
| --- | --- |
| `1`–`9` | Place a number |
| `0` / `⌫` | Erase the cell |
| `↑ ↓ ← →` | Move the selection |
| `N` | Toggle pencil marks |
| `C` | Toggle auto-check |
| `P` | Pause |
| `Ctrl+Z` | Undo |
| `Ctrl+Y` | Redo |

## The coach

The panel below the board answers questions about the position — why a digit is
blocked, whether your pencil marks are right, what to look at next.

The division of labour is the point: **the solver decides what is true, the
language model only puts it into words.** Every fact the model is allowed to
state — candidates for the selected cell, which unit blocks each ruled-out
digit, whether a naked or hidden single is available, which cells are in
conflict — is computed by the game first and handed over as verified ground
truth. A model reasoning unaided over a Sudoku grid invents confident nonsense;
this one is not asked to reason about the grid at all.

The solution is never sent. Everything in the context is derivable from what
the player can already see, so the coach cannot spoil the puzzle even if asked.

**Offline by default.** With no API key, a rule-based answerer phrases the same
facts. It is blunter, never wrong, and needs no network. This is the mode the
game ships in.

**Online.** Paste an Anthropic API key into the field in the coach panel and it
calls the Messages API directly from the browser (`x-api-key` plus
`anthropic-dangerous-direct-browser-access`). The key is stored in
`localStorage` on your machine and sent nowhere but `api.anthropic.com`. Note
that a browser-visible key is exposed to anything running on the page — fine
for a local single-player game, not a pattern to carry into anything shared.

The model is set by the `MODEL` constant near the coach section of the script:

```js
const MODEL = "claude-sonnet-4-6";
```

Newer models are available (`claude-sonnet-5`, `claude-opus-5`); change that one
line to switch.

## Code layout

Everything is `index.html`: design tokens and styles in the `<style>` block,
markup, then a single `<script>` divided into numbered sections — storage, grid
maths, solver and generator, game state, history, mutations, validation, timer,
statistics, rendering, input, lifecycle, and the coach.

Vanilla ES6+. The solver uses bitmask occupancy tables and a most-constrained-cell
heuristic, which keeps the uniqueness check fast enough to run dozens of times
per generated puzzle.

## Browser support

Any current browser. Uses CSS custom properties, `clamp()`, flexbox and grid,
and `fetch` for the online coach. JavaScript is required.
