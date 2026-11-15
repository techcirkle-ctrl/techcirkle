---
layout: post
title: "Generative AI Procurement Checklist: Security, Privacy and Compliance"
date: 2026-11-15 00:00:00 +0000
categories: ["Security", "Artificial Intelligence"]
tags: ["generativeai", "security", "privacy", "compliance", "procurement"]
description: "A procurement checklist for generative AI tools covering training, residency, access, retention and indemnity, plus an acceptable-use policy outline."
image: "https://cdn.sanity.io/images/563mnkns/production/73773c246e0b3bd7607e7a9d814ab659ba9c5571-1600x959.jpg"
author: "TechCirkle"
---

This document is a working reference for anyone who has to approve a generative AI tool for use with company data. It covers three things: a pre-screen, a seven-item vendor checklist, and an outline for an acceptable-use policy. It applies to all three layers of generative AI a business buys: assistants licensed per seat, AI features embedded in existing software, and platforms and APIs billed by usage.

![A hand holding a tablet that projects a glowing AI head icon with gears and chat bubbles](https://cdn.sanity.io/images/563mnkns/production/73773c246e0b3bd7607e7a9d814ab659ba9c5571-1600x959.jpg)

## Scope and assumptions

- The checklist is vendor-neutral. It does not rank products.
- Each item should be confirmed in writing. A default setting in an admin console is not a commitment.
- The checklist addresses security, privacy and compliance. Accuracy, workflow fit and cost need their own evaluation.

## Stage 0: Pre-screen

Two questions decide whether a tool should reach a pilot at all.

1. **Data handling.** Is company data used to train models? Where is it stored, and for how long? Are enterprise terms, audit logs and single sign-on available?
2. **Access to context.** Can the tool reach the documents, tickets, code or records it needs using the same permissions the user already has?

A tool that fails either question should not proceed, however good the demonstration was. The first question alone removes most consumer-grade tools.

## Stage 1: Vendor checklist

### 1.1 Training on customer data

**Requirement:** Customer data is not used to train the vendor's models, by contract and not only by default setting.

**Why it matters:** Settings can be changed by the vendor or by an administrator. A contractual term cannot be changed without your agreement.

**Evidence to request:** The relevant clause in the terms or data processing agreement.

### 1.2 Data residency

**Requirement:** Residency options match your legal obligations, including GDPR for EU and UK data.

**Why it matters:** Where data is stored and processed determines which rules apply to it.

**Evidence to request:** A list of available regions and confirmation of which one applies to your plan.

### 1.3 Identity, access and audit

**Requirement:** Single sign-on, role-based access and audit logs are available on the plan you are buying.

**Why it matters:** These features are sometimes reserved for higher tiers. Confirm them against the specific plan on the order form, not the vendor's top tier.

**Evidence to request:** A plan comparison and a sample audit log export.

### 1.4 Certification and agreements

**Requirement:** The vendor holds current SOC 2 or ISO 27001 certification and will sign a data processing agreement.

**Why it matters:** Certification gives independent assurance about the vendor's controls. The agreement defines responsibilities for personal data.

**Evidence to request:** The current report or certificate and the signed agreement.

### 1.5 Retention and deletion

**Requirement:** Retention periods are configurable and deletion can be verified.

**Why it matters:** Prompts and uploaded files can contain sensitive material. You need to control how long they persist and to confirm removal.

**Evidence to request:** Documentation of retention settings and the deletion process.

### 1.6 Connector permissions

**Requirement:** Connectors respect source-system permissions instead of indexing everything under one service account.

**Why it matters:** This is the item most likely to cause an internal incident. If permissions are loose, an AI search tool will surface the salary spreadsheet to anyone who asks. Audit access in the source systems before rollout, since the tool will faithfully reflect whatever permissions it finds.

**Evidence to request:** A description of how each connector authenticates and how permission changes are synchronised.

### 1.7 Intellectual property and indemnity

**Requirement:** There is a clear position on intellectual property and indemnity for generated output.

**Why it matters:** Before generated text, images, audio or video are used commercially, you need to know the licence terms on that content and whether the vendor offers indemnity.

**Evidence to request:** The licence terms for outputs and any indemnity clause.

## Stage 2: Checks that depend on the type of tool

| Tool type | Additional check |
| --- | --- |
| Meeting assistants that record and transcribe | Consent rules for recording in every country you operate in |
| Enterprise search across connected systems | A permissions audit of the source systems before go-live |
| Image, video and voice generation | Output licence terms, vendor indemnity and your own policy on disclosing AI-generated media |
| Coding assistants | Written guidance on code review, security scanning and what must never be pasted into a prompt |
| Agents that act across systems | Permissions, human approval steps and logging for each action |

## Stage 3: Acceptable-use policy outline

Once a tool is approved, staff need a short policy. Keep it brief enough to be read. A workable outline:

1. **Purpose and scope.** Who the policy applies to and which kinds of tool it covers.
2. **Approved tools.** A named list, with the plan or tenant staff must use. Personal accounts are excluded.
3. **Prohibited data.** The categories of information that must never be entered into any AI tool.
4. **Human review.** When a person must review output before it reaches a customer.
5. **AI-generated media.** The company's position on disclosing generated images, audio and video.
6. **Requesting a new tool.** How a team asks for something not on the approved list.
7. **Ownership and review date.** Who maintains the policy and when it will next be reviewed.

Items 2, 3 and 4 are the core. The rest supports them.

## The risk this process is meant to reduce

Shadow use of personal AI accounts is the largest real risk in most companies. Staff paste company data into tools nobody has reviewed because the approved alternative is missing or worse.

A checklist cannot fix that alone. The cure is providing a sanctioned tool that is at least as good as the one people would otherwise use.

## When the checklist cannot be satisfied

Some workloads involve data that cannot leave your environment, or internal systems that no vendor connects to. No off-the-shelf product will pass the checklist for those cases. The alternative is to use a leading model through an API, often on the cloud platform your company already runs on, and build the retrieval, connections and guardrails around it. That work is described on our [LLM integration](https://techcirkle.com/llm-integration) page. For a broader view of what we build, see our [AI development services](https://techcirkle.com/ai-development-services).

## Further reading

This checklist is one section of a longer guide. For tool recommendations by category, evaluation criteria and a rollout plan, read the [best generative AI tools for business](https://techcirkle.com/blog/best-generative-ai-tools-for-business).

## Frequently Asked Questions

### Are business plans of generative AI tools safe for company data?

Business and enterprise plans from the major vendors generally do not train on your data and provide access controls and audit logs. Confirm the training, retention, residency and certification terms in the contract before use.

### Why is a default "no training" setting not sufficient?

A setting can be altered. A contractual commitment gives you something enforceable and auditable, which is why the checklist asks for it in writing.

### What should an acceptable-use policy for AI contain?

At its core, three things: which tools are approved, what data must never be entered, and when a human must review output before it reaches a customer.

### What is the most common cause of internal data exposure with AI search?

Loose permissions in the source systems. A search tool that respects permissions will still expose a file if that file was shared too widely in the first place.

### Do free or personal AI plans meet these requirements?

Often they do not. Free and personal plans frequently lack the contractual protections, access controls and audit logs that this checklist requires.
