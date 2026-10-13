---
layout: post
title: "A 90-Day Cloud Healthcare Landing Zone Runbook"
date: 2026-10-13 00:00:00 +0000
categories: ["Architecture", "Healthcare Engineering"]
tags: ["architecture", "healthcare", "cloud", "fhir", "adr"]
description: "A developer runbook for a cloud healthcare landing zone: identity, logging, keys, FHIR ingestion and an AI pilot, with ADR-style decisions per phase."
image: "https://cdn.sanity.io/images/563mnkns/production/5db34138cdc1e15d9082962b8ff7a35fe7162cf4-1600x1067.jpg"
author: "TechCirkle Editorial Team"
---

![Two doctors reviewing clinical data on a tablet](https://cdn.sanity.io/images/563mnkns/production/5db34138cdc1e15d9082962b8ff7a35fe7162cf4-1600x1067.jpg)

This is a runbook for engineering teams standing up a cloud environment that will hold protected health information (PHI) and support a first AI use case. It is provider-neutral, assumes infrastructure as code, and records each significant choice as a short architecture decision record (ADR).

It is not legal advice. Your compliance and clinical safety leads own the obligations; this runbook gives them something concrete to review.

## Scope and assumptions

- One target cloud provider, with a signed business associate agreement (US) or equivalent contractual and assurance arrangements for your market.
- One first use case with a named clinical owner (for example, remote monitoring triage or ambient documentation).
- Everything is provisioned through code and reviewed via pull requests. No console-only changes.

## Phase 1 (Days 1–30): Map, decide, scaffold

### Checklist

- [ ] Inventory data sources for the chosen use case: system, format (HL7v2, FHIR, DICOM, CSV), owner, volume.
- [ ] Produce a data-flow diagram showing every hop PHI makes, including to any model endpoint.
- [ ] Tag each flow with the jurisdictions it touches and the controls required.
- [ ] Create the account/subscription structure: separate environments for PHI production, non-PHI development and shared security tooling.
- [ ] Bootstrap the IaC repository with policy-as-code checks in CI.

### ADR-001: Separate PHI and non-PHI environments

**Context.** Developers need fast iteration; PHI needs tight control. Mixing them makes every engineer a potential PHI handler.
**Decision.** PHI lives only in dedicated production (and tightly controlled staging) accounts. Development uses synthetic or de-identified data.
**Consequences.** Need a synthetic data generator early. Access reviews become simpler because the PHI boundary is an account boundary.

### ADR-002: Data residency is enforced by policy, not convention

**Context.** Some markets, such as the UAE and Saudi Arabia, generally require health data to stay in-country.
**Decision.** Organisation-level policies deny resource creation outside approved regions for PHI accounts.
**Consequences.** Some managed services may be unavailable in the approved region; record those gaps rather than working around them.

## Phase 2 (Days 31–60): Security baseline and ingestion

### Checklist

- [ ] SSO for all humans with phishing-resistant MFA. No long-lived user access keys.
- [ ] Least-privilege roles; break-glass role with alerting and mandatory post-use review.
- [ ] Customer-managed encryption keys in a managed key service; key admins separate from data admins.
- [ ] Centralised, immutable audit logs shipped to the security account.
- [ ] Alerts for anomalous access, especially bulk reads or exports of patient resources.
- [ ] Backups copied to a separate account with immutability enabled; restore tested, not assumed.
- [ ] Managed FHIR store or interface engine deployed; minimum sources connected for the use case.

![Patient on a telehealth video call with connected home monitoring devices](https://cdn.sanity.io/images/563mnkns/production/70074160a683122029aa23422947c00d36ec7777-1600x900.jpg)

### ADR-003: FHIR as the operational data contract

**Context.** Sources arrive as HL7v2, proprietary exports and device feeds.
**Decision.** Normalise into FHIR resources at the integration layer. Downstream services read FHIR only.
**Consequences.** Mapping work is front-loaded. Every resource carries provenance (source system, timestamp, ingestion job ID).

### ADR-004: Patient identity resolved before analytics

**Context.** Duplicate or unlinked patient records silently corrupt AI inputs.
**Decision.** Introduce a master patient index step in the pipeline; unresolved matches go to a human review queue.
**Consequences.** Some records are delayed until reviewed. That is the intended trade-off.

### Device and RPM ingestion notes

If the use case involves remote monitoring, design for:

- Late, duplicate and out-of-order readings (idempotent writes keyed on device + timestamp).
- Normalisation to FHIR `Observation` with device provenance attached.
- Buffering on device/app side for unreliable home connectivity.

## Phase 3 (Days 61–90): AI pilot with an audit trail

### Checklist

- [ ] Evaluation set and clinical ground truth agreed with the clinical owner before any model work.
- [ ] Model endpoint configured so your data is not retained or used for training unless explicitly agreed.
- [ ] Prompt/output filtering for identifiers the use case does not need.
- [ ] Every inference logged with model version, prompt version, retrieval index version and input references.
- [ ] Subgroup performance report (age, sex, site, device) before go-live.
- [ ] Drift monitor on input distributions; override-rate dashboard for clinicians.
- [ ] Clinical safety documentation completed with your clinical safety lead (in the UK, typically aligned with DCB0129/DCB0160).
- [ ] Limited pilot with explicit rollback criteria.

### ADR-005: Every output must be reproducible

**Context.** Incident reviews will ask what the model saw and said for a given patient on a given day.
**Decision.** Version and log model, prompt, retrieval index and input resource IDs per inference. Retain according to policy.
**Consequences.** Storage and logging cost rise modestly. Incident response becomes possible rather than speculative.

### ADR-006: Autoscale inference; no idle GPUs

**Context.** Always-on GPU instances for low-traffic pilots waste budget.
**Decision.** Use autoscaling or serverless inference where latency allows; prefer smaller specialised models when accuracy permits.
**Consequences.** Cold-start latency must be measured against the clinical workflow's tolerance.

## Cost guardrails

- Tiered storage for imaging; avoid cross-region image movement.
- Budget alerts per environment.
- Report cost per encounter or per patient-month alongside usage.

## Exit criteria for day 90

- The use case runs on the landing zone with real (pilot-scope) data.
- Security baseline is enforced by code and verified by a restore test and an access review.
- Clinical owner has signed off the evaluation results and safety documentation.
- ADRs are merged in the repo, so the next use case starts from recorded decisions.

The broader context, deployment models, US/UK/Gulf regulation, interoperability and cost, is in our full guide to [cloud healthcare in 2026](https://techcirkle.com/blog/cloud-healthcare). We build landing zones and clinical applications like this through [custom software development](https://techcirkle.com/development/custom-software-development) engagements.

## Frequently Asked Questions

### What is a cloud healthcare landing zone?

It is a pre-configured cloud environment with account structure, identity, logging, encryption, networking and policies already in place, so workloads holding health data start on a secure, compliant baseline.

### Why use ADRs for healthcare infrastructure?

ADRs record the context and consequences of each decision. That helps compliance reviews, onboarding and incident investigations, which all ask why something was built a certain way.

### Should development environments use real patient data?

Generally no. Use synthetic or properly de-identified data in development and keep PHI in tightly controlled environments.

### Why normalise to FHIR at ingestion?

It gives downstream services one consistent contract, makes provenance easier to attach, and aligns with the API-based access many regulators now expect.

### How do you stop a model provider training on PHI?

Use contractual terms and service configuration that disable data retention and training, and verify them during vendor review before any PHI is sent.

### Is 90 days realistic?

For a foundation plus one scoped pilot, often yes. Migrating core clinical systems is a separate, longer programme.
