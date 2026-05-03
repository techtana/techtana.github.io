---
layout: post
title: Uncontrolled internal bias estimate
subtitle: It's normal for an controller's internal bias estimate to move, but when is it considered bad? 
author: Tech Tana
categories: engineering
tags: controls
---

Bias movement in an EWMA or R2R controller is normal. The internal bias estimate naturally goes up and down as the tool warms, ages, stabilizes after preventive maintenance, or reacts to environmental cycles. A controller is specifically designed to track and cancel this drift, so if the bias estimate never moved, the loop would fail to compensate for everyday shifts in the process.

"Bias estimate creeping" refers not to normal movement but to a specific unhealthy pattern: a persistent, one-directional increase or decrease that continues wafer after wafer without leveling off or reversing. This differs from normal drift, which is typically bounded and follows recognizable patterns.

Healthy drift can look like:
* The bias rising in the morning as the tool heats, then falling at night as it cools.
* Chamber seasoning after PM, where the bias shifts for 20–30 wafers and then stabilizes.
* Consumables slowly wearing down and then being replaced, causing the bias to return to baseline.
* Lot-by-lot thermal load effects, where bias oscillates depending on run order or product mix.

In all of these cases, the bias moves but remains within a consistent, repeatable range. It may rise for a while, level off, and later come back down. Movement and cycling are completely normal.

Unbounded creeping, however, is different. It shows up when the bias estimate keeps increasing (or decreasing) indefinitely:

Example of unhealthy creeping:  
> 100 → 102 → 104 → 106 → 108 → 110 → …

This indicates that the controller is compensating more and more to hold the process on target, which usually reflects real tool deterioration such as chamber contamination, aging of components, sensor drift, or weakening thermal control. The controller can hide this degradation for a while, but eventually it runs into physical or recipe limits. At that point, an excursion occurs because the underlying tool is no longer capable of producing the target outcome, even with maximal control effort.

The danger is not the movement itself; it is that creeping:

* masks underlying tool wear,
* pushes the controller toward saturation,
* increases move variance and noise sensitivity, and
* hides problems from SPC charts until it is too late.

In summary, bias movement is expected and healthy; bias cycling is normal; but sustained one-way creeping signals that the tool is slowly failing while the controller compensates in the background. The key distinction is that healthy drift eventually stabilizes or reverses, while creeping does not.