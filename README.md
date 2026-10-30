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

No ODE library advances the solution. Time stepping is our own forward Euler in `euler.ipynb`.
`convergence.ipynb` also contains an RK4 step, used only as an accuracy reference for error studies
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
python verification.ipynb   # about 30 s; prints PASS/FAIL for each check
python convergence.ipynb    # a few minutes; add --quick for a faster, coarser run
python baseline.ipynb       # [what it saves, where]
python sweep.ipynb          # [runtime, what it saves, where]
python plots.ipynb          # [which figures it writes, where]
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
from degrees to radians in `parameters.ipynp`.

## Parameters

| Quantity | Value | Source |
|---|---|---|
| Entry altitude $h_0$ | 121.92 km | FAA Returning form Space Guidance |
| Entry speed $V_0$ | varies | Orion Artemis I Entry Performance |
| Baseline entry angle $\gamma_0$ | $-6$ | Orion Artemis I Entry Performance |
| Mass $m$ | 9300 kg | Orion Reference Guide |
| Reference area $A$ | 10 m² | Orion Reference Guide |
| Drag coefficient $C_D$ | 0.8 | Allen and Eggers; Orion Entry Studies |
| Lift-to-drag ratio $L/D$ | 0.3 | Orion skip-entry studies |
| Time step $\Delta t$ | 0.1 s | convergence study |

## Notes on Verification

`verification.ipynb` compares against closed-form results rather than only checking convergence:
the exact discrete Euler map, a second from-scratch implementation of the equations, conservation
of energy and angular momentum in vacuum, the Allen–Eggers limit, and an independent RK4 solution.
If `parameters.ipynb`, `model.ipynb`, or `euler.ipynb` change names or signatures, update the interface
block at the top of `verification.ipynb`.

##Acknowledgements

This project was inspired by the atmospheric skip-entry technique demonstrated by NASA's Orion spacecraft during the Artemis I mission. Vehicle properties, entry conditions, and performance characteristics were obtained from NASA Orion mission documentation, Orion reference materials, Orion entry-performance analyses, historical atmospheric-entry literature, and FAA reentry guidance documents.

Several modeling assumptions were made to keep the problem tractable for a first numerical investigation. The Earth was modeled as spherical and non-rotating, the atmosphere was represented using an exponential density model, vehicle mass and aerodynamic coefficients were assumed constant throughout entry, bank angle effects were neglected, and thrust was assumed to be zero during atmospheric flight. These assumptions allow the governing equations to capture the primary dynamics of skip entry while remaining suitable for analysis with a Forward Euler time-marching method.

The authors acknowledge the use of generative artificial intelligence tools during this project. AI was used primarily during the brainstorming stage to explore possible project ideas and identify suitable aerospace dynamical systems for study. AI tools were also used to assist in writing code implementations, debugging portions of the numerical model, and improving document formatting and presentation. All final engineering decisions, model development, verification procedures, analysis, and interpretation of results were reviewed and validated by the project team.

## Contributions

Anjana Yaramaka: 

Charlotte Damewood: 

Blanca Albuixech-Robinson: Completed the verification and convergence studies, including implementation of the numerical accuracy and validation tests used to confirm the correctness of the Forward Euler method. Participated in group discussions that defined the project premise and study objectives. Prepared the repository README and assisted with report formatting and organization. 
Estimated contribution: 20–25%.

Teagan Arndt: 


## Generative AI

Generative AI tools were used throughout the project to support brainstorming, software development, and documentation. AI was initially used to help generate and evaluate potential project ideas before the team selected an atmospheric skip-entry study inspired by the Artemis I mission. AI was also used to assist with writing and debugging code for the numerical simulations, generating portions of the repository documentation, and improving report formatting and presentation. All generated content, code, analyses, and conclusions were reviewed, modified where necessary, and validated by the project team prior to inclusion in the final submission.
