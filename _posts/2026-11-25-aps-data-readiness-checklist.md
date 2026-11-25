---
layout: post
title: "APS Data-Readiness Checklist: What to Fix Before Any Scheduler (AI or Not) Touches Your Plant"
date: 2026-11-25 00:00:00 +0000
categories: ["Guides", "Manufacturing"]
tags: ["manufacturing", "data quality", "aps", "checklist", "ai"]
description: "A README-style checklist for APS data readiness: items, routings, changeovers, actuals and overrides to validate before AI scheduling goes live."
image: "https://cdn.sanity.io/images/563mnkns/production/c7cab271d9f52104301adff850f80c958ff826f3-1600x1244.jpg"
author: "TechCirkle"
---

> **TL;DR** Every planning project has a data-readiness phase, whether or not it is in the plan. This checklist makes it explicit. Work top to bottom; do not skip to the solver.

![Isometric illustration of a smartphone surrounded by glowing line charts, currency icons and cloud sync symbols, representing validated planning data](https://cdn.sanity.io/images/563mnkns/production/c7cab271d9f52104301adff850f80c958ff826f3-1600x1244.jpg)

This checklist complements our full guide to [production planning software for manufacturing](https://techcirkle.com/blog/production-planning-software-for-manufacturing), which covers APS vs ERP, buy vs build, costs and rollout. Use this page as the working document for the data phase.

---

## Contents

1. [Scope](#1-scope)
2. [Master data](#2-master-data)
3. [Constraints](#3-constraints)
4. [Demand and supply](#4-demand-and-supply)
5. [Actuals feed](#5-actuals-feed)
6. [Override capture](#6-override-capture)
7. [ML-assisted validation](#7-ml-assisted-validation)
8. [Exit criteria](#8-exit-criteria)

---

## 1. Scope

- [ ] Identify the **bottleneck resource(s)**. Readiness work starts there.
- [ ] List every system that feeds the planner: ERP, MES, quality, maintenance, HR or shift rosters.
- [ ] Name an **owner** for each data entity (items, routings, work centres, calendars).
- [ ] Agree the planning horizons in scope: S&OP, MPS, finite scheduling, dispatching.

## 2. Master data

### Items and BOMs
- [ ] Every active item has a current, released BOM.
- [ ] Phantom and obsolete components are flagged or removed.
- [ ] Units of measure are consistent between BOM, routing and inventory.

### Routings
- [ ] Each routing lists only operations that are **actually performed** today.
- [ ] Inspection and quality steps that happen on the floor are present in the routing.
- [ ] Alternate routings that live in a supervisor's head are documented.
- [ ] Overlap rules (can operation 20 start before 10 finishes?) are explicit.

### Work centres
- [ ] Work centres in the ERP match the physical floor, including merged or split cells.
- [ ] Each work centre is defined as a single machine or a pool, not ambiguous.
- [ ] Calendars, shifts and planned downtime are current.

### Standard times
- [ ] Run times reviewed against recent actuals for the top items by volume.
- [ ] Standards older than a set age are flagged for review.

## 3. Constraints

- [ ] **Changeover matrix** exists per resource, by product pair or attribute family, not one average setup time.
- [ ] Labour skills and certifications by shift are modelled.
- [ ] Secondary resources (fixtures, tooling, ovens, moulds) are modelled where they constrain flow.
- [ ] Shelf-life, cleaning-validation or campaign rules captured, if relevant.
- [ ] Planned maintenance windows flow in from the maintenance system.

## 4. Demand and supply

- [ ] Open customer orders carry a requested date and a priority.
- [ ] Purchase orders carry the supplier's **promised** date, kept separate from any **predicted** date.
- [ ] Inventory positions are reconciled with a recent cycle count for bottleneck inputs.

## 5. Actuals feed

The planner must know what really happened, quickly.

| Signal | Source | Target latency |
| --- | --- | --- |
| Operation start / stop | MES or machine | Minutes |
| Good and scrap quantity | MES | Minutes |
| Resource state (up, down, reason) | Machine / MES | Minutes |
| Quality holds | QMS | Minutes to an hour |
| Goods receipt | ERP | On posting |

- [ ] Events include **event time** and **ingestion time**.
- [ ] Events carry an idempotency key so replays are safe.
- [ ] Edge collectors push outbound if the OT network is segmented.

## 6. Override capture

- [ ] Supervisors can change sequence only through an interface that records the change.
- [ ] Each override records who, when, from what, to what, and a **reason code**.
- [ ] Reason codes are a short, reviewed list, quick to enter on a tablet.
- [ ] A monthly review of top override reasons is scheduled.

Overrides are not failures. They are labelled data about constraints the model is missing.

## 7. ML-assisted validation

Machine learning is useful here before it is useful for scheduling.

- [ ] Plan-vs-actual gap measured **per operation** in shadow mode.
- [ ] Worst offenders ranked by impact on the bottleneck.
- [ ] A model trained on MES history proposes corrected standard times for review.
- [ ] Routings whose actuals diverge consistently are flagged automatically.
- [ ] Setup-time and supplier-arrival models are versioned, and their outputs treated as owned, validated data.

If you want help building the validation and prediction layer, see our [AI development services](https://techcirkle.com/ai-development-services).

## 8. Exit criteria

Move to bottleneck go-live only when:

- [ ] Shadow-mode plans for the bottleneck are close enough to actuals that supervisors find them credible.
- [ ] The actuals feed has been stable for an agreed period.
- [ ] Override capture is live and reasons are being reviewed.
- [ ] A named owner exists for the planning model after go-live.

---

## Notes

- **AI amplifies good data; it does not rescue bad data.** No solver fixes a routing that skips an inspection step.
- Treat data readiness as a product with an owner, not a one-off cleanup.
- Re-run this checklist when the floor changes: new products, retired machines, new crew patterns.

Need a second pair of eyes on your planning data? [Contact TechCirkle](https://techcirkle.com/contact-us).

---

## Frequently Asked Questions

### Why start with the bottleneck?

Because schedule quality on the constraint resource drives most of the throughput. Clean data there gives the fastest visible payoff.

### How do we find routings that are wrong?

Run the planner in shadow mode and compare planned versus actual time per operation. Consistent gaps point to outdated routings or standards.

### Do we need a changeover matrix for every resource?

Prioritise resources where changeovers are long or vary a lot by product pair, starting with the bottleneck. Others can use simpler estimates initially.

### Should predicted values overwrite master data?

No. Keep promised and predicted values separate, and treat model outputs as versioned data with an owner and review process.

### How quickly must actuals reach the planner?

Within minutes for operation and resource events. Slower feeds mean the planner is working from a picture of the floor that is already out of date.

### When is data "ready enough"?

When shadow-mode plans for the bottleneck are credible to supervisors and the actuals feed is stable. Perfection is not required, but honesty is.
