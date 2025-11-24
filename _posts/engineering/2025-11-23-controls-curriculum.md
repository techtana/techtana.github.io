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

### 2. Minimal Math
> [!NOTE]
> If you have a STEM degree, many of these topics should be familiar. 
> I asked ChatGPT to create a quiz to test the readiness of your knowledge, so you can "test-out": 
> [Quiz and answer keys.](2025-11-23-control-engineering-testout.md)
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
* Process capability: Cpk (ignore Cp and Ppk; less practical)
* Implication of Analysis of Variance (ANOVA) when assigning variance to subgroups
  * Characterize the incoming variation for FF design
* Understanding variance
  * When to use variance vs capability index? 
  * SPC: Define baseline sigma before/after enabling control
  * SPC: Comparing Long-term sigma vs short-term sigma to indicate creeping shifts
  * Time-Series: Consistent $Δy[k]$ in the same direction
  * Time-Series: Observer residual slowly growing (in state-space/ARX systems)
  * [Bias estimate creeping in EWMA/R2R controllers](2025-11-23-controls-uncontrolled-bias.md)
* [Controller tuning and optimization](2025-11-23-controls-parameter-tuning.md)
  * [How to tune process model?](2025-11-23-controls-process-model-inaccuracy.md)