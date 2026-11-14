# tiny-chess-01 — a complete chess engine in 4,058 bytes of Python

## What this is

A chess engine and its tournament client packed into a single Python
file. `main.py` holds an lzma-compressed engine (about 8 KB of source)
behind a two-line unpacker; the whole repository stays inside the
competition's size caps — 4,096 bytes for the entry file itself and
8,192 bytes for the full snapshot. Only the Python standard library is
used, and the program reads nothing but the brief and the game
environment it is pointed at.

## How it plays

- Board: 0x88 array with make/unmake and a full undo record (captured
  piece, castling rights, en-passant square, halfmove clock, hash key
  and the incremental score are all restored on take-back).
- Move generation is fully legal: castling through check is rejected,
  en-passant pins are handled, promotions expand to all four pieces.
  Perft results match the standard reference counts, and a legality
  suite (26 tests) passes on the shipped file.
- Search: iterative-deepening negamax with principal-variation search,
  a transposition table, null-move pruning, late-move reductions,
  check extensions and a quiescence search over captures, promotions
  and check evasions. Repetitions and the hundred-ply rule score as
  draws inside the search.
- Ordering: transposition-table move first, then captures by
  MVV-LVA, then the killer move stored for the ply.
- Evaluation: material plus piece-square tables, kept incrementally
  in make/unmake so a node never rescans the board; king tables swap
  between middlegame and endgame sets on a material test.

## Determinism

Every move is a pure function of the position, the game history and a
fixed node budget of 150,000 nodes per move. Hash keys come from a
fixed-seed generator; there is no wall clock and no unseeded
randomness anywhere, so a replayed trial reproduces the live one move
for move.

## Development note

Four candidate improvements were played off against the previous
version in equal-budget self-play sprints on the practice openings.
Killer-move ordering won its sprint clearly and is in. A transplanted
tuned evaluation, quiescence delta pruning and a second killer slot
each lost theirs and were left out.
