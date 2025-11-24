---
layout: post
title: Understanding system capability gaps
subtitle: Align systems, process and tools between factories 
author: Tech Tana
categories: career
tags: management
---

One of my first 90-day goals is to identify any gaps in SOP, tools, and training to prepare the team to be successful. In the beginning, it felt overwhelming to find out what the team needs. Having a global quality coach visiting for a few weeks tremendously helps me understand the expectations, challenges and opportunities in the HVM environment. Over the past few weeks, the shift in business from R\&D to manufacturing and various issues that got escalated point to the weakness that we can address. 

When tasking my team member to list out the gaps in our operations, I learned that it's more practical to think in terms of capability, risks, and trade-offs. As controls engineers, our core purpose is reducing variation. Even though our work often expands into areas like tool availability, cost efficiency, cycle-time improvement, and technology enablement, the anchor for every major decision is the same: does this help us control and reduce process variation? Every controls capability exists to shield the factory from a specific class of risks.

Before going into risk categories, it's important to note that yield and quality often move together, but they are not the same. Yield is the product outcome — how many units pass specification. Quality is the health and stability of the process that creates those units. A process can be high-quality yet produce low yield if it's consistently centered on the wrong target, and it can have high yield yet poor quality if it stays in spec by luck or wide limits. Treating them as separate lets engineers apply the right levers: quality fixes stabilize the process, while yield fixes optimize the product outcome. It also gives us a guideline in how to use metrics in the right scenarios: If Spec limits are setup correctly, Cpk directly ties to yield. Harris Index describes stability of a process compared to minimal variance, but hot how close you are to the spec.
   * Good Harris + Low Cpk → Low variation; its a centering/recipe issue.
   * High Cpk + bad Harris → you're living on borrowed time; excursions are coming.

**Follow-up:** How does this impact how and when we look at Cpk, Harris Index, chart criticality? Or, how do we think through project prioritization?


### Key Risk Categories for Control Systems
1. **Product Yield Risk:** Will this cause chronic scrap or parametric loss?
   This is about whether wafers meet spec — pure product outcome.
   For example, unstable recipes, lack of control capability, wrong control parameters, uncontrolled drift, shrinking process windows, poor tool performance matching.
2. **Product Quality Risk:**  Will something go out-of-control and we don't notice in time?
   This is about how fast you detect a problem and implement containment, not product yield.
   For example, invisible issue, slow detection, inconsistent decisions, manual data gaps, false alarms, untrusted data or signals.
3. **Process Throughput and Cycle-Time Risk:**  Will this slow the factory down?
   Even if yield is fine, slow decisions, rework loops, or manual steps create factory bottlenecks, not quality problems.
   For example, out-of-control tools causing rework, slow decisions, heavy manual checks, weak automation.
4. **Human Operation Risk:**  Will two engineers make different decisions for the same issue? Can we trust the data and understand what happened?
   This is about human processes, consistency, reproducibility, and productivity.
   For example, hard-to-reconstruct events, different sites making different calls, non-repeatable BKMs, inconsistent escalation, training gaps.
5. **System Operation Risk:**  Can the system run automatically or integrated into larger ecosystems?
   Lack of APIs or workflow plumbing hurts scalability, even if throughput and quality are good.
   For example, data or log discrepancy, missing context/timestamps, fragmented workflows, high manual intervention, incompatible systems across sites.
6. **Compliance and Audit Risk:**  Can we prove our control logic and decisions are documented and compliant?
   This matters for audits, certifications, and external scrutiny beyond daily operations.
   For example, missing audit trails, weak documentation, unclear control logic, outdated systems.



When evaluating new tools, we can as the following questions: 

1. **Capability:**
   What capability does this tool add (e.g. faster decisions, lower scrap, more accurate automation, more uniform workflow)?
   What concrete problem does it solve (e.g. excursion response, cycle time variance)?
2. **Risk:**
   Which risk does it address—yield, quality, throughput, automation, operations, compliance?
3. **Decision impact:**
   What breaks today because we lack this capability?
   What decisions or actions become possible that we cannot do today?
4. **Redundancy check:**
   What existing tools or workarounds offer similar capability? How complete, reliable or efficient are they?
   Could two existing tools combined replace it?
   Does redundancy improve resilience or just increase noise/complexity?
5. **Hidden or Secondary Value:**
   Does the tool bring extra benefits (e.g. automation hooks, dashboards, integration points) that aren't obvious?
6. **Consequence of not adopting:**
   What is the most realistic negative outcome if we don't adopt it?
7. **Adoption Cost:**
   What are the integration costs — training, system integration, data alignment, workflow changes, and support?
8. **Usage scenario:**
   In what specific scenarios would we use this tool?
   How often have those occurred in the last 12 months?
9. **New risks:**
   What new risks does the tool introduce—training burden, integration complexity, maintenance load, or dependency?

***

An example of a form:

| Capability / Tool                                   | Fab A                                                          | Fab B                                              | Gap?                                | Impact (H/M/L)                              | Current Workaround                    | Unique Value                                                                           | Decision (Y/N/M)    | Expected Gain                                                        | Cost/Risk                                            | Integration                                          | Usage Scenario                                                        |
| --------------------------------------------------- | -------------------------------------------------------------- | -------------------------------------------------- | ----------------------------------- | ------------------------------------------- | ------------------------------------- | -------------------------------------------------------------------------------------- | ------------------- | -------------------------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------- | --------------------------------------------------------------------- |
| **Wafer-Level CD Prediction (Virtual Metrology)**   | None                                                           | VM engine with ML-based CD prediction              | **Yes**                             | **Medium** (cycle-time, sampling cost)      | Extra metrology sampling              | Reduces sampling and provides early warnings                                           | **M** (pilot first) | Better run-to-run correction, reduced metrology load                 | High (model training, validation, data prep)         | Needs data lake + APC link                           | High-throughput steps with long metrology queues                      |
| **Control Config Copy Between Derivative Products** | Automated config-clone with parameter remapping + guard checks | Manual rebuild of control configs for each product | **Yes**                             | **Medium** (slow deployment, config errors) | Copy-paste + engineer review          | Cuts config setup time, reduces human error, ensures consistent tuning across products | **Y**               | Faster rollout of R2R, fewer mistakes, consistent FF/PID/EWMA tuning | Low (scripts + training)                             | Needs link to recipe database + APC config templates | When spinning up a new derivative product with similar process window |
| **Advanced SPC Dashboard (3rd-party tool)**         | Built-in APC/SPC dashboard with auto-alerts                    | Proposes new 3rd-party SPC analytics tool          | **No** (capability already covered) | **Low**                                     | Existing SPC charts + APC auto-alerts | Slightly nicer UI, more chart types                                                    | **N**               | Minimal (no new decisions enabled)                                   | Medium (license + training + dual systems confusion) | Would need MES/APC connectors                        | Rare; current system already meets control and quality needs          |