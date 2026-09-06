# Theory extraction — manual sign-off checkpoint

Source of every quote below: `SourceThesis/TesisCIMATVersionFirmada.pdf`, "Numerical
and Constrained Controllability of the Heat Equation" (Cipriano Callejas Hernández,
CIMAT, 2024) — Chapters 1–2, the pre-numerical theory in Chapter 4, Chapter 3, and
Appendix A. This document extracts the **continuous** theory only (nothing about
discretization/pseudocode — that's `ALGORITHMS.md`). Exact quotes are used
throughout, each tagged with page/equation/theorem number and citation attribution.
Reviewed against the plan's `Docs/` outline before any `.tex` is written (Step 10).

Organized under the same eight headings as the plan, with **Internal** and
**Boundary** control kept as separate sub-items wherever the theory genuinely
differs — never blended into one generic paragraph.

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

Semilinear boundary case (Ch. 3, p.32), same transposition + fixed-point route:
*"...by means of transposition and fixed point methods [LM12], we have
y ∈ C([0,T];L²(Ω))."*

**Gap to flag, not fill silently:** no mild/semigroup well-posedness statement in the
classic parabolic energy form `y ∈ C([0,T];L²(Ω)) ∩ L²(0,T;H¹₀(Ω))` is stated
anywhere in the thesis — it consistently uses either the abstract semigroup
framework (Ch. 2, citing Pazy) or the transposition/weak-solution framework in `H⁻¹`
(Ch. 3/4). If `Docs/` wants the energy-space estimate stated explicitly, it must be
added with its own citation (e.g. Evans) — not attributed to this thesis.

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

- Appendix A.2, "On the construction of regular controls" (p.78 onward) is
  technical machinery for Ch. 3's *constrained* (state-positivity) problem
  specifically, not a general HUM control-regularity result — do not cite it as if
  it applied to the unconstrained baseline.
- Ch. 3 Remark 7 (p.32): *"The controllability can be achieved at exactly
  T0(y0,y1), but the controls need to have a special regularity [LTZ17, Theorem
  4]."* — citing **[LTZ17]** = Lohéac, Trélat & Zuazua, *Minimal controllability
  time for the heat equation under unilateral state or control constraints*, 2017.
  Also belongs to the constrained (Ch. 3) thread, not the baseline HUM.
- No parabolic-smoothing theorem (e.g. `y0∈L² ⇒ y(t)∈H¹₀` for `t>0`) is stated
  anywhere in the thesis text, though it is implicit in `D(A)=H²∩H¹₀` (p.24). If
  `Docs/` wants this stated as a theorem, it needs its own citation, not the thesis.

---

## 3. Controllability definitions

Shared across both control types — not split, matching the thesis's own
presentation (abstract definitions in Ch. 2, p.13-14, restated concretely for the
boundary-controlled problem in Ch. 3, p.27-29).

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

Ch. 3 restates for the boundary-controlled heat equation (Eq. 3.1-3.2, p.27-28) and
adds:

**Definition 1** (p.29): *"We say the (3.1) is (exactly) controllable to
y1 ∈ L²(Ω) if for any initial data y0 ∈ L²(Ω), there exist some Ty0 > 0 and control
u ∈ L²(Σ), such that the solution of (3.1) verifies y(Ty0) = y1."*

**Definition 2 (Steady-State)** (p.30, Eq. 3.5): `−∆ȳ=0` in Ω, `ȳ=ū` on `∂Ω`.

**Definition 3 (Steady-State Controllability)** (p.30): *"Let y0,y1 be steady
states of (3.1). System (3.1) is said to be exactly-steady-state controllable in
time T > 0 if there exist a control u ∈ L²(Σ) such that the solution of (3.1)
verifies y(x,0) = y0 and y(x,T) = y1 for all x ∈ Ω."*

---

## 4. HUM duality theory (continuous, infinite-dimensional)

The finite-dimensional template (**Theorem 2**, observability ⇔ controllability,
p.17, citing **[Zua02, Theorem 2.1.1]**; **Theorem 3**, HUM control, pp.17-19) is
already in `ALGORITHMS.md`. Section 2.3 (pp.21-24) is the separate
infinite-dimensional development:

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
discussion to Remark 5 (see §6 below), and develops **penalized HUM** (§5) as the
actual route to a well-posed infinite-dimensional minimization.

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

## 6. Known theoretical/numerical issues

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
implemented. The thesis's own cost-of-control results are: (a) an exponential-in-**time**
estimate from Theorem 4's proof (Ch. 3, p.31, `‖v‖_{L∞(Σ)} ≤ e^{-2λ1T}C(T-τ)‖z0‖_{L²(Ω)}`
— cost in `T`, not `ε`), and (b) the purely experimental control-norm blow-up as
`ω` shrinks (Table 4.3, p.63).

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

## 7. Chapter 3 aside — state-positivity / constrained controllability

Flagged throughout as **related but not implemented** in `HUM Code/` (which
implements unconstrained/penalized HUM only).

**Lemma 1 (Maximum principle)** (p.29), citing **[Pao12, Lemma 4.1]** — justifies
that the state-positivity constraint (Eq. 3.3) reduces to positivity of the
boundary control (Eq. 3.4).

**Theorem 4** (p.30, linear case, full statement):

> "Let y0 ∈ L²(Ω) be such that y0 ≥ 0, and let y1 ∈ L²(Ω) be a steady-state of
> (3.1). We assume y0 ≠ y1, and that there exists ϵ > 0 such that y1 ≥ ϵ. Then,
> there exists T0 = T0(y0,y1) > 0 such that for any T > T0, there exists a control
> u ∈ L²(Σ) such that the corresponding solution of (3.1) is non-negative and
> satisfies y(·,T) = y1."

Proof sketch (pp.30-31): shift `z=y−y1` (Eq. 3.6); observability inequality
(Eq. 3.7 → 3.9-3.10 via Parseval and the first Dirichlet-Laplacian eigenvalue λ1);
regular controls via Appendix A.2; cost-of-control estimate
`‖v‖_{L∞(Σ)} ≤ C(T)‖z0‖_{L²(Ω)}`, `C(T)=e^{-2λ1T}C(T-τ)`, `C(T)<1` for `T>T0`.

**Remark 7** (p.32): the "waiting time" minimal-time phenomenon, plus the
regularity caveat citing **[LTZ17, Theorem 4]** for controls achieving exactly
`T0`.

**Theorem 5** (p.33, semilinear staircase method, full statement):

> "Let y0 and y1 be path connected bounded steady states. Assume there exists
> ν > 0 such that ūs ≥ ν ∀s ∈ [0,1]. (3.13) Then, if T is large enough, there
> exists u ∈ L∞(Σ) such that □ Problem (3.11) with initial datum y0 and control u
> admits a unique solution y verifying y(·,T) = y1; □ Moreover, u ≥ 0 a.e. on
> (0,T)."

Requires **Lemma 2** (local controllability to trajectories, p.33), citing
**[PZ18, Lemma 2.1]** = Pighin & Zuazua, *Controllability under positivity
constraints of semilinear heat equations*, 2018.

Adjacent Ch. 3 citations for context: **[CT04]** Coron-Trélat 2004 (staircase
method origin); **[Sch80]** Schmidt 1980 (first approximate steady-state result);
**[OY93]** Imanuvilov 1993, **[FC97]** Fernández-Cara 1997 (semilinear null
controllability); **[CCG05]**, **[MRR16]** (negative results, n≥2, time-only
controls); **[GTGT77]** Gilbarg-Trudinger (elliptic regularity ensuring the
steady-state set is nonempty, p.32).

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
  search). If `Docs/` wants it for completeness on spectral/approximate
  controllability, it must be added independently, not attributed to the thesis.
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
- **Discrete-continuous bridge**: **[LZ98]** = López & Zuazua — source of
  **Theorem 13** (discrete null controllability + convergence of discrete
  controls, p.55-56). This is discretization content already covered by
  `ALGORITHMS.md`'s scope, flagged here only for citation completeness.
- **Ill-posedness / cost-of-control-vs-ω**: **[MZ10]** = Münch & Zuazua,
  *Numerical approximation of null controls for the heat equation: ill-posedness
  and remedies*, Inverse Problems 2010 (Remark 5, p.25-26; p.63).
- **Approximate controllability, variational approach**: **[FPZ95]** =
  Fabre-Puel-Zuazua 1995 (p.25).

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

1. No classic energy-space (`L²∩H¹₀`) well-posedness theorem is stated in the
   thesis — if wanted, cite a standard textbook independently.
2. No standalone "Regularity" theorem for states/controls/adjoint exists — the only
   genuine regularity content is the one-sentence `φT∈H¹₀` remark (p.26) for
   boundary control, and Ch. 3's `[LTZ17]`-cited control-regularity caveat (which
   belongs to the constrained problem, not the baseline).
3. No infinite-dimensional HUM existence/uniqueness *theorem* (as opposed to the
   penalized-HUM route) is separately numbered — Section 2.3 develops the
   functional directly and moves to penalization.
4. `k∼e^{1/ε}` is **not** a thesis result — confirmed absent; do not attribute it
   here.
5. The `ε∼Ch^p` scaling is an experimental **conjecture** (p.72), not a proven
   theorem — must be labeled as such if included.
6. Lebeau–Robbiano is not in this thesis — add independently if desired, with its
   own citation, not as if the thesis referenced it.
