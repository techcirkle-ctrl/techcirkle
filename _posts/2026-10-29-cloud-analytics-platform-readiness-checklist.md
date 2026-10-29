---
layout: post
title: "Cloud Analytics Platform Readiness Checklist (Cost, Governance, AI)"
date: 2026-10-29 00:00:00 +0000
categories: ["Data Engineering", "Checklists", "Cloud"]
tags: ["cloud-analytics", "checklist", "data-engineering", "finops", "governance"]
description: "A working checklist for analytics in the cloud: architecture, cost controls, data quality, semantic layer, AI agent access, security and residency."
image: "https://cdn.sanity.io/images/563mnkns/production/cdd95aef90115bf03c5c0d29a43c8ad53dda54a0-1600x1067.jpg"
author: "TechCirkle"
---

![A team reviewing cloud analytics dashboards during a platform review](https://cdn.sanity.io/images/563mnkns/production/cdd95aef90115bf03c5c0d29a43c8ad53dda54a0-1600x1067.jpg)

This page is a working checklist we use when auditing an existing platform for analytics in the cloud or scoping a new one. It is deliberately terse. Mark each item **Done**, **N/A (with reason)** or **Open**, and treat any Open item in sections 2, 4 or 5 as a blocker before connecting AI assistants to your data.

## 1. Architecture

| # | Item | Why it matters |
|---|---|---|
| 1.1 | Storage and compute are separated; workloads run on independent compute | Heavy jobs never slow executive dashboards |
| 1.2 | Raw data lands untouched and is retained | You can reprocess when logic changes |
| 1.3 | Storage uses a warehouse or open table format (Iceberg, Delta) by deliberate choice | Limits lock-in; documents the trade-off |
| 1.4 | Each layer (ingest, store, transform, semantic, serve, observe) is replaceable | Vendor changes do not force a rewrite |
| 1.5 | Twelve-month workload model, including AI query growth, done before any multi-year commit | Avoids buying on a demo |

## 2. Cost governance (FinOps)

| # | Item | Why it matters |
|---|---|---|
| 2.1 | Every warehouse, cluster, job and service account tagged by team, product, env | Spend has an owner |
| 2.2 | Monthly cost report sent to each budget owner | Waste becomes visible |
| 2.3 | Resource monitors and query timeouts per workload class | Runaway queries are contained |
| 2.4 | Auto-suspend on idle compute (60-120 s) | Idle time is not billed |
| 2.5 | Large models are incremental, with periodic full refresh | Usually the single biggest saving |
| 2.6 | Separate capped budget for AI-generated queries | Agent growth cannot surprise finance |
| 2.7 | Top 20 expensive queries reviewed fortnightly | A few queries drive most of the bill |
| 2.8 | Unused dashboards culled quarterly (no views in 60-90 days) | Each refresh costs compute |
| 2.9 | A unit metric (cost per 1,000 queries or per active consumer) tracked monthly | Leadership can manage it |

## 3. Transformation and data quality

| # | Item | Why it matters |
|---|---|---|
| 3.1 | Transformations in version control with code review | They are application code |
| 3.2 | Tests on keys, nulls, accepted values and relationships | Breakage caught before users see it |
| 3.3 | CI runs tests on every change; failing tests block merge | Prevents silent regressions |
| 3.4 | Freshness alerts on critical sources | Stale data is flagged, not discovered |
| 3.5 | Known quality issues (duplicate customers, product codes, broken tracking) logged and budgeted | Typically a quarter of first-project effort |

## 4. Semantic layer

| # | Item | Why it matters |
|---|---|---|
| 4.1 | 15-25 core business metrics defined in one governed place | One number per question |
| 4.2 | Each metric has an owner, definition, grain, valid dimensions and caveats | Humans and models read the same truth |
| 4.3 | BI tools, notebooks and assistants all read from it | No parallel formulas |
| 4.4 | Metric descriptions written for machine readers | Directly drives AI answer accuracy |
| 4.5 | Common metric results cached and keyed by data role | Lower cost without leaking rows |

![A data centre interface showing cloud infrastructure for analytics workloads](https://cdn.sanity.io/images/563mnkns/production/86e34c9a9e83249a3a6536f8c05f661ba5e66b9c-1600x949.jpg)

## 5. AI and agent access

| # | Item | Why it matters |
|---|---|---|
| 5.1 | Agents query governed metrics, not raw tables, by default | Removes most hallucinated numbers |
| 5.2 | Agent sessions run as the requesting user (row-level security applies) | Bots see only what the person may see |
| 5.3 | Every question, generated query, data touched and answer is logged | Audit trail, debugging, evaluation set |
| 5.4 | Evaluation set of real questions with verified answers runs on every model, prompt or schema change | Accuracy drift is caught |
| 5.5 | Questions outside the metric catalogue are logged as backlog, not answered with free SQL | Gaps become new metrics |
| 5.6 | Any agent that takes actions has confirmations and an owner | Required before [agentic workflows](https://techcirkle.com/agentic-workflow-development) go live |

## 6. Security, compliance and residency

| # | Item | Why it matters |
|---|---|---|
| 6.1 | Region chosen deliberately; dataset locations documented | EU, UK and Middle East obligations |
| 6.2 | Encryption at rest and in transit; customer-managed keys for sensitive domains | Baseline control |
| 6.3 | Identity through SSO; no shared service accounts | Access is attributable |
| 6.4 | Personal data masked or tokenised outside production | Reduces exposure |
| 6.5 | All access, including agent access, logged and retained for audit | Satisfies reviewers |
| 6.6 | Written, technically enforced policy on sending data to external model APIs | Policy by code, not memo |

## 7. Ownership and handover

| # | Item | Why it matters |
|---|---|---|
| 7.1 | Named analytics engineer(s) sized to active consumers | Someone maintains it |
| 7.2 | Runbooks for incidents, cost spikes and source changes | Faster recovery |
| 7.3 | Documentation and training handed to the internal team | No permanent dependency on a vendor |

## Using this checklist

Run it at discovery, before connecting the first AI consumer, and every six months. Rationale, architecture diagrams, cost ranges and a 90-day roadmap are in [TechCirkle's full guide to analytics in the cloud](https://techcirkle.com/blog/analytics-in-the-cloud). If you want help closing the Open items, [contact our team](https://techcirkle.com/contact-us).

## Frequently Asked Questions

### When should this checklist be used?

At discovery for a new platform, before connecting any AI assistant to data, and on a regular six-monthly review.

### Which sections are blockers for AI assistants?

Cost governance, semantic layer and AI access.

### Does every item apply to small companies?

No. Mark items N/A with a reason. Smaller teams often defer customer-managed keys or complex residency work, but should not skip tagging or definitions.

### How long does an audit against this list take?

Typically one to two weeks for a mid-sized estate.

### What is the most commonly Open item?

Cost attribution by tag, closely followed by a governed semantic layer.

### Is this checklist vendor-specific?

No. It applies to any major warehouse or lakehouse.
