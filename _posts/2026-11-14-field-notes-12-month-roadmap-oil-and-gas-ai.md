---
layout: post
title: "Field Notes: A 12-Month Roadmap for Oil and Gas AI"
date: 2026-11-14 00:00:00 +0000
categories: ["Engineering Notes", "Industrial AI"]
tags: ["oilandgas", "ai", "roadmap", "mlops", "checklist"]
description: "Engineering field notes on a 12-month oil and gas AI roadmap: four stages, with entry and exit checks for selection, proof, hardening and scale."
image: "https://cdn.sanity.io/images/563mnkns/production/73773c246e0b3bd7607e7a9d814ab659ba9c5571-1600x959.jpg"
author: "TechCirkle"
---

These notes assume an operator with historians and maintenance systems already in place, a small mixed team, and a mandate to get one AI use case into production and a second one started within a year. Each stage has entry checks, work items and an exit gate. A stage ends with a decision. If you cannot answer the exit question, you have not finished the stage, whatever the calendar says.

![A hand holding a tablet that projects a glowing AI brain icon surrounded by gears](https://cdn.sanity.io/images/563mnkns/production/73773c246e0b3bd7607e7a9d814ab659ba9c5571-1600x959.jpg)

## Ground rules

- Advisory mode only. Nothing in this plan writes a setpoint.
- The safety instrumented system is independent of every model, permanently.
- Data leaves the control network through a one-way or brokered path into a DMZ. No direct connections.
- Build only the data foundation the current use case needs.
- Every model has a named owner before it goes live.

## Stage 1: Weeks 0 to 6, select

**Entry checks**

- [ ] A sponsor with budget authority is named
- [ ] At least one production, process or maintenance engineer is assigned to the team
- [ ] You have read access, or a route to it, for the historian and the document stores

**Work items**

- [ ] List two or three candidate use cases
- [ ] Score each on data availability, value and likelihood of adoption
- [ ] Interview the engineers who would use the output, and record what they would do differently with it
- [ ] Pick one sensor-based use case and one document-based use case
- [ ] Choose the single asset for the proof of value
- [ ] Agree the outcome metric and write down the current baseline

**Note:** the sensor and document pairing is deliberate. They de-risk different parts of the stack. One exercises the OT data path and time-series quality. The other exercises parsing, metadata, access control and retrieval evaluation.

Good first candidates are artificial lift surveillance, rotating equipment health, and a retrieval assistant over well files and procedures. Poor ones are autonomous drilling and closed-loop anything.

**Exit gate:** can you name the use case, the asset, the user and the baseline number on one page? If yes, proceed.

## Stage 2: Week 6 to month 4, prove

**Entry checks**

- [ ] Baseline is signed off by operations, not only by the project team
- [ ] Data extraction path is approved by OT security
- [ ] Historical events (failures, trips, workovers) are available to validate against

**Work items**

- [ ] Build the asset model for this asset only: tags to equipment, equipment to well, station or unit
- [ ] Add data quality checks for frozen sensors, drift and gaps ahead of the model
- [ ] Join time series to events from the maintenance and workover records
- [ ] Train and backtest against historical events
- [ ] Run in shadow mode alongside current practice
- [ ] Log every model call and compare it with what actually happened
- [ ] Attach evidence to every recommendation: the signals, the similar cases, the source page
- [ ] For the document use case, build an evaluation set of real questions with known answers and check citations

**Note:** where the asset moves outside its training conditions, pure data-driven models extrapolate badly. Consider a hybrid: first-principles equations with machine learning correcting the residual. It is also easier to defend in front of a sceptical engineer.

**Note:** for retrieval, the wrong revision of a procedure is a worse outcome than no answer. Version metadata is a requirement, not a refinement.

**Exit gate:** would the field team miss it if it were switched off? Ask them directly. Accuracy figures do not answer this.

## Stage 3: Months 4 to 8, harden

This is where most of the engineering effort goes, and it is the stage pilots skip.

**Entry checks**

- [ ] Shadow-mode results reviewed with the users and accepted
- [ ] Funding covers integration, monitoring and support, not only the model
- [ ] A model owner is named

**Work items**

- [ ] Integrate with the maintenance or production system (work order drafts, approvals, audit trail)
- [ ] Human approval step on every action that writes to a system of record
- [ ] Monitoring for input drift and output quality, with alert thresholds
- [ ] Retraining procedure, documented and tested once end to end
- [ ] Security review: treat model artefacts like any other software release
- [ ] Restrict what an LLM-based assistant can retrieve to the user's existing permissions
- [ ] Decide which documents may ever be sent to an external model provider, and whether private deployment is required
- [ ] Management of change completed, with a rule that each new model version goes through it
- [ ] Edge deployment where latency or connectivity requires it, with store-and-forward for retraining data
- [ ] User training and a support route

![A business person using a smartphone surrounded by floating analytics and strategy icons](https://cdn.sanity.io/images/563mnkns/production/29c3888beae03adcddfadbc8a4f9719421437a72-1600x1022.jpg)

**Note:** sensors get replaced, wells decline, crude slates change. A model that was right at commissioning will drift. Silent failure is the expensive kind, so monitoring is part of the build.

**Exit gate:** is the system used in daily work without the project team in the room, and can you show the outcome metric moving against the baseline?

## Stage 4: Month 8 onward, scale

**Entry checks**

- [ ] Measured result against baseline is documented
- [ ] Runbook exists for onboarding a new asset
- [ ] Run cost (compute, monitoring, retraining, support) is in the operating budget

**Work items**

- [ ] Extend the asset model to the next field, station or unit
- [ ] Record what differed: tag naming, historian gaps, missing equipment mappings
- [ ] Start the second and third use cases on the same foundation
- [ ] Track effort per new asset and per new use case
- [ ] Review ownership: code, models and pipelines should sit with the operator

**Note:** by now the asset model, the quality layer and the deployment pattern exist. Each new use case should cost less than the one before. If it does not, the foundation is not being reused, and that is worth a retrospective.

**Exit gate:** is the marginal cost of the next use case falling? That trend is the argument for running AI as a programme and not as a series of experiments.

## Things to refuse to skip

- The baseline. Without it the gain is an opinion.
- The operations engineer on the team. Without one the model is ignored.
- Shadow mode. Without it nobody has grounds to trust the output.
- Stage 3. Without it you have a demo.

Much of this roadmap is ordinary [custom software development](https://techcirkle.com/development/custom-software-development) with models inside it, and the document track is mostly careful [LLM integration](https://techcirkle.com/llm-integration). Background on use cases, architecture, costs and failure modes is in our [full guide to AI in the oil and gas industry](https://techcirkle.com/blog/ai-in-oil-and-gas-industry).

## Frequently Asked Questions

### Why pick both a sensor use case and a document use case at the start?

They test different parts of the stack. The sensor case proves the OT data path and time-series quality. The document case proves parsing, access control and retrieval. Together they expose most integration risk early.

### What is shadow mode and how long should it run?

Shadow mode means the model runs alongside current practice without influencing it, while its calls are compared with real outcomes. It should run long enough to cover the events you care about on that asset.

### What is the exit test for a proof of value?

Whether the field team would miss the system if it were switched off. That question measures adoption and usefulness, which accuracy metrics alone cannot show.

### Do we need a full data platform before stage 1?

No. Build the asset model, quality checks and pipelines for one use case on one asset, then extend them. A platform with no user tends not to survive budget review.

### When is it appropriate to move beyond advisory mode?

Not within this twelve-month plan. Writing to a control system requires formal engineering, safeguards and management of change, and belongs to a later, separately justified project.
