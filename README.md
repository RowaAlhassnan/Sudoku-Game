<div align="center">

# Sudoku

**Every digit owns a hue.**
The board starts near-monochrome and colours itself in as you solve it.

<img src="https://img.shields.io/badge/single%20file-index.html-2A1B4A?style=flat-square" alt="Single file">
<img src="https://img.shields.io/badge/dependencies-none-3FA34D?style=flat-square" alt="No dependencies">
<img src="https://img.shields.io/badge/build%20step-none-3FA34D?style=flat-square" alt="No build step">
<img src="https://img.shields.io/badge/vanilla-ES6%2B-8A45D9?style=flat-square" alt="Vanilla ES6+">

<img src="https://img.shields.io/badge/1-E8384F?style=flat-square&labelColor=E8384F&color=E8384F" alt="1">
<img src="https://img.shields.io/badge/2-F5821F?style=flat-square&labelColor=F5821F&color=F5821F" alt="2">
<img src="https://img.shields.io/badge/3-B8930A?style=flat-square&labelColor=B8930A&color=B8930A" alt="3">
<img src="https://img.shields.io/badge/4-3FA34D?style=flat-square&labelColor=3FA34D&color=3FA34D" alt="4">
<img src="https://img.shields.io/badge/5-00A19A?style=flat-square&labelColor=00A19A&color=00A19A" alt="5">
<img src="https://img.shields.io/badge/6-1487D8?style=flat-square&labelColor=1487D8&color=1487D8" alt="6">
<img src="https://img.shields.io/badge/7-3F5BD9?style=flat-square&labelColor=3F5BD9&color=3F5BD9" alt="7">
<img src="https://img.shields.io/badge/8-8A45D9?style=flat-square&labelColor=8A45D9&color=8A45D9" alt="8">
<img src="https://img.shields.io/badge/9-D93BA0?style=flat-square&labelColor=D93BA0&color=D93BA0" alt="9">

</div>

```
┌───────┬───────┬───────┐   Clues stay in ink.
│ 5 3 · │ · 7 · │ · · · │   Your entries take the digit's colour,
│ 6 · · │ 1 9 5 │ · · · │   and so do its pencil marks, its key on
│ · 9 8 │ · · · │ · 6 · │   the pad, and every cell it highlights.
├───────┼───────┼───────┤
│ 8 · · │ · 6 · │ · · 3 │   The palette is functional, not
│ 4 · · │ 8 · 3 │ · · 1 │   ornamental: colour is how the board
│ 7 · · │ · 2 · │ · · 6 │   tells you what it knows.
├───────┼───────┼───────┤
│ · 6 · │ · · · │ 2 8 · │
│ · · · │ 4 1 9 │ · · 5 │
│ · · · │ · 8 · │ · 7 9 │
└───────┴───────┴───────┘
```

---

## Quick start

There is nothing to install and nothing to build.

```sh
open index.html                 # macOS — just open the file
python3 -m http.server 8000     # or serve it, then visit localhost:8000
```

> [!NOTE]
> On `file://` origins `localStorage` can throw, so storage falls back to an
> in-memory store for the session. The game still plays; statistics and saved
> games just won't survive a reload.

---

## What's in it

<table>
<tr><td width="50%" valign="top">

**The puzzle**

Generated in the browser, never fetched. A randomised backtracking solver
builds a full grid, then clues are dug out one at a time — a clue is only
removed when the grid still has **exactly one** answer.

| Tier | Clues |
| :--- | ---: |
| Easy | 45 |
| Medium | 34 |
| Hard | 27 |

</td><td width="50%" valign="top">

**Playing**

- Pencil marks, one 9-bit mask per cell
- Undo / redo history
- Auto-check against the solution
- Conflict highlighting across row, column and box
- Peer highlighting for the selected cell
- Keypad badges counting each digit still missing
- Timer with pause — the board hides while stopped
- Reset, new puzzle, win overlay with confetti

</td></tr>
</table>

---

## Keyboard

| | | | |
| :-- | :-- | :-- | :-- |
| `1`–`9` — place a number | `N` — pencil marks | `Ctrl+Z` — undo | `↑ ↓ ← →` — move selection |
| `0` `⌫` — erase | `C` — auto-check | `Ctrl+Y` — redo | `P` — pause |

---

## The coach

The panel under the board answers questions about the position: why a digit is
blocked, whether your pencil marks are right, what to look at next.

> [!IMPORTANT]
> **The solver decides what is true. The model only puts it into words.**
> Candidates, which unit blocks each ruled-out digit, whether a naked or hidden
> single exists, which cells conflict — all computed by the game first and
> handed over as verified ground truth. A model reasoning unaided over a Sudoku
> grid invents confident nonsense, so this one is never asked to reason about
> the grid at all.

The solution is never sent. Every fact in the context is derivable from what
the player can already see, so the coach cannot spoil the puzzle even if asked.

**Offline** is the default and needs no network — a rule-based answerer phrases
the same facts. Blunter, never wrong.

**Online** — paste an Anthropic API key into the coach panel and it calls the
Messages API straight from the browser. The key is stored in `localStorage` on
your machine and goes nowhere but `api.anthropic.com`.

> [!WARNING]
> A browser-visible key is readable by anything running on the page. Fine for a
> local single-player game; not a pattern to carry into anything shared.

The model is one constant in the coach section of the script:

```js
const MODEL = "claude-sonnet-4-6";   // newer: claude-sonnet-5, claude-opus-5
```

---

## Under the hood

Everything lives in [`index.html`](index.html) — design tokens and styles, then
markup, then a single script in numbered sections:

<div align="center">

`storage` · `grid maths` · `solver + generator` · `state` · `history`
`mutations` · `validation` · `timer` · `statistics`
`rendering` · `input` · `lifecycle` · `coach`

</div>

The solver runs on bitmask occupancy tables with a most-constrained-cell
heuristic, which keeps the uniqueness check fast enough to run dozens of times
per generated puzzle.

**Stored keys**, all local to the browser:

| Key | Holds |
| :-- | :-- |
| `sudoku.hue.game.v1` | the game in progress |
| `sudoku.hue.stats.v1` | best time per tier, games won, current streak |
| `sudoku.hue.key.v1` | the coach's API key, if you set one |

<br>

<div align="center">
<sub>Any current browser. CSS custom properties, <code>clamp()</code>, flexbox and grid, <code>fetch</code> for the online coach. JavaScript required.</sub>
</div>

---

<div align="center">

### SDAIA Academy

Built at **SDAIA Academy**.

<img src="https://img.shields.io/badge/SDAIA-Academy-00A19A?style=for-the-badge&labelColor=2A1B4A" alt="SDAIA Academy">

</div>
