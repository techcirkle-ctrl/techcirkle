---
layout: post
title: "Field Notes: A Checklist for Shipping ACH in 90 Days"
date: 2026-11-09 00:00:00 +0000
categories: ["Engineering Notes", "Fintech"]
tags: ["ach", "engineering", "checklist", "fintech", "payments"]
description: "Engineering notes for adding ACH to a product in 90 days: a phase-by-phase checklist covering role, ledger, returns, fraud controls and pilot exit."
image: "https://cdn.sanity.io/images/563mnkns/production/c0c10c82e55742289ef4f0c60b4ed30f821af940-1600x1064.jpg"
author: "TechCirkle"
---

These notes assume an existing product, a small engineering team and a mandate to get ACH live for real users inside a quarter. The sequence puts risk in front of polish. Tick items off in order, and treat any phase you cannot close as a reason to move the date.

![An executive holding a smartphone and completing an online financial transaction at a desk beside a tablet](https://cdn.sanity.io/images/563mnkns/production/c0c10c82e55742289ef4f0c60b4ed30f821af940-1600x1064.jpg)

## Ground rules

- The ledger is the source of truth. The network, the provider and the webhooks report into it.
- Submitted means accepted for processing and nothing more.
- A language model never gets a code path to money. Advisory output only.
- Every checkbox below needs an owner's name beside it.

## Phase 0: Days 1 to 15, settle the non-code questions

Nothing here is a ticket, and all of it can reshape the plan.

- [ ] Confirm your Nacha role with counsel and a prospective bank partner: Originator, Third-Party Service Provider or Third-Party Sender
- [ ] Write down each payment flow and who the money belongs to at each step
- [ ] Assign Standard Entry Class codes per flow (PPD, CCD, WEB and so on)
- [ ] Decide the funds availability policy: at submission, at settlement or after most of the return window
- [ ] State an acceptable loss figure and per-customer limits

**Note:** if users can spend against unsettled debits, you are extending credit. Get a named sign-off on that exposure.

## Phase 1: Days 16 to 40, build the core first

Pick the provider or bank path, then build underneath the eventual UI.

- [ ] Provider or bank selected, with the export of history, tokens and authorizations confirmed in writing
- [ ] Double-entry ledger, append-only, with balances derived from immutable rows
- [ ] App database role cannot update or delete ledger rows
- [ ] State machine with illegal transitions rejected and covered by tests
- [ ] Unique constraint on the instruction idempotency key
- [ ] Dedupe table for provider webhook event IDs
- [ ] Raw account numbers confined to one narrow service, tokens everywhere else
- [ ] Rail hidden behind an internal interface, so RTP or FedNow can be added later

## Phase 2: Days 41 to 65, integrate and break it

- [ ] Origination wired to the provider or the 94-character file generator
- [ ] Account validation checks ownership wherever possible, in addition to existence
- [ ] Authorization capture stores evidence by entry type, and is retrievable per entry
- [ ] Cut-off and federal holiday calendar logic in one module
- [ ] Same Day ACH windows and the $1 million per-payment limit enforced
- [ ] Return handling posts an offsetting journal and moves the state
- [ ] Reinitiation cap of two, after an initial NSF or uncollected-funds return, enforced in the domain layer
- [ ] Notifications of Change applied automatically within the required timeframe, with history
- [ ] Simulated return file exercised for every code you expect
- [ ] Duplicate and out-of-order events tested

**Test to run on a Friday:** an R01 lands four days after a payment was marked complete. Check balance, notifications and the next scheduled attempt.

Remember the long tail. Most returns come within two banking days of settlement, but consumer unauthorized returns can appear for up to 60 calendar days.

## Phase 3: Days 66 to 80, controls and visibility

Start with plain rules here. They are readable, and they produce the history a model would need later.

- [ ] Payee creation or edit cannot be followed by first-payment approval from the same user, enforced in code
- [ ] Cooling-off hold on first payments to new or changed accounts
- [ ] Out-of-band callback recorded, using the number already on file
- [ ] Rules-based monitoring on amount, velocity and new payees
- [ ] Daily reconciliation of ledger, provider and bank statement at trace level
- [ ] Every break routed to a named person
- [ ] Return rates dashboarded per customer against 0.5% unauthorized, 3% administrative and 15% overall
- [ ] Access to full account numbers logged
- [ ] Fraud response runbook: who calls the bank, how fast, with what evidence

**Compliance note:** Nacha's 2026 rules require covered parties to keep risk-based processes for identifying fraudulent entries, reviewed at least annually. Phase 2 had a practical date of June 22, 2026 and reached remaining non-consumer originators and third parties. WEB debit and Micro-Entry originators must use commercially reasonable fraud detection. This is a working summary and not legal advice, so confirm against the rule text and your ODFI.

## Phase 4: Days 81 to 90, limited pilot

- [ ] Low per-transaction and per-customer limits switched on
- [ ] Every exception reviewed by hand, with notes
- [ ] Risk-based fraud process written down and a review date booked
- [ ] Returns queue owner and an on-call path in place
- [ ] Exit criteria for raising limits agreed before the pilot starts

## After day 90

The backlog that should wait until you have clean labeled history:

- Anomaly scoring for payments and payee changes
- Language-model summaries for return triage and reconciliation suggestions
- Agents preparing payment runs, with no ability to release funds or edit payees

When you get there, the [LLM integration](https://techcirkle.com/llm-integration) layer needs to be built for regulated data. That matters more than which model you choose.

## Reality check on timing

As a planning estimate from delivery experience, a focused embedded integration with a proper ledger and exception handling commonly takes two to four months, so 90 days sits at the ambitious end and assumes a provider path. A direct bank build is a different project, commonly six to twelve months. Scope accordingly.

Background reading on the buy, embed or build decision is in our [complete guide to ACH software](https://techcirkle.com/blog/ach-software). If you want someone to review your plan, you can [reach the team here](https://techcirkle.com/contact-us).

## Frequently Asked Questions

### What should be built first when adding ACH to a product?

The ledger, the state machine and the idempotency layer. They determine whether every later feature can be trusted, and they are much harder to retrofit once real payments exist.

### Is 90 days realistic for shipping ACH?

It can be for a limited pilot on a provider path with narrow scope and low limits. Treat 90 days as ambitious, since a focused integration commonly takes two to four months as a planning estimate.

### Which tests matter most before an ACH pilot?

Simulated return files for each code you expect, duplicated and out-of-order events, and a return landing after a payment was marked complete. Together they exercise the paths that demos and basic integration tests skip.

### Why leave AI features until after the pilot?

Models need clean labeled history, and a pilot with hand-reviewed exceptions is how you get it. Rules are also easier to explain while your fraud process is new and still being documented.
