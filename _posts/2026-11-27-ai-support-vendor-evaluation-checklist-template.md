---
layout: post
title: "AI Support Vendor Evaluation: A Forkable Checklist Template"
date: 2026-11-27 00:00:00 +0000
categories: ["Checklists", "Engineering"]
tags: ["checklist", "ai", "saas", "support", "procurement"]
description: "A forkable markdown checklist for evaluating AI support vendors: pilot plan, evaluation set, metrics, billing definitions and data-export clauses."
image: "https://cdn.sanity.io/images/563mnkns/production/b5015c453649bf54bbadc5f841259363901f685c-1600x1067.jpg"
author: "TechCirkle"
---

![Support agent with a headset at a laptop, representing a structured AI support vendor pilot](https://cdn.sanity.io/images/563mnkns/production/b5015c453649bf54bbadc5f841259363901f685c-1600x1067.jpg)

This is a working template for evaluating AI customer support vendors. Fork it, drop it into an issue or a `docs/` folder, assign owners and fill in the blanks. Every item is meant to be answerable with yes, no, a number or a link to evidence.

It also works for build-vs-buy: treat your in-house prototype as one more vendor.

For the pricing models, cost drivers and build-vs-buy reasoning behind these checks, read the full guide: [AI customer support software for SaaS pricing](https://techcirkle.com/blog/ai-customer-support-software-for-saas-pricing).

* * *

## 0. Evaluation metadata

```yaml
evaluation:
  owner: ""            # one accountable person
  vendors: []          # e.g. [vendor-a, vendor-b, in-house]
  start_date: ""
  end_date: ""         # 2-4 weeks is usually enough
  decision_date: ""
  channels: [chat, email]   # add voice only if you will quote it separately
```

## 1. Baseline (complete before any demo)

- [ ] Monthly conversation volume by channel: `____`
- [ ] Volume growth rate, tied to **user** growth, not revenue: `____%`
- [ ] Fully loaded cost per human-resolved ticket: `$____`
- [ ] Current median and p90 handle time: `____ / ____`
- [ ] Current CSAT or equivalent quality score: `____`
- [ ] Top 20 ticket categories by volume, exported to `baseline/categories.csv`

If the cost-per-ticket box is empty, stop.

## 2. Evaluation set

Build one fixed set and run every vendor against the same inputs.

- [ ] 200-500 historical conversations, sampled across the top categories
- [ ] Personal data removed or masked before sharing with any vendor
- [ ] Each item labelled with the correct answer or outcome
- [ ] Each item tagged `simple`, `account-specific` or `needs-product-action`
- [ ] 20-30 adversarial items: off-topic, prompt injection attempts, angry customers, questions with no documented answer
- [ ] Set frozen and versioned (`eval-set-v1`) before the pilot starts

```text
eval-set/
  v1/
    conversations.jsonl
    labels.jsonl
    README.md   # sampling method, date range, masking notes
```

## 3. Pilot plan

- [ ] Scope: which channels, which categories, what share of live traffic (if any)
- [ ] Offline run on the evaluation set first, live traffic second
- [ ] Hand-off path to humans tested end to end
- [ ] Written agreement that pilot and sandbox conversations are **not billed**, or billed at a stated rate
- [ ] Same knowledge base snapshot given to every vendor
- [ ] Weekly check-in with each vendor, notes logged in the repo

## 4. Metrics

Record per vendor, per category tag.

| Metric | How to measure | Vendor A | Vendor B |
|---|---|---|---|
| Resolution rate (your definition) | Labelled outcomes on eval set | | |
| Wrong-answer rate | Answers contradicting the label | | |
| "I don't know" behaviour | Correct refusals on unanswerable items | | |
| Hand-off quality | Human rates summary 1-5 | | |
| Median turns per conversation | Logs | | |
| Cost per resolved conversation | Projected fees / resolutions | | |
| Adversarial pass rate | Adversarial subset | | |

Notes:

- Define "resolved" **yourself** before the pilot. Do not adopt the vendor's definition for scoring.
- A cheaper vendor with a higher wrong-answer rate is usually the more expensive one once escalations and churn are counted.

## 5. Billing definitions

- [ ] Written definition of each billable unit (resolution, conversation, credit, minute)
- [ ] Does "customer stopped replying" count as resolved? `yes / no`
- [ ] For email: is each reply billed, or the whole thread once?
- [ ] For voice: per-minute rate quoted separately
- [ ] Which demoed features are add-ons (workflow actions, analytics, languages, residency)
- [ ] Sample invoice at 1x, 2x and 5x current volume

For reference, outcome-based list prices have commonly been around $1 per resolution. Use your own quotes in the model, not that figure.

## 6. Contract and data-export clauses

- [ ] Price lock on AI units of at least 12 months
- [ ] Notice period for any unit-price change: `____ days`
- [ ] Volume commitment and what happens if volume falls
- [ ] Spend cap or tiered unit prices that fall with volume
- [ ] Credit for AI-handled conversations a human had to correct
- [ ] Export of raw transcripts **and** vendor-generated labels, summaries and classifications
- [ ] Export format documented (e.g. JSONL with schema) and tested during the pilot
- [ ] Data residency and model-provider disclosure, if regulated
- [ ] Initial term length and expansion options

## 7. Integration depth

- [ ] Read access to billing / subscription data
- [ ] Ability to take actions (plan change, refund) with scoped permissions
- [ ] Identity verification before account-specific answers
- [ ] Audit log of every action the AI takes
- [ ] Clear limit on which product data the AI can reach

If most of your high-cost tickets are tagged `needs-product-action` and no vendor can reach that data safely, consider a hybrid: vendor AI for simple questions, a custom agent for product-specific flows. That is the kind of build we cover in [agentic workflow development](https://techcirkle.com/agentic-workflow-development).

## 8. Decision record

```markdown
## Decision
- Chosen option:
- Pricing model:
- Projected 12-month cost at 1x / 2x / 5x:
- Key risks accepted:
- Review date (cost per resolved conversation, monthly):
```

* * *

## Frequently Asked Questions

### How large should an evaluation set be?

A few hundred labelled historical conversations is usually enough to compare vendors, provided it covers your top categories and includes some adversarial cases.

### Why freeze the evaluation set before the pilot?

So every vendor is scored on identical inputs. Changing the set mid-pilot makes results impossible to compare fairly.

### Should we use the vendor's definition of a resolution for scoring?

No. Write your own definition first and score everyone against it. The vendor's definition belongs in the billing section, not the quality section.

### Can this checklist compare an in-house build with vendors?

Yes. Treat your prototype as another vendor, run it on the same set and project its running costs in the same metrics table.

### Why test data export during the pilot?

Because an untested export clause may produce incomplete or unusable files. Finding that out before signing is far cheaper than after.

### Which single metric should we track after go-live?

Cost per resolved conversation, reviewed monthly, alongside the wrong-answer rate.
