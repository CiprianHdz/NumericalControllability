# Theory extraction — manual sign-off checkpoint

Source of every quote below: `SourceThesis/TesisCIMATVersionFirmada.pdf`, "Numerical
and Constrained Controllability of the Heat Equation" (Cipriano Callejas Hernández,
CIMAT, 2024) — Chapters 1–2, the pre-numerical theory in Chapter 4, and
Appendix A — **plus the thesis's own LaTeX source**
(`SourceThesis/TesisLaTeXCode/`, untracked) for §6 below, which draws on material
present in source but not in the compiled PDF's page numbering (`introduction.tex`,
`Cap2.tex`, `Preliminary.tex`, `apendixA.tex`; quotes from these are tagged by file
and line number instead of page, to distinguish them from the PDF-sourced quotes
elsewhere in this document). This document extracts the **continuous theory**, plus
the **theory of time-/space-discrete uniform controllability** — i.e. theorems about
whether and how discretization preserves controllability as the mesh refines,
grounded in classical finite-dimensional (ODE) controllability results. It does
**not** cover discretization *pseudocode*, numerical-scheme implementation, or
algorithmic HUM variants (exact/penalized, HUM1/HUM2) — those remain
`ALGORITHMS.md`'s scope. In short: theorems about whether discretization preserves
controllability are in scope here; how to code the discretized solver is not. Exact
quotes are used throughout, each tagged with page/equation/theorem number (or
file:line for LaTeX-source quotes) and citation attribution. Reviewed against the
plan's `Docs/` outline before any `.tex` is written (Step 10). **By explicit
instruction, Chapter 3 (state-positivity / steady-state constrained
controllability) is excluded entirely** — this document and `Docs/theory.tex`
cover only the unconstrained/penalized HUM baseline that `HUM Code/` actually
implements; nothing about constrained controllability appears anywhere below.

Organized under the same eight headings as the plan, with **Internal** and
**Boundary** control kept as separate sub-items wherever the theory genuinely
differs — never blended into one generic paragraph. Per explicit instruction, the
new discrete-controllability material in §6 treats **internal (distributed)
control as the primary case**, with boundary control mentioned only where the
source itself draws a contrast.

---

## 1. Well-posedness of the heat equation

**Abstract standing assumption** (Ch. 2, p.13, before Eq. 2.1 — applies to both
cases at the abstract level):

> "y′(t) + Ay(t) = Bu(t) in (0,T), y(0) = y0 ∈ H, (2.1) where u ∈ L²(0,T;U) is the
> control function, A and B are unbounded linear operators respectively defined on
> their domains, dense linear subspaces, D(A) ⊂ H and D(B) ⊂ U. We also assume that
> system (2.1) is well posed, i.e., for every (y0,u) ∈ H×L²(0,T;U) there exists a
> unique solution y ∈ C([0,T];H) which depends continuously on the given data. The
> family of solutions ... is understood in the weak sense by means of the semigroup
> approach, see [Paz12]."

Citation: **[Paz12]** = Pazy, *Semigroups of Linear Operators and Applications to
Partial Differential Equations*, 2012.

### Internal (distributed) control

Eq. 4.3, p.50: *"Assuming that y0 ∈ L²(0,L) and v ∈ L²((0,L)×(0,T)) then problem
(4.3) admits an unique solution y ∈ C([0,T];L²(0,L)). In [AVOY96] it is also proved
the null controllability of (4.3)."*

### Boundary control

Eq. 4.1, p.49-50: *"For any y0 ∈ L²(0,L) and v ∈ L²(0,T) problem (4.1) has a unique
weak solution (defined by transposition) y ∈ C([0,T];H⁻¹(0,L)). It is well known
that system (4.1) is null controllable [AVOY96]..."*

Ch. 3 restates this for the boundary-controlled problem more generally (p.28):
*"Let us remark, by the regularity of the solution defined by transposition (see
Appendix A.1), we have y ∈ C([0,T];H⁻¹(Ω)). However, the objectives defined above
impose that y(·,T) ∈ L²(Ω) ... see [BK11]."*

**Solution-by-transposition construction** (Appendix A.1, pp.77-78) is the rigorous
justification behind the boundary case's `H⁻¹` well-posedness, and is also the
continuous-theory root of HUM2's dependence on "solution by transposition"
(`ALGORITHMS.md` notes HUM2 needs this):

> "We know that the solution of (A.1) verifies w ∈ L²(0,T;L²(Ω)) ∩ C⁰([0,T];H⁻¹(Ω)),
> (A.3), see [Eva10]." (p.77)
>
> "...consider the problem −pt−∆p=f(x,t) in Q, p=0 on Σ, p(x,T)=0 in Ω. (A.4) with
> f ∈ L²(0,T;L²(Ω)). It is well known that p, the solution of (A.4), belongs to
> L²(0,T;H¹₀(Ω)∩H²(Ω)) and pt ∈ L²(Q)." (p.78) — followed by the transposition
> identity (A.5) and: "It can be also proved that additionally z ∈ C⁰([0,T];H⁻¹(Ω)),
> hence the solution to (3.1) is well defined ... see [LM12]." (p.78)

Citations: **[AVOY96]** = Fursikov & Imanuvilov, *Controllability of Evolution
Equations*, 1996; **[BK11]** = Ben Belgacem & Kaber, *On the Dirichlet boundary
controllability of the 1D heat equation*, Inverse Problems, 2011; **[Eva10]** =
Evans, PDE textbook; **[LM12]** = Lions & Magenes, *Non-Homogeneous Boundary Value
Problems and Applications, Vol. 1*.

**Gap in the thesis, now filled independently — not from the thesis:** no
mild/semigroup well-posedness statement in the classic parabolic energy form
`y ∈ C([0,T];L²(Ω)) ∩ L²(0,T;H¹₀(Ω))` is stated anywhere in the thesis — it
consistently uses either the abstract semigroup framework (Ch. 2, citing Pazy) or
the transposition/weak-solution framework in `H⁻¹` (Ch. 3/4). For the document to
stand on its own, the standard energy-estimate theorem is added here directly:

> For `y0 ∈ L²(Ω)` and `f ∈ L²(0,T;L²(Ω))`, the linear heat equation
> `y_t − ∆y = f` in `Q`, `y=0` on `Σ`, `y(·,0)=y0`, has a unique weak solution
> `y ∈ C([0,T];L²(Ω)) ∩ L²(0,T;H¹₀(Ω))`, with `y_t ∈ L²(0,T;H⁻¹(Ω))`, obtained via
> the Galerkin method and the standard parabolic energy estimate
> `sup_t‖y(t)‖²_{L²} + ∫₀ᵀ‖y‖²_{H¹₀} dt ≤ C(‖y0‖²_{L²} + ‖f‖²_{L²(Q)})`.

Citation: **[Eva10, §7.1]** = Evans, L.C., *Partial Differential Equations*, AMS,
2010 — same book already cited elsewhere in this document (§1 above, `[Eva10]`) for
a thesis-attributed quote about the transposition-construction auxiliary problem;
this is a *different*, independently-added theorem from the same book, not
attributed to the thesis. Unlike the thesis quotes throughout this document (which
were verified line-by-line against the LaTeX/PDF source), this statement was not
checked page-by-page against Evans' book in this session — it is standard material
reproduced from memory of the textbook's structure, not a verified quote.

---

## 2. Regularity of states, controls, adjoint state

**There is no separate numbered "Regularity" theorem** in Ch. 2 or Ch. 4's theory
for the HUM-optimal control or the state. What genuinely exists:

### Internal control

Adjoint final datum lives in `L²(0,1)` — `φT ∈ L²(0,1)` (Eq. 2.29 context, p.24-25).
No further regularity statement beyond the well-posedness class of §1 above.

### Boundary control

The adjoint final datum's space is explicitly flagged as a point of departure from
the internal case (p.26): *"The adjoint system to the above is again (2.29),
however there is a subtle difference, φT ∈ H¹₀(0,1), which we shall discuss later."*
— this single sentence is the thesis's entire explicit statement on why the
boundary case needs the `H¹₀`/`H⁻¹` machinery, and should be quoted verbatim and
prominently in `Docs/`'s §3b.

### Shared / adjacent

- **Gap in the thesis, now filled independently — not from the thesis:** no
  parabolic-smoothing theorem is stated anywhere in the thesis text, though it is
  implicit in `D(A)=H²∩H¹₀` (p.24). The standard analytic-semigroup smoothing
  statement is added here directly: since `A=-Δ` with `D(A)=H¹₀(Ω)∩H²(Ω)`
  generates an analytic semigroup `(e^{-tA})_{t≥0}` on `L²(Ω)` (per the semigroup
  framework already cited in §1, `[Paz12]`), the uncontrolled solution satisfies
  `y0 ∈ L²(Ω) ⇒ y(t) ∈ D(A) ⊂ H¹₀(Ω) ∩ H²(Ω)` for every `t > 0`, with
  `‖y(t)‖_{D(A)} ≤ (C/t)‖y0‖_{L²(Ω)}` — the regularizing effect of the heat
  semigroup. Citation: **[Paz12, Ch. 2]** = Pazy, *Semigroups of Linear Operators
  and Applications to Partial Differential Equations*, 2012 — same book already
  used in §1 for the abstract well-posedness framework; this statement was not
  checked page-by-page against Pazy's book, unlike the thesis quotes elsewhere in
  this document.
- See §6 below for the *finite-dimensional* Kalman-condition/observability
  machinery underlying discrete uniform controllability — that is "regularity of
  controllability" for an ODE system, a different concern from the PDE state/control
  regularity discussed in this section, and is kept separate deliberately.

---

## 3. Controllability definitions

Shared across both control types — not split, matching the thesis's own
presentation (abstract definitions in Ch. 2, p.13-14).

Abstract definitions (Ch. 2, p.13, unnumbered bullets immediately after Eq. 2.1):

> "□ System (2.1) is said to be **exactly controllable** in time T > 0 if, for all
> (y0,y1) ∈ H×H, there exists u ∈ L²(0,T;U) such that the corresponding weak
> solution of (2.1) verifies y(T;y0,u) = y1.
> □ System (2.1) is said to be **exactly controllable to trajectories** in time
> T > 0 if, for all (y0, ŷ0) ∈ H×H, û ∈ L²(0,T;U), there exists u ∈ L²(0,T;U) such
> that ... y(T;y0,u) = y(T; ŷ0, û).
> □ System (2.1) is said to be **null controllable or exactly controllable to
> zero** in time T > 0 if, for all y0 ∈ H, there exists u ∈ L²(0,T;U) such that ...
> y(T;y0,u) = 0.
> □ System (2.1) is said to be **approximately controllable** in time T > 0 if, for
> all (y0,y1) ∈ H×H and for all ϵ > 0 given, there exists u ∈ L²(0,T;U) such that
> ... ‖y(T;y0,u)−y1‖H < ϵ."

Range-operator reformulation, Eqs. 2.2–2.4 (p.14): `F: L²(0,T;U)→H`,
`u ↦ y(T;y0,u)`, `R(T) = {y(T;y0,u) : u ∈ L²(0,T;U)}` — restating exact/null/
approximate controllability as `R(T)=H` / `R(T)∋0` / `R(T)` dense.

**Remark 1** (p.14): *"When there is no restriction on the norm of the given
initial condition the above properties hold globally. If such restriction exist we
say that the controllability holds locally."*

---

## 4. HUM duality theory (continuous, infinite-dimensional)

### 4a. Finite-dimensional (ODE) prototype — Kalman condition, ODE observability, ODE-HUM

The infinite-dimensional development below (Section 2.3) generalizes a
finite-dimensional template stated earlier in the same chapter for the ODE system
`y′(t)=Ay(t)+Bv(t)` in `(0,T)`, `y(0)=y0` (Eq. `eq12`, `A`,`B` real matrices). The
theorem *statements* are given here in full — previously this was only a one-line
pointer to `ALGORITHMS.md` — because they are the direct prerequisite for §6's
discrete/ODE uniform-controllability theory (a space semi-discretization reduces the
PDE control problem to exactly this ODE setting, see §6.1). Pseudocode/numerical
implementation of ODE-HUM stays in `ALGORITHMS.md`.

**Theorem [Kalman rank condition]** (`Preliminary.tex:136-146`, citing
**[Cor07, Theorem 1.16]** — matches the PDF's Theorem 1, p.14):

> "Consider the (Kalman) matrix defined as `K = [B, AB, ⋯, A^{n-1}B]`. (Kal) The
> Kalman matrix K is of rank n (or full rank) if and only if `L_T` is surjective."

— where `L_T: L¹(0,T;ℝⁿ) → C([0,T];ℝᵐ)`, `v ↦ ∫₀ᵀ e^{-(T-s)A}Bv(s)ds` (surjectivity
of `L_T` is equivalent to exact controllability of `y′=Ay+Bv`).

Adjoint ODE system (`eq:dualODE`, `Preliminary.tex:219-224`): `φ′(t) = -A*φ(t)` in
`(0,T)`, `φ(T)=φT`, where `A*` is the adjoint matrix of `A`.

**Theorem `pobs`** (`Preliminary.tex:231-239`, citing **[Zua02, Theorem 2.1.1]** —
matches the PDF's Theorem 2, p.17):

> "System (eq12) is controllable in time T if and only if the adjoint system
> (eq:dualODE) is observable in time T, that is, if there exists a constant
> C = C(T) > 0 such that, for any solution φ of (eq:dualODE), we have
> `|φ(0)|²_{ℝⁿ} ≤ C∫₀ᵀ|B*φ|²_{ℝᵐ} dt`. (obs) Both properties hold in all time T if
> and only if the Kalman rank condition (Kal) is satisfied."

**Theorem [HUM control] `teo:humcontrol`** (`Preliminary.tex:244-260`, matches the
PDF's Theorem 3, pp.17-19):

> "Let `𝒥: ℝⁿ → ℝ` be defined by `𝒥(φT) = ½∫₀ᵀ|B*φ|² + (y0,φ(0))`, where φ is the
> solution of (eq:dualODE) with final datum `φ(T)=φT`... Suppose 𝒥 has a minimizer,
> say `φ̂T ∈ ℝⁿ`... Then, setting `v = B*φ̂` ... it follows that v is the control of
> minimal norm of system (eq12) with initial datum `y(0)=y0`, such that null
> controllability holds at time T, i.e. `y(T)=0`."

**Gramian operator** `Λ` (`eq:gramian`, `Preliminary.tex:305-311`):

> "Consider the linear operator `Λ: ℝⁿ → ℝⁿ`, defined by
> `g ↦ Λ(g) := ∫₀ᵀ B*φg(s) dt`, where φg is the solution of (eq:dualODE) with final
> datum g. This is known as the Gramian operator and is well defined by duality,
> see [Boy13]."

This is the same **[Boy13]** citation used below (§4/§5) for the PDE-level Gramian —
the ODE and PDE constructions share the identical duality apparatus, just over
`ℝⁿ` instead of an infinite-dimensional Hilbert space `H`.

### Infinite-dimensional generalization

Section 2.3 (pp.21-24) is the infinite-dimensional development:

> "While in finite dimension ... the Kalman condition, in the PDEs setting more
> tools are needed ... it is known that the exact controllability of the
> one-dimensional heat equation in general does not hold, but the approximate
> controllability (weaker condition) does, see [Zua02]. In spite of that, the HUM
> control also works in the infinite dimensional setting." (p.21)

Adjoint system in `H` (Eq. 2.21, p.21): `φ′(t) = −A*φ(t)` in `(0,T)`, `φ(T)=g∈H`.

### Internal control

Gramian `Λ: H → L²(0,T;U)` (Eq. 2.15, restated p.22): *"linear operator ...
defined by (2.15), where U and H are suitable functional spaces so that it is
positive definite, symmetric and bounded, see [Boy13]."* Bilinear form and dual
functional (Eq. 2.22, p.22): `a(g,ξ) = ∫₀ᵀ B*φg · B*φξ`, and
`J(g) = ½a(g,g) + ⟨y0,φg(0)⟩` (unnumbered display, p.22), with `⟨x,y⟩` the
`H×H′` dual pairing. This *is* the infinite-dimensional HUM dual functional — it is
not given its own theorem number; the text defers the existence/coercivity
discussion to Remark 5 (see §7 below), and develops **penalized HUM** (§5) as the
actual route to a well-posed infinite-dimensional minimization.

**Gap in the thesis, now filled independently — not from the thesis:** the thesis
never states a standalone existence/uniqueness theorem for the unpenalized,
infinite-dimensional HUM minimizer — Section 2.3 defines `J` and moves directly to
penalization (§5). The standard argument, added here directly, is the original
Lions/HUM completion-space construction: let
`F := completion of H with respect to the seminorm ‖g‖_F := a(g,g)^{1/2}`. If this
seminorm is genuinely a norm on `H` — equivalent to a unique-continuation property
for the adjoint system (`B*φg ≡ 0` on `(0,T)` ⟹ `g = 0`) — then `H` embeds
continuously and densely in `F`, the linear form `g ↦ −⟨y0,φg(0)⟩` extends
continuously to `F`, and Riesz representation on the Hilbert space `F` gives a
unique `ĝ ∈ F` minimizing `J` over `F` (not merely over `H`). This is exactly the
completion Remark 5 already describes informally as "the space where J is coercive
... a much larger space than L²(0,1)" (quoted in full in §7 below) — the formal
existence/uniqueness statement the thesis gestures at without naming as a theorem.
Citation: **[Lio88]** = Lions, J.-L., *Contrôlabilité Exacte, Perturbations et
Stabilisation de Systèmes Distribués, Tome 1*, Masson, 1988 — the original HUM
source. Note: this is the first place in this document citing Lions' book
*directly*; the thesis itself never does so — it only cites Russell's review of it
(**[Rus90]**, §8 below). As with the other three independently-added results in
this document, this statement was not checked page-by-page against Lions' book in
this session (unlike the thesis quotes, which were verified line-by-line against
the LaTeX/PDF source) — no specific theorem/page number is claimed.

Observability inequality, continuous 1D internal-control case (Eq. 2.30, p.25):

> "If the observability property holds, i.e., if there exists C > 0 such that
> |φ(·,0)|²_{L²(0,1)} ≤ C ∫₀¹∫₀ᵀ |1ωφ|² dxdt, (2.30) then we have the coercivity
> of J."

Remark 6 (p.26) defers the proof of (2.30) itself: *"The observability inequality
(2.30) is also proved by means of more complicated techniques that we consider are
out of the reach of this work ... we refer to [AVOY96] where Carleman estimates are
used."*

### Boundary control

Per §2 above, the boundary case's dual functional `Jb(φT)` lives over `φT∈H¹₀(0,1)`
rather than `L²(0,1)` — the "subtle difference" quoted in §2. This is the continuous
root of `ALGORITHMS.md`'s HUM1 (H¹₀-norm CG, iterates the adjoint datum) vs. HUM2
(iterates the control directly via the primal penalized functional's closed-form
Gâteaux derivative, which itself depends on the solution-by-transposition
construction of §1). No further boundary-specific infinite-dim HUM theorem beyond
this is stated separately from the general Section 2.3 development.

---

## 5. Penalized HUM — convergence and threshold results

**Correction relative to earlier working notes:** Eq. 2.24 is **not** the
convergence result. It is the penalized functional's critical-point/variational
inequality. The convergence-as-`ε→0` statement is **Eq. 2.28 only**. Verified exact
statements below (this correction should propagate into `Docs/`'s citations).

Penalized functional, Eq. 2.23 (p.22): `Jϵ(g) = ½a(g,g) + ⟨y0,φg(0)⟩ + ϵ‖g‖H`.

Critical-point inequality, **Eq. 2.24** (p.22 — exact text):

> "Indeed, suppose Jϵ attains its minimum at ĝ ∈ H, then for any ξ ∈ H and t∈R, due
> to its convexity, we have 0 ≤ (1/t)(Jϵ(ĝ+tξ)−Jϵ(ĝ)), by taking carefully the
> limit t→0, this leads to the following inequality a(ĝ,ξ) + ⟨y0,φξ(0)⟩ ≤ ϵ‖ξ‖H,
> ξ∈H. (2.24) Thus, with (2.12) and the identity (2.14), we have ‖yv(T)‖ ≤ ϵ."

Constrained form, Eq. 2.25 (p.22): `min_{v∈L²(0,T;U)} (F1(v) + F2(Lv))`.

Boyer duality/penalized-HUM properties, pp.23-24, **[Boy13, Proposition 1.5]**:

> "1. Let Fε(v) := F1(v) + (1/2ϵ)‖yv(T)‖H, refer as the primal functional, and Jϵ
> the dual functional, they verify: Fε(v̂) = −Jε(ĝ). (2.27) In particular, this
> identity is a consequence of the Fenchel-Rockafellar duality theorem [ET99], so
> it is also true for ϵ = 0.
> 2. The final datum of the primal system satisfies y(T;y0,v̂) = −εĝ.
> 3. We have the estimates ‖y(T;y0,v̂)‖H ≤ ‖y(T;y0,0)‖H, ‖ĝ‖H ≤ (1/ε)‖y(T;y0,0)‖H"

**Eq. 2.28 (the convergence result)**, p.24, **[Boy13, Theorem 1.11]**:

> "Let us denote yε = (t;y0,v̂) the controlled solution by means of the penalized
> HUM control (2.26) denoted by vε = v̂(ε). Regarding the convergence of these two
> functions with respect to the penalization parameter, ϵ, we cite the Boyer's
> work [Boy13, Theorem 1.11], where it is proved that: yε(T) → P_QF(yf(T)), ϵ → 0,
> (2.28) where QF := {g : B*φg = 0, ∀t∈[0,T]} = ker Λ, is the set of
> non-observable states and P_QF is the orthogonal projection onto QF and yf is the
> free solution y(t;y0,0). Then, when QF={0} then the solution yε(T) converges to
> 0 as ε becomes smaller, which is the desired target."

**[Boy13, Proposition 1.7]**, p.24 (immediately after 2.28):

> "1. If system (2.5) is approximately or null controllable, then
> y(T;y0,v̂ε) → 0 when ε → 0.
> 2. If system (2.5) is null controllable, then vε → v0 as ε → 0, strongly in
> L²(0,T;U). The need of null controllability property for the strong convergence
> relies on the fact that, in such case inf_{L²(0,T;U)} Fε < ∞, when ϵ → 0."

Remark 4 (p.24): *"QF in finite dimension is the kernel of the adjoint Kalman
matrix (2.7) ... Thus, QF={0} if and only if the Kalman condition is satisfied or
equivalent, when the primal system is exactly controllable."*

### Internal control

The above (Eqs. 2.15/2.22/2.23/2.24/2.27/2.28) is stated directly for the internal
(`L²`) case.

### Boundary control

Same machinery restated over `H¹₀`/`H⁻¹` per §4b's dual functional. The
**condition-number result** — the "threshold in error rate" result the whole
document is built around — is stated once, generically, applying to whichever
Gramian is in play (internal or boundary), on **p.59**, unnumbered:

> "In our case, in (4.21), Λ is the Grammian operator, in particular, regarding the
> penalized HUM, we have Λ := Λ + ϵI, where I is the identity operator. However,
> the rate of convergence will depend strongly on the condition number for the
> operator Λ. Indeed, we have ‖ϵI + Λ‖ = ϵ + ‖Λ‖ and ‖(ϵI + Λ)⁻¹‖ = ϵ⁻¹, then
> νa = 1 + (1/ϵ)‖Λ‖."

CG convergence-rate bound, p.59, **[Glo08]** = Glowinski, *Lectures on Numerical
Methods for Non-Linear Variational Problems*, Springer, 2008:

> "In [Glo08], it is proven that the generated sequence of approximations
> {un}n≥1 verifies ‖un−u‖ ≤ C‖u0−u‖((√νa−1)/(√νa+1))ⁿ, ∀n≥1, where νa is the
> condition number of the bilinear form a(·,·), and νa = ‖Λ‖‖Λ⁻¹‖. Thus: – The
> closer is u0 to u, then the convergence is faster. – The closer is √νa to 1,
> then the convergence is faster."

Boundary-specific restriction: HUM2 requires `ε>0` structurally (thesis p.68-69,
already in `ALGORITHMS.md`) — this connects directly to the primal-functional
closed-form derivative of §1's transposition construction, which is only
well-defined for the penalized (`ε>0`) functional.

**No further quantitative threshold/error-rate result exists** in the thesis
beyond the above (confirmed by full-text search for `1/ε`, `e^{1/`, `exp(1/`
patterns). The only additional "threshold" material is the **experimental**
conjecture on p.72: *"we found out that the penalized term ε not only depends on
the discretization parameter h but also on the choice of the diffusion coefficient
α. We hypothesize that for problem (4.27) the parameter ϵ is of the form Chᵖ, for
C=C(α) a function depending on α and p greater than the order of accuracy of the
proposed space discrete operator."* — this is a **conjecture from numerical
experiments**, not proven continuous theory; it belongs in `Docs/` (if at all) as an
explicitly-labeled open conjecture, not alongside the proven Boyer results.

---

## 6. Discrete-in-time and discrete-in-space uniform controllability

Source for this whole section: the thesis's **LaTeX source**
(`SourceThesis/TesisLaTeXCode/`), not the compiled PDF — quotes are tagged by
`file:line`. **Internal (distributed) control is the primary case throughout**, per
explicit scope; boundary control appears only where the source itself draws a
one-line contrast.

### 6.1 Framing — three discretization formulations

`Cap2.tex:11-13` (chapter-opening framing): *"In recent years, few works have been
focused on the study of discrete control problems with the interest of establishing
its theoretical framework, see [BHS20] and references therein. In those works,
discrete control problems are proposed using three different formulations:
space-discrete, time-discrete and fully discrete."*

**The central PDE→ODE bridge statement**, `Cap2.tex:81`:

> "Notice that a space semi-discrete approximation of the parabolic control
> problems (eq:bcheat) or (eq:icheat), as we shall see below, **reduces the initial
> problem to a control problem for an ordinary differential equations (ODE's)
> system**."

The full state-of-the-art literature review, `introduction.tex:111-122`:

> "...we have the pioneering work [GLL90] for the wave equation, treated as an
> unconstrained optimization problem for the adjoint system, by means of duality
> theory of convex optimization [ET99], known as the Hilbert uniqueness method
> (HUM). This later adapted to the heat equation to find null controls, see
> [CGL94]. The HUM is part of a family of methods, so called dual methods, as well
> as the weighted HUM, which incorporates a small perturbation to avoid the
> oscillatory behaviour of the control function [Zua06]. Recently primal methods
> [FM14] (a variational approach which works directly with the primal system using
> Carleman weights) and least squared methods [MP14] have been used, nonetheless
> the results are not so distant to those with the dual approach. In particular,
> the study of discrete control of parabolic problems may be divided into three
> different formulations, which we briefly discuss the state of the art in each
> one:
>
> **Time-discrete setting.** In [Zh08], the authors proved that, in general, the
> semi-discrete multi-dimensional heat equation not only is not null controllable,
> but it is also not approximately null controllable. In this direction, it is
> proposed another sense of controllability in [BHS20] where they establish
> Carleman-type estimations for time-discrete approximations of the parabolic
> operator `−∂t−∆`, allowing to obtain certain controllability results.
>
> **Space-discrete setting**. This type of discretization has received more
> attention in the past few years [LZ98, LT06, BHLR10, CHS21]. However, most of the
> literature has been focused on whether some control properties can be retained
> after discretization. In particular, **the uniform controllability is only
> established for the semi-discrete 1D heat equation** [Zua06, LZ98]. In [Zua05],
> the authors show with a counterexample that the space semi-discrete version of a
> 2D heat equation is not controllable.
>
> **Fully discrete setting**. Results in this category are more scarce and
> limited. Recently in [CHS21], the authors showed by means of Carleman
> inequalities that a family of one dimensional parabolic equations are in fact
> null controllable, but in terms of certain relaxed sense of controllability."
>
> **"Thus, in general, discretization and controllability do not commute."**

New citations introduced in this section: **[Zh08]** = Zheng, C., *Controllability
of the time discrete heat equation*, Asymptotic Analysis, 2008; **[BHS20]** =
Boyer, F. & Hernández-Santamaría, V., *Carleman estimates for time-discrete
parabolic equations and applications to controllability*, ESAIM: Control,
Optimisation and Calculus of Variations, 2020; **[LT06]** = Labbé, S. & Trélat, E.,
*Uniform controllability of semidiscrete approximations of parabolic control
systems*, Systems & Control Letters, 2006; **[BHLR10]** = Boyer, F., Hubert, F. &
Le Rousseau, J., *Discrete Carleman estimates for elliptic operators and uniform
controllability of semi-discretized parabolic equations*, Journal de Mathématiques
Pures et Appliquées, 2010 (this thesis's own `biblio.bib` dates it 2010; the
Proposal's dossier cites what appears to be the same paper as "2011" in the
Cross-check section below — likely a preprint-vs-journal-issue date difference,
not a different paper; not resolved further here); **[Zua06]** = Zuazua, E., *Control and numerical
approximation of the wave and heat equations*, ICM Madrid, 2006 (**distinct from
[Zua02]** above — a different Zuazua paper, confirmed via `biblio.bib`); **[Zua05]**
= Zuazua, E., *Propagation, observation, and control of waves approximated by
finite difference methods*, SIAM Review, 2005; **[CHS21]** = Casanova, P.G. &
Hernández-Santamaría, V., *Carleman estimates and controllability results for
fully discrete approximations of 1D parabolic equations*, Advances in
Computational Mathematics, 2021; **[FM14]** = Fernández-Cara, E. & Münch, A.,
*Numerical exact controllability of the 1D heat equation: duality and Carleman
weights*, J. Optim. Theory Appl., 2014; **[MP14]** = Münch, A. & Pedregal, P.,
*Numerical null controllability of the heat equation through a least squares and
variational approach*, Eur. J. Appl. Math., 2014.

### 6.2 Space-discrete uniform controllability — the ODE-controllability bridge (internal control)

This is the formulation actually used in this work, and the only one of the three
proven to give uniform controllability — see §6.3 below for why the other two are
not pursued further here.

`Cap2.tex:71`: *"Based on the fact that the uniform controllability for the
space-discrete formulation of the 1D heat equation has been proved [LZ98], in this
chapter we will analyze numerical approaches using such formulation and based on
the HUM method."*

**Internal-control semi-discrete ODE system** (`discreteI`, `Cap2.tex:190-220`),
from central-difference approximation of `y_t − αy_xx = v(x,t)1_ω`:

> `[y′_1, y′_2, ⋯, y′_N]ᵀ = (α/h²)·tridiag(1,−2,1)·[y_1, y_2, ⋯, y_N]ᵀ + B_h v_h`

with `B_h = 1_{ω_h}` the diagonal `N×N` matrix `(1_{ω_h})_{j,j} = 1` if `x_j ∈ ω`,
`0` otherwise (the identity matrix when `ω` is the whole domain). Contrast with
boundary control (`Cap2.tex:230-232`, one-line remark): *"the difference between
both semi-discrete control problems (discreteB) and (discreteI) is in the term
`B_h v_h`; where in the case of the boundary control `v_h` is a scalar control and
`B_h` is a column vector, whereas in the distributed case `v_h` is a column vector
and `B_h` is in fact a matrix."*

**Explicit closed-form spectrum** of `A_h` (`Cap2.tex:272-281`, citing
`[Section 3.4]{leveque2007finite}` — add **[Lev07]** = LeVeque, R.J., *Finite
Difference Methods for Ordinary and Partial Differential Equations*, SIAM, 2007):
eigenvalues `λ_j = (2α/h²)(1 − cos(jπh))`, eigenvectors `e^j_i = sin(ijπh)`,
`j,i = 1,…,N`. Per `Cap2.tex:402`, this explicit spectrum is *why* the
uniform-controllability proof below is tractable: it "relies solely in fact that
the spectrum of the Laplacian can be computed explicitly."

**Governing principle — the precise definition of uniform controllability**,
`Cap2.tex:365`:

> "Controllability properties must be considered for proving the convergence of the
> discretized control problem. Following a classic way to prove controllability
> [AVOY96], an observability inequality has to be demonstrated for the associated
> discrete approximation of the adjoint system. Moreover, **if the constant of the
> observability inequality does not depend on the discretization parameter then
> uniform controllability is achieved**, see [Zua06]." (See §6.3 below: this
> uniform property is the one thing that fails, or only holds in a relaxed sense,
> for the time-discrete and fully-discrete formulations.)

**Important nuance — preserve, do not smooth over: why the Kalman rank condition
alone is not enough.** `Cap2.tex:376-377`:

> "Recall that there is a necessary and sufficient condition for the exact
> controllability of ODE's which is called the Kalman condition... However, since
> we have a dependency with respect to h, it is more convenient to transform the
> control problem into an observability problem for the semi-discrete adjoint
> system."

The Kalman rank condition (§4a above) is a **qualitative, binary** criterion for a
*fixed* finite-dimensional system — full rank or not. It says nothing about how the
observability constant `C` behaves as the system size grows with `h → 0`. Uniform
controllability requires exactly that missing quantitative control, which is why
the thesis reformulates the question as a discrete observability inequality instead
of stopping at "the Kalman matrix has full rank for every h."

Discrete adjoint system (`eq:discreteDual`, `Cap2.tex:392-399`): `φ_h′(t) =
−A_h^T φ_h(t)`, `φ_h(T) = φ_h^T`.

**Discrete observability inequality** (`Cap2.tex:404-413` — present in the LaTeX
source as a `\begin{comment}`-wrapped theorem, not rendered in the compiled thesis;
included here since the identical inequality reappears, uncommented, as item 1 of
the "uniformly controllable" definition below — no unverified content is added by
stating it in theorem form):

> "For any T > 0, there exists a positive constant C(T) > 0 such that
> `h∑ⱼ₌₁ᴺ|φ_j(0)|² ≤ C∫₀ᵀ|φ_N(t)/h|²dt`, holds for any φ_h solution of
> (eq:discreteDual) and any h > 0."

**Theorem [Null controllability] `teo:nullcontrolapprox`** (`Cap2.tex:416-430`,
active/compiled — this is the theorem previously left as a one-line stub, "Theorem
13," in §8 below):

> "For any T > 0 and `y⁰_h`, there exists a control `v_h ∈ L²(0,T)` such that the
> solution of control problem (discretePrimal)-(discreteB) satisfies `y_j(T)=0`,
> `j=1,…,N`. Moreover, let `y⁰ ∈ L²(0,L)`, then the controls `v_h` of system
> (discretePrimal) may be built such that `v_h → v` in `L²(0,T)` as `h → 0`, where
> v is a null control for the continuous heat equation provided the initial data in
> (bcheat) are chosen in an appropriate way."

**Explicit internal-control extension**, `Cap2.tex:433`:

> "Theorem (teo:nullcontrolapprox) can be modified to also obtain the convergence
> of the discretization for **the internal case** (discreteI), see
> [Zua06, Remark 3.4]."

This is the source's own explicit bridge confirming the null-controllability +
convergence theorem — stated above for the boundary case — carries over to internal
(distributed) control.

**"Uniformly controllable with respect to h" — definition and scope caveat**
(`Cap2.tex:439-454`, numbered list):

> "1. System (discretePrimal) is said to be **uniformly controllable with respect
> to h**, if the discrete observability inequality
> `h∑ⱼ₌₁ᴺ|φ_j(·,0)|² ≤ C∫₀ᵀ|φ_N(·,t)/h|²` holds for a suitable constant C > 0
> independent of h. **This uniform property only holds for the one-dimensional
> heat equation**, see [LZ98].
> 2. The convergence in L²(0,T) of `v_h → v` is a consequence of having an explicit
> way of choosing the initial datum `y⁰` in L²(0,T) by means of Fourier series, see
> [LZ98, Remark 1.2].
> 3. `v_h` converges to the control v of minimal norm, which can be built by means
> of the analytic HUM, presented in the previous chapter."

**2D counterexample — dimension-general scope caveat**, `introduction.tex:117`
(already quoted in §6.1): the space semi-discrete 2D heat equation is **not**
controllable [Zua05] — this is stated for the equation generally, not tied to
internal vs. boundary control, and directly explains why item 1 above restricts
the uniform property to 1D.

**[LT06] and [BHLR10]** are cited by the thesis only as part of the space-discrete
literature list (`introduction.tex:116`), not worked through in the thesis body —
noted here for completeness as further references establishing/refining uniform
controllability for the space-discrete case; their theorem statements are not
reproduced since the thesis itself does not state them.

**Gap to flag, not fill silently:** every uniform-controllability result above is
for **space** semi-discretization only — `h → 0` with time kept continuous
(`φ_h′(t)=−A_h^Tφ_h(t)`, `t` continuous throughout). No joint `(h,k)` fully-discrete
uniform-controllability theorem is stated in the thesis body. This is consistent
with, and should be read alongside, the open `(h,k)`-scaling question already
flagged in the "Cross-check" section below (the condition-number result `ν_a` is
`ε`-only, and a proven `(h,k)` joint scaling law remains open research) — the two
gaps reinforce each other and should not be presented as separately resolved.

### 6.3 Other formulations (brief, not pursued here)

Not used in this work — noted only for completeness, per the literature review
already quoted in full in §6.1. **Time-discrete**: a genuine negative result, not a
milder version of the space-discrete case above — [Zh08] shows the time-semi-discrete
multi-dimensional heat equation is in general neither null nor approximately
controllable, and [BHS20]'s Carleman estimates for the time-discrete operator give
only "another sense of controllability," not the same uniform-observability
property. **Fully-discrete**: results are scarce; [CHS21] proves null
controllability for a family of 1D parabolic equations via Carleman inequalities,
again only "in terms of certain relaxed sense of controllability."

---

## 7. Known theoretical/numerical issues

### Internal control

Ill-posedness of minimizing `J` over `L²(0,1)`, Remark 5 (p.25-26):

> "The space where J is coercive is a much larger space than L²(0,1), in fact, it
> is the completion of L²(0,1) with respect to the norm
> [½∫₀¹∫₀ᵀ|1ωφ|²dxdt]^{1/2}. This space can be hardly approximated by finite
> (discrete) dimensional basis, see [MZ10] and as we shall see the numerical
> problem of minimizing the dual functional is ill-posed."

**Confirmed: no `k∼e^{1/ε}`-type exponential blow-up statement exists anywhere in
this thesis** (verified by full-text search). Do not attribute that result to the
thesis — it is only in the separate Glowinski survey paper
(`SourceDocuments/Glowinski_and_numerical_control_problems.pdf`), which
`ARCHITECTURE.md`/`ALGORITHMS.md` already establish is background literature, not
implemented. The thesis's own cost-of-control result, within the scope of this
document, is the purely experimental control-norm blow-up as `ω` shrinks
(Table 4.3, p.63).

### Boundary control

The thesis's Conclusions-level remark on boundary-specific instability, p.75
(quoted here for `Docs/`'s §6b, exact wording to be confirmed against the
Conclusions chapter text directly when drafting): boundary control requires "a
change of norms and spaces" relative to internal control, making the method "more
unstable."

### §4.5 Remarks (p.73) — full verified quotes, split by relevance

> "□ The convergence of the HUM method based on Algorithm 3 is highly dependent on
> the time integrator; explicit schemes blow up in most of the pairs (N,M)."
> — general, both cases (Algorithm 3 is internal control specifically, but the
> time-integrator sensitivity is a shared numerical-theory point).
>
> "□ Simulations also show the failure of the optimization methods of the gradient
> type even when convergent and uniformly controllable finite-difference schemes
> are used. The use of preconditioners or iterative methods for solving large
> linear systems can be an option to improve the convergence of the descent
> conjugate gradient method." — general.
>
> "□ In consideration of the foregoing, there is a lack of robustness in the
> approximation; small perturbations in the input data produces wildly different
> results." — general.
>
> "□ In the distributed control we remark that, although the literature ensures
> that there is no restriction over the geometry of control, as pointed in [Zua02],
> the condition number for the Grammian operator Λ strongly depends on the location
> of ω, Also when it is discretized with respect to the space variable, ΛN, it
> blows up as soon as the N increases and the region becomes smaller, see [MP11].
> We verify this in Table 4.3 in terms of the cost of the control." — **internal
> control specific**.
>
> "□ There is an strong dependency of the regularity of the adjoint state with the
> minimizer of the (dual) functional. As such, considering different norms of
> penalization creates different solution behaviours to be controlled, see
> [Kin99]." — general, but most directly explains why `ALGORITHMS.md`'s
> exact/penalized/HUM1/HUM2 variants genuinely behave differently.
>
> "□ For HUM1 instead of a uniform distribution, maybe the results might be
> slightly better if a denser distribution near the extreme over the control acts
> is chosen. [KÖD21]" — **boundary control specific** (HUM1 named explicitly).

Citations: **[Zua02]** = Zuazua 2002; **[MP11]** = Münch & Periago, *Optimal
distribution of the internal null control for the one-dimensional heat equation*,
J. Diff. Eq., 2011; **[Kin99]** = Kindermann, *Convergence rates of the Hilbert
uniqueness method via Tikhonov regularization*, J. Optim. Theory Appl., 1999;
**[KÖD21]** = Kalimeris, Özsarı & Dikaios, *Numerical computation of Neumann
controllers for the heat equation on a finite interval*, arXiv 2021.

---

## 8. Other foundational citations for completeness

- **Origin of HUM**: **[Rus90]** = David L. Russell's review of J.-L. Lions,
  *Contrôlabilité exacte, perturbations et stabilisation de systèmes distribués*,
  Bull. AMS, 1990 — p.17: *"the Hilbert uniqueness method (HUM) consists in
  finding the control of minimal norm, introduced in [Rus90] to solve
  controllability problems for linear PDEs."* This is the thesis's pointer to
  Lions' book — cite it as such in `Docs/` rather than citing Lions' book directly
  (the thesis doesn't).
- **Fursikov–Imanuvilov**: **[AVOY96]** — null controllability of the 1D heat
  equation and the observability inequality via Carleman estimates (pp.25-26,
  49-50).
- **Lebeau–Robbiano**: confirmed **not cited anywhere** in this thesis (full-text
  search). **Gap in the thesis, now filled independently — not from the thesis:**
  the Lebeau–Robbiano spectral inequality is added here for completeness — a
  spectral inequality for the Dirichlet Laplacian's eigenfunctions on a bounded
  domain, of the form `‖u‖²_{L²(Ω)} ≤ C e^{Cλ} ‖u‖²_{L²(ω)}` for any
  eigenfunction-combination `u` with eigenvalues `≤ λ` and any open `ω ⊂ Ω`, which
  yields the null controllability (from any open subset `ω`, any time `T>0`) of the
  heat equation via a Lebeau–Robbiano-type iterative/telescoping argument — an
  alternative route to the Carleman-estimate route the thesis actually uses
  ([AVOY96]). Citation: **[LR95]** = Lebeau, G. & Robbiano, L., *Contrôle exact de
  l'équation de la chaleur*, Communications in Partial Differential Equations, 20
  (1-2), 1995. As with the other independently-added results in this document, the
  spectral-inequality statement above is standard-form, not a page-verified quote
  from the original paper (unlike the thesis quotes elsewhere, which were checked
  line-by-line against the LaTeX/PDF source) — no specific theorem/page number is
  claimed.
- **Numerical HUM origin (wave equation)**: **[GLL90]** = Glowinski, Li & Lions,
  *A numerical approach to the exact boundary controllability of the wave equation
  (I)*, 1990 — p.9, the pioneering numerical-HUM work (wave equation), "treated as
  an unconstrained optimization problem for the adjoint system, by means of
  duality theory of convex optimization [ET99]."
- **Adaptation to the heat equation**: **[CGL94]** = Carthel, Glowinski, Lions
  1994 — p.9, adapting HUM to the heat equation for null controls. (This is the
  same paper cited in the repo's original, now-removed README.)
- **Fenchel-Rockafellar duality**: **[ET99]** = Ekeland & Temam — used throughout
  §2.3 (Eq. 2.20, p.21; pp.22-23).
- **Base textbooks (Ch. 2 framing, p.12)**: *"we will only focus on the
  differential equations framework and will take as base textbooks
  [Cor07, Zab20, Zua02]"* — **[Cor07]** = Coron, *Control and Nonlinearity*, AMS
  2007 (also source of **Theorem 1**, Kalman rank condition, p.14, citing
  *"[Cor07, Theorem 1.16]"*); **[Zab20]** = Zabczyk, *Mathematical Control
  Theory*, 2020; **[Zua02]** = Zuazua's 2002 survey/lecture notes (repeatedly used
  for uniform-controllability discussion).
- **Discrete-continuous bridge**: **[LZ98]** = López, A. & Zuazua, E., *Some new
  results related to the null controllability of the 1-d heat equation*, Séminaire
  Équations aux Dérivées Partielles, 1998 — source of the discrete null-controllability
  + convergence-of-discrete-controls theorem (p.55-56 in the PDF's numbering). Now
  given **in full in §6** (the "Discrete-in-time and discrete-in-space uniform
  controllability" section) rather than left as a citation-only stub, since the
  document's scope now covers discrete/ODE uniform-controllability theory
  explicitly.
- **Ill-posedness / cost-of-control-vs-ω**: **[MZ10]** = Münch & Zuazua,
  *Numerical approximation of null controls for the heat equation: ill-posedness
  and remedies*, Inverse Problems 2010 (Remark 5, p.25-26; p.63).
- **Approximate controllability, variational approach**: **[FPZ95]** =
  Fabre-Puel-Zuazua 1995 (p.25).
- **Discrete Laplacian spectrum**: **[Lev07]** = LeVeque, R.J., *Finite Difference
  Methods for Ordinary and Partial Differential Equations*, SIAM, 2007 — source of
  the closed-form eigenvalues/eigenvectors of the semi-discrete Laplacian `A_h`
  used in §6.2 (`[Section 3.4]{leveque2007finite}`).

---

## Cross-check against the Proposal's literature review

Verified against `Proposal/numerical_controllability_research_plan.pdf` and
`Proposal/numerical_controllability_dossier_revised.pdf` (the dossier's Priority
A/B/C reference appendix, ~39 works, §7.2 and §18). **No result quoted above is
contradicted or shown to be a mistake** — the Proposal itself treats penalized HUM
(Boyer 2013) as "a mature numerical framework" and background, not a target for
correction. Two things nonetheless need explicit, careful wording in `Docs/` so it
doesn't misrepresent the current state of the art:

1. **Do not conflate Boyer (2013)'s `ε` (penalization) with Boyer &
   Hernandez-Santamaria (2026)'s `φ(τ)=τᵖ` (a *different* relaxation-scale device
   for a *moment-method* boundary-controllability formulation, item A12 in the
   dossier).** They are adjacent ideas, not the same result — the 2026 paper
   post-dates the thesis (2024) and solves a related-but-distinct problem. If
   `Docs/` discusses "known results for penalization," it must name Boyer (2013)'s
   `Jε(g)=J(g)+(ε/2)‖g‖²` machinery specifically and not present the 2026
   moment-method device as if it were the same or a refinement of it.
2. **The thesis's condition-number result (p.59, `ν_a=1+‖Λ‖/ε`) is a function of
   `ε` only** — it does not state how this scales under mesh refinement `(h,k)`.
   The dossier confirms a proven `(h,k)`-scaling law for the discrete HUM/penalized
   HUM Gramian's conditioning remains **open** (its own contribution target C3,
   "ideally proved rather than only observed," positioned as "a refinement of
   Boyer, Hubert & Le Rousseau (2011)" — a different, closest-precedent paper for
   joint `(h,k)` claims, not the 2013 penalized-HUM paper). `Docs/` should present
   the thesis's `ε`-only condition-number result honestly, and explicitly label the
   `(h,k)`-scaling question as open/future work — not as if it were already
   established.
3. **Preconditioning is now a crowded, fast-moving area** (17 works, several from
   2025–2026, postdating the thesis) — this matches and validates the thesis's own
   forward-looking remark (§4.5, p.73: *"The use of preconditioners or iterative
   methods for solving large linear systems can be an option to improve
   convergence"*), so no correction is needed there, just an optional note that
   this suggestion is now an active, competitive research area rather than an open
   suggestion nobody has pursued.
4. Glowinski-Lions-He (1994) and the classical Lions/Russell HUM-origin work are
   not cited anywhere in the Proposal documents — this is not a contradiction
   (the Proposal simply doesn't engage with them by name), just a gap to be aware
   of if `Docs/` leans on them as foundational citations (it will need its own
   attribution, independent of the Proposal).

## Summary of gaps to handle explicitly in `Docs/` (not fabricate)

1. ~~No classic energy-space (`L²∩H¹₀`) well-posedness theorem is stated in the
   thesis.~~ **Now filled independently in §1**, citing **[Eva10, §7.1]** — not a
   thesis result, and not page-verified against Evans' book the way thesis quotes
   are.
2. No standalone "Regularity" theorem for states/controls/adjoint exists in the
   thesis — the only genuine thesis regularity content is the one-sentence
   `φT∈H¹₀` remark (p.26) for boundary control. The parabolic-smoothing piece of
   this gap (`y0∈L² ⇒ y(t)∈H¹₀` for `t>0`) **is now filled independently in §2**,
   citing **[Paz12, Ch. 2]** — same caveat as above.
3. ~~No infinite-dimensional HUM existence/uniqueness *theorem* (as opposed to the
   penalized-HUM route) is separately numbered.~~ **Now filled independently in
   §4**, citing **[Lio88]** (the original Lions HUM source, cited directly here for
   the first time in this document) — same caveat as above.
4. `k∼e^{1/ε}` is **not** a thesis result — confirmed absent; do not attribute it
   here. (Unaffected by this round's additions — this was never a gap to fill, it's
   a non-result to keep excluded.)
5. The `ε∼Ch^p` scaling is an experimental **conjecture** (p.72), not a proven
   theorem — must be labeled as such if included. (Unaffected — stays a labeled
   conjecture, not something to "fill in" with an independent proof.)
6. ~~Lebeau–Robbiano is not in this thesis.~~ **Now filled independently in §8**,
   citing **[LR95]** — same page-verification caveat as above.
