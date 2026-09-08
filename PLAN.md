# Dropline Duel — plan

Connect-four against a minimax alpha-beta AI with a live search-depth slider and a
visible evaluation bar.

## Goal

A small, polished browser game that makes a classic search algorithm *legible*: you
can drag the AI's search depth from 2 to 10 mid-game and watch the evaluation bar and
the "nodes searched" counter jump as the engine suddenly sees the trap you were setting.

## Features (all required)

- 7x6 board with animated disc drops (gravity ease + squash bounce) and win-line highlight
- Negamax with alpha-beta pruning, center-first move ordering `[3,2,4,1,5,0,6]`,
  iterative deepening from depth 1 up to the slider value (max 10)
- Depth slider; per-move "nodes visited / time" readout
- Vertical two-tone evaluation bar showing the engine's current score for the position
- Modes: Human vs AI, AI vs AI demo, two-player local
- Undo, hint (AI's preferred column), move history list
- Win / draw detection, including full-board draws

## Architecture

```
src/
  engine/
    bitboard.ts     position as two bitboards; each is a pair of 32-bit ints
                    (no BigInt), 7 columns x 7 bits (6 rows + sentinel row)
                    hasWon = four shift-and-AND checks per direction
    search.ts       negamax + alpha-beta, move ordering, heuristic on open
                    windows of 2 and 3, scores encode distance-to-win,
                    iterative deepening with node/time accounting
    *.test.ts       vitest specs (spec's five assertions + extras)
  game/
    useGame.ts      React reducer-ish hook: history, modes, undo, AI turn scheduling
  ui/
    Board.tsx       royal-blue punched board, animated discs, win-line highlight
    EvalBar.tsx     vertical thermometer
    Controls.tsx    mode picker, depth slider, undo/hint/new game
    History.tsx     move list + search stats
  App.tsx
```

Bitboard layout (Pascal Pons style, split into lo/hi 32-bit halves):

```
bit index = col * 7 + row      row 6 of every column is a sentinel (always 0)
col 0 -> bits 0..6, col 1 -> bits 7..13, ... col 6 -> bits 42..48
lo = bits 0..31, hi = bits 32..48
```

Win test for a direction with shift `d` (1 vertical, 7 horizontal, 6 and 8 diagonals):
`m = b & (b >> d); if (m & (m >> 2d)) won`. Shifts across the lo/hi boundary are
implemented once in a `shr(b, n)` helper.

## Milestones

1. Plan, license, git init
2. Vite react-ts scaffold + Tailwind 4 + vitest
3. Engine: bitboard + negamax with tests green
4. Board UI with animated drops, win highlight, game hook (HvAI)
5. Controls: depth slider, eval bar, stats, hint, undo, history, modes
6. Build, headless smoke screenshot, polish
7. README, publish to private GitHub repo
