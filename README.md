# AE-370_Proj1
Atmosphere spacecraft skipping 

\documentclass[11pt]{article}
\usepackage[margin=1in]{geometry}
\usepackage{amsmath}
\usepackage{booktabs}
\usepackage{array}
\usepackage{listings}
\usepackage[hidelinks]{hyperref}

\lstset{
  basicstyle=\ttfamily\small,
  frame=single,
  breaklines=true,
  columns=fullflexible
}

\title{Atmospheric Skip Entry of an Artemis~I-Inspired Capsule\\[2pt]
\large AE 370 Project 1, Fall 2026 --- Code README}
\date{}

\begin{document}
\maketitle

\section{Overview}
This repository integrates a point-mass entry model with forward Euler to find which entry angles
produce an atmospheric skip, and what peak aerodynamic load each produces, as a function of entry speed.

\section{Requirements}
Python \textbf{[3.x]} with \texttt{numpy} and \texttt{matplotlib} (add \texttt{scipy} if any file uses it).
\begin{lstlisting}[language=bash]
pip install numpy matplotlib
\end{lstlisting}
No ODE library advances the solution. Time stepping is our own forward Euler in \texttt{euler.py}.
\texttt{convergence.py} also contains an RK4 step, used only as an accuracy reference for error studies
and never in the study itself.

\section{Repository Layout}
All files must be in the same folder, and every script should be run from that folder.

\begin{center}
\begin{tabular}{@{}ll@{}}
\toprule
File & Purpose \\
\midrule
\texttt{parameters.py}   & Constants, vehicle values, initial conditions, event thresholds \\
\texttt{model.py}        & Density, aerodynamic forces, load factor, state derivatives \\
\texttt{euler.py}        & Forward Euler time marching \\
\texttt{baseline.py}     & Runs one case and saves diagnostics \\
\texttt{verification.py} & Independent checks of the equations and Euler code \\
\texttt{convergence.py}  & Step refinement, error calculations, recommended $\Delta t$ \\
\texttt{sweep.py}        & Entry-angle and entry-speed study, boundary refinement \\
\texttt{plots.py}        & Generates report figures from saved results \\
\bottomrule
\end{tabular}
\end{center}

\section{Reproducing the Results}
Run the scripts in this order:
\begin{lstlisting}[language=bash]
python verification.py   # about 30 s; prints PASS/FAIL for each check
python convergence.py    # a few minutes; add --quick for a faster, coarser run
python baseline.py       # [what it saves, where]
python sweep.py          # [runtime, what it saves, where]
python plots.py          # [which figures it writes, where]
\end{lstlisting}
\texttt{verification.py} should report that all checks passed before any results are trusted.
\texttt{convergence.py} writes CSV files and \texttt{fig\_convergence.png} to \texttt{results/}
and prints the time step used in the study.

\section{Model Summary}
The state is $\mathbf{s} = [h,\, V,\, \gamma,\, \theta]^{T}$: altitude, speed, flight-path angle
(positive when climbing), and downrange angle. The model is a point mass in a vertical plane around a
spherical, non-rotating Earth, with constant mass and aerodynamic coefficients, an exponential
atmosphere, zero bank, and no thrust. With $r = R_E + h$ and $g = \mu / r^{2}$,
\begin{align}
  \dot{h}      &= V \sin\gamma, \\
  \dot{V}      &= -\frac{D}{m} - g \sin\gamma, \\
  \dot{\gamma} &= \frac{L}{mV} + \left(\frac{V}{r} - \frac{g}{V}\right)\cos\gamma, \\
  \dot{\theta} &= \frac{V \cos\gamma}{r},
\end{align}
with aerodynamic forces and density
\begin{equation}
  D = \tfrac{1}{2}\rho V^{2} C_D A, \qquad
  L = \tfrac{1}{2}\rho V^{2} C_L A, \qquad
  \rho(h) = \rho_0 e^{-h/H}.
\end{equation}
The initial condition is $\mathbf{s}(0) = [h_0,\, V_0,\, \gamma_0,\, 0]^{T}$.

A trajectory is a \emph{skip} if it descends below $h_0$ and later climbs back through $h_0$.
The aerodynamic load, in units of standard gravity, is
\begin{equation}
  n = \frac{\sqrt{L^{2} + D^{2}}}{m\, g_0}, \qquad g_0 = 9.80665~\mathrm{m/s^2},
\end{equation}
and the reported peak load is its maximum over the first atmospheric encounter.

Forward Euler advances the state by
\begin{equation}
  \mathbf{s}_{k+1} = \mathbf{s}_k + \Delta t\, \mathbf{f}(\mathbf{s}_k),
\end{equation}
which has first-order global error in $\Delta t$.

\section{Conventions}
SI units everywhere (m, s, kg, rad). Entry angles are negative when descending, and are converted
from degrees to radians in \texttt{parameters.py}.

\section{Parameters}
\begin{center}
\begin{tabular}{@{}lll@{}}
\toprule
Quantity & Value & Source \\
\midrule
Entry altitude $h_0$                 & 121.92 km                       & \textbf{[cite]} \\
Entry speed $V_0$                    & \textbf{[value]}                & \textbf{[cite]} \\
Baseline entry angle $\gamma_0$      & $-5.66^{\circ}$                 & \textbf{[cite]} \\
Mass $m$                             & \textbf{[value]} kg             & \textbf{[cite]} \\
Reference area $A$                   & \textbf{[value]} m$^{2}$        & \textbf{[cite]} \\
Drag coefficient $C_D$               & \textbf{[value]}                & \textbf{[cite]} \\
Lift-to-drag ratio $L/D$             & 0.3                             & \textbf{[cite]} \\
Time step $\Delta t$                 & \textbf{[from convergence.py]} s & convergence study \\
\bottomrule
\end{tabular}
\end{center}

\section{Notes on Verification}
\texttt{verification.py} compares against closed-form results rather than only checking convergence:
the exact discrete Euler map, a second from-scratch implementation of the equations, conservation of
energy and angular momentum in vacuum, the Allen--Eggers limit, and an independent RK4 solution.
If \texttt{parameters.py}, \texttt{model.py}, or \texttt{euler.py} change names or signatures, update the
interface block at the top of \texttt{verification.py}.

\section{Contributions}
\textbf{[One line per member, matching the contributions section of the report.]}

\section{Generative AI}
\textbf{[Short description of how AI was used, matching the Acknowledgments section of the report.]}

\end{document}
