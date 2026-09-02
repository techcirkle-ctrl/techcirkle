---
layout: post
title: "Architecture Notes — A Cloud Restaurant Point of Sale System"
date: 2026-09-02 00:00:00 +0000
categories: ["Architecture", "Engineering"]
tags: ["Architecture", "Distributed Systems", "Restaurant Tech", "SaaS", "AI"]
description: "A structured architecture reference for a cloud restaurant POS — local-first storage, merge semantics, domain model, tenancy and edge observability."
image: "https://cdn.sanity.io/images/563mnkns/production/2964892bd3ff95990023d66cc7f2bd4bc8d39d80-1600x900.jpg"
author: "TechCirkle Editorial Team"
---

![Restaurant point of sale terminal](https://cdn.sanity.io/images/563mnkns/production/2964892bd3ff95990023d66cc7f2bd4bc8d39d80-1600x900.jpg)

Structured architecture notes for a cloud-based restaurant point of sale system, organised by component. Written as a reference rather than an essay.

## Constraint set

- Network: consumer-grade, shared, fails during service
- Hardware: heterogeneous tablets, multi-year age spread, thermal throttling near heat sources
- Users: high speed, minimal training, physical queue forms on hesitation
- Failure cost: an outage during service means the business cannot take money

Every decision below follows from these.

## Terminal storage

- Local durable database holding menu, open checks, session orders, outbound operation queue
- All user-facing writes local and synchronous; no interaction blocks on network I/O
- Synchronisation is a background reconciliation process, never an interaction gate
- Identifiers generated client-side; human-facing sequences assigned per venue per day with a terminal prefix

## Check representation

- Append-only operation stream, not a mutable document
- Operation: `{ id, checkId, terminalId, seq, ts, type, payload }`
- State derived by folding operations
- Merge of divergent terminals: union of operation sets, deterministic replay
- Ordering by per-terminal sequence plus logical clock; wall-clock timestamps for display and audit only

Rationale: document-level last-write-wins discards one terminal's operations, producing either uncharged items or items billed after being voided.

## Arbitration set

Operations requiring a single authority:

- Check settlement
- Void of an item already fired and served
- Any operation with an external side effect (refund, loyalty redemption)

Require connectivity for these, surface the requirement in the interface, and consider a LAN-elected coordinator so the failure domain is the local switch rather than the internet.

## Divergence bounds

- Maximum offline window of one service day
- Beyond the bound, the terminal requires reconciliation before accepting new work
- Bounding also makes conflict testing a finite problem

![Restaurant manager with a payment terminal](https://cdn.sanity.io/images/563mnkns/production/925e088f9ef873fd70d2113f32b06e1c7463b71f-1600x1067.jpg)

## Domain model

Four distinct concepts, frequently conflated:

- **Order** — what the guest requested
- **Ticket** — what a kitchen station must prepare, routed and fired independently
- **Check** — the financial document, which may span orders or be split across guests
- **Seat** — position at a table, enabling per-guest splitting without reconstruction

Conflating these makes course firing, table transfer and guest splitting into special cases grafted onto a model that cannot express them.

## Menu and modifiers

- Modifier groups as first-class shared entities, attached to items by reference
- Group attributes: selection type, required flag, min and max selections, pricing strategy
- Pricing strategies must include price-difference upgrades, not only flat additions
- One level of conditional availability (set menus, size-dependent options)
- Menu versioned with effective dates; version referenced on every order line
- Channel as a dimension on pricing and availability, not a boolean flag
- Availability causes modelled distinctly by lifetime (out of stock tonight vs seasonal removal)

## Financial layer

- Immutable records; corrections as new entries referencing the original
- Idempotent payment operations
- Authorisation and capture modelled separately to accommodate tipping
- Partial settlement as a first-class state for split payments
- Semi-integrated card terminals to keep card data out of application PCI scope

## Real-time layer

- Kitchen display communication over the venue LAN, sub-second target
- Cloud as durability and reporting channel, not as the message bus
- Thermal printer support: escape codes, discovery, paper-out handling, per-model quirks

## Tenancy

- Hierarchy: brand → region → venue → terminal
- Settings, menus, pricing and permissions inheritable at each level, overridable below
- Permission model built on the hierarchy rather than flat roles
- Reporting scoped by hierarchy position

## Data retention for analytics

- Store item-level, timestamped operations including modifiers and voids
- Do not collapse to settled check totals

This is the input for item-level demand forecasting, labour scheduling derived from those forecasts, menu engineering, and anomaly detection on voids and discounts. History discarded here cannot be reconstructed.

## Observability

Edge-weighted signals:

- Sync queue depth and time since last successful sync, per terminal
- Merge conflicts requiring arbitration
- Printer and card reader availability
- Crash rate segmented by device model
- Transaction anomaly detection against each venue's own baseline

## Rollout

1. One venue, parallel run against the existing system, one full month, including a real outage
2. Three to five venues of deliberately different formats (quick service, full service, bar)
3. General availability only after a full month covering a public holiday

Treat every staff workaround observed during piloting as a specification that was missed.

Full guide: [Cloud Based Restaurant POS Systems: The 2026 Build Guide](https://techcirkle.com/blog/cloud-based-restaurant-pos-systems). Related: [custom software development](https://techcirkle.com/development/custom-software-development).

## Frequently Asked Questions

### Which component should be built first?

The synchronisation layer, before the interface, tested against simulated partition and divergence from the start. It has the least forgiving failure mode and the highest cost of late redesign.

### Is a venue-local coordinator necessary?

Only if settlement must work during an internet outage, which many operators require. It reduces the failure domain from the internet to the local switch, at the cost of a leader election among terminals.

### How is multi-tenancy best isolated?

Logical isolation within shared infrastructure, with the hierarchy as a first-class attribute on every record, is sufficient for most deployments. Physical isolation becomes relevant only under specific contractual or regulatory requirements.

### What are the fiscalisation requirements to watch for?

Numerous countries mandate certified fiscal devices or signed digital receipts transmitted to a tax authority, with per-country rules that change periodically. These land in the receipt pipeline, so research target markets before the architecture solidifies.

### How long is a realistic first release?

Nine to fifteen months for a single market with one payment provider and a focused feature set. The sync layer and the menu and modifier model account for most of the variance.
