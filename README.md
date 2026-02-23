# PMDC Motor Speed Control — PI Controller | ngspice

State-space averaged model of a permanent magnet DC motor 
with PI speed controller, simulated in ngspice.

## Overview

Simulates closed-loop speed control of a PMDC motor using 
a PI controller. Motor dynamics are modeled as a state-space 
system directly in ngspice using B-sources and capacitors as 
state variables — no switching, no algebraic loops.

## Model

Two state variables:
- `V(nd_ia)` = armature current [A]
- `V(nd_omega)` = angular velocity [rad/s]

Differential equations:
La * dIa/dt    = Va - RaIa - Keomega
J  * dOmega/dt = KtIa - Bmomega
dIntg/dt       = Ki * (omega_ref - omega)
Va = clamp(Kp*err + Intg, 0, Vbus)
## Motor Parameters

| Parameter | Value | Unit |
|-----------|-------|------|
| Ra | 0.5 | Ω |
| La | 2 | mH |
| Ke = Kt | 0.02 | V·s/rad |
| J | 1e-4 | kg·m² |
| Bm | 1e-4 | N·m·s/rad |

## Steady-State (analytical)

- Target: ω = 314 rad/s (~3000 rpm)
- Ia_ss = 1.57 A
- Va_ss = 7.07 V
- Duty cycle = 29.4%

## Results

- Step response settles to 314 rad/s with minor overshoot
- PI gains: Kp = 0.05, Ki = 0.5
- PWM visualization (5 kHz) included as separate output layer

## Why State-Space in ngspice?

Cycle-by-cycle PWM simulation in ngspice creates severe 
numerical stiffness due to three conflicting time scales 
(tau_e = 4ms, tau_m = 1s, T_pwm = 200µs). The averaged 
state-space approach eliminates switching transients while 
preserving control dynamics — standard practice in control 
system design.

## Files

- `dcmotor_minimal.cir` — core PI + state-space model

## Tools

ngspice 40+
