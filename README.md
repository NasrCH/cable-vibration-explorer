# Inclined cable — static shape and vibrations

Interactive tool for an inclined cable (stay cables, suspension-bridge cables) with two
models: the **static equilibrium shape under self-weight** (catenary solution) and the
**in-plane vibration frequencies and mode shapes** of the sagged catenary cable. Single
self-contained `index.html` file, no dependencies, no build step.

The user interface is in French. Live version once deployed:
`https://<user>.github.io/<repo>/`

## Static shape

### Physics

With chord length $L$, inclination $\alpha$, horizontal span $l = L\cos\alpha$, mass per
unit length $\mu$, horizontal tension $H$, and $\tau = \mu g\,l/H$:

$$\frac{Y(X)}{l} = \frac{1}{\tau}\left[\cosh C_1 - \cosh\!\left(C_1 - \tau\frac{X}{l}\right)\right],\qquad
C_1 = \ln\!\left(C + \sqrt{C^2 + e^{\tau}}\right),\quad
C = \frac{\tau e^{\tau}}{e^{\tau}-1}\tan\alpha$$

$Y$ is measured downward from the upper anchorage, so $Y(l) = l\tan\alpha$ (the far
anchorage lies on the chord). The tool reports the derived quantities (dimensionless
$\tau$, horizontal span, max vertical sag, cable arc length, elongation, along-chord
thrust $T = H/\cos\alpha$, support height difference) and plots the cable between its two
anchorages.

The chord is drawn at its **true angle**; the sag below the chord is exaggerated by a
**labelled factor** for legibility, with the real sag reported in the derived panel.

**Assumptions:** equilibrium under uniform self-weight, geometrically inextensible cable,
no point loads.

Checked numerically against known limits: $\tau\to 0$ gives a straight chord,
$\alpha = 0$ reduces to the parabolic sag $\tau l/8$, and $Y(l) = l\tan\alpha$ exactly.

## Vibrations

In-plane free vibrations of the sagged inclined cable are governed by

$$\varphi''(s) + \left(\frac{\Omega}{L_c}\right)^2\varphi(s) + \kappa\,Y''(s)\int_0^{L_c} Y''(s')\varphi(s')\,ds' = 0,
\qquad \varphi(0)=\varphi(L_c)=0,$$

on the arc-length coordinate $s\in[0,L_c]$ ($L_c = R_c\,l$ = cable length), with coupling
$\kappa = E_cA_c/(H\,l\,\chi^*)$ where $\chi^* = \frac{1}{l}\int_0^l (ds/dX)^4\,dX$.
Natural frequencies follow from $f = \Omega/(2\pi L_c\sqrt{\mu/H})$.

The eigenproblem is solved **two independent ways**, shown side by side in the tool as a
mutual check: finite differences + Jacobi (which also yields the mode shapes) and a
sine-Galerkin secular equation. Both agree to <0.1% and, at $\alpha=0$ with shallow sag,
reproduce the Irvine in-plane spectrum to <0.1% (validated). The document's own
Homotopy-Analysis-Method (HAM) closed form could not be transcribed reliably from the
source images, so this validated numerical solution — the exact limit the HAM
approximates — is used instead.

## History

Earlier revisions also included other vibration models (ideal string, bending stiffness,
Irvine sag theory, a general finite-difference eigensolver, and a custom-formula sandbox),
available in the git history.

## Deploy to GitHub Pages

**Option A — deploy from branch (simplest):** push to `main`, then in the repository
**Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch:
`main` / `(root)`**. The site appears at `https://<user>.github.io/<repo>/` within a minute.

**Option B — GitHub Actions:** the included `.github/workflows/deploy.yml` publishes the
site on every push to `main`. Set **Settings → Pages → Source: GitHub Actions** once.

The `.nojekyll` file disables Jekyll processing (not needed for a static file, and faster).

## References

- H.M. Irvine, T.K. Caughey, *The linear theory of free vibrations of a suspended cable*, Proc. R. Soc. Lond. A 341 (1974) 299-315.
- H.M. Irvine, *Cable Structures*, MIT Press, 1981.
