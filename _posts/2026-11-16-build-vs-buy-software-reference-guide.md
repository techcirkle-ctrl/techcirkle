---
layout: post
title: "Build vs Buy Software: A Reference Guide with Checklists and Decision Tables"
date: 2026-11-16 00:00:00 +0000
categories: ["Reference", "Software Strategy"]
tags: ["reference", "software", "strategy", "checklist"]
description: "A quick-reference guide to build vs buy software: decision tests, cost checklists, hybrid rules and revisit triggers you can copy into your docs."
image: "https://cdn.sanity.io/images/563mnkns/production/6b212f0908642c255ee9b2eeb18002cbd4ae9eea-1600x1244.jpg"
author: "TechCirkle"
---

![A printed checklist and laptop on a desk, representing a reference guide to software decisions](https://cdn.sanity.io/images/563mnkns/production/6b212f0908642c255ee9b2eeb18002cbd4ae9eea-1600x1244.jpg)

This page is a reference, not a narrative. It condenses our full [guide to build vs buy software](https://techcirkle.com/blog/build-vs-buy-software) into tables and checklists you can copy into a decision record.

## Contents

1. [When to use this](#when-to-use-this)
2. [Decision tests](#decision-tests)
3. [Cost checklists](#cost-checklists)
4. [Outcome table](#outcome-table)
5. [AI-specific guidance](#ai-specific-guidance)
6. [Hybrid guardrails](#hybrid-guardrails)
7. [Two-week process](#two-week-process)
8. [Readiness signals](#readiness-signals)
9. [Revisit triggers](#revisit-triggers)
10. [FAQ](#frequently-asked-questions)

## When to use this

Use it when a team must justify a build or buy choice to a board, CFO or steering group. If your last analysis is more than eighteen months old, redo it: AI-assisted engineering has lowered the cost of commodity builds while vendors have raised effective prices through AI add-ons and usage tiers.

## Decision tests

Run in order. Stop at the first decisive answer.

| # | Test | Question | Leans |
|---|------|----------|-------|
| 1 | Differentiation | Do customers choose us partly because of how this works? | Yes: build or hybrid. No: buy |
| 2 | Fit | Does a product cover about 80% of the real workflow, with the gap workaroundable and outside the risky steps? | Yes: buy. No: build |
| 3 | Control | Must data stay in our hands, or must logic change weekly? | Yes: build |

Notes:

- Judge fit on the workflow, not the feature list.
- Verify fit in a sandbox with a scripted scenario and real work.
- Test 3 can override tests 1 and 2 in regulated sectors.

## Cost checklists

### Buy: three-to-five year TCO

- [ ] Subscription including seat and usage growth
- [ ] Implementation and data migration (often 0.5x to 1.5x first-year licence for mid-market tools)
- [ ] Integrations to identity, data warehouse and billing, and their upkeep
- [ ] Administration headcount and training
- [ ] Workarounds for missing features
- [ ] AI add-ons (sometimes 20 to 40% on the base plan)
- [ ] Exit cost: data export in a usable form
- [ ] Opportunity cost of delayed roadmap items

### Build: three-to-five year TCO

- [ ] First release: design, build, test, deploy
- [ ] Maintenance (planning heuristic: 15 to 25% of build cost per year)
- [ ] Operations: patching, upgrades, on-call
- [ ] Compliance: audit evidence, accessibility, documentation
- [ ] Named long-term owner and a backup engineer
- [ ] Evaluation and monitoring for AI features
- [ ] Opportunity cost of engineers not shipping other work

Always present ranges, not single figures.

## Outcome table

| Situation | Typical outcome |
|-----------|-----------------|
| Commodity capability, strong product fit | Buy |
| Differentiating capability, strong platform fit | Hybrid: buy platform, build layer |
| Differentiating capability, poor fit | Build |
| Sensitive data or weekly logic change | Build or self-hosted platform |
| Cross-system automation specific to your operations | Build, often agentic |

![A table-style graphic mapping decision tests to build, buy and hybrid outcomes](https://cdn.sanity.io/images/563mnkns/production/ecaf7f19c5c9e794f85eb10178b473f2bfc7195f-900x600.jpg)

## AI-specific guidance

| Layer | Default | Reason |
|-------|---------|--------|
| Foundation models, managed vector stores, hosted speech and vision | Buy | Building a model needs data and capital few firms have |
| Retrieval, orchestration, evaluation, guardrails | Open, often build | Encodes domain knowledge and proprietary data |
| User-facing workflow | Build if differentiating, buy for generic copilots | Depends on strategic value |

Budget separately for evaluation: test sets, regression checks and monitoring. This cost does not shrink when code generation gets cheaper. See our [AI development services](https://techcirkle.com/ai-development-services) for how this layer is typically scoped.

## Hybrid guardrails

- Keep the integration surface small and versioned.
- Own the data model for anything differentiating.
- Contract for data export and API stability at signing.
- Run the custom layer as a product with an owner, roadmap and budget.

Common patterns: CRM plus custom scoring and routing; payments processor plus custom ledger and reconciliation; support platform plus custom AI triage agent.

## Two-week process

| When | Activity |
|------|----------|
| Days 1-2 | Write the workflow, split commodity vs differentiating steps, name an executive owner |
| Days 3-5 | Shortlist two vendors, run a scripted sandbox scenario, score fit |
| Week 2, days 1-3 | Build a throwaway AI-assisted prototype of the hardest step |
| Week 2, day 4 | Produce five-year TCO ranges for buy, build and hybrid |
| Week 2, day 5 | Apply the tests, record the decision and revisit conditions |

## Readiness signals

**Ready to build:**

- A product owner with authority to prioritise
- Two engineers able to maintain the system
- Existing deployment, observability and incident process
- Executive agreement to fund it as a product

**Buy and move on:**

- Workflow matches most competitors and customers would not notice a difference
- Vendor already maintains compliance evidence you would otherwise build
- You need it within a quarter and cannot absorb delivery risk

## Revisit triggers

- Price increase above an agreed threshold
- Vendor acquisition or API deprecation
- Strategy change affecting what differentiates you
- AI tooling shift that changes build effort materially

Review major decisions at least annually.

## Next steps

For custom builds, see our [custom software development](https://techcirkle.com/development/custom-software-development) services. For a second opinion, use the [contact page](https://techcirkle.com/contact-us). The narrative version of this reference is the [full build vs buy software guide](https://techcirkle.com/blog/build-vs-buy-software).

## Frequently Asked Questions

### What is the minimum I should document for a build vs buy decision?

The three test results, a three-to-five year cost range for each option, the named owner and the revisit triggers.

### Why does the reference use ranges instead of single numbers?

Inputs such as implementation cost and maintenance are uncertain. Ranges show where cost differences are real and where options overlap.

### What does 80 percent fit mean?

The share of workflow steps a product covers without awkward workarounds, measured in a sandbox using real work rather than a demo.

### When does the hybrid outcome apply?

When a capability differentiates you but a mature platform covers the commodity core. Buy the platform and build a thin custom layer.

### Who should own the decision?

A named executive owner, with engineering, finance and the workflow owner contributing inputs.
