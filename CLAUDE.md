# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single self-contained web app (`index.html`) — an interactive tool (UI in French) for
computing natural frequencies and mode shapes of vibrating cables. Aimed at
civil-engineering use (stay cables, suspension bridges). Deployed on GitHub Pages.

No build system, no dependencies, no package manager. Open `index.html` directly in a
browser (`xdg-open index.html`). All CSS is inline in `<head>`; all logic is one IIFE at
the bottom `<script>`. UI strings are French; code and comments are English.

Repo layout: `index.html` (the app), `README.md` (physics + deploy docs, uses GitHub
LaTeX math), `.nojekyll`, `.github/workflows/deploy.yml` (Pages deploy).

## The six models (`MODELS` registry in the `<script>`)

1. **`string`** — ideal taut string: `f_n = (n/2L)√(T/μ)`.
2. **`beam`** — tensioned member with bending stiffness EI. Option `bc`: `pinned` (exact
   `√(1 + n²π²EI/(TL²))` factor) or `fixed` (clamped approx. with `ξ = L√(T/EI)`, valid
   ξ ≳ 20). Refs: Zui 1996; Mehrabi & Tabatabai 1998.
3. **`irvine`** — horizontal cable with sag, Irvine linear theory. Option `plane`:
   `out` (Ω=nπ) or `in` (antisymmetric Ω=2kπ + symmetric roots of the transcendental
   equation, sorted; crossover at λ²=4π²). Refs: Irvine & Caughey 1974; Irvine 1981.
4. **`combined`** — sag AND bending together (the realistic stay cable). No closed form:
   finite-difference discretization of the in-plane PDE solved as a symmetric eigenvalue
   problem. Options `plane` and `bc`. Marked `heavy: true` → its refresh is debounced.
5. **`static`** — `staticShape: true`. NOT a frequency model: it computes the static
   equilibrium *profile* of an inclined cable under self-weight (catenary solution, eqs
   1.4-1.6 of the source doc): `Y(X)/l = (1/τ)[cosh C₁ − cosh(C₁ − τX/l)]`,
   `τ = μg·l/H`, `l = L·cos α`. `compute` returns `{staticShape:true, profile, derived,
   note}`; `refresh()` branches on `staticShape` to hide the frequency cards/table/mode
   slider and call `drawProfile()` instead of `draw()`. Verified against limits (τ→0 →
   straight chord; α=0 → parabola sag τl/8; Y(l)=l·tan α exactly).
6. **`custom`** — the dependency-free formula sandbox; params auto-detected from the formula.

Every frequency model's `compute(P, O, N)` returns `{modes, derived, note}`; `modes` are
`{n, label, family, f, shape}` and `shape(t)` maps `t = x/L ∈ [0,1]` to displacement.
A `staticShape` model returns a `profile` (`{pts, l, tanA, maxSag}`) instead of `modes`.

## Numerical core (most fragile part — verify before trusting changes)

- **Irvine transcendental solver** (`hSym`/`findSymRoots`): equation multiplied by
  `cos(Ω/2)` to remove tangent poles, then dense-scan + bisection. Never root-find on the
  raw `tan` form.
- **Finite-difference solver** (`buildFDMatrix`/`jacobiEig`/`solveFD`): governing
  equation `EI·v'''' − H·v'' + (EA/Le)(w/H)²∫v dx = μω²v` on `Ng=100` interior nodes.
  The `-Hv''` term is tridiagonal, `EIv''''` is the `[1 -4 6 -4 1]` stencil, and the sag
  integral is a rank-one `kc·h`-per-entry coupling — all symmetric, so a plain symmetric
  Jacobi eigensolver applies. Boundary conditions live in the D4 ghost-node correction:
  pinned → `-1·beta`, fixed → `+1·beta` on the two end diagonals. `ω² = eigenvalue/μ`.
  `familyOf` labels modes sym/antisym by comparing an eigenvector to its mirror.
- This solver subsumes the other models as limits and was validated numerically against
  all of them (string, pinned beam, Irvine spectrum, λ²=4π² crossover) to ~0.1%. If you
  touch any frequency formula or the solver, re-run those checks — this is used by a
  researcher for a thesis.

## Other implementation notes

- `g = 9.81` is module constant `G`, not user input.
- `draw()` normalizes each mode shape to unit amplitude, so shapes are illustrative and
  not to scale between modes.
- **Custom-formula evaluator** (`compile`/`extractVars`): rewrites `^`→`**`, rejects any
  identifier outside the declared vars or `SCOPE_FUNCS` whitelist (the only sandbox before
  `new Function()`). Name-based gate — fine for a static-hosted single-user tool.
- **Parameter UI** (`buildParamsUI`/`syncRow`): number inputs auto-expand their range when
  a value is typed outside it, so the slider display can never contradict the value.
- `requestRefresh()` debounces `heavy` models (60ms) so slider drags stay smooth; light
  models refresh synchronously.

## When editing

- New math function for the custom model: add to `SCOPE_FUNCS`.
- New physical model: add to `MODELS` + `MODEL_ORDER`; implement `compute(P, O, N)`, plus
  `params` and any `options` specs. Set `heavy: true` if it does an eigensolve. Set
  `staticShape: true` if it draws a profile rather than frequencies (see `static`).
- Keep UI text French, code/comments English.
