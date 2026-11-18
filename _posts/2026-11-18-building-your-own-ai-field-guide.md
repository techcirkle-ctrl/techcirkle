---
layout: post
title: "Building Your Own AI: A Field Guide with Decision Tables"
date: 2026-11-18 00:00:00 +0000
categories: ["Reference", "AI Engineering"]
tags: ["ai", "reference", "llm", "field-guide", "architecture"]
description: "Engineering field notes on building your own AI: decision tables for rung choice, RAG vs fine-tuning, hosting, build vs buy, risks and phase gates."
image: "https://cdn.sanity.io/images/563mnkns/production/12648007e72a86c02ae0d62461390ce48159757c-1600x959.jpg"
author: "TechCirkle"
---

![A hand holding a tablet with a glowing outline of a human head, gears and speech bubbles above it on a teal background](https://cdn.sanity.io/images/563mnkns/production/12648007e72a86c02ae0d62461390ce48159757c-1600x959.jpg)

These are field notes, not an essay. They condense our full guide on [building your own AI](https://techcirkle.com/blog/building-your-own-ai) into tables you can paste into a design doc or decision record. Read the guide for the reasoning; use this page when you need the answer quickly.

## Core assumption

In 2026, "our own AI" means owning the layers that differentiate the product and renting the base model. Pretraining from scratch is out of scope unless a specific condition in the tables below says otherwise.

## Layer ownership

| Layer | Default | Why |
|---|---|---|
| Base model | Rent (API or open-weight) | Commodity that improves and gets cheaper every few months |
| Data and pipelines | Own | Condition of your data is hard to copy |
| Prompts and workflow logic | Own | Encodes product and process knowledge |
| Evaluation suite | Own | Lets you swap models and prove quality |
| UX and integrations | Own | Where adoption is won or lost |

## Choosing a rung

| If the task... | Start at | Notes |
|---|---|---|
| Needs no private knowledge (extract, classify, summarise, draft) | Rung 1: prompt plus API | Treat prompts as code: versioned, reviewed, tested |
| Must answer from your documents or records | Rung 2: RAG | Most effort goes into ingestion and search quality |
| Needs a consistent format or style at very high volume | Rung 3: fine-tune or small model | Prove it beats the prompted baseline on evals |
| Must take actions across several systems | Rung 4: workflow or agent | Narrow tools, step and spend caps, human checkpoints |
| Uses unusual proprietary data general models handle badly | Rung 5: domain pretraining | Rare; models tend to be small and targeted |

Rule: climb only when evals or cost data show the current rung is insufficient.

## RAG vs fine-tuning

| Question | Answer favours |
|---|---|
| Does the information change weekly or monthly? | RAG |
| Do users need citations to sources? | RAG |
| Must results respect per-user permissions? | RAG (filter at retrieval) |
| Is the problem inconsistent output format or tone? | Fine-tuning |
| Do you want a small model to match a large one on one task? | Fine-tuning |

Common error: using fine-tuning to store company facts. It shapes behaviour well and stores changing knowledge poorly.

## Hosted vs self-hosted

| Condition | Lean toward |
|---|---|
| Getting started, low to moderate volume | Hosted API |
| Data legally cannot leave your environment | Self-hosted open-weight |
| API cost at your volume exceeds hosting cost | Self-hosted open-weight |
| A model version must be pinned for years (regulatory) | Self-hosted open-weight |

## Workflow vs agent

| Signal | Fixed workflow | Agent |
|---|---|---|
| Steps can be enumerated in advance | Yes | |
| Auditors need to follow the logic | Yes | |
| Task paths genuinely vary case by case | | Yes |
| Errors are expensive | Yes, with human approval | Only with checkpoints |
| Number of tools | Few, fixed | Small, well-defined registry |

Arithmetic to remember: per-step accuracy compounds. Eight steps at 95 percent each is roughly a two-in-three chance the whole chain is right. If you are deciding where autonomy is worth it, see our [agentic workflow development](https://techcirkle.com/agentic-workflow-development) work.

## Build, buy or hybrid

| Use case type | Decision |
|---|---|
| Transcription, writing help, coding assistants, standard helpdesk bot | Buy |
| Touches core workflow, proprietary data or customer-facing product | Build (usually hybrid) |
| Most real systems | Hybrid: rent model, buy commodity parts (vector DB, observability), build retrieval, workflow, evals, UX |

One-line test: if a competitor bought the same tool tomorrow, would you lose anything? No means buy.

## Cost planning

Ranges for a first production release by an experienced team. Orders of magnitude, not quotes.

| Scope | Build range | Typical time |
|---|---|---|
| Prompt plus API feature | ~$15k to $60k | 3 to 8 weeks |
| Production RAG assistant | ~$40k to $150k | 6 to 14 weeks |
| Fine-tuned or self-hosted model for one task | ~$60k to $250k | varies |
| Agentic workflow across several systems | ~$80k to $300k+ | varies |
| Domain model pretraining | seven figures and up | long |

Hidden costs to budget: data preparation (can reach a third or more of rungs 2 and 3) and ongoing operation (monitoring, eval reruns, prompt upkeep, inference).

## Team

| Role | Needed for | Often missing? |
|---|---|---|
| Technical lead with model and cost judgement | All rungs | |
| Backend and data engineers | All rungs | |
| ML engineer | Fine-tuning, self-hosting, classical ML | |
| Domain expert / product owner | Defining correct answers and evals | Yes, most often |
| Security and compliance | Regulated or customer-facing data | |

## Phase gates

| Phase | Duration | Exit criterion |
|---|---|---|
| 0. Use-case selection | 1 to 3 weeks | One or two cases scored on value, data readiness, risk, measurability |
| 1. Prototype plus eval set | 3 to 6 weeks | Eval set of 50 to 300 real examples; baseline pass rate known |
| 2. Production pilot | 6 to 12 weeks | Limited users; measured gain vs baseline on time, errors, satisfaction |
| 3. Scale and optimise | Ongoing | Cost per successful task trending down; tuning only if evals justify |
| 4. Adjacent workflows | Ongoing | Reuse of pipelines, gateway and eval tooling |

## Risk register

| Risk | Mitigation |
|---|---|
| Hallucination | Grounding, citations, constrained outputs, human review where costly |
| Data leakage | ACL filtering at retrieval, data-class policy per provider |
| Prompt injection | Treat all inputs as untrusted, scope agent tools, validate before side effects |
| Cost runaway | Per-feature and per-tenant budgets, step caps, caching, small-model routing |
| Model deprecation | Abstraction layer plus eval replay |
| Adoption failure | Embed in existing tools, keep humans in control, train users |

## Ownership clause (copy into contracts)

The company owns all code, prompts, evaluation datasets, fine-tuned model weights and data pipelines produced under this engagement. Model providers may not train on submitted inputs.

Need a costed plan for a first pilot? You can [contact the TechCirkle team](https://techcirkle.com/contact-us).

## Frequently Asked Questions

### What is the single most useful table on this page?

The rung table. Starting at the lowest rung that fits avoids most of the cost and risk people associate with building AI.

### How do I know it is time to climb a rung?

When your evaluation results or cost data show a measurable gap that the next rung would close. Enthusiasm is not a trigger.

### Can I mix RAG and fine-tuning?

Yes, and mature systems often do: retrieval supplies current knowledge while a tuned model handles a narrow, high-volume step.

### Which phase is safe to skip?

None, but phases 0 and 1 are short. Skipping the eval set in phase 1 is the most expensive shortcut later.

### Is the cost table usable for budgeting?

As an order-of-magnitude guide for planning only. Data condition, integration count and compliance requirements move the real figure more than the model choice.
