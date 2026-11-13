# judaboyk tiny-chess v14

A 4 KB chess engine for the make.fun tiny-chess-01 competition. One file,
`main.py` (4,057-byte engine, 4,074 on disk), standard library only, deterministic — no model
calls, no wall clock, no unseeded randomness.

## The engine

Alpha-beta search with late move reductions (LMR), 75,000 nodes per move.
The v14 change is small and the story behind it is the whole point: in v13
the LMR code was dead. The move counter was assigned `1` after the first
move instead of being incremented, so the "4th move and later" reduction
condition could never fire — v11, v12 and v13 all ran with LMR silently
off, and the earlier "LMR changes nothing" reading was void. v14 fixes
that one character, then raises the bar: reduce by 1 only from the 7th
move at depth 6 or more (conservative LMR). The rest of the engine —
piece-square tables, quiescence, null-move pruning — is unchanged from v13.

Practice Elo (grade.sh, 3 trials of 8 games vs the house gauntlet):
trial 0: 1,464 (4W/1D/3L), trial 1: 1,669 (4W/4D/0L, unbeaten), trial 2:
1,253 (2W/2D/4L). Mean 1,462, up from v13's 1,298 and above the ~1,325
house baseline; all three trials clear the 1,154 floor.

## Running it

The harness starts the engine and talks to it over HTTP: `POST /game {}`
starts a game, `POST /move {"game": ..., "move": "e2e4"}` plays a move in
UCI notation, `POST /resign {"game": ...}` resigns. The engine reads its
prompt from `/input/prompt.txt` and the environment URL from
`MAKEFUN_ENV_URL`. Run it directly with:

```
python3 main.py
```

Per-move limit is 10,000 ms (late loses); games cap at 200 plies and are
then adjudicated by material (pawn 1, knight 3, bishop 3, rook 5, queen 9).
Draws are immediate on stalemate, insufficient material, a third
occurrence, or 100 plies without a capture or pawn move.

## Files

- `main.py` — the whole engine, packed: a one-line wrapper that
  LZMA-decompresses a base85 blob (7,655 bytes decoded) and runs it.
  The decoded source is the readable engine this entry was graded from.
- `LICENSE` — MIT.
- `.makefun/owner.json` — the payout wallet for this entry.

## Notes

Built and graded locally with the competition kit's `grade.sh` (practice
trials only). The engine is deterministic: same position, same move,
every time — the final scoring replays entries byte for byte.
