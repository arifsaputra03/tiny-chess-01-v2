# Tiny Chess v2

A chess engine in at most 4,096 bytes of Python, written for make.fun's
`tiny-chess-01` bounty. Version 2 of the entry in `tiny-chess-01`
(v1: best practice Elo 1654 on trial-2). It plays the house gauntlet
through the env at `MAKEFUN_ENV_URL`: `POST /game` serves the game in
play, `POST /move` plays a move in UCI, and the loop ends when the env
answers `done`.

## How it plays

- 0x88 board with make/unmake, Zobrist hashing and an occupied-squares
  list so move generation and evaluation only visit pieces on the board.
- Negamax alpha-beta with iterative deepening, a transposition table,
  two killer moves per ply, a history heuristic, MVV-LVA capture
  ordering and a quiescence search over captures, promotions and every
  reply to a check (with delta pruning).
- Principal-variation search and late-move reductions for quiet moves.
- Null-move pruning (never in check, never without non-pawn material).
- Evaluation: material plus piece-square tables tapered between
  middlegame and endgame king tables, the bishop pair, rooks on open
  and semi-open files, and pawn structure — doubled and isolated
  pawns, and passed-pawn bonuses. Repetitions and the fifty-move rule
  score slightly negative (contempt), so the engine prefers playing on
  when it is ahead.
- Deterministic: every move searches to a fixed node budget
  (120,000 nodes), never to a clock, with no unseeded randomness, so a
  replay of a trial plays the same moves. Slowest bench move 5.75 s,
  inside the env's 10 s per-move limit.

## Run it

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:18080 python3 main.py
```

Python 3.13, standard library only. `main.py` alone is the engine: it
reads, imports and runs no other file in this repository. It is an
LZMA payload (`pack.py` packs `src.py` into `main.py` the same way v1
did: FORMAT_ALONE, minimal escaping, `.encode('l1')`).

## Size

`main.py` is 4,048 bytes as committed (cap 4,096) and the whole
repository is under the 8,192-byte cap, comments and whitespace
included.

## Measured results

Practice trials against the kit's env, scored by the kit scorer
(6 games each, first 6 gauntlet rungs). The final engine uses a
120,000-node budget:

- trial-0: 3 won, 1 drawn, 2 lost — Elo 1364, spare_plies 404
- trial-1: 3 won, 1 drawn, 2 lost — Elo 1364, spare_plies 358
- trial-2: 6 won, 0 drawn, 0 lost — Elo 2112, spare_plies 585

Across the three trials: 12 won, 2 drawn, 4 lost; average Elo 1613;
best Elo 2112. An earlier build with a 90,000-node budget scored
trial-0 4-2-0 Elo 1654, trial-1 4-0-2 Elo 1451, trial-2 4-0-2
Elo 1451 (average 1519); raising the budget to 120,000 nodes turned
trial-2 into a 6-0 sweep.

v1 for comparison (same harness): trial-0 3-1-2 Elo 1364, trial-1
3-1-2 Elo 1364, trial-2 5-0-1 Elo 1654 (reproduced locally 2026-11-15:
5 won, 0 drawn, 1 lost, spare_plies 442, wall 686 s).
