---
layout: post
title: "A Layer-by-Layer Decision Checklist for Transformation Stacks"
date: 2026-09-20 00:00:00 +0000
categories: ["Architecture", "Engineering"]
tags: ["architecture", "enterprise", "ai", "checklist", "integration"]
description: "A practical checklist for deciding what to buy, assemble and build across the four layers of an enterprise transformation stack, with the AI caveats."
image: "https://cdn.sanity.io/images/563mnkns/production/a1d96f5e1e31e690372e252014dc1cd854c22b36-1600x800.jpg"
author: "TechCirkle Editorial Team"
---

![Abstract visualisation of a modern enterprise technology stack](https://cdn.sanity.io/images/563mnkns/production/a1d96f5e1e31e690372e252014dc1cd854c22b36-1600x800.jpg)

This is written for the engineer who inherits an architecture decision rather than the executive who makes it — which, in practice, is who ends up living with it.

The central claim is simple: a transformation stack has four layers, each with different economics, and a single blended build-versus-buy decision applied across all four will be wrong in at least two of them.

## Layer 1 — Systems of record

ERP, CRM, HRIS, finance. State storage with regulatory obligations attached.

**Default: buy.** The cost of these systems was never dominated by development. It is regulatory surface, decades of edge cases, certification and liability. AI-assisted development does not reduce any of those, so the recent fall in coding cost does not change this decision at all.

Checks before committing:

- [ ] Bulk export, scheduled, service-account authenticated, no professional services required
- [ ] Deletions are represented explicitly rather than records silently disappearing from query results
- [ ] Incremental extraction filters on modification time, not creation time
- [ ] Rate limits permit a full sync inside a maintenance window
- [ ] Configuration itself is readable and writable via API, so environment promotion can be automated

That last one decides whether you have real environments or two systems that drift apart.

## Layer 2 — Data and integration

Warehouses, pipelines, event transport, identity resolution.

**Default: assemble.** Mature components exist for every part, and the differentiation is in how they are composed. This is also where most programmes fail, because the work is invisible and therefore scheduled last.

Checks:

- [ ] Canonical entities have canonical identifiers — confirm empirically rather than assuming
- [ ] Deletion and merge semantics are defined before the first pipeline is written
- [ ] Event transport offers at-least-once delivery, ordering information and replay after outage
- [ ] Reprocessing from source is a routine operation, not an incident response
- [ ] Schema changes upstream surface as pipeline failures rather than silent nulls

Build this layer first. Retrofitting it later, under delivery pressure, produces the worst version of it.

## Layer 3 — Workflow and orchestration

The rules: what happens next, who approves, what escalates, how exceptions route.

**Default: it depends, and this is the layer where the answer recently changed.**

Building this logic used to mean months of engineering, which is why buying a general-purpose engine made sense. That code — high-volume, well-specified, heavily testable, driven by domain knowledge you already hold — is close to a best case for AI-assisted development, and its cost has fallen substantially. Licence prices have not.

Decision test:

- [ ] Is our process genuinely non-standard, and is that difference why customers choose us?
- [ ] Would configuring a platform to express our rules require ongoing consultancy?
- [ ] Are there rules we need that a general engine structurally cannot express?
- [ ] Do we already maintain three or more spreadsheets alongside an existing platform?

Three or four yeses point to building. Zero or one point to buying, where you benefit from the platform's generality rather than paying for it.

## Layer 4 — Intelligence

Retrieval, models, agents.

**Default: build thin, and keep it swappable.** The mistake here is neither building nor buying — it is committing. Capability improves faster than any contract term, so a multi-year commitment to an embedded vendor AI is a commitment to a capability snapshot.

Architecture checks:

- [ ] Provider sits behind an interface expressing application intent, not vendor call shapes
- [ ] Prompts are versioned artefacts inside the adapter, not inline strings at call sites
- [ ] Output schemas are declared by the application and enforced by the adapter
- [ ] Retrieval — chunking, index, reranking — is owned in-house, not delegated to a managed service
- [ ] Index can be rebuilt from source on demand, since embeddings are not portable across providers
- [ ] A frozen evaluation set of real production tasks runs in CI against every candidate adapter
- [ ] Agent authority is scoped structurally: read before write, suggestion before action, human confirmation on anything financially or legally consequential
- [ ] Logs capture inputs, retrieved context, reasoning and action, sufficient to reconstruct why

Without the evaluation set, provider switching is unsafe, so it never happens, so the abstraction that enables it is wasted effort.

## Sequencing

1. Data and integration
2. One high-friction workflow, end to end, against a measured baseline
3. Extension of that pattern
4. Intelligence layer, once real data exists
5. Systems-of-record replacement, last

This order front-loads learning and back-loads irreversibility. Building the intelligence layer before step one produces confident answers over sparse data, which costs credibility that is difficult to recover.

## Frequently Asked Questions

### Why decide build-versus-buy per layer rather than once?
Because the four layers have genuinely different cost structures. Regulatory complexity dominates systems of record, composition dominates integration, domain specificity dominates orchestration, and volatility dominates intelligence. One blended decision will be wrong in at least two of them.

### What is the single most important check on a system of record?
Whether configuration is readable and writable via API. If it is UI-only, environment promotion cannot be automated and your staging and production systems will drift apart.

### Why build the integration layer first?
Everything downstream depends on it, and it is invisible work that gets scheduled last if not deliberately prioritised. Retrofitting it under delivery pressure reliably produces the worst version of it.

### When should orchestration logic be built rather than bought?
When the process is genuinely non-standard and that difference is a competitive reason customers choose you, when configuring a platform would require ongoing consultancy, or when necessary rules cannot be structurally expressed by a general engine.

### Why is an evaluation set essential in the intelligence layer?
Because without a frozen set of real production tasks run in CI, quality regressions from a provider change are invisible. Teams therefore never switch, which makes the swappable abstraction pointless.

### What order should the intelligence layer come in?
After integration and at least one real workflow. Models over sparse, inconsistent data produce confident nonsense, and the resulting credibility loss is hard to recover from.

---

The full write-up, including cost ratios and procurement traps: [Digital Transformation Software: A 2026 Buyer's Playbook](https://techcirkle.com/blog/digital-transformation-software)

TechCirkle builds these layers — [agentic workflow development](https://techcirkle.com/agentic-workflow-development) | [LLM integration](https://techcirkle.com/llm-integration)
