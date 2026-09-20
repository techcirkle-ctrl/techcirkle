---
layout: post
title: "An Entity Checklist for Property Development Systems"
date: 2026-09-20 00:00:00 +0000
categories: ["Architecture", "Data Modelling"]
tags: ["architecture", "datamodeling", "proptech", "checklist", "ai"]
description: "A concrete schema checklist for development software: assets, stage gates, commitments, conditional dependencies, ownership and document provenance."
image: "https://cdn.sanity.io/images/563mnkns/production/f6d550af3539b835847ddf253023565909cebf80-1600x1059.jpg"
author: "TechCirkle Editorial Team"
---

![Construction engineers reviewing a development project on a tablet at a building site](https://cdn.sanity.io/images/563mnkns/production/f6d550af3539b835847ddf253023565909cebf80-1600x1059.jpg)

This is a schema checklist for whoever implements a property development system, as opposed to whoever selects one. Each item exists because omitting it produces a specific, predictable class of manual work later.

## Asset — the durable root

The asset outlives every project performed on it.

- [ ] `Asset` exists independently of `Project`
- [ ] One asset supports many projects over time (acquisition, redevelopment, disposition)
- [ ] Location, title and physical characteristics hang off the asset, not the project
- [ ] History survives project closure — including documents and approvals

Getting this wrong means losing the record of what was done to a building the moment the project that did it is archived.

## StageGate — an entity, not a status

- [ ] Stage gates are objects with identity, not an enum on the project
- [ ] Each carries its own `conditions[]`, `approvals[]` and `evidence[]`
- [ ] Approvals are append-only; corrections supersede rather than edit
- [ ] Every approval records approver, role and the authority under which it was given
- [ ] Delegation of authority is stored as data with validity periods
- [ ] The system can answer: was this approval valid at the moment it was given?

That last check is the one auditors and lenders actually exercise, and it is impossible if authority exists only as an organisational convention.

## Dependency — conditional, not temporal

- [ ] Dependencies trigger on events, not elapsed durations
- [ ] Event conditions cover approvals, document receipt, inspections and evidenced conditions
- [ ] Composite conditions are supported (all-of, any-of)
- [ ] Time estimates exist for planning but never transition state
- [ ] Downstream milestones move on the event firing, never on estimate expiry

If an estimate can change state, the durations model is back and the schedule will be wrong from day one.

## Commitment — one record, three projections

- [ ] Cost, schedule impact and draw impact live on a single commitment record
- [ ] Budget reports, programme views and draw requests are projections over commitments
- [ ] A change order is one write that updates all three views
- [ ] Commitments carry counterparty and evidence references
- [ ] Superseded commitments remain queryable for history

This single decision removes the largest recurring reconciliation burden in the domain.

## Ownership — a reporting dimension from day one

- [ ] Legal entity and ownership structure modelled explicitly
- [ ] Reports parameterised by reporting perspective over one dataset
- [ ] No duplication of projects to serve different reporting views
- [ ] Capital source tracked as a dimension, not a note

Duplicating projects per reporting view is the common shortcut, and the duplicates diverge within a quarter.

## Documents — provenance over folders

- [ ] Documents attach to the entity they evidence, not a folder path
- [ ] Full version history retained and referenceable
- [ ] Approvals reference a specific document version, not the latest
- [ ] Provenance survives supersession

The recurring dispute is always about which drawing revision an approval referenced. A folder tree cannot answer that.

## Observations — append-only field data

- [ ] Client-generated UUIDs so records are referenceable before sync
- [ ] Append-only; current state derived by folding observations
- [ ] Device clock skew corrected at sync, since timestamps are evidential
- [ ] Media queued separately from metadata
- [ ] Upload idempotent on client UUID

Last-write-wins is wrong here: two inspectors observing different things are not a conflict.

## Reporting dimensions — decide before the first migration

- [ ] Entity, stage, asset class, capital source, jurisdiction all present in the model
- [ ] The three most painful hand-assembled reports reproducible with zero manual steps

Retrofitting a dimension into a live system is the most expensive change in this list, and the one most commonly required in year two.

## Intelligence layer — sequence it last

- [ ] Extraction targets a defined schema (`Commitment`, `Condition`, `Evidence`)
- [ ] Human verification on each extracted record rather than blind ingestion
- [ ] Variance detection deferred until consistent history exists
- [ ] Agent actions logged with inputs, context, reasoning and outcome
- [ ] Anything financially or legally consequential requires explicit human confirmation

Extraction first because the model gives it a target. Detection later because it needs history.

## Frequently Asked Questions

### Why separate asset from project?
Because one asset undergoes several projects over a decade and the record of what was done must survive project closure. Collapsing them loses the building's history when the project is archived.

### Why must approvals be append-only?
Because the question later is what was approved at a specific past moment and by whom. Editable approvals cannot answer that, and lenders, auditors and partners all ask it.

### What does modelling delegation of authority achieve?
It lets the system determine whether an approval was valid when it was given. An approval beyond someone's authority is a finding, and it is undetectable when authority is only a convention.

### Why should dependencies trigger on events?
Because the critical dependencies in development — permits, inspections — have no predictable duration. Estimates must stay advisory; allowing one to transition state reintroduces a schedule that is wrong from the outset.

### What does the single commitment record solve?
It removes reconciliation between cost, schedule and capital draw by making them projections of one fact. A change order becomes a single write updating all three views.

### Why sequence the intelligence layer last?
Extraction needs a defined target schema, and variance detection needs consistent history. Building either before the model and data exist produces confident output over sparse data.

---

The full guide, with cost analysis and build sequencing: [Real Estate Project Management Software: The 2026 Build Guide](https://techcirkle.com/blog/real-estate-project-management-software)

TechCirkle: [custom software development](https://techcirkle.com/development/custom-software-development) | [AI development services](https://techcirkle.com/ai-development-services)
