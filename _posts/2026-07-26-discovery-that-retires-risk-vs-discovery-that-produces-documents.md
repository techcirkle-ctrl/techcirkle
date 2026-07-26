---
layout: post
title: "Discovery That Retires Risk vs Discovery That Produces Documents"
date: 2026-07-26 00:00:00 +0000
categories: ["Engineering", "Process"]
tags: ["discovery", "engineering", "process", "estimation"]
description: "A discovery phase should end with working code or a killed assumption. If it ends with personas and a roadmap, no risk was retired."
image: "https://cdn.sanity.io/images/563mnkns/production/1330b5dffc4f36bbd3ed286c18b7ce1380b86b5a-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

![Development team working through technical architecture and code](https://cdn.sanity.io/images/563mnkns/production/1330b5dffc4f36bbd3ed286c18b7ce1380b86b5a-1600x1066.jpg)

Discovery is where agency proposals diverge most, and where the largest share of wasted budget sits. Two engagements can both be labelled "four-week discovery" and differ by an order of magnitude in what they actually leave you with.

The test is a single question: **what technical risk is smaller at the end than it was at the start?**

If you cannot answer that with something specific, no discovery happened. Workshops happened.

## What useful discovery looks like

Useful discovery reduces uncertainty about something specific and produces an artefact you can act on. Concretely, it looks like one of these:

**A technical spike that proves or kills an integration.** You believe you can get the data you need out of the legacy system. Discovery finds out. The output is working code that either successfully pulls a representative record, or a written explanation of why it cannot and what the alternative costs. Either outcome is worth the money — the second arguably more.

**A data audit that sizes the migration.** Everyone's estimate for a data migration is wrong until someone profiles the actual data. Nulls where the schema says not-null. Three date formats. Duplicate keys the old system tolerated. The output is a defect profile and a revised estimate, and it routinely moves a "two-week migration" to two months or vice versa.

**A prototype tested with real users that kills a feature.** Not a clickable mockup reviewed internally — put it in front of eight actual users. The output is usually a smaller build than the one you were about to commission, which is the highest-return outcome available in the whole process.

**A load or latency spike against a real constraint.** If the requirement is a P95 under 200ms against a third-party API with a 400ms median, someone should discover that before the architecture is signed off rather than during UAT.

Two to four weeks. Ends with a decision, and usually some running code.

## What padding looks like

Eight to twelve weeks of workshops producing personas, journey maps, a service blueprint, a capability model, and a phased roadmap.

The tell is straightforward: at the end, no technical risk is smaller, nothing runs, and the documents largely restate what you told the supplier during the briefing in a more expensive format.

This is not always cynical. Discovery of this kind genuinely helps when the real problem is that stakeholders disagree about priorities — sometimes an organisation needs a shared artefact more than it needs a technical answer. But that is an alignment exercise, and it should be named as one, scoped accordingly, and not permitted to absorb the budget that should have gone into retiring build risk.

The honest test: **could the discovery output have changed anybody's mind?** A roadmap that simply sequences the features you already listed cannot. A spike that reveals the integration is impossible can.

## Structuring it properly

If I were writing the discovery scope myself, it would have four parts.

**1. Name the assumptions.** Write down the three to five things that, if wrong, would most change the cost or shape of the build. Usually: an integration works as documented, the data is cleaner than feared, a third-party service can meet the latency requirement, users want the thing you plan to build. These are your discovery targets.

**2. Attack each with the cheapest possible test.** For each assumption, the smallest experiment that resolves it. Often a day or two of code. The temptation is to build a partial version of the real system; resist that — a spike is disposable by design and mixing spike code into the eventual build is how you end up shipping exploratory code to production.

**3. Define the decision each test feeds.** Before running it, write down what you will do for each outcome. "If the integration cannot deliver events in real time, we accept a fifteen-minute batch and drop the live dashboard requirement." Deciding this in advance prevents the common failure where a spike returns bad news and everyone rationalises around it.

**4. Fix the duration and the fee.** Four weeks maximum, fixed fee. Then make the build contract contingent on the findings, with either party able to walk.

That last point is the one that reveals a supplier's confidence in their own estimating. Any agency comfortable with their numbers will accept it readily. Reluctance usually means the estimate depends on nobody looking too closely.

## Spike hygiene

Three practical rules, all learned the expensive way.

**Spike code is disposable and should be labelled as such.** Separate branch or separate repository, never merged. The moment a spike becomes the foundation of the real implementation, you have shipped code written to answer a question rather than to be maintained.

**Timebox before starting, and honour it.** "Two days to find out whether this API can give us what we need." If two days pass without an answer, that *is* the answer — the integration is harder than assumed, which is exactly the finding you wanted. Extending the timebox converts a cheap answer into an expensive one.

**Write the finding down, including the negative ones.** A one-page note per assumption: what we tested, what we found, what we decided. Six months later this is the only record of why the architecture looks the way it does, and negative findings are the ones most often lost and most often re-discovered at cost.

## What this means when comparing proposals

Two things to check in any discovery proposal:

- **Is there working code in the deliverables?** If the outputs are exclusively documents, ask which technical risk each one retires. The answers are usually revealing.
- **Is it longer than four weeks?** Beyond that, the marginal week buys less than starting the build and learning from real progress. Long discovery is frequently a way to book revenue while the delivery team finishes another project.

And one thing to offer: agree the discovery, insist it include at least one spike against your real systems, and make the build contingent on its outcome. If discovery goes badly you have spent a small sum to avoid a large mistake, which is the entire point of the exercise.

Full guide with 2026 UK rate tables, IR35 context, and a six-week selection process: [Software Development Agency UK: How to Choose the Right One in 2026](https://techcirkle.com/blog/software-development-agency-uk).

We run discovery this way at TechCirkle — [custom software development](https://techcirkle.com/development/custom-software-development) and [app development in the UK](https://techcirkle.com/app-development-uk).

![Two business teams shaking hands after successfully negotiating a software development contract](https://cdn.sanity.io/images/563mnkns/production/f3894d1a6e2f65a6749381e75bd76ebdcd72a960-1600x906.jpg)

## Frequently Asked Questions

### How do I tell useful discovery from expensive discovery?

Ask what technical risk is smaller at the end than it was at the start. Useful discovery produces a proven or killed integration, a data profile that resizes a migration, a prototype that removes features, or a latency finding. Padding produces personas, journey maps, and a roadmap that restates the brief.

### How long should a discovery phase be?

Two to four weeks maximum, at a fixed fee. Beyond four weeks the marginal week buys less than starting the build and learning from real progress. Long discovery is frequently a way to book revenue while the delivery team finishes another engagement.

### What should a discovery scope actually contain?

Four parts: the three to five assumptions that would most change cost or shape if wrong, the cheapest possible test for each, the decision each test feeds written down in advance, and a fixed duration and fee with the build contract contingent on the findings.

### Why write down the decision before running the spike?

Because it prevents the common failure where a spike returns bad news and everyone rationalises around it. Deciding in advance — for example that a batch interval is acceptable if real-time events prove impossible — means the finding actually changes the plan rather than being absorbed.

### Should spike code become part of the real build?

No. Keep it on a separate branch or repository and never merge it. Spike code is written to answer a question, not to be maintained, and promoting it to a foundation means shipping exploratory code to production. Label it as disposable from the outset.

### What if a timeboxed spike doesn't produce an answer?

That is the answer. If two days pass without resolving whether an API can supply what you need, the integration is harder than assumed — which is precisely the finding the spike existed to produce. Extending the timebox converts a cheap answer into an expensive one.

### Are workshop-based discovery phases ever worthwhile?

Yes, when the real problem is that stakeholders disagree about priorities and the organisation needs a shared artefact. But that is an alignment exercise and should be named, scoped, and funded as one — not permitted to consume the budget that should have retired build risk.
