---
layout: post
title: "The Software Audit Checklist: 60 Checks Across Nine Areas"
date: 2026-11-26 00:00:00 +0000
categories: ["Checklists", "Engineering"]
tags: ["checklist", "audit", "security", "devops", "ai"]
description: "A reviewable software audit checklist: architecture, code, security, dependencies, infrastructure, data, delivery, AI features and knowledge risk."
image: "https://cdn.sanity.io/images/563mnkns/production/b45d2b2b4895479d2a826f999c61403883e50f09-1600x1067.jpg"
author: "TechCirkle"
---

![Programming code on a monitor, representing the systems covered by a software audit checklist](https://cdn.sanity.io/images/563mnkns/production/b45d2b2b4895479d2a826f999c61403883e50f09-1600x1067.jpg)

This is a working checklist for a combined technical audit. Copy it into an issue, assign owners and mark each item yes, no or partial. A "no" is not automatically bad. It is a prompt to decide whether the gap matters for your next stage.

For the reasoning behind these checks, plus process, costs and report structure, see the full guide: [Software Auditing in 2026](https://techcirkle.com/blog/software-auditing).

## 1. Architecture and scalability

- [ ] An up-to-date architecture diagram exists and matches reality.
- [ ] The system's known bottlenecks are documented.
- [ ] Load has been tested at a multiple of current peak traffic.
- [ ] Services can be deployed independently where intended.
- [ ] The data model supports planned expansion (currencies, regions, tenants).
- [ ] Background jobs and queues have retry and dead-letter handling.

## 2. Code quality and maintainability

- [ ] Coding conventions are written down and enforced by linting.
- [ ] Hotspots (complex and frequently changed files) are identified.
- [ ] Tests cover core business logic, not just utilities.
- [ ] Mutation testing or equivalent shows tests catch real bugs.
- [ ] Duplicated logic has been measured with a clone detector.
- [ ] A new engineer can ship a small change safely in their first week.
- [ ] AI-generated code is reviewed against the same standards as hand-written code.

## 3. Security

- [ ] Every endpoint that takes a resource ID checks ownership or tenancy.
- [ ] Tenant isolation is enforced below the application layer where data is sensitive.
- [ ] Secrets live in a secrets manager, not in code or CI variables in plain text.
- [ ] Git history has been scanned for committed secrets.
- [ ] Authentication supports MFA and SSO where customers need it.
- [ ] Input validation and output encoding follow OWASP guidance.
- [ ] A penetration test has been run within the last 12 months.

## 4. Dependencies and licensing

- [ ] A software bill of materials (SBOM) can be generated.
- [ ] Known vulnerable dependencies are listed by severity and reachability.
- [ ] No abandoned libraries sit in critical paths.
- [ ] Open-source licences have been reviewed for copyleft obligations.
- [ ] Dependency updates are automated and reviewed regularly.

## 5. Infrastructure and cloud cost

- [ ] Infrastructure is defined as code and reproducible from scratch.
- [ ] Backups exist and a restore has actually been tested.
- [ ] A disaster recovery plan has target recovery times.
- [ ] Monitoring and alerting cover user-facing symptoms.
- [ ] Cost per unit of usage (user, tenant, transaction) is tracked.
- [ ] Unused and oversized resources have been reviewed.
- [ ] Production access is limited and logged.

## 6. Data and privacy

- [ ] Personal data locations are mapped.
- [ ] Access to personal data follows least privilege.
- [ ] Retention and deletion rules are implemented, not just documented.
- [ ] Data processing agreements exist with all processors, including AI providers.
- [ ] Relevant frameworks (GDPR, HIPAA, PCI DSS, SOC 2) have been mapped to controls.
- [ ] Audit logs record who accessed or changed sensitive data.

## 7. Delivery process

- [ ] CI runs tests, linting and security checks on every change.
- [ ] Deployment is automated and repeatable.
- [ ] Rollback is possible within minutes.
- [ ] Change failure rate and lead time are measured.
- [ ] Incidents get blameless reviews with tracked actions.
- [ ] Code review is required for production changes.

## 8. AI features

- [ ] All model call sites and tool definitions are inventoried.
- [ ] Untrusted content cannot override instructions or trigger tools (tested).
- [ ] Tool permissions are enforced server-side with least privilege.
- [ ] Irreversible agent actions require human approval.
- [ ] A versioned evaluation set runs on prompt or model changes.
- [ ] Per-user and per-tenant spend limits exist.
- [ ] Agent loops have iteration caps and timeouts.
- [ ] Model calls are logged with prompt version, model, tokens and latency.
- [ ] Switching provider is feasible within weeks.

## 9. People and knowledge

- [ ] No critical module has a bus factor of one.
- [ ] Runbooks exist for common operational tasks.
- [ ] Onboarding documentation is current.
- [ ] The business, not a vendor, owns all accounts and credentials.
- [ ] Key decisions are recorded (for example as architecture decision records).

## Scoring and next steps

For each "no", record the business impact, likelihood, effort and owner. Group related gaps by root cause, since ten missing authorisation checks usually means one missing shared layer. Then sort into three buckets: fix before proceeding, fix this quarter, monitor.

Re-run the automatable checks quarterly and compare trends.

Section 8 is where most teams have the biggest gaps right now. If you are building or hardening AI features, our [LLM integration](https://techcirkle.com/llm-integration) practice covers these controls by default. For the full narrative and examples behind this checklist, read [the complete software auditing guide](https://techcirkle.com/blog/software-auditing).

## Frequently Asked Questions

### How should this checklist be used?

Copy it into your tracker, assign an owner to each area and mark each check as yes, no or partial, then prioritise the gaps by business impact.

### Is every "no" a problem?

No. Some gaps are acceptable at your stage. The checklist prompts a decision rather than demanding perfection.

### Which section matters most?

It depends on your next milestone, but security, tenancy and AI features are where serious gaps appear most often today.

### How often should we run this checklist?

Run the automatable checks quarterly and the full checklist before major milestones such as fundraising, acquisition or scale-up.

### Can this replace an external audit?

It supports internal health checks. Independent auditors are still valuable when stakes are high and objectivity matters.
