---
layout: post
title: "A Scoring Rubric for Software Agency Selection"
date: 2026-10-26 00:00:00 +0000
categories: ["Engineering Practices", "Procurement"]
tags: ["engineering", "procurement", "rubric", "vendor-selection"]
description: "An eight-criterion scoring rubric for evaluating software development agencies, with the evidence required to score each line honestly."
image: "https://cdn.sanity.io/images/563mnkns/production/5fec60516f6fb2174bb7538e534637980a0d7c99-1600x1034.jpg"
author: "TechCirkle"
---

![IT specialists working in a modern office](https://cdn.sanity.io/images/563mnkns/production/5fec60516f6fb2174bb7538e534637980a0d7c99-1600x1034.jpg)

Most agency evaluations are a pitch with a question period attached. This is the rubric I use instead. It is designed so that each line requires **evidence** rather than assertion — if a criterion can be satisfied by a confident answer alone, it is not on the list.

Score each candidate 0–5 per criterion. Weight to taste; suggested weights are given.

---

## The rubric

| # | Criterion | Weight | Evidence required |
|---|---|---|---|
| 1 | Domain evidence | ×3 | Architecture of a structurally similar system they shipped — not a logo |
| 2 | Named senior ownership | ×3 | A named individual, % of their time allocated, contract clause |
| 3 | Engineering hygiene | ×3 | Read access to a real pipeline and repository |
| 4 | Written artefact quality | ×2 | A real ticket, a real ADR, a real incident write-up |
| 5 | AI practice maturity | ×2 | A written generation and review policy |
| 6 | Timezone overlap | ×2 | Working hours of the specific decision-makers |
| 7 | Exit terms | ×2 | Draft contract language, not verbal assurance |
| 8 | Pricing logic | ×1 | An explanation that survives one follow-up question |

---

## Scoring guidance per criterion

### 1. Domain evidence (×3)

- **0** — No comparable system. Case studies describe scale, never a problem.
- **3** — Adjacent domain, transferable constraints, can describe what surprised them.
- **5** — Shipped two or more structurally similar systems; can name the architectural decision they would make differently now.

The strongest single predictor of first-release quality. Ask for the architecture, not the client list.

### 2. Named senior ownership (×3)

- **0** — "Our team" with no named individual, or an owner absent after the sales process.
- **3** — Named technical owner, time allocation stated, no contractual backing.
- **5** — Named in the contract, notice required before replacement, your approval over the replacement.

As implementation automates, more engagement value concentrates in a few senior judgements. Personnel substitution is a material change in what you bought, not an administrative detail.

### 3. Engineering hygiene (×3)

Score from a two-hour audit of a real repository and pipeline:

- Default branch red less than 5% of the last 90 days.
- Median PR review latency in hours, not days.
- Rubber-stamp ratio (approvals with zero comments) under ~30%.
- Tests assert behaviour; at least one fails if a core business rule is inverted.
- A real rollback that actually happened, not documentation of one.
- Infrastructure as code; no hand-configured environments.
- No credentials anywhere in history.

A refusal to provide any read access at all — including on a redacted or internal project — scores 0, not "unknown".

### 4. Written artefact quality (×2)

- **Ticket** — acceptance criteria testable? Is the *why* present, or only the *what*?
- **ADR** — alternatives considered and tradeoff accepted stated explicitly? An ADR with no rejected options is marketing.
- **Incident write-up** — names a cause without naming a scapegoat?

This criterion has become disproportionately important. Specifications are now inputs to code generation rather than documentation of it, so vague prose produces fast, confident, wrong implementations — frequently with tests asserting the wrong behaviour.

### 5. AI practice maturity (×2)

- **0** — "We don't use it" or enthusiasm with no policy.
- **3** — Informal norms, generated changes not identifiable.
- **5** — Written policy covering: what may be generated; explicit exclusions (crypto, authorisation, billing calculations, production migrations); who reviews; whether AI-authored changes are labelled; and the rule for merging code no reviewer fully understands.

The last item is the discriminator. Acceptable answers are a size threshold triggering a second reviewer, a mandatory walkthrough, or a refusal. A shrug is not an answer.

### 6. Timezone overlap (×2)

Measure between **your decision-makers and their technical lead** — overlap with their juniors does not count.

- **0** — Zero overlap.
- **3** — 2–3 hours, with demonstrated async discipline (written proposals carrying a decision deadline and a default action).
- **5** — 4+ hours, or 2–3 hours plus a documented async decision process you have seen in practice.

Ask the diagnostic question: *when a developer hits an ambiguity mid-task, what is the standing instruction?* "They ask the client" means you are the bottleneck.

### 7. Exit terms (×2)

- Code in your repository organisation, runtime in your cloud accounts, domains under your registrar — from the first commit.
- Documented handover as a deliverable.
- Initial term of 30–60 days scoped to a verifiable deliverable, then a decision point.

Score 5 only on draft contract language. Verbal assurance scores 2.

### 8. Pricing logic (×1)

- **0** — Headcount × hours × regional discount, with no answer to "what changes if your tooling improves?"
- **5** — Priced against verifiable increments, with a coherent account of where senior judgement sits.

Low weight because it is highly correlated with criteria 1–3 in practice; a firm scoring well there rarely prices incoherently.

---

## What this rubric deliberately omits

**Location.** Nothing above correlates with distance. Proximity genuinely matters in exactly four situations — software touching physical equipment, discovery dependent on stakeholders who will not join a call, facilities where access is physically gated for regulatory reasons, and procurement rules prohibiting a foreign counterparty. Where one of those applies it is a hard constraint, not a scored criterion, and the usual right structure is a local prime holding the physical dependencies with specialist depth subcontracted beneath it.

**Company size.** Small firms have thin benches; large firms rotate staff and bury you in process. Score bench depth under criterion 2 directly rather than inferring it.

**Certifications.** Weakly correlated with delivery quality and strongly correlated with having a compliance budget.

---

## Running it

Score the shortlist, then run a **paid parallel pilot** on the top two: three to four weeks, the same real slice of your product, identical brief and stakeholders, output scoped to be useful either way.

The pilot is where the rubric gets validated. A candidate who scored 5 on written artefacts and 2 on senior ownership will show you exactly which number was load-bearing.

---

Full buyer-side discussion: [Software Development Agency Near Me: How to Choose One in 2026](https://techcirkle.com/blog/software-development-agency-near-me). How we structure engagements against these criteria: [custom software development](https://techcirkle.com/development/custom-software-development).

## Frequently Asked Questions

### How should the rubric weights be adjusted for a small project?

For anything under roughly three months, raise domain evidence and named senior ownership and lower exit terms — short engagements rarely reach a painful migration. For multi-year platform work, invert that: exit terms and engineering hygiene dominate because you will live with the consequences.

### What if a candidate refuses repository access?

Score engineering hygiene 0 rather than leaving it unknown. Offer alternatives first — a redacted project, an internal one, a live walkthrough — because the substance of those artefacts is not confidential, only their detail. A flat refusal with no alternative is itself the finding.

### Is "we don't use AI tools" a safe answer from a vendor?

No, it scores 0 on AI practice maturity. It means either the claim is untrue or the firm is accepting a significant speed and cost disadvantage. What earns a high score is a written policy with explicit exclusions and a stated rule for merging code no reviewer fully understands.

### Why is timezone overlap weighted lower than engineering hygiene?

Because overlap is partly fixable through asynchronous discipline, whereas weak engineering hygiene is cultural and will not change during your engagement. A two-hour-overlap team with written proposals and documented assumptions routinely outperforms a four-hour team without them.

### Should certifications such as ISO or CMMI factor into the score?

Not materially. They correlate with having a compliance budget more than with delivery quality. Where a certification is a procurement requirement, treat it as a gate rather than a scored criterion.

### How many candidates should this rubric be run against?

Four to six for scoring, then a paid parallel pilot on the top two. Fewer than four and you have not tested the market; more than six and the scoring effort exceeds the value of the additional signal.
