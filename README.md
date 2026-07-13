# Inclined cable — static shape

Interactive tool that draws the **static equilibrium shape of an inclined cable under its
own weight** (catenary solution), for civil-engineering use (stay cables,
suspension-bridge cables). Single self-contained `index.html` file, no dependencies, no
build step.

The user interface is in French. Live version once deployed:
`https://<user>.github.io/<repo>/`

## Physics

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

## Validation

Checked numerically against known limits: $\tau\to 0$ gives a straight chord,
$\alpha = 0$ reduces to the parabolic sag $\tau l/8$, and $Y(l) = l\tan\alpha$ exactly.

## History

Earlier revisions of this tool also computed cable **vibration** frequencies and mode
shapes (ideal string, bending stiffness, Irvine sag theory, a finite-difference
eigensolver, and a custom-formula sandbox). Those models were removed to keep the tool
focused on the static shape; they remain available in the git history.

## Deploy to GitHub Pages

**Option A — deploy from branch (simplest):** push to `main`, then in the repository
**Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch:
`main` / `(root)`**. The site appears at `https://<user>.github.io/<repo>/` within a minute.

**Option B — GitHub Actions:** the included `.github/workflows/deploy.yml` publishes the
site on every push to `main`. Set **Settings → Pages → Source: GitHub Actions** once.

The `.nojekyll` file disables Jekyll processing (not needed for a static file, and faster).

## References

- H.M. Irvine, *Cable Structures*, MIT Press, 1981.
