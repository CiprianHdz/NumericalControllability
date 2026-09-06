# Architecture

Fixed reference for the HUM baseline codebase. Any structural change to the tree or
the interface contracts below is a diff to this file first, code second.

This document (and `ALGORITHMS.md`) live in `Proposal/` alongside the research plan
and dossier, not inside `HUM Code/` itself — they are project/planning documents
describing the code, not part of the installable package.

Sole algorithmic reference: `SourceThesis/TesisCIMATVersionFirmada.pdf`, Ch. 4 +
Appendix B (Algorithms 1–5, pp. 81–89). See `ALGORITHMS.md` for the full variant
inventory. `SourceDocuments/Glowinski_and_numerical_control_problems.pdf` is cited as
background literature only — it does not cover boundary control or CG-based HUM and
is not implemented here.

## Directory tree

```
NumericalControllability/
├── RawCode/                        (untouched, historical reference)
├── Proposal/, SourceThesis/, SourceDocuments/   (untouched)
├── .gitignore                      (repo root)
├── Proposal/
│   ├── numerical_controllability_research_plan.pdf
│   ├── numerical_controllability_dossier_revised.pdf
│   ├── ARCHITECTURE.md         # this file
│   └── ALGORITHMS.md           # algorithm inventory, manual sign-off checkpoint
└── HUM Code/
    ├── pyproject.toml
    ├── README.md                   # traceability doc: thesis alg/eq ↔ module map;
    │                               #   also documents the 3 excluded broken/abandoned files
    ├── WORKFLOW.md                 # the manual: how to run, branch, track results
    ├── main.py                     # THE MAIN APP — single entry point, config-driven
    ├── src/hum/
    │   ├── __init__.py
    │   ├── discretization/                    # AXIS 1: space grid + control operators
    │   │   ├── grid.py                        # 1D uniform grid (space + time)
    │   │   ├── operators.py                   # Laplacian Ah, dense now / sparse seam later
    │   │   └── control_operators.py           # Bh: internal (indicator on ω) / boundary (1- or 2-sided)
    │   ├── solvers/                            # AXIS 2: ODE/time integration
    │   │   ├── base.py                        # TimeScheme protocol
    │   │   ├── explicit_euler.py
    │   │   ├── implicit_euler.py              # DEFAULT
    │   │   ├── rk4.py
    │   │   ├── factory.py                     # get_scheme(name) — config-driven selection
    │   │   └── pde.py                         # forward_solve / adjoint_solve, scheme-agnostic
    │   ├── optimization/                       # AXIS 3: CG engine + norms + HUM variants
    │   │   ├── norms.py                       # L2Norm, H10Norm, HInvNorm (cached factorization)
    │   │   ├── gramian.py                     # Algorithm 2, generalized: shared by
    │   │   │                                  #   hum_internal.py AND hum1_boundary.py
    │   │   ├── linear_cg.py                   # ONE generic CG engine, preconditioner seam
    │   │   ├── result.py                      # CGResult: n_iter, converged, residual_history
    │   │   ├── hum_internal.py                # Algorithm 3 — exact + penalized (eps=None|float)
    │   │   ├── hum1_boundary.py               # Algorithm 4 — exact + penalized (eps=None|float)
    │   │   └── hum2_boundary.py               # Algorithm 5 — penalized only; bug-fixed here
    │   ├── problem.py                          # HeatProblem config dataclass (main.py's schema)
    │   ├── diagnostics/
    │   │   ├── functionals.py                 # J()/J_eps() dual, F_primal()/F_primal_eps() primal
    │   │   ├── duality_check.py                # Fε(v̂) ≈ -Jε(ĝ) sanity check (Eq. 2.27)
    │   │   └── exact_solution.py               # y_e(x,t)=10e^{-απ²t}sin(πx) comparison
    │   └── results.py                          # RunResult: git_commit, timestamp, config snapshot
    ├── runs/                                    # git-tracked JSON run logs written by main.py
    │   └── .gitkeep
    ├── tests/                                   # pytest, mirrors src/hum/
    └── examples/
        └── configs/                             # example JSON configs consumed by main.py
            ├── thesis_eq427_internal_exact.json
            ├── thesis_eq427_internal_penalized.json
            ├── thesis_eq427_hum1_exact.json
            ├── thesis_eq427_hum1_penalized.json
            ├── thesis_eq427_hum2_penalized.json
            └── scheme_robustness_demo.json
```

## Module responsibilities (one line each)

- `main.py` — single, config-file-driven entry point. Reads a JSON config into a
  `HeatProblem`, dispatches to the right algorithm + scheme, runs it, writes a
  git-commit-tagged `RunResult` to `runs/`.
- `discretization/grid.py` — builds the 1D uniform space grid and the time grid.
- `discretization/operators.py` — assembles the discrete Laplacian `Ah`.
- `discretization/control_operators.py` — assembles the control operator `Bh`, for
  internal (indicator over ω) or boundary (one- or two-sided) control.
- `solvers/base.py` — `TimeScheme` protocol shared by forward (primal) and backward
  (adjoint) solves.
- `solvers/explicit_euler.py`, `solvers/implicit_euler.py`, `solvers/rk4.py` —
  concrete `TimeScheme` implementations.
- `solvers/factory.py` — `get_scheme(name)`, the one place a scheme name is mapped to
  an implementation.
- `solvers/pde.py` — `forward_solve`/`adjoint_solve`; loops over the time grid,
  never references a concrete scheme by name.
- `optimization/norms.py` — `Norm` protocol + `L2Norm`, `H10Norm`, `HInvNorm`.
- `optimization/gramian.py` — Algorithm 2 (thesis p.86), **generalized**: apply the
  Gramian once (adjoint backward solve with datum → control via `Bh` → primal forward
  solve), control-type-agnostic — `Bh` (from `discretization/control_operators.py`)
  already carries whatever distinguishes internal from boundary control (operator
  identity, any scaling). This is the same three-step recipe Algorithm 3's Step 0 and
  per-iteration step (eq. B.6–B.9) use inline without naming it — so it's shared by
  both `hum_internal.py` (Algorithm 3) and `hum1_boundary.py` (Algorithm 4).
  **Not** used by `hum2_boundary.py` (Algorithm 5): its solve order is forward-first
  then backward (eq. B.12–B.15), structurally different from the Gramian recipe.
- `optimization/linear_cg.py` — the one generic CG kernel shared by every HUM variant.
- `optimization/result.py` — `CGResult` (iteration count, convergence flag, residual
  history).
- `optimization/hum_internal.py` — Algorithm 3 (thesis p.87), exact and penalized
  paths; builds its `apply_operator`/`rhs` for `linear_cg` on top of `gramian.py`.
- `optimization/hum1_boundary.py` — Algorithm 4 (thesis p.88), exact and penalized
  paths; also builds on `gramian.py`.
- `optimization/hum2_boundary.py` — Algorithm 5 (thesis p.89), penalized only, with
  the confirmed zero-IC bug fixed relative to `RawCode/hum_frontera_2.py`.
- `problem.py` — `HeatProblem`, the config schema.
- `diagnostics/functionals.py` — dual/primal functional evaluation.
- `diagnostics/duality_check.py` — Fenchel–Rockafellar sanity check (Eq. 2.27).
- `diagnostics/exact_solution.py` — the closed-form uncontrolled solution used for
  validation.
- `results.py` — `RunResult`, the traceability record.

## Fixed interface contracts

### `TimeScheme` (`solvers/base.py`)

```python
class TimeScheme(Protocol):
    name: ClassVar[str]

    def step_forward(self, y: np.ndarray, A: np.ndarray, Bv: np.ndarray, tau: float) -> np.ndarray:
        """One step of y' = A y + Bv (primal, forward-in-time)."""

    def step_backward(self, phi: np.ndarray, A: np.ndarray, tau: float) -> np.ndarray:
        """One step of -phi' = A^T phi, integrated backward-in-time (adjoint)."""

    def stability_note(self, A: np.ndarray, tau: float) -> str | None:
        """Advisory CFL-type warning, or None. Never raises."""
```

Concrete: `ExplicitEuler`, `ImplicitEuler` (default; canonical
`np.linalg.solve(I - tau*A, ...)` form), `RK4` (Butcher tableau, thesis Appendix
B.1.2).

### `Norm` (`optimization/norms.py`)

```python
class Norm(Protocol):
    def norm(self, x: np.ndarray) -> float: ...
    def inner(self, x: np.ndarray, y: np.ndarray) -> float: ...
```

Concrete: `L2Norm(h)`, `H10Norm(A, h)` (canonical quadratic form
`sqrt(h·xᵀAx)`), `HInvNorm(A, h)` (factorizes `A` once at construction, reused across
every CG iteration).

### `apply_gramian` (`optimization/gramian.py`)

```python
def apply_gramian(
    f0: np.ndarray,
    x0: np.ndarray,
    A: np.ndarray,
    Bh: np.ndarray,
    scheme: TimeScheme,
    tau: float,
    t: np.ndarray,
) -> tuple[np.ndarray, np.ndarray, np.ndarray]:
    """Algorithm 2 (thesis p.86), generalized: solve the adjoint backward from final
    datum f0, derive the control from phi via Bh, then solve the primal forward from
    x0 with that control. Returns (y, phi, u). Control-type-agnostic: Bh already
    encodes whatever distinguishes internal (indicator over omega) from boundary
    (one- or two-sided trace) control, per discretization/control_operators.py.
    Shared by hum_internal.py (Algorithm 3, x0 = the real IC on the first call, then
    zero on every CG-loop call) and hum1_boundary.py (Algorithm 4, same pattern).
    hum2_boundary.py (Algorithm 5) does NOT call this — its solve order is
    forward-then-backward (eq. B.12-B.15), not this backward-then-forward recipe."""
```

### `linear_cg` (`optimization/linear_cg.py`)

```python
def linear_cg(
    apply_operator: Callable[[np.ndarray], np.ndarray],
    rhs: np.ndarray,
    x0: np.ndarray,
    eps: float | None,
    norm: Norm,
    tol: float,
    max_iter: int,
    preconditioner: Callable[[np.ndarray], np.ndarray] | None = None,
) -> CGResult:
    """Standard linear CG for (eps*I + apply_operator)(x) = rhs in the inner product
    defined by `norm`. `eps=None` omits the eps term entirely (exact variant).
    `preconditioner` is an explicit, currently-unused seam for future PCG work."""
```

One shared numerical kernel; `hum_internal`/`hum1_boundary`/`hum2_boundary` each
supply their own `apply_operator`/`rhs` construction — never a copy-pasted CG loop.

### `HeatProblem` (`problem.py`)

```python
@dataclass(frozen=True)
class HeatProblem:
    alpha: float
    interval: tuple[float, float]
    T: float
    n_space: int
    n_time: int
    control_type: Literal["internal", "boundary"]
    control_region: tuple[float, float] | Literal["right", "left", "both"]
    scheme: Literal["explicit_euler", "implicit_euler", "rk4"]
    eps: float | None          # None => structurally exact variant
    tol: float
    max_iter: int
    initial_condition: Callable[[np.ndarray], np.ndarray]
```

This is both the internal config object and the JSON schema `main.py` reads.
`hum2_boundary` raises a clear error if `eps` is `None` or `0`.

### `RunResult` (`results.py`)

```python
@dataclass
class RunResult:
    control: np.ndarray
    adjoint_final_datum: np.ndarray
    y: np.ndarray
    n_iter: int
    converged: bool
    residual_history: list[float]
    dual_functional: float
    primal_functional: float
    duality_gap: float
    wall_time_s: float
    git_commit: str
    timestamp: str
    problem: HeatProblem
```

JSON-serializable; written to `runs/<timestamp>_<commit>.json` by `main.py`.
