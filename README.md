# Scalable Pressurized Water Reactor (PWR) Simscape Simulator

A high-fidelity, non-linear cyber-physical simulation environment of a commercial-scale Pressurized Water Reactor (PWR) nuclear power plant. Built entirely in MATLAB/Simulink using the **Simscape Two-Phase Fluid** and **Thermal Liquid** domains, this model serves as a robust environment simulator for testing advanced intelligent automation, fault diagnostics, and reinforcement learning agents.

---

## Project Overview

This repository contains a closed-loop digital twin of a nuclear power station. Unlike simplified, linearized models, this simulation captures complex, "stiff" thermodynamic transients, structural fluid dynamics, and true two-phase phase changes (boiling and condensing) across the primary and secondary loops. 

### Key Capabilities
* **Multi-Domain Physics:** Simulates a complete Rankine cycle coupled dynamically to a Point Kinetics reactor core.
* **Intelligent Systems Ready:** Designed explicitly as a high-stiffness state-space environment for benchmarking Machine Learning (ML), Reinforcement Learning (RL), and Anomaly Detection algorithms.
* **Transient Testing:** Capable of simulating severe operational transients (e.g., prompt critical excursions, Loss of Coolant Accidents, station blackouts, grid load-following surges) without computational divergence.

---

## Core Neutronics Physics (State-Space Model)

The reactor core heat source is governed by the zero-dimensional **Point Kinetics Equations** with delayed neutron precursors and localized thermal-hydraulic reactivity feedback. This creates a highly non-linear, coupled differential equation system that defines the primary state-space of the environment:

$$\frac{dn(t)}{dt} = \frac{\rho(t) - \beta}{\Lambda} n(t) + \sum_{i=1}^{G} \lambda_i C_i(t)$$

$$\frac{dC_i(t)}{dt} = \frac{\beta_i}{\Lambda} n(t) - \lambda_i C_i(t), \quad i = 1, \ldots, G$$

Where:
* $n(t)$ is the neutron density (directly proportional to reactor thermal power).
* $\rho(t)$ is the total reactivity of the core.
* $\beta$ is the total delayed neutron fraction ($\sum \beta_i$).
* $\Lambda$ is the prompt neutron lifetime.
* $C_i(t)$ is the concentration of the $i$-th group of delayed neutron precursors.
* $\lambda_i$ is the decay constant of the $i$-th precursor group.

### Reactivity Feedback Loop
The total core reactivity $\rho(t)$ is dynamically updated at every time step to simulate the inherent physical safety features of a PWR via the Moderator Temperature Coefficient (MTC):

$$\rho(t) = \rho_{ext}(t) + \alpha_T (T_c(t) - T_{c,0})$$

Where $\rho_{ext}(t)$ is the external reactivity insertion (e.g., control rod movement), $\alpha_T$ is the negative temperature feedback coefficient, and $T_c(t)$ is the real-time coolant temperature shifting away from the baseline operating temperature $T_{c,0}$.

---

## Technical Specifications & Architecture

The plant architecture is split into three main interconnected physical domains:

### 1. The Primary Loop (Thermal Liquid)
* **Reactor Core:** Driven by the point-kinetics model detailed above, transferring thermal energy directly to the primary coolant.
* **Mass Flow:** Scaled to maintain structural thermal equilibrium across the reactor pressure vessel.

### 2. The Secondary Loop (Two-Phase Fluid Rankine Cycle)
* **Steam Generator (Boiler):** Acts as the critical boundary heat exchanger, turning high-pressure subcooled liquid into 100% dry saturated steam.
* **Main Steam Line & Turbine:** High-volume piping routes steam to a multi-stage turbine block that extracts fluid enthalpy and exhausts wet steam to the condenser.
* **Condenser & Hotwell Vacuum:** Drops steam back to liquid phase under deep vacuum conditions anchored by a large thermal mass reservoir and an external cooling water loop.
* **Feedwater Pumping Station:** Features **3 Parallel Centrifugal Pumps (2P)** designed for industrial parallel pumping operations. The configuration divides total loop mass flow evenly across the three pumps pushing against the boiler operating pressure.

### 3. Mechanical & Electrical Domain
* **Gear Transmission:** Connects the main turbine drive shaft to a step-up gear system to optimize generator speed.
* **Power Generation:** Implemented via a Rotational Electromechanical Converter acting as a synchronous generator.
* **Grid Load:** Simulates an islanded grid framework via a variable passive load resistor, allowing the generator shaft speed to dynamically self-balance based on electrical demand.

---

## Intelligent Systems Applications

This simulator provides a multi-variable, highly non-linear environment ready for computer science research:

* **Reinforcement Learning (RL):** Training autonomous agents (PPO, DQN) via the MATLAB Reinforcement Learning Toolbox to manage control rod reactivity ($\rho_{ext}$) and parallel pump configuration during load-following transients.
* **Fault Detection, Isolation, and Recovery (FDIR):** Synthesizing time-series datasets under healthy and anomalous conditions (e.g., pump degradation, valve friction leaks) to train Random Forests, SVMs, or LSTMs for predictive maintenance.
* **Physics-Informed Neural Networks (PINNs):** Training deep learning surrogate models to map inputs directly to non-linear physical state outputs, bypassing heavy numerical solver steps to achieve near-instantaneous execution speeds.

---

## Simulation & Initialization Guide

Because of the high numerical stiffness involved in simulating two-phase boiling and condensing fluid boundaries concurrently, the model must be initialized carefully.

### Recommended Solver Configurations
To prevent solver timeout or computational "crawling" during violent transients, apply the following configuration parameters:
* **Solver Type:** Variable-step
* **Solver:** `ode15s (stiff/NDFs)` or `ode23t`
* **Max Step Size:** `auto`

### Achieving Steady State
1. Open the model settings and navigate to Data Import/Export.
2. Uncheck Initial state.
3. Check Save final operating point and define a workspace variable.
4. Set a long simulation duration to allow the primary fluid boundaries, neutronics, and turbine/generator inertia to naturally settle into thermodynamic equilibrium.
5. Once complete, check Initial state and point it to your saved workspace variable. You can now execute rapid transient tests directly from full power.
