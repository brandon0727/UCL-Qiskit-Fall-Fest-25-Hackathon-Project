# Quantum Battleships

Turn the Elitzur–Vaidman “bomb tester” + **Quantum Zeno** effect into a duel-symmetric Battleships-style game.  
Choose a **strategy** (`rows`, `cols`, `binary`) and **Zeno depth** `N`, fire a probe at a subset of cells, and get
**Found / Empty / Explosion**. Score with **EV**; optimize `(strategy, N)` under gate/turn budgets.

## Features
- Single or two-player, **duel-symmetric** rules.
- Strategies: `rows`, `cols`, **`binary`** (group testing refinement).
- **Zeno probe** circuit (Qiskit): mid-circuit measure+reset; `N` controls risk vs. cost.
- Scoring & costs:
  - `EV = #Found / max(#Explosions, 1)`
  - `Turns = Probes + P * #Explosions`
  - `Gates ≈ Probes * N`
- **Optimizer**: sweep `(strategy, N)`, average over seeds, respect **Gates/Turns** budget.
- Plots: EV vs `N`, strategy×`N` heatmap, density×`N` heatmap.

##  Gameplay & Scoring

- **Board:** `M×M` with `S` hidden 1×1 ships (uniform, non-overlapping).
- **Probe subsets by strategy:**
  - **rows / cols:** probe a full line; **split** the segment on positive signals (Found/Explosion).
  - **binary:** probe a rectangle; **split along the longer side** on positives (group testing).
- **Outcomes:**
  - **Explosion:** any ancilla = 1 during Zeno steps → remove one random ship inside the probed subset; add `+P` to **Turns**.
  - **Found:** no explosion, final path qubit = 0 (interaction-free evidence; no ship removed).
  - **Empty:** no explosion, final path qubit = 1.
- **Metrics:**
  - `EV = #Found / max(#Explosions, 1)`
  - `Turns = Probes + P * #Explosions`
  - `Gates ≈ Probes * N`
- **Win:** sink all opponent ships, or at round-limit compare: **EV** → lower **Turns** → lower **Gates** → fewer **Probes** → **Draw**.

---

##  Optimizer

- **Inputs:** `M`, density/`S`, strategies, `N_list`, round limit `R`, penalty `P`, trials `T`, optional budget `Gates ≤ B` or `Turns ≤ B`.
- **Process:** symmetric duels across seeds → averages of `{EV, Turns, Gates, Probes}`.
- **Rank:** EV (desc) → constrained cost  → Probes  → smaller `N`.
- **Good defaults:** `M = 8–12`, density `10–30%`, `N = {4,8,12,16,24,32}`, `R = 16–24`, `P = 1–3`, `T = 6–10`.
