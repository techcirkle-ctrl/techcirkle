---
layout: post
title: "AI Compliance Readiness: An Engineering Checklist"
date: 2026-08-15 00:00:00 +0000
categories: ["Engineering", "Governance", "Artificial Intelligence"]
tags: ["checklist", "compliance", "ai", "engineering"]
description: "A concrete engineering checklist for AI regulatory readiness — inventory, disclosure, provenance, logging, evaluation evidence and tenant isolation."
image: "https://cdn.sanity.io/images/563mnkns/production/9b86a3466f7723f025b047f6c41e2bbe13d47f14-1600x1218.jpg"
author: "TechCirkle Editorial Team"
---

![AI systems under regulatory scrutiny — most of the work is engineering](https://cdn.sanity.io/images/563mnkns/production/9b86a3466f7723f025b047f6c41e2bbe13d47f14-1600x1218.jpg)

A checklist for engineering teams, not legal teams. Each item includes the test that reveals whether the control actually works, because most of these fail in ways that policy review cannot detect.

Assumes you ship AI features and are not operating a high-risk system. If you are — hiring, credit, biometrics, critical infrastructure — this is a subset of a much larger programme.

*We build software, not legal advice. Classification decisions belong with counsel.*

---

## 1. Know what you run

- [ ] A model registry exists: logical name, provider, **exact pinned version**, purpose, owning team, permitted data categories, deployment and retirement dates
- [ ] Fine-tuning lineage recorded, so "everything derived from base model X" is one query
- [ ] Models soft-retired, never deleted — questions arrive about systems no longer running
- [ ] Registry populated by **discovery**, not by asking teams: egress logs to provider domains, billing records, codebase scan for SDK imports

**Test:** ask three teams what models they run, then check the egress logs. The delta is your shadow AI, and there is almost always a delta.

---

## 2. Never resolve model versions from mutable config

- [ ] Model identifiers pinned explicitly, not read from an environment variable updated in place
- [ ] The resolved version recorded with every inference call, not just at deploy time
- [ ] Version changes are code changes, reviewable in history

**Test:** "which model version produced this output on 14 March?" If it takes more than five minutes, you have the most common and most expensive gap on this list.

---

## 3. Log inference in structured form

- [ ] Inference ID, timestamp, registry reference, exact model identifier
- [ ] Caller service **and tenant**
- [ ] Input and output stored by reference, with token counts in the record
- [ ] **Retrieved context with scores** — without this, RAG failures cannot be diagnosed
- [ ] Decision taken and confidence
- [ ] Human review status and outcome
- [ ] **Join key to the downstream business outcome**
- [ ] Logging is asynchronous and never on the response path
- [ ] Retention policy differs for metadata and payloads

**Test:** pick a customer complaint from last quarter and reconstruct exactly what the system saw and did. If you cannot, the log is an artefact rather than an instrument.

---

## 4. Disclosure at every AI surface

- [ ] Every interface where a user encounters model output discloses it at the point of interaction, not in terms of service
- [ ] Mixed human and AI content distinguished **in the data model**, not inferred after the fact
- [ ] Third-party AI output transiting your product carries the same disclosure

**Test:** list every surface where a user sees model output. Most teams miss at least one — commonly an email template, a notification, or an internally-facing tool that turned out to be customer-visible.

---

## 5. Provenance that survives your own pipeline

This is the item most likely to be silently broken right now.

- [ ] Content credentials applied at generation
- [ ] Every transform stage — resize, transcode, format conversion, CDN optimisation — audited for metadata preservation
- [ ] Transforms you control **re-sign** rather than merely carrying bytes across, since altering pixels invalidates the original signature
- [ ] Verification at the **egress boundary**, not at generation
- [ ] A CI test asserting credentials survive the full pipeline end to end

```bash
c2patool generated-original.jpg           # source — expect a manifest
curl -sL "https://cdn.example.com/x.jpg" -o delivered.jpg
c2patool delivered.jpg                    # delivered — expect the same
```

**Test:** the two commands above. Manifest present at source, absent after delivery, is the common result — not the exception.

---

## 6. Evaluation evidence, retained

- [ ] Automated evaluation suite run on every model or prompt change
- [ ] Results **persisted and versioned** against the registry entry, not reviewed and discarded
- [ ] Baseline comparison, so regressions are detectable rather than debatable

**Test:** produce evidence that the model version currently in production was evaluated before release, with the results. Most teams have run the evaluation and kept nothing.

---

## 7. Tenant isolation on retrieval

- [ ] Tenant filter applied **within** the vector query, not as a post-filter on results
- [ ] Isolation covered by an automated test with realistic tenant counts

```python
# WRONG — retrieves across tenants, filters after
results = index.query(embedding, top_k=10)
results = [r for r in results if r.tenant_id == tenant]

# RIGHT — filter is part of the query
results = index.query(embedding, top_k=10, filter={"tenant_id": tenant})
```

**Test:** the wrong version passes in staging with three tenants and leaks in production with three hundred. Test with realistic cardinality or the test proves nothing.

This is a security bug with a compliance consequence rather than the reverse — and it ends customer relationships independently of any regulator.

---

## 8. Human oversight that is not theatre

- [ ] Reviewers see the **basis** for a recommendation, not just its conclusion
- [ ] Override is as low-friction as approval
- [ ] Review workload bounded so genuine review is possible
- [ ] **Override rate monitored** as a metric

**Test:** check your reviewers' override rate. If it is at or near zero, the control has failed — that is not evidence the model is excellent.

---

## 9. Data lineage

- [ ] Recorded: what trained or fine-tuned each model, what is in each retrieval corpus, under what rights
- [ ] Deletion requests demonstrably honourable in fine-tuned models and vector indices
- [ ] Customer data flows into training or retrieval documented with a lawful basis

**Test:** trace one deletion request end to end through every index and fine-tuned artefact.

---

![Engineering teams reviewing production AI systems](https://cdn.sanity.io/images/563mnkns/production/944cc1604567b780927e2868c631e1f9891c0198-1600x900.jpg)

## Effort and sequencing

| Item | Effort | Notes |
|---|---|---|
| Registry | 1–2 weeks | Longer if discovery is needed, which it usually is |
| Inference logging | 3–4 weeks | Includes storage, retention, query interface |
| Disclosure audit | 1 week | Mostly finding surfaces you forgot |
| Provenance pipeline | 4–8 weeks | Scales with number of transform stages |
| Evaluation retention | 3–6 weeks | Improves shipping velocity independently |
| Tenant isolation fix | days | If it is broken, do it first |

Retrofitting later runs three to five times higher, based on our project history — you are locating call sites across services written by people who have left, and backfilling nothing because the data was never captured.

## Why the timing

The EU deferred high-risk obligations to 2 December 2027 (Annex III) and 2 August 2028 (Annex I). Not deferred: prohibitions (February 2025), GPAI obligations and penalties (August 2025), the remainder of the Act (August 2026), and machine-readable content marking (**2 December 2026**).

If your product is not high-risk, none of the deferral applied to you. The December date is the nearest real deadline for most teams shipping AI features.

Full context: **[AI Regulations in 2026: What Product Teams Must Build Now](https://techcirkle.com/blog/ai-regulations)**.

## Frequently Asked Questions

**Is this over-engineering for a small team?**
Items 1, 2, 3 and 7 are justified for any team running models in production, on operational grounds alone. The rest scale with risk tier.

**Do our existing MLOps tools cover this?**
Partially — most centre on training rather than inference-time governance. The usual gaps are retrieved-context capture and outcome joining. Audit against those two specifically.

**How do we classify risk tier?**
You do not, definitively. Engineering supplies the inventory and data-flow facts; counsel supplies the classification. Record the legal determination, not an engineering guess.

**What if models are called from a dozen services?**
Introduce a thin shared client wrapping the provider SDKs and migrate incrementally. Cheaper than instrumenting each call site, and it gives you one place to enforce future policy.

**Does logging conflict with data minimisation?**
Not if payloads are referenced and retained separately from metadata. "Model X was called at time T" and the prompt content deserve different retention policies.

**Where should provenance be verified?**
At egress. Verifying at generation confirms the step that was already working and misses the transforms that actually break it.

---

*TechCirkle builds AI systems with governance and observability from the first sprint. [AI development services](https://techcirkle.com/ai-development-services) · [Contact us](https://techcirkle.com/contact-us)*
