---
layout: post
title: "Manufacturing AI: A Pilot-to-Scale Checklist"
date: 2026-09-11 00:00:00 +0000
categories: ["Reference", "AI"]
tags: ["manufacturing", "ai", "checklist", "engineering"]
description: "A reference checklist for taking a factory AI pilot to scale: owner, baseline, controlled metrics, data pipeline, costs at scale and a decision date."
image: "https://cdn.sanity.io/images/563mnkns/production/b37ab4f0ff56a1a94aa7616f017707ed88de3ed2-1600x650.jpg"
author: "TechCirkle Editorial Team"
---

Plenty of manufacturing AI pilots work and still go nowhere. The causes repeat: the use case was picked for being easy rather than valuable, it was run by an innovation team with no production ownership, or it relied on manual data wrangling nobody could sustain. This is a reference checklist for avoiding all three. Work through it before the pilot starts, not after.

![Engineer monitoring automated robot arms on a real-time software dashboard in a smart factory](https://cdn.sanity.io/images/563mnkns/production/b37ab4f0ff56a1a94aa7616f017707ed88de3ed2-1600x650.jpg)

## 0. Use case gate

- [ ] The use case sits in the top three of the plant's loss tree (downtime, scrap, changeover, energy, labour hours).
- [ ] It passes the one-sentence test: *we want to change decision X, made today by role Y using information Z, and a better decision is worth roughly N per year.*
- [ ] It maps to the plant manager's own targets, not just to what is easy to demonstrate.

If any box is empty, stop here. Nothing below will rescue a use case that fails the gate.

## 1. Owner

- [ ] A named operational owner (maintenance lead, quality manager, lead planner) who will use the output daily.
- [ ] That person's feedback shapes the product during the pilot.
- [ ] The owner has authority to change how work is done on the pilot line.

**Why it matters:** pilots owned by innovation teams produce reports. Pilots owned by operations produce changed decisions.

## 2. Baseline

- [ ] The current value of the success metric is recorded **before** the system goes live.
- [ ] The baseline covers a long enough period to include normal variation (shifts, product mix, seasonality).
- [ ] The baseline is expressed in money or hours, not model accuracy.

## 3. Metric with a control group

- [ ] One primary metric agreed with the owner and finance.
- [ ] A control is defined: a comparable line, asset group or time period without the system.
- [ ] Reporting runs from week one, not from the end of the pilot.

Without a control, a demand swing, a new supervisor or a raw-material change can be mistaken for AI impact, in either direction.

Reference metrics by use case:

| Metric | What it shows | Typical use case | Suggested control |
|---|---|---|---|
| OEE: availability | Share of planned time the asset actually ran | Predictive maintenance | Similar assets without monitoring |
| OEE: performance | Actual speed against ideal cycle time | Scheduling, process optimisation | Same line, prior comparable period |
| OEE: quality | Good parts as a share of total produced | Vision inspection, soft sensors | Sister line with manual inspection |
| Unplanned downtime hours | Lost time from unscheduled stops | Predictive maintenance | Unmonitored asset group |
| Mean time to repair (MTTR) | How long a stop lasts once it happens | Maintenance, knowledge copilot | Technicians not yet using the tool |
| Scrap, rework, customer escapes | Cost of quality that got through or was caught late | Vision inspection | Comparable line or shift |
| Schedule adherence and overtime | How often the plan survives the week | Planning optimisation | Prior period with legacy scheduling |
| Time to diagnose, first-time fix rate | Whether technicians fix it right, first time | Knowledge copilot | Control crew or shift |

Report OEE split into its three components. A single OEE number hides which part moved.

![Engineer performing maintenance on an industrial robot arm on the factory floor](https://cdn.sanity.io/images/563mnkns/production/126e3410498ceaac125fb3637760145585bdc088-1600x1066.jpg)

## 4. Data pipeline

Build it properly on day one, even for a pilot. Scale-up should not mean starting again.

- [ ] **Connectivity:** OT-to-IT bridge in place (for example OPC UA or MQTT), or retrofit sensors on machines with no digital output.
- [ ] **Context:** raw tags mapped to asset, line, product and batch.
- [ ] **Quality:** handling for sensor drift, gaps, clock mismatches and units that change after firmware updates.
- [ ] **Labels:** maintenance records and quality outcomes joined back to sensor data as ground truth.
- [ ] **Governance:** an owner, retention period and data-egress rule per source.
- [ ] Designed as a reusable data layer, so the next use case does not pay for the plumbing again.

Expect 60 to 70 percent of total effort to land in this section.

## 5. Integration and deployment

- [ ] Output lands where the owner already works (CMMS, MES, ERP), not in a separate dashboard.
- [ ] Models are versioned, with staged rollout to one line before all lines.
- [ ] Rollback to the previous model version is tested.
- [ ] Drift monitoring is in place.
- [ ] No AI component has direct write access to PLCs or safety systems.

## 6. Costs at scale

- [ ] Licence pricing modelled at full plant scale (for example fifty assets, not five).
- [ ] Hardware (sensors, cameras, edge devices) costed for the second and third lines.
- [ ] Annual run costs budgeted at roughly 15 to 25 percent of build cost.
- [ ] Contract reviewed for data export rights and model ownership if you leave the vendor.

## 7. Oversight

- [ ] Decisions the system may take alone are listed (for example flagging a part for re-inspection).
- [ ] Decisions that always need a person are listed (stopping a line, changing a validated parameter).
- [ ] An auditable record exists of each recommendation, its input data and the approver.

## 8. Decision date

- [ ] A fixed date to **scale, adjust or stop**, agreed before kickoff.
- [ ] The criteria for each outcome are written down in advance.
- [ ] Plans for lines two and three are drafted while line one is still running.

Pilots without an end date turn into permanent science projects.

## Further reading

The source guide covers the five tool categories, the data layer, costs and buy-versus-build in depth: **[AI Automation Tools for Manufacturing: What Actually Works on a Real Factory Floor](https://techcirkle.com/blog/ai-automation-tools-for-manufacturing)**.

For the integration and data-layer work this checklist depends on, see our [custom software development](https://techcirkle.com/development/custom-software-development) and [AI development services](https://techcirkle.com/ai-development-services).

## Frequently Asked Questions

### Why do successful manufacturing AI pilots fail to scale?

Usually because the pilot was chosen for ease, had no production owner, or depended on hand-built data work that could not be repeated on other lines.

### What is the most important item on the checklist?

The named operational owner. Without someone who uses the output daily and has authority to change the process, even an accurate model changes nothing.

### Why report OEE as three separate components?

Availability, performance and quality respond to different interventions. Splitting them shows which one the AI actually moved and prevents gains in one hiding losses in another.

### How long should a pilot run before the decision date?

Long enough to cover normal variation in shifts and product mix. For a single-line pilot that typically sits within an 8 to 16 week build and run window.

### What cost is most often missed at scale?

Per-asset or per-camera licensing. It looks small on a pilot line and can become the biggest line item across a whole plant.
