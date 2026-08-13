---
layout: post
title: "A CI Pipeline for Analytics That Catches Bad Data Early"
date: 2026-08-13 00:00:00 +0000
categories: ["Data Engineering", "DevOps"]
tags: ["ci-cd", "dbt", "data-engineering", "testing"]
description: "Analytics code needs the same CI discipline as application code. Here is a pipeline that catches broken data before anyone builds a dashboard on it."
image: "https://cdn.sanity.io/images/563mnkns/production/a22cad78e1cc7b9644086e23304d7f80d8d8a919-1600x900.jpg"
author: "TechCirkle Editorial Team"
---

![Data analyst reviewing a business analytics dashboard showing charts, metrics and KPIs](https://cdn.sanity.io/images/563mnkns/production/a22cad78e1cc7b9644086e23304d7f80d8d8a919-1600x900.jpg)

Analytics code gets held to a standard that would be unacceptable anywhere else in engineering. SQL is reviewed by eye, deployed manually, and validated by whether a chart looks approximately right. Breakages are discovered by a stakeholder asking why a number moved.

The fix is not exotic. It is the CI discipline you already apply to application code, adapted for the fact that your inputs change without you deploying anything.

## The distinguishing constraint

Application CI assumes inputs are stable and code changes. Analytics CI has to assume the opposite as well: your code can be untouched for a month and still start producing wrong output, because an upstream system changed a field, a vendor altered an enum, or a backfill rewrote history.

So the pipeline needs two triggers — on pull request, and on schedule against production data.

## Stage 1 — Static checks on PR

Cheap, fast, run on every change.

```yaml
name: analytics-ci
on: [pull_request]

jobs:
  static:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Lint SQL
        run: sqlfluff lint models/ --dialect snowflake
      - name: Check for undocumented models
        run: dbt-checkpoint check-model-has-description --all
      - name: Enforce test coverage on keys
        run: dbt-checkpoint check-model-has-tests-by-name --tests unique not_null
```

The third step matters more than it looks. Requiring that every model has uniqueness and not-null tests on its keys prevents the most common category of silent breakage — a join that starts fanning out and inflating every sum downstream.

## Stage 2 — Build and test against a slice

Full production builds on every PR are too slow and too expensive. Build the modified models and their downstream dependents against a sampled dataset.

```yaml
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build changed models and children
        run: |
          dbt build \
            --select state:modified+ \
            --defer --state ./prod-manifest \
            --target ci
```

The `state:modified+` selector plus deferral is the key economy: build only what changed and what depends on it, resolving everything else against production. A three-hour full build becomes six minutes.

## Stage 3 — Assertions that encode business rules

Generic tests catch structural problems. They do not catch a revenue figure that is technically valid and commercially impossible.

```sql
-- tests/assert_no_negative_net_revenue.sql
select date_day, net_revenue
from {{ ref('mart_revenue_daily') }}
where net_revenue < 0
  and date_day >= dateadd(day, -90, current_date)
```

```sql
-- tests/assert_customer_counts_stable.sql
with daily as (
  select date_day, count(distinct customer_id) as n
  from {{ ref('fct_activity') }}
  group by 1
),
changes as (
  select date_day, n,
         lag(n) over (order by date_day) as prev
  from daily
)
select * from changes
where prev is not null
  and abs(n - prev) > prev * 0.4
```

That second one — flagging day-over-day swings beyond a threshold — catches more real incidents than any schema test. A pipeline that silently drops half its rows produces perfectly valid output with a plausible shape.

![Team reviewing business growth data and financial charts together in a meeting](https://cdn.sanity.io/images/563mnkns/production/d9f1fa29ddc3f7b5f6181f2985f8c6b70cf8572e-1600x900.jpg)

## Stage 4 — Scheduled production checks

This is the stage most teams skip and the one that catches upstream drift.

```yaml
on:
  schedule:
    - cron: '0 */4 * * *'

jobs:
  freshness:
    steps:
      - name: Source freshness
        run: dbt source freshness
      - name: Run tests against production
        run: dbt test --select source:* tag:critical
```

Source freshness deserves particular attention. Stale data is more dangerous than missing data — missing data announces itself, stale data answers confidently about the wrong period. Set an explicit warn and error threshold per source rather than a global default, because a daily batch and a streaming source have nothing in common.

## Stage 5 — Fail visibly, in the right channel

A failed test that emails a shared mailbox is a failed test nobody sees.

Route failures to the channel where the owning team already works, and include enough context to triage without opening the warehouse: which model, which assertion, how many rows, and the last successful run. Tag critical models so that a failure on a board-reported metric pages someone, while a failure on an exploratory model files a ticket.

## What this costs and what it buys

Roughly a week to set up properly, plus ongoing discipline in reviews. It is not free.

What it buys is the difference between finding out about a data problem from your pipeline and finding out from a stakeholder. That difference is mostly about trust rather than time, and trust in an analytics platform is close to binary — one wrong number in a consequential room costs you confidence in every other number you have produced.

This matters more now than it did two years ago. When every question went through an analyst, that analyst caught anomalies before they reached anyone. With natural-language querying, business users hit the warehouse directly and a broken mart produces a confident wrong answer with nobody in between.

The CI pipeline is what replaces that human check.

---

The full buyer-oriented guide — what data analytics services include, cost ranges, engagement models, and vendor evaluation questions — is here: **[Data Analytics Services: A 2026 Guide for Technical Buyers](https://techcirkle.com/blog/data-analytics-services)**.

TechCirkle builds data platforms alongside broader [custom software development](https://techcirkle.com/development/custom-software-development) work.

## Frequently Asked Questions

### Why does analytics CI need scheduled runs as well as PR runs?

Because analytics code can be untouched and still produce wrong output when an upstream system changes a field, alters an enum, or rewrites history in a backfill. PR-triggered runs only catch problems you introduced.

### How do you keep CI builds fast on a large warehouse?

Build only modified models and their downstream dependents, deferring everything else to production state. A full build measured in hours becomes a targeted build measured in minutes.

### What tests catch more incidents than schema tests?

Volume-change assertions. Flagging day-over-day swings beyond a threshold catches pipelines that silently drop rows, which produce structurally valid output that passes every uniqueness and not-null check.

### Why is source freshness so important?

Stale data is more dangerous than missing data. Missing data announces itself; stale data answers confidently about the wrong period. Set thresholds per source, since batch and streaming sources have nothing in common.

### Should every model have the same alerting severity?

No. Tag critical models — those feeding board-reported metrics — so failures page someone, while failures on exploratory models file a ticket. Uniform alerting trains people to ignore alerts.

### Why has analytics CI become more important recently?

Analysts used to catch anomalies before numbers reached anyone. With natural-language querying, users hit the warehouse directly and a broken mart returns a confident wrong answer with no human in between.
