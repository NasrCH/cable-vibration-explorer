# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single self-contained web app (`index.html`) — an interactive tool (UI in French) for
an inclined cable, with two models: the **static equilibrium shape** under self-weight
(catenary), and the **in-plane vibration** frequencies / mode shapes of the sagged
catenary cable. Civil-engineering use (stay cables, suspension bridges). Deployed on
GitHub Pages.

No build system, no dependencies, no package manager. Open `index.html` directly
(`xdg-open index.html`). All CSS is inline in `<head>`; all logic is one IIFE at the
bottom `<script>`. UI strings are French; code and comments are English.

Repo layout: `index.html` (the app), `README.md` (physics + deploy docs, GitHub LaTeX),
`.nojekyll`, `.github/workflows/deploy.yml` (Pages deploy). `form-*.jpg` / `*.pdf` are the
researcher's source-document images — **gitignored, never commit them** (see the memory
`git-never-add-A-source-images`; they were accidentally pushed once). Never `git add -A`
here; stage files by name.

## The two models (`MODELS` registry in the `<script>`)

Both share `geom(L, alphaDeg, mu, H)`: `l=L·cosα` (horizontal span), `τ=μg·l/H`,
`C1=ln(C+√(C²+e^τ))` with `C=(τe^τ/(e^τ−1))tanα`, `Rc=[sinh C1−sinh(C1−τ)]/τ` (arc/span),
`Lc=Rc·l` (cable length), `χ*=(1/l)∫₀^l cosh⁴(C1−τX/l)dX` (numeric).

1. **`static`** — `staticShape: true`. Profile `Y(X)/l = (1/τ)[cosh C1 − cosh(C1 − τX/l)]`.
   Returns `{profile, derived, note}`; `refresh()` branches on `staticShape` to hide the
   frequency UI and call `drawProfile()` (true chord angle + labelled sag exaggeration).
   Verified: `τ→0`→chord, `α=0`→parabola `τl/8`, `Y(l)=l·tanα` exactly.
2. **`vib`** — `heavy: true`. Governing eigenproblem on arc-length `s∈[0,Lc]`:
   `φ'' + (Ω/Lc)²φ + κ·Y''(s)·∫Y''φ ds = 0`, `φ(0)=φ(Lc)=0`, `κ=EA/(H·l·χ*)`,
   `Y''(s)=−(Rcτ/Lc)(1+ψ²)^(−3/2)`, `ψ=sinh C1 − (Rcτ/Lc)s`. Solved **two independent
   ways** shown side by side as a mutual check: `solveFD` (finite differences + Jacobi,
   also gives eigenvectors→mode shapes) and `solveSpectral` (sine-Galerkin secular
   equation). `f = √(μ_eig·H/μ)/(2π)`, `μ_eig=(Ω/Lc)²`. Coupling sign is POSITIVE
   (negative breaks mode 1 vs Irvine).

## Numerical core (most fragile — verify before trusting changes)

- `jacobiEig(A, wantVec)` — symmetric Jacobi; `wantVec` returns eigenvectors (needed by
  `solveFD` for mode shapes). `familyOf` labels modes sym/antisym by mirror comparison.
- **Validated**: at `α=0` with shallow sag, `vib` reproduces the Irvine in-plane spectrum
  to <0.1%; FD and spectral agree to <0.1% across inclined cases too. If you touch `geom`,
  `d2Yds2`, `κ`, the coupling sign, or either solver, re-run the α=0 → Irvine check.
- The document's own HAM closed-form solution (eqs 3.32-3.35 of the source) could NOT be
  transcribed reliably from photos (c₀ dimensionally ambiguous, eq 3.34 too dense); the
  validated numerical solution above is used as the equivalent. See memory
  `catenary-vibration-model` for the full derivation and locked-in constants.

## Other notes

- `g = 9.81` is module constant `G`.
- `drawModes` normalizes each shape to unit amplitude (illustrative, not to scale between modes).
- **Parameter UI** (`buildParamsUI`/`syncRow`): number inputs auto-expand their range when
  a value is typed outside it, so the slider can never contradict the value.
- `requestRefresh()` debounces `heavy` models (60ms); light models refresh synchronously.

## When editing

- New model: add to `MODELS` + `MODEL_ORDER`; implement `compute(P,O,N)` + `params`/`options`.
  Set `heavy: true` if it eigensolves, `staticShape: true` if it draws a profile not frequencies.
- Keep UI text French, code/comments English.
