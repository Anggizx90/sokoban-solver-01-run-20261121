# Roomwise Push-Optimal Sokoban Solver

[![make.fun](https://make.fun/badge/sokoban-solver-01-run-20261121.svg)](https://make.fun/bounty/sokoban-solver-01-run-20261121)
[![rank](https://make.fun/badge/sokoban-solver-01-run-20261121/Anggizx90/sokoban-solver-01-run-20261121/rank.svg)](https://make.fun/bounty/sokoban-solver-01-run-20261121)
[![reward](https://make.fun/badge/sokoban-solver-01-run-20261121/Anggizx90/sokoban-solver-01-run-20261121/reward.svg)](https://make.fun/bounty/sokoban-solver-01-run-20261121)

A push-optimal Sokoban solver built for make.fun's
["Build the best Sokoban solver" bounty](https://make.fun/bounty/sokoban-solver-01-run-20261121)
($100 in USDC). One program answers a whole set of 5 levels in a single run,
in as few pushes as possible — the scoring cliff makes this the whole game:
a solution at exactly the push-optimal count P\* earns 1000 points, while even
one push over earns under half.

## How it works

Levels are a 1-wide hallway plus 1–4 rooms, each opening onto the hallway
through a single door. A box pushed out of a room into the hallway can never
be pushed back, so every box stays in its room and the rooms are independent:
the level optimum is the sum of the room optima. The solver therefore splits
each level into rooms and solves each room on its own.

Each room is solved by breadth-first search over **push states** (not steps):
a state is the boxes' squares plus the player's reachable region (named by its
lowest square), so walking is free and the first solved state BFS meets is
push-optimal. Two prunings keep the search small without ever cutting a
solvable state:

- **Dead squares** — a square from which no sequence of pushes, ignoring the
  other boxes, can ever bring a box onto a goal (computed by pulling
  backwards from the goals).
- **Frozen 2×2 blocks** — four squares of walls/boxes containing a box off
  its goal, which can never move again.

The search is budgeted by work only (a fixed 400,000-state cap per room),
never by the wall clock, so two runs write byte-identical output. If a level
does not split into rooms, the same push BFS runs over the whole level as a
fallback.

## Results

Practice set (`grade-local`, process driver): **1,000,000 / 1,000,000** —
every instance solved at exactly P\* (i001: 13 pushes, i002: 16, i003: 29,
i004: 47, i005: 60), in ~1.3 s for the whole set. The house baseline
(plain-astar 1.0.0) scores 200,000 on the same set (only i001 solved).

Robustness: on 30 freshly drawn instances from the kit's own generator
(6 unseen sets), the solver scored 30,000/30,000 — every instance at P\*.

Determinism verified: two full runs on the practice set produce
byte-identical output, and `grade-local`'s L1 regrade reproduced byte for
byte.

## Run it

Requires Node 22 (erasable TypeScript only, no dependencies):

```sh
node main.ts
```

It reads `/input/<id>/level.txt` for each instance and writes
`/output/<id>/solution.txt` (player moves `u`/`d`/`l`/`r`, one letter per
step). For a local run against the kit's practice set:

```sh
MAKEFUN_INPUT_DIR=practice/input MAKEFUN_OUTPUT_DIR=out node main.ts
```

## Files

- `main.ts` — the whole solver (single file, no dependencies)
- `LICENSE` — MIT

SPDX-License-Identifier: MIT
