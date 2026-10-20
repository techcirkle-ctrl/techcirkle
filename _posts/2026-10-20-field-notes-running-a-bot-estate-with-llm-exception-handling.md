---
layout: post
title: "Field Notes: Running a Bot Estate With LLM-Assisted Exception Handling"
date: 2026-10-20 00:00:00 +0000
categories: ["Engineering Notes", "Automation"]
tags: ["rpa", "sre", "llm", "automation"]
description: "Engineering notes on operating an RPA estate with LLM-assisted exceptions: incident patterns, metrics, runbooks and lessons from production."
image: "https://cdn.sanity.io/images/563mnkns/production/74d2cc943181b89cb6a946b06152f124ef1aba18-1600x751.jpg"
author: "TechCirkle"
---

![Hand pressing an RPA button on a glowing control screen](https://cdn.sanity.io/images/563mnkns/production/74d2cc943181b89cb6a946b06152f124ef1aba18-1600x751.jpg)

These are working notes on the run side of robotic process automation: what happens after go-live, when a few dozen bots are in production and someone has to keep them healthy. They complement the buyer-level view in TechCirkle's [guide to robotic process automation consulting services](https://techcirkle.com/blog/robotic-process-automation-consulting-services), which covers scoping and partner selection.

## 1. Incident taxonomy

Classify every bot failure into one of five buckets. Without this, post-mortems turn into anecdotes.

| Bucket | Typical trigger | Share of incidents (typical) |
|---|---|---|
| UI drift | Target app or portal release changes selectors | 35-50% |
| Rule change | Policy or business rule updated, bot not told | 15-25% |
| Infra | OS patch, browser update, VM image drift | 10-20% |
| Credentials | Password expiry, MFA change, locked account | 10-15% |
| Data | Unexpected input shape, upstream data quality | 5-15% |

UI drift dominates for estates heavy on third-party web portals. That is the bucket LLM-assisted field location most affects.

## 2. Metrics worth tracking

- **Incidents per bot per month.** Healthy estates sit well below 1; struggling ones see 10-20% of bots needing a fix every month.
- **MTTR** (mean time to repair), split by bucket.
- **Straight-through rate** per process, measured weekly.
- **Exception handling time**: minutes of human effort per exception.
- **Auto-apply rate and override rate** for the LLM resolver: how often the model's fix was applied automatically, and how often sampled reviews disagreed.

The last pair is new. If the override rate on sampled reviews creeps above a few percent for any reason code, disable auto-apply for that code until the regression set is updated.

![Monitoring dashboard showing bot health metrics and exception rates](https://cdn.sanity.io/images/563mnkns/production/bd2c7b25e7074a966cbccb74a793fe0364f7170d-1600x1270.jpg)

## 3. What the resolver changed

Adding an LLM step at the bot's exception points had three measurable effects on estates we have worked with:

1. Straight-through rate on document-heavy processes rose from the 70s into roughly 85-92%.
2. Human handling time on remaining exceptions dropped to about a minute, because the suggested fix arrives pre-filled.
3. UI-drift incidents fell for steps that switched from selector-based to meaning-based field location.

It also introduced a new failure class: **silent wrong answers**. Unlike a selector miss, these do not raise an incident. The only detection mechanism is sampling, so the weekly sample review is not optional.

## 4. Runbook entries we always write

- **Selector failure on portal X:** check the portal's release notes, capture a DOM snapshot, compare with last known good, patch, rerun failed queue.
- **Credential lockout:** rotate from vault, never from a local config; confirm the bot's dedicated service identity, not a shared one.
- **Resolver timeout or schema violation:** fail closed to the human queue; alert only if the rate exceeds baseline.
- **Model provider version change:** freeze auto-apply, run regression set of historical exceptions, compare agreement per reason code, then re-enable.

## 5. Day-one setup we would not skip again

- One named service identity per bot, least privilege, credentials vaulted.
- Bots in version control with peer review and a rollback path.
- A single audit stream recording, for every transaction: bot, inputs, outputs, and for AI steps the prompt version, model version, output and confidence.
- A regression set of a few hundred real historical exceptions with their human resolutions.
- Regional processing for model calls where data residency applies.

## 6. Cost notes

Screen-bot maintenance landed in the usual 20-40% of build cost per year. The resolver added inference cost (fractions of a cent to a few cents per call) and roughly a few hours a week of sample review. For document-heavy processes the net was lower total run cost. For simple, stable screen tasks the resolver was not worth adding.

## 7. If you are hiring help

Ask any prospective partner for their incident taxonomy and MTTR numbers. Teams who run bots for a living have them. If you want to discuss the agent side of this setup, TechCirkle's [agentic workflow development](https://techcirkle.com/agentic-workflow-development) page has more detail.

## Frequently Asked Questions

### What causes most RPA bot incidents?

UI drift in target applications is usually the largest single cause, followed by business rule changes, infrastructure updates, credential problems and unexpected data.

### How do you monitor an LLM step inside an RPA workflow?

Track auto-apply and override rates per reason code, log every input and output with model and prompt versions, and review a sample of automated decisions weekly.

### What should happen when the LLM resolver fails?

It should fail closed: timeouts, invalid output or low confidence all route the item to the human queue, so the process is never worse than before.

### What is a healthy incident rate for a bot estate?

Well under one incident per bot per month. Estates where 10-20% of bots need fixes every month usually have selector-heavy designs or weak change control.

### Do I need a regression set for AI steps?

Yes. A labelled set of historical exceptions and their human resolutions is the only reliable way to validate prompt changes and model upgrades.
