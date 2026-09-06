# Algorithm inventory

Lives in `Proposal/` alongside `ARCHITECTURE.md`, the research plan, and the dossier
— a planning/reference document, not part of the `HUM Code/` package itself.

Manual sign-off checkpoint. This document is the authoritative statement of which
HUM variants exist in `RawCode/`, which are canonical sources for the baseline, and
which are excluded as broken or abandoned. **Read and confirmed before
`optimization/hum2_boundary.py` (Algorithm 5) is implemented** — the zero-IC bug fix
described below acts on this inventory, not on an assumption.

Sole algorithmic reference: `SourceThesis/TesisCIMATVersionFirmada.pdf`, Ch. 4 +
Appendix B (Algorithms 1–5, pp. 81–89).
`SourceDocuments/Glowinski_and_numerical_control_problems.pdf` was read in full and
confirmed to cover neither boundary control nor CG-based HUM (it uses a different,
Carleman-weighted mixed-FEM / Uzawa–Arrow-Hurwicz method, and only internal control)
— it is background literature only, not implemented here.

## Internal control — two variants (Algorithm 3, thesis p.87)

| | **Exact** | **Penalized (Boyer-style)** |
|---|---|---|
| Source | `RawCode/HUM.py`, `RawCode/main.py` | `RawCode/HUM_interno.py`, confirmed by `RawCode/Internal-Control-Heat-Implicit-Euler-Penalizado__.ipynb`'s explicit `Jε`/`Fε` markdown derivation |
| Gradient | `g = y(T) − target` — **no ε term at all** | `g = ε·f + y(T) − y1` |
| Functionals | plain `J`, `F_primal` | `J_eps`/`F_primal_eps`, matching Boyer's `Jε(φT)=J(φT)+(ε/2)‖φT‖²`, `Fε(u)=F(u)+(1/2ε)‖y(T)‖²` |
| → new module | `optimization/hum_internal.py`, `eps=None` path | `optimization/hum_internal.py`, `eps=<float>` path |

## Boundary control — three working variants (Algorithms 4 & 5, thesis pp.88-89)

| | **Exact HUM1** | **Penalized HUM1** | **Penalized HUM2** |
|---|---|---|---|
| Source | `RawCode/HUM_boundary.py` | `RawCode/HUM_boundary_modified.py` (**canonical**) | `RawCode/hum_frontera_2.py` |
| Norm | H¹₀ via `grad_vector`-based gradient dot-product (own discretization — not ported, see below) | H¹₀ via quadratic form `√(h·xᵀAx)` (**canonical, adopted**) | L²(0,T) — over time, not space |
| Iterates | adjoint datum f | adjoint datum f | control u(t) directly |
| Solve order | backward (adjoint) → control → forward (zero IC) | backward (adjoint) → control → forward (zero IC), then mapped through `(Ah)⁻¹` | **forward first** (zero IC, control=iterate) → elliptic map → backward (adjoint) → boundary trace |
| ε term | none | `g = ε·f + (Ah)⁻¹y(T)` (multiplicative) | baked into elliptic map `f0 = ε·(Ah)⁻¹y(T)` |
| ε restriction | n/a | 0 or >0 | **must be >0** (thesis p.68-69 — HUM2 needs the closed-form Gâteaux derivative from solution-by-transposition, which requires penalization) |
| Status | works | works | works, **confirmed zero-IC bug** |
| → new module | `optimization/hum1_boundary.py`, `eps=None` path | `optimization/hum1_boundary.py`, `eps=<float>` path | `optimization/hum2_boundary.py`, **bug fixed on migration** |

### The confirmed HUM2 bug

`RawCode/hum_frontera_2.py`, `HUM()` (line 164):

```python
y = BackwardEuler(x0, t, A, w0, tau)  # with 0 initial value
```

The comment says zero initial value, but `x0` (the real, nonzero initial condition —
`10·sin(πx)`, set globally at line 269) is passed instead of a zero vector. Compare to
`RawCode/HUM_boundary_modified.py`'s correct handling of the equivalent step (line
165, inside `gramiano`, called from `HUM()`'s loop):

```python
phi, u, y = gramiano(w, 0 * x0, tau, h, A)  # zero initial datum as is the grammian defined
```

**Fix, on migration into `optimization/hum2_boundary.py`:** the forward solve inside
the CG loop uses IC `= zeros`; the real `x0` (initial condition) enters the algorithm
only once, in the one-time additive term folded into `rhs` before the loop starts —
exactly as Algorithms 3 and 4 already do it correctly. This is a correctness fix
relative to `hum_frontera_2.py` as written; the discrepancy is documented in
`README.md` rather than silently reproduced.

## Excluded — broken or abandoned (left in `RawCode/`, not migrated)

Re-verified directly against `RawCode/` source and the thesis's own Appendix B
pseudocode (Algorithms 4 & 5, eq. B.10–B.15, pp. 88–89) — see "Thesis cross-check"
below each bullet. Confirmed: none of the three is a fixable/distinct algorithm, so
none is revived even where the bug itself is trivial to patch.

- **`RawCode/HUM_frontera.py`, `HUM2()`** — H⁻¹-norm boundary variant iterating the
  adjoint datum f. Its `gramiano()` helper returns a variable `sol_u` that is never
  assigned inside the function → calling it raises `NameError`. This is exactly what
  the file's own `test_single()` calls, so **this script cannot run to completion as
  written**.
  *Thesis cross-check:* Algorithm 4's pseudocode (B.10–B.11) iterates the CG in
  **H¹₀** exclusively (`‖g‖²_{H¹₀}` in the step-size, stopping-criterion, and γₙ
  formulas) — H⁻¹ never appears in the boundary-control pseudocode. So fixing the
  `NameError` alone would not recover Algorithm 4; it would still be a wrong-norm
  variant, already superseded by the canonical H¹₀ implementation in
  `HUM_boundary_modified.py`.
- **`RawCode/HUM_frontera.py`, `HUM()` ("Algoritmo 2.5")** — H⁻¹-norm, also iterates
  f, but computes `g0 = (1/ε)·f0 + (Ah)⁻¹y(T)` — **dividing** by ε rather than
  multiplying, inconsistent with every other working file's convention. Never
  actually called by this file's own `test_single()` — dead code.
  *Thesis cross-check:* same H⁻¹/H¹₀ mismatch as `HUM2()` above against Algorithm 4's
  `g⁰ = εf⁰ − (Ah)⁻¹y(T)` (B.10, multiplicative) — no fix recovers a distinct
  algorithm here either.
- **`RawCode/Boundary_Control_Heat_Euler_Penalizacion.ipynb`** — attempts a
  "large-k" penalization trick (`k=1e+100`, `f0=np.linalg.solve(-A, k*y[-1])`), but
  the computed `f0` is discarded before the CG loop starts (the loop resets to
  `f0=phi_T=zeros`, cell 26) — an abandoned experiment, not a working second
  penalized-HUM1 path.
  *Thesis cross-check:* even fixing the discard bug, the large-k elliptic trick
  doesn't reproduce Algorithm 5's construction (B.12–B.15: explicit forward solve
  with control `u⁰`, then exact `(Ah)⁻¹` map, then backward adjoint solve) — it's a
  cruder, non-canonical stand-in for the same idea, superseded by the correct-shape
  `hum_frontera_2.py` (whose only defect is the documented zero-IC bug above, already
  slated for a migration-time fix).

These three are documented in `HUM Code/README.md` as known-broken/abandoned when the
migration lands, not silently dropped.

Also excluded, unrelated to the HUM algorithm family (unchanged from the original
scoping): `RawCode/Semi_approx.py` (forward-solver accuracy test, no CG loop),
`RawCode/iterativePoisson.py`/`iterativePoisson` (1D Poisson + Gauss-Seidel, no time
dependence or control), `RawCode/prueba.py`/`prueba` (empty/scratch), and the
extensionless drafts (`HUM_boundary`, precursor to `HUM.py`).

## Norm discretization note

`HUM_boundary.py`'s alternate H¹₀ discretization (gradient-dot-product form, via its
own `grad_vector` helper) is **not** carried into the exact-HUM1 variant as a second
selectable norm. The exact-HUM1 code path reuses the canonical quadratic-form
`H10Norm` with no ε term — mirroring the "same CG shape, ε on/off" relationship
already established between exact and penalized internal control. If this alternate
discretization turns out to matter later, it can be added as a second `Norm`
implementation without touching the CG engine (`optimization/linear_cg.py`).

## Cross-cutting conventions adopted for the baseline

- `ε` is always a **multiplicative** penalization weight, matching thesis Eq. 4.20
  (`Jε(g) = J(g) + (ε/2)‖g‖²`) — never the `1/ε` convention found in
  `HUM_frontera.py`'s dead `HUM()` function.
- The *exact* variants of internal control and HUM1 structurally omit the `ε` term
  entirely (`eps=None`), rather than merely passing `eps=0`, matching how
  `HUM.py`/`HUM_boundary.py` are actually written (no `eps` term appears in their
  gradient formulas at all).
- `H⁻¹` inverses (`HInvNorm`, and the `(Ah)⁻¹` map inside HUM1/HUM2) are computed via
  **one cached factorization per solve**, not recomputed on every CG iteration or
  every call as several RawCode files do (`np.linalg.inv(-A)` inside the loop).
