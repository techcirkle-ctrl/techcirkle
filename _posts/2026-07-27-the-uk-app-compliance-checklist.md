---
layout: post
title: "The UK App Compliance Checklist"
date: 2026-07-27 00:00:00 +0000
categories: ["Engineering", "Compliance"]
tags: ["gdpr", "compliance", "mobile", "accessibility"]
description: "UK GDPR, the ICO Children's Code, store privacy declarations and WCAG 2.2 — the compliance work that adds weeks and is missing from cheap quotes."
author: "TechCirkle Editorial Team"
---

Compliance is the most commonly under-quoted part of a UK app build, because it is invisible until it blocks something. Nobody sees it in a demo. Everybody feels it in week eighteen.

This is the checklist, organised by what it actually means for engineering rather than by statute.

## UK GDPR and the Data Protection Act 2018

The baseline for any app handling personal data.

**Lawful basis** — established per processing purpose, before you build. This is a product decision with architectural consequences: consent-based processing needs granular, withdrawable consent recorded per purpose, which affects your data model.

**Data minimisation** — collect what you need. In practice this means auditing what your SDKs collect, which is frequently more than you think and always more than your privacy notice says.

**Subject access** — users can request everything you hold. Doing this by hand does not scale; build the export path.

**Erasure** — the expensive one. Real erasure reaches:

- Primary tables and every denormalised copy
- Analytics warehouses and event streams
- Search indices
- Logs, which routinely contain personal data nobody intended to log
- Backups, where documented rotation is the usual accepted position
- Third-party processors, including AI providers you sent context to

A `deleted_at` column does not satisfy this. Building erasure into the schema costs a little; retrofitting it costs a lot, because it touches everything.

**Data portability** — structured, commonly used, machine-readable format. Usually shares an implementation path with subject access, so build them together.

## The ICO Age Appropriate Design Code

Applies if under-18s are **likely** to access your service — which is a lower bar than "designed for children" and catches more products than teams expect.

Fifteen standards. The ones with direct engineering consequences:

- **High privacy by default.** Every setting defaults to the most protective option. Not opt-out — default.
- **Geolocation off by default**, with a visible indicator when active.
- **Profiling off by default.**
- **No nudge techniques** toward weaker privacy settings. This constrains your onboarding flow and your interface copy.
- **Age assurance proportionate to risk.** A self-declared date of birth may be adequate for low-risk services and is not adequate for high-risk ones.
- **Data minimisation**, applied more strictly than the general standard.

This is prescriptive and it is enforced. If there is any chance under-18s use your product, treat it as in scope and design for it rather than assessing later.

## App Store and Play Store privacy declarations

Both stores require detailed disclosure of what data your app collects and how it is used.

The requirement that catches teams: **the declaration must match what your code actually does**, including everything your third-party SDKs collect on your behalf. Analytics and advertising libraries collect more than most teams realise.

Practical step: audit SDK data collection before completing the declaration, not after. Mismatches trigger rejections, and increasingly, removals of live apps.

Re-audit whenever you add or upgrade an SDK. This is a recurring obligation, not a launch task.

## Accessibility — WCAG 2.2 AA

A procurement requirement for UK public sector work and a growing expectation everywhere else.

For mobile specifically:

- Dynamic type support — the app must remain usable at large text sizes, which affects every layout
- Minimum contrast ratios
- Touch targets meeting minimum size
- Full screen reader support: VoiceOver and TalkBack, with meaningful labels on every interactive element
- Focus order that makes sense
- No information conveyed by colour alone
- Respect for reduced-motion preferences

Retrofitting accessibility costs several times what building it in does, because it touches every screen. Dynamic type in particular tends to break layouts that were designed at a single text size.

## Sector-specific requirements

- **FCA** — if you touch payments, credit, or investments. Authorisation requirements, specific disclosures, and rules around communications.
- **MHRA** — if your app makes clinical claims it may be a medical device, which is a fundamentally different regulatory path and needs assessing early.
- **PCI DSS** — if you handle card data directly. Almost always better to delegate to a provider and keep card data out of your scope entirely.

## Engineering practices that make all of this cheaper

**Centralise personal data.** A small number of owning services with clear identifiers makes erasure and export bounded operations rather than estate-wide searches.

**Tag data at the schema level.** Annotate which fields are personal data and which processing purpose they serve. Makes audits tractable and makes erasure implementable.

**Keep an SDK inventory** with what each collects. Review on every dependency update.

**Log deliberately.** Personal data ends up in logs by accident constantly. Filter at the logging layer rather than hoping.

**Version your consent records.** You need to demonstrate what a user agreed to and when, against the notice text current at that moment.

## Budget

Four to eight weeks of genuine effort for a regulated build, spread through the project rather than bolted on at the end. Bolting it on at the end is what turns compliance into a crisis.

A vendor who raises compliance unprompted in the first conversation has shipped in your sector. One who waits for you to raise it probably has not.

---

Full guide to choosing a UK app development company — costs, archetypes, portfolio red flags and contract terms — at <https://techcirkle.com/blog/mobile-app-development-company-uk>.

## Frequently Asked Questions

### Does the ICO Children's Code apply to my app?

If under-18s are *likely* to access your service, yes — which is a lower threshold than being designed for children and catches more products than teams expect. If there is a realistic chance, treat it as in scope and build to it rather than assessing after launch, since several of the requirements are architectural.

### What makes erasure so expensive to retrofit?

Because it reaches far beyond the primary table: denormalised copies, analytics warehouses, event streams, search indices, logs that captured personal data unintentionally, backups, and third-party processors including AI providers. Centralising personal data behind a few owning services from the start turns it into a bounded operation.

### How strict are the app store privacy declarations?

Strict enough that mismatches trigger rejections and, increasingly, removal of live apps. The declaration must reflect what your code and every third-party SDK actually collect. Audit SDK collection before completing the declaration, and re-audit whenever you add or upgrade a dependency.

### Is WCAG 2.2 AA legally required for private-sector UK apps?

It is a procurement requirement for public sector work and a growing commercial expectation elsewhere, alongside general Equality Act obligations. Practically, dynamic type support and screen reader labelling are the items that most often force layout rework, which is why building them in is far cheaper than retrofitting.

### How much time should I budget for compliance?

Four to eight weeks of genuine effort on a regulated build, distributed across the project rather than concentrated at the end. Treating it as a pre-launch task is what converts a manageable workload into a release-blocking crisis.
