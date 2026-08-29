---
layout: post
title: "Cost Per Resolved Task: The Only AI Metric That Matters"
date: 2026-08-29 00:00:00 +0000
categories: ["Engineering", "Artificial Intelligence"]
tags: ["ai", "observability", "cost-optimisation", "engineering"]
description: "Per-token dashboards hide the truth about AI spend. Instrument cost per resolved task instead — and instrument it before the bill surprises you."
image: "https://cdn.sanity.io/images/563mnkns/production/8d0f2ecc98b49c713028719cb7e60266dbf3d0e6-1600x978.jpg"
author: "TechCirkle Editorial Team"
---

![Team analysing AI system cost and usage data](https://cdn.sanity.io/images/563mnkns/production/8d0f2ecc98b49c713028719cb7e60266dbf3d0e6-1600x978.jpg)

Every model provider ships a cost dashboard. Spend per token, per model, per API key, broken down by day.

These are input metrics wearing an outcome metric's clothes, and they systematically mislead. Here is the concrete case.

## Two configurations, same question

**Configuration A** — cheap model, minimal retrieval:

```
call 1  →  wrong answer          (400 tokens)
user rephrases
call 2  →  partially right       (450 tokens)
user escalates
human agent resolves it          (8 minutes)
```

**Configuration B** — larger model, good retrieval:

```
call 1  →  correct answer        (2,100 tokens)
```

On a per-token dashboard, A costs about 40% of B and looks like the obvious win.

In reality A costs more — three times the round trips, plus eight minutes of a human agent, plus a customer who had to ask twice. The dashboard cannot see any of that, because none of it is a token.

## The metric

```
cost per resolved task = total cost / tasks completed to an acceptable standard
```

Two things make this awkward, and both are worth the effort:

1. **You have to define "resolved."** That is a product decision, not a telemetry one — and like the evaluation rubric, writing it down usually surfaces genuine disagreement inside the team.
2. **It must be instrumented in your application**, not at the model boundary. Only your application knows whether the task actually completed. The provider only knows a request was billed.

## Instrument this from day one

Retrofitting cost attribution after a bill surprises you is considerably harder than adding it up front, because by then you have no historical baseline and no per-path breakdown to compare against.

Minimum viable instrumentation, attached to a task ID that spans the whole interaction:

| Field | Why |
|---|---|
| `task_id` | Groups every call, retry and escalation for one user goal |
| `total_cost` | Summed across every model call in the task |
| `call_count` | Retries are where cost hides |
| `resolved` | Boolean, set at the point of outcome |
| `escalated_to_human` | The most expensive outcome, and usually invisible |
| `model_path` | Which models handled it, in order |
| `latency_total` | End to end, not per call |

With that in place you can answer the question that actually drives architecture: **which task types are expensive, and why?** Usually the answer is retries on a small set of inputs, which is a targeted fix rather than a general one.

## Three levers, in order of impact

**1. Route by difficulty.** The distribution of requests in any real system is skewed — most are easy and repetitive, a minority genuinely hard. Send the easy majority to a small fast model and escalate the rest. This is the largest lever available and a surprising number of teams still are not using it. The router itself can be a cheap classifier, and it commonly pays for its own development inside a month.

**2. Cache at the prompt level.** Your system instructions, reference material and schema descriptions are stable content being re-billed on every call. Prompt caching is widely supported and materially cheaper on the cached portion. Close to free money, unclaimed mostly through inattention.

**3. Reduce round trips.** A task resolved in one call is cheaper than the same task resolved in three, almost regardless of model price. This is usually a retrieval quality problem in disguise — the first answer was wrong because the context was wrong.

## The strategic point

Here is why this matters beyond the finance conversation.

**Your AI unit economics are now an engineering artefact, not a vendor price.**

Two companies buying identical access to identical models, serving comparable workloads, routinely differ by an order of magnitude in cost per outcome. That difference is not procurement leverage. It is routing, caching, retrieval quality, round-trip count and how cheaply failures are detected.

Which means AI cost is not a line item you negotiate annually. It is a capability you build — and like most capabilities, the gap compounds. A team that instrumented cost per resolved task in Q1 makes better architecture decisions for the rest of the year, because it can see the consequence of each one.

## Frequently Asked Questions

### Why not just use the provider's cost dashboard?

Because it reports tokens, and tokens are an input. It cannot see retries, escalations to humans, or tasks abandoned by frustrated users — which is exactly where the expensive outcomes are.

### How do we define "resolved"?

It is a product decision. For support it might be no follow-up contact within 24 hours. For extraction it might be passing validation without human correction. Write it down explicitly; the definition itself is often contested and worth settling.

### What if a task spans multiple sessions?

Carry the task ID across sessions rather than resetting per request. A goal that takes a user three attempts across two days is one expensive task, not three cheap ones, and treating it as three is how the metric gets gamed.

### Is difficulty-based routing hard to build?

Less than expected. The router can be a small classifier trained on the same labelled data you use for evaluation, or a confidence threshold on the small model's own output. It should be cheap and should fail toward escalation.

### Does prompt caching work with retrieval-augmented systems?

Yes, for the stable portion — system instructions, schemas, standing reference material. The retrieved context changes per request and will not cache, but that is usually the smaller share of a long prompt.

### When should we start measuring this?

Day one of the first AI feature. Retrofitting cost attribution after a surprise bill is much harder and leaves you without the historical baseline needed to tell whether a change helped.

---

The full 2026 picture — agents, evaluation, retrieval, regulation and org structure — is here: [Top AI Developments for Business in 2026: What Actually Changed](https://techcirkle.com/blog/top-ai-developments-for-business-2026).

We build [LLM integrations](https://techcirkle.com/llm-integration) and [AI products](https://techcirkle.com/ai-development-services) where unit economics have to hold up. [Contact us](https://techcirkle.com/contact-us).
