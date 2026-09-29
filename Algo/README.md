# 🐍 Snake Algo — Pathfinding

> Part 2 of the [Snake 2026 project](../README.md) — pathfinding algorithms.

Project built for the **NLPF 2026** course, second assignment (due 22/09): pilot the
Snake with a deterministic pathfinding algorithm instead of reinforcement learning. The
base game (`serpent-algo.py`) is imported as-is, unmodified — clock, grid size and
scoring are left untouched. We reused the same socket as Part 1 (no-argument launch for
the instructor, score/time logs, fast mode `xN` with counted time `x N`, headless `eval`
mode), and put all algorithms in a single file so they can be compared fairly on the
same grid, the same torus (edges wrap around), and the same random seeds.

## What the project does

Two algorithms are the course's own; four are ours.

- **`dijkstra`** — course algorithm: uniform-cost shortest path to the food.
- **`gbfs`** — course algorithm: Greedy Best-First Search, Manhattan-distance heuristic
  on the torus.
- **`sur`** ("safe greedy") — an intermediate step: shortest path to the food (Dijkstra),
  but only commits to it if the head can still reach its own tail after eating.
  Otherwise it just follows its tail until room opens up.
- **`hybride`** (ours) — a Hamiltonian cycle covering the whole grid acts as a safety
  net; the snake takes a shortcut across the cycle whenever it's safe to. Safety rule: a
  shortcut may never jump past the tail in the cycle's order, so the head always stays
  ahead of the body and a collision becomes impossible. **Always wins** (fills the
  entire grid), at the cost of being slow.
- **`glouton`** (ours) — no cycle: rushes straight for the food as long as it's safe
  (checked via reachability of the tail), and past `SEUIL_LONGUEUR` (60% of the grid by
  default) stops targeting food entirely and just preserves free space instead. Faster
  than the Hamiltonian cycle, but doesn't win every time.
- **`audacieux`** (ours) — built for raw speed: always beelines for the food, with a
  configurable safety check (`--controle aucun|espace|queue`, from none to strict).
  Rarely wins, but wins fast — meant to be run thousands of times in parallel
  (`chercher.py`) to fish out a lucky, very fast win.

`chercher.py` plays thousands of games in parallel (multiprocessing), each identified by
a seed, to find the fastest guaranteed win; `graphique.py` plots score/time across
algorithms into `evolution.svg` and `mesures.csv`.

## Results

Official measurements, 50 identical games per algorithm (see the timeline below for the
full tuning story):

| Algorithm | Average score | Record | Wins / 50 | Notes |
|---|---:|---:|---:|---|
| `dijkstra` | 65.9 | 100 | 0 | explores 54.4 cells/move |
| `gbfs` | 68.1 | 105 | 0 | explores 13.1 cells/move (vs Dijkstra) |
| `sur` | 104.4 | 177 | 0 (42 deaths, 8 loops) | "tail reachable" check alone isn't enough |
| `hybride` | 223 | 223 | **50 / 50** | grid fully filled every time, ~1237.6s/game, 0.02ms compute/move |
| `glouton` (speed-tuned) | — | 223 | rare, but fastest win found: **763s** | found via `chercher.py` over thousands of parallel games |

## How to run

```bash
cd Algo

# Real game with the default algorithm (glouton)
python snake-algo.py

# Real game with the guaranteed-win algorithm
python snake-algo.py --algo hybride

# Fast mode (displayed time = real time x factor)
python snake-algo.py rapide 20 --algo glouton

# Evaluate an algorithm over N games, no display
python snake-algo.py eval --algo dijkstra --parties 100 --seed 1

# Score/time chart across algorithms (writes mesures.csv + evolution.svg)
python graphique.py --parties 10 --seed 1

# Search thousands of seeds in parallel for the fastest guaranteed win
python chercher.py --algo audacieux --parties 2000 --controle espace
```

Fastest win found so far:

```bash
python snake-algo.py rapide 20 --algo glouton --graine 1127 --seuil 0.6 --patience 0
```

## Folder structure

```
Algo/
├── serpent-algo.py   # base game (unmodified)
├── snake-algo.py       # the 6 algorithms + demo, rapide, eval modes
├── chercher.py           # parallel search for the fastest guaranteed win
└── graphique.py            # score/time comparison chart across algorithms
```

## Timeline / changelog

*(HH:MM: Activity / Observation / Hypothesis / Answer)*

19:50: Start — firing up our friend Claudius Opus V.V and preparing its context.
19:57: Structure decision: reuse the socket validated last week (base game imported
unmodified, no-argument launch for the instructor, score/time logs, fast mode `xN` with
counted time `x N`, headless eval mode), and put all three course algorithms in a single
file so they can be compared.
19:58: Dijkstra and GBFS coded on the torus (edges wrap around, so distance sometimes
goes through the edge); measured over 50 identical games: Dijkstra average 65.9 record
100, GBFS average 68.1 record 105.
19:59: Matches the course: GBFS explores 13.1 cells/move vs 54.4 for Dijkstra, but
neither ever finishes the game (0 wins / 50) — both rush the food and trap themselves.
20:00: Hypothesis: since the best score wins and time only breaks ties, we should aim
for a win (filling the grid) rather than the best average score.
20:00: Key idea: a circuit visiting every cell exactly once is impossible on a closed
15x15 grid (225 cells, an odd number), but it exists here precisely because the edges
wrap around; built a cycle (each row swept rightward through the wrap-around edge, then
one row down) and verified it: 225 cells, no repeats, every step valid.
20:01: Intermediate "safe greedy" version (shortest path to the food, accepted only if
the head can still reach its tail after eating): bug, score 0 everywhere; cause found —
the safety check started from the head's own cell, always occupied by the snake itself.
20:01: Bug fixed; safe greedy rises to average 104.4, record 177, but 0 wins / 50 games
(42 deaths, 8 loops): the "tail reachable" check alone doesn't guarantee anything.
20:02: Our "hybride" algorithm: follow the Hamiltonian cycle, and take a shortcut
whenever it's safe; safety rule: a shortcut must never jump past the tail in the cycle's
order, so the head always stays ahead of the body and collision becomes impossible.
20:02: First hybride run: 20 wins / 20, score 223 (grid full), 1377s of play on average.
20:03: Tuning search: scoring shortcuts by "closest cell to the food in Manhattan
distance" or by "take the biggest shortcut" both give 0 wins (the snake overshoots the
food and loops); kept "closest cell to the food in the cycle's own order" instead.
20:04: Counter-intuitive finding: stopping shortcuts earlier (0.45 free-cell threshold
instead of 0.25) actually saves time, because late shortcuts jam the head right against
the tail and force long detours afterwards.
20:04: A 1-cell margin before the tail eventually killed the snake (score 103) over 25
games; margins of 2 and 3 win 60 / 60; decision: margin 3 and threshold 0.45, since score
outranks time and the worst game is still shorter (1437s vs 1552s).
20:05: Official measurement over 50 games: hybride wins 50 / 50, record 223 in 1237.6s,
0.02ms compute per move (budget: 200ms per frame at 5 FPS) — the clock stays unaffected.
20:06: `python Algo/snake-algo.py` launches hybride in a real game with no arguments;
the tracking chart (`evolution.svg`) and measurements (`mesures.csv`) are generated by
`python Algo/graphique.py`.
20:11: Hamiltonian method wins (223 points) in 21 minutes.
20:12: Tom starts barking at Claudius Opus V.V.
20:16: Now looking for an alternative to the Hamiltonian cycle — trying to find
something faster, even if less consistent.
20:21: Safer greedy: 1 win (223 points) out of 30 games, in 1070s.
20:55: An optimized greedy variant (rush the food, but only if the tail stays
reachable, and past 55% of the grid stop targeting food altogether) wins 10% of games in
950s.
20:55: Tom grabs the keyboard and tries this idea: "once we're past 55%, also try to
reduce the number of holes so future apples land in front of the snake instead of stuck
in the middle of its own body" — spoiler: it doesn't work and performs worse than the
previous version.
21:17: Claude goes off on its own and tries another trick, "hug the walls and its own
body": wins in 992s (2 / 300).
21:18: Now trying to run as many games as possible with an algorithm that has a low win
rate but is as fast as possible.
21:19: Win in 835s using the boosted greedy, running 20,000 games at once on the M3 Pro
MacBook — going all out (Tom is annoyed a Mac runs this well).
21:21: 763 seconds for the win.

Best seed (run from inside `Algo/`, as above):
`python snake-algo.py rapide 20 --algo glouton --graine 1127 --seuil 0.6 --patience 0`
