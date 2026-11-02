---
layout: post
title: "Telecom Software Project Reference: Evaluation Checklist and Compliance Map"
date: 2026-11-02 00:00:00 +0000
categories: ["Telecom", "Reference"]
tags: ["telecom", "checklist", "compliance", "vendor-evaluation"]
description: "A reference for evaluating telecom software companies: a scoping and vendor checklist, plus a US, UK, EU and Gulf compliance map for legal review."
image: "https://cdn.sanity.io/images/563mnkns/production/19888d435263f06d570e64bc01059b0d72524c9d-1600x1094.jpg"
author: "TechCirkle"
---

![An engineer holding a safety helmet in front of a telecommunication tower during a site visit](https://cdn.sanity.io/images/563mnkns/production/19888d435263f06d570e64bc01059b0d72524c9d-1600x1094.jpg)

This page is a working reference for teams scoping a project or comparing telecom software companies. Copy the lists into your own tracker and strike out what does not apply. For the reasoning behind them, see the full [guide to telecom software companies](https://techcirkle.com/blog/telecom-software-companies).

**Note:** the compliance section lists prompts for your legal and compliance team. It is not legal advice, and it is not exhaustive.

## 1. Scope the problem first

Tick the category that matches the problem, since each points to a different kind of vendor.

| Problem | Usual vendor type |
| :-- | :-- |
| Radio, core network functions, network management | Network equipment vendor |
| Broad charging, billing, inventory, orchestration | BSS and OSS suite vendor |
| Fast launch for an MVNO, digital brand or small operator | Cloud-native SaaS BSS |
| Messaging, voice or verification in a product | CPaaS or API provider |
| Apps, portals, integration layers, data products, AI copilots | Custom engineering firm |

Then answer these:

- [ ] Which business outcome does the project serve?
- [ ] Is this a system of record or a source of differentiation?
- [ ] Which legacy interfaces does it touch, and are they documented?
- [ ] Who internally will run it after go-live?
- [ ] Which markets does it serve, and where can the data live?

## 2. Buy, extend or build

- [ ] **Buy** for common systems of record: core network, charging, core billing.
- [ ] **Extend** through APIs where a product mostly fits and processes differ.
- [ ] **Build** for customer experience, partner tools, integration and proprietary data products.
- [ ] For each proposed customisation, estimate the cost across the next three upgrades.

## 3. Vendor evaluation checklist

### Evidence and team

- [ ] Reference customer of similar size and complexity
- [ ] Reference conversation held without the vendor present
- [ ] Named delivery team, with telecom experience per person

### Integration

- [ ] Described approach to a poorly documented interface
- [ ] Discovery phase included before full commitment
- [ ] Contract tests against real legacy behaviour
- [ ] Parallel running plan for charging and billing
- [ ] Rollback path that does not take service down

### Standards

- [ ] TM Forum Open API claims list APIs and versions
- [ ] Certification status confirmed

### Commercial and exit

- [ ] Five-year cost: licences, implementation, integration, upgrades, infrastructure, internal team
- [ ] Data export formats specified
- [ ] Source code or escrow arrangements agreed
- [ ] Transition support defined
- [ ] Inference and usage charges allocated

## 4. AI due diligence

- [ ] What data does the model see, and where is it processed?
- [ ] Is customer or traffic data masked before inference?
- [ ] Does our data train anything shared with other customers?
- [ ] How are outputs evaluated before and after launch?
- [ ] What happens when the model is wrong?
- [ ] Who pays for inference, and how is usage capped?
- [ ] Is telemetry filtered statistically before it reaches an LLM?
- [ ] Does a person or deterministic policy engine approve state changes?
- [ ] Are rating and charging kept fully deterministic?
- [ ] Is there an audit trail of agent actions?

Our [agentic workflow development](https://techcirkle.com/agentic-workflow-development) work follows these boundaries, and they make a reasonable baseline whoever builds your system.

![Fibre optic cables connected to optic ports and network cables in ethernet ports inside a rack](https://cdn.sanity.io/images/563mnkns/production/503506ad29d90300329e40cbe83a1727a74815e1-1600x1020.jpg)

## 5. Compliance map

Telecom is regulated nationally, so confirm specifics for each country you serve.

| Region | Items to raise with counsel | Likely design impact |
| :-- | :-- | :-- |
| United States | FCC CPNI rules, CALEA lawful intercept, state privacy laws, supply chain restrictions on certain equipment | Access controls on customer network data, intercept capability, vendor and component review |
| United Kingdom | Telecommunications (Security) Act 2021 and its code of practice, UK GDPR, high-risk vendor rules | Security controls, vendor assessment, data protection review |
| European Union | GDPR, ePrivacy framework, NIS2, national implementations | Consent and traffic data handling, incident processes, country-level variation |
| Gulf and other markets | National telecom regulators, data localisation requirements | Where systems run, including AI inference; regional hosting |

### Questions for every region

- [ ] Which datasets count as sensitive? Consider call transcripts, location data and traffic records.
- [ ] Can this data go to a third-party model endpoint at all?
- [ ] Is private deployment or regional hosting needed?
- [ ] What are the retention obligations, and who can access the logs?
- [ ] Do AI prompts and outputs need the same protection as the source data?
- [ ] Does the vendor's support model move data across borders?

## 6. Delivery planning

Rough planning estimates only, not quotes. Public benchmarks are unreliable.

| Work | Rough cost | Rough timeline |
| :-- | :-- | :-- |
| Paid discovery phase | Varies with scope | 4 to 8 weeks |
| Focused AI copilot pilot | $100,000 to $400,000 | 3 to 6 months |
| Self-care app and integration layer | $200,000 to $800,000 | 6 to 12 months |
| SaaS BSS for an MVNO | Driven by subscribers and integrations | Months |
| Full BSS transformation at a large operator | Tens of millions of dollars or more | Several years |

- [ ] Assess data and interface quality first, since these predict overruns most reliably.
- [ ] Plan incremental migration by product line or customer segment.
- [ ] Define the cutover criteria for parallel runs before starting them.

Need a second opinion on your own list? You can [talk to our team](https://techcirkle.com/contact-us).

## Frequently Asked Questions

### How should this checklist be used in a selection?

Use it as a shared tracker. Mark the items that matter for your project, then ask every vendor for evidence against the same list, so answers can be compared.

### Which checklist items are non-negotiable?

Comparable references, a named team, a rollback path, exit terms and deterministic charging. Any item you drop should be a deliberate decision with a named owner.

### Does this compliance map cover every requirement?

No. It only lists common starting points for discussion, and local counsel should confirm what applies in each market.

### Do data localisation rules affect AI features?

They can. Rules may decide where inference is allowed to run, which can rule out certain hosted model endpoints or require regional deployment.

### Can we skip the discovery phase on a small project?

For very small work, perhaps. Anything touching legacy interfaces benefits from it, because data and interface quality are the main sources of overrun.
