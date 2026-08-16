---
layout: post
title: "An Architecture Decision Record Template for Choosing a Cloud Provider"
date: 2026-08-16 00:00:00 +0000
categories: ["Architecture", "Cloud", "Documentation"]
tags: ["architecture", "adr", "cloud", "devops"]
description: "A reusable architecture decision record for the AWS vs GCP choice, with the inputs to collect, the trade-offs to state, and the revisit conditions to record."
image: "https://cdn.sanity.io/images/563mnkns/production/54233ab08a43fb57fb97bf2bdb89d1d3e0e01dc2-1600x896.jpg"
author: "TechCirkle Editorial Team"
---

![Rows of blue-lit servers in a modern data center](https://cdn.sanity.io/images/563mnkns/production/54233ab08a43fb57fb97bf2bdb89d1d3e0e01dc2-1600x896.jpg)

The cloud provider choice gets made once, early, usually informally, and then governs technical decisions for years without anybody being able to reconstruct the reasoning behind it.

That is exactly the situation architecture decision records exist for. Yet in the many codebases we have reviewed, the provider decision is almost never among the recorded ones — teams write ADRs for framework choices and message formats and skip the decision with the longest half-life.

Here is a template, with notes on what each section should actually contain.

## The template

```markdown
# ADR-001: Cloud Provider Selection

## Status
Accepted — 2026-08-16
Revisit by: 2027-08-16

## Context

### Workload characterisation
- Request profile:       [steady-state | spiky | bursty-to-zero]
- Peak/median ratio:     [measured or estimated]
- Projected data @12mo:  [TB]
- Monthly egress:        [TB, and to where]
- Statefulness ratio:    [% of services holding durable state]
- Inference intensity:   [calls per user action; hosted API | self-hosted | hybrid]

### Constraints
- Go-to-market:          [self-serve | mid-market | enterprise procurement]
- Compliance:            [SOC 2 | HIPAA | none yet | ...]
- Hiring model:          [local | distributed | external partner]
- Existing team skills:  [...]

## Decision
We will use [provider] as our primary cloud.

## Rationale
1. [Primary reason, tied to a specific workload property above]
2. [Secondary reason]
3. [Tie-breaker, if applicable]

## Consequences
### Accepted couplings
| Service | Equivalent elsewhere? | Why accepted |
|---------|----------------------|--------------|
|         |                      |              |

### Rejected alternatives
- [Other provider]: rejected because [specific, not generic]
- Multi-cloud: rejected because [...]

## Revisit conditions
This decision should be re-examined if any of the following becomes true:
- [ ] Data volume exceeds [X] TB
- [ ] We commit to self-hosting models at scale
- [ ] Monthly egress exceeds [Y] TB
- [ ] A customer contract requires a specific provider or region
- [ ] Annual spend exceeds [Z], making committed-use terms negotiable
```

## Notes on the sections that matter

### Workload characterisation

This is the part that makes the ADR useful rather than decorative. Four measurable properties do most of the discriminating work between platforms.

The **request profile** determines whether scale-to-zero economics apply. If your trough never approaches zero, they buy you nothing and cost you cold-start latency.

**Data volume and egress** determine how long the decision stays reversible. Egress in particular is chronically under-measured, and it is the mechanism by which the decision hardens.

The **statefulness ratio** is a portability metric. The stateless portion of a system moves for roughly the cost of a container build. The rest is where migrations spend their time.

**Inference intensity** is the newest input and the one most often missing. Record the hosting mode explicitly, because it changes the analysis completely: if you call hosted model APIs, inference cost is decoupled from your provider and AI should be removed from the comparison entirely. Only self-hosting at scale makes it a genuine input, and then availability of accelerators in your region matters more than published pricing.

### Accepted couplings

![Team reviewing cost data on a laptop](https://cdn.sanity.io/images/563mnkns/production/9831ffc3cff060976f5fd138dc03e79a2723b389-1600x1066.jpg)

This table is the highest-value part of the document eighteen months later.

Every managed service you adopt with no equivalent at the other provider is a coupling. Individually each one is usually a good trade — a managed queue saves you from operating a queue. Collectively, in load-bearing positions, they constitute an exit cost that nobody explicitly approved.

Recording them as they are adopted converts an accident into a decision. It also gives whoever scopes a future migration a starting inventory rather than an archaeology project.

### Revisit conditions

Write these as observable thresholds, not intentions. "Revisit if we grow a lot" is not actionable. "Revisit if data volume exceeds 20 TB" can be checked by a script.

This is the section that addresses the actual failure mode. Almost all cloud regret comes not from a wrong decision but from a right decision that nobody knew how to re-evaluate after the surrounding context changed — usually because the people who made it have moved on and the reasoning went with them.

## Where to keep it

In the repository, in version control, reviewed like code. A wiki page will be stale within a year and nobody will notice; a file next to the infrastructure definitions it justifies gets read when the infrastructure changes.

If you already have infrastructure and no ADR, write it retroactively. Reconstructing the reasoning is uncomfortable — you will find couplings nobody remembers accepting — and that discomfort is exactly the value.

The full decision framework behind this template, including the database comparison and migration cost analysis, is here: [AWS vs Google Cloud for Startups](https://techcirkle.com/blog/aws-vs-google-cloud-startups). We also [review architecture decisions](https://techcirkle.com/contact-us) when teams want an outside read.

## Frequently Asked Questions

### Is an ADR worth it for a three-person team?

Especially for a three-person team, because the reasoning lives entirely in people's heads and those heads change. The template above takes about twenty minutes to fill in and is the only artefact that survives turnover.

### What if we already chose and never documented it?

Write it retroactively. Reconstructing the rationale will surface couplings nobody remembers accepting and assumptions that have already expired, which is precisely why the exercise pays for itself.

### How specific should revisit conditions be?

Specific enough to check mechanically. Data volume thresholds, egress thresholds, annual spend levels, and explicit commitments like self-hosting models. Intentions like "revisit when we scale" never trigger.

### Should the ADR cover multi-cloud?

Record it as a rejected alternative with a real reason. Multi-cloud is usually wrong before you have spend large enough for providers to negotiate, because provider-independent architecture forfeits managed services and doubles operational surface — but writing down why you rejected it prevents the question being re-litigated quarterly.

### Where does the AI question fit?

Under inference intensity in the workload characterisation, and under revisit conditions as an explicit trigger. Most teams call hosted APIs, which decouples inference cost from the provider entirely; if that changes, the decision genuinely should be re-examined.

### How often should the ADR be reviewed?

Annually as a default, plus whenever a revisit condition triggers. Twelve months is early enough that switching is still affordable if the assumptions turn out to have failed.
