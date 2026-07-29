---
layout: post
title: "A Pre-Launch Compliance Checklist for UAE Applications"
date: 2026-07-29 00:00:00 +0000
categories: ["Engineering", "Compliance"]
tags: ["checklist", "compliance", "engineering", "architecture"]
description: "A working checklist for engineering teams shipping into the UAE: residency, classification, subprocessors, RTL, identity, and deletion cascades."
image: "https://cdn.sanity.io/images/563mnkns/production/dbefb81f4535bc21ee770b450519125a582dce99-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

![Dubai downtown skyline meeting the desert, representing the UAE technology market](https://cdn.sanity.io/images/563mnkns/production/dbefb81f4535bc21ee770b450519125a582dce99-1600x1066.jpg)

This is a working checklist rather than an essay. It is organised by *when* each item is cheap, because that is the property that matters — several of these cost almost nothing in week one and require a live migration or a re-architecture in month eight.

Nothing here is legal advice. It is the engineering-side translation of constraints that teams shipping into the UAE repeatedly hit, written so you can walk it with your own counsel.

---

## Phase 0 — Before infrastructure exists

These are the irreversible ones. Everything in this section is a twenty-minute conversation now and a re-architecture later.

- [ ] **Identify which data protection regimes bind you.** Federal law is one layer. DIFC and ADGM operate separate regimes with their own commissioners. Health data under DHA/DoH oversight and financial data under Central Bank supervision carry stricter expectations again. Write down which apply and why.
- [ ] **Confirm your entity structure and its consequences.** Mainland versus free zone determines permissible data locations, payment gateway eligibility, national digital identity access, and tax invoicing logic. This is an engineering input, not just a corporate one.
- [ ] **Choose the cloud region deliberately, and record the reasoning.** Both major hyperscalers operate UAE regions at a premium over European ones. Budget the delta. Verify that every managed service your architecture assumes is actually available in the chosen region — newer regions lag, and discovering a missing service after design is a genuine constraint.
- [ ] **Decide the residency model.** Single strict region (simplest, usually correct), vertical partition by data class (only where a natural domain seam exists), or tokenized reference (where analytics on non-restricted attributes is central).

## Phase 1 — Initial schema and scaffolding

- [ ] **Define a data classification taxonomy** and attach it at table and column level in a version-controlled registry that the build reads. Not a wiki page.
- [ ] **Gate migrations on classification.** New columns on classified tables must declare a class or fail CI. This prevents the most common drift: a personal field added to an internal table two years later.
- [ ] **Model names correctly.** One required authoritative `full_name`; `given_name` and `family_name` optional forever. Arabic naming does not decompose reliably, and a `NOT NULL` surname corrupts real user data from week one.
- [ ] **Model addresses correctly.** Emirate as a controlled enumeration, area and building name as primary identifiers, `landmark` as a first-class field, coordinates stored explicitly, `postal_code` optional. A required postcode fills with `00000` within days.
- [ ] **Add the normalized search column now.** Apply NFKC, strip diacritics and tatweel, unify alef variants, map ta marbuta to ha and alef maqsura to ya. Store indexed, written at insert time. Normalize queries with the identical shared function. Adding this later means a backfill on a hot table plus a rewrite of every query touching it.
- [ ] **Choose a locale-aware collation** for sorting. Byte order produces lists that are arbitrary to Arabic readers.
- [ ] **Model translations in a keyed table** — entity, locale, field — with explicit fallback behaviour and independent workflow state per locale. Parallel `_ar`/`_en` columns do not survive a third language.

## Phase 2 — Interface

- [ ] **Use CSS logical properties from the first component.** `margin-inline-start`, `padding-inline-end`, `inset-inline`. Retrofitting physical properties across an accumulated component library is a full-surface sweep.
- [ ] **Audit directional assets.** Back arrows, chevrons, progress indicators, and send icons mirror. Logos, media controls, and clock icons do not. There is no automatic rule — this needs a per-asset decision.
- [ ] **Isolate bidirectional runs.** Arabic text containing Latin brand names or numerals needs `<bdi>` or isolate marks (U+2068/U+2069), or punctuation lands in visually wrong positions.
- [ ] **Test text expansion.** Arabic strings differ in length from English. Fixed-width components sized against English copy are the first casualty.
- [ ] **Verify third-party components.** Date pickers, charts, rich text editors, and map controls each have their own RTL story, ranging from excellent to nonexistent.
- [ ] **Check directional animations and gestures.** Slide-ins, swipe-to-dismiss, and carousels carry left-to-right assumptions.

## Phase 3 — Integrations

- [ ] **Enumerate every external system by name**, with an owner and a dependency date. Integration risk is schedule risk, and an unenumerated integration list is the strongest predictor of slip.
- [ ] **Start payment gateway onboarding early.** Trade licence, bank account, and underwriting run on a timeline independent of your sprint plan, and frequently longer than the development work they block.
- [ ] **Start national digital identity onboarding early** if you are government-facing. Integration is straightforward; approval is the calendar cost, and it depends on entity type.
- [ ] **Account for the long tail** — Emirates ID verification, VAT-compliant invoicing, SMS sender ID registration, mapping and geocoding suited to local addressing conventions, and courier APIs of highly variable quality.

## Phase 4 — Subprocessors and egress

- [ ] **Maintain a subprocessor register** with, per entry: data classes touched, processing location, data processing agreement status, last review date. Treat it as production configuration under change control.
- [ ] **Reconcile the register against actual network egress** periodically. This is what catches the tool someone added eleven months ago.
- [ ] **Audit the usual leaks specifically** — APM capturing request bodies, error trackers capturing local variables, session replay recording form contents, support widgets storing conversation history, email and SMS delivery logs, CI systems holding production fixtures, and backup targets in a different region from primary.
- [ ] **Gate egress in CI.** Code paths exporting from restricted classes to external processors should fail the build.
- [ ] **Ensure replicas inherit classification.** Read replicas, search indexes, caches, and warehouse copies in permissive regions are the most common real-world violation and are almost always accidental.

## Phase 5 — AI components

- [ ] **Classify prompts.** A prompt containing restricted fields inherits their classification. A model call with personal data to an endpoint outside a permitted jurisdiction is a cross-border transfer, whatever the architecture diagram calls it.
- [ ] **Prefer regional deployment** or in-region hosting of smaller open models where strict residency binds.
- [ ] **Redact or tokenize** before the call where the task permits — most extraction and classification work fine on pseudonymized input.
- [ ] **Apply source-data retention rules to prompt and completion logs.** Logging is usually configured by a different team than the one that classified the data, which is exactly why this gets missed.
- [ ] **Register model providers as subprocessors** with the same fields as everyone else.
- [ ] **Budget inference as an operating cost** that scales with usage. A feature cheap to build can carry a monthly bill that grows in a way traditional features never did.
- [ ] **Define what may be generated and what must be reasoned through**, and strengthen review accordingly. AI-assisted code is produced faster than it is understood; teams treating generated code as trusted accumulate defects that eventually consume the time saved.

## Phase 6 — Rights and deletion

- [ ] **Engineer the deletion cascade.** A policy promising deletion in thirty days is fiction without the implementation. Foreign keys, soft deletes, backups, search indexes, and downstream analytics copies each need a defined story.
- [ ] **Implement access and export** for data subject requests, including data held by subprocessors.
- [ ] **Implement consent capture and withdrawal** as state the system enforces, not as a checkbox recorded once.
- [ ] **Verify breach detection and notification** paths actually work, with a rehearsed runbook.

## Phase 7 — Commercial, before signing

- [ ] **IP assigns progressively** as work is delivered and paid for — not at final acceptance, which is precisely where troubled projects stall.
- [ ] **Name the categories explicitly**: source, designs, infrastructure-as-code, documentation, and — increasingly valuable, almost always omitted — prompts, evaluation datasets, and fine-tuned artifacts.
- [ ] **Infrastructure in your name from day one.** Cloud accounts, DNS, repositories, CI, app store accounts. Trivial now, near-impossible later.
- [ ] **Acceptance criteria describe behaviour**, not deliverable names. "Backend complete" generates arguments; a behavioural specification does not.
- [ ] **Choose governing law and dispute forum deliberately.** UAE onshore, DIFC, and ADGM courts are genuinely different environments.
- [ ] **Write the exit clause** while everyone is optimistic — credential handover, transition window, documentation standards, named participants for knowledge transfer.

---

Items in Phase 0 and the schema section of Phase 1 are the ones that cannot be cheaply undone. If you only walk part of this list, walk those.

Full country-level buyer's guide, including vendor selection, AED cost bands, and emirate-by-emirate detail: **[App Development Companies in UAE: The 2026 Buyer's Guide](https://techcirkle.com/blog/app-development-companies-in-uae)**.

![Product designers working through mobile app interface wireframes](https://cdn.sanity.io/images/563mnkns/production/18398a9338cfe4b675064cbc0920e98d54b28030-1600x1066.jpg)

## Frequently Asked Questions

### Which items on this list are genuinely irreversible?
Cloud region selection and the schema decisions — name modelling, address modelling, and the normalized search column. Everything else can be refactored at ordinary cost. These require live migrations on production tables or a full re-architecture.

### Do I need all of this for a simple MVP?
Phase 0 in full, plus the name, address, and search-column items from Phase 1. Those cost almost nothing to do correctly at the start. The rest scales with your regulatory exposure and can follow as the product matures.

### How often should the subprocessor register be reconciled?
Quarterly at minimum, and reconciled against actual network egress rather than against documentation. The register drifts because tools get added in sprints without anyone updating a compliance artifact nobody reads.

### Are AI inference calls really a compliance concern?
Yes. A prompt containing personal data sent to an endpoint outside a permitted jurisdiction is a cross-border transfer, architecturally identical to POSTing that data to any third-party API. Classify prompts, register providers as subprocessors, and apply retention rules to logs.

### What is the single most commonly skipped item?
The deletion cascade. Privacy policies promise deletion; systems frequently do not implement it across backups, search indexes, and downstream analytics copies. It surfaces during the first data subject request or the first serious audit.
