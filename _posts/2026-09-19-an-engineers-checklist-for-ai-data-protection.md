---
layout: post
title: "An Engineer's Checklist for AI Data Protection"
date: 2026-09-19 00:00:00 +0000
categories: ["AI", "Security", "Engineering"]
tags: ["ai", "security", "checklist", "rag"]
description: "A pre-launch checklist for LLM apps handling sensitive data: retrieval authorisation, classification, agent tools and audit logging."
image: "https://cdn.sanity.io/images/563mnkns/production/a733ea25403343f807b21a8cb95b400e2e6dc534-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

![Engineering team reviewing architecture](https://cdn.sanity.io/images/563mnkns/production/a733ea25403343f807b21a8cb95b400e2e6dc534-1600x1066.jpg)

A pre-launch checklist for LLM features that touch internal data. Each item corresponds to a failure that occurs in production, with a check that detects it before it does.

Use it as a review gate, not as reading material.

---

## Retrieval authorisation

- [ ] **Search is filtered by the requesting user's identity, not a service account.** The most common serious defect. If retrieval runs as a service account and permissions are applied afterwards, the model has already read unauthorised content and the answer contains it.
- [ ] **Filtering is pre-ranking, not post-ranking.** Confirm this in the engine's documentation — several vector databases accept a filter and apply it after retrieving top-k, silently returning fewer results rather than correctly-scoped ones.
- [ ] **Every chunk carries its source document's ACL as queryable metadata.**
- [ ] **Chunking respects document boundaries.** A chunk spanning a public intro and a restricted appendix carries two ACLs. Never intersect them into something permissive.
- [ ] **Principal sets are flattened and cached with a short TTL.** Deeply nested group membership produces filter clauses that some engines handle badly.
- [ ] **ACL changes propagate to the index.** Event-driven sync on permission-change events is the practical default. State your revocation latency target explicitly.

**Test that catches it:** create a user with no access to document X, ask a question only answerable from X, and confirm the answer does not contain X's content — not merely that the citation is hidden.

---

## Ingestion and classification

- [ ] **Classification runs before embedding.** Retroactive classification requires re-embedding the corpus; the cost grows with every week of delay.
- [ ] **Content above the sensitivity threshold routes to a separate index** with its own endpoint, logging, retention, and where required its own model region.
- [ ] **Document deletion removes chunks and vectors.** Deleting the source does not delete derived data by default. Wire deletion in as a first-class pipeline operation.
- [ ] **Deletion is tested, not assumed.** This is also what makes erasure requests answerable.
- [ ] **Classification accuracy is monitored as an ongoing metric,** not signed off once.

---

## Agent tools

- [ ] **Tool inventory exists and is reviewed.** An agent's effective permission is the union of its tools, regardless of what the system prompt says.
- [ ] **Tools are parameterised and purpose-specific.** `lookup_order_status(order_id)` is bounded; `run_sql(query)` is not.
- [ ] **Agents execute under the requesting user's identity,** so existing authorisation applies transitively.
- [ ] **Human approval gates anything with external effect** — sending messages, writing to external systems, moving money.
- [ ] **Outbound network destinations are allowlisted.** This bounds exfiltration when prompt injection succeeds, which it eventually will.
- [ ] **Tool calls are rate-limited per session.** A runaway extraction loop has a very different shape from normal use.

![Engineer monitoring systems](https://cdn.sanity.io/images/563mnkns/production/d02c4d466e097bf83668bbd2be33af4b5e1258e8-1600x1066.jpg)

---

## Prompt injection posture

- [ ] **Injection is modelled as privilege escalation, not a prompt quality issue.** No instruction reliably prevents untrusted text from being treated as instruction.
- [ ] **Retrieved content is structured as data with an explicit boundary,** never concatenated as instruction.
- [ ] **Agents that read external content do not hold write credentials in the same session.**
- [ ] **Output filtering strips credentials and secrets** on the way out.

**Design question to answer honestly:** if an injection succeeded right now, what is the worst reachable outcome? If the answer is unbounded, the containment work is not done.

---

## Audit logging

Product analytics — latency, token spend, thumbs-up rate — does not answer the questions an incident responder or regulator will ask. Log:

- [ ] **Requesting identity** for every interaction.
- [ ] **Documents retrieved, with classification labels.**
- [ ] **Every tool invoked, with parameters and result size.**
- [ ] **Model and serving region** that handled the request.
- [ ] **Retention applied** to the payload.
- [ ] **Records are immutable and queryable by data subject.**

Build this in the first release. Retrofitting is disproportionately hard, because the interesting state lives in memory between service calls and is gone by the time you need it.

---

## Data residency

- [ ] **Every surface is mapped to a jurisdiction:** object storage, vector index, cache, model endpoint, evaluation datasets, provider logs.
- [ ] **Provider retention is set explicitly, not left at default.** Enterprise agreements usually allow zero retention; the default frequently does not match your DPA.
- [ ] **The serving region is verified, not assumed** from the console setting.

The common error is assuming the model endpoint's advertised region covers the chain. It covers inference only.

---

## Training data

- [ ] **No personal or customer data in fine-tuning or evaluation sets.** Weights admit no deletion operation; erasure means retraining.
- [ ] **Evaluation datasets are inventoried.** A golden set built from production data is a permanent copy, usually replicated to every developer who cloned the repo.
- [ ] **Training corpora have a documented, durable legal basis.**

---

## Launch gate

Three questions. If any answer takes more than a few minutes, the feature is not ready for sensitive data.

1. Which regulated data does this process, and under what legal basis?
2. If a subject requests erasure today, what happens and how long does it take?
3. If this were compromised an hour ago, what could it have reached, and what would the logs show?

---

## Frequently Asked Questions

### What is the single highest-priority item here?

Query-time retrieval authorisation under the requesting user's identity. It closes the most probable breach path, and it is the first thing an auditor asks about.

### How do I verify pre-filtering versus post-filtering?

Check the engine documentation, then test empirically: query as a restricted user against a large corpus and confirm you receive a full top-k of authorised results rather than a truncated set.

### Is audit logging really needed before launch?

Yes. The state required to reconstruct an incident exists only in memory during the request. Once the feature is live without it, you cannot answer basic questions about your own system.

### Can we fine-tune on internal documents?

Only on material with a durable legal basis for indefinite use. Anything that may need deletion belongs in retrieval, which deletes cleanly.

### How often should ACL sync run?

Event-driven on permission changes is the practical default. Define an explicit revocation latency target rather than letting it be whatever the implementation happens to produce.

### What if our vector database cannot pre-filter efficiently?

That is a database selection problem, not a justification for filtering late. Late filtering is not a weaker version of the control — it is the absence of the control.

---

*Full article with architecture rationale and a ninety-day rollout sequence: [Enterprise Data Protection in the Age of AI Agents](https://techcirkle.com/blog/enterprise-data-protection-ai-agents).*

*We build this for teams shipping AI into regulated environments — [LLM integration](https://techcirkle.com/llm-integration) and [custom software development](https://techcirkle.com/development/custom-software-development).*
