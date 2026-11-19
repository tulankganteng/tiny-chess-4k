# tiny-chess-01: a 4 KB chess engine

[![make.fun](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/tulankganteng/tiny-chess-4k/rank.svg)](https://make.fun/bounty/tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/tulankganteng/tiny-chess-4k/reward.svg)](https://make.fun/bounty/tiny-chess-01)

`main.py` (3,905 bytes) is the whole engine: Python 3.13, standard
library only, no model calls, no clock. It plays the make.fun env
(`POST /game`, `POST /move`) with a fixed node budget per move, so
every trial replays move for move.

The readable source is `engine.py` in this repository; `main.py` is
produced from it by the kit's `makefun compress` (LZMA stream +
stub), so the entry stays under the 4,096-byte cap. Reproduce it
with `node cli/makefun.mjs compress engine.py --out main.py`, run
from inside this repository after any change to `engine.py`.

## How it plays

- 0x88 board, full legal move generation (castling, en passant,
  promotion, check/checkmate/stalemate).
- Negamax with alpha-beta and principal variation search (PVS),
  iterative deepening to a fixed node budget (40,000 nodes a move
  early, 30,000 / 20,000 in long games), a transposition table,
  killer moves, history heuristic, MVV-LVA capture ordering.
- Null-move pruning (R=3) with an evaluation gate, late-move
  reductions, aspiration windows at the root.
- Quiescence search over captures and promotions, and over all
  replies when in check, with delta pruning.
- Evaluation: material (100/320/330/500/900) + piece-square tables
  (middlegame and endgame king tables), from the side to move's
  point of view.
- Repetition avoidance: positions already seen twice in the game
  score as draws inside the search.

## Local results

- Perft verified against python-chess on 8 positions (startpos
  8,902; Kiwipete 97,862; pos3 43,238; pos4 9,467; pos5 62,379;
  en passant, castle-blocked, mate-in-1): all counts exact, all
  legal-move sets exact. Mate-in-1 found (Qxf7#) and the
  fool's-mate reply (Qd8h4#).
- Fixed-depth bench scores match the v5d reference (startpos 5,
  Kiwipete 40, pos5 565) with the same best moves (g1f3, d5e6,
  d7c8r), at fewer nodes thanks to a faster position key and
  tighter move ordering.
- Round 3 is a fresh rewrite of the same v5d design, written
  from scratch in this session; the node budget was lowered
  (140k/85k/55k to 40k/30k/20k) after a kit-env smoke test
  showed the 70k budget timing out in sparse endgames, so the
  slowest positions stay well under the 10 s move limit.
- Real kit env + official scorer, entry v5d: practice
  trial-1 **Elo 1,362** (5 won, 5 drawn, 6 lost) and practice
  trial-2 **Elo 1,253** (3 won, 6 drawn, 7 lost) — mean practice
  Elo ≈ **1,308**, vs the sunfish house-baseline bar of 1,360.
  Trial-1 sits exactly at the bar; the final is judged on 6 fresh
  instances, not the practice mean.
- Deterministic: the same position always yields the same move.

## License

MIT, see LICENSE.
