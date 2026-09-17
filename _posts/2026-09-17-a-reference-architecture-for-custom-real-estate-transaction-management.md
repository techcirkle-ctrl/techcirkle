---
layout: post
title: "A Reference Architecture for Custom Real Estate Transaction Management"
date: 2026-09-17 00:00:00 +0000
categories: ["Architecture", "PropTech"]
tags: ["architecture", "proptech", "ai", "integrations", "realestate"]
description: "A component and integration map for custom real estate transaction management software: rules engine, documents, AI pipeline, ledger, events."
image: "https://cdn.sanity.io/images/563mnkns/production/96d9802fb5089370bd3e9a2df8a107f6c455bcd6-1600x1001.jpg"
author: "TechCirkle Editorial Team"
---

![Contract signing at a brokerage desk, representing the workflow a transaction platform models](https://cdn.sanity.io/images/563mnkns/production/96d9802fb5089370bd3e9a2df8a107f6c455bcd6-1600x1001.jpg)

If your team has been asked to scope custom real estate transaction management software, the first whiteboard session usually produces a single box labelled "transactions" with arrows to everything else. This post breaks that box apart into the reference architecture we use as a starting point.

## Design constraints first

Before components, pin down the constraints that shape them:

- **Regulatory retention.** Brokers must keep transaction records for years and reproduce them on request. Originals must be immutable and deletions must be governed.
- **Rules change without releases.** Checklists vary by state, deal side, property type, financing and office, and regulations shift (the August 2024 NAR buyer agreement change is a recent example). Compliance admins must edit rules without a deploy.
- **Multi-tenancy.** Even a single brokerage often has multiple brands or acquired offices. Model tenants from day one.
- **Sensitive data.** Files include names, addresses, loan details and sometimes bank details. Field-level access and region-aware storage are requirements, not extras.

## Core services

**Transaction service.** Owns the domain model: property, parties, sides (listing, buyer, dual), stage, office, brand, tenant. Emits events on every state transition.

**Rules and checklist engine.** Evaluates which requirements apply to a transaction, given jurisdiction, deal attributes and office. Store rules as versioned data, not code.

**Document service.** Encrypted object storage with immutable originals, derived versions (split, redacted, OCR text), retention policies and legal hold. Every read and write is audited.

**Review service.** Broker queues with approve, reject and request-correction actions per checklist item. Approvals are only writable by human principals with the right role.

**Workflow and scheduler.** Deadline computation (business versus calendar days, per the contract terms), escalations and multi-channel notifications.

**Commission ledger.** A double-entry style ledger for gross commission, splits, caps, fees, royalties and disbursements. Compensation plans are versioned with effective dates so mid-year changes do not rewrite history. Keep this deterministic and heavily tested.

**Identity and permissions.** SSO, role-based access by tenant, office and team, plus field-level restrictions for financial data.

**Integration layer.** An event bus plus API gateway, so downstream systems subscribe to changes rather than polling.

## The AI pipeline

Treat AI as an asynchronous, queued pipeline sitting beside the document service, never inside the approval path.

```
upload -> split -> OCR -> classify -> extract -> check -> store findings -> human review
```

Each stage writes its output alongside evidence (page number and bounding region). Practical guardrails:

1. **Deterministic rules run first.** A blank field on a known template is a rule, not a model call.
2. **Grounding validation.** An extracted date or amount must appear verbatim on the cited page, or it is flagged instead of saved.
3. **Per-check confidence thresholds.** Signature detection and handwritten counter-offer interpretation deserve different thresholds. Below threshold routes to a person.
4. **No approval rights.** The pipeline's service principal can create findings and return files to agents. It cannot approve checklist items.
5. **Full provenance.** Log model identifier and version, prompt template version, output, confidence and the subsequent human decision.
6. **Hard exclusion for payments.** No AI component drafts, sends or modifies wiring instructions.

Run a shadow period, typically four to eight weeks, where findings are compared against human decisions but not shown to agents. That produces precision and recall per document type on your own files. For orchestration patterns with explicit steps and checkpoints, see our approach to [agentic workflow development](https://techcirkle.com/agentic-workflow-development).

![Engineering whiteboard mapping services and integrations for a brokerage platform](https://cdn.sanity.io/images/563mnkns/production/c9f1672099ca51a12bad7074bbb518831f1f2f23-1600x1066.jpg)

## Integration map

List every system and the direction of data flow before estimating. Integration scope is the most common cause of overruns.

| System | Direction | Notes |
| --- | --- | --- |
| MLS (RESO Web API) | Inbound | Standardised fields; access still needs per-MLS agreements |
| CRM | Bidirectional | Accepted offer creates transaction; closing returns won deal and commission |
| Accounting (QuickBooks, Xero, NetSuite) | Outbound | Journal entries for commission income and agent payables; agree chart of accounts early |
| E-signature and form libraries | Bidirectional | Licensing and API limits vary; signed copies plus certificates return automatically |
| Title and escrow | Bidirectional | Disbursement authorisation out, settlement statement in |
| Data warehouse | Outbound | Event stream or CDC for forecasting and agent analytics |

AI has one narrow role on the money side: reading the final settlement statement and flagging mismatches against the authorised commission. It should never compute splits.

## Customise before you rebuild

Many teams do not need the whole diagram. If an existing platform such as SkySlope or dotloop remains the system of record, you can deploy the AI pipeline, commission ledger or warehouse sync as standalone services against its API. Audit the vendor API first: confirm which objects are readable and writable, which events fire webhooks, and the rate limits.

## Migration note

Cut over by contract acceptance date so no transaction spans two systems, and import closed files as verified read-only archives.

The business-side view of this decision, including buy versus customise versus build and typical cost ranges, is covered in our main guide to [real estate transaction management software](https://techcirkle.com/blog/real-estate-transaction-management-software). Need help scoping it? Our [custom software development](https://techcirkle.com/development/custom-software-development) team builds platforms of this shape.

## Frequently Asked Questions

### Should checklist rules live in code or data?

In data, versioned, and editable by compliance administrators. Record the ruleset version each transaction was evaluated against so historical decisions remain explainable.

### Why keep the AI pipeline asynchronous?

Document splitting, OCR, classification and extraction are slow and variable. Queued jobs isolate that latency from the user interface and make retries, reprocessing and provenance logging straightforward.

### How do we stop an AI model from inventing contract dates?

Validate every extracted value against the source text on the cited page. Values that cannot be grounded are flagged for a person rather than written to the transaction.

### Is the RESO Web API enough for MLS integration?

It standardises field names and simplifies consumption compared with older RETS feeds, but each MLS still requires its own data access agreement, so plan for licensing time as well as engineering.

### Why use a ledger for commissions instead of calculated fields?

Commission plans change, payments arrive late and adjustments happen. A ledger with versioned plans keeps a reconcilable history that ties out to accounting.
