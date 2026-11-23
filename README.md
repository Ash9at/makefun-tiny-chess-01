# tiny-chess-01 entry

A chess engine in a single `main.py` of 4,047 bytes (limit: 4,096), written for
the make.fun bounty "Build the strongest chess engine in 4 KB of Python".

## How it works

`main.py` is a self-unpacking program: an LZMA-compressed (FORMAT_ALONE,
preset 9 extreme) + base85-encoded copy of the engine source, unpacked and
executed at startup via a one-line loader. The decompressed engine uses only
the Python 3.13 standard library and reads no other file.

The engine itself:

- Negamax alpha-beta search with iterative deepening to a **fixed budget of
  45,000 nodes per move** (fully deterministic: no clock, no randomness), so
  replayed trials reproduce move-for-move.
- Transposition table with exact/lower/upper bound flags, PVS (principal
  variation search), late move reductions, null-move pruning, check
  extensions, killer/history move ordering, and in-search repetition
  detection.
- Quiescence search over captures and promotions with check evasions,
  stand-pat, and delta pruning.
- Evaluation: material + piece-square tables with a middlegame/endgame
  phase blend (separate endgame king table) and a bishop-pair bonus;
  tables are stored as half-tables (left-right symmetry) in a hex string
  and expanded at startup.
- Speaks the competition environment protocol: polls `MAKEFUN_ENV_URL`
  `/game`, answers `/move` with UCI moves (castling as the king move,
  promotions lowercase), exactly as the kit specifies.

## Local verification

Measured with the kit's own environment + scorer against the house
gauntlet (6 games per practice trial): **Elo 1279 on trial-0**
(2 wins, 2 draws, 2 losses) and **Elo 1279 on trial-1** (3 wins, 3 losses),
above the 1,154 practice floor and close to the ~1,365 house-baseline
practice mean; individual 6-game trials vary widely (a 30,000-node build
scored 1451 / 876 / 1099 on trials 0 / 1 / 2). Trial wall time is
480-630 s per 6 games, well inside the 1,800 s trial limit.
Perft suites (start position, Kiwipete, en-passant and promotion positions)
all pass.

## License

MIT — see `LICENSE`.
