# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single self-contained web app (`index.html`) — an interactive tool (UI in French) that
draws the **static equilibrium shape of an inclined cable under its own weight**
(catenary solution). Civil-engineering use (stay cables, suspension-bridge cables).
Deployed on GitHub Pages.

No build system, no dependencies, no package manager. Open `index.html` directly in a
browser (`xdg-open index.html`). All CSS is inline in `<head>`; all logic is one IIFE at
the bottom `<script>`. UI strings are French; code and comments are English.

Note: earlier revisions also computed vibration frequencies (ideal string, bending
stiffness, Irvine sag theory, a finite-difference eigensolver, a custom-formula sandbox).
Those models were removed on purpose — the tool is now static-shape only. They remain in
git history (commit `b0c02e7` and earlier) if ever needed again.

Repo layout: `index.html` (the app), `README.md` (physics + deploy docs, uses GitHub
LaTeX math), `.nojekyll`, `.github/workflows/deploy.yml` (Pages deploy).

## The physics (`compute()` in the `<script>`)

Static profile of an inclined cable, catenary solution (eqs 1.4-1.6 of the source doc):

```
Y(X)/l = (1/τ)·[cosh C₁ − cosh(C₁ − τX/l)]
τ = μg·l/H,  C = (τe^τ/(e^τ−1))·tan α,  C₁ = ln(C + √(C²+e^τ))
```

- `L` = chord length between anchorages; `α` = chord inclination; `μ` = mass/length;
  `H` = horizontal tension component.
- `l = L·cos α` is the horizontal span; `Y` is measured downward from the upper anchorage,
  so `Y(l) = l·tan α` (far anchorage lies exactly on the chord).
- `compute()` returns `{profile:{pts,l,tanA,maxSag}, derived:[…], note}` and is pure —
  reads the module-level `params` map, no DOM.
- Verified numerically against limits: `τ→0` → straight chord; `α=0` → parabolic sag
  `τl/8`; `Y(l) = l·tan α` exactly. Re-run those checks if you touch the formula.

## Other implementation notes

- `g = 9.81` is module constant `G`, not user input.
- **`drawProfile()`** draws the chord at its TRUE angle (uniform scale) and exaggerates
  the sag below the chord by a labelled factor `K` (target ≈ 18% of span, clamped
  `[1, 500]`) so realistic tiny sags stay visible. The real sag is always shown in the
  derived panel ("Flèche verticale max"). Anchorages are labelled A (upper) and B (lower).
- **Parameter UI** (`buildParamsUI`/`syncRow`): slider + number + min/max per parameter,
  from `PARAM_SPECS`. Typing a value outside the range auto-expands it, so the slider
  display can never contradict the value.

## When editing

- Add/adjust a parameter: edit `PARAM_SPECS` (also add it to `compute()` if it feeds the
  physics).
- Keep UI text French, code/comments English.
