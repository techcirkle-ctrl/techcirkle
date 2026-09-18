---
layout: post
title: "A 90-Day Plan for Shipping Customer-Facing Analytics"
date: 2026-09-18 00:00:00 +0000
categories: ["Engineering", "Delivery"]
tags: ["engineering", "analytics", "delivery", "saas", "architecture"]
description: "A week-by-week delivery plan for shipping analytics dashboards to customers: metric definitions, pipeline, tenant isolation, internal dogfooding and rollout."
image: "https://cdn.sanity.io/images/563mnkns/production/d3f755161749435428b10e85735dc65f1cb93ae3-1600x1244.jpg"
author: "TechCirkle Editorial Team"
---

![Engineering team planning an analytics delivery](https://cdn.sanity.io/images/563mnkns/production/d3f755161749435428b10e85735dc65f1cb93ae3-1600x1244.jpg)

Most customer-facing analytics projects do not fail. They stall — somewhere around week seven, when the data model turns out to need a rewrite and the charts nobody agreed on are still in the backlog.

This is a delivery plan designed to prevent that specific failure. It is deliberately narrow, and the narrowness is the mechanism.

## Weeks 1–3: Define, Do Not Build

**Pick five metrics.** Not fifty. Interview the customers who asked, and find the five numbers they would check every Monday. Look for what they already track manually — a spreadsheet or a saved report is evidence of real effort, which is a far stronger signal than a stated preference.

**Define each one precisely, in code.** Not in a document. A semantic layer — dbt metrics, Cube, or a governed view layer with a hard rule that application code never touches base tables. The definition specifies the aggregation, the filter, the grain, and the join path.

**Write down the edge cases next to the definition.** What counts as active. How refunds affect revenue. Which timezone a day boundary uses. What happens to a record mid-migration. These are the disputes that surface in month four; resolving them in week two costs a meeting.

**Set a latency budget.** Under 500ms p95 for default views, under 2s for filtered views, explicitly async beyond. Write it in the ticket. It will eliminate architectural options later, which is the point.

**Deliverables:** five metric definitions in version control, an agreed latency budget, a written edge-case register.

**Do not:** design screens, choose a chart library, or start the pipeline.

## Weeks 4–7: Pipeline and Isolation

**Build the data model for exactly those five.** Rollup tables keyed on tenant, metric, time bucket and dimension. Populate incrementally from a watermark; reprocess a trailing window on each run to absorb late arrivals.

**Backfill in throttled batches, off-peak.** An unthrottled backfill causes the exact production incident you are building this to avoid.

**Enforce tenant isolation below the application layer.** Row-level security, or a query gateway that injects the tenant predicate before anything reaches the engine. Never rely on developers remembering a `WHERE` clause — in analytics a missing predicate exposes aggregates, meaning a competitor's totals rather than one stray record.

**Add the CI test that proves it.** A test that attempts cross-tenant access and asserts failure. Run it on every schema change.

**Instrument every query now.** Tenant, duration, rows scanned, rows returned, metric, filter set, cache hit. You will want this before the first incident, not after.

**Deliverables:** rollups populated and backfilled, isolation enforced and tested, per-query telemetry live.

## Weeks 8–9: Internal Release

Ship it to your own support and customer success teams. No customers yet.

This is the step teams skip and it is the highest-value two weeks in the plan. Internal users argue about definitions immediately and loudly, which is exactly what you want — every disagreement found here is a credibility problem that never reaches a customer.

Expect to find: at least one metric where two departments disagree about the definition, at least one timezone bug, and at least one case where a soft-deleted record is being counted.

**Checkpoint:** if your own team does not use it daily by the end of week nine, customers will not either. Do not proceed to rollout — fix the reason first.

![Analytics rollout and validation](https://cdn.sanity.io/images/563mnkns/production/cc4d4337cb584069bfe12940f16d0d633c7ec08f-1600x1022.jpg)

## Weeks 10–11: Limited Customer Rollout

Release to three to five friendly customers behind a feature flag. Choose customers who differ in size and data shape, so you see how the query patterns behave under variety rather than under your median case.

**Instrument usage from day one:** which metrics get opened, which filters get applied, where sessions end, and what the p95 latency looks like per tenant rather than in aggregate.

**Watch the telemetry for the outlier.** The first performance problem will come from one customer's data shape, and you will identify them in minutes with per-tenant query logging and in a week without it.

## Week 12 Onward: Expand on Evidence

Do not return to the original requirements list. Expand based on what the first cohort actually used.

In practice roughly half the originally requested charts go unopened once the core five exist, because the real need was narrower than the stated one. That finding is the return on doing it this way.

The highest-value next item is usually not a chart at all. It is alerting — notice when a metric moves outside its normal range, tell the customer, and name the segment driving it. That removes the requirement to remember the dashboard exists, and it typically outperforms the next ten charts on engagement.

## What This Plan Deliberately Defers

- **Ad-hoc exploration.** A curated metric set is a bounded engineering problem; open-ended querying is a BI product. Decide that separately, on evidence.
- **Custom dashboards per customer.** Expensive, and demanded far less often than expected once the core five are good.
- **Real-time everything.** Ask what action a human takes when the number changes and how quickly. If the answer is "next Tuesday," a scheduled rollup is the correct engineering decision.

## Checkpoints

| Week | Gate |
|---|---|
| 3 | Five metrics defined in code, edge cases written down, latency budget agreed |
| 7 | Isolation enforced in the database with a passing CI test; telemetry live |
| 9 | Internal teams using it daily |
| 11 | p95 within budget for every customer in the cohort, measured per tenant |
| 12 | Expansion decided by usage data, not by the original list |

The full architectural treatment — build versus embed versus buy, tenant isolation patterns, the semantic layer and the three-year cost model — is in our guide to [product analytics dashboards](https://techcirkle.com/blog/product-analytics-dashboards).

## Frequently Asked Questions

### Why spend three weeks defining metrics before writing code?

Because the definitions determine the data model, and changing a definition after the pipeline exists means a rewrite plus a backfill. The three weeks are not overhead; they are the cheapest point at which to have the arguments that would otherwise surface in month four, in front of a customer.

### Is two weeks of internal-only release really necessary?

It is the highest-return step in the plan. Internal users exercise the feature daily and complain immediately, which surfaces definition disagreements and timezone bugs at zero reputational cost. Every one of those found internally is one that never reaches a customer and never damages trust in the numbers.

### What if stakeholders insist on more than five metrics?

Show them the cost driver. Five metrics need a narrow, fast data model; fifty need a general one, and the generality — extra joins, indexes, rollup permutations and a permissions system that scopes anything by anything — is where the schedule goes. Offer the five now and the rest on evidence, which is usually accepted once the trade-off is explicit.

### How do we pick the pilot customers?

Choose three to five that differ in size and data shape rather than the five friendliest. You are testing how query patterns behave under variety, and your largest or most unusual customer is the one that will expose the performance problem you need to find before general release.

### When should we add ad-hoc exploration?

Only after the curated set is in production and you have usage data showing that customers are hitting its limits. Exploration turns a bounded engineering problem into an open-ended one, and in most B2B products the demand for it turns out to be aspirational — instrument the filter combinations customers actually use before committing.
