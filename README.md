# tiny-chess-01 — a chess engine in 4,041 bytes of Python

`main.py` is the whole entry: an lzma-compressed payload (an 8.0 KB engine)
plus a two-line unpacker, within the 4,096-byte limit. Standard library
only; it reads no files and calls no APIs for moves.

The engine: 0x88 board, make/unmake, full legal move generation (castling,
en passant incl. pins, promotions). Negamax with PVS, transposition table,
null-move pruning, late-move reductions, check extensions and quiescence
search; killer-move ordering; material + piece-square-table evaluation
maintained incrementally.
Deterministic: a fixed budget of 150,000 nodes per move, fixed-seed Zobrist
keys, no wall-clock or unseeded randomness anywhere.

The client speaks the trial protocol: it polls `POST /game` on
`MAKEFUN_ENV_URL`, reconstructs each position from `start_fen` + the move
list, validates its move against the server's `legal_moves`, and answers
with `POST /move` in UCI. It never resigns; on any internal error it plays
the first legal move rather than forfeiting the trial.
