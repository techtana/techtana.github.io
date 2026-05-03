---
layout: post
title: Optimizing Non-PID R2R Controllers
subtitle: How to Tune Parameters and Read Metrics
author: Tech Tana
categories: engineering
tags: controls
---
> [!Note]
> Written by ChatGPT after series of prompts

Run-to-Run (R2R) controllers in semiconductor manufacturing behave differently from classical PID loops. Instead of continuous, high-frequency control, R2R loops adjust recipes wafer-by-wafer or lot-by-lot based on metrology results. Because of this discrete nature, the parameters you tune—and the metrics you monitor—are different. This article explains how R2R controllers are tuned, what parameters matter, and how to interpret performance metrics to guide engineering decisions.

---

## 1. Tuning EWMA Feedback Controllers

In an EWMA controller, the lambda (λ) parameter determines how aggressively the controller updates its bias estimate. This makes λ the central tuning knob.

A larger λ makes the controller respond quickly to recent measurements. It is suitable for:

* processes with minimal noise,
* tools with rapid drift,
* startup or post-PM periods where the chamber is unstable.

However, high λ also makes the loop more sensitive to noise and special-cause events. If you see jumpy recipe movements or large σₘᵥ (move variance), λ is likely too high.

A smaller λ makes the controller slower and more stable. It is better for:

* high-noise processes,
* tools with stable drift patterns,
* long time-constant processes like CMP or CVD.

EWMA tuning also involves managing the bias-reset strategy. If bias estimates creep upward without returning—indicating unbounded drift—a more frequent reset may be needed to prevent the model from hiding tool deterioration.

__Example:__ In an etch chamber with periodic seasoning cycles, λ = 0.25 allows the controller to adapt to the cycle while filtering noise. After PM, temporarily raising λ to 0.5 helps stabilize the chamber faster.

---

## 2. Optimizing EWMA + Feed-Forward (FF)

When feed-forward variables are available, the engineer also tunes the FF gain, which determines how much the controller uses incoming information to compensate for predictable shifts. A strong FF gain is only useful when the FF variable has strong correlation with the output.

You also must evaluate:

* alignment between the FF variable and the response (delays matter),
* predictor quality (variance, calibration, missing data),
* outlier handling (suppress FF values that jump due to single-lot anomalies).

__Example:__ If thickness incoming metrology has a correlation R² of 0.6 with the final film thickness, an FF gain near 0.8 may significantly reduce σᵧ. But if R² is below 0.2, FF becomes noise, increasing σₘᵥ and harming stability.

---

## 3. Tuning ARX and ARIMAX Controllers

ARX/ARIMAX controllers rely on a statistical model to predict the next wafer’s output. Tuning involves selecting:

* the model order (number of past outputs included),
* lag structure for FF variables,
* prediction horizon (usually 1-step-ahead),
* regularization to avoid overfitting,
* model update frequency to handle drift.

A well-tuned ARX model improves prediction accuracy and reduces output variance. A poorly tuned one overreacts, ignores relevant FF signals, or lags behind drift.

Residual analysis is essential. If residual autocorrelation grows, or RMSE increases, the model is stale or missing variables.

__Example:__ A litho focus model might use y[k-1] and y[k-2] along with stage temperature as FF. If residuals start trending upward after a chamber dry clean, retraining the ARX coefficients restores prediction accuracy.

---

## 4. Tuning State-Space / Observer-Based Controllers

State-space controllers include two main tuning elements:

* the observer gain (L), which sets how fast the state estimate reacts to measurements,
* the feedback gain (K), which determines how strongly the controller corrects deviation.

High observer gains make the state estimate fast but noisy; low gains make it stable but sluggish. Engineers tune L based on desired responsiveness and noise level.

K determines aggressiveness. Too high makes recipe moves volatile; too low makes drift go uncorrected. State resets are sometimes required after PM or model mismatch.

__Example:__ In multi-loop thickness + profile control, tuning L and K allows partial decoupling of variables and better handling of spatial drift patterns across wafers.

---

## 5. How to Read Metrics and Make Tuning Decisions

### Output Variation (σᵧ)

If σᵧ is high, increase aggressiveness: raise λ or strengthen FF.
If σᵧ is low but σₘᵥ is exploding, the controller is too reactive.

### Move Variation (σₘᵥ)

High σₘᵥ means the recipe is jittering. Lower λ or FF gain to stabilize the loop.
A smooth σₘᵥ indicates good filtering and healthy behavior.

### Cpk

Improving Cpk means the controller is reducing variation relative to spec limits.
If Cpk stagnates while recipe jitter rises, the controller isn’t helping—intervention is needed.

### Harris Index

Harris Index is one of the best composite metrics. Higher values indicate overall better control performance.
If Harris improves after tuning, the changes are justified.

### Short-term vs Long-term Sigma

If long-term σ grows relative to short-term σ, the tool is drifting or unstable.
Adjust λ downward or trigger a bias reset.

### Residual RMSE (ARX/ARIMAX)

Increasing RMSE means the model is stale. Retrain or add better FF variables.

### Bias Estimate Creeping

Bounded creeping is normal.
Unbounded creeping indicates the tool is degrading and the controller is compensating too much.
This is a sign for PM, recalibration, or reset.

### Observer Residual

Flat residuals mean the model matches the tool.
Rising residuals indicate mismatch, causing bad predictions.

---

## Summary
Non-PID R2R tuning is about balancing responsiveness and stability across EWMA, FF, ARX, and state-space models. Lambda, FF gain, model structure, and observer gains are the key parameters, while metrics like σᵧ, σₘᵥ, Cpk, Harris, long/short-term sigma, residuals, and bias behavior guide decisions. By interpreting these metrics correctly, engineers can maintain stable processes, prevent excursions, and improve tool performance without relying on guesswork.