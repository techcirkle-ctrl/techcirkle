---
layout: post
title: "A Contract Checklist for Outsourced Engineering"
date: 2026-07-27 00:00:00 +0000
categories: ["Engineering", "Business"]
tags: ["contracts", "outsourcing", "engineering management", "intellectual property"]
description: "The clauses that decide whether you own your code, keep your team, and can leave. Written for the engineer reviewing the contract, not the lawyer."
author: "TechCirkle Editorial Team"
---

Legal review of a development contract focuses on liability, indemnity and payment terms. Those matter, and someone qualified should look at them.

But several clauses that determine whether the engagement succeeds are technical rather than legal, and lawyers routinely pass over them because they do not read as risk. If you are the engineer in the room, these are yours.

## 1. IP assignment on invoice payment, not project completion

The single most important clause, and the most commonly wrong.

Many standard agreements assign intellectual property on completion of the project. That sounds reasonable until you consider the case where it matters: the engagement ends early. Disputes, budget cuts, a vendor going under, a relationship breaking down in month five.

Under that clause, you have paid for five months of work and own none of it.

**Ask for:** assignment on payment of each invoice. Each tranche of work becomes yours as you pay for it. Vendors accept this far more readily than you would expect, because it is a reasonable position and refusing it is conspicuous.

## 2. Code in your repository from day one

Not delivered at milestones. Not handed over at the end. In your GitHub or GitLab organisation, continuously, with your team holding admin access.

This has three effects, all of them useful:

- You can read the code as it is written rather than after it is finished.
- Your CI runs on your infrastructure, so you see build health directly.
- If the relationship ends abruptly, you already have everything.

A vendor who resists this is telling you something about how they work, and it is worth asking directly what the objection is.

## 3. Named key personnel with substitution rights

The most common failure mode in this industry: the senior architect who presents at the pitch and is never seen again.

**Ask for:** named individuals in an appendix, with allocation percentages, and a clause requiring your written consent for substitutions above a stated threshold. You are not trying to prevent all change — people leave jobs. You are trying to ensure that a swap is a conversation rather than a surprise.

## 4. An explicit AI clause

This belongs in every 2026 development agreement and is still missing from most.

Three things to specify:

- **Does the vendor use AI coding assistants?** Almost certainly yes. You want it stated rather than assumed.
- **Is your code or data transmitted to third-party model providers, and which ones?** These are sub-processors and belong in your DPA, your privacy notice, and your security review.
- **Are those providers contractually barred from training on your material?** Enterprise tiers generally provide this. Consumer tiers frequently do not.

Worth adding: a statement that AI-generated code is reviewed to the same standard as human-written code. It is unenforceable in practice, but asking for it surfaces whether the vendor has thought about the problem, and the conversation is more useful than the clause.

## 5. A defined exit path

Negotiate the divorce at the wedding, because handover terms agreed during a dispute are handover terms you will not get.

**Specify now:**

- Notice period — thirty days is standard and sufficient.
- The handover package, itemised: documentation, runbooks, architecture decision records, credentials, infrastructure access, deployment procedures.
- A day rate for transition support after termination, agreed in advance.
- A defined window during which the vendor will answer questions.

The itemised package is the part people skip. "Reasonable handover assistance" means nothing when you need it.

## 6. Subcontracting disclosure

Many firms subcontract portions of delivery, often offshore. This can be entirely fine and sometimes produces excellent results.

What is not fine is discovering it in month four, after being sold a local team.

**Ask for:** disclosure of any subcontracting, identification of the entities involved, and a requirement for written consent before new subcontractors are added. If the vendor's model is fundamentally offshore-delivered with a local sales presence, you want that on the table during procurement, where you can price it.

## 7. Warranty period

Sixty to ninety days post-launch during which defects are fixed at no charge, with a clear definition of what constitutes a defect versus a change request.

That definition is where the arguments happen. The workable line is roughly: a defect is behaviour that does not match the agreed specification; a change is a specification that turned out to be wrong. Write it down, because you will be relitigating it in month seven otherwise.

## 8. Data processing terms that match your actual architecture

If personal data is involved, the DPA needs to reflect what the system really does — not a template.

- Where does data physically live, and in which regions?
- Which sub-processors touch it, AI providers included?
- What are the breach notification timelines, and are they compatible with your own obligations?
- Who is responsible for satisfying erasure and portability requests, and through what mechanism?

Generic DPAs are common and mostly harmless right up until a regulator or an enterprise customer's security team reads one carefully.

## 9. Acceptance criteria that are testable

"The software shall perform as described in the specification" is not an acceptance criterion, it is an invitation to a dispute.

**Ask for:** specific, measurable criteria per milestone. Response times under stated load. Defined browser and device support. A defect threshold by severity. Something a person can execute and get a binary answer from.

Vague acceptance criteria always resolve in favour of whoever has more leverage at the moment of the argument, and that is rarely you.

## Quick checklist

- [ ] IP assigns on payment of each invoice
- [ ] Code in your repository from day one, your team holds admin
- [ ] Named personnel with substitution consent
- [ ] AI usage, providers and training restrictions stated
- [ ] Itemised handover package and post-termination day rate
- [ ] Subcontracting disclosed and consent required
- [ ] 60–90 day warranty with a defect/change definition
- [ ] DPA matching actual architecture and sub-processors
- [ ] Testable acceptance criteria per milestone

---

The full vendor evaluation guide — archetypes, 2026 rate ranges, regional differences and a ten-point checklist — is at <https://techcirkle.com/blog/software-development-companies-in-canada>.

## Frequently Asked Questions

### When should intellectual property transfer in a development contract?

On payment of each invoice, not on project completion. Assignment at completion leaves you owning nothing if the engagement ends early — which is precisely the scenario where ownership matters. Vendors generally accept invoice-based assignment because it is reasonable and refusing it is conspicuous.

### Why insist on code in my repository from the start?

Three reasons: you can read the work as it happens rather than after it is finished, your CI runs on your infrastructure so build health is directly visible, and an abrupt end to the relationship leaves you already holding everything. Resistance to this arrangement is itself informative.

### What should an AI clause in a development contract say?

That the vendor discloses whether it uses AI coding assistants, identifies any third-party model providers receiving your code or data, and confirms those providers are barred from training on your material. Adding a statement that generated code is reviewed to the same standard as human-written code is worth doing mainly for the conversation it prompts.

### How do I define the difference between a defect and a change request?

A workable line: a defect is behaviour that does not match the agreed specification, and a change is a specification that turned out to be wrong. Write the definition into the contract, because this is the single most relitigated point in any warranty period.

### What belongs in a handover package?

Documentation, runbooks, architecture decision records, credentials, infrastructure access and deployment procedures — itemised explicitly, not described as "reasonable assistance." Agree a post-termination day rate for questions at the same time. Handover negotiated during a dispute is handover you will not receive.
