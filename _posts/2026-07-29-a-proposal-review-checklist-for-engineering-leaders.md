---
layout: post
title: "A Proposal Review Checklist for Engineering Leaders"
date: 2026-07-29 00:00:00 +0000
categories: ["Engineering", "Management"]
tags: ["checklist", "procurement", "engineering", "management"]
description: "A section-by-section checklist for reviewing a software vendor proposal, ordered by what most reliably predicts an overrun."
image: "https://cdn.sanity.io/images/563mnkns/production/c1eb5bd44e026369bf64811abfefb679cc30e5e9-1600x900.jpg"
author: "TechCirkle Editorial Team"
---

![Software engineer writing code while colleagues collaborate at a shared desk](https://cdn.sanity.io/images/563mnkns/production/c1eb5bd44e026369bf64811abfefb679cc30e5e9-1600x900.jpg)

A checklist rather than an essay. Ordered by what most reliably predicts an overrun, so if you only get through the first two sections you have still caught the majority of the risk.

Read these before you look at the price. The price is the section designed to anchor you, and reading it first changes how you evaluate everything else.

---

## 1. Integration inventory — read this first

- [ ] Every external system named individually, not summarised as "third-party integrations"
- [ ] An owner identified for each, including the person at the *other* organisation
- [ ] A dependency date per integration
- [ ] Documentation quality assessed per system, not assumed
- [ ] Sandbox or test environment availability confirmed
- [ ] Onboarding lead times noted where approval is required (payment processors, identity providers, financial data aggregators)

**If absent:** the proposal was estimated, not engineered. This is the strongest single predictor of schedule slip, because the delay lives in waiting for third parties rather than in writing code.

## 2. Assumptions

- [ ] An explicit assumptions section exists
- [ ] It states what the vendor assumes about *your* readiness — data quality, stakeholder availability, decision latency, existing documentation
- [ ] It states what happens to price and schedule if each assumption fails
- [ ] Assumptions are specific enough to be falsifiable

**If thin:** the change orders are already implicit in the structure, whether or not anyone intends it. A vendor with a detailed assumptions list is telling you where their number is fragile, which is useful information rather than a weakness.

## 3. Team composition

- [ ] Named individuals, not role titles
- [ ] Seniority stated per person
- [ ] Allocation percentage per person, and how it is verified
- [ ] Current commitments disclosed — is this person on another project today?
- [ ] Rate impact stated when a senior rolls off mid-project
- [ ] Ratio checked: more than four engineers per technical lead degrades architectural coherence; fewer than two makes coordination overhead dominate

**Watch for:** senior profiles in the proposal who quietly disappear after kickoff. Ask directly whether the named people are contractually committed.

## 4. Non-functional requirements

- [ ] Expected load, stated in concrete terms
- [ ] Latency targets per critical path
- [ ] Availability target and what it implies architecturally
- [ ] Backup, recovery, and tested restore procedure
- [ ] Observability — logging, metrics, tracing — treated as scope rather than assumed
- [ ] Security posture described, not asserted

**If silent:** these are the easiest things to quietly under-deliver, because functional acceptance criteria do not test them. Silence here is how a system passes acceptance and fails in production.

## 5. Compliance

- [ ] Frameworks named explicitly — SOC 2, HIPAA, state privacy statutes, WCAG, sector-specific rules
- [ ] Vendor has shipped under them before, with specifics on what it changed architecturally
- [ ] Audit logging treated as a subsystem, not as application logging
- [ ] Access control centralised enough to enumerate
- [ ] Subprocessor list included, with data flows
- [ ] Observation window accounted for in the timeline if SOC 2 Type II is required

**Note:** Type II cannot be fully retrofitted. It assesses whether controls operated over a past period, so late compliance means waiting out a new window — usually the largest cost in the calculation, and not an engineering one.

## 6. AI-specific scope

- [ ] Which activities the estimate assumes AI compresses, stated explicitly
- [ ] Which activities it does not touch, stated explicitly
- [ ] A written standard for what may not be generated — authentication, authorization, cryptography, ledger operations, production migrations, retention cascades
- [ ] Review process adaptation described: PR size limits, provenance tracking, verification standards
- [ ] Inference costs modelled with a usage envelope, not omitted
- [ ] Evaluation infrastructure and drift monitoring in scope
- [ ] Model providers registered as subprocessors

**Watch for:** a blanket percentage discount attributed to AI with no activity-level breakdown. It means the incompressible majority of the work has not been costed.

## 7. Third-party running costs

- [ ] Every service enumerated with a monthly figure
- [ ] Usage-scaling costs distinguished from flat ones
- [ ] Inference charges included where applicable
- [ ] Data processing locations noted per service

## 8. Exclusions

- [ ] An exclusions section exists

**If absent:** the proposal is not comprehensive; it is deferring a conversation you will have later from a weaker position. A vendor willing to write down what they are *not* doing has thought about the boundary.

## 9. Commercial structure

- [ ] Contract type matched to scope stability — fixed bid only where scope is genuinely knowable, capped T&M for evolving product work
- [ ] Caps set per phase rather than per project, so pressure arrives while direction can still change
- [ ] Acceptance criteria expressed as demonstrable behaviour, not deliverable names
- [ ] Change process defined before it is needed
- [ ] Contingency of 15–20% present in your own budget, even if not in theirs
- [ ] Post-launch budget of at least 20% of build cost for the first six months

## 10. Rights and exit

- [ ] IP assigns progressively as work is delivered and paid for, not at final acceptance
- [ ] Categories named explicitly: source, designs, infrastructure-as-code, documentation
- [ ] Prompts, evaluation datasets, and fine-tuned artifacts named where AI is in scope — routinely omitted, and often holding more institutional knowledge than the application code
- [ ] Cloud accounts, DNS, repositories, CI, and app store accounts in your name from day one
- [ ] Source escrow where the vendor is small and the system is business-critical, with a tested release trigger
- [ ] Written transition clause: credential handover, support window at agreed rates, documentation standards, named knowledge transfer participants

## 11. Discovery

- [ ] Priced and scoped separately from the build
- [ ] Two to six weeks
- [ ] Produces architecture with rejected alternatives, integration inventory, data model, risk register, and an estimate with a stated confidence interval
- [ ] Deliverables yours unconditionally, whether or not you proceed

**If the vendor refuses the last item:** they are making your architecture conditional on awarding them the build. That refusal is a complete answer.

---

## The five questions to ask alongside the document

1. Which activities does AI compress here, and which does it not touch?
2. What is your standard for what may not be generated?
3. What are the three most likely reasons this runs over, and how would we detect them early?
4. Describe a project that went badly and what you changed afterward.
5. What in this scope would you cut if it were your budget?

Four and five are the strongest discriminators. Both ask a firm to say something against its immediate commercial interest, and the ones willing to do it are reliably the ones that deliver.

Full guide with US delivery models, real rate bands, compliance detail, contract structures, and a twelve-month budget model: **[Custom Software Development Services in USA: The 2026 Cost and Vendor Guide](https://techcirkle.com/blog/custom-software-development-services-in-usa)**.

![Business partners shaking hands after signing a software development agreement](https://cdn.sanity.io/images/563mnkns/production/afda019a8ca4510290d88c6969954d01ffa3c397-1600x1042.jpg)

## Frequently Asked Questions

### Which proposal section should I read first?
The integration inventory. Every external system named, with an owner and a dependency date. Integration risk is schedule risk, and the delay is almost always in waiting for third parties rather than in development work.

### What does a thin assumptions section indicate?
That change orders are structurally implicit, whether or not anyone intends it. A detailed assumptions list tells you where the estimate is fragile, which is useful. Its absence means you will discover the fragility from inside the project.

### Why do non-functional requirements matter so much?
Because they are the easiest scope to quietly under-deliver — functional acceptance criteria do not test load behaviour, recovery, or observability. Silence here is how a system passes acceptance and then fails in production.

### What should an AI-heavy proposal include that others do not?
An activity-level breakdown of what AI compresses and what it does not, a written standard for what may not be generated, review process adaptations, an inference cost model with a usage envelope, and model providers registered as subprocessors.

### What IP category is most commonly missing?
Prompts, evaluation datasets, and fine-tuned model artifacts. Contracts written from pre-2023 templates omit them entirely, leaving ownership ambiguous on assets that may represent more accumulated knowledge than the application code itself.

### Should discovery be a separate purchase?
Yes. It should be scoped and priced independently, run two to six weeks, and produce artefacts you own unconditionally whether or not you proceed. A vendor refusing that term is holding your architecture against the next contract.
