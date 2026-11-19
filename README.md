# tiny-chess-01: a 4 KB chess engine

[![make.fun](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/tulankganteng/tiny-chess-4k/rank.svg)](https://make.fun/bounty/tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/tulankganteng/tiny-chess-4k/reward.svg)](https://make.fun/bounty/tiny-chess-01)

`main.py` (4,066 bytes) is the whole engine: Python 3.13, standard
library only, no model calls, no clock. It plays the make.fun env
(`POST /game`, `POST /move`) with a fixed node budget per move, so
every trial replays move for move.

## How it plays

- 0x88 board, full legal move generation (castling, en passant,
  promotion, check/checkmate/stalemate).
- Negamax with alpha-beta and principal variation search (PVS),
  iterative deepening to a fixed node budget (140,000 nodes a move
  early, 85,000 / 55,000 in long games), a transposition table,
  killer moves, history heuristic, MVV-LVA capture ordering.
- Null-move pruning (R=3) with an evaluation gate, late-move
  reductions.
- Quiescence search over captures and promotions, and over all
  replies when in check, with delta pruning.
- Evaluation: material (100/320/330/500/900) + piece-square tables
  + bishop pair, from the side to move's point of view.
- Repetition avoidance: positions already seen twice in the game
  score as draws inside the search.

The source is golfed then packed as an LZMA payload (base85) with a
one-line unpacker; the payload and unpacker together are the 4,066
bytes. Development source and tests live outside this repository.

## Local results

- Perft verified against python-chess (startpos, Kiwipete, positions
  with castling/en passant/promotion): all counts exact.
- 16/16 wins by checkmate vs the greedy-capture starter over the
  trial-0 openings (both colours); mean move time 1.29 s, max 2.56 s
  (limit 10 s).
- Real kit env + official scorer, current entry v5d: practice
  trial-1 **Elo 1,362** (5 won, 5 drawn, 6 lost) and practice
  trial-2 **Elo 1,253** (3 won, 6 drawn, 7 lost) — mean practice
  Elo ≈ **1,308**, vs the sunfish house-baseline bar of 1,360.
  Trial-1 sits exactly at the bar; the final is judged on 6 fresh
  instances, not the practice mean.
- Deterministic: the same position always yields the same move.

## License

MIT, see LICENSE.
