# Architecture

Fixed reference for the HUM baseline codebase. Any structural change to the tree or
the interface contracts below is a diff to this file first, code second. **Nothing
in this document has been scaffolded yet** — it is a proposal to be finalized here,
then executed as a separate, explicit step.

This document (and `ALGORITHMS.md`) live in `Proposal/` alongside the research plan
and dossier, not inside `HUM Code/` itself — they are project/planning documents
describing the code, not part of the installable package.

Sole algorithmic reference: `SourceThesis/TesisCIMATVersionFirmada.pdf`, Ch. 4 +
Appendix B (Algorithms 1–5, pp. 81–89). See `ALGORITHMS.md` for the full variant
inventory. `SourceDocuments/Glowinski_and_numerical_control_problems.pdf` is cited as
background literature only — it does not cover boundary control or CG-based HUM and
is not implemented here.

**Revision note:** this replaces an earlier, heavier design (a JSON-config-driven
`src/hum/` axis-split with `HeatProblem`/`InitialCondition`/`RunResult` dataclasses
and an `examples/configs/*.json` pair) with the leaner design below, per an explicit
decision to start fresh with something simplistic and scalable for running
experiments — not a maintained software product. See "Deferred" at the end for what
that dropped, and why it isn't lost.

## Design principles

- This is a single-researcher thesis/PhD codebase. Its job right now is executing
  experiments and reading off numbers, not serving external users or surviving
  arbitrary future requirements — so JSON config schemas, run-traceability
  databases, and protocol/dataclass layers are premature until the baseline itself
  is trusted.
- It still keeps **one** shared CG engine and **one** norms module across every HUM
  variant. This is not optional simplicity — it is the direct defense against
  `RawCode/`'s actual, confirmed failure mode (`Proposal/ALGORITHMS.md`'s "Excluded"
  section: copy-pasted CG loops, one file dividing by `ε` where every other
  multiplies, a wrong norm space). Collapsing to fully inline, per-script algorithm
  code would reproduce that failure mode by construction.
- Scoped deliberately narrow right now — `research_plan.pdf`'s own "first
  experiment" (quoted below), not the dossier's full ~123-configuration protocol.
  Everything wider is listed under "Deferred," not silently dropped.

## Directory tree

```
NumericalControllability/
├── RawCode/                        (untouched, historical reference)
├── Proposal/, SourceThesis/, SourceDocuments/   (untouched)
├── .gitignore                      (repo root)
└── HUM Code/
    ├── hum/
    │   ├── __init__.py
    │   ├── discretization.py       # grid, Laplacian Ah, control operator Bh (internal/boundary)
    │   ├── solvers.py              # explicit/implicit Euler + RK4, forward & backward solves
    │   ├── norms.py                # L2, H10 (no H^-1 -- confirmed unused by any of the 5 algorithms)
    │   ├── algorithms.py           # one shared CG loop + apply_gramian (Alg 2) + hum_internal (Alg 3);
    │   │                           #   hum1_boundary/hum2_boundary designed, deferred (see below)
    │   └── diagnostics.py          # spectrum, closed-form exact solution, the standard plot set
    ├── run_experiment.py           # single-experiment runner: plain params, # %% cells
    ├── run_all_experiments.py      # batch runner: the concrete experiment list, appends to runs_summary.csv
    └── runs_summary.csv            # accumulated per-run summaries (git-tracked)
```

No `pyproject.toml`, no packaging, no `tests/`, no `README.md`/`WORKFLOW.md`, no
`examples/configs/`. `hum/` is imported directly (both scripts live next to it), no
`pip install` step.

## Module responsibilities (one line each)

- `hum/discretization.py` — `build_grid`, the discrete Laplacian `Ah`, and `Bh` for
  internal (indicator over `ω`) or boundary (one-sided) control.
- `hum/solvers.py` — `forward_solve`/`backward_solve`, scheme selected by name
  (`"implicit_euler"` default, `"explicit_euler"`, `"rk4"`).
- `hum/norms.py` — `l2_norm`/`l2_inner` (Algorithm 3), `h10_norm`/`h10_inner`
  (Algorithm 4, for when boundary control is implemented).
- `hum/algorithms.py` — `linear_cg` (the one generic CG engine), `apply_gramian`
  (Algorithm 2, generalized — shared by internal and, later, HUM1 boundary control),
  `hum_internal` (Algorithm 3, implemented), `hum1_boundary`/`hum2_boundary`
  (Algorithms 4/5, designed but raise `NotImplementedError` until a boundary
  experiment is scoped).
- `hum/diagnostics.py` — `spectrum`/`condition_number`, `exact_solution_sine`, and
  the plot set: state snapshots, control profile, CG convergence curve.
- `run_experiment.py` — `run(...)`: builds the discretization, calls `hum_internal`,
  computes every summary metric, optionally plots, returns the summary dict.
- `run_all_experiments.py` — imports `run` and calls it once per experiment in the
  list, appending each summary to `runs_summary.csv`.

## Fixed interface contracts

### `hum/discretization.py`

```python
def build_grid(interval: tuple[float, float], n_space: int, n_time: int, T: float) -> tuple[np.ndarray, np.ndarray, float, float]:
    """Returns (x, t, h, tau). x has n_space+1 points including both boundaries."""

def laplacian(n_interior: int, alpha: float, h: float) -> np.ndarray:
    """Ah = alpha * (discrete d^2/dx^2) on the interior nodes. Already has the
    correct sign for y' = Ah y + Bu (RawCode's own comment: "w/o the minus") --
    symmetric negative semi-definite."""

def control_operator_internal(x_interior: np.ndarray, omega: tuple[float, float]) -> np.ndarray:
    """Diagonal indicator matrix on omega, shape (n_interior, n_interior)."""

def control_operator_boundary(n_interior: int, alpha: float, h: float, side: str = "right") -> np.ndarray:
    """Single control channel at one boundary node, shape (n_interior, 1) -- kept
    2-D like the internal case so apply_gramian treats both uniformly:
    u = B^T . phi, forcing = B . u. "both" (two-sided) is a future extension."""
```

### `hum/solvers.py`

```python
def forward_solve(y0: np.ndarray, A: np.ndarray, B: np.ndarray, u: np.ndarray, tau: float, scheme: str = "implicit_euler") -> np.ndarray:
    """y' = A y + B u(t), y(0) = y0. Returns y, shape (n_time, len(y0))."""

def backward_solve(fT: np.ndarray, A: np.ndarray, tau: float, n_time: int, scheme: str = "implicit_euler") -> np.ndarray:
    """The adjoint -phi' = A^T phi, phi(T) = fT, backward in time."""
```

**Sign-convention note (verified against RawCode, easy to get wrong):**
`backward_solve` steps with `+A` directly, not `A.T` and not `-A`. `Ah` is always
symmetric here, and the time reversal `s = T - t` turns the backward adjoint problem
into a *forward* problem in `s` with the *same* operator `A`
(`d(phi)/ds = A phi`) — this is exactly what every canonical RawCode file's
`BackwardEuler_b` does (e.g. `HUM_boundary_modified.py`), and it must be reproduced
exactly, not "corrected" to `A.T`.

### `hum/algorithms.py`

```python
def linear_cg(
    apply_operator: Callable[[np.ndarray], np.ndarray],
    rhs: np.ndarray,
    x0: np.ndarray,          # CG initial iterate -- an optimization variable, NOT a PDE IC
    eps: float | None,       # None omits the eps term entirely (structurally exact)
    norm_fn: Callable[[np.ndarray], float],
    inner_fn: Callable[[np.ndarray, np.ndarray], float],
    tol: float,
    max_iter: int,
) -> tuple[np.ndarray, CGResult]:
    """Generic linear CG for (eps*I + apply_operator)(x) = rhs."""

def apply_gramian(f0: np.ndarray, ic: np.ndarray, A: np.ndarray, B: np.ndarray, tau: float, t: np.ndarray, scheme: str = "implicit_euler") -> tuple[np.ndarray, np.ndarray, np.ndarray]:
    """Algorithm 2 (thesis p.86), generalized: backward adjoint solve from f0 ->
    control u = B^T . phi -> forward primal solve from `ic`. Returns (y, phi, u).
    Control-type-agnostic via B. Shared by hum_internal (ic = the real PDE IC on
    the first call, zero on every CG-loop call) and, later, hum1_boundary.
    hum2_boundary does NOT use this -- its order is forward-then-backward."""

def hum_internal(ic, A, B, h, tau, t, eps=None, tol=1e-6, max_iter=200, scheme="implicit_euler", target=None) -> dict:
    """Algorithm 3 (thesis p.87), exact (eps=None) or penalized. Returns
    {f, phi, u, y, n_iter, converged, residual_history}."""
```

**Design note on `hum_internal`:** rather than literally recomputing
`g0 = eps*f0 + y(T) - y1` via `Gramian(f0, ic)` at every CG step (the thesis's
literal phrasing), it folds `ic`'s free (uncontrolled) response into a fixed `rhs`
once via `apply_gramian(0, ic, ...)`, then reuses the *same* generic `linear_cg`
against a zero-IC operator. These are exactly equivalent by linearity of the primal
PDE whenever the CG initial guess is `f0=0` (always the case, matching every
canonical RawCode file's own `f0 = np.zeros(...)`) — this is what lets
`hum1_boundary` reuse the identical `linear_cg` later without a second CG
implementation, instead of copy-pasting the loop per variant (the exact pattern
`ALGORITHMS.md` flags as `RawCode`'s failure mode).

`hum1_boundary`/`hum2_boundary` (Algorithms 4/5) are declared with their intended
signatures but raise `NotImplementedError` — implement when a boundary experiment
is scoped; `hum2_boundary` must carry the confirmed zero-IC fix from
`Proposal/ALGORITHMS.md` relative to `RawCode/hum_frontera_2.py`.

### `hum/diagnostics.py`

```python
def spectrum(A: np.ndarray) -> tuple[float, float]: ...          # (lambda_min, lambda_max)
def condition_number(A: np.ndarray) -> float: ...                # kappa(Ah)
def exact_solution_sine(x, t, alpha, interval, amplitude=10.0, mode=1) -> np.ndarray: ...
def plot_state_snapshots(x, t, y_bc, title=...): ...             # 5 snapshots, y including boundary values
def plot_control(t, u, title=...): ...                           # u(t) (1 channel) or ||u(t)|| (many channels)
def plot_cg_convergence(residual_history, title=...): ...        # semilog ||g_n||/||g_0|| vs iteration
def summarize(problem_summary: dict) -> str: ...
```

`exact_solution_sine` generalizes the thesis's `10*exp(-alpha*pi^2*t)*sin(pi*x)`
(amplitude=10, mode=1, domain `(0,1)`) to any domain/amplitude/mode — it stays a
genuine Dirichlet eigenfunction of `-d^2/dx^2` for every choice.

### `run_experiment.py`

```python
def run(
    alpha: float = 1.0,
    interval: tuple[float, float] = (0.0, 1.0),
    T: float = 1.0,
    n_space: int = 80,
    n_time: int = 400,
    omega: tuple[float, float] = (0.3, 0.8),
    eps: float | None = None,
    tol: float = 1e-6,
    max_iter: int = 200,
    amplitude: float = 10.0,
    mode: int = 1,
    scheme: str = "implicit_euler",
    make_plots: bool = True,
) -> dict:
    """Builds the discretization, runs hum_internal, computes every summary metric
    (see "Experiments" below), optionally plots, returns the summary dict."""
```

Every parameter is a plain Python argument with a sensible thesis-matching default —
no config file, no schema class. Editing an experiment means calling `run(...)` with
different keyword arguments (from `run_experiment.py`'s own `# %%` cell, or from
`run_all_experiments.py`).

## Experiments

**Format:** plain `.py` files using `# %%` cell markers (VS Code/Spyder/PyCharm
render these as notebook-style cells), not `.ipynb` notebooks. Chosen over notebooks
because: (1) `RawCode/` is itself mostly near-duplicate `.ipynb` files, and notebook
sprawl is exactly what this whole consolidation effort exists to fix; (2) plain
scripts are git-diffable while notebook JSON (execution counts, embedded output
images) is not; (3) `run_all_experiments.py` needs to execute headlessly, which a
real notebook would need `papermill`/`nbconvert --execute` for.

**Scope right now — `research_plan.pdf`'s "first experiment" only:**

> Start with 1-D heat + HUM + implicit Euler. The first objective is not to explore
> every parameter, but to establish a clean baseline and identify a reproducible
> numerical phenomenon. Record: κ(Ah), λmin, λmax, ‖uh‖, ‖yh(T)‖, CG iterations, CPU
> time, and control error against a manufactured or high-resolution reference.

Internal control (Algorithm 3), exact (`eps=None`), implicit Euler, the thesis's own
`10·sin(πx)` initial condition on `(0,1)`. "Control error against a … reference" is
left as a deliberate follow-up in `run_all_experiments.py` (needs a second, finer
comparison run) rather than guessed at here.

**Visualization**, per the comparison already made and agreed on:

- *Per experiment* (`run_experiment.py`, always produced): state snapshots
  (`y(x,·)` at 5 times — does the control flatten the solution by `T`?), the control
  profile, the CG convergence curve (semilog residual vs. iteration), and a printed
  summary line.
- *Cross experiment* (`run_all_experiments.py`): every run's summary appended as a
  row to `runs_summary.csv` — the minimal seed of `research_plan.pdf`'s
  "experiment-tracking layer," without its database/frontend yet.
- *Deferred*: log-log scaling plots (error/iterations/`κ` vs. mesh size or `ε`) —
  no sweep exists yet to plot; a shareable dashboard — worth it once there are dozens
  of runs to browse, not one.

## Deferred (explicit, not silently dropped)

- **JSON config schema / `HeatProblem`/`InitialCondition` dataclasses** — replaced by
  plain Python keyword arguments to `run()` for now. Revisit if/when config-file-
  driven runs are actually needed (e.g. for Month 2+'s larger sweeps).
- **`RunResult`/`runs/` git-tracked JSON-per-run traceability** — replaced by
  `runs_summary.csv` for now. `research_plan.pdf`'s own Month-1 goal ("an
  experiment-tracking layer… store parameters, numerical outputs, solver statistics,
  operators/matrices, controls, states, plots, code commit, machine metadata… ~100
  experiments… a lightweight frontend") is bigger than this and should be built once
  the baseline is proven, not before.
- **`hum1_boundary`/`hum2_boundary`** (Algorithms 4 & 5) — designed (signatures,
  shared `apply_gramian`) but not implemented; add when a boundary experiment is
  scoped.
- **Everything past the first experiment**: independent/coupled mesh refinement,
  regularization (`ε=h^p`) and tolerance sweeps, preconditioning, the least-squares
  formulation comparison, the Burgers pilot, the dossier's ~123-configuration
  protocol, EOC tables, condition-number scaling laws — all real goals in the
  proposal documents, all explicitly out of scope until this baseline runs and is
  trusted.
- **Packaging/process**: `pyproject.toml`, `tests/`, `README.md`/`WORKFLOW.md` — none
  of these exist yet either; add when there's more than one contributor or more than
  a handful of files to navigate.
