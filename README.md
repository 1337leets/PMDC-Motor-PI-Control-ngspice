# PMDC Motor Speed Control — PI Controller | ngspice

State-space averaged model of a permanent magnet DC motor 
with PI speed controller, simulated in ngspice.

## Overview

Simulates closed-loop speed control of a PMDC motor using 
a PI controller. Motor dynamics are modeled as a state-space 
system directly in ngspice using B-sources and capacitors as 
state variables — no switching, no algebraic loops.

## Model

Three state variables, each stored as the voltage across a capacitor:
- `V(nd_ia)` = armature current [A]
- `V(nd_omega)` = angular velocity [rad/s]
- `V(nd_intg)` = PI integrator state [V]

Differential equations:

```
La * dIa/dt    = Va - Ra*Ia - Ke*omega
J  * dOmega/dt = Kt*Ia - Bm*omega
dIntg/dt       = Ki * (omega_ref - omega)
Va = clamp(Kp*err + Intg, 0, Vbus)
```

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
- PWM visualization (5kHz): sawtooth ramp via floor(), gate comparator confirms duty = 29.4% at steady-state

### Step Response
![Step Response](plots/step_response.png)

### PWM Visualization (steady-state, zoomed)
![PWM](plots/pwm.png)

## Why State-Space in ngspice?

Cycle-by-cycle PWM simulation in ngspice creates severe 
numerical stiffness due to three conflicting time scales 
(tau_e = 4ms, tau_m = 1s, T_pwm = 200µs). The averaged 
state-space approach eliminates switching transients while 
preserving control dynamics — standard practice in control 
system design.
PWM visualization is implemented as a separate observation layer 
(Bramp, Bgate) that reads nd_va but does not feed back into motor 
dynamics — equivalent to probing without disturbing the circuit.

## Files

- `dcmotor_minimal.cir` — core PI + state-space model
- `dcmotor_pwm.cir` — adds PWM visualization layer (5kHz sawtooth comparator)
- `archive/` — earlier development iterations, kept for reference (see below)

## Development History

The final model came out of a series of attempts, kept in `archive/`.
The first versions simulated the switching converter cycle by cycle;
the stiffness problem described above is what pushed the design toward
the averaged state-space model.

| File | Approach |
|------|----------|
| `dcmotor_pid.cir` | First switching PWM model (voltage-controlled switch + ramp reset) |
| `dcmotor_pid2.cir` | Back-calculation anti-windup and steady-state `.measure` checks |
| `dcmotor_pid3.cir` | Current sensing fix, freewheel diode, PWM reset pulse, solver options |
| `dcmotor_pid4.cir` – `dcmotor_pid6.cir` | Analytical checks of steady state, ramp amplitude, diode and sensor polarity |
| `dcmotor_pid_final.cir` | Last switching version (back-EMF sign and freewheel direction fixed) |
| `dcmotor_averaged.cir` | Switch removed: PI output drives the motor as average voltage |
| `dcmotor_statespace.cir` | Ia and omega as capacitor states, no algebraic loops |
| `dcmotor_pwm_vis.cir` | First version of the PWM observation layer |

The archived netlists are kept as they were, with only their comments
translated to English. They are not maintained.

## Tools

ngspice 40+
