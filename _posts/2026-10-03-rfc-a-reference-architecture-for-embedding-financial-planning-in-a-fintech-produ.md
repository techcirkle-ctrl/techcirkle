---
layout: post
title: "RFC: A Reference Architecture for Embedding Financial Planning in a Fintech Product"
date: 2026-10-03 00:00:00 +0000
categories: ["Engineering", "Architecture"]
tags: ["fintech", "architecture", "api", "wealth management", "llm"]
description: "An RFC-style reference architecture for embedding a licensed planning engine in a fintech product: API boundary, entitlements, data model and AI."
image: "https://cdn.sanity.io/images/563mnkns/production/cd39c13bcbc261ec411cece508d476209e7a6bb5-1600x1067.jpg"
author: "TechCirkle Editorial Team"
---

![Financial advisor reviewing a plan on a laptop with clients, representing planning embedded in a fintech product](https://cdn.sanity.io/images/563mnkns/production/cd39c13bcbc261ec411cece508d476209e7a6bb5-1600x1067.jpg)

**Status:** Reference draft
**Audience:** Fintech product leads, platform architects, engineering managers at wealth platforms and broker-dealers

## 1. Context

Off-the-shelf advisor planning tools are designed to be used, not embedded. When a wealth platform, a digital advice product or a bank wants planning inside its own advisor desktop or client app, under its own brand, the usual tools fit poorly.

This document describes a durable pattern: license the calculation engine, own everything around it. It is not a vendor recommendation; the boundaries below should hold whichever engine you choose.

## 2. Goals and non-goals

**Goals**

- Planning results presented natively in our product, in our design system
- A single household data model we control, independent of the engine vendor
- Swappable engine: replacing the vendor should be weeks of adapter work, not a rewrite
- Every plan presented to a client reproducible, with its inputs and assumptions, at any later date
- AI used for unstructured work only, never as the source of a number

**Non-goals**

- Writing our own tax, withdrawal-sequencing or Monte Carlo logic
- Letting a language model produce recommendations without advisor review

## 3. High-level components

1. **Household service.** Owns people, relationships, accounts, goals, income, expenses and documents. Our source of truth.
2. **Engine adapter.** Translates our household model into the engine's request format and normalises responses. The only code that knows the vendor's schema.
3. **Plan store.** Immutable, versioned plan snapshots: input payload, assumption set, engine version, outputs, timestamps.
4. **Entitlements service.** Decides who can see, edit, run or present what.
5. **AI layer.** Document intake, narratives, meeting summaries and natural-language scenario parsing, behind a model gateway.
6. **Integration hub.** Custodian feeds, aggregation, CRM sync, e-signature and compliance archive.
7. **Presentation layer.** Advisor desktop and client app, consuming only our own APIs.

## 4. The engine boundary

Treat the licensed engine as a pure function: `plan = engine(household_snapshot, assumptions)`.

- Requests are built from a frozen household snapshot, never live records, so a run is reproducible.
- The adapter pins the engine version and records it with every result.
- No business logic lives in the adapter beyond mapping and validation; rules belong in the household service.
- Calls are idempotent on a hash of snapshot plus assumptions plus engine version. Identical inputs return the cached plan.

When evaluating engine vendors, the test is simple: can every input you need be passed via a documented API, and can every output be retrieved in structured form? A few read-only endpoints are not enough to embed against.

## 5. Data model essentials

At minimum:

- `household` with members, relationships and jurisdiction
- `account` with ownership, tax treatment, source (custodian, aggregated, manual) and `as_of` timestamp
- `goal`, `cashflow`, `liability`, `insurance_policy`
- `assumption_set`, versioned and owned by the firm (the "house view"), referenced by id
- `document`, with extraction status and links to the fields it populated
- `plan_snapshot`, immutable, referencing household snapshot id, assumption set id and engine version

Every field populated from a document should carry provenance: which document, which extraction run, and who confirmed it. This makes review and audit tractable.

![Clients and a financial adviser reviewing plan results together on a laptop](https://cdn.sanity.io/images/563mnkns/production/13fe78c0d91409b5d0dc375ff509f6141e26fec5-1600x1020.jpg)

## 6. Entitlements

Planning data is among the most sensitive personal data a product holds. Model permissions explicitly rather than inferring them from the UI.

- **Roles:** client, household member, advisor, paraplanner, supervisor, compliance, support
- **Scopes:** per household, per firm or branch, per action (`view`, `edit_inputs`, `run_plan`, `present_plan`, `approve_ai_output`)
- **Rules:** a household member sees shared goals but not another member's private accounts unless granted; support access is time-boxed and logged
- Enforce in the API, not the client app. Retrieval for AI features must apply the same checks so a model never sees data the requesting user could not open.

## 7. AI layer

Keep a clean separation: deterministic numbers from the engine, language from models.

- **Document intake:** extract fields into a staging area, show confidence, require human confirmation before writing to the household service.
- **Narratives:** generated from a `plan_snapshot`, not from raw data, so text always matches the numbers.
- **Scenario parsing:** natural language becomes a structured scenario diff; the engine runs it.
- **Gateway:** every model call passes through one service that handles routing, redaction, regional processing, retention settings and logging.
- **Audit:** store prompt reference, output, and the advisor's action (accepted, edited, rejected) next to the plan version it relates to.

Orchestrating multi-step flows such as "ingest documents, update household, rerun plan, draft review agenda" is a good fit for [agentic workflows](https://techcirkle.com/agentic-workflow-development) with explicit approval gates.

## 8. Compliance and recordkeeping

- A presented plan is a record. Store the snapshot, assumption set, engine version and rendered output together.
- Retention follows the strictest jurisdiction you operate in.
- Vendor due diligence evidence (security reports, data-processing terms, AI data handling) lives alongside the integration's documentation.

## 9. Alternatives considered

- **Iframe the vendor UI.** Fastest to ship; loses design control, data ownership and audit coherence.
- **Build the engine.** Full control, very high cost, ongoing regulatory upkeep. Justified only when planning logic is the product.
- **Licensed engine, owned everything else (this RFC).** Moderate cost, strong control, vendor swappable.

## 10. Open questions

- Which engine capabilities are mandatory for your clients (equity compensation, entities, cross-border)?
- Where must AI processing run to meet data-residency commitments?

## Further reading

The source guide covers advisor tool categories, evaluation criteria, compliance requirements and rollout. [Read the full guide on TechCirkle](https://techcirkle.com/blog/best-financial-planning-software-for-advisors). If you want help designing the adapter, data model or AI layer, our [custom software development](https://techcirkle.com/development/custom-software-development) team works on exactly these builds.

## Frequently Asked Questions

### What does it mean to embed a financial planning engine?

Using a licensed calculation engine through its API while your own product owns the data model, user experience, permissions and records, so planning appears natively under your brand.

### Why keep a separate household data model instead of using the vendor's?

It keeps you independent of any one engine, lets you add provenance and permissions, and makes replacing the vendor an adapter change rather than a migration.

### How do you make financial plans reproducible?

Run the engine on frozen household snapshots, version the assumption set, pin the engine version, and store all three with the output as an immutable plan snapshot.

### Where should AI sit in an embedded planning architecture?

Around the engine, behind a gateway: document intake, narratives and scenario parsing. Numbers always come from the engine, and AI outputs are logged with the advisor's decision.

### How should permissions work for planning data?

Define roles and per-action scopes explicitly, enforce them in the API, and apply the same checks to AI retrieval so models only see what the requesting user is allowed to see.

### When is building your own planning engine justified?

When the planning logic itself is the product you sell and you can fund ongoing tax and regulatory maintenance. Otherwise, license the engine and own the layers around it.
