---
layout: post
title: "ADR-007: Custom Core, Low-Code Edges for a B2B SaaS Platform"
date: 2026-10-20 00:00:00 +0000
categories: ["Architecture", "Engineering"]
tags: ["adr", "architecture", "saas", "lowcode"]
description: "An ADR template for SaaS teams choosing custom development, low-code or AI app builders, with context, options, consequences and revisit triggers."
image: "https://cdn.sanity.io/images/563mnkns/production/5dc66e5eb59e536ff821f400871fd195627b92ec-1600x1066.jpg"
author: "TechCirkle"
---

![Engineer typing with low-code platform icons displayed on a virtual screen](https://cdn.sanity.io/images/563mnkns/production/5dc66e5eb59e536ff821f400871fd195627b92ec-1600x1066.jpg)

Architecture decision records (ADRs) are short documents that capture one important technical choice, the context behind it and its consequences. They are especially useful for the build-versus-platform question, because the reasoning tends to be forgotten long before the consequences arrive.

Below is a worked example for a fictional but typical B2B SaaS company. Copy the structure, replace the specifics, and keep it in your repository. The reasoning draws on TechCirkle's longer guide to [custom development vs low code for SaaS founders](https://techcirkle.com/blog/custom-development-vs-low-code-saas-founders).

---

## ADR-007: Custom core, low-code edges

**Status:** Accepted
**Date:** 2026-10
**Deciders:** CEO, CTO, Head of Product

### Context

- The product is a B2B workflow tool currently running entirely on a low-code platform. It has around 120 paying companies and roughly 3,000 active end users.
- Platform fees are about 9 percent of monthly recurring revenue and have grown faster than revenue for three consecutive quarters, because external users are billed as seats.
- Two enterprise prospects in the UK and UAE have asked how tenant data is isolated and whether it can be hosted in-region. The platform offers neither database-level isolation nor region choice.
- The roadmap for the next 12 months includes retrieval over customer documents and an assistant that can update records on a user's behalf.
- A Series A raise is planned in roughly 9 to 12 months.
- The team has one senior engineer and access to an external development partner. AI coding assistants are in use.

### Decision drivers

1. Tenant isolation enforceable at the data layer.
2. Gross margin that improves, not erodes, with scale.
3. Ownership of business logic and source code ahead of due diligence.
4. Ability to build AI features with per-tenant retrieval, permission checks, audit logs and cost metering.
5. Preserve the operations team's ability to change internal tools without engineering.

### Options considered

**Option A: Stay fully on low-code.** Lowest short-term effort. Fails drivers 1, 2, 3 and 4. Acceptable only if growth stalls.

**Option B: Full rewrite in custom code.** Satisfies drivers 1 to 4. Fails driver 5 and carries big-bang delivery risk during a fundraising window.

**Option C: Regenerate with an AI app builder.** Fast to a demo. Generated output reviewed in a spike had permissive data rules, no tenancy model and backend services tied to the vendor. Fails drivers 1 and 3 without substantial rework.

**Option D: Hybrid. Custom core, low-code edges.** Customer-facing app, data model, tenancy, billing, public API and AI layer move to custom code. Internal admin and ops dashboards remain on low-code, reading and writing through the new API.

![Developer focused on code at a computer in an office](https://cdn.sanity.io/images/563mnkns/production/018158cfc420c189e1ddf6bddded64b52a4d620e-1600x1067.jpg)

### Decision

Adopt **Option D**, delivered incrementally using a strangler approach:

1. Stand up Postgres with row-level security keyed on `tenant_id`; sync data from the platform.
2. Build a versioned API; repoint low-code screens to read from it.
3. Flip writes entity by entity until Postgres is authoritative.
4. Rebuild customer-facing screens; leave internal tools on low-code.
5. Build the AI layer (tenant-filtered retrieval, permissioned agent tools, prompt versioning, evaluation in CI, token metering) on the new core.

AI coding assistants will be used for scaffolding, migrations and test generation. All generated code requires review by a senior engineer, with particular attention to authorisation paths.

### Consequences

**Positive**

- Data-layer isolation and regional deployment become possible, unblocking enterprise deals.
- Customer-facing costs scale with infrastructure rather than seats.
- Clear IP ownership for due diligence.
- AI features can be built with proper controls.

**Negative**

- Two toolsets to maintain during and after migration.
- Upfront engineering spend, estimated in the range of a lean custom first release.
- Temporary sync complexity while data flows both ways.

### Revisit triggers

Re-open this decision if platform fees for the retained low-code tools exceed 3 percent of revenue, if the ops team needs workflows that require core data model changes, or if a new platform demonstrably satisfies drivers 1 to 4.

---

## Using this template

The value of an ADR is not the format, it is forcing the context and drivers to be written down before the decision. If your team is facing the same choice and wants an outside review, TechCirkle's [custom software development](https://techcirkle.com/development/custom-software-development) team can help pressure-test the options.

## Frequently Asked Questions

### What is an architecture decision record?

A short document capturing one significant technical decision, its context, the options considered and the expected consequences, stored alongside the code.

### Why use an ADR for low-code versus custom decisions?

Because the consequences often appear 12 to 24 months later. A written record explains why the choice was made and when it should be revisited.

### Should AI app builders be listed as an option in the ADR?

Yes. They are a real third option in 2026. Evaluate their output against the same drivers, especially tenancy, security and ownership.

### How often should a build-versus-platform ADR be revisited?

Whenever a revisit trigger fires, and at least every six months for a fast-growing SaaS product.
