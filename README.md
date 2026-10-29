# AE-370_Proj1
Atmosphere spacecraft skipping 

# Atmospheric Skip Entry of an Artemis I-Inspired Capsule

AE 370 Project 1, Fall 2026. Forward Euler integration of a point-mass entry model to find which
entry angles produce an atmospheric skip, and what peak aerodynamic load each produces, as a
function of entry speed.

## Requirements

Python **[3.x]** with `numpy` and `matplotlib` (add `scipy` if any file uses it).

```
pip install numpy matplotlib
```

No ODE library advances the solution. Time stepping is our own forward Euler in `euler.py`.
`convergence.py` also contains an RK4 step, used only as an accuracy reference for error studies
and never in the study itself.

## Repository Layout

All files must be in the same folder, and every script should be run from that folder.

| File | Purpose |
|---|---|
| `parameters.ipynb` | Constants, vehicle values, initial conditions, event thresholds |
| `model.ipynb` | Density, aerodynamic forces, load factor, state derivatives |
| `euler.ipynb` | Forward Euler time marching |
| `baseline.ipynb` | Runs one case and saves diagnostics |
| `verification.ipynb` | Independent checks of the equations and Euler code |
| `convergence.ipynb` | Step refinement, error calculations, recommended time step |
| `sweep.ipynb` | Entry-angle and entry-speed study, boundary refinement |
| `plots.ipynb` | Generates report figures from saved results |

## Reproducing the Results

Run the scripts in this order:

```
python verification.py   # about 30 s; prints PASS/FAIL for each check
python convergence.py    # a few minutes; add --quick for a faster, coarser run
python baseline.py       # [what it saves, where]
python sweep.py          # [runtime, what it saves, where]
python plots.py          # [which figures it writes, where]
```

`verification.py` should report that all checks passed before any results are trusted.
`convergence.py` writes CSV files and `fig_convergence.png` to `results/` and prints the
time step used in the study.

## Model Summary

The state is $\mathbf{s} = [h,\ V,\ \gamma,\ \theta]^T$: altitude, speed, flight-path angle
(positive when climbing), and downrange angle. The model is a point mass in a vertical plane around
a spherical, non-rotating Earth, with constant mass and aerodynamic coefficients, an exponential
atmosphere, zero bank, and no thrust. With $r = R_E + h$ and $g = \mu / r^2$:

$$
\begin{aligned}
\dot{h} &= V \sin\gamma \\
\dot{V} &= -\frac{D}{m} - g \sin\gamma \\
\dot{\gamma} &= \frac{L}{mV} + \left(\frac{V}{r} - \frac{g}{V}\right)\cos\gamma \\
\dot{\theta} &= \frac{V \cos\gamma}{r}
\end{aligned}
$$

The aerodynamic forces and density are:

$$
D = \frac{1}{2}\rho V^2 C_D A, \qquad
L = \frac{1}{2}\rho V^2 C_L A, \qquad
\rho(h) = \rho_0 e^{-h/H}
$$

The initial condition is $\mathbf{s}(0) = [h_0,\ V_0,\ \gamma_0,\ 0]^T$.

A trajectory is a **skip** if it descends below $h_0$ and later climbs back through $h_0$.
The aerodynamic load, in units of standard gravity, is:

$$
n = \frac{\sqrt{L^2 + D^2}}{m\, g_0}, \qquad g_0 = 9.80665\ \mathrm{m/s^2}
$$

and the reported peak load is its maximum over the first atmospheric encounter.

Forward Euler advances the state by:

$$
\mathbf{s}_{k+1} = \mathbf{s}_k + \Delta t\, \mathbf{f}(\mathbf{s}_k)
$$

which has first-order global error in $\Delta t$.

## Conventions

SI units everywhere (m, s, kg, rad). Entry angles are negative when descending and are converted
from degrees to radians in `parameters.py`.

## Parameters

| Quantity | Value | Source |
|---|---|---|
| Entry altitude $h_0$ | 121.92 km | [cite] |
| Entry speed $V_0$ | [value] | [cite] |
| Baseline entry angle $\gamma_0$ | $-5.66^\circ$ | [cite] |
| Mass $m$ | [value] kg | [cite] |
| Reference area $A$ | [value] m² | [cite] |
| Drag coefficient $C_D$ | [value] | [cite] |
| Lift-to-drag ratio $L/D$ | 0.3 | [cite] |
| Time step $\Delta t$ | [from convergence.py] s | convergence study |

## Notes on Verification

`verification.py` compares against closed-form results rather than only checking convergence:
the exact discrete Euler map, a second from-scratch implementation of the equations, conservation
of energy and angular momentum in vacuum, the Allen–Eggers limit, and an independent RK4 solution.
If `parameters.py`, `model.py`, or `euler.py` change names or signatures, update the interface
block at the top of `verification.py`.

## Contributions

[One line per member, matching the contributions section of the report.]

## Generative AI

[Short description of how AI was used, matching the Acknowledgments section of the report.]
