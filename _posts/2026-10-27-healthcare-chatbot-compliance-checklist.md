---
layout: post
title: "A Compliance Checklist for Healthcare Chatbot Builds (HIPAA, GDPR, Device Rules)"
date: 2026-10-27 00:00:00 +0000
categories: ["Compliance", "Healthcare", "Engineering Practices"]
tags: ["hipaa", "gdpr", "healthcare", "compliance", "checklist"]
description: "A working checklist for healthcare chatbot builds: HIPAA BAAs, GDPR special-category data, medical-device boundaries, AI transparency and audit."
image: "https://cdn.sanity.io/images/563mnkns/production/28d7f786c9cbd0250edc80b6477a638daf60e468-1600x900.jpg"
author: "TechCirkle"
---

![A doctor working alongside an AI medical assistant interface](https://cdn.sanity.io/images/563mnkns/production/28d7f786c9cbd0250edc80b6477a638daf60e468-1600x900.jpg)

Compliance on a healthcare chatbot is not a phase at the end. It determines which vendors you can use, where data may flow, what the model may see, and what the assistant is allowed to say. This page is a working checklist we use when scoping builds. It is not legal advice; it is a way to make sure the right questions get asked early.

Mark each item **Done**, **N/A (with reason)** or **Open**.

## 1. Scope and classification

| # | Item | Why it matters |
|---|---|---|
| 1.1 | Use cases listed and classified: administrative, navigation, care-journey, clinician-facing, payer | Risk and regulation differ sharply by category |
| 1.2 | Written boundary: no diagnosis, dosing or treatment advice from the assistant | Clinical outputs may make it a regulated medical device |
| 1.3 | Regulatory opinion obtained if any feature informs clinical decisions (FDA SaMD, EU MDR, UK MHRA) | Classification must happen before scoping, not after |
| 1.4 | Red-flag symptom list agreed with clinicians | Drives deterministic escalation rules |

## 2. United States — HIPAA

| # | Item | Why it matters |
|---|---|---|
| 2.1 | BAA signed with the model provider | The model sees PHI in prompts |
| 2.2 | BAAs with hosting, logging, analytics and contact-centre vendors | Every component in the PHI path counts |
| 2.3 | Minimum-necessary data passed to the model per task | Limits exposure and injection blast radius |
| 2.4 | Encryption in transit and at rest | Baseline safeguard |
| 2.5 | Access controls and audit logging for every PHI access | Required for accountability and investigations |
| 2.6 | Model provider zero-retention / no-training terms confirmed in contract | Sales decks are not contracts |

## 3. UK and EU — UK GDPR and GDPR

| # | Item | Why it matters |
|---|---|---|
| 3.1 | Lawful basis plus special-category condition documented | Health data is special-category data |
| 3.2 | Data-protection impact assessment completed | Expected for high-risk processing |
| 3.3 | International transfers mapped (model, hosting, support) | Cross-border processing needs a valid mechanism |
| 3.4 | Data-residency options selected where available | Often required by public-sector and regulated buyers |
| 3.5 | Retention schedule for transcripts and logs | Storage limitation principle |
| 3.6 | Process for access and erasure requests covering transcripts | Data-subject rights extend to chat history |

![A nurse showing a senior patient how to use a mobile health app](https://cdn.sanity.io/images/563mnkns/production/57a0361067628f2bd3b625234e5c71beb6c8a713-1600x1067.jpg)

## 4. AI transparency and oversight

| # | Item | Why it matters |
|---|---|---|
| 4.1 | Users told clearly they are talking to an automated assistant | Ethical baseline; expected under the EU AI Act |
| 4.2 | Visible, easy route to a human at all times | Human oversight and patient safety |
| 4.3 | Model, prompt and retrieval index versions documented | Reproducibility for incident review |
| 4.4 | Evaluation suite for safety behaviour, run on every change | Model upgrades can regress safety |

## 5. Identity, consent and proxy access

| # | Item | Why it matters |
|---|---|---|
| 5.1 | Authentication before any patient-specific data | Prevents disclosure to the wrong person |
| 5.2 | Delegated / proxy access rules (parents, carers) enforced per request | Carers are frequent users |
| 5.3 | Consent captured and logged per data use | Evidence for lawful processing |

## 6. Safety engineering

| # | Item | Why it matters |
|---|---|---|
| 6.1 | Pre-generation red-flag and scope routing, deterministic | Model must not be the only safeguard |
| 6.2 | Grounded answers from approved content with citations | Traceability of every answer |
| 6.3 | Post-generation clinical-advice filter | Catches drift |
| 6.4 | Emergency responses are clinician-reviewed templates | No generated emergency guidance |

## 7. Integration and audit

| # | Item | Why it matters |
|---|---|---|
| 7.1 | Model reaches EHR/scheduling only via a narrow integration service | Central authorisation and rules |
| 7.2 | PHI redacted before logs and analytics | Keeps PHI out of non-essential systems |
| 7.3 | Audit trail of routing, retrieval, tool calls and blocked outputs | Supports clinical governance review |
| 7.4 | Weekly sampled transcript review with clinical governance | Finds problems metrics miss |

## How to use this

Run the checklist at discovery, again before pilot launch, and again before scaling. Items that stay **Open** at pilot launch should be explicitly risk-accepted by a named owner.

The reasoning behind each item, plus architecture and cost guidance, is in [TechCirkle's full guide to building an AI chatbot for healthcare](https://techcirkle.com/blog/ai-chatbot-for-healthcare). If you need help designing the safety and integration layers, see our [AI development services](https://techcirkle.com/ai-development-services).

## Frequently Asked Questions

### Is any healthcare chatbot HIPAA compliant out of the box?
No. Compliance depends on how it is operated: BAAs with every PHI-handling vendor, minimum-necessary access, encryption, access controls and audit logging.

### Does GDPR allow AI chatbots to process health data?
Yes, with a lawful basis plus a special-category condition, a DPIA, appropriate transfer safeguards and respect for data-subject rights.

### When does a healthcare chatbot become a medical device?
When its output is intended to inform diagnosis or treatment decisions. Administrative assistants generally are not; symptom checkers often are. Seek a regulatory opinion early.

### Do chat transcripts count as personal health data?
Usually yes, when they relate to an identifiable person's health. They need retention rules, access controls and inclusion in access and erasure processes.

### How often should a healthcare chatbot be reviewed for compliance?
At discovery, before pilot, before scaling, and whenever the model, prompts, data flows or vendors change.
