---
layout: post
title: "A Software Risk Register Template and Review Checklist"
date: 2026-11-26 00:00:00 +0000
categories: ["Templates", "Engineering"]
tags: ["template", "checklist", "risk-management", "engineering", "ai"]
description: "A copy-ready software risk register template with fields, examples, a pre-mortem agenda and a sprint review checklist, including AI feature risks."
image: "https://cdn.sanity.io/images/563mnkns/production/3b31deaac968cea95277b823d160666121da4164-1600x829.jpg"
author: "TechCirkle"
---

![Business professional assessing risk on a digital interface, representing a software risk register template](https://cdn.sanity.io/images/563mnkns/production/3b31deaac968cea95277b823d160666121da4164-1600x829.jpg)

This page is a working template. Copy it into your repository, wiki or tracker and adapt it. It is the hands-on companion to our full guide to [software risk analysis](https://techcirkle.com/blog/software-risk-analysis), which explains the reasoning behind each field.

## 1. Risk register fields

For each risk, capture the following.

- **ID:** short identifier, for example `R-012`.
- **Statement:** "Because [cause], [event] may happen, which would [consequence]."
- **Category:** requirements, technical, integration, people/vendor, security/compliance, or AI/model.
- **Likelihood range:** for example 10 to 30 percent.
- **Impact range:** in weeks or money, for example 4 to 6 weeks.
- **Detectability:** early (within a sprint), mid (before launch) or late (after launch).
- **Time to impact:** this month, this quarter or later.
- **Reversible?** yes or no.
- **Response:** avoid, reduce, transfer or accept, plus the concrete action.
- **Owner:** one named person.
- **Trigger:** the early signal that this risk is materialising.
- **Status and trend:** open, retired; improving, stable, worsening.
- **Last reviewed:** date.

## 2. Example entries

**R-001.** Because the partner bank offers only a shared sandbox, integration testing may be blocked for days at a time, delaying release by 4 to 6 weeks. Category: integration. Likelihood: 30 to 50 percent. Detectability: early. Reversible: yes. Response: reduce; request a dedicated sandbox and build against a mock in parallel. Owner: backend lead. Trigger: no dedicated sandbox by sprint 3.

**R-002.** Because the AI assistant reads customer emails, a crafted email could instruct it to trigger a refund, causing direct financial loss. Category: AI/model. Likelihood: 5 to 15 percent. Detectability: late. Reversible: no. Response: reduce; refunds require human approval and an injection test suite runs in CI. Owner: platform lead. Trigger: any guardrail bypass in testing.

**R-003.** Because one engineer holds all payments knowledge, their absence would stall billing work for 2 to 4 weeks. Category: people. Likelihood: 15 to 25 percent. Detectability: early. Reversible: yes. Response: reduce; pair on all payments work and write runbooks each sprint. Owner: engineering manager. Trigger: payments PRs reviewed by only one person.

## 3. Pre-mortem agenda (60 minutes)

1. **5 min:** state the goal, launch date and budget.
2. **5 min:** frame it: "It is twelve months from now and this project failed."
3. **10 min:** everyone writes reasons silently, one per sticky note or line.
4. **20 min:** go round the room, one reason each, until exhausted.
5. **10 min:** cluster and rewrite the top items in cause, event, consequence form.
6. **10 min:** assign provisional owners and pick the top five for scoring.

## 4. Prioritisation rules

1. Irreversible and late-detected risks go to the top, regardless of likelihood.
2. Then sort by expected cost (midpoint likelihood times midpoint impact).
3. Move up anything with time to impact of this month.
4. Any risk without an owner after review is escalated.

## 5. Fortnightly review checklist (15 minutes)

- [ ] Retire risks whose window has passed.
- [ ] Add new risks raised since the last review.
- [ ] Check every trigger: has any fired?
- [ ] Re-score the top five; note the trend.
- [ ] Confirm each top risk has a concrete next action and owner.
- [ ] Check delivery signals: cycle time, review latency, reopen rate, carry-over.
- [ ] Update the one-page leadership summary.

## 6. AI feature checklist

If the product includes AI or LLM features, confirm each item has an owner and a risk entry.

- [ ] Evaluation suite exists and is a release gate.
- [ ] Model versions are pinned; an alternative provider is tested.
- [ ] Per-tenant spend limits and agent iteration caps are enforced.
- [ ] Tool permissions are least privilege; irreversible actions need approval.
- [ ] Sensitive data is classified or redacted before prompts.
- [ ] Every model call is logged with cost, latency and model version.
- [ ] p95 latency is tested under realistic load.

These controls are what we build into [AI development services](https://techcirkle.com/ai-development-services) projects by default.

## 7. Leadership one-pager

For each of the top five risks: one-sentence statement, impact range, response and owner, trend, and any decision needed from leadership. Keep the format identical each month.

Read the full guide to [software risk analysis](https://techcirkle.com/blog/software-risk-analysis) for the reasoning, scoring model and a worked fintech example.

## Frequently Asked Questions

### What should a software risk register contain?

A clear statement, category, likelihood and impact ranges, detectability, time to impact, reversibility, response, owner, trigger, status and review date for each risk.

### How long should a pre-mortem take?

About an hour for a core team, including silent writing, sharing, clustering and assigning owners to the top risks.

### How often should the risk register be reviewed?

Every sprint or every two weeks for about fifteen minutes, with a fuller review at major milestones.

### How should risks be prioritised?

Irreversible and late-detected risks first, then by expected cost, moving up anything that could hit this month.

### What AI risks should be in the register?

Quality variance, provider changes, cost volatility, prompt injection, data leakage and latency, each with an owner and a control.
