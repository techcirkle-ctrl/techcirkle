---
layout: post
title: "Migrating Off Point-to-Point Integrations Without a Big Bang"
date: 2026-09-18 00:00:00 +0000
categories: ["Engineering", "Migration"]
tags: ["integration", "migration", "architecture", "engineering", "erp"]
description: "A staged plan for moving a tangled point-to-point integration estate to a hub, with inventory, parallel running and the connections worth leaving alone."
image: "https://cdn.sanity.io/images/563mnkns/production/c3f76722d2e652e827aed03af86d39a49ee03c74-1600x879.jpg"
author: "TechCirkle Editorial Team"
---

![Integration estate and system connections](https://cdn.sanity.io/images/563mnkns/production/c3f76722d2e652e827aed03af86d39a49ee03c74-1600x879.jpg)

Nobody gets to design an integration estate from scratch. What you have is a decade of individually reasonable decisions that add up to something nobody fully understands, and the realistic question is how to improve it without a stop-the-world rewrite.

This is a staged plan. Each stage is independently valuable, which matters because migration programmes that only pay off at the end tend not to reach the end.

## Stage 0: The Honest Inventory

A week, and it is the highest-value week in the programme.

Document every flow that exists. Not every flow you know about — every flow. Sources to check:

- Scheduled jobs on every server, including the one nobody has logged into since 2021
- Database links and replication configurations
- API credentials issued to internal systems (audit which ones are still being used)
- SFTP accounts and what writes to them
- Any middleware already in place, and what is routed through it

For each flow, record: source, target, entity, trigger (schedule or event), transformation logic location, owner, and last modified date.

Two things always come out of this. There are flows still running that everyone believed were decommissioned. And there are flows with no owner — the person left and nobody inherited them.

Produce a diagram. Not a beautiful one; an accurate one. Put it somewhere everybody can find it.

## Stage 1: Stop the Bleeding

Before migrating anything, mandate that every **new** integration goes through the hub. No exceptions for urgency, because urgency is how every existing point-to-point connection was justified.

This needs real authority behind it, and it needs the hub to be genuinely easy to use — if going through it takes three weeks and a direct connection takes two days, the rule will be broken and you will have a governance problem instead of an architecture problem.

Make the paved path faster than the shortcut. That is an engineering investment, not a policy one.

## Stage 2: Instrument What Exists

Before changing behaviour, gain visibility into it.

Add reconciliation to the existing flows, even the ones you intend to replace. Compare counts and value totals between systems on a schedule, and alert on divergence.

This does two things. It tells you which flows are already broken — in most estates, at least one is, and nobody knows. And it gives you the comparison baseline you will need during parallel running.

Add basic telemetry too: run duration, record counts, error counts per flow. You need to know which flows are large and which are fragile before you sequence the migration.

![Migration planning and parallel running](https://cdn.sanity.io/images/563mnkns/production/32d709526bcf108af533811484bdc95ccee6f374-1600x959.jpg)

## Stage 3: Sequence by Value, Not by Ease

The instinct is to migrate the simplest flow first, for a quick win. Resist it.

Migrate by a combination of **failure frequency** and **financial materiality**. The flow that breaks monthly, or the one carrying data that ends up in the ledger, goes first. That is where the return is, and it is what justifies continuing the programme when attention drifts.

A rough scoring approach:

| Factor | Weight |
|---|---|
| Incidents in the last 12 months | High |
| Financial materiality of the data | High |
| Number of downstream consumers | Medium |
| Age and documentation quality | Medium |
| Technical difficulty | Low (a tiebreaker, not a driver) |

## Stage 4: Migrate One Flow, Properly

For each flow:

1. **Document the current behaviour**, including the special cases. Read the transformation code; a model can draft this quickly from the source, which is far faster than a human reading it, though a human must verify it.
2. **Rebuild on the hub**, including idempotency keyed on the business event and explicit handling of every special case you found.
3. **Run in parallel.** Both paths process the same inputs; the new path writes to a staging location rather than the live target. Compare outputs automatically and log every mismatch.
4. **Run parallel for several weeks**, not days. You need to cross a month boundary to catch period-end behaviour, and you need enough volume to hit the rare cases.
5. **Cut over behind a flag.** Keep the old path dormant but runnable for one release cycle.
6. **Decommission properly** — remove the credentials, delete the scheduled job, update the diagram.

Step 3 is where the value is. The mismatches you find during parallel running are real defects — timezone boundaries, soft deletes, late-arriving records, the special case nobody documented. Every one of them would have been a production incident.

## Stage 5: Know When to Stop

Some point-to-point connections should stay.

A stable, low-volume, well-documented connection between two systems that will never talk to anything else gains nothing from migration. It adds a hop, a dependency and an operational surface for architectural tidiness alone.

Leave it. Document it, reconcile it, and move on. The goal is maintainability, not purity, and a migration programme that insists on completeness usually runs out of political capital before it runs out of flows.

## What Success Looks Like

Not "everything is on the hub." The measures worth tracking:

- **Time to answer "what breaks if we change this field?"** — should be minutes, from the inventory
- **Mean time to detect a silent failure** — should be hours, from reconciliation
- **Number of flows with no named owner** — should be zero
- **New integrations going through the hub** — should be all of them

Those four are achievable within a year on most estates, and they matter more than the percentage migrated.

The full architectural context — the four integration patterns, canonical data models, master data, reliability design, security and cost — is in our guide to [ERP integrations](https://techcirkle.com/blog/erp-integrations).

## Frequently Asked Questions

### Where do we start if nobody knows what integrations exist?

With a week-long inventory covering scheduled jobs on every server, database links, API credentials still in use, SFTP accounts and any existing middleware. Record source, target, entity, trigger, transformation location, owner and last modified date for each. Expect to find flows everyone believed were decommissioned.

### Should we migrate the easiest flow first for a quick win?

No. Sequence by failure frequency and financial materiality instead. The flow that breaks monthly or carries data that reaches the ledger delivers the return that justifies continuing the programme. Technical difficulty should be a tiebreaker, not a driver.

### How long should parallel running last before cutover?

Several weeks, crossing at least one month boundary so period-end behaviour is exercised, and long enough to hit low-frequency cases at your actual volumes. The mismatches found during this period are real defects that would otherwise have been production incidents.

### How do we stop new point-to-point connections appearing during the migration?

Mandate that new integrations use the hub, and make the hub genuinely faster to use than a direct connection. If the paved path takes three weeks and the shortcut takes two days, the rule will be ignored, and you will have converted an architecture problem into a governance problem.

### Is it acceptable to leave some point-to-point connections in place?

Yes. A stable, low-volume, well-documented connection between two systems that will never talk to anything else gains nothing from migration. Document it, add reconciliation, and move on — the objective is maintainability, not architectural completeness.
