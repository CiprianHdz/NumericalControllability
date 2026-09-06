# Experiments

Unifies `numerical_controllability_research_plan.pdf` (narrative 4-month plan) and
`numerical_controllability_dossier_revised.pdf` (the exact, numbered experiment
protocol — Blocks V/C/S/R/P, acceptance thresholds, Gate 0/1/3) into one catalog,
restricted to **HUM + 1-D heat equation + the linear case**. The dossier is the
more precise document where the two overlap (e.g. its Block C mesh lists are
identical to the research plan's Month-2 refinement lists) and is treated as
authoritative on configuration counts and acceptance thresholds; the research plan
contributes framing and a few parameters the dossier doesn't separately name
(diffusion coefficient, control-region *location*).

**Explicitly excluded, per instruction** (both are in the source documents, neither
is here):
- The primal/least-squares formulation comparison ("Paper 2" in the dossier,
  Month 4 in the research plan — "HUM versus least-squares/variational
  formulations").
- Viscous Burgers ("Paper 3" in the dossier, the Month-4 "Burgers pilot" in the
  research plan) — nonlinear.

**Open scoping note — internal vs. boundary control:** the dossier's own worked
model (§4.1) is written for **internal** control (`1_ω u` forcing) only, and its
Blocks V/C/S/R/P are specified against that model. The research plan's
experiment-tracking schema separately lists "internal/boundary" as a `Control`
dimension to record per run, and §3 notes the source thesis already implements
both. Neither document specifies boundary-control versions of these blocks
explicitly. Read every experiment below as **internal control** unless noted; where
a boundary-control run of the same block would be a natural extension (not an
assertion from the source documents), it's flagged as such.

**Format**, per experiment: **Objective** — **Parameters** — **Expected behavior**
— **Control check** (a sanity criterion for confirming the *code* is right, not
just a result to report) — **Source**.

---

## Foundational tool: manufactured solution / high-resolution reference

Not a standalone experiment — the validation mechanism nearly every experiment
below relies on for its control check.

- **Objective.** Establish a ground truth to measure error against, since the
  discrete HUM control has no simple closed form in general.
- **Two available references:**
  1. *Closed-form uncontrolled solution* — for the sine initial condition
     `y0 = A·sin(mπx)` on `(0, L)`, the uncontrolled (u=0) solution is exactly
     `y_e(x,t) = A·exp(-ν(mπ/L)²t)·sin(mπx/L)` (an eigenfunction of `-∂²/∂x²`).
     Cheap, exact, but only validates the *forward solver*, not the control.
  2. *Manufactured-control experiment* (research plan, "Experiments that become
     important after the baseline") — construct a target trajectory/control pair
     analytically (e.g. pick a smooth `y` with `y(·,0)=y0`, `y(·,T)=0` by
     construction, then define `u := ∂ₜy - ν∂ₓₓy` on `ω`) so the *exact* control is
     known, not just the exact uncontrolled state.
  3. *High-resolution reference* (dossier Block V, "reference-control
     consistency," 3 configs) — nested high-resolution meshes; the reference is
     accepted only once two successive refinements agree to within the dossier's
     own threshold (see Acceptance criteria below).
- **Control check.** The forward solver alone (control off) must reproduce (1) to
  within its own discretization order before anything control-related is trusted —
  if this fails, the bug is in `discretization`/`solvers`, not in HUM.
- **Source.** Dossier §8.2 (Block V); research plan "Experiments that become
  important after the baseline."

---

## Experiment 0 — Baseline / smoke test

- **Objective.** "Establish a clean baseline and identify a reproducible numerical
  phenomenon" before exploring any parameter — deliberately narrow.
- **Parameters.** 1-D heat, internal control, implicit Euler, one fixed mesh, HUM
  exact (`ε=None`).
- **Record.** `κ(Ah)`, `λmin`, `λmax`, `‖uh‖`, `‖yh(T)‖`, CG iterations, CPU time,
  control error against a manufactured or high-resolution reference.
- **Expected behavior.** CG converges within a modest iteration budget; `‖yh(T)‖`
  is small relative to the uncontrolled `‖y(T)‖` (the control is actually doing
  something); the free (`u=0`) response matches the closed-form solution above.
- **Control check.** This *is* the control check for everything downstream — if
  `converged=False`, or `‖yh(T)‖` is not markedly smaller than the uncontrolled
  norm, or the free response doesn't match `y_e`, stop and fix the implementation
  before running anything below.
- **Source.** Research plan, "First experiment: deliberately narrow."

---

## Experiment 1 — Block V: Verification (15 configurations)

- **Objective.** Confirm the forward/adjoint solvers converge at their theoretical
  order, and that a chosen reference is trustworthy — before trusting any error
  numbers from later blocks.
- **Parameters.**
  - Spatial order: 6 levels, `N ∈ {20, 40, 80, 160, 320, 640}` (fine, fixed time
    step).
  - Temporal order: 6 levels, `M ∈ {50, 100, 200, 400, 800, 1600}` (fine, fixed
    space step).
  - Reference-control consistency: 3 nested high-resolution meshes.
- **Expected behavior.** Observed order of accuracy approaches the schemes'
  theoretical order in the asymptotic regime: **2 in space** (centered
  finite-difference Laplacian), **1 in time** (implicit Euler) — or the order of
  whichever `solvers.py` scheme is under test (RK4 should show up to 4th order,
  explicit Euler 1st, if/when those are exercised).
- **Control check** (dossier's own acceptance criteria, §9):
  - *EOC*: accept only once **at least 3 consecutive** computed EOCs are stable and
    their mean is within **~15%** (or 0.2 absolute order) of the theoretical order;
    a non-converging or wildly oscillating EOC sequence signals a discretization or
    solver bug, not a research finding — report it as a failure, don't tune it
    away.
  - *Reference accuracy*: the difference between the two finest nested references
    must be **≤ 0.1×** the candidate solution's own error, else refine the
    reference further before using it downstream.
- **Source.** Dossier §8.2 Block V; §9 acceptance table (Forward/adjoint EOC,
  Reference-control accuracy rows).

---

## Experiment 2 — Block C: Conditioning and refinement (24 configurations)

- **Objective.** Determine whether clean, reproducible scaling laws emerge for
  conditioning and error under mesh refinement — this is the dossier's headline
  open question (claim C3), *not* a known-in-advance fact to assert.
- **Parameters**, 4 sweeps × 6 mesh levels each:
  - Spatial sweep (space refined, time fixed fine) — same `N` list as Block V.
  - Temporal sweep (time refined, space fixed fine) — same `M` list as Block V.
  - Coupled `k = C·h` (β=1).
  - Coupled `k = C·h²` (β=2).
  (Research plan's Month-2 also names `β ∈ {0.5, 1, 1.5, 2}` — a superset; the
  dossier's 24-config count only fixes two of those four, β=1 and β=2.)
- **Record.** `λmin`, `λmax`, `κ`, `Eu` (control error), `rT` (terminal residual),
  `‖u‖`, Krylov iterations, work (CPU/memory).
- **Expected behavior.**
  - The *discrete Laplacian itself* (`Ah`, independent of HUM) has a well-known
    scaling — `κ(Ah) = O(1/h²) = O(N²)` for the standard centered 1-D stencil —
    which is a plain linear-algebra fact, not a research question, and is a good
    quick check of `discretization.py` alone (see control check).
  - Whether the *HUM/penalized-HUM Gramian's* conditioning follows a comparably
    clean power law under refinement (independent and coupled) is the actual open
    question this block investigates — do not assume a specific exponent going in.
  - Coupled `k=Ch²` should balance the O(h²) spatial and O(h) temporal errors
    better than `k=Ch`, since implicit Euler is only first-order in time; expect
    `k=Ch²` to reach a given accuracy on a coarser space grid.
- **Control check.**
  - Sanity-check `κ(Ah)` alone (not the Gramian) against the known `O(N²)` law
    first — an implementation bug in the Laplacian assembly (wrong sign, wrong
    boundary handling) would show up here immediately, before it contaminates any
    HUM-specific number.
  - `λmin`/`λmax`/`κ`/`Eu` should vary **smoothly and monotonically** with mesh
    refinement — jumps, sign flips, or non-monotonic behavior across adjacent mesh
    levels indicate a bug (e.g. an indexing error in the control operator, or a
    stale cached factorization) rather than genuine numerical phenomenon.
  - Meaningful only once Gate 0 (below) passes.
- **Gate 0 (dossier §8.1, must pass before the *full* Block C-P budget is spent):**
  run Block V in full plus a reduced Block C (3 of the 6 `k=Ch` mesh levels) as a
  pilot. Do not proceed to the full 123-configuration core unless this pilot shows
  a **stable asymptotic window of at least 3 consecutive mesh levels** — otherwise
  widen the mesh range or revisit the discretization first.
- **Source.** Dossier §8.1 (Gate 0), §8.2 Block C; research plan Month 2.

---

## Experiment 3 — Block S: Solver tolerance and oversolving (30 configurations)

- **Objective.** Determine when CG's optimization error becomes negligible
  compared with discretization error, to avoid wasted iterations ("oversolving").
- **Parameters.** 6 production meshes × 5 tolerances,
  `τ ∈ {10⁻⁴, 10⁻⁶, 10⁻⁸, 10⁻¹⁰, 10⁻¹²}`.
- **Record/plot.** `Eu(τ)` (control error), `Ealg(τ)` (algebraic error), `Nit(τ)`
  (iteration count) — identify the plateau where tightening `τ` further stops
  improving `Eu`.
- **Expected behavior.** `Nit` grows as `τ` tightens (expected, CG needs more
  iterations for a tighter residual); `Eu` should **decrease then plateau** once
  algebraic error drops below discretization error — it should never *increase* as
  `τ` tightens, and should never keep decreasing all the way to `τ=10⁻¹²` (if it
  does, discretization error is being masked, likely by a bug or too coarse a
  reference).
- **Control check.** Dossier's own rule: `Ealg ≤ 0.1·Edisc` once an error estimator
  exists; otherwise the plateau rule — the largest `τ` giving at most **5%**
  degradation vs. the tightest solve. A curve that never plateaus, or plateaus
  immediately at the loosest `τ`, is a red flag for the CG implementation
  (`linear_cg`), not a research finding.
- **Source.** Dossier §8.2 Block S; §9 acceptance table (Algebraic stopping row);
  research plan Month 3 ("sweep optimization tolerances").

---

## Experiment 4 — Block R: Regularization path (30 configurations)

- **Objective.** Characterize the bias-vs-conditioning tradeoff of the penalization
  parameter `ε`.
- **Parameters.** 6 meshes × 5 values, `ε ∈ {0, h², h⁴, h⁶, h⁸}` (the dossier's
  grid; the research plan's Month-3 `ε=h^p, p=0,…,8` is a finer superset of the
  same idea — worth running the fuller `p` grid if budget allows).
- **Pre-registered hypothesis** (dossier, stated *before* running the block, so it
  tests a hypothesis rather than only fitting a curve after the fact): bias should
  fall roughly like `ε` while `κ` should grow roughly like `ε⁻¹` near the small-`ε`
  end.
- **Expected behavior.** As `ε→0`: terminal residual/bias shrinks, condition number
  grows — the two ends of the Pareto tradeoff in `Jε(u) = ½‖u‖² + (1/2ε)‖y(T)‖²`
  (dossier §4.2). `ε=0` (exact HUM) should show the *worst* conditioning and the
  *smallest* bias of the sweep; the largest `ε` the mildest conditioning and
  largest bias.
- **Control check.** The *direction* of both trends (bias down, `κ` up as `ε→0`)
  must hold even if the exact exponents don't match the pre-registered guess — a
  regularization sweep where `κ` does *not* respond to `ε` at all, or responds in
  the wrong direction, means `ε` isn't actually wired into the CG operator
  correctly (check the `eps*v + apply_operator(v)` term). State explicitly whether
  the observed slope matches the pre-registered hypothesis or not; don't silently
  revise the hypothesis to fit the data.
- **Source.** Dossier §8.2 Block R, §4.2; research plan Month 3.

---

## Experiment 5 — Block P: Preconditioning (24 configurations)

- **Objective.** Test whether preconditioning gives mesh-independent (or provably
  improved) CG iteration counts.
- **Parameters.** 4 choices × 6 meshes: none (baseline), diagonal/Jacobi,
  incomplete factorization, one operator/block-structured candidate — plus a fifth
  comparison column against at least one published 2024–2026 preconditioner
  (dossier's Priority B appendix) as an **external** baseline, not only the
  internal "none" one.
- **Expected behavior.** Preconditioned iteration counts should be **flat or only
  mildly growing** with mesh refinement, versus the unpreconditioned baseline's
  iteration count, which is expected to grow with `κ(Ah,k)` per the standard CG
  bound (dossier §4.1: `‖eₘ‖/‖e₀‖ ≤ 2·((√κ-1)/(√κ+1))^m`).
- **Control check.** "Mesh-independent" is defined operationally (dossier §9):
  iteration-count variation **≤ 20%** across the production mesh sequence, or a
  fitted growth exponent statistically indistinguishable from zero. At minimum,
  every preconditioned column should need **no more** iterations than the
  unpreconditioned baseline at every mesh level — if a "preconditioner" increases
  iteration count, that's a sign error or an incorrectly applied operator, not a
  legitimate negative result.
- **Source.** Dossier §8.2 Block P, §9 (Preconditioner robustness row), §4.1;
  research plan Month 3 / "Experiments that become important."

---

## Experiment 6 — Robustness slices (42 configurations)

Run only after Gates 0 and 1 pass (below). Each slice states in advance what a
*failure* would mean for the headline scaling claim (C3) — so these can falsify
it, not just decorate it.

- **6a. Control horizon `T`** — `T ∈ {1, 0.5, 0.25, 0.125}` on 3 meshes (12
  configs). *Pre-registered failure meaning*: if conditioning/control cost depends
  materially on `T`, the scaling law must be restated as `T`-dependent. *Expected*:
  smaller `T` (less time to act) should generally cost more control effort
  (`‖u‖` up) — the research plan separately flags a small-`T` regime (`T→0`) as
  worth extra attention since it stresses the numerics hardest.
- **6b. Control-region size** — `|ω| ∈ {0.8, 0.5, 0.25, 0.1}` on 3 meshes (12
  configs). *Pre-registered failure meaning*: if `κ` depends materially on `|ω|`,
  restate C3 as `ω`-dependent, not universal. *Expected*: a **smaller** actuation
  region should generally require **more** control effort / worse conditioning —
  a standard controllability fact; the opposite trend suggests a bug in
  `control_operator_internal`'s indicator assembly.
- **6c. Initial-condition frequency** — `m ∈ {1,2,4,8,16,32}` for
  `y0 = sin(mπx)` on 3 meshes (18 configs). *Expected*: higher-frequency modes
  decay faster under diffusion (rate `∝ m²`) but may be harder to resolve/control
  numerically as `m` approaches the mesh's Nyquist limit — watch for the
  discretization simply failing to represent high `m` on coarse meshes, which is
  an expected resolution limit, not a controllability finding.
- **6d. Control-region *location*** (research plan; not separately counted in the
  dossier's 42) — e.g. `ω` centered vs. off-center vs. touching a boundary.
  *Expected*: by the heat kernel's symmetry, a centered `ω` on a symmetric IC
  should behave differently from an off-center one; this is a good place to catch
  an accidental hardcoded assumption about `ω`'s position.
- **6e. Diffusion coefficient `ν`** (research plan; not in the dossier's named
  blocks) — vary `ν` across a moderate range (the *vanishing-viscosity* extreme is
  explicitly a Burgers/nonlinear concern and is out of scope here). *Expected*:
  larger `ν` diffuses/mixes faster, generally easing control; `κ(Ah)` scales
  linearly with `ν` directly from its assembly (`Ah = ν·(discrete d²/dx²)`) — a
  trivial, exactly-checkable sanity fact, independent of any HUM-specific result.
- **Source.** Dossier §8.3; research plan Month 3 ("Vary T, diffusion/viscosity,
  control-region size/location, and initial-condition frequency") and "Experiments
  that become important after the baseline."

---

## Cross-cutting acceptance criteria (dossier §9, "now provisional")

Reusable as the control check for multiple experiments above. All are explicitly
labeled *provisional* in the dossier pending Gate 0 (Experiment 2's pilot) —
treat none as fixed until the pilot shows it's attainable at the planned
mesh/precision budget:

| Quantity | Criterion |
|---|---|
| Forward/adjoint EOC | ≥3 consecutive stable EOCs, mean within ~15% (or 0.2 absolute order) of theory |
| Reference-control accuracy | Two finest references agree to ≤ 0.1× candidate error |
| Algebraic stopping | `Ealg ≤ 0.1·Edisc`, or plateau rule (≤5% degradation vs. tightest solve) |
| Terminal null residual | Report `rT/‖y0‖` at 10⁻⁴, 10⁻⁶, 10⁻⁸ bands (no universal target) |
| Condition-number power law | ≥4 asymptotic points; fitted slope stable (<10% change between nested fit windows) |
| Preconditioner robustness | Iteration-count variation ≤20% across meshes, or fitted exponent ≈0 |
| Estimator effectivity | Reliable if in [0.5, 2]; sharp if in [0.8, 1.25] asymptotically |
| Roundoff onset | Flag first level where error fails to drop ≥20% despite refinement + tighter tolerance |
| Timing | 5 repeats after warm-up, report median/IQR, fixed hardware/software state |

**Gate 1** (dossier §13, applies once Experiments 3/4 are underway): a tolerance
rule is credible only if it predicts the observed plateau across several mesh
levels *without being retuned per grid*.

---

## Excluded (for reference — not part of this catalog)

- **Paper 2 — HUM vs. primal/least-squares** (dossier §11; research plan Month 4):
  2 formulations × 6 mesh levels × 3 accuracy targets = 36 matched configurations.
  Excluded because it requires a second, non-HUM formulation.
- **Paper 3 — Burgers pilot** (dossier §12; research plan Month 4): 24
  configurations, `A ∈ {0.01,0.1,0.5,1}`, `ν ∈ {10⁻¹,10⁻²,10⁻³}`, `N ∈ {80,160}`.
  Excluded — nonlinear, and both documents explicitly gate it on the linear study
  being stable first.
