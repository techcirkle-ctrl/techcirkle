---
layout: post
title: "A Database Selection Checklist"
date: 2026-09-13 00:00:00 +0000
categories: ["Architecture", "Databases"]
tags: ["databases", "architecture", "systemdesign", "postgres"]
description: "An ordered checklist for choosing a database: transactional guarantees, scan profile, AI retrieval, write throughput, and team familiarity as the tiebreak."
image: "https://cdn.sanity.io/images/563mnkns/production/b7f068bc866e28057d938ed47c81c4737afa92bd-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

![Database infrastructure](https://cdn.sanity.io/images/563mnkns/production/b7f068bc866e28057d938ed47c81c4737afa92bd-1600x1066.jpg)

Run these in order. Stop as soon as an answer constrains you — the earlier questions dominate the later ones.

## 1. Does any part of this system require multi-record transactional guarantees?

If yes, **that part is relational.** Not negotiable.

This is the question most often skipped, usually because the answer is assumed. Check it explicitly: is there any operation where two or more records must change together or not at all? Payments, inventory reservation, double-entry accounting, anything with a balance.

Note the phrasing — *that part*. A system can have a relational core and a document store for a genuinely different workload. What it cannot have is a transactional requirement served by a store without transactions.

## 2. Do aggregate scans over large datasets form a meaningful share of the workload?

If yes, **plan a separate columnar store from the start.**

Row-oriented and columnar storage are opposite physical layouts, and no index reconciles them. Running heavy aggregations against a transactional primary works at small scale and then degrades customer-facing latency, with the symptom appearing in a different part of the system from the cause.

Standard pattern: change data capture into a columnar engine, accepting seconds to minutes of lag. A read replica protects the primary but does not make the queries fast — it is row-oriented too.

## 3. Is there an AI retrieval path?

If yes, four sub-checks:

- [ ] **Latency specified at p99**, not median. Retrieval sits inside a chain containing model inference; tail latency compounds into perceived sluggishness. Teams measure medians in development and experience tails in production.
- [ ] **Filter selectivity measured.** Production retrieval is always filtered by tenant, date, or type. Engines differ enormously in pre-filtered approximate search, and published benchmarks measure the unfiltered case.
- [ ] **Result-count conformance asserted.** Post-filtering engines can silently return fewer than k results. Nothing errors; quality just degrades.
- [ ] **Reindex path designed.** Embedding models change and vectors from different models are not comparable. Versioned collections, model identifier stored per vector, switchable query layer.

Default: **start with pgvector** if already on Postgres. Vectors in the same transaction and permission model as your data is worth more than raw throughput below tens of millions of vectors.

![Engineer reviewing query performance](https://cdn.sanity.io/images/563mnkns/production/86f8b1b3864071555efa6c7ed5764d54f3b7677d-1600x966.jpg)

## 4. Does write throughput genuinely exceed single-primary capacity?

Two conditions, both required:

- [ ] Measured write rate exceeds what one node absorbs
- [ ] The workload shards cleanly on a natural partition key

If only the first holds, you are not ready for a distributed store. Adopting one without a clean partition key means paying the full operational cost for none of the scaling benefit, and discovering it after migration.

## 5. What has the team operated successfully before?

All else close, **familiarity wins.**

This is a legitimate technical criterion, not a compromise. At 2am the useful property of a database is how well your team — and the public record — understands its failure modes. A marginally better engine nobody has operated is worse than a well-understood one.

## Default outcome

If questions 1–4 leave you unconstrained, **use PostgreSQL** and revisit on production evidence. Most systems never revisit.

## Cross-cutting checks

Independent of engine:

- [ ] **Database-specific logic behind a repository boundary.** Migration cost is mostly application code that absorbed the engine's semantics — transaction boundaries, consistency assumptions, error handling. Concentrating that in one reviewable place does not make migration cheap, but it makes it survivable.
- [ ] **Consistency assumptions written down.** An undocumented assumption cannot be checked during a migration. Code written against a strongly consistent store breaks subtly and intermittently on an eventually consistent one — the worst failure profile, because it passes tests.
- [ ] **Data transfer modelled.** At mid scale, egress and cross-availability-zone traffic frequently exceed compute cost and are invisible until the invoice.
- [ ] **Serverless pricing modelled against actual traffic shape.** Excellent for spiky low-volume workloads; multiples of a provisioned instance under steady load.
- [ ] **Restore rehearsed, not just backups configured.** Many teams have backups. Far fewer have performed a restore under time pressure. This is the main thing a managed service is selling.

## Frequently Asked Questions

### Why run the checklist in order?

Because earlier constraints dominate. A transactional requirement settles the question regardless of anything below it, and answering later questions first wastes effort on options already eliminated.

### Can one database serve both transactional and analytical workloads?

At small scale, yes. As aggregate scans grow they degrade transactional latency because the two storage layouts are physically opposite. HTAP engines exist but are operationally heavier with less familiar failure modes; separation with replication remains the reliable choice.

### Why is pgvector the default for retrieval?

Because vectors live in the same transaction and permission model as your data — deletions are atomic, access filters are ordinary SQL predicates. A dedicated store adds a synchronisation path and a class of bug where restricted content stays retrievable.

### When is a distributed database actually justified?

Only when measured write throughput exceeds single-primary capacity *and* the workload shards cleanly on a natural key. Satisfying only the first means inheriting operational cost without scaling benefit.

### Why does team familiarity count as a technical criterion?

Because operational knowledge determines recovery time during incidents. A well-understood engine with documented failure modes outperforms a marginally superior one nobody has run in production.

---

Full reasoning behind each item: [Best Database Software in 2026: A CTO Selection Guide](https://techcirkle.com/blog/best-database-software-selection-guide)

[TechCirkle](https://techcirkle.com/development/saas-development) builds SaaS platforms and data-intensive systems.
