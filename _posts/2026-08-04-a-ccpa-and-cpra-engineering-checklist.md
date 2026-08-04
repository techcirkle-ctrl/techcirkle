---
layout: post
title: "A CCPA and CPRA Engineering Checklist"
date: 2026-08-04 00:00:00 +0000
categories: ["Engineering", "Privacy"]
tags: ["privacy", "ccpa", "architecture", "compliance", "devops"]
description: "A concrete engineering checklist for CCPA and CPRA — data inventory, deletion orchestration, backup strategy, log hygiene, and the CI test that proves it."
author: "TechCirkle Editorial Team"
---

This is the engineering version of California's privacy obligations — written as system requirements rather than as a legal summary. It is the list we work through when a product serves California residents, and it is far cheaper to satisfy at design time than to retrofit.

Note at the outset: these obligations attach to serving California residents, not to where your engineering team is located. A San Francisco address confers no compliance advantage; a demonstrated prior implementation does.

---

## 1. Maintain a personal data inventory

**Requirement:** a maintained, verifiable list of every store that holds personal information, with the deletion mechanism for each.

Minimum coverage:

- [ ] Primary database(s)
- [ ] Read replicas, caches (Redis and equivalents), materialised views
- [ ] Search indices — Elasticsearch documents survive a primary delete quite happily
- [ ] Analytics warehouse (BigQuery, Snowflake, Redshift)
- [ ] Event streams and message queues, including dead-letter queues
- [ ] Application logs and log aggregation
- [ ] Object storage — exported CSVs, generated PDFs, data-science working copies
- [ ] Backups
- [ ] Every third-party processor: email, payments, error tracking, session recording, support desk, CRM, feature flags, marketing automation

**Failure mode without it:** the next datastore someone adds silently breaks compliance, and nobody notices for a year.

**Make it verifiable.** A wiki page decays. Generate it from infrastructure code, or add a test that fails when a new store appears without an inventory entry.

---

## 2. Design the backup strategy deliberately

Deleting one record from an immutable snapshot is generally not feasible, and that immutability is the point of backups. Pick an approach on purpose:

- [ ] **Retention-window expiry** — document the window (30/60/90 days), track pending deletions, and *reapply deletion after any restore within the window*. That reapplication step must actually exist, not be assumed.
- [ ] **Crypto-shredding** — per-user encryption key held outside backup scope; deleting the key renders backed-up ciphertext unrecoverable. Solves it properly. Must be designed in from the start.
- [ ] **Tokenisation** — personal data in a separate vault, referenced by token elsewhere. Delete the vault entry and backups elsewhere hold only meaningless identifiers. Also a design-time decision.

**Signal:** a team that raises backups in the first two minutes of this conversation has done it before.

---

## 3. Build deletion as an orchestrated workflow

Not a function. A durable workflow.

- [ ] Idempotent per-target deletion steps
- [ ] Retries with backoff — third-party APIs fail routinely
- [ ] Per-target status tracking, queryable
- [ ] Audit record of what was deleted, from where, and when
- [ ] Defined behaviour when a target permanently fails
- [ ] Explicit handling of retained-by-exception data (transaction records under other legal obligations), documented per category

Treat it with the seriousness of a payment flow. It has the same properties: distributed, partially failing, and consequential when wrong.

---

## 4. Enforce log hygiene at build time

- [ ] Structured logging with an explicit field allowlist
- [ ] CI check that fails on whole-request-body logging
- [ ] Defined log retention aligned to the inventory
- [ ] Redaction at the logging library layer, not at the call site

**Why it matters:** log aggregators are typically retained for months and fully indexed. Personal data logged during a debugging session is both durable and searchable. Cleaning a year of log history is enormously more expensive than never writing it.

---

## 5. Fix identifier correlation early

- [ ] A single canonical user identifier propagated to every downstream system
- [ ] Documented mapping where a third party forces a different key (email address, external customer id)
- [ ] The mapping itself treated as personal data and included in the inventory

**Failure mode:** the warehouse keys on one identifier, the primary database on another, and the support desk on email. Locating one individual across all three becomes a join nobody wants to maintain under a statutory deadline.

---

## 6. Implement access and portability

- [ ] Export covering every store in the inventory, not just the primary database
- [ ] Machine-readable portable format
- [ ] Identity verification before disclosure — this is itself a security-sensitive path
- [ ] Rate limiting and audit logging on the export endpoint

The access request is the deletion request's easier sibling: same discovery problem, no destructive step. If your export is incomplete, your deletion is too.

---

## 7. Handle opt-out signals

- [ ] Opt-out of sale/sharing of personal information, honoured downstream rather than only recorded
- [ ] Browser-level opt-out preference signals respected
- [ ] Sensitive personal information categories identified and handled separately
- [ ] Opt-out state propagated to advertising and analytics integrations, not just stored locally

**Common gap:** the preference is recorded correctly and never reaches the third-party tags that continue firing.

---

## 8. Write the test that proves it

The single highest-value test in this domain, and almost nobody has it:

```
1. Create a user
2. Exercise the system so data propagates to every store in the inventory
3. Issue a deletion request
4. Assert emptiness across every store in the inventory
5. Fail loudly if the inventory contains a store the test does not check
```

Step 5 is what keeps it honest as the system grows. Without it the test silently stops covering new stores and reports green forever.

---

## Using this in a vendor conversation

If you are evaluating an engineering partner, this list converts a vague question into a checkable one: *describe a data subject deletion you implemented end to end.*

Teams who have done it open with backups and third-party processors, because that is where the difficulty lives. Teams who have not describe a settings page and a database cascade.

It is one of the few compliance questions with a genuinely wide range of answers, which makes it unusually informative.

---

Full guide to evaluating California engineering partners — market segments, regional differences, rate bands, contract specifics under California law, and twelve-month cost ranges: **[Software Companies in California, USA](https://techcirkle.com/blog/software-companies-in-california-usa)**.

## Frequently Asked Questions

### Does CCPA require deletion from backups?

The obligation is to delete personal information; documented retention-window expiry has generally been accepted as reasonable for backups, provided deletion is reapplied after any restore within the window. Crypto-shredding and tokenisation address it more completely but are design-time choices.

### What is the difference between CCPA and CPRA?

CPRA amended and expanded CCPA — adding a sensitive personal information category, correction rights, and stricter obligations around sharing. For engineering purposes the requirements compose into one system design; you build for the stricter combined set.

### How do I keep the data inventory from going stale?

Generate it from infrastructure code where possible, and add a CI check that fails when a new datastore appears without an inventory entry. A manually maintained wiki page decays within about two quarters.

### Why does deletion need to be a workflow rather than a function?

Because it spans systems with different latencies and failure modes, and third-party APIs fail routinely. You need idempotency, retries, per-target status, and an audit record — the same properties a payment flow requires.

### What is the most commonly missed store?

Application logs, followed by the analytics warehouse. Both accumulate personal data quietly through ordinary operation, both are retained for months, and neither appears in most teams' mental model of "where user data lives."

### Do these obligations depend on where my engineers are?

No. They attach to serving California residents. When evaluating a partner, ask for a specific prior end-to-end implementation rather than treating a California address as evidence of capability.
