---
layout: post
title: Understanding system capability gaps
subtitle: Align systems, process and tools between factories 
author: Tech Tana
categories: leadership
tags: leadership
---


Here’s a clean, practical structure you can give your team. It forces them to think in terms of capability, gaps, trade-offs, and decision criteria—not just listing tools.

Use this as a template + set of leading questions.


---

1. Capability-First Comparison

Purpose: Ensure the discussion isn’t about tools themselves but about what problems they solve.

Format

Capability	Factory A Tools	Factory B Tools	Capability Match?	Notes

Example: Inline drift detection	Tool X	Tool A + Tool B	Partial	Factory B needs extra scripting


Ask these questions

What capability does this tool deliver? (Define in one sentence.)

Is this capability already delivered by any combination of tools in our fab?

Does the other factory deliver this capability more efficiently or reliably?

Are there hidden capabilities (e.g., automation hooks, dashboards, workflow integration)?



---

2. Gap & Impact Assessment

Purpose: Identify where you’re missing capability that actually matters.

Format

Capability Gap	Impact if Unaddressed	Severity (H/M/L)	Current Workaround	Cost of Workaround



Ask these questions

What breaks today because we lack this capability?

What decisions, automation, or yield improvements depend on this capability?

How much time/effort do engineers spend on workarounds?

Would the capability reduce risk (e.g., excursion response, cycle time variance)?



---

3. Trade-off & Redundancy Check

Purpose: Prevent bringing in tools that don’t add differentiated value.

Format

Tool	Incremental Capability	Overlap With Existing	Trade-off	Recommendation



Ask these questions

What does this tool do that no existing tool can do?

Could two existing tools combined replace it?

What is the opportunity cost of deploying, qualifying, training, and maintaining it?

Does redundancy improve resilience or just increase noise/complexity?



---

4. “Should We Bring It In?” Analysis

Purpose: A structured, defendable decision.

Format

Tool	Needed? (Yes/No/Maybe)	Why	Expected Gains	Cost/Risk	Timeline to Deploy	Usage Scenarios



Ask these questions

If adopted, what measurable improvement would this tool create?

Faster decisions?

Lower scrap/risk?

Better automation?

More uniform workflow across sites?


When would engineers actually use it? Daily? Rarely?

What training does it require?

What systems does it need to integrate with?



---

5. Usage Scenarios (“How and When to Use It”)

Purpose: Avoid tools that become shelfware.

Format

Scenario A: (e.g., recipe drift >5%, or SPC alarm, or CD tool re-qualification)

What tool handles this scenario today?

What value does new tool add?


Scenario B: …

Scenario C: …


Ask these questions

In what situations does the new tool outperform the current system?

Are there scenarios where the new tool is the only practical option?

How often will these scenarios occur in the real fab environment?



---

6. Final Recommendation Packet (What you review)

Ask the engineer to deliver a 1–2 page memo containing:

1. Capability Map

Side-by-side capabilities of Factory A vs Factory B

Highlight gaps and overlaps clearly


2. Gaps That Actually Matter

Which missing capabilities affect yield, cycle time, automation, or quality?

Which gaps are noise?


3. Tool-by-Tool Decision Summary

Bring in / Don’t bring in / Monitor for future

Clear reasons


4. Usage Guidance

When the capability is needed

How it plugs into your existing workflow

Who would own it



---

7. Starter Set of Prompts for the Team

Give them these exact questions to structure their thinking:

1. What capability does this tool provide, and what problem does that solve?


2. Which existing tools provide similar capability, and to what degree?


3. What is the capability gap if we don’t bring this tool in?


4. What is the operational impact of that gap?


5. If we bring it in, what capabilities improve and who benefits?


6. What are the integration costs (training, data, workflow)?


7. Under what specific scenarios will this tool be used?


8. How often will those scenarios occur?


9. What decision can be made with the new capability that can’t be made today?


10. Is the tool uniquely impactful, or is it mostly redundant?




---

Add a “Risk Addressed” Column for Every Capability

Every capability exists because it protects the factory from a specific set of risks.
Have them answer one question per capability:

> “What risk does this capability reduce, prevent, or make recoverable?”



Format

Capability	Tool(s)	Risk Addressed	Risk Level (H/M/L)	Notes

Recipe drift detection	Tool X	Undetected drift causing scrap or tool offline	High	Current workaround is manual trending



---

Categories of Risk to Consider

Give your team these buckets so they think broadly, not just yield or SPC.

1. Yield Risk

Wrong recipe parameters

Measurement drift

Process window shrinkage

Poor tool matching

Recipe instability after PM


2. Quality / Excursion Risk

Delayed detection of out-of-control events

Inconsistent operator decisions

Manual data manipulation or gaps

False alarms masking real issues


3. Factory Throughput / Cycle-Time Risk

Tools running out-of-window causing rework

Slow decision cycles

Excessive manual checks

Poorly automated feedback loops


4. Automation and Integration Risk

Missing API hooks for automated R2R

Fragmented workflows

Tools that require too much manual intervention

Different sites using incompatible systems


5. Data Integrity & Visibility Risk

Untrusted metrology data

Missing timestamps or missing lot context

Tool logs not aligned

Difficulty reconstructing an event chain during excursions


6. Operational Consistency Risk

Sites making different decisions for the same scenario

BKM not reproducible across factories

Inconsistent escalation logic

Training gaps or tribal knowledge dependence


7. Compliance / Audit Risk

Missing audit trails

Incomplete documentation

Inability to demonstrate control logic or decisions

Outdated or non-compliant systems



---

Add a Crisp Risk Assessment for Each Capability

Tell your team to answer these clearly:

1. What failure mode does this capability protect us from?

Example:
“Protects from slowly drifting CD measurements that escape SPC.”

2. What happens if this risk is NOT addressed?

Scrap?

Rework?

Lost WIP?

Quarter-over-quarter yield drops?

Engineering time wasted?

Customer quality risk?


3. How often does this risk appear?

Rare / occasional / frequent.

4. How severe is the consequence?

High / Medium / Low.


---

Add Risk to the Final Decision Template

This makes your recommendation packet fully defendable.

Tool	Capability	Risk Addressed	What Happens If We Don’t Bring It In	Incremental Value	Decision

Tool Y	Drift modeling	Undetected param drift	Scrap + delayed matching	High	Bring in


---

Leading Questions for Your Team

Give them these to think deeply:

1. What specific failure does this capability prevent?


2. What is the worst realistic outcome if we don’t have it?


3. What current workaround is being used, and what risk does it leave open?


4. Does the new tool reduce that risk significantly, or only slightly?


5. Is the risk tied to yield, quality, throughput, automation, or consistency?


6. How frequently has this risk materialized in the past 12–24 months?

7. If we adopt this capability, what new risks are introduced?
(Training risk, integration risk, maintenance risk.)
