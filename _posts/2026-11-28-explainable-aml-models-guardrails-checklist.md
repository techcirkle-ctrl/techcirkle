---
layout: post
title: "Explainable AML Models: A Guardrails Checklist for Engineering Teams"
date: 2026-11-28 00:00:00 +0000
categories: ["Checklists", "Engineering"]
tags: ["checklist", "aml", "mlops", "explainability", "fintech"]
description: "A technical checklist of guardrails for explainable AML models: human sign-off, reason codes, grounded generation, versioning, validation and data."
image: "https://cdn.sanity.io/images/563mnkns/production/62aaecd6f8440eb6345a510b1e4ec664da128037-2561x1445.jpg?w=1600"
author: "TechCirkle"
---

![Anti-money laundering dashboard on a laptop and tablet, the kind of system these model guardrails protect](https://cdn.sanity.io/images/563mnkns/production/62aaecd6f8440eb6345a510b1e4ec664da128037-2561x1445.jpg?w=1600)

This is a working checklist for teams adding machine learning or LLM agents to an anti-money laundering stack. Copy it into an issue tracker, assign owners and mark each item yes, no or partial.

The premise: regulators do not object to AI in AML. They object to decisions nobody can explain. Each item below is far cheaper to design in during the first sprint than to retrofit after an audit finding. Treat the list as product requirements.

For the wider context, including architecture, costs and build vs buy, see the full guide: [AML Software Solutions in 2026](https://techcirkle.com/blog/aml-software-solutions).

## 0. Governance before code

- [ ] A model governance policy exists before the first model is trained.
- [ ] Each model has a documented purpose, owner and intended use.
- [ ] The policy references the model risk guidance you follow (for example the Federal Reserve's SR 11-7).
- [ ] The compliance officer has signed off the business-wide risk assessment the models rely on.
- [ ] Baselines are recorded: alert volume, false positive rate, handling time and backlog.

## 1. Human decision boundaries

- [ ] No model or agent can file a regulatory report.
- [ ] No model or agent can exit a customer.
- [ ] No model or agent can close a high-risk alert.
- [ ] Agent tools are read-only and scoped to the case in hand.
- [ ] Four-eyes approval is enforced in the application for filings and high-risk closures.
- [ ] Auto-closure, if used, is restricted to a documented low-risk tier.
- [ ] Auto-closed alerts are randomly sampled and quality reviewed on a schedule.

## 2. Reason codes and explanations

- [ ] Every prioritisation score is stored with the factors that drove it.
- [ ] Reason codes are written in language an analyst and an examiner can follow.
- [ ] Entity and sanctions matches include an explanation of why the records matched.
- [ ] Analysts can see which rule versions and model versions fired on an alert.
- [ ] Explanations are tested with analysts, not only with data scientists.

## 3. Grounded generation

- [ ] Summaries and narratives are generated only from retrieved case evidence.
- [ ] Each generated statement cites the underlying record.
- [ ] Statements without a citation are removed before a human sees the draft.
- [ ] Retrieval is limited to your own case data and approved policy documents.
- [ ] Draft narratives are clearly marked as drafts until an officer approves them.
- [ ] An evaluation set of past cases is used to measure unsupported claims before each release.

## 4. Versioning and replay

- [ ] Rules are versioned.
- [ ] Models are versioned.
- [ ] Prompts are versioned.
- [ ] Thresholds are versioned, with a recorded rationale for each change.
- [ ] Features are stored with point-in-time correctness.
- [ ] Any historic decision can be replayed with the versions and data that applied at the time.
- [ ] A sample of decisions is replayed automatically and mismatches are investigated.

A minimal decision record to support replay:

```yaml
decision_id: dec_7f3a
alert_id: alr_88213
decision_time: 2026-03-04T10:15:22Z
actor: { type: agent, name: l1-triage }
versions:
  rules: [STRUCT_CASH@7]
  model: alert_prio@3.2
  prompt: l1_disposition@14
  thresholds: thresholds_uk@2026-02
reason_codes: [ACTIVITY_MATCHES_PROFILE, NO_ADVERSE_MEDIA]
evidence_ids: [kyc_5521, txn_range_771]
recommendation: close
human_review: pending
```

## 5. Validation and monitoring

- [ ] Models are validated before release by someone other than their builders.
- [ ] Validation results and limitations are documented.
- [ ] Drift is monitored for inputs and outputs.
- [ ] Bias and performance decay are reviewed on a defined schedule.
- [ ] New models and agents run in shadow mode beside the existing process before go-live.
- [ ] Analysts score agent recommendations during the shadow period.
- [ ] Rollout proceeds one alert type or typology at a time.

## 6. Below-the-line testing

- [ ] Activity that did not alert is sampled regularly.
- [ ] Sampling covers alerts suppressed by models, not only rule thresholds.
- [ ] Findings feed back into threshold and model changes.
- [ ] Threshold changes are back-tested against historic activity in a sandbox first.
- [ ] No threshold is raised to reduce queue size without documented evidence.

## 7. Data handling

- [ ] Customer data sent to any model stays within approved boundaries.
- [ ] Model providers are under a private deployment or an enterprise agreement with no training on your data.
- [ ] Hosting region matches data residency rules for each jurisdiction.
- [ ] Access to investigation data is narrower than access to ordinary customer records (tipping-off controls).
- [ ] Audit logs are append-only and record who or what did what, when, and on which data.
- [ ] Logs reference evidence by ID or hash instead of duplicating sensitive records.

## 8. Jurisdiction and reporting

- [ ] Thresholds, report formats, retention periods and filing deadlines live in configuration, not code.
- [ ] Report generation traces every data point back to source.
- [ ] Required formats are supported for each market you operate in (for example FinCEN e-filing or goAML).
- [ ] Interpretation of obligations is owned by the compliance officer and counsel, and recorded.

## How to use the results

Count the "no" answers in sections 1 to 4 first. Those are the items most likely to stop a regulator or banking partner accepting your deployment. Sections 5 and 6 are what keep it accepted over time.

If the list feels long, start with one alert type. Shadow first, one typology at a time, with measured evidence at each step. That sequence is what makes the result defensible.

Teams that want help implementing these controls can look at how we approach [LLM integration](https://techcirkle.com/llm-integration) and [agentic workflow development](https://techcirkle.com/agentic-workflow-development).

This checklist is engineering guidance and not legal advice.

## Frequently Asked Questions

### What makes an AML model explainable?

Its outputs come with reason codes a person can follow, its inputs and versions are recorded, and its decisions can be replayed.

### Is auto-closing alerts ever acceptable?

It can be, for a documented low-risk tier with random sampling and quality review. High-risk alerts are closed by people.

### What is below-the-line testing?

Sampling activity that did not generate an alert, to check that thresholds and suppression models are not hiding real risk.

### Who should validate an AML model?

Someone independent of the team that built it, before release, with ongoing monitoring afterwards.

### Do LLM prompts fall under model governance?

They should. A prompt influences output in the same way a model version does, so it needs versioning and change control.
