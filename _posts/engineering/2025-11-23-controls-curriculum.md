---
layout: post
title: Curriculum for Process Control Engineering
subtitle: What do you have to learn to be an effective process control engineer?
author: Tech Tana
categories: engineering
tags: controls
---

As part of my quest to learn how to be a good control engineer, I did a lot of google searches, ChatGPT prompts, read books and articles, and watch a lot of YouTube videos. In case I step out from this field sometimes in the future or for others who would also like to go down the path of Process Control Systems, I'm compiling the curriculum that I think makes sense for an engineer to have the most impact in a short time. Additionally, being able to implement things in real world quickly is a great hook for new learner as well.

When planning curriculum, I first identify what someone would need to implement in their job, then walk backward to list the most fundamental knowledge (coding, math, stats, or other concepts) to enable that. Finally, we can leave deeper understanding, more advanced optimization or pathfinding concepts to later. If I map these curriculum to engineering level in a company, it would be:
1. Junior engineers to learn the basics, enough to understand the detailed instructions and able to explain the technical part of their job sufficiently.
2. Senior engineers to learn the fundamental principles, allowing them to derive the decision from scratch and confidently breakdown problems to fundamental pieces.
3. Staff engineers to learn beyond their single, narrow field and start merging multiple domains together to create new understanding. Or, in exceptional case, a very deep understanding of a highly complex field.
4. Principal engineers to be the technical multipliers that elevate the entire engineering ecosystem and solve problems where no playbook exists.

At the higher levels, it becomes impossible to give a generic learning plan. So, I'll focus on the learning path from newbies to senior contributions.
# Process Control Engineering Curriculum: Newbie to Senior 
## Foundation
### 1. Intro to process control systems (SPC, APC, FDC)
* What is the SPC, APC, FDC in a nutshell?
* How controls fit into bigger picture and deliver values?

---

### 2. Minimal Math
> [!NOTE]
> If you have a STEM degree, many of these topics should be familiar. 
> I asked ChatGPT to create a quiz to test the readiness of your knowledge, so you can "test-out": 
> [Quiz and answer keys.](control-engineering-testout.html)
#### Algebra & Data Operations
* Mean, variance, moving variance
* Weighted averages (EWMA)
* Covariance / correlation
* Linear regression basics (for ARX and FF)
#### Discrete-Time Calculations
* Difference equations
* Online variance estimators
* Variance reduction factors
#### Probability
* Normal distribution (for Cp/Cpk)
* Short vs long-term sigma
* Outlier identification (3σ)

---

### 3. Fundamental SPC
#### Core SPC Concepts to Support Control
**Goal:** To enable engineers to judge variation and decide if control is needed. 
* Common vs special cause variation
* Control charts:
  * X-bar chart
  * Range/SD chart
  * EWMA chart
  * Individuals (I) chart
* Rational subgrouping (why wafer and lot grouping matters)
* Run rules to detect excursions, including Western Electric and Nelson Rules
#### How SPC and Performance Metrics Are Used in Daily Engineering
* How SPC is placed where engineers actually use it:
  * Before control (baseline understanding)
  * During controller tuning (does the loop stabilize variation?)
  * After control (did Cpk/Harris improve?)
* Process capability: Cpk (ignore Cp and Ppk; less practical)
* Implication of Analysis of Variance (ANOVA) when assigning variance to subgroups
  * Characterize the incoming variation for FF design
* Understanding variance
  * When to use variance vs capability index? 
  * SPC: Define baseline sigma before/after enabling control
  * SPC: Comparing Long-term sigma vs short-term sigma to indicate creeping shifts
  * Time-Series: Consistent $Δy[k]$ in the same direction
  * Time-Series: Observer residual slowly growing (in state-space/ARX systems)
  * [Bias estimate creeping in EWMA/R2R controllers](controls-uncontrolled-bias.html)
* [Controller tuning and optimization](controls-parameter-tuning.html)
  * [How to tune process model?](controls-process-model-inaccuracy.html)

---

### 4. Controller Implementation
#### 4.1 PID
##### Implementation
* Discrete P, I, D
* Anti-windup
* Rate limits
##### Performance Metrics
* IAE/ISE (integrated error)
* Overshoot, settling time
* Actuator usage
* Oscillation detection
##### SPC Tie-in
* Use I-chart to evaluate loop stability
* Use Xbar chart for drift before tuning
* Cpk improvement after PID is applied
* Control limits narrowing = success
* 
#### 4.2 EWMA Feedback (Semiconductor R2R)
##### Implementation
* Bias estimator
* Delay compensation
##### Performance Metrics
* Harris Index (FB-only): σy, σmv, delay penalty
* Lambda vs variance reduction
* Cpk improvement vs baseline SPC data
##### SPC Tie-in
* EWMA chart = natural SPC chart for wafer-level loops
* Stability of moving average trend
* I-chart to confirm no over-control
* Detect false alarm rate reduction
  
#### 4.3 EWMA + Feed-Forward (FF)
##### Implementation
* Predictor construction
* Align FF variables
* Bias reset logic
##### Performance Metrics
* Harris with FF (opportunity/effectiveness)
* FF predictor R²
* Incoming vs outgoing sigma ratio
##### SPC Tie-in
* Partition variation:
  * common-cause: FF should reduce
  * special-cause: identify instead of suppress
* Compare Xbar control limits before/after FF
* Quantify % variation explained by FF

#### 4.4 ARIMAX / Model-Based R2R
##### Implementation
* ARX/ARIMAX model-fitting
* One-step prediction
* Control move = target – forecast
##### Performance Metrics
* Prediction RMSE
* Residual autocorrelation
* Move smoothness
* Cpk delta vs EWMA baseline
##### SPC Tie-in
* Autocorrelated residuals indicate missing variables
* Use EWMA chart on prediction residual to detect model drift
* Control chart limits shrink when model is effective

#### 4.5 State-Space Control
##### Implementation
* A, B, C matrices (minimal dimension)
* Observer (Luenberger)
* State feedback u = –Kx̂
##### Performance Metrics
* Observer residual (fault or drift detection)
* State convergence time
* Control energy (||u||)
* MIMO crosstalk index
##### SPC Tie-in
* Observer residual chart = SPC for hidden states
* Detect PM events via step-change in residual
* Subsystem-specific control charts (profile, thickness, uniformity independently)

---

### 5. Practical Integration Topics (With Metrics)
#### 5.1 Noise & Filtering
* MA, median, IQR-based outlier removal
##### Metrics
* Noise-to-signal ratio
* False excursion rate
* Filter lag impact on variance
##### SPC tie-in
* Use MR chart to measure short-term noise
* Detect measurement tool instability

#### 5.2 Metrology Delay
* FIFO buffer
* Aligning y[k] with u[k]
##### Metrics
* Harris delay penalty
* Data-age histogram
##### SPC tie-in
* Long delays increase false alarms in SPC charts
* Adjust sampling rules for delayed metrology

#### 5.3 Drift, PM, Reset Logic
* Bias reset based on drift detection
##### Metrics
* Drift slope
* Bias error
* Mean shift during PM cycles
##### SPC tie-in
* Xbar shift rules detect PM transitions
* Post-PM shakedown period monitoring

---

### 6. Case Studies (SPC + Control Together)
##### Semiconductor R2R
* Pre-control SPC → apply EWMA → post-control SPC
* Using Harris + Cpk to justify FF addition
* ARIMAX model drift detection via residual control chart
##### PID Loops
* I-chart for process output
* MR-chart for process noise
* Compare control limits before/after tuning
##### Multivariable Loops
* Use SPC per variable + state residual chart

---

> [!Note] Checkpoint
> At this stage in the curriculum (after SPC, basic math, PID, EWMA, FF, ARIMAX, and state-space fundamentals), an engineer should be able to:
> #### 1. Diagnose variation with confidence
> * Separate common-cause vs special-cause variation
> * Use the right SPC chart (I-chart, EWMA, Xbar, MR, etc.)
> * Interpret trends, shifts, drift slopes, and subgroup patterns
> * Explain Long-term vs Short-term sigma and how it affects Cpk
> #### 2. Implement basic controllers correctly
> * Tune and deploy discrete PID with anti-windup
> * Implement EWMA feedback with proper delay handling
> * Build simple feed-forward predictors using covariance or regression
> * Apply bias reset logic safely
> #### 3. Evaluate control performance with proper metrics
> * Compute and interpret Cpk changes before/after control
> * Compute Harris Index (FB and FB+FF)
> * Use incoming vs outgoing sigma ratio to quantify FF value
> * Diagnose under/over-control using SPC + time-series signals together
> #### 4. Build and validate simple models
> * Fit ARX/ARIMAX models and interpret residuals
> * Understand when residual autocorrelation means “missing physics”
> * Use RMSE, R², and residual charts to judge model validity
> * Use one-step prediction as control logic (simple model-based R2R)
> #### 5. Work across the system, not just algorithms
> * Align metrology timestamps, delays, FIFO logic
> * Detect PM events properly and protect the controller
> * Understand measurement noise vs process noise
> * Integrate filtering without masking real excursions
> #### 6. Communicate like a control engineer
> * Explain why a loop works or fails using sigma, bias, drift, delay
> * Justify FF or redesign using Harris + Cpk
> * Translate statistical signals into practical control actions
> * Provide clear tuning recommendations and risk assessments

---

## 📍 What Comes Next in the Curriculum (the Senior Track)

After this point, a learner is ready for deeper or more specialized topics that elevate them toward senior-level capability:

### 1. Advanced R2R / APC Concepts (Senior Level)
* Multivariate control (profile + thickness, CD + overlay)
* Constraint handling (slew limits, operating windows)
* Dead-time compensation and Smith predictors
* Hierarchical control (tool-level vs chamber-level vs fab-level)
* Control of profile/uniformity metrics
* Running control simulations for tuning before fab deployment
### 2. Robustness, Stability, and Fail-Safety
* Stability margins for discrete controllers
* EWMA stability regions with delay
* MPC feasibility and constraint robustness
* Designing safe fallback modes
* Preventing over-control and oscillation
* Implementing health checks on FF / model validity
### 3. System Integration & Architecture
* API structures
* Data quality pipelines
* Event detectors and excursion engines
* Versioning and reproducible controller deployment
* Cross-tool matching strategy
* Building telemetry dashboards for control health
### 4. Real-World Practice and Tooling
* Using Python or R for offline tuning experiments
* Running DOE on simulated or historical data
* Code frameworks for controllers (libraries, templates)
* Unit-testing control logic before deploying
* Building generic controllers that can be reused across tools

---

## 📍 Graduation Criteria for “Senior-Ready” Control Engineers
After completing the senior-track content, an engineer should be able to:
* Design, justify, and deploy a new controller from scratch
* Explain tuned parameters in terms of math + process physics
* Diagnose excursions and stability problems without guesswork
* Select and combine FF, FB, filtering, and SPC appropriately
* Communicate risk, variance sources, and ROI to managers
* Mentor junior engineers in control thinking
* Own the control strategy for an entire toolset or module
