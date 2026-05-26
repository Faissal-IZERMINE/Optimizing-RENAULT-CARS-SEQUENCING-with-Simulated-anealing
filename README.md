# Renault car sequencing — KIRO 2024 challenge

Simulated-annealing heuristic for the **Renault Car Sequencing** problem,
solved during the **KIRO 2024** academic optimisation challenge run in
partnership with Renault. The task is to order a batch of vehicles through
a chain of workshops (body, paint, assembly, ...) while respecting
combinatorial constraints — minimising the total constraint-violation cost.

> KIRO is an annual French inter-school operations-research competition
> (École Polytechnique / ENSTA / CentraleSupélec / etc.) where each team
> tackles a real industrial instance provided by a corporate partner.

---

## 1. The problem

Each JSON instance under this folder (`tiny.json` … `large_2.json`)
describes:

- **`shops`** — an ordered list of workshops, each with a *resequencing
  lag* (number of slots before a constraint becomes "active" downstream).
- **`vehicles`** — a list of vehicles with a `type` (regular / special) to
  schedule.
- **`parameters`** — global problem parameters (batch limits, weights).
- **`constraints`** — per-shop constraints, each with a `cost` paid when
  violated. The instances I solved include the **`batch_size`** constraint
  family: a subset of vehicles must appear in a contiguous window of size
  between `min_vehicles` and `max_vehicles` in the named shop.

A **solution** is a permutation of the vehicle list. The objective is the
sum of per-constraint costs incurred by the permutation, evaluated after
resequencing lag.

Instances ship in 4 size buckets: `tiny.json`, `small_*.json`,
`medium_*.json`, `large_*.json` (use `tiny.json` to sanity-check any
change to the cost function before running on the larger instances).

## 2. The approach: simulated annealing

For these objectives the cost is piecewise-constant in the permutation
and expensive to compute exactly via MILP at the larger sizes. Simulated
annealing is a strong fit:

1. **Initial solution**: a random permutation, or a greedy one (insert
   high-violation vehicles into low-cost slots first).
2. **Neighbourhood**: random pairwise swap (or 2-opt-style segment
   reversal) of vehicles in the sequence.
3. **Acceptance**: standard Metropolis criterion
   `accept(Δcost) = min(1, exp(-Δcost / T))`.
4. **Cooling**: geometric schedule `T ← α · T` with `α ∈ [0.95, 0.999]`.

The implementation lives entirely in
[`KIRO-this-year.ipynb`](KIRO-this-year.ipynb). It loads an instance,
defines the cost function from the JSON spec, runs the SA loop with a
progress bar, and saves the best permutation found.

The notebook has cell outputs committed, so you can read the
cost-trajectory plots without re-running the solver.

## 3. Why simulated annealing here

This is a classic case where **the choice of heuristic matters less than
the formulation of the cost function**. Once the cost is correctly
implemented (matching the resequencing-lag semantics), almost any
local-search heuristic finds high-quality solutions in seconds on the
small/medium instances and in minutes on the large ones. SA was chosen
over (e.g.) tabu search or genetic algorithms for its minimal
hyperparameter footprint (`T₀`, `α`, `n_iters`) and the fact that the
Metropolis acceptance does not need any problem-specific reasoning.

## 4. Running

```bash
pip install jupyterlab tqdm
jupyter lab
```

Open `KIRO-this-year.ipynb`. The notebook's first cell hardcodes a path
to the instance — change `instance = 'medium_1'` and the input path to
point to any of the `*.json` files in this folder.

## 5. What this is *not*

- **Not a production solver** — KIRO instances are small enough that
  exact methods (MILP via OR-Tools / Gurobi) are competitive; SA was
  chosen for tractability under contest time limits.
- **Not a research contribution** — the SA framework is textbook
  (Kirkpatrick et al., 1983). The actual work is the cost-function
  derivation from the contest spec and the SA loop tuning.

## 6. Acknowledgements

Problem statement, instances, and scoring rules are property of the
KIRO 2024 organisers and Renault. The repo contains my solver code and
the public instances; refer to the KIRO website for the official
problem statement and benchmark results.
