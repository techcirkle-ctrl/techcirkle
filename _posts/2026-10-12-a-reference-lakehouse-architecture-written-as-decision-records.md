---
layout: post
title: "A Reference Lakehouse Architecture, Written as Decision Records"
date: 2026-10-12 00:00:00 +0000
categories: ["Architecture", "Data Engineering"]
tags: ["architecture", "lakehouse", "iceberg", "dataengineering", "adr"]
description: "A modern lakehouse explained as a set of architecture decision records: storage, table format, catalog, engines, orchestration and governance choices."
image: "https://cdn.sanity.io/images/563mnkns/production/30836f77a0a073adb6add53235251629009aff8d-1600x1067.jpg"
author: "TechCirkle Editorial Team"
---

![Engineer working on a laptop displaying big data visualisations](https://cdn.sanity.io/images/563mnkns/production/30836f77a0a073adb6add53235251629009aff8d-1600x1067.jpg)

Architecture diagrams show what was built. They rarely show why, which is the part a new engineer or a future you actually needs. So here is a reference lakehouse for a mid-sized software company, written as a set of short architecture decision records (ADRs). Adapt the decisions; keep the habit.

## ADR-001: Object storage as the system of record

**Context.** Analytical data is growing, several teams and AI workloads need it, and warehouse-managed storage is expensive per terabyte.

**Decision.** Store all analytical data as Parquet in cloud object storage, in a dedicated account with versioning and lifecycle rules.

**Consequences.** Storage becomes cheap and engine-independent. We take on responsibility for file layout and table maintenance.

## ADR-002: Apache Iceberg as the table format

**Context.** Raw Parquet lacks atomic commits, safe schema evolution and efficient deletes. We need several query engines to read the same tables.

**Decision.** Use Iceberg for all managed tables.

**Consequences.** Multi-engine portability, time travel and row-level deletes. We must schedule compaction, snapshot expiry and manifest rewrites (see ADR-007). Delta Lake would be equally reasonable in a Databricks-centred stack.

## ADR-003: One catalog as the governance control point

**Context.** Access rules implemented separately in each tool drift apart.

**Decision.** A single REST-compatible catalog owns table metadata, access policies, column masking and row filters.

**Consequences.** Engines must integrate with the catalog. In return, governance is defined once, and lineage and auditing have a natural home.

## ADR-004: Change data capture for operational databases

**Context.** Timestamp-based extracts miss hard deletes and load the source.

**Decision.** Use log-based CDC into an append-only bronze change table, merged into current-state silver tables every few minutes.

**Consequences.** Correct deletes, low source load, replayable history. We must monitor replication slot lag and retained WAL.

![Lakehouse data visualised on a laptop](https://cdn.sanity.io/images/563mnkns/production/5efbbeae27d81cc91d7465ec63696c92659239de-1600x900.jpg)

## ADR-005: Engines chosen per workload

**Context.** Heavy batch transformations and interactive queries have different cost and latency profiles.

**Decision.** A distributed engine for large batch jobs; a fast serverless or MPP engine for interactive and BI queries; a lightweight embedded engine for development and small jobs.

**Consequences.** Lower cost per workload. More engines to integrate with the catalog, so we cap the list at three.

## ADR-006: Asset-oriented orchestration and tested transformations

**Context.** Hand-written task graphs make backfills painful and dependencies opaque.

**Decision.** Declare tables as assets, version transformations in Git, require tests on every model, and develop against isolated environments.

**Consequences.** Idempotent reruns, simpler backfills and code review for data logic.

## ADR-007: Scheduled table maintenance

**Decision.** Daily compaction on hot tables, snapshot expiry aligned to our time-travel and privacy-deletion policies, weekly manifest rewrites, clustering on common filter columns.

**Consequences.** Stable query performance and storage cost; expired snapshots make privacy deletions real.

## ADR-008: Data contracts and observability

**Decision.** Contracts on money-critical source tables enforced in producer CI; monitoring for freshness, volume, schema and distribution, routed to named owners and tiered by business criticality.

**Consequences.** Breakages caught at pull-request time; fewer, higher-confidence alerts.

## ADR-009: A semantic layer for metrics, humans and agents

**Context.** BI tools and AI agents both need consistent definitions of business metrics.

**Decision.** Define metrics, dimensions and joins once in a semantic layer; agents query it rather than raw tables.

**Consequences.** Consistent numbers and enforced permissions across dashboards, APIs and agents.

## ADR-009b: Unstructured data pipelines for AI

**Context.** LLM applications need documents, tickets and transcripts, not just tables, and they must respect source-system permissions.

**Decision.** Run document ingestion, parsing, chunking and embedding on the same orchestrator, with embedding model versions recorded per vector and source ACLs carried into the index as metadata.

**Consequences.** One governance and observability model for structured and unstructured data. Re-embedding becomes a planned, evaluated migration rather than an ad-hoc job.

## ADR-010: Cost tagging from day one

**Decision.** Every job and query is tagged by team and purpose; budgets and alerts per domain; monthly review of top queries.

**Consequences.** Costs stay explainable as usage grows.

## When this is too much

For a company with a few hundred gigabytes and hourly freshness needs, most of these ADRs collapse into "use one managed warehouse." Write that ADR instead; it is a perfectly good decision.

The full write-up, including how AI changes these decisions and how to vet a delivery partner, is in our guide to [what a big data development company builds in 2026](https://techcirkle.com/blog/big-data-development-company). We design and build platforms like this through [custom software development](https://techcirkle.com/development/custom-software-development) engagements.

## Frequently Asked Questions

### Why write architecture as decision records?

ADRs capture context and consequences, so future engineers understand why choices were made and can revisit them deliberately when conditions change.

### Is a lakehouse always better than a warehouse?

No. Warehouses are simpler to operate at modest scale. Lakehouses pay off with growing volume, multiple engines, AI workloads or a need to avoid storage lock-in.

### Why limit the number of query engines?

Every engine must integrate with the catalog and governance model. More engines mean more surface for inconsistency and more to operate.

### Where do AI workloads fit in this architecture?

They read governed tables directly for training and features, use the semantic layer for metric questions, and add unstructured pipelines for documents and embeddings alongside the existing layers.
