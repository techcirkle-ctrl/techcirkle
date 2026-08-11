---
layout: post
title: "Process Mining vs Interviews — Finding the Process That Actually Runs"
date: 2026-08-11 00:00:00 +0000
categories: ["Engineering", "Process"]
tags: ["process mining", "automation", "discovery", "architecture"]
description: "Documented processes describe the design. Three discovery methods find what actually runs, and each catches what the others miss."
image: "https://cdn.sanity.io/images/563mnkns/production/a6eca5c9579b6744e81f0c77ac4caccf991b194c-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

Every automation project starts with a process document. Every process document is wrong.

Not through negligence. It describes the process as designed, usually at the point it was designed. What runs in production is that process plus years of accumulated exceptions, workarounds, informal rules, and at least one spreadsheet that turns out to be structurally load-bearing.

Automating the documented version is one of the more reliable ways to spend a quarter and produce something that handles half your cases.

![Engineer reviewing a process diagram on a laptop](https://cdn.sanity.io/images/563mnkns/production/a6eca5c9579b6744e81f0c77ac4caccf991b194c-1600x1066.jpg)

There are three ways to find the real process. Each catches what the others miss, and mature programs use all three.

## 1. Process mining

Reconstructing the actual process from system event logs — timestamps, actors, state transitions, case identifiers.

**What it gives you:** objective, quantitative variant distribution. Not "the process usually goes A→B→C" but "62% of cases go A→B→C, 19% go A→B→D→B→C, 8% go A→C directly, and there is an 11% long tail across forty distinct paths."

That distribution is the single most useful artefact in discovery, because it tells you what fraction of value each automation increment captures. The forty-path long tail is where your exception budget goes.

**What it misses:** anything that did not touch a system. Every decision made in an email thread, every judgment call, every phone call to a supplier is invisible. In processes with meaningful human judgment, mining shows you the skeleton and none of the muscle.

**Practical requirements:** a case identifier that survives across systems. This is the usual blocker. If your [ERP](https://techcirkle.com/blog/custom-erp-development) calls it an order number, your ticketing system uses its own ID, and nothing links them, you are doing correlation work before you can mine anything. Budget for it.

A minimal event log needs only four fields:

```
case_id, activity, timestamp, actor
```

Getting those four out of three systems in a consistent format is typically a week of work and it is worth it.

## 2. Structured observation

Sitting with people doing the work for a full day and recording every decision, lookup and judgment.

**What it gives you:** the knowledge that never touched a system. The supplier who always sends the wrong reference number. The customer category that requires a manual check nobody documented. The rule everyone follows that exists nowhere in writing. The reason a step that looks redundant is not.

It also surfaces the workarounds, which are the highest-signal thing you will find. A workaround is a place where the designed process fails often enough that people built a habit around it. That is a specification, written by the people who suffer the failure.

**What it misses:** frequency. Someone will describe a case vividly and you will assume it matters, and mining will show it happens four times a year. People remember the painful, not the common. Observation without mining leads you to build for the memorable rather than the frequent.

**How to do it properly:** do not interview. Interviews produce the documented process, because that is what people recall when asked to describe their work abstractly. Watch them work on real cases and ask "why did you do that?" at each decision. The answers to that question are your actual business rules.

![Automated robotic system in a warehouse](https://cdn.sanity.io/images/563mnkns/production/bb4878247735b20c79f790a0c93cacbe50cf419b-1600x926.jpg)

## 3. Exception archaeology

Pulling six to twelve months of escalations, rework tickets, manual overrides and complaint records, then clustering by cause.

**What it gives you:** a pre-written catalogue of everything that will break your automation, already documented, already categorised by people who dealt with it, sitting in your support system.

This is the highest-value hour of the entire discovery phase and it is skipped almost universally, because it does not feel like process discovery. It feels like reading old tickets.

**What it misses:** the happy path entirely. Exception archaeology tells you nothing about what normally happens — you need the other two for that. It only tells you what goes wrong, which is exactly what your automation will encounter and exactly what determines whether it survives contact with production.

**Practical approach:** export the tickets, cluster by cause rather than by symptom, count each cluster, and sort by frequency multiplied by handling cost. The top five clusters usually account for most of your exception burden, and each one is either something to handle explicitly or something to fix upstream before automating at all.

## Putting them together

Run all three in parallel over one to two weeks. The output should not be a flowchart.

What you want is a table:

| Variant | % of cases | Trigger | Human decision points | Automatable tier |
|---|---|---|---|---|
| Standard | 62% | — | none | Tier 1 |
| Missing PO reference | 19% | supplier omits field | match by amount + date | Tier 3 |
| Amount mismatch | 8% | tolerance exceeded | approve or query | Tier 2 + human |
| Long tail | 11% | 40 distinct causes | varies | human, with logging |

That table tells you exactly what to build first, what to build second, and what to deliberately leave to humans with good instrumentation so the next iteration can be driven by data rather than by guessing.

It also gives you an honest coverage estimate before you commit. "We will automate 62% in phase one and 81% by phase two" is a claim you can defend, unlike "we will automate this process."

Full guide — automation tiers, agent boundaries, exception architecture, ROI framing and a 12-week rollout blueprint: [Business Process Automation in 2026](https://techcirkle.com/blog/business-process-automation)

TechCirkle: [AI development services](https://techcirkle.com/ai-development-services) | [custom software development](https://techcirkle.com/development/custom-software-development)

## More from TechCirkle

- [Agentic workflow development](https://techcirkle.com/agentic-workflow-development)
- [Custom AI agent development](https://techcirkle.com/custom-ai-agent-development)
- [Generative AI development services](https://techcirkle.com/generative-ai-development-services)
- [LLM integration services](https://techcirkle.com/llm-integration)
- [Enterprise AI development services](https://techcirkle.com/blog/enterprise-ai-development-services)
- [Top AI automation tools for businesses](https://techcirkle.com/blog/top-ai-automation-tools-for-businesses)
- [Digital transformation services](https://techcirkle.com/digital-transformation-services)

## Frequently Asked Questions

### What is process mining?

Reconstructing the actual process from system event logs — case identifiers, activities, timestamps and actors — to produce an objective picture of how work really flows. Its main output is a variant distribution showing what fraction of cases follow each path.

### Why are interviews unreliable for process discovery?

Because when asked to describe their work abstractly, people recall the designed process rather than the adapted one they actually follow. Observation of real cases with "why did you do that?" at each decision point produces the real rules; interviews produce the documentation you already have.

### What is the prerequisite for process mining?

A case identifier that survives across systems. If your ERP, ticketing and document systems each use their own IDs with nothing linking them, correlation work comes first. Extracting a consistent four-field event log — case, activity, timestamp, actor — typically takes about a week.

### What is exception archaeology and why does it matter?

Clustering six to twelve months of escalations, rework tickets and manual overrides by cause. It matters because exceptions are what break automations, and yours are already documented by the people who handled them. It is the highest-value hour of discovery and the most commonly skipped.

### Can I rely on just one of the three methods?

No. Mining misses everything that did not touch a system. Observation misses frequency, so you build for the memorable rather than the common. Exception archaeology misses the happy path entirely. Each covers the others' blind spots.

### What should discovery produce?

A variant table: each path, its share of cases, what triggers it, what a human decides, and which automation tier suits it. Not a flowchart. The table is what lets you make a defensible coverage claim before committing budget.

### How long should discovery take?

One to two weeks running all three methods in parallel for a first process. Longer than that usually means you are documenting rather than deciding. The goal is enough resolution to choose what to build first, not a complete model of the process.
