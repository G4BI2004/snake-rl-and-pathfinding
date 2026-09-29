# 🐍 Snake 2026 — NLPF

Two-part Snake AI project for the **NLPF 2026 — Reinforcement Learning with Python**
course. The instructor's base game (`serpent-algo.py`, an identical copy sits in both
`IA/` and `Algo/`) is never modified in either part: clock, 15×15 grid size and scoring
are left untouched, as required for fair comparison between groups.

- **[`IA/`](IA/README.md)** — Part 1: a Deep Q-Learning agent that learns to play Snake
  through reinforcement learning (PyTorch).
- **[`Algo/`](Algo/README.md)** — Part 2: the game piloted by pathfinding algorithms
  (Dijkstra, GBFS) plus a couple of our own, including one that always wins.

## Authors

- Gabriel Franchi — [G4BI2004](https://github.com/G4BI2004)
- Tom Archambaud — [TomVendee](https://github.com/TomVendee)

## Repository structure

```
.
├── IA/                 # Part 1 — Reinforcement Learning (Deep Q-Learning)
│   ├── serpent-algo.py  # base game (unmodified)
│   ├── snake-ia.py       # Game / Model / Agent + train, demo, eval
│   ├── model/            # trained models + score history
│   └── README.md         # architecture, state/reward design, results, full log
└── Algo/               # Part 2 — Pathfinding algorithms
    ├── serpent-algo.py  # base game (unmodified)
    ├── snake-algo.py     # the 6 algorithms + demo, rapide, eval modes
    ├── chercher.py        # parallel search for the fastest guaranteed win
    ├── graphique.py        # score/time comparison chart across algorithms
    └── README.md            # algorithm-by-algorithm log and results
```

## Part 1 — IA (Deep Q-Learning)

A DQN agent (PyTorch) learns to play Snake from scratch: 14-value state (danger,
direction, food, free space), epsilon-greedy exploration, experience replay, Bellman
equation. Best result in a real game: **150 apples**.

```bash
cd IA
python snake-ia.py                              # real game, pre-trained model
python snake-ia.py rapide 20                     # same, sped up x20
python snake-ia.py train --parties 300 --seed 2  # train a new model
python snake-ia.py eval --parties 100 --seed 1   # evaluate, no display
```

Details: [IA/README.md](IA/README.md).

## Part 2 — Algo (pathfinding)

The two course algorithms (Dijkstra, Greedy Best-First Search) compared, plus a "safe
greedy" variant, and two of our own: **hybride**, a Hamiltonian-cycle-based algorithm
that always wins (fills the whole grid, 223 points, 10/10 games in testing), and
**glouton**/**audacieux**, faster variants tuned to win the speed-run race at the cost
of a lower win rate.

```bash
cd Algo
python snake-algo.py                                # real game, default algo (glouton)
python snake-algo.py --algo hybride                  # guaranteed-win algorithm
python snake-algo.py eval --algo dijkstra --parties 100 --seed 1
python graphique.py --parties 10 --seed 1            # score/time chart, all algorithms
python chercher.py --algo audacieux --parties 2000   # search for the fastest win
```

Details: [Algo/README.md](Algo/README.md).
