---
layout: post
title: "DMS Automotive Software Field Guide: Decision Tables for Dealer Group Tech Teams"
date: 2026-11-19 00:00:00 +0000
categories: ["Reference", "Engineering Notes"]
tags: ["reference", "automotive", "dms", "engineering-notes"]
description: "Reference tables for DMS automotive software decisions: modules, integration checks, buy vs extend vs build, AI use cases, security and timelines."
image: "https://cdn.sanity.io/images/563mnkns/production/e8333ca6cec6896aadea78221a49b2a4b8cdc0bf-1600x1244.jpg"
author: "TechCirkle"
---

![Isometric web dashboard with floating cloud icons, layered data panels and a glowing red globe on a dark blue grid](https://cdn.sanity.io/images/563mnkns/production/e8333ca6cec6896aadea78221a49b2a4b8cdc0bf-1600x1244.jpg)

These notes compress our full [DMS automotive software guide](https://techcirkle.com/blog/dms-automotive-software) into reference tables for engineering teams, IT leads and product managers working on dealer management system evaluations, integrations or builds.

## Contents

- Mental model
- Module table
- Integration checks
- Data-rights clauses
- Buy vs extend vs build
- Reference architecture
- AI use case table
- Security controls
- Timeline ranges
- Conversion checklist
- Failure modes

## Mental model

The DMS is a bundle of tightly coupled ledgers (accounting, inventory, deals, F&I, service, parts, customers). It is the system of record. Every other dealership tool reads from or writes to it. Treat it as **authoritative for accounting** and **one data source among several for everything else**.

## Module table

| Module | Must have | Where platforms differ |
|---|---|---|
| Accounting / GL | Multi-rooftop, OEM chart of accounts, month-end statements | Rarely; table stakes |
| Inventory | VIN decode, aging, floorplan, recon cost | Recon workflow, pricing history |
| Sales / desking | Deal structuring, trades, incentives, deal jacket | UX, digital retail handoff |
| F&I | Contracts, lender submission, product menus, e-contracting | Lender coverage, compliance checks |
| Service | Appointments, ROs, tech time, MPI, warranty, recalls | Capacity scheduling, mobile tools |
| Parts | Stock, special orders, counter/wholesale, OEM ordering | Integration with service |
| Reporting | OEM statements, sales and service reports | Group-level, cross-platform views |

## Integration checks

| Question | Good answer | Red flag |
|---|---|---|
| Documented, versioned API? | Public reference plus sandbox | "Available via partner program" only |
| Per-vendor access cost? | Clear, predictable or none | Opaque, negotiated per vendor |
| Dealer self-export? | Full history, documented format, on demand | Only at termination, via services |
| Events or webhooks? | Yes, for core entities | Scheduled polling only |
| Exit terms? | Defined transition support and timeline | Undefined |

## Data-rights clauses

| Clause | Why it matters |
|---|---|
| On-demand full historical export | Analytics, AI and a credible exit |
| Right to authorise third-party access | Stops per-tool access fees landing on you |
| Retention and deletion obligations | Privacy and safeguarding compliance |
| Transition assistance with timeline and cost | Makes switching realistic |

## Buy vs extend vs build

| Option | Fits | Main risk |
|---|---|---|
| Buy and standardise | Uniform brands, small IT team | Roadmap tied to vendor |
| Extend (data layer + API + apps) | Most dealer groups, mixed DMS estates | Requires owning a data platform |
| Build full DMS | Niche-segment software companies, OEM agency retail | Multi-year accounting and lender integration effort |

## Reference architecture (extend option)

```text
[DMS A] [DMS B] -> connectors (change detection, raw append-only store)
                -> canonical model (vehicle, customer, deal, RO, RO line, GL)
                -> internal API + events (auth, scopes, rate limits, audit log)
                -> apps (service lane, recon, group BI) + AI agents
                -> narrow write-back services (idempotent, human-approved)
```

## AI use case table

| Use case | Input | Output | Human role | Metric |
|---|---|---|---|---|
| RO drafting | Complaint, history, open recalls | Suggested lines and op codes | Advisor approves | Time per RO |
| Warranty pre-check | Closed warranty RO | Issues list | Warranty admin fixes | Rejection rate |
| Funding check | Signed contract, deal, stipulations | Mismatch list | F&I corrects | Contracts in transit |
| Service status replies | Live RO status | Customer message | Escalation path | Inbound call volume |
| Accounting exceptions | GL schedules | Exceptions with likely cause | Controller reviews | Days to close |

Agent design rule: read-mostly tools, proposals stored outside the DMS, human approval for anything touching money or contracts. More on that pattern: [agentic workflow development](https://techcirkle.com/agentic-workflow-development).

## Security controls

| Control | Note |
|---|---|
| Role-based access by job function | Sales, F&I, service, accounting differ |
| Field-level encryption | Credit applications, government IDs |
| MFA everywhere | Required under FTC Safeguards Rule for US dealers arranging finance |
| Audit log of record access | Who viewed which deal or customer |
| Vendor reviews and DPAs | Every connected tool |
| Outage runbook | Tested; informed by the June 2024 provider outage |

## Timeline ranges (indicative, not quotes)

| Work item | Typical range |
|---|---|
| Data pipeline + warehouse, one DMS platform | 8–14 weeks |
| Internal API + first two integrations rerouted | 6–10 weeks |
| Service lane or recon app, one-rooftop pilot | 10–16 weeks |
| AI use case to supervised pilot (clean data) | 6–10 weeks |
| Multi-tenant niche DMS product, first version | 12–24 months |

Biggest driver: number of DMS platforms and cross-rooftop data inconsistency. Run a data audit first.

## Conversion checklist

- Extract full history into your own warehouse before conversion starts.
- Cut over at the start of an accounting month.
- Run a parallel close at a pilot rooftop.
- Reconnect integrations via your internal API.
- Train by role with real data; floor support for two weeks.
- Track days to close, ROs per advisor and contracts in transit weekly.

## Failure modes

- Evaluating screens instead of data access.
- Letting every tool integrate directly with the DMS.
- Starting AI before op codes and pay types are normalised.
- Writing to accounting through unsupported interfaces.
- No outage plan.

For the narrative version and the business reasoning behind each table, see the [full guide](https://techcirkle.com/blog/dms-automotive-software). For build help, see [custom software development](https://techcirkle.com/development/custom-software-development).

## Frequently Asked Questions

### Which table should drive a DMS evaluation?

The integration checks. Data access determines total cost and flexibility more than module features do.

### Why keep raw DMS payloads in an append-only store?

So mapping bugs discovered later can be fixed by replaying history rather than re-extracting it.

### Are the timeline ranges reliable for budgeting?

They are orientation ranges from typical work, not quotes. A data audit gives a far better estimate.

### What is the riskiest part of a DMS extension layer?

Write-back. Keep it narrow, idempotent, supported and human-approved.

### When does building a full DMS make sense?

Mainly for software companies serving under-served segments or manufacturers running agency-model retail.
