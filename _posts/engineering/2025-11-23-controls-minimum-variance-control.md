---
layout: post
title: Minimum Variance Control
subtitle: Why you cannot run a real Minimum-Variance Controller (MVC) on every tool
author: Tech Tana
categories: engineering
tags: controls
---
> [!Note]
> Written by ChatGPT after series of prompts

# 1. Why you cannot run a real Minimum-Variance Controller (MVC) on every tool

A true MVC is a theoretical benchmark, not a practical controller.
There are four reasons fabs don’t implement it directly:

### (A) MVC requires perfect process models

MVC assumes you know the exact ARMA/ARX/CARMA structure, noise characteristics, and dynamics.
Real tools have:

* nonlinearities
* time-varying gain
* chamber seasoning
* input delays
* unmeasured disturbances
* drift after PM

If the model is wrong, MVC becomes unstable or overly aggressive.

---

### (B) MVC demands aggressive, noise-amplifying moves

To eliminate all predictable structure, MVC often produces control moves that:

* overshoot intentionally
* invert dynamics
* amplify sensor noise
* make huge jumps in recipe settings

This is unacceptable for wafer safety, tool health, and hardware limits.

---

### (C) MVC conflicts with tool and recipe constraints

MVC assumes unbounded control authority.

But semiconductor tools have:

* recipe limits
* slew limits (rate constraints)
* temperature/ramp limits
* chamber damage risks
* actuator quantization

MVC does not honor any of these.

---

### (D) MVC is not robust to unmodeled disturbance

MVC is extremely sensitive to model mismatch.
One wrong ARMA coefficient → unstable, oscillatory, or divergent behavior.

Industry prefers controllers that trade optimality for robustness, like:

* EWMA
* PID
* QR MPC
* State-space MPC
* Hybrid FF/FB

Thus, you cannot run a true MVC because it is unsafe, unrealistically aggressive, and assumes perfect knowledge of the process.

---

# 2. If I use QR-MPC as feedback, is every ARIMA component controllable?

No. Not even QR-MPC can control all ARIMA components.

Here’s why:

---

## (A) ARIMA = predictable (AR/MA) + unpredictable (innovation)

ARIMA decomposes the signal into:

1. Deterministic/predictable part
   (autoregressive trends, moving-average structure, slow drift)
2. Innovation noise
   (pure randomness that no controller can cancel)

QR-MPC can cancel (1), but not (2).

In notation:  
$$y_k = \underbrace{\hat{y}_k}_{\text{predictable}} + \underbrace{e_k}_{\text{innovation noise}}$$

QR-MPC can act on $\hat{y}_k$.
Nothing in the world can act on $e_k$.

---

## (B) MPC still cannot “control” components that violate physics/constraints

Even predictable ARIMA components may be uncontrollable if:

* The dynamics are non-invertible
* Tool doesn’t have enough actuator authority
* You hit recipe/temperature/thickness limits
* Response time is slower than incoming disturbances
* There are delays longer than your prediction window
* Inputs must stay inside strict operational envelopes

Thus, some ARIMA structure may be *partially* controllable, not fully.

---

## (C) MPC cannot control upstream-driven variation that arrives too fast

If an upstream tool introduces a variation pattern that bypasses the ability of the downstream controller to react (e.g., short-run wafer-to-wafer noise), MPC cannot eliminate it.

Predictable ≠ controllable
Controllable ≠ allowable

---

## (D) MPC does not magically eliminate innovation noise

Every ARIMA model ends with an innovation term:

$$e_k \sim \text{white noise}$$

This term cannot be removed by *any* controller:

* MPC
* PID
* EWMA
* State-space
* LQR
* Kalman + control
* Even MVC itself (innovation is the noise floor)

This innovation variance is exactly what becomes $σ_{mv^2}$ in Harris Index.

---

# 3. Summary

### Why you can’t run MVC everywhere:

* Requires perfect models
* Extremely aggressive moves
* Ignores constraints
* Not robust
* Unsafe for real manufacturing tools

### With QR-MPC:

* Controllable: predictable ARIMA components that obey tool dynamics & constraints
* Not controllable:

  * innovation noise
  * upstream noise faster than tool response
  * components violating actuator limits or physical constraints
  * unmodeled dynamics

MPC reduces variation but never reaches ARIMA’s innovation floor. Only MVC reaches that theoretical limit — and only in theory.