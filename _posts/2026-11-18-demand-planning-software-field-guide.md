---
layout: post
title: "Demand Planning Software Field Guide: Engineering Notes and Decision Tables"
date: 2026-11-18 00:00:00 +0000
categories: ["Reference", "Engineering Notes"]
tags: ["reference", "forecasting", "supply-chain", "engineering-notes"]
description: "Engineering field notes on demand planning software: capability, AI, metric, build vs buy, cost and rollout decision tables for quick reference."
image: "https://cdn.sanity.io/images/563mnkns/production/c7cab271d9f52104301adff850f80c958ff826f3-1600x1244.jpg"
author: "TechCirkle"
---

![Isometric smartphone with layered translucent screens surrounded by rising line charts, currency icons and linked rings on a dark background](https://cdn.sanity.io/images/563mnkns/production/c7cab271d9f52104301adff850f80c958ff826f3-1600x1244.jpg)

These notes compress our full [demand planning and forecasting software guide](https://techcirkle.com/blog/demand-planning-and-forecasting-software) into tables an engineering team can reference during evaluation, design reviews or a build vs buy discussion.

## Contents

1. [Mental model](#mental-model)
2. [Outgrown-spreadsheet signals](#outgrown-spreadsheet-signals)
3. [Capability table](#capability-table)
4. [AI: where it helps, where it does not](#ai-where-it-helps-where-it-does-not)
5. [Metrics table](#metrics-table)
6. [Integration notes](#integration-notes)
7. [Build vs buy table](#build-vs-buy-table)
8. [Cost drivers](#cost-drivers)
9. [Rollout phases](#rollout-phases)
10. [Failure modes](#failure-modes)

## Mental model

Three layers. Projects fail when they invest in only the first.

| Layer | Job | Typical failure if neglected |
|---|---|---|
| Forecasting engine | Turn data into numbers | Poor accuracy |
| Planning workflow | Turn numbers into agreed decisions | Departments keep their own spreadsheets |
| Execution link | Push decisions into ERP and purchasing | Forecast never drives an order |

Demand planning answers what customers will want. Supply planning answers what you can make or buy. S&OP reconciles the two each month.

## Outgrown-spreadsheet signals

| Signal | What it usually means |
|---|---|
| Planners mostly gather and clean data | Data pipeline gap |
| Monthly category forecasts, weekly item purchasing | Grain mismatch |
| Stockouts and overstock at the same time | Forecasting problem, not cash |
| Several "official" numbers | No consensus workflow |
| Past forecasts not saved | Accuracy unmeasurable |
| New channels behave differently | One model no longer fits |

Three or more rows true: tooling likely already pays for itself.

## Capability table

| Capability | Must-have test |
|---|---|
| Statistical baseline | Picks a method per series and shows which |
| ML forecasting | Backtested against baseline, used only where it wins |
| Demand sensing | Uses recent POS, orders or web signals for short horizons |
| Hierarchy reconciliation | Item, category, region and total add up |
| Overrides | Reason code required, audit trail kept |
| Scenarios | Copies never mutate the live plan |
| S&OP support | Tasks, deadlines, version locks, cell comments |
| Probabilistic output | Quantiles feed inventory policy directly |

## AI: where it helps, where it does not

| Area | Verdict | Note |
|---|---|---|
| External signals at scale | Helps | Only if signals exist at forecast time |
| Probabilistic forecasts | Helps | Safety stock should depend on uncertainty |
| Pretrained time-series models | Helps, conditionally | Long tail, prototypes, ensemble member; still test |
| LLM copilots | Helps | Explaining forecasts and drafting commentary, not producing numbers |
| Replenishment agents | Helps with guardrails | Limits, escalation, logs, and proven forecasts first |
| New product cold start | Weak | Analogy-based; human judgement and pre-orders still win |
| Promotions | Weak | Misses cannibalisation and post-promo dips |
| Dirty data | Cannot fix | Stockouts recorded as zero demand mislead models |
| Structural breaks | Weak | Detect fast, shorten window, weight human input |

For the copilot and agent layers, see our [AI development services](https://techcirkle.com/ai-development-services).

## Metrics table

| Metric | Use it for | Watch out for |
|---|---|---|
| WAPE | Headline accuracy | Measure at decision grain |
| Bias | Systematic over or under forecasting | Track per category and planner |
| MAPE | Secondary only | Breaks on low volume and zeros |
| Forecast value add | Which process steps help | Requires every forecast to be stored |

Rules of thumb:

```text
grain     = level where orders are placed (e.g. item x location x week)
horizon   = purchasing or production lead time
baseline  = measure current accuracy before any change
storage   = keep every published forecast, forever
```

## Integration notes

| Direction | Data | Source or target |
|---|---|---|
| In | Orders and shipment history | ERP |
| In | Inventory and receipts | WMS |
| In | Sell-through | POS or partner feeds |
| In | Product and customer masters | MDM or ERP |
| In | Promotions and price changes | Commercial calendars |
| Out | Approved forecast | ERP or planning module |

Settle early: ERP version coverage, refresh frequency, master data changes, write-back failure behaviour and upgrade ownership.

## Build vs buy table

| Situation | Recommendation |
|---|---|
| Standard industry forecasting, workflow fits | Buy |
| Unusual demand drivers or channels | Build or extend |
| Forecasting is a competitive edge | Build or extend |
| Several ERPs or legacy plus cloud mix | Hybrid or build integration layer |
| Licence cost at scale exceeds build plus maintenance | Build |
| Want proven S&OP plus custom models | Hybrid |

Shorthand: buy what is standard, build what differentiates. Our [custom software development](https://techcirkle.com/development/custom-software-development) team usually starts builds with a narrow pilot.

## Cost drivers

Orientation only, not quotes.

| Option | Rough shape |
|---|---|
| Light e-commerce forecasting tools | Hundreds to a few thousand dollars per month |
| Mid-market planning SaaS | Tens of thousands per year |
| Enterprise suites | Often six or seven figures per year, implementation extra |
| Custom pilot (one category, one integration) | Tens of thousands |
| Full custom platform | Typically low-to-mid six figures over several months |

Cost goes up with messy source systems, item and location count, real-time sensing, complex approvals and data-residency needs.

Compare three-to-five-year total cost against inventory and planner-time value.

## Rollout phases

| Phase | Window | Exit criterion |
|---|---|---|
| Discovery and baseline | Weeks 1 to 6 | Current accuracy measured, metrics agreed |
| Pilot | Weeks 6 to 14 | Parallel run beats existing process |
| Workflow and integration | Months 4 to 6 | Overrides, approvals and ERP write-back live |
| Scale and intelligence | Months 6 to 12 | All categories live, FVA reviewed monthly |

Always name a business owner in operations alongside the IT owner.

## Failure modes

| Failure | Prevention |
|---|---|
| Buying for the demo | Proof of concept on your own history |
| No baseline | Measure before changing anything |
| Stockouts in history | Flag and mask unavailable periods |
| IT-only project | Planners shape the workflow |
| Override everything | Use FVA to limit harmful overrides |
| Black-box models | Driver views and an explaining copilot |
| No write-back | Connect approved plans to purchasing |
| Automating too early | Agents only after forecasts are proven |

Full reasoning behind each table: [Demand Planning and Forecasting Software: A 2026 Buyer's Guide](https://techcirkle.com/blog/demand-planning-and-forecasting-software).

## Frequently Asked Questions

### Which table matters most in an evaluation?

The metrics table. Agreeing on WAPE, bias and forecast value add at the right grain and horizon prevents misleading demo comparisons.

### Is demand sensing always worth adding?

No. It pays off for fast-moving products with short replenishment cycles. For slower categories, a weekly batch forecast is usually enough.

### Why keep every forecast instead of only the latest?

Without the forecast as it was published, you cannot measure accuracy at the lead time that mattered or work out forecast value add for each step.

### When do these cost figures stop being useful?

As soon as you have a scoped estimate. They are orientation ranges based on typical market patterns, not quotes for any specific project.

### Who should own the forecast after go-live?

A named business owner in operations, supported by the engineering team that owns the platform.
