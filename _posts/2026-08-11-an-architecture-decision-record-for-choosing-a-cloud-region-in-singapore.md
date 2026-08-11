---
layout: post
title: "An Architecture Decision Record for Choosing a Cloud Region in Singapore"
date: 2026-08-11 00:00:00 +0000
categories: ["Architecture", "Cloud"]
tags: ["architecture", "cloud", "adr", "compliance"]
description: "Region choice looks like a five-minute console decision and becomes a permanent constraint. Here is the decision record it deserves."
image: "https://cdn.sanity.io/images/563mnkns/production/cee6ef3b0294bd7e2c4136586c3369cf82a7301f-1600x1086.jpg"
author: "TechCirkle Editorial Team"
---

Region selection is a dropdown. It takes five seconds and it becomes a constraint you live inside for the life of the system — on latency, on which managed services you can use, on what you can tell an assessor, and increasingly on where your AI inference runs.

Most teams make it by accepting whatever the console defaulted to. Here is the decision written down properly, as an ADR you can adapt.

![Singapore business district skyline](https://cdn.sanity.io/images/563mnkns/production/cee6ef3b0294bd7e2c4136586c3369cf82a7301f-1600x1086.jpg)

---

## ADR-004: Primary deployment region for regulated Singapore workload

**Status:** Accepted
**Date:** 2026-08-11
**Deciders:** Platform lead, security lead, product owner

### Context

We are building a customer-facing application processing personal data of Singapore residents, subject to PDPA. A subset of functionality falls under MAS Technology Risk Management expectations. The system uses managed AI services for document extraction and classification.

We need a primary region, a documented position on data residency, and a stated rationale that survives an external assessment.

### Forces

- **Regulatory.** PDPA does not mandate in-country storage, but it does require a documented, defensible position on where data resides and which sub-processors touch it. MAS expectations add scrutiny of resilience and third-party risk. An unexamined default is the weakest possible answer to "why here?"
- **Latency.** Our users are overwhelmingly in Singapore and ASEAN. Round-trip to a US region adds 200ms+ before any processing, which is material for interactive flows.
- **Service availability.** Not every managed service reaches every region simultaneously. Newer AI and data services frequently launch in US regions first, sometimes by a year or more.
- **Cost.** Singapore region pricing runs meaningfully above US regions across compute, storage and egress.
- **Resilience.** Single-region deployment carries a documented recovery position. Multi-region within ASEAN is possible but multiplies operational complexity.
- **AI inference locality.** Managed model endpoints have their own regional availability, distinct from our compute region, and their own retention terms. This is the force that most existing templates omit.

### Options considered

**Option A — Singapore region, single region, in-region AI inference.**

Data at rest, in transit and in inference all remain in Singapore. Strongest compliance story: one sentence answers the residency question. Latency optimal. Cost highest. Constrained to the model families available in-region, which lags the newest releases.

**Option B — Singapore region for compute and storage, cross-region AI inference.**

Primary data stays local; inference calls cross a border. Access to a wider and newer model selection. Requires a documented cross-border transfer position for whatever is included in a prompt, plus contractual confirmation of the provider's retention behaviour. Complicates the compliance narrative in exactly the way an assessor will probe.

**Option C — US region primary.**

Cheapest, widest service availability, best model access. Latency penalty on every interactive request. Requires a full cross-border transfer justification for all personal data. Difficult to defend for a MAS-supervised workload without a strong specific reason.

### Decision

**Option A**, with a documented exception process for Option B.

Singapore region is primary for compute, storage, backups and inference. Where a specific capability is unavailable in-region and materially improves the product, an exception may be granted, requiring: a written data minimisation analysis of what is transmitted, contractual confirmation that the provider retains nothing, an entry in the cross-border transfer register, and security lead sign-off.

### Consequences

**Positive.** The residency question has a one-sentence answer. Latency is optimal for our user base. Third-party risk assessment is simplified because the sub-processor list is short. Any deviation is deliberate, documented and reviewable rather than accidental.

**Negative.** Higher run cost — budget approximately 15% above equivalent US-region infrastructure. Access to the newest managed AI models will lag, sometimes by quarters. Some architectural patterns requiring services not yet in-region will need to be built rather than consumed.

**Neutral but important.** Backups must be verified as in-region. This is not automatic — several managed backup services replicate cross-region by default configuration, and discovering that during an assessment is a bad way to discover it.

### Compliance notes

- Region choice recorded in the data inventory with this ADR referenced.
- Cross-border transfer register initialised, empty at time of writing.
- Recovery objectives defined and tested against single-region failure, with the documented recovery position stating explicitly that a full regional outage exceeds our RTO. This is an accepted risk, signed off, rather than an omission.
- Any vector store or retrieval index inherits this decision and is covered by the same residency statement and deletion path.

---

![Two engineers reviewing infrastructure code](https://cdn.sanity.io/images/563mnkns/production/330c54432ea871f17b3e82987fd4e830cf2cff82-1600x1066.jpg)

## Why write this down

Three reasons, in ascending order of importance.

An assessor will ask. "Why is your data here?" has two possible answers: a reasoned decision with recorded tradeoffs, or a shrug. The first takes an hour to write and closes the conversation. The second opens several more.

Your future team will ask. In eighteen months someone will propose moving a workload for cost reasons and will not know what the current placement was protecting. The ADR is the artefact that prevents a compliance-motivated decision from being unwound as an optimisation.

And writing it forces you to check. The backup replication point above is real — several teams discover their in-region position is untrue only when someone actually verifies it, and the moment of verification is much better placed before an assessment than during one.

Broader context on building regulated software in this market — PDPA architecture, MAS TRM expectations, realistic costs and vendor evaluation: [Application Development Singapore: What Senior Buyers Get Wrong in 2026](https://techcirkle.com/blog/application-development-singapore).

TechCirkle: [custom software development](https://techcirkle.com/development/custom-software-development) | [LLM integration](https://techcirkle.com/llm-integration)

## Frequently Asked Questions

### Does PDPA require data to be stored in Singapore?

No. PDPA permits cross-border transfer provided the receiving jurisdiction offers comparable protection and you have taken reasonable steps to ensure it. What it does require is that you have a documented, defensible position. An unexamined console default is not one.

### What is the cost difference between Singapore and US regions?

Typically 10% to 20% higher across compute, storage and egress, varying by provider and service. Budget around 15% as a planning figure. For most applications this is a smaller number than the cost of a weak compliance position.

### Should AI inference run in the same region as my data?

Ideally yes, because it keeps the residency story to one sentence. If a capability you need is only available elsewhere, treat it as a documented exception: minimise what is transmitted, confirm contractually that the provider retains nothing, and record it in a cross-border transfer register.

### Do backups automatically stay in the same region?

Not always. Several managed backup and snapshot services replicate cross-region under default configuration. Verify explicitly rather than assuming, and record the verification. This is a common gap and a bad one to find during an assessment.

### Is single-region deployment acceptable for a regulated workload?

Often yes, provided your recovery objectives are defined, tested, and honestly state that a full regional outage exceeds your RTO — signed off as an accepted risk. What is not acceptable is an implicit single-region position with recovery objectives that quietly assume otherwise.

### What should an ADR contain?

Context, the forces in tension, the options genuinely considered, the decision, and the consequences including the negative ones. The consequences section is the most valuable and the most often skipped, because it is what tells a future reader what the decision was protecting.

### How does this affect vector stores and search indexes?

They inherit the same decision and must be covered by the same residency statement, access controls and deletion path. Derived stores are the most common place where a stated residency position turns out not to be true in practice.
