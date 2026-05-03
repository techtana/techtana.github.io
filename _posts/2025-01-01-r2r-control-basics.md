---
author: Tech Tana
categories:
- jupyter
- test
layout: post
subtitle: jupyter notebook
title: Run-to-run control calculation
---
## Run-to-run controls

Some of the most common control algorithms in manufacturing include Feed-forward, Proportional-Integral-Derivative (PID), EWMA-based R2R, ARMA-based R2R, State-space R2R, and QP-based state-space R2R. "R2R" is short for Run-to-Run control where we modify the process parameters or tuning knobs after the material is already processed, only to correct the issues in the subsequent runs.


```python
from dataclasses import dataclass
from typing import Optional
```

## Feed Forward Controller

Feedforward control is used to anticipate and counteract disturbances before they affect the output. It is particularly useful in situations where the process variable (final dimension of your product) is known to be affected by external or incoming disturbances, such as thickness of paper before folding into a box or changes in humidity of the factory. By measuring these disturbances and applying corrective actions before they impact the output, feedforward control can maintain the desired output and prevent deviations. This proactive approach is especially beneficial in predictable systems where regular disturbances are expected.


```python
@dataclass
class FeedForwardController:
    """
    Simple Single-In-Single-Out (SISO) feed-forward controller:
    
        u_ff = G * dist
    
    Optionally clamps output between u_min and u_max.
    """
    G: float                       # feed-forward gain
    u_min: Optional[float] = None  # lower clamp (None = no limit)
    u_max: Optional[float] = None  # upper clamp (None = no limit)

    def compute(self, dist: float) -> float:
        """
        Compute feed-forward control for given reference r.
        """
        u = self.G * dist

        # Saturation (optional)
        if self.u_min is not None:
            u = max(self.u_min, u)
        if self.u_max is not None:
            u = min(self.u_max, u)

        return u
```


```python
# Suppose process model is roughly: y≈a*u  →  u≈(1/a)*y_target
ff = FeedForwardController(G=1/0.8, u_min=0.0, u_max=100.0)

disturbance = 10.0   # target CD, thickness, etc.
u_recipe = ff.compute(disturbance)
print("Incoming disturbance:", disturbance,"--> Recommended adjustment:", u_recipe)
```

    Incoming disturbance: 10.0 --> Recommended adjustment: 12.5


## PID Feedback Controller

While Feedforward (FF) control anticipates potential disturbances and takes corrective actions before they affect the system's output, Feedback (FB) control corrects model mismatch, drift, and unknown disturbances. 


For PID controller (a type of FB controller), the **P** term reacts to error, the **I** term removes steady-state offset, and the **D** term adds damping and stability. Together they clean up whatever FF cannot predict.


Feedback control is used when the system's output is monitored and compared against the desired input. If a discrepancy is found, an error signal is generated, and the controller adjusts the process to minimize the error. This method is reactive and is particularly effective for slow processes or systems with significant dead time, where the delay between input and output response is considerable. 


```python
@dataclass
class PIDController:
    """
    Simple SISO PID controller (feedback):
    
        e   = r - y
        u   = Kp*e + Ki*∫e dt + Kd*de/dt

    Discrete-time implementation with fixed dt.
    """
    Kp: float
    Ki: float
    Kd: float
    dt: float

    u_min: Optional[float] = None   # lower clamp for output
    u_max: Optional[float] = None   # upper clamp for output

    # Internal states
    integral: float = 0.0
    prev_error: float = 0.0
    initialized: bool = False

    def reset(self) -> None:
        """Reset integrator and previous error."""
        self.integral = 0.0
        self.prev_error = 0.0
        self.initialized = False

    def compute(self, r: float, y: float) -> float:
        """
        Compute PID control action given reference r and measurement y.
        """
        # Error
        error = r - y

        # Initialize previous error on first call
        if not self.initialized:
            self.prev_error = error
            self.initialized = True

        # Proportional
        P = self.Kp * error

        # Integral (with simple accumulation)
        self.integral += error * self.dt
        I = self.Ki * self.integral

        # Derivative (on error)
        d_error = (error - self.prev_error) / self.dt
        D = self.Kd * d_error

        # Raw control
        u = P + I + D

        # Save for next step
        self.prev_error = error

        # Saturation
        if self.u_min is not None:
            u = max(self.u_min, u)
        if self.u_max is not None:
            u = min(self.u_max, u)

        return u
```

In practice, using FF and FB together is the most common setup. FF handles the nominal behavior based on your model, and FF trims the remaining errors and disturbances. The combination is faster, more stable, and more robust than either method alone.


```python
# Given your feed-forward controller:
ff = FeedForwardController(Kff=1.25, u_min=0.0, u_max=100.0)
pid = PIDController(Kp=0.8, Ki=0.1, Kd=0.0, dt=1.0, u_min=0.0, u_max=100.0)

def controller_step(r, y):
    u_ff = ff.compute(r)
    u_fb = pid.compute(r, y)
    return u_ff + u_fb
```

Additionally for PID controller, we can improve the robustness by implementing Anti-windup and Derivative on measurement.

#### Anti-windup:
When the controller hits a limit (like max or min recipe), the integrator can "pile up" error and cause a big overshoot later. Anti-windup stops this buildup so the controller recovers quickly.

#### Derivative on measurement, instead of error:
Because measurements can be noisy, taking the derivative on the measurement makes the controller react to real changes instead of jumping around because of noise or setpoint steps.


```python
# ---------- PID with anti-windup & D on measurement ----------

@dataclass
class PIDController:
    """
    Discrete PID with:
      - Optional anti-windup (integrator clamping)
      - Optional derivative on measurement (less noise-sensitive)

    e   = r - y
    u   = Kp*e + Ki*∫e dt + Kd*de/dt   (or -Kd*dy/dt)

    dt is the (approx.) fixed sample time.
    """
    Kp: float
    Ki: float
    Kd: float
    dt: float

    u_min: Optional[float] = None
    u_max: Optional[float] = None

    anti_windup: bool = True
    use_d_on_measurement: bool = True

    # Internal states
    integral: float = 0.0
    prev_error: float = 0.0
    prev_y: float = 0.0
    initialized: bool = False

    def reset(self) -> None:
        self.integral = 0.0
        self.prev_error = 0.0
        self.prev_y = 0.0
        self.initialized = False

    def compute(self, r: float, y: float) -> float:
        # Error
        error = r - y

        # First call init
        if not self.initialized:
            self.prev_error = error
            self.prev_y = y
            self.initialized = True

        # --- P term ---
        P = self.Kp * error

        # --- D term ---
        if self.use_d_on_measurement:
            # D on measurement: D = -Kd * dy/dt
            dy = (y - self.prev_y) / self.dt
            D = -self.Kd * dy
        else:
            # Classic D on error: D = Kd * de/dt
            de = (error - self.prev_error) / self.dt
            D = self.Kd * de

        # --- I term (candidate before applying anti-windup) ---
        integral_candidate = self.integral + error * self.dt
        I_candidate = self.Ki * integral_candidate

        # Unsaturated control
        u_unsat = P + I_candidate + D

        # Apply saturation
        u = u_unsat
        if self.u_min is not None:
            u = max(self.u_min, u)
        if self.u_max is not None:
            u = min(self.u_max, u)

        # --- Anti-windup: integrator clamping ---
        if self.anti_windup:
            # Allow integration if:
            #  - output not saturated, OR
            #  - output is saturated but error would drive it back in-range
            at_upper = (self.u_max is not None) and (u >= self.u_max - 1e-12)
            at_lower = (self.u_min is not None) and (u <= self.u_min + 1e-12)

            if (not at_upper and not at_lower) or \
               (at_upper and error < 0) or \
               (at_lower and error > 0):
                self.integral = integral_candidate
        else:
            self.integral = integral_candidate

        # Save for next step
        self.prev_error = error
        self.prev_y = y

        return u


# ---------- FF + PID combined ----------

@dataclass
class FFPlusPIDController:
    """
    Combines feed-forward and PID feedback:

      u_total = u_ff + u_fb

    Optionally applies a final saturation.
    """
    ff: FeedForwardController
    pid: PIDController
    u_min: Optional[float] = None
    u_max: Optional[float] = None

    def reset(self) -> None:
        self.pid.reset()

    def compute(self, r: float, y: float) -> float:
        u_ff = self.ff.compute(r)
        u_fb = self.pid.compute(r, y)
        u = u_ff + u_fb

        if self.u_min is not None:
            u = max(self.u_min, u)
        if self.u_max is not None:
            u = min(self.u_max, u)

        return u
```

#### Multi-loop (cascade) control

Use multi-loop (cascade) control when one variable reacts much faster than the other. The fast loop stabilizes the "inner” variable—like power, temperature, or dose—so the outer loop can control the slower variable, such as CD, thickness, or profile, without fighting noise or delays. This makes the outer loop smoother and more predictable.

Cascade also helps when the final output depends on an intermediate variable you can control more directly. For example, thickness depends on deposition rate, and CD depends on dose. Tightening control of that intermediate variable first gives the outer loop a cleaner and more reliable actuator.

Another common scenario is when the main variable alone is too noisy or slow to control directly. Metrology on CD or profile can have delay and variation. A fast inner loop—like dose or temperature control—absorbs disturbances early, so the outer loop only needs to make small, steady adjustments.

Overall, use multi-loop when you have a fast-tracking inner variable, a slow but important outer variable, or a noisy process that needs stabilization before the main control loop can perform well. This is why fabs pair things like dose + CD, rate + thickness, or heater zones + profile.


```python
# Outer: controls "slow" variable y_outer (e.g. CD)
outer_pid = PIDController(Kp=1.0, Ki=0.2, Kd=0.0, dt=1.0,
                          u_min=0.0, u_max=100.0)

# Inner: controls "fast" variable y_inner (e.g. power, temperature)
inner_pid = PIDController(Kp=2.0, Ki=0.5, Kd=0.1, dt=0.1,
                          u_min=0.0, u_max=100.0)

def cascade_step(r_outer, y_outer, y_inner):
    # Outer loop output becomes inner loop setpoint
    r_inner = outer_pid.compute(r_outer, y_outer)
    u = inner_pid.compute(r_inner, y_inner)
    return u
```

## EWMA controller

Another method to implement control is using EWMA-based controller. When using FF and FB together, we have a general concept:
* **FF (feed forward):** use a simple model to guess the recipe from the target. Example: if $y≈a*u$, then $u_{ff}≈r/a$.
* **FB (EWMA feedback):** track the bias between model and reality with an EWMA, and add it on top of FF. That bias slowly corrects for drift and model error.


```python
@dataclass
class EWMAR2RController:
    """
    Simple EWMA feedforward + feedback R2R controller.

    Model assumption (rough):
        y ≈ G * u   →   u ≈ (1/G) * y_target

    - Feedforward:
        u_ff = Kff * r         (Kff ≈ 1/G)

    - Feedback (EWMA bias):
        bias_{k+1} = (1 - λ) * bias_k + λ * Kff * (r - y)

      so the new recipe is:
        u = u_ff + bias

    Parameters
    ----------
    Kff : float
        Feedforward gain (approx. inverse process gain).
    lam : float
        EWMA gain in (0, 1]. Higher → faster correction, more noise.
    u_min, u_max : Optional[float]
        Optional min/max recipe limits.
    """
    Kff: float
    lam: float
    u_min: Optional[float] = None
    u_max: Optional[float] = None

    # Internal state (bias term)
    bias: float = 0.0

    def reset(self) -> None:
        """Reset the EWMA bias."""
        self.bias = 0.0

    def compute(self, r: float, y: float) -> float:
        """
        Compute next recipe given target r and last measured y.

        r : target (e.g., CD, thickness, etc.)
        y : last measured value
        """
        # Feedforward term from model
        u_ff = self.Kff * r

        # Feedback error (target - measured)
        error = r - y

        # EWMA bias update (feedback)
        self.bias = (1.0 - self.lam) * self.bias + self.lam * self.Kff * error

        # Total recipe = FF + FB correction
        u = u_ff + self.bias

        # Saturation
        if self.u_min is not None:
            u = max(self.u_min, u)
        if self.u_max is not None:
            u = min(self.u_max, u)

        return u

```


```python
# Example: process gain G ≈ 0.8 → Kff ≈ 1 / 0.8
G = 0.8
ctrl = EWMAR2RController(Kff=1.0 / G, lam=0.3, u_min=0.0, u_max=100.0)

r = 10.0   # target CD / thickness
y = 9.4    # last measured value

u_next = ctrl.compute(r, y)  # recipe for next lot
print("Target:", r, "Measured:", y, "→ Next recipe:", u_next)
```

    Target: 10.0 Measured: 9.4 → Next recipe: 12.725


#### When to use PID or EWMA?

**Use EWMA when**

* You’re in a **run-to-run or lot-to-lot system**, not continuous time.
* Measurements come only **after each run or wafer** (no real-time data).
* The process is mostly **linear and slow-changing**, like CD vs dose or thickness vs power.
* You want something **robust, low-maintenance, and easy to tune.**
* The main goal is to correct **drift and offset** over time, not track dynamic behavior.

In other words, EWMA fits semiconductor **R2R control** perfectly: one sample per run, simple linear bias, noisy metrology, and long process delays.

**Use PID when**

* You have **continuous or fast, time-based feedback** (seconds or milliseconds, not wafers).
* You need to control **dynamic response** — speed, overshoot, oscillation, etc.
* You can measure the output in real time or at least within the same run.
* The process behaves like a **real-time system** (e.g., temperature, pressure, or motor speed).


**Rule of thumb:**

* **EWMA → wafer-to-wafer or batch-to-batch control.**
* **PID → continuous or within-run control.**
  Sometimes fabs use both: **PID for within-run stabilization** (e.g., power or temperature) and **EWMA for R2R correction** (e.g., CD or overlay drift).

