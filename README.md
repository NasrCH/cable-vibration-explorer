# Cable vibration explorer

Interactive tool to compute the natural frequencies and mode shapes of a vibrating
cable, for civil-engineering use (stay cables, suspension-bridge cables, transmission
lines). Single self-contained `index.html` file, no dependencies, no build step.

The user interface is in French. Live version once deployed:
`https://<user>.github.io/<repo>/`

## Models

| Model | Physics | Notes |
|-------|---------|-------|
| **Ideal string** | taut string, simple supports | reference case, pure harmonics |
| **Bending stiffness EI** | tensioned member with flexural rigidity | pinned (exact) and clamped (approx.) ends |
| **Sag (Irvine)** | horizontal cable with sag, linear theory | in-plane (symmetric + antisymmetric) and out-of-plane |
| **Coupled (Irvine + EI)** | sag **and** bending together | finite-difference eigensolve; the realistic stay-cable case |
| **Custom formula** | your own `f(n, ...)` | auto-detects variables, builds sliders |

### 1. Ideal string

$$f_n = \frac{n}{2L}\sqrt{\frac{T}{\mu}}$$

### 2. Bending stiffness

Pinned–pinned (exact):

$$f_n = \frac{n}{2L}\sqrt{\frac{T}{\mu}}\;\sqrt{1 + \frac{n^2\pi^2 EI}{T L^2}}$$

Clamped–clamped (approximation, valid for $\xi = L\sqrt{T/EI} \gtrsim 20$; Zui et al. 1996,
Mehrabi & Tabatabai 1998):

$$f_n = \frac{n}{2L}\sqrt{\frac{T}{\mu}}\left[1 + \frac{2}{\xi} + \frac{4 + n^2\pi^2/2}{\xi^2}\right]$$

### 3. Sag — Irvine linear theory

With non-dimensional frequency $\Omega = \omega L\sqrt{\mu/H}$ and the Irvine parameter

$$\lambda^2 = \left(\frac{wL}{H}\right)^2\frac{EA\,L}{H\,L_e},\qquad
w = \mu g,\quad d = \frac{wL^2}{8H},\quad L_e = L\!\left(1 + 8\left(\tfrac{d}{L}\right)^2\right)$$

- Out-of-plane and in-plane **antisymmetric** modes: $\Omega = n\pi$ and $\Omega = 2k\pi$ (string-like).
- In-plane **symmetric** modes: roots of the transcendental equation

$$\tan\!\left(\frac{\Omega}{2}\right) = \frac{\Omega}{2} - \frac{4}{\lambda^2}\left(\frac{\Omega}{2}\right)^3$$

Mode-1 crossover between symmetric and antisymmetric occurs at $\lambda^2 = 4\pi^2 \approx 39.5$.

### 4. Coupled sag + bending (finite differences)

No closed form. The in-plane governing equation (Mehrabi & Tabatabai 1998), shallow cable:

$$EI\,v'''' - H\,v'' + \frac{EA}{L_e}\left(\frac{w}{H}\right)^2\!\int_0^L v\,dx = \mu\,\omega^2 v$$

is discretized on a uniform grid (100 interior points), giving a symmetric eigenvalue
problem $A\,\phi = \mu\,\omega^2\phi$ solved with cyclic Jacobi rotations. Boundary
conditions (pinned / clamped) are applied through the fourth-difference stencil.
Out-of-plane and antisymmetric in-plane modes drop the sag coupling term ($\int v\,dx = 0$).

**Assumptions:** linear theory, shallow (parabolic) profile, uniform section and mass,
no damping, no aerodynamic effects. Numerical accuracy is ~0.1% on the lowest modes;
higher modes (n near 10) are slightly underestimated by the finite grid.

## Validation

The finite-difference solver was checked against every analytic case: it reproduces the
ideal-string and pinned-beam formulas and the Irvine transcendental spectrum to within
0.1%, and the $\lambda^2 = 4\pi^2$ crossover to five digits.

## Deploy to GitHub Pages

**Option A — deploy from branch (simplest):**

```sh
git init
git add .
git commit -m "Cable vibration explorer"
git branch -M main
git remote add origin git@github.com:<user>/<repo>.git
git push -u origin main
```

Then in the repository: **Settings → Pages → Build and deployment → Source: Deploy from
a branch → Branch: `main` / `(root)`**. The site appears at
`https://<user>.github.io/<repo>/` within a minute.

**Option B — GitHub Actions:** the included `.github/workflows/deploy.yml` publishes the
site on every push to `main`. Set **Settings → Pages → Source: GitHub Actions** once.

The `.nojekyll` file disables Jekyll processing (not needed for a static file, and faster).

## References

- H.M. Irvine, T.K. Caughey, *The linear theory of free vibrations of a suspended cable*, Proc. R. Soc. Lond. A 341 (1974) 299–315.
- H.M. Irvine, *Cable Structures*, MIT Press, 1981.
- H. Zui, T. Shinke, Y. Namita, *Practical formulas for estimation of cable tension by vibration method*, J. Struct. Eng. 122(6) (1996).
- A.B. Mehrabi, H. Tabatabai, *Unified finite difference formulation for free vibration of cables*, J. Struct. Eng. 124(11) (1998).
