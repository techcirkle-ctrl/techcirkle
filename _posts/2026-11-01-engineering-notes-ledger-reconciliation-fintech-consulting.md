---
layout: post
title: "Engineering Notes: Ledger and Reconciliation Decisions in Fintech Consulting"
date: 2026-11-01 00:00:00 +0000
categories: ["Engineering", "Fintech"]
tags: ["ledger", "reconciliation", "fintech", "databases"]
description: "Working notes on fintech ledger design: double-entry invariants, provider reconciliation, exception queues and decisions to settle early."
image: "https://cdn.sanity.io/images/563mnkns/production/740ddcd77db26754726314b7b467ea29b9d89bf1-1600x1067.jpg"
author: "TechCirkle"
---

![Consultants reviewing financial reports in front of a screen of market data](https://cdn.sanity.io/images/563mnkns/production/740ddcd77db26754726314b7b467ea29b9d89bf1-1600x1067.jpg)

These are working notes from fintech engagements. They focus on one area that consulting projects tend to underweight: the ledger and the reconciliation loop around it. Other architecture choices — frameworks, clouds, even vendors — can change. Ledger semantics rarely do.

## 1. Decide what the ledger is

Three options come up in almost every discovery:

- **Build your own ledger** — most control, most responsibility.
- **Ledger-as-a-service** — faster, with a dependency you must be able to export from.
- **Rely on a BaaS partner's ledger** — fastest, and the hardest to leave.

The deciding question is not speed. It is: *if this partner disappeared or doubled its prices, could we reconstruct every customer balance from data we own?* If the answer is no, you have outsourced your source of truth.

## 2. Non-negotiable invariants

Whatever you choose, the ledger should satisfy:

- **Double-entry.** Every transaction's debits equal its credits.
- **Append-only.** Corrections are new entries, never updates or deletes.
- **Integer minor units.** No floating point. Store currency explicitly.
- **Idempotent writes.** A retried request with the same key cannot post twice.
- **Explicit state transitions.** `pending → posted → settled` (or your equivalent), each with an owner.

```sql
-- invariant check, run continuously
SELECT txn_id
FROM ledger_entries
GROUP BY txn_id
HAVING SUM(CASE WHEN direction = 'debit' THEN amount_minor ELSE -amount_minor END) <> 0;
```

An empty result is the only acceptable result.

## 3. Reconciliation is a product, not a script

Every external party — processor, sponsor bank, card network, payout provider — sends you a version of the truth. Reconciliation compares it to yours.

A workable design:

1. Ingest provider statements and files into raw tables, unchanged.
2. Normalise to a common shape behind each vendor adapter.
3. Match against ledger entries by reference, amount, currency and date window.
4. Push every unmatched item into an **exception queue** with an owner and SLA.
5. Close exceptions with correcting entries — never by editing history.

Teams that skip steps 4 and 5 end up with spreadsheets, and spreadsheets do not pass audits.

![An audit team checking reports with a tablet and paper in a finance office](https://cdn.sanity.io/images/563mnkns/production/da7a854813e4c7fe8ad9df6a0bd68e9cbb8b3465-1600x1067.jpg)

## 4. Where AI helps (and where it must not)

Exception queues are a good place for AI assistance. A model can read a mismatch, the related provider records and recent ledger history, and propose an explanation plus candidate correcting entries for a human to approve. That removes a lot of manual digging.

What it must not do is post entries itself. The approval step, the posting and the invariants stay in deterministic code. Log every suggestion with its sources, model version and the reviewer's decision. More on this pattern in our [LLM integration](https://techcirkle.com/llm-integration) notes.

## 5. Questions to settle in the first month

- Which system is the source of truth for each balance type?
- What is the matching key per provider, and how are partial or batched settlements handled?
- What is the SLA for exception resolution, and who owns it?
- How long are raw provider files retained?
- Can we rebuild balances from our own data alone?

If a consulting engagement leaves these open, the most expensive decisions in the system are still unmade.

## 5b. Common failure modes we see

- **Balances cached in product tables** and updated in place, drifting from the ledger with no audit trail.
- **Provider IDs used as primary keys**, so a second provider breaks every join.
- **Webhooks applied directly** to money state without re-fetching, so duplicate or out-of-order events double-post.
- **Timezone-naive settlement dates**, causing daily matches to fail around midnight UTC.
- **Fees and FX booked as a single net amount**, hiding the components auditors and finance teams ask about.
- **Reconciliation run manually** by one person, so it stops when they go on holiday.

Each of these is easy to avoid at design time and painful to fix once real customer money depends on it.

## 6. Signals of a healthy setup

- Invariant checks run continuously and alert on any breach.
- Exception queue size and age are tracked like on-call metrics.
- Manual reconciliation hours trend down release over release.
- Switching a provider is a project measured in weeks, not a rewrite.

---

These notes expand one section of a longer guide on choosing and working with fintech consultants: [financial technology consulting services](https://techcirkle.com/blog/financial-technology-consulting-services). For ledger and payments builds, see our [custom software development](https://techcirkle.com/development/custom-software-development) work.

## Frequently Asked Questions

### Should a fintech build its own ledger?

Not always, but you should own enough data to reconstruct every customer balance. Building, using ledger-as-a-service, or relying on a partner each trade speed against control.

### Why must a ledger be append-only?

Append-only history makes every change traceable. Corrections become new entries, which auditors and reconciliation processes can follow.

### Why avoid floating point for money?

Floating point introduces rounding errors. Storing integer minor units with an explicit currency keeps amounts exact.

### What is an exception queue in reconciliation?

A tracked list of items that did not match between the ledger and provider records, each with an owner and resolution deadline.

### Can AI fix reconciliation mismatches automatically?

It can propose explanations and correcting entries, but a human should approve them, and posting should stay in deterministic code that enforces ledger invariants.

### How do you know reconciliation is working?

Invariant checks pass continuously, exception queues stay small and young, and manual reconciliation effort falls over time.
