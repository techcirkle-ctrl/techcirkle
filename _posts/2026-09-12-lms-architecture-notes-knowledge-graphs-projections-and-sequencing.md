---
layout: post
title: "LMS Architecture Notes: Knowledge Graphs, Projections and Sequencing"
date: 2026-09-12 00:00:00 +0000
categories: ["Engineering", "Architecture"]
tags: ["architecture", "edtech", "eventsourcing", "ai"]
description: "An engineering reference for learning platform architecture — knowledge graph schema, event-sourced learner model projections, and sequencing policy."
image: "https://cdn.sanity.io/images/563mnkns/production/bf7a88a9f6c1b61c17d46c7ba1ddbb247f42eb78-1600x900.jpg"
author: "TechCirkle Editorial Team"
---

![Learning platform interface](https://cdn.sanity.io/images/563mnkns/production/bf7a88a9f6c1b61c17d46c7ba1ddbb247f42eb78-1600x900.jpg)

A reference for engineers building the adaptive core of a learning platform. Assumes you have decided you need one — the case for that is elsewhere. This covers how to build it.

## The three-layer shape

```
    events (immutable)
        |
        v
  learner model (projection)
        |
        v
  sequencing (policy over projection)
```

Everything else — delivery surfaces, authoring, administration — sits around this core and is comparatively conventional. Getting these three right determines whether the platform can make outcome claims.

## Layer 1 — Knowledge graph

```sql
CREATE TABLE concept (
  id          uuid PRIMARY KEY,
  domain_id   uuid NOT NULL,
  name        text NOT NULL,
  description text
);

CREATE TABLE concept_prerequisite (
  concept_id      uuid NOT NULL,
  prerequisite_id uuid NOT NULL,
  strength        real NOT NULL CHECK (strength BETWEEN 0 AND 1),
  PRIMARY KEY (concept_id, prerequisite_id)
);

CREATE TABLE item_concept (
  item_id    uuid NOT NULL,
  concept_id uuid NOT NULL,
  weight     real NOT NULL DEFAULT 1.0,
  PRIMARY KEY (item_id, concept_id)
);
```

Design constraints worth enforcing:

- **Acyclic.** Enforce it. A cycle in prerequisites makes gating undecidable and is usually a sign the decomposition is wrong.
- **Granularity.** A concept should be masterable in roughly 15–40 minutes and support 3–5 distinct items. This is the heuristic that keeps the model both discriminating and evidenced.
- **Versioned.** The graph will change as evidence accumulates about what actually discriminates. Version it, because changing it invalidates projections computed against the old structure.

This artefact requires domain experts and cannot be generated. It is the bottleneck and it blocks everything downstream, so schedule it first.

## Layer 2 — Event log and projection

```sql
CREATE TABLE learning_event (
  id             bigserial PRIMARY KEY,
  learner_id     uuid NOT NULL,
  occurred_at    timestamptz NOT NULL,
  event_type     text NOT NULL,
  concept_ids    uuid[] NOT NULL,
  item_id        uuid,
  payload        jsonb NOT NULL,
  schema_version int NOT NULL
) PARTITION BY RANGE (occurred_at);

CREATE TABLE mastery_projection (
  learner_id                uuid NOT NULL,
  concept_id                uuid NOT NULL,
  p_mastery                 real NOT NULL,
  last_evidence_at          timestamptz NOT NULL,
  reinforcement_count       int NOT NULL DEFAULT 0,
  model_version             text NOT NULL,
  graph_version             text NOT NULL,
  computed_through_event_id bigint NOT NULL,
  PRIMARY KEY (learner_id, concept_id)
);
```

The `model_version` and `graph_version` columns are what make this architecture worth the complexity. When either changes, projections become stale in a detectable way and can be rebuilt per learner on next access, with a background drain for the rest.

Update rule, standard BKT:

```python
def update(prior, correct, p):
    if correct:
        post = (prior * (1 - p.slip)) / \
               (prior * (1 - p.slip) + (1 - prior) * p.guess)
    else:
        post = (prior * p.slip) / \
               (prior * p.slip + (1 - prior) * (1 - p.guess))
    return post + (1 - post) * p.learn
```

Propagation to prerequisites, damped:

```python
DAMPING, MAX_HOPS = 0.3, 2
```

Undamped propagation over a dense graph inflates mastery domain-wide on minimal evidence.

Decay, applied lazily at read:

```python
def current_mastery(rec, now, base_half_life):
    hl = base_half_life * (1 + rec.reinforcement_count * 0.5)
    elapsed = (now - rec.last_evidence_at).total_seconds()
    return max(rec.p_mastery * 0.5 ** (elapsed / hl), FLOOR)
```

Lazy evaluation avoids a background job over every learner-concept pair, which at a hundred thousand learners and a few hundred concepts is a large cross product to touch daily.

![Workplace learning session](https://cdn.sanity.io/images/563mnkns/production/5febd7988637fbd2a8e3d4c29aba36fdf672bcb6-1600x1112.jpg)

## Layer 3 — Sequencing

Sequencing is policy over the projection. Keep it explainable — the output should carry a reason string, both for learner-facing explanation and for debugging cohorts that get stuck.

```python
def next_activity(learner, now):
    m = mastery_map(learner, now)

    # 1. Review anything decayed below threshold
    due = [c for c in m if m[c] < REVIEW_THRESHOLD
                        and was_previously_mastered(learner, c)]
    if due:
        return review_item(max(due, key=lambda c: REVIEW_THRESHOLD - m[c])), \
               "scheduled review"

    # 2. Frontier: prereqs satisfied, concept not yet mastered
    frontier = [c for c in concepts
                if all(m[p] >= GATE for p in prereqs(c))
                and m[c] < MASTERY_THRESHOLD]

    # 3. Within frontier, target 0.6-0.8 predicted success
    candidates = items_for(frontier)
    return best_by_target_difficulty(candidates, m, 0.6, 0.8), \
           "new material at target difficulty"
```

Two policy notes that are decisions rather than implementation details:

**Interleave rather than block.** Mixing concepts produces better long-term retention than grouping them, and learners consistently rate it as less effective. If you tune sequencing against satisfaction metrics, it will drift toward blocked practice. Make this choice deliberately.

**Review before new material.** Decayed concepts come first. This is unpopular in product review because it slows visible progress, and it is what makes the mastery claims true.

## Operational notes

**Partition the event table by month.** Volume is modest — 100k MAU at 50 events each is ~5M events/month, well inside Postgres territory. Teams reaching for a dedicated event store here are usually solving a problem they do not have.

**Snapshot long histories.** Periodic snapshots bounded by event count keep replay cheap, recorded with the model version they were computed under.

**Upcast, never mutate.** Payload schemas change. Version events and apply upcasting at read. Mutating stored events reintroduces exactly what event sourcing was adopted to avoid.

**Emit xAPI from the same stream.** Actor-verb-object maps directly onto the event record. Enterprise buyers increasingly require warehouse streaming and this makes it a serialisation concern.

Full guide including assessment design, tutor guardrails, standards and costs: [LMS Development in 2026: Architecting a Learning Platform Around AI](https://techcirkle.com/blog/lms-development). Related: [web app development](https://techcirkle.com/development/web-app-development) and [AI development services](https://techcirkle.com/ai-development-services).

## Frequently Asked Questions

### Why must the knowledge graph be acyclic?

A cycle in prerequisite relationships makes gating undecidable — no concept in the cycle can ever be reached. Cycles almost always indicate the decomposition is wrong, so enforce acyclicity at write time rather than discovering it in the sequencer.

### Why version both the model and the graph in projections?

Because either changing invalidates previously computed mastery estimates. Recording both versions makes staleness detectable, so projections can be rebuilt per learner on next access with a background job handling the remainder.

### Why apply decay lazily rather than in a background job?

Because the learner-by-concept cross product is large — a hundred thousand learners across a few hundred concepts is tens of millions of rows to touch daily. Computing decay at read time from `last_evidence_at` gives the same answer for a fraction of the cost.

### How should sequencing balance review against new material?

Review first for concepts decayed below threshold and previously mastered, then new material at the frontier targeting 0.6–0.8 predicted success probability. Review-first is unpopular in product review because it slows visible progress and is what makes mastery claims honest.

### Does a learning platform need a dedicated event store?

Usually not. Around five million events a month is comfortable for partitioned Postgres. The operational complexity lies in projection rebuild machinery and schema versioning discipline rather than in event throughput.

### Should sequencing decisions be explainable?

Yes — return a reason string with every selection. It supports learner-facing explanation, instructor trust, and debugging when a cohort gets stuck in an unexpected loop, which is difficult to diagnose from model output alone.
