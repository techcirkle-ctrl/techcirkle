---
layout: post
title: "An Idempotent Metering Pipeline for Usage Billing"
date: 2026-10-23 00:00:00 +0000
categories: ["Engineering", "Data", "Billing"]
tags: ["metering", "idempotency", "datapipeline", "billing"]
description: "Double-billing is a metering problem, not a payments problem. A concrete pipeline design with dedup keys, late arrivals and replay safety."
image: "https://cdn.sanity.io/images/563mnkns/production/0fe6b864d1f98a5e7e65496d214b11c05086c8c0-1600x1066.jpg"
author: "TechCirkle"
---

![Invoice and billing data displayed on a computer](https://cdn.sanity.io/images/563mnkns/production/0fe6b864d1f98a5e7e65496d214b11c05086c8c0-1600x1066.jpg)

Double-billing is almost never a payments bug. It is a metering bug that payments faithfully executed.

The mechanism is mundane. A client sends a usage event. The network times out after the write succeeded. The client retries. Two rows land. At month end the rating engine prices both, and a customer gets charged twice for one API call.

This post is the pipeline shape that makes that impossible, plus the two adjacent problems — late arrivals and replay — that share the same solution.

## Stage 1: Keys originate upstream

The single most important decision: the idempotency key is produced by whatever performed the operation, not by the ingest service.

```json
{
  "idempotency_key": "req_01HQ8ZJ4-acct_8812",
  "account_id": "acct_8812",
  "metric": "api_calls",
  "quantity": 1,
  "occurred_at": "2026-10-23T09:14:22.511Z"
}
```

A key generated at ingest is unique per *delivery*, not per *operation*, which is precisely the wrong granularity. It looks like protection and provides none — arguably worse than having no key at all, because it stops anyone looking closer.

Good key sources: the originating request ID, a database transaction ID, a job execution ID, or a deterministic hash of (subject, metric, window) for aggregated meters.

## Stage 2: Insert-or-ignore on a unique constraint

```sql
CREATE TABLE usage_events (
  idempotency_key  text        PRIMARY KEY,
  account_id       text        NOT NULL,
  metric           text        NOT NULL,
  quantity         numeric     NOT NULL,
  occurred_at      timestamptz NOT NULL,
  received_at      timestamptz NOT NULL DEFAULT now()
);

CREATE INDEX ON usage_events (account_id, metric, occurred_at);
```

```sql
INSERT INTO usage_events (idempotency_key, account_id, metric, quantity, occurred_at)
VALUES ($1, $2, $3, $4, $5)
ON CONFLICT (idempotency_key) DO NOTHING;
```

Let the database enforce it. Application-level "check then insert" has a race window, and under retry storms that window is exactly when it gets hit.

Return 200 on a conflict, not 409. The client's intent was satisfied; a 409 invites it to retry again, which is the opposite of what you want.

## Stage 3: Both timestamps, always

`occurred_at` comes from the device or client. `received_at` is set by the server. Store both.

You need `occurred_at` to bill the correct period, and `received_at` to reason about everything that goes wrong. A client with a skewed clock, a logger that was offline for two days, a backfill that replayed last month — all of these are indistinguishable without both columns, and you cannot reconstruct the missing one later.

## Stage 4: Late arrivals are normal, not exceptional

A battery-powered device or an offline client will deliver two days of history in one burst. If your aggregates were computed when the window closed, they are now wrong.

Two workable approaches:

**Recompute on read.** Aggregate from raw events at query time, with a materialised rollup as a cache that is invalidated when a late event lands in its window. Simple and correct; costs query performance.

**Watermark with corrections.** Keep an open window for a defined grace period, then emit a correction when something arrives after close. More efficient, and it requires rating to accept corrections to a closed period — which it must anyway for other reasons.

Either way, define the grace period explicitly and document what happens to events arriving after it. The answer that causes trouble later is "nobody decided."

## Stage 5: Rating reads, it does not write

```
usage_events  ──►  rate(events, contract_version)  ──►  line_items
```

The rating function takes an event set and a contract version and returns line items. No writes to the event store, no reads from anywhere else, no clock.

This is what makes the pipeline replayable: given the same events and the same contract version, you get the same line items, today or in a year. Which in turn is what makes a historical pricing bug a backfill rather than a few hundred manual credit notes.

The temptation to store the computed price back onto the event row is strong. Resist it. The moment the price lives in the event, you cannot re-derive it, and you have lost the only property that matters.

## Stage 6: A reconciliation job you actually look at

```
daily:
  - count events by (account, metric, day) from raw
  - compare against the rollup
  - compare rollup against billed line items for closed periods
  - alert on any non-zero delta
```

Three populations that should agree: raw events, aggregates, billed quantities. Any disagreement is either a late arrival inside the grace window (expected, and should be labelled as such) or a bug.

Run it daily. A monthly reconciliation finds problems after invoices have been sent, which is the wrong order.

## Where generation fits

Most of this is schema-heavy, well-specified work: the ingest handler, the migrations, the rollup jobs, the reconciliation harness, fixtures generated from historical data, the admin screens for inspecting an account's event stream. Fast to generate and easy to review against a spec.

The parts to specify by hand: what the idempotency key is derived from, how long the grace period is, and the rating logic itself. Those are decisions about your contracts and your clients, and a rating bug does not throw — it silently charges the wrong amount.

## Checklist

- Idempotency key derived from the originating operation, not from the delivery.
- Unique constraint in the database, insert-or-ignore, 200 on conflict.
- Both `occurred_at` and `received_at` stored on every row.
- Grace period for late arrivals defined and documented.
- Rating is a pure function of events plus contract version.
- No computed prices stored on event rows.
- Daily reconciliation across raw, rollup and billed.

The broader architecture and build-versus-buy framing is in TechCirkle's [guide to small business software for billing](https://techcirkle.com/blog/small-business-software-for-billing), and our [custom software development](https://techcirkle.com/development/custom-software-development) work covers how these systems get phased.

## Frequently Asked Questions

### Why must idempotency keys come from the client rather than the server?
Because a key generated when the event is received is unique per delivery, so two deliveries of the same operation get two different keys and both are stored. The key has to be derived from the originating operation — a request or transaction ID — to remain stable across retries.

### Should a duplicate event return 409 or 200?
200. The client's intent has been satisfied, and a 409 signals a problem that invites further retries. Reserve error codes for cases where the client needs to change something.

### How do you handle usage events that arrive days late?
Either aggregate from raw events at read time with rollups as an invalidated cache, or use watermarks with an explicit grace period and emit corrections for anything arriving after a window closes. Whichever you choose, define the grace period and document what happens beyond it.

### Why store both occurred_at and received_at?
Billing needs `occurred_at` to assign usage to the right period, and debugging needs `received_at` to distinguish a skewed client clock from an offline device from a replayed backfill. The missing timestamp cannot be reconstructed afterwards.

### Can I store the computed price on the usage event?
You can, and it will cost you replayability. Once the price is baked into the event you cannot re-derive it from the contract, so retroactive pricing fixes become manual credit notes rather than a backfill.

### How often should billing reconciliation run?
Daily. Compare raw events, aggregates and billed quantities for closed periods, and alert on any unexplained delta. Monthly reconciliation discovers problems after invoices have already gone out.
