# neutron-transport-mc
Monte Carlo k-eigenvalue neutron transport solver using power iteration (MCMC)


## Project Scope — Monte Carlo Neutron Transport (k-Eigenvalue)

- **Problem Definition**
  - Solve the steady-state neutron transport $k$-eigenvalue problem via fission-source power iteration (MCMC).
  - Governing equation (steady-state Boltzmann transport equation):
    $$
    \Omega \cdot \nabla \psi(\mathbf{r}, \Omega, E)
    + \Sigma_t \psi
    = \int \Sigma_s \psi \, d\Omega' dE'
    + \frac{\chi(E)}{k} \int \nu \Sigma_f \psi \, d\Omega' dE'
    $$
  - Variables:
    - $\mathbf{r}$: spatial position
    - $\Omega$: neutron direction (unit vector)
    - $E$: neutron energy
    - $\psi$: angular neutron flux
    - $\Sigma_t$: total macroscopic cross section
    - $\Sigma_s$: scattering macroscopic cross section
    - $\Sigma_f$: fission macroscopic cross section
    - $\nu$: average neutrons produced per fission
    - $\chi$: fission neutron energy spectrum
    - $k$: effective multiplication factor

- **Geometry & Boundary Conditions**
  - Spatial domain $\mathbf{r} \in V$
  - Vacuum boundary: neutrons exiting $V$ are lost
  - Optional reflective boundary: $\Omega \to \Omega_\text{reflected}$

- **Material Model (One-Group Approximation)**
  - Energy-independent cross sections
  - Cross-section decomposition:
    $$
    \Sigma_t = \Sigma_a + \Sigma_s + \Sigma_f
    $$
  - Variables:
    - $\Sigma_a$: absorption macroscopic cross section
    - $\Sigma_s$: scattering macroscopic cross section
    - $\Sigma_f$: fission macroscopic cross section

- **Neutron State Representation**
  - $\mathbf{r}$: position
  - $\Omega$: direction
  - $E$: energy group index (single group)
  - $w$: statistical weight (optional)
  - alive/dead flag

- **Initial Fission Source**
  - Source particle count: $N$
  - Spatial source density: $S(\mathbf{r})$
  - Isotropic angular sampling:
    $$
    \mu = \cos\theta = 2\xi_1 - 1, \qquad
    \phi = 2\pi\xi_2
    $$
  - Variables:
    - $\theta$: polar angle
    - $\phi$: azimuthal angle
    - $\xi_1, \xi_2 \sim \mathcal{U}(0,1)$: uniform random variables

- **Free-Flight Sampling**
  - Sample distance to next interaction:
    $$
    \ell = -\frac{\ln \xi}{\Sigma_t}
    $$
  - Variables:
    - $\ell$: neutron flight distance
    - $\xi \sim \mathcal{U}(0,1)$
    - $\Sigma_t$: total macroscopic cross section
  - Distribution:
    - Exponential distribution arising from a Poisson collision process

- **Collision Site Determination**
  - New position:
    $$
    \mathbf{r}_{\text{new}} = \mathbf{r}_{\text{old}} + \ell\,\Omega
    $$
  - Check for boundary crossing (leakage)

- **Interaction Type Sampling**
  - Probability of reaction $i \in \{a, s, f\}$:
    $$
    P_i = \frac{\Sigma_i}{\Sigma_t}
    $$
  - Variables:
    - $\Sigma_i$: macroscopic cross section for reaction $i$

- **Scattering Physics**
  - Isotropic scattering:
    $$
    \mu' = 2\xi_1 - 1, \qquad \phi' = 2\pi\xi_2
    $$
  - Variables:
    - $\mu' = \cos\theta'$: cosine of scattering angle
    - $\phi'$: azimuthal scattering angle

- **Fission Physics**
  - Expected neutrons produced per collision:
    $$
    \bar{n} = \nu
    $$
  - Optional integer sampling:
    $$
    n \sim \text{Poisson}(\nu)
    $$
  - Variables:
    - $n$: number of fission neutrons generated
    - $\nu$: mean neutrons per fission

- **Fission Bank**
  - Collection of fission sites:
    $$
    \{\mathbf{r}_j\}_{j=1}^{N_f}
    $$
  - Variables:
    - $N_f$: total fission sites recorded in a cycle

- **Source Normalization**
  - Renormalize fission bank to fixed source size $N$:
    $$
    S_{n+1}(\mathbf{r}) = \frac{N}{N_f}\,S_f(\mathbf{r})
    $$
  - Variables:
    - $S_f$: raw fission source distribution
    - $S_{n+1}$: normalized source for next cycle

- **Power Iteration Loop**
  - Iterate cycles $n = 1,2,\dots$
  - Discard first $N_\text{inactive}$ cycles
  - Accumulate statistics over active cycles

- **k-Effective Estimation**
  - Per-cycle estimator:
    $$
    k^{(n)} = \frac{N_f}{N}
    $$
  - Variables:
    - $k^{(n)}$: cycle-wise estimate of $k_\text{eff}$
    - $N$: source neutrons per cycle
    - $N_f$: fission neutrons produced
  - Final estimate:
    $$
    \langle k \rangle = \frac{1}{N_\text{active}} \sum k^{(n)}
    $$

- **Flux Estimation (Track-Length Estimator)**
  - Scalar flux in volume $V$:
    $$
    \phi = \frac{1}{V} \sum_{i} \ell_i
    $$
  - Variables:
    - $\ell_i$: path length of neutron $i$ within $V$

- **Random Number Generation**
  - $\xi \sim \mathcal{U}(0,1)$ for all stochastic sampling
  - Fixed seed for reproducibility

- **Diagnostics & Outputs**
  - $k_\text{eff}$ mean and standard deviation
  - Leakage fraction:
    $$
    f_\text{leak} = \frac{N_\text{leak}}{N}
    $$
  - Spatial fission source and flux tallies

- **Software Architecture (C++)**
  - Core objects: `Geometry`, `Material`, `Neutron`, `Transport`, `RNG`, `FissionBank`
  - Deterministic builds and reproducible runs
