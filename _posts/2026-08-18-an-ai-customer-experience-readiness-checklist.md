---
layout: post
title: "An AI Customer Experience Readiness Checklist"
date: 2026-08-18 00:00:00 +0000
categories: ["Customer Experience", "Checklists", "AI"]
tags: ["checklist", "ai", "customerexperience", "governance"]
description: "A runnable checklist covering contact-driver analysis, corpus readiness, escalation design, resolution instrumentation, cost modelling and governance."
image: "https://cdn.sanity.io/images/563mnkns/production/2b2267f5fa2b7090a8519512eccf8196d744213b-1600x900.jpg"
author: "TechCirkle Editorial Team"
---

![Customer rating a service experience on an app](https://cdn.sanity.io/images/563mnkns/production/2b2267f5fa2b7090a8519512eccf8196d744213b-1600x900.jpg)

Most AI customer experience programmes start at deployment and discover the prerequisites afterwards. This checklist runs in the opposite order.

## 0. Before automating anything

```
[ ] Last quarter's contacts clustered by ROOT CAUSE,
    not by stated intent
    └ "delivery status enquiry"  ✗
    └ "shipment delayed past promised window,
       customer not notified"     ✓
[ ] Top 3 clusters identified
[ ] For each: is this fixable AT SOURCE?
    [ ] product defect        → product backlog, with cost attached
    [ ] process gap           → operations
    [ ] genuine service need  → automation candidate
[ ] Contact-driver analysis shared with whoever owns
    the PRODUCT roadmap, with annual cost per cluster
```

Automating a defect-generated contact makes it cheap, which makes it invisible, which makes it permanent. The discomfort was doing useful work.

## 1. Corpus readiness (the actual blocker)

```
[ ] Contradiction audit run
    └ generate Q&A pairs per doc, cluster questions
      semantically, compare answers, flag divergence
[ ] Stale content DELETED, not left to rank
[ ] Metadata present on every document:
    [ ] market
    [ ] product
    [ ] effective_from / effective_to   ← removes most wrong answers
    [ ] audience (customer | internal)
    [ ] owner (a team, not a person who left)
    [ ] last_verified
[ ] Retrieval filters on metadata BEFORE ranking
[ ] Review cadence defined; past-review docs deprioritised
[ ] Budget line exists for content operations
    └ most commonly missing item in the whole programme
```

Diagnostic before tuning anything:

```
sample wrong answers → was the correct doc retrieved?
   retrieved, still wrong   → generation problem
   not retrieved, exists    → retrieval problem
   absent or contradicted   → CONTENT problem  ← usually this
```

## 2. Agent assist proving ground (≈1 quarter)

```
[ ] Deployed to human agents BEFORE any customer-facing autonomy
[ ] Per-suggestion telemetry captured:
    { conversation_id, intent, suggestion_id,
      retrieved_doc_ids, action: accepted|edited|rejected,
      edit_distance, model_version }
[ ] After a quarter, per intent:
    [ ] acceptance rate
    [ ] factual vs stylistic edit ratio
    [ ] docs most frequent in REJECTED suggestions
        → content backlog, ranked by impact
[ ] Automation shortlist derived from this data,
    NOT from which intents seemed simple in a workshop
```

## 3. Escalation design (non-negotiable)

![Customer selecting a satisfaction rating](https://cdn.sanity.io/images/563mnkns/production/424d91f4f58c80d5e84c196c191a7240d2773497-1600x1047.jpg)

```
[ ] Human reachable in ONE step, always, no negotiation
    └ NOT "fail three times first"
[ ] Context transfers COMPLETELY:
    [ ] full transcript
    [ ] detected intent
    [ ] actions already attempted
    [ ] verified identity
[ ] Proactive escalation triggers configured:
    [ ] frustration signals
    [ ] repetition / rephrasing
    [ ] cancellation
    [ ] complaint
    [ ] bereavement
    [ ] regulatory / legal
[ ] Never-automate list written down and enforced structurally
[ ] Seam between conversation platform and agent desktop
    has a NAMED OWNER          ← where this usually fails
```

## 4. Resolution instrumentation

```
[ ] Identity resolution captured AT CONVERSATION TIME
    conversation_id → customer_id → {orders, payments, tickets}
    └ anonymous sessions cannot be attributed retroactively
[ ] Stored as an explicit event, not a query-time join
[ ] Signals captured:
    [ ] repeat contact within 7 days (semantic similarity,
        threshold calibrated against labelled sample)
    [ ] intended action completion
    [ ] downstream event CONFIRMATION
        └ refund created ≠ refund processed
    [ ] channel switching (chat → phone within 1h)
[ ] Comparison is MATCHED on intent, tenure, product, complexity
    └ contained conversations are systematically easier;
      unmatched comparison flatters automation
[ ] Headline metric:
    repeat_rate(contained) vs repeat_rate(human_handled)
    └ contained higher = containment is measuring FAILURE
```

## 5. Cost model

```
[ ] Modelled PER TURN, not per contact
    └ one contact = 2 turns or 20
[ ] Realistic distribution of conversation lengths used
[ ] Retrieval context tokens included per turn
[ ] Summarisation / QA passes included
[ ] Stressed at 3x volume
[ ] Stressed for LONGER conversations
    (system handles harder cases as it improves)
[ ] Routing designed IN, not retrofitted:
    small cheap model → high-volume simple intents
    larger model      → escalated hard cases
```

## 6. Governance

```
[ ] Hard topic boundaries enforced STRUCTURALLY, not by instruction
    [ ] legal advice
    [ ] medical guidance
    [ ] competitor comparisons
    [ ] sector-specific regulated topics
[ ] Constraints preventing commitments the business cannot honour
    [ ] refunds
    [ ] timelines
    [ ] policy exceptions
[ ] Logging sufficient to reconstruct ANY interaction:
    [ ] full transcript
    [ ] retrieved context
    [ ] model version
[ ] WEEKLY review of real transcripts by someone who
    understands the business
    └ not metrics — actual conversations
    └ every deployment surfaces something here that
      no dashboard indicated, usually within month one
[ ] Review findings feed back into content + config
    (deployment is an operation, not a launch)
```

## 7. Sequencing

```
1. [ ] root-cause analysis → fix at source
2. [ ] corpus repair
3. [ ] agent assist (1 quarter)
4. [ ] narrow autonomy on proven intents
5. [ ] proactive outbound on predictable drivers
6. [ ] voice — LAST, narrow intents only
       [ ] latency budget verified (turn-taking is unforgiving)
       [ ] barge-in / interruption handling works
```

Full guide behind this checklist: [AI Customer Experience: What Actually Moves the Numbers](https://techcirkle.com/blog/ai-customer-experience). We also [run readiness assessments](https://techcirkle.com/contact-us).

## Frequently Asked Questions

### Why start with root-cause clustering?

Because stated intent describes what was asked, not why. Root-cause clustering surfaces product and process defects generating contacts as a side effect, and automating those makes them permanently invisible.

### What is the most commonly missing budget line?

Content operations. Teams fund the engineering and not the curation of the knowledge corpus, then find quality degrading after the launch quarter as the corpus drifts.

### Why must identity resolution happen at conversation time?

Because anonymous sessions cannot be attributed retroactively. Without the link between conversation and customer captured live, resolution measurement is impossible for that traffic.

### Why does the resolution comparison need matching?

Contained conversations are systematically the easier ones. Comparing them against all human-handled contacts biases the result in automation's favour and conceals genuine failures.

### Why model inference cost per turn?

Because one contact may be two turns or twenty, and retrieval context injected into every turn multiplies per-contact cost quietly. Flat per-contact models diverge from reality around the time adoption accelerates.

### What does weekly transcript review catch?

Things no dashboard indicates — tone problems, commitments the system should not make, categories of question the corpus does not cover. Every deployment surfaces something in the first month.
