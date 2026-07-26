# Economic Dispatch Playground

An interactive, browser-based demonstration of **economic dispatch** — how to split a power system's load across generators at minimum fuel cost — solved live by the classic **equal-incremental-cost (λ) criterion**.

**▶ Live demo: https://panas-bhattarai.github.io/economic-dispatch-playground/**

No install, no server: a single self-contained `index.html` (vanilla JavaScript + HTML Canvas) that runs entirely in your browser. Clone it and double-click it to run offline.

> **Companion project:** this is the interactive, visual counterpart to the MATLAB PSO solver in
> [**Economic-Dispatch-Using-Particle-Swarm-Optimization**](https://github.com/panas-bhattarai/Economic-Dispatch-Using-Particle-Swarm-Optimization).
> It defaults to the same 5-unit, 850 MW test system so you can see the exact optimum that the swarm is chasing.

---

## The problem

Each generator $i$ has a quadratic fuel-cost curve $C_i(P_i) = a_i P_i^2 + b_i P_i + c_i$, and we choose outputs to

$$ \min \sum_{i=1}^{N} C_i(P_i) \quad \text{s.t.} \quad \sum_{i=1}^{N} P_i = P_d, \qquad P_i^{\min} \le P_i \le P_i^{\max}. $$

## The idea it makes visible

At the optimum, every generator that is **not** pinned at a limit runs where its **incremental cost** equals a single system-wide value $\lambda$:

$$ \frac{\mathrm{d}C_i}{\mathrm{d}P_i} = 2 a_i P_i + b_i = \lambda. $$

Units stuck at a limit sit *off* the λ line (at their max, their incremental cost is below λ; at their min, above). The playground draws exactly this: each unit's incremental-cost line, a draggable λ line, and the resulting dispatch — so the equal-incremental-cost principle stops being an equation and becomes something you can *see*.

---

## What you can do

- **Drag the λ line** (or the slider) and watch every unit load to where its incremental-cost line meets λ, clamped to its limits.
- **Auto-solve** — one click runs λ-iteration (bisection on λ until generation meets demand) to land the exact least-cost dispatch.
- **Change the load demand** with a slider and watch the dispatch and λ move.
- **Edit the generators** — cost coefficients $a, b, c$ and limits $P^{\min}, P^{\max}$ — live, and see the solution update.
- **Read the results** — per-unit output bars, total generation vs. demand, power balance, and total cost, plus feasibility warnings when demand exceeds capacity.

### Default case (matches the MATLAB repo)

Five units, $P_d = 850$ MW. The exact optimum is:

| $\lambda$ | dispatch G1…G5 (MW) | total cost |
|---|---|---|
| **10.6 $/MWh** | **[150, 250, 200, 150, 100]** | **7165 $/h** |

Here G2–G5 are maxed out and **G1 is the marginal unit** on the λ line. Drag the demand lower and you'll see units come off their limits.

---

## Two cost models — convex vs. non-convex

A **Cost model** switch chooses what you're solving:

- **Smooth (convex)** — the 5-unit system above. λ-iteration is exact, and the PSO solver just converges to the same answer. Watch a swarm of candidate dispatch vectors fall onto the optimum on a live log-scale convergence plot, with the swarm's best guess overlaid on the dispatch bars.

- **Valve-point (non-convex)** — loads the classic **Walters & Sheble (1993) 3-unit benchmark**, adding a rectified-sine ripple $|e_i\sin(f_i(P_i^{\min}-P_i))|$ to each cost curve. Because the units have nearly-equal incremental costs they all stay *interior* (free to move), so the ripples create many local optima. Now **λ-iteration is suboptimal** — it only sees the smooth trend — and **PSO finds a genuinely cheaper dispatch**, dipping below the λ-iteration baseline on the convergence plot. A ripple-scale slider exaggerates the effect.

  For this benchmark ($P_d = 850$ MW): the λ-iteration dispatch costs ≈ **8482 $/h**, while PSO reaches the true optimum of ≈ **8234 $/h** — beating the classical method by ≈ 3% by exploiting the ripples, exactly the regime where metaheuristics earn their keep.

Both modes use the same PSO core: each particle is a full dispatch vector, updated by the standard velocity rule. Smooth mode handles the power balance with a penalty and random initialization (so its convergence is visible from a high starting cost). Valve mode adds three things that let the swarm reach the global optimum on the rugged rippled landscape: it **warm-starts** one particle at the λ-iteration solution, keeps every particle **feasible by repair** (projecting onto ΣP = Pd so the search runs on the feasible manifold), and applies light **turbulence** (occasionally re-randomizing a unit) to escape ripple local optima.

## Running / hosting it yourself

```bash
git clone https://github.com/panas-bhattarai/economic-dispatch-playground.git
cd economic-dispatch-playground
# then open index.html in any browser
```

It's one static file, so GitHub Pages (Settings → Pages → deploy from `main`, root) serves it with no build step and no backend.

## License

[MIT](LICENSE) © 2026 Panas Bhattarai
