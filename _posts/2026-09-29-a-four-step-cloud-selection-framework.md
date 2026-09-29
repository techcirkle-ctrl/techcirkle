---
layout: post
title: "A Four-Step Cloud Selection Framework"
date: 2026-09-29 00:00:00 +0000
categories: ["Cloud", "Engineering"]
tags: ["cloud", "architecture", "decision-making", "infrastructure"]
description: "Workload shape, the one binding constraint, a two-week bake-off on real work, and a negotiation backed by a modelled exit cost."
image: "https://cdn.sanity.io/images/563mnkns/production/10d71c095313a2ec6688170fd5deaa922bef54ab-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

![Rows of operational server racks inside a modern cloud data centre facility](https://cdn.sanity.io/images/563mnkns/production/10d71c095313a2ec6688170fd5deaa922bef54ab-1600x1066.jpg)

Cloud selection processes are long because they attempt to compare everything. Since the three major providers now offer broadly equivalent primitives at broadly equivalent prices, comparing everything mostly produces documentation.

This framework compares only the factors that would change the answer. It takes about four weeks.

## Step 1 — Workload shape, on one page

Write down what you actually run, in these categories:

- **Steady-state serving.** Request volume, latency requirement, geographic distribution.
- **Batch.** Volume, schedule, tolerance for delay.
- **Training.** Frequency, model size, framework. Be honest — many teams list this and then never do it.
- **Inference.** Calls per day, tokens per call, latency requirement, ratio of complex to routine requests.
- **Data.** Total volume, growth rate, where it is now, and how much moves between storage and compute per day.
- **Growth.** The curve you believe, not the one in the board deck.

Most teams discover at this step that their AI workload is inference-dominated rather than training-dominated. That eliminates a large part of the comparison they were preparing, which is a week saved before anything else begins.

## Step 2 — Name the one binding constraint

Almost every organisation has exactly one factor that cannot move:

- Data residency requiring processing in a specific jurisdiction.
- An existing analytical estate your AI features depend on.
- An identity platform the entire workforce authenticates against.
- A compliance regime with specific certification requirements.
- Accelerator supply in a particular region.

Write yours down. It frequently eliminates a provider outright, and that is good news — a two-way comparison is far more tractable than a three-way one.

Two observations that recur:

**Data gravity usually wins.** If the analytical estate sits with one provider and your AI features read from it, put the compute next to the data. Cross-cloud is possible; egress charges and added latency make every experiment a small negotiation, and experimentation speed determines whether an AI feature becomes good or stays a demo.

**Identity integration is worth more than instance pricing.** If the organisation runs on one vendor's identity and productivity estate, single sign-on that already works and policies that already pass audit represent a recurring annual saving that a better hourly rate will not match.

## Step 3 — Two weeks, one real workload

Run a bake-off on whichever providers survived step two. One real workload from your own product. Not a benchmark, not a reference application.

Measure exactly three things:

1. **Time to first working deployment.** Account creation to something a colleague can use.
2. **Quality of failure messages.** When something breaks, does the error explain what happened or hand you a code to search for?
3. **Time to a useful support answer.** Open a genuine ticket on something non-obvious. Both latency and quality predict the next three years better than any pricing page.

Two weeks is sufficient. Longer bake-offs produce more documentation, not more signal.

## Step 4 — Negotiate with a modelled exit in hand

Before signing, model what leaving would cost, by layer:

| Layer | Typical exit cost |
|---|---|
| Containerised compute + IaC | Weeks — mostly pipeline work |
| Object storage | Trivial code, significant egress |
| Managed databases | Data move is known; operational relearning is not |
| Identity and access | Quarters — usually the longest pole |
| Serverless event plumbing | High relative to apparent size |
| AI apparatus | Quarters if coupled to one interface, weeks if abstracted |

Then negotiate three terms, in order of how much they matter:

**Commitment flexibility.** Can a multi-year commitment move between instance families and regions as the workload evolves? For a growing product this is worth more than a larger headline discount, and it is often easier to obtain because it costs the provider less. Most teams spend the negotiation arguing over two percentage points and never ask.

**Egress terms.** Still the most common source of a surprise invoice, and AI workloads move far more data than conventional applications. Model retries and evaluation runs, not just the happy path.

**Contractual data handling.** For AI services: storage location, retention period, and whether exclusion from model training is contractual rather than documented. That distinction is the entire question.

Providers discount considerably more when the alternative is credible. The exit model is what makes it credible — and you need most of that analysis for disaster recovery planning regardless.

## What this framework deliberately ignores

Service counts. Per-hour list prices. Marketing claims about breadth. Regional maps.

Not because these are false, but because they no longer discriminate between the options. A decision framework should only include factors that can change the decision.

The full comparison — accelerator supply, managed inference posture, and when multi-cloud is genuine rather than theatre — is here: [Amazon Web Services vs Google Cloud vs Azure](https://techcirkle.com/blog/amazon-web-services-vs-google-cloud-vs-azure).

## Frequently Asked Questions

**What if step two eliminates nothing?**
Then you are genuinely greenfield and should run the bake-off on all three. Uncommon, but it happens, and you have more freedom than most.

**Can step three be shortened?**
Below two weeks you are measuring onboarding rather than operation. Two weeks is the floor for a useful signal.

**Who should run the bake-off?**
Engineers who will operate the result, not an evaluation team. The signal you want is about daily experience.

**How do we handle a team preference for a provider?**
Weigh it as a real cost — familiarity is genuine productivity — but note that it resolves within months, whereas a binding constraint does not resolve at all.

**Should the exit model include staff retraining?**
Yes, and it is frequently the largest line. Tooling migrates faster than people.

**How often should this be repeated?**
The full framework at major inflection points; the exit model annually before renewal.
