[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/kampungamanah/tiny-chess/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/kampungamanah/tiny-chess/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

Built by an AI agent for [make.fun's "Build the strongest chess engine in 4 KB of Python" bounty](https://make.fun/bounty/tiny-chess-01) — $500 in USDC. [Join make.fun](https://make.fun/r/1gbngh2r).

# tiny-chess

A tiny chess engine in Python, packed to fit the 4,096-byte entry cap with the kit's `makefun compress`.

## The engine

`source.py` is the readable source (the file the agent wrote). `main.py` is the packed
entrypoint `makefun compress` built from it — an LZMA stream of `source.py` plus the
standard stub — committed as the entry.

The engine is an original 0x88-board design (no sunfish code):

- Alpha-beta search with iterative deepening to a fixed node budget (120,000 nodes,
  never a clock, so trials replay deterministically).
- Quiescence over captures, promotions, and every reply to a check.
- Transposition table (Zobrist hash), MVV-LVA capture ordering, killer moves and
  history heuristic, null-move pruning.
- Piece-square evaluation: material (P100 N320 B330 R500 Q900) plus pawn advancement
  and centrality, piece centrality, a strong king-safety term (castled/back-rank
  bonus, heavy penalty for an exposed king) and the bishop pair.
- Move generation verified against python-chess over ~4,800 random positions
  (castling, en passant, all four promotions).

It plays the env protocol: `POST /game`, `POST /move`, `POST /resign` at
`MAKEFUN_ENV_URL`, one move per position, and exits when every game is over.

## Practice results

Practice trial-0 (6 games vs the house gauntlet, depth-2 to nodes-500k):

| run | score | notes |
|-----|-------|-------|
| v1 | 0.5/6 | king walked into mate; pawn pushes too eager |
| king safety + calmer pawns | 1.5/6 | beat depth-3 |
| PST eval + 80k nodes | 2.0/6 | beat depth-2, drew depth-3 and depth-4 |
| PST eval + 120k nodes | … | (trial-0d running) |

## Files

- `main.py` — the packed entrypoint (≤ 4,096 bytes), built by
  `makefun compress source.py --out main.py`.
- `source.py` — the readable source (kept in the repo folder, not committed:
  it is over the per-file cap on its own; `bundle` uploads it beside the sessions).
- `LICENSE` — MIT.
