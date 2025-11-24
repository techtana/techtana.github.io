---
layout: post
title: Handling Inaccurate Process Models
subtitle: How to know if control's process model is not accurate? Can we estimate new model based on historical data?
author: Tech Tana
categories: engineering
tags: controls
---
> [!Note]
> Written by ChatGPT after series of prompts

Here’s a compact markdown “mini-article” with explanation, bullets, and an example walk-through.

---

# Detecting Model Mismatch and Re-Identifying Process Models (EWMA & State-Space MPC)

In any model-based control (EWMA+FF, ARX/ARIMAX, state-space MPC, etc.), the controller assumes:

> “Given this input, the process will respond like this.”

When reality drifts away from that assumption, you get **model mismatch**. If it’s small, the controller copes. If it’s large, you get bad moves, sluggish response, or instability.

Below:

1. How to detect model mismatch in **EWMA-based** and **State-Space/MPC** controllers
2. How to **estimate a new model** from historical data
3. A **worked example** for identification

---

## 1. EWMA-Based Control – How to Detect Model Mismatch

For pure EWMA feedback with no FF, the “model” is very simple:

> The process will keep drifting slowly and the EWMA bias will track it.

Model mismatch shows up more clearly when you have **FF models, ARX, or predictors** on top of EWMA.

### 1.1 Signals that the EWMA/FF model is wrong

Key indicators:

* **Prediction residuals are biased**
  Residual = measured – predicted

  * Mean(residual) ≠ 0 → model systematically off (wrong gain, offset, or drift).

* **Residual variance grows over time**

  * Increasing spread of residuals → noise or dynamics not captured by the model.

* **Residuals are autocorrelated**

  * Patterns like residual[k] tends to have the same sign as residual[k–1].
  * Means the model is missing memory (lags) or missing FF variables.

* **Bias estimate creeping (unbounded)**

  * EWMA bias keeps going up/down in one direction, never stabilizing.
  * Controller is compensating for a systematic unmodeled drift.

* **Long-term sigma increases while controller is supposedly “on”**

  * Short-term noise looks okay, but long-term variation grows → model not capturing slow changes.

### 1.2 Very practical rule of thumb

If you see **any two** of these at once:

* Biased residual mean
* Autocorrelated residuals
* Unbounded bias creeping

…then assume **your model is stale** and plan to re-identify it.

---

## 2. State-Space / MPC – How to Detect Model Mismatch

In state-space MPC (or observer-based control), the model is:

[
x[k+1] = A x[k] + B u[k] \
y[k] = C x[k]
]

You typically run an **observer** (e.g. Luenberger or Kalman filter) that generates predictions. The mismatch shows up in the **innovation / residual**:

[
r[k] = y[k] - \hat{y}[k]
]

### 2.1 Signals of mismatch

* **Non-zero mean residual**

  * Residuals consistently positive or negative → wrong offset or bias in model.

* **Increasing residual variance**

  * The model no longer explains the spread of data (e.g. new noise sources, tool change).

* **Residual autocorrelation**

  * Residuals show trends or oscillations → wrong dynamics (A/B matrices), missing states, or unmodeled delays.

* **Observer gain “fights” more and more**

  * State estimate corrections become large and frequent.

* **Controller moves get more aggressive to maintain spec**

  * MPC keeps changing u more wildly, but y is not much better → model not predictive anymore.

### 2.2 Reading this operationally

If you see residual trending, plus moves getting larger for the same quality, your **MPC’s internal model is out of date** (tool PM, hardware changes, recipe changes, etc.). Time to re-identify.

---

## 3. Re-Estimating a Model from Historical Data

High-level procedure (for both EWMA+FF/ARX and state-space):

1. **Select a time window**

   * Use data from a period when the tool behavior was “representative” (after PM settled, before major drift or incidents).
   * Include variation in both **inputs (u)** and **outputs (y)**.

2. **Clean the data**

   * Remove obvious bad lots / outliers (sensor glitches, aborted lots).
   * Align timestamps so each y[k] corresponds to the correct u[k–d] (handle delay).

3. **Choose a model structure**

   * EWMA+FF → usually an **ARX** model is enough:
     [
     y[k] = a_1 y[k-1] + \dots + b_1 u[k-1] + \dots + c
     ]
   * State-space → a **discrete-time first-order** or low-order MIMO model:
     [
     x[k+1] = A x[k] + B u[k]; \quad y[k] = C x[k]
     ]

4. **Form regression equations**

   * Stack data and solve a least-squares problem to get A, B, etc.

5. **Validate the new model**

   * Check residual mean ≈ 0, no strong autocorrelation, and smaller RMSE than the old model.
   * Simulate replay on historical data and see if predictions track reality.

---

## 4. Example Walkthrough: Simple ARX / State-Space Identification

Let’s do a small noise-free example to keep the math readable.

### 4.1 True (unknown) process

Assume the real process is:

[
y[k] = 0.8,y[k-1] + 0.5,u[k-1]
]

We don’t know 0.8 and 0.5. We want to estimate them from historical (u, y) data.

Suppose we ran the tool with:

* Initial condition: y[0] = 0
* Inputs: u = [0, 1, 1, 1, 1]

Compute the outputs (this is what the factory logs would show):

* k = 1: y[1] = 0.8·0 + 0.5·0 = 0
* k = 2: y[2] = 0.8·0 + 0.5·1 = 0.5
* k = 3: y[3] = 0.8·0.5 + 0.5·1 = 0.9
* k = 4: y[4] = 0.8·0.9 + 0.5·1 = 1.22
* k = 5: y[5] = 0.8·1.22 + 0.5·1 ≈ 1.476

Now you have historical data:

* (y[1], u[0]), (y[2], u[1]), … etc.

### 4.2 Build the ARX regression

We use the ARX form:

[
y[k] \approx a,y[k-1] + b,u[k-1]
]

We’ll use samples k = 2, 3, 4, 5.

Write it as:

* y[2] ≈ a·y[1] + b·u[1]
* y[3] ≈ a·y[2] + b·u[2]
* y[4] ≈ a·y[3] + b·u[3]
* y[5] ≈ a·y[4] + b·u[4]

Plug in numbers:

* y[2] = 0.5,  y[1] = 0,    u[1] = 1  →  0.5 ≈ a·0    + b·1
* y[3] = 0.9,  y[2] = 0.5,  u[2] = 1  →  0.9 ≈ a·0.5  + b·1
* y[4] = 1.22, y[3] = 0.9,  u[3] = 1  →  1.22 ≈ a·0.9 + b·1
* y[5] = 1.476,y[4] = 1.22, u[4] = 1  →  1.476 ≈ a·1.22 + b·1

In matrix form:

$$
\begin{bmatrix}
0.5\
0.9\
1.22\
1.476
\end{bmatrix}
\approx
\begin{bmatrix}
0 & 1\
0.5 & 1\
0.9 & 1\
1.22 & 1
\end{bmatrix}
\begin{bmatrix}
a\
b
\end{bmatrix}
$$

Call the left side Y, the matrix Φ, and θ = [a b]ᵀ. Then:

$$
\theta \approx (\Phi^\top \Phi)^{-1} \Phi^\top Y
$$

Because this example is noise-free and consistent with the assumed model form, you’ll recover:

* a ≈ 0.8
* b ≈ 0.5

That’s your **identified ARX process model**.

### 4.3 Turn ARX into state-space (simple case)

For a single-output, single-state system, we can set:

$x[k] = y[k]$  
$x[k+1] = a,x[k] + b,u[k]$  
$y[k] = x[k]$

So:

* A = [a]
* B = [b]
* C = [1]

Using the identified parameters:

* A = [0.8]
* B = [0.5]
* C = [1.0]

This becomes your **new process model** for state-space control or MPC.

### 4.4 How you’d use this in practice

1. Use historical (u, y) from the live tool over a chosen window.

2. Fit ARX parameters (a, b, etc.) via least squares.

3. Construct A, B, C matrices for state-space.

4. Plug this updated model into:

   * your **EWMA+FF**, **ARX-based R2R**, or
   * your **MPC** design.

5. Validate:

   * Recompute residuals on the same historical data.
   * Confirm mean residual ≈ 0, reduced RMSE, and no strong autocorrelation.

If that checks out, deploy the new model (carefully, with monitoring).

---

# 2×2 MIMO Model Identification for R2R / State-Space MPC

In many real R2R and MPC applications, you don’t just control one variable with one knob. You often have **two outputs** (e.g. thickness and profile) and **two inputs** (e.g. power and time, or two different actuators). That’s a 2×2 MIMO system.

Below is a **much more concise** version that still captures the essentials of detecting model mismatch in a 2×2 system using residual charts.

---

# Detecting 2×2 Model Mismatch Using Residual Charts (Concise Guide)

In a 2×2 R2R or MPC model, predictions are:

[
\hat{\mathbf{y}}[k] = \hat{A},\mathbf{y}[k-1] + \hat{B},\mathbf{u}[k-1]
]

Residuals are:

[
\mathbf{r}[k] = \mathbf{y}[k] - \hat{\mathbf{y}}[k]
]

where
(\mathbf{r}[k] = \begin{bmatrix} r_1[k] \ r_2[k] \end{bmatrix}).

You analyze **r₁[k]** and **r₂[k]** separately but also compare them together.

---

## 1. Charts to Use

For each residual series (r₁ and r₂):

* **Residual vs. wafer index** – reveals drift or bias.
* **Histogram** – should be centered at zero; offset means bias.
* **Autocorrelation** – indicates missing dynamics, wrong A matrix.
* **Residual vs input (u₁, u₂)** – patterns indicate wrong B gains.
* **Cross-correlation (r₁ vs r₂)** – indicates missing coupling terms between outputs.

---

## 2. What Patterns Mean

### **Non-zero mean residuals**

* Model systematically over/under predicts.
* Usually wrong gain, offset, or post-PM shift.

### **Trending residuals**

* Residual rising or falling over many wafers.
* A matrix no longer matches real process drift.

### **Autocorrelation**

* Residuals “remember” previous values.
* Missing lag, wrong time constant, or unmodeled delay.

### **Residual correlated with inputs**

* Scatter plot shows slope vs u₁ or u₂.
* B matrix wrong or actuator response changed.

### **Structured r₁–r₂ correlation**

* Both residuals moving together.
* Missing cross-coupling terms (a₁₂, a₂₁, b₁₂, b₂₁).

---

## 3. Tiny Example

Suppose your controller uses:

[
\hat{A} =
\begin{bmatrix}
0.7 & 0.1\
0.2 & 0.8
\end{bmatrix}
]

but after a hardware change the **real** tool behaves like:

[
A_{\text{true}} =
\begin{bmatrix}
0.9 & 0.1\
0.2 & 0.7
\end{bmatrix}
]

If you compute residuals:

* r₁ becomes **mostly positive and growing** → model underpredicts y₁.
* r₁ shows **autocorrelation** → wrong time constant (0.7 vs 0.9).
* r₂ slightly drifts due to cross-coupling mismatch.

A residual-vs-time chart would show r₁ trending upward;
ACF plot would show strong lag-1 correlation.

---

## 4. Decision Rule

Re-identify the model if you see **any two** of:

* Residual mean ≠ 0
* Trend in residuals across wafers
* Strong autocorrelation
* Correlation between residuals and inputs
* Structured cross-correlation between r₁ and r₂

These signals mean your 2×2 process model no longer represents the tool and should be updated from recent data.


---

We want to build a simple **discrete-time linear model** from historical data so that our controller (EWMA+FF, ARX, or MPC) can predict how changing both inputs will affect both outputs.

## 1. The 2×2 Model Structure

We use a first-order, linear, discrete model with one-step lag:

$$
\begin{aligned}
y_1[k] &= a_{11}y_1[k-1] + a_{12}y_2[k-1] + b_{11}u_1[k-1] + b_{12}u_2[k-1] \\
y_2[k] &= a_{21}y_1[k-1] + a_{22}y_2[k-1] + b_{21}u_1[k-1] + b_{22}u_2[k-1]
\end{aligned}
$$

Here:

* $y_1$: first output (e.g. thickness)
* $y_2$: second output (e.g. profile or uniformity)
* $u_1$: first input (e.g. power)
* $u_2$: second input (e.g. time)

We can write this in matrix form as:

$$
\mathbf{y}[k] =
\underbrace{\begin{bmatrix}
a_{11} & a_{12} \
a_{21} & a_{22}
\end{bmatrix}}_{A}
\mathbf{y}[k-1]
+
\underbrace{\begin{bmatrix}
b*{11} & b_{12} \
b_{21} & b_{22}
\end{bmatrix}}_{B}
\mathbf{u}[k-1]
$$

where  
$\mathbf{y}[k] = \begin{bmatrix}y_1[k]\ y_2[k] \end{bmatrix}$  
$\mathbf{u}[k] = \begin{bmatrix}u_1[k]\ u_2[k] \end{bmatrix}$

This **A, B** pair is exactly what a state-space MPC or MIMO R2R controller wants.

---

## 2. A Concrete Example (True System)

Let’s pick a simple, “true” underlying process (unknown to us in practice):

$$
A =
\begin{bmatrix}
0.7 & 0.1 \
0.2 & 0.8
\end{bmatrix},
\quad
B =
\begin{bmatrix}
0.5 & 0.1 \
0.0 & 0.3
\end{bmatrix}
$$

So the real process is:

$$\begin{aligned}
y_1[k] &= 0.7y_1[k-1] + 0.1y_2[k-1] + 0.5u_1[k-1] + 0.1u_2[k-1] \\
y_2[k] &= 0.2y_1[k-1] + 0.8y_2[k-1] + 0.0u_1[k-1] + 0.3u_2[k-1]
\end{aligned}
$$

We’ll assume:

* Initial outputs: $y_1[0] = 0$, $y_2[0] = 0$
* A few input moves over time:
  (this is like real recipe changes from your historical log)

| k | u₁[k] | u₂[k] |
| - | ----- | ----- |
| 0 | 0.0   | 0.0   |
| 1 | 1.0   | 0.0   |
| 2 | 0.0   | 1.0   |
| 3 | 1.0   | 1.0   |
| 4 | 0.5   | 0.2   |
| 5 | 0.0   | 1.0   |
| 6 | 1.0   | 0.5   |

Using the true A and B, we can compute the outputs step by step (this is what your historian would record):

* At k = 1, using u[0] = (0, 0):  
  y[1] = A·y[0] + B·u[0] = 0 → (0, 0)

* At k = 2, using u[1] = (1, 0):  
  (y_1[2] = 0.7·0 + 0.1·0 + 0.5·1 + 0.1·0 = 0.5)  
  (y_2[2] = 0.2·0 + 0.8·0 + 0.0·1 + 0.3·0 = 0)

* At k = 3, using u[2] = (0, 1):  
  (y_1[3] = 0.7·0.5 + 0.1·0 + 0.5·0 + 0.1·1 = 0.35 + 0 + 0 + 0.1 = 0.45)  
  (y_2[3] = 0.2·0.5 + 0.8·0 + 0.0·0 + 0.3·1 = 0.1 + 0 + 0 + 0.3 = 0.4)

Continuing this way, you get a time series of y₁[k], y₂[k] and u₁[k], u₂[k].
In reality, you don’t know A and B — you only have the data.

---

## 3. Building the Regression for Identification

We want to estimate the 8 unknowns:

$$\theta =
\begin{bmatrix}
a_{11} & a_{12} & b_{11} & b_{12} & a_{21} & a_{22} & b_{21} & b_{22}
\end{bmatrix}^\top$$

For each k ≥ 1, we write both output equations in the form:

$$\begin{aligned}
y_1[k] &\approx a_{11} y_1[k-1] + a_{12} y_2[k-1] + b_{11} u_1[k-1] + b_{12} u_2[k-1] \\
y_2[k] &\approx a_{21} y_1[k-1] + a_{22} y_2[k-1] + b_{21} u_1[k-1] + b_{22} u_2[k-1]
\end{aligned}$$

Each time step k gives **two equations**. With k = 1…6, we get 6 × 2 = 12 equations, enough to solve for the 8 parameters via least squares.

We can write all equations as:

$$\mathbf{Y} \approx \Phi , \theta$$

where:

* **Y** is a stacked vector of outputs
  ([y_1[1], y_2[1], y_1[2], y_2[2], \dots ]^\top)
* **Φ** is a matrix built from past y and u values

For example, at time k:

* Row for y₁[k]:
  ([y_1[k-1],; y_2[k-1],; u_1[k-1],; u_2[k-1],; 0,; 0,; 0,; 0])

* Row for y₂[k]:
  ([0,; 0,; 0,; 0,; y_1[k-1],; y_2[k-1],; u_1[k-1],; u_2[k-1]])

Stack these rows for k = 1…6 → that’s your Φ.

Then you compute:

$$\hat{\theta} = (\Phi^\top \Phi)^{-1} \Phi^\top \mathbf{Y}$$

That gives estimates:
* (\hat{a}*{11}, \hat{a}*{12}, \hat{b}*{11}, \hat{b}*{12})
* (\hat{a}*{21}, \hat{a}*{22}, \hat{b}*{21}, \hat{b}*{22})

In a clean (noise-free) example like this, you recover the original A and B very closely:

$$\hat{A} \approx
\begin{bmatrix}
0.7 & 0.1 \
0.2 & 0.8
\end{bmatrix},
\quad
\hat{B} \approx
\begin{bmatrix}
0.5 & 0.1 \
0.0 & 0.3
\end{bmatrix}$$

In real data with noise, they’ll be close but not exact.

---

## 4. Turning the Identified Model into State-Space / MPC Form

Once you have A and B for this 2×2 system, you can directly use:

* State: (x[k] = y[k]) (for a first-order model)
* State update: (x[k+1] = A x[k] + B u[k])
* Output: (y[k] = I x[k]), with (C = I)

So:

$$x[k+1] =
\begin{bmatrix}
0.7 & 0.1 \
0.2 & 0.8
\end{bmatrix}
x[k]
+
\begin{bmatrix}
0.5 & 0.1 \
0.0 & 0.3
\end{bmatrix}
u[k]
\\
\quad
y[k] =
\begin{bmatrix}
1 & 0 \
0 & 1
\end{bmatrix}
x[k]$$

This is a **2×2 MIMO state-space model** ready to plug into:

* A simple **state-feedback controller**, or
* A **MPC formulation** (with constraints on u and y).

---

## 5. How You’d Use This in Practice

1. Pull historical logs (u₁, u₂, y₁, y₂) over a “good” time window.
2. Align metrology and recipe timings (handle delays).
3. Build the Φ matrix and Y vector as shown.
4. Solve for θ using least squares.
5. Construct A, B from θ.
6. Validate:

   * Check that residuals (y – predicted) have ~zero mean.
   * Residual RMSE smaller than the old model.
   * No strong autocorrelation in residuals.

If validation passes, you’ve got a **re-identified 2×2 model** that better matches the current tool behavior and can be used to update EWMA+FF logic or MPC.