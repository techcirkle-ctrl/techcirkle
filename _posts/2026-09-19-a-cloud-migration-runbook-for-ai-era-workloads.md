---
layout: post
title: "A Cloud Migration Runbook for AI-Era Workloads"
date: 2026-09-19 00:00:00 +0000
categories: ["Cloud", "DevOps", "Engineering"]
tags: ["cloud", "migration", "runbook", "devops"]
description: "A phase-by-phase migration runbook with entry and exit criteria, discovery commands, cutover gates and rollback triggers you can copy and adapt."
image: "https://cdn.sanity.io/images/563mnkns/production/58db2e52709897ca23f13d3295ec4bfca08626a4-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

![Data centre engineering](https://cdn.sanity.io/images/563mnkns/production/58db2e52709897ca23f13d3295ec4bfca08626a4-1600x1066.jpg)

A runbook, not an essay. Phases with entry and exit criteria, and the specific checks that catch failures while they are still cheap.

Adapt the thresholds; keep the gates.

---

## Phase 0 — Placement decisions

**Exit criteria (all must be true before Phase 1 begins):**

- [ ] Gravity wells identified — transactional store, document corpus, event stream — and their target provider + region recorded.
- [ ] Accelerator availability confirmed **in writing** for those regions, with lead times.
- [ ] Inference placement decided: co-located with data, or separated with the data path explicitly designed.
- [ ] Residency obligations mapped per dataset.

> **Why this is Phase 0:** compute is portable, petabytes are not. Everything reading a dataset becomes anchored to its location by egress pricing and latency. Deciding this after applications start moving is how one logical system ends up split across two providers.

---

## Phase 1 — Empirical discovery

**Do not use interviews as the primary source.** Collect observationally:

```bash
# Flow logs → weighted edge list (src, dst, bytes, conns)
# AWS: VPC Flow Logs · Azure: NSG Flow Logs · GCP: VPC Flow Logs
# Plus: DB query logs, LB access logs, eBPF tracing (pixie/cilium)
```

**Exit criteria:**

- [ ] Collection ran across a **full quarter-end cycle**. Non-negotiable — shorter windows miss periodic jobs, which are disproportionately what breaks cutovers.
- [ ] Edge list weighted by bytes *and* connection count.
- [ ] Every LLM-derived dependency claim cross-checked against observed telemetry.
- [ ] Zero-traffic systems flagged as retire candidates.

```python
# Cluster into candidate waves
import networkx as nx
from networkx.algorithms import community

G = nx.Graph()
for src, dst, bytes_, conns in edges:
    G.add_edge(src, dst, weight=0.7*norm(bytes_) + 0.3*norm(conns))

waves = community.louvain_communities(G, weight="weight", seed=42)
```

> **Trap:** LLMs are excellent at reading undocumented stored procedures and cron definitions, and confidently wrong often enough that one unverified summary puts a false assumption into the cutover plan. Treat output as leads, never conclusions.

---

## Phase 2 — Landing zone

**Exit criteria:**

- [ ] Encryption on by default.
- [ ] Public network exposure denied unless explicitly granted.
- [ ] Tagging enforced **at creation** — cost allocation works from day one or not at all.
- [ ] Centralised logging that a workload cannot opt out of.
- [ ] Policy-as-code, preventive not detective. Detective controls in a fast migration produce a backlog nobody clears.

**AI-specific guardrails — add now, not in a later pass:**

- [ ] Model endpoints restricted to approved providers and regions.
- [ ] Outbound network allowlists for anything running agent tooling.
- [ ] Data classification metadata travels with datasets into the new environment.

![Network systems](https://cdn.sanity.io/images/563mnkns/production/6331ee0b79285680f5ad0cd364f9761f2aea9bba-1600x1066.jpg)

---

## Phase 3 — Retire list

**Exit criteria:**

- [ ] Candidate list derived from measured traffic across a full business cycle, not attestation.
- [ ] Executive sign-off obtained **before** individual owners are consulted.
- [ ] Decommission dates scheduled with a defined restore window.

> Typically 10–20% of an estate qualifies. Highest return in the programme, most under-used, and the resistance is organisational rather than technical.

---

## Phase 4 — Wave execution

**Per-wave entry criteria:**

- [ ] Wave = dependency cluster, not a single application.
- [ ] Named accountable owner with real authority for every application in the wave.
- [ ] All organisational questions closed — ownership, outage tolerance, contract windows.
- [ ] Cutover pattern chosen and its scaffolding built and tested.

**Cutover patterns, in increasing order of effort:**

| Pattern | Reversibility | Use when |
|---|---|---|
| Strangler-fig routing | Per endpoint, immediate | Traffic splits cleanly by route |
| Dual-write + reconciliation | Full, until switchover | Consistency evidence required |
| Shadow traffic | Total (responses discarded) | Behavioural differences unknown |
| Big-bang | None in practice | Hours of downtime genuinely acceptable |

**Rollback triggers — agreed numerically, in advance:**

```yaml
rollback_if:
  error_rate_5xx:       "> 0.5% sustained 5m"
  p99_latency:          "> 800ms sustained 5m"
  reconciliation_drift: "> 0.01% of records"
decision_owner: <named individual>   # may call it without convening anyone
```

> Criteria negotiated during an incident never fire. Agree them while everyone is calm and rested.

---

## Phase 5 — Post-migration FinOps

**Exit criteria:**

- [ ] Token and inference consumption attributed **per feature and per customer segment** from first deployment.
- [ ] Unit economics per transaction reported.
- [ ] Egress monitored per environment pair while any hybrid state persists.

> AI costs are usage-driven, not capacity-driven. A feature that becomes popular becomes proportionally more expensive — nothing else in the estate behaves this way. Retrofitting attribution across a live system is painful, and the question "which feature is driving this bill" always arrives urgently.

**Optimisation levers, in rough order of value:**

1. Model selection — route routine requests to a smaller model.
2. Caching — aggressive, at the semantic level where possible.
3. Prompt size — context trimming.
4. Batching.
5. Instance rightsizing (last, and usually worth least).

---

## Programme-level anti-patterns

- **Migrating application by application.** Maximises metered cross-boundary traffic for the entire hybrid period, which reliably overruns.
- **Banking tooling gains as schedule.** Arrives at cutover faster with the same unresolved organisational risk.
- **Starting technical work before organisational questions close.** Concentrates every contested decision into the final third, at peak schedule pressure.
- **Deferring modernisation to a phase two** that is rarely funded once phase one is declared a success.

---

## Frequently Asked Questions

### Why must discovery run a full quarter-end cycle?

Observational discovery only sees what executes during the window. Quarter-end batch jobs are exactly the dependencies that surface at 3am during cutover.

### Can I trust LLM analysis of legacy code?

As leads to verify against telemetry, yes. As conclusions to plan against, no. Cross-check every claimed dependency.

### Why cluster waves instead of migrating per application?

Per-application waves separate tightly coupled systems across a metered boundary for the whole hybrid period. Clusters keep heavy traffic inside a wave.

### When is big-bang cutover acceptable?

Only where several hours of downtime is genuinely tolerable for that system, stated explicitly rather than assumed.

### What AI guardrails belong in the landing zone?

Approved model providers and regions, outbound allowlists for agent tooling, and classification metadata travelling with datasets — all cheap now, awkward to retrofit.

### What should be instrumented from the first deployment?

Token and inference attribution per feature and segment. Retrofitting it across a live system is painful, and the question always arrives urgently.

---

*Full article with rationale: [Cloud Migration Best Practices for AI-Era Workloads](https://techcirkle.com/blog/cloud-migration-best-practices-ai-workloads).*

*We help teams design landing zones and sequence waves — [cloud application development](https://techcirkle.com/blog/cloud-application-development-guide) and [custom software development](https://techcirkle.com/development/custom-software-development).*
