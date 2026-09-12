---
layout: post
title: "A Field Guide to Compliance System Design"
date: 2026-09-12 00:00:00 +0000
categories: ["Engineering", "Architecture"]
tags: ["architecture", "fintech", "compliance", "engineering"]
description: "An engineer's field guide to building regulated financial systems: data layer, control objects, evidence architecture and where model inference belongs."
image: "https://cdn.sanity.io/images/563mnkns/production/c9718905a6817087167e6739563fda6b5d3ad751-1600x598.jpg"
author: "TechCirkle Editorial Team"
---

![Compliance architecture dashboard](https://cdn.sanity.io/images/563mnkns/production/c9718905a6817087167e6739563fda6b5d3ad751-1600x598.jpg)

Written for engineers who have just been handed a compliance platform and have not worked in a regulated environment before. The constraints here are unusual enough that ordinary good practice will lead you astray in specific, expensive ways.

## The constraint that drives everything

Regulated systems are not judged by whether they work. They are judged by whether they can explain themselves, under adversarial questioning, years after the fact, to someone who was not present.

This single constraint reorders normal engineering priorities. Mutation becomes hostile. Current-state storage becomes a liability. Configuration becomes something requiring approval workflow. And "we can log that later" becomes a permanent loss of capability.

## Layer 1 — The data substrate

**Bitemporal by default.** Every record that can influence a decision carries two time ranges: valid time (when the fact was true in the world) and transaction time (when this system believed it). Regulators ask what you knew, which is transaction time.

```
customer_risk_rating(
  customer_id, rating,
  valid_from, valid_to,
  recorded_from, recorded_to
)
```

Reconstruction is `recorded_from <= T AND recorded_to > T`.

**Never update in place.** A correction is a new row closing the previous one. The moment you `UPDATE`, the history required to explain past decisions is gone and cannot be recovered.

**Reconciliation as a first-class service.** Continuous completeness checks between systems of record and the compliance store, alerting on variance. This is not monitoring hygiene — a feed that silently stops delivering a subset of records produces controls that evaluate on partial data and report green.

**Lineage metadata.** Any field in any decision must be traceable to the system of record that produced it. Budget ten to fifteen percent of the build here; it is the part that prevents the failure nobody plans for.

## Layer 2 — Control objects

A control is a versioned, immutable artefact declaring:

- the obligations it satisfies (bidirectionally traversable)
- the population it applies to
- its logic and thresholds
- its data dependencies
- its approval record and effective-from timestamp

Properties worth enforcing:

1. **Immutability.** Version 7 never changes. A threshold adjustment is version 8.
2. **Alert lineage.** Every alert stores the control version that produced it.
3. **Dependency-driven disablement.** A stale dependency disables the control loudly. This is the cheapest defence against the most common silent failure.
4. **Replay before deploy.** New version runs against historical transactions using historical reference data, producing a concrete delta report attached to the approval.

Replay using *current* reference data is worthless — those ratings changed in response to alerts the old control generated.

## Layer 3 — The decision log

```
decision(
  id, subject_type, subject_id,
  decided_at, decided_by,
  outcome,
  control_version_id,
  policy_version_id,
  inputs_snapshot_ref,
  reasoning_ref,
  model_invocation_id NULL,
  supersedes_decision_id NULL
)
```

Two notes that save considerable pain later:

**Materialise the input snapshot** in addition to being able to reconstruct it. Reconstruction queries are correct in principle and brittle across two years of schema change. The snapshot is ground truth; the query is the cross-check.

**Store reasoning as presented**, not as generated. If a human saw a summarised brief, persist the brief. Persisting only raw model output leaves you unable to show what actually informed the decision.

![Monitoring and analysis screens](https://cdn.sanity.io/images/563mnkns/production/f4489bb7cd60b4ea639c1d395b7c6c51c7417ab0-1600x586.jpg)

## Layer 4 — Where inference belongs

Three positions, in order of governance burden.

**Enrichment (in front of controls).** Entity resolution, counterparty classification, behavioural baselining. Outputs feed rules rather than being decisions, so explainability of the control is preserved. Highest value per unit of governance overhead.

**Triage (behind controls).** Queue prioritisation and context assembly. Cannot cause a miss, because every alert the controls generate still exists — only the order and preparation of human review changes. This is where the forty-minutes-to-eight-minutes reduction lives.

**Auto-closure (behind controls, high governance).** Requires: written approved eligibility policy, complete persisted reasoning trace per closed alert, and continuous statistically meaningful human sampling feeding back. All three or it is an undocumented model operating as a control.

**Never inside a control.** A model adjusting thresholds autonomously is a control change without approval.

Operational requirements for any of these:

- Pin explicit model versions. Floating aliases in a control path mean uncontrolled change.
- Held-out evaluation set as a promotion gate on every version change.
- Persist prompt, retrieved context, model version, parameters and raw output, linked to the decision. Store large context by reference to an immutable document store.

## Sequencing

The counterintuitive part. Roughly:

| Months | Work | Visible output |
|---|---|---|
| 1–3 | Data substrate, ingestion, reconciliation, lineage | None |
| 3–6 | Evidence store, case workflow, one control category migrated | Some |
| 6–9 | Detection layer, versioned controls, replay harness | Yes |
| 9–12 | Triage in advisory mode, regulatory reporting, decommission legacy | Yes |

Inference arrives last and arrives advisory. It is only valuable once its inputs are trustworthy, and only defensible once there is a measured baseline to compare it against.

Teams that invert this build a good demonstration on unreliable data and spend a year discovering why the output is wrong.

## Things that are always underestimated

- **Regulatory reporting.** Every regime has its own schema, validation rules and submission mechanics, specified for lawyers rather than engineers.
- **Reconciliation.** Problems only appear at production data volumes.
- **Analyst workflow.** Looks like UI work, is actually workflow design that determines the operating cost of the entire function. If alerts land outside the tool the team already uses, detection quality is irrelevant.

Full write-up with cost breakdown and integration detail: [Financial Compliance Software: Architecture for AI-Native Controls](https://techcirkle.com/blog/financial-compliance-software). Related: [LLM integration](https://techcirkle.com/llm-integration).

## Frequently Asked Questions

### What is the defining constraint of compliance system design?

That the system must explain its own past decisions under adversarial questioning years later. This makes mutation hostile, current-state-only storage a liability, and configuration something requiring approval workflow — reordering normal engineering priorities.

### Why bitemporal rather than a simple history table?

Because two different time questions matter: when a fact was true in the world, and when the system believed it. Regulators ask the second. A history table tracking only effective dates cannot distinguish a backdated correction from what you actually knew at decision time.

### Where should model inference sit in the architecture?

As enrichment feeding controls, and as triage behind them. Never inside a control, since that would be a control changing without approval. Enrichment offers the best value per unit of governance burden; auto-closure offers the most saving and carries the heaviest requirements.

### What is the most common silent failure?

A control evaluating against stale or partial data while continuing to report normally. Declaring data dependencies per control so a stale feed disables it loudly is the cheapest and most frequently omitted defence.

### Why does the inference layer come last?

Because context assembly over unreliable data produces confident and wrong output, at which point humans stop verifying. Inference is only valuable once inputs are trustworthy and only defensible once a measured human baseline exists to compare against.

### What percentage of the build is data work?

Around thirty percent for ingestion and reconciliation, plus ten to fifteen percent for lineage infrastructure. The first three months typically produce nothing user-visible, which is the phase most often cut and the most common cause of failure.
