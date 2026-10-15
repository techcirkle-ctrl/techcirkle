---
layout: post
title: "ADR: Build, Buy, or Extend Our Enterprise Risk Platform? An Engineering Checklist"
date: 2026-10-15 00:00:00 +0000
categories: ["Architecture", "Engineering Management"]
tags: ["architecture", "adr", "integration", "enterprise"]
description: "An ADR-style checklist for ERM software decisions: build vs buy vs extend, scoring criteria, and API and webhook patterns for GRC, ERP, and ticketing."
image: "https://cdn.sanity.io/images/563mnkns/production/569dc49eec485704b54b53ea0dc77d803b4eccd2-1600x1067.jpg"
author: "TechCirkle Editorial Team"
---

Engineering teams are increasingly pulled into risk-platform decisions that used to belong entirely to the CRO's office. The reason is simple: the hard part of modern enterprise risk management is no longer the register itself, it is the data plumbing, integrations, and AI services around it. This post packages that decision as an architecture decision record (ADR) template you can copy into your own repository, followed by the integration patterns that usually determine the real cost.

![Business team analyzing risk charts and graphs on a large digital screen](https://cdn.sanity.io/images/563mnkns/production/569dc49eec485704b54b53ea0dc77d803b4eccd2-1600x1067.jpg)

The business-side reasoning, including typical cost ranges and a phased rollout plan, is covered in our guide to [ERM software solutions: build, buy, or extend](https://techcirkle.com/blog/erm-software-solutions). Below is the engineering view.

## ADR Template

Copy this into `docs/adr/00NN-erm-platform-strategy.md` and fill it in with your stakeholders.

```markdown
# ADR-00NN: ERM platform strategy

## Status
Proposed | Accepted | Superseded by ADR-XXXX

## Context
- Current state: (spreadsheet register / legacy GRC module / vendor X)
- Drivers: (board request, regulatory review, IPO, audit finding, M&A)
- Deadline and hard constraints: (data residency, SSO, budget envelope)

## Options
1. Buy: configure vendor platform as-is
2. Extend: vendor platform + custom integrations and AI services
3. Build: custom platform owned by us

## Decision
<option> because <top three reasons>.

## Consequences
- What we gain:
- What we give up:
- What we must now own (pipelines, models, on-call):
- Exit plan: export format for register, controls, history, audit log

## Review date
<6-12 months out>
```

The "exit plan" line is the one teams skip and later regret. If you cannot export the full register, control library, assessment history, and audit trail in an open format, you have not actually made a reversible decision.

## Scoring the Options

Before debating, list your top ten must-have requirements. For each shortlisted vendor, mark whether it is met:

- **A**: out of the box
- **B**: with configuration
- **C**: with custom integration
- **D**: not at all

A practical rule of thumb: if more than three of the ten land in C or D, extending or building deserves a serious costing rather than a dismissal.

Then score these engineering dimensions from 1 to 5 for each option:

- [ ] **Data model fit.** Does it handle multi-entity groups, risk aggregation from sub-risks to principal risks, and many-to-many risk-to-control mappings?
- [ ] **Integration surface.** Are there documented REST or GraphQL APIs, outbound webhooks, bulk import and export, and sandbox environments?
- [ ] **AI control.** Can you choose model hosting, see prompts and outputs, and switch off AI features per business unit?
- [ ] **Auditability.** Is the audit log immutable, queryable, and exportable, and does it record AI suggestions alongside human decisions?
- [ ] **Access model.** Can access be scoped by role, legal entity, business unit, and risk category?
- [ ] **Operational burden.** Who patches, monitors, and is on call?
- [ ] **Five-year cost curve.** License growth with headcount versus build and run costs.

![Risk management and assessment concept graphic interface with charts and indicators](https://cdn.sanity.io/images/563mnkns/production/6149cd5ae2902767deb26193e62c9d3ac4b8d952-1600x807.jpg)

## Signals That Point to Each Option

**Buy** tends to win when processes are close to industry standard, the deadline is three to six months, and internal engineering capacity is thin.

**Extend** tends to win when a vendor covers most needs but cannot reach KRIs that live in your ERP, warehouse, ticketing, or HRIS, or when you want AI-driven sensing the vendor does not offer or will not let you govern.

**Build** tends to win when risk controls are part of the product you sell (fintech, insurance, marketplaces), the data model is unusual, per-seat licensing outgrows ownership costs at your scale, or you need full control of models and data residency. If you land here, our [custom software development](https://techcirkle.com/development/custom-software-development) team often runs a short discovery to validate the decision before committing.

## Integration Patterns

Integration is usually the largest single line item in an extend or build project, so it deserves its own section in the ADR.

### Pattern 1: Shared control taxonomy via a canonical ID

GRC, compliance, internal audit, and ERM often describe the same control three different ways. Pick one system as the master for control definitions and propagate a canonical control ID everywhere else. Mappings to frameworks (SOX, ISO 31000, NIST CSF, DORA) attach to that ID, so one test result can satisfy several frameworks.

### Pattern 2: ERP and warehouse KRIs via scheduled pulls

Financial KRIs such as liquidity cover or credit concentration change slowly. A scheduled job reading from your data warehouse, with documented lineage and freshness checks, is simpler and more reliable than wiring directly into the ERP. Alert when a feed is late; never show a stale number silently.

### Pattern 3: Ticketing integration via webhooks, both directions

- **Inbound:** subscribe to incident and change webhooks from Jira or ServiceNow, normalize the payload, and queue it for classification against the risk taxonomy.
- **Outbound:** when a risk owner creates a mitigation action, create a ticket in the delivery team's tool and store the external key. Status flows back by webhook, so nobody updates two systems.

A minimal inbound handler should verify signatures and be idempotent:

```ts
app.post("/webhooks/itsm", async (req, res) => {
  if (!verifySignature(req.headers["x-signature"], req.rawBody, process.env.ITSM_SECRET)) {
    return res.status(401).end();
  }
  const event = normalizeIncident(req.body);           // map to internal envelope
  const inserted = await db.inboundEvents.insertIfAbsent(event.externalId, event);
  if (inserted) await queue.publish("incident.received", event);
  res.status(202).end();                               // ack fast, process async
});
```

### Pattern 4: Identity and access via your IdP

Use SSO and SCIM provisioning from the identity provider, and map group membership to risk-platform roles and entity scopes. Avoid maintaining a separate user directory inside the risk tool.

### Pattern 5: AI services as an adjacent layer

Whether you buy or build the register, run document classification, KRI anomaly detection, and narrative drafting as a separate service that writes proposals back through the platform's API. That keeps the system of record clean and lets you swap models without migrating data. If you are designing that layer, our [LLM integration](https://techcirkle.com/llm-integration) practice can help with retrieval, structured outputs, and audit logging.

## Checklist Before Marking the ADR Accepted

- [ ] Top ten requirements scored A through D against each option
- [ ] Integration inventory listing every source system, owner, and access method
- [ ] Exit and export plan validated with a sample export
- [ ] AI governance owner named and models added to the risk register
- [ ] Internal audit consulted on audit-trail and evidence requirements
- [ ] Review date set

## Frequently Asked Questions

### What belongs in an ADR for ERM software selection?

Context and drivers, the options considered, the decision with reasons, consequences including what your team must now operate, an exit plan, and a review date. Keep it short enough that stakeholders actually read it.

### Should webhooks or polling be used for ticketing integrations?

Prefer webhooks for incidents and action status because they are timely and cheaper at scale, but add a periodic reconciliation job that polls for missed events. Webhook delivery is rarely guaranteed.

### How do we avoid duplicate controls across GRC and ERM tools?

Choose one master system for control definitions, issue canonical IDs, and have every other tool reference those IDs instead of creating its own records. Framework mappings attach to the canonical control.

### What makes an ERM vendor API good enough to extend?

Documented and versioned endpoints for risks, controls, assessments, and actions; outbound webhooks; bulk export; a sandbox; and rate limits that support your sync volume. Missing bulk export is a common dealbreaker.

### How often should the build vs buy decision be revisited?

Set a review date six to twelve months out, and revisit earlier if licensing tiers change, a major acquisition adds legal entities, or new regulation shifts data residency requirements.
