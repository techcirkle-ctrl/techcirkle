---
layout: post
title: "Own Your Signing Keys: A Contract Checklist for Mobile Work"
date: 2026-08-24 00:00:00 +0000
categories: ["Engineering", "Checklists", "Mobile"]
tags: ["checklist", "mobile", "contracts", "devops"]
description: "A checklist of accounts, credentials, contract terms and handover artefacts to settle before a contractor or agency writes a line of mobile code."
image: "https://cdn.sanity.io/images/563mnkns/production/9d777c92d9408f8de580960ce07517292646e72a-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

![Mobile development team working on an application](https://cdn.sanity.io/images/563mnkns/production/9d777c92d9408f8de580960ce07517292646e72a-1600x1066.jpg)

The expensive mistakes in outsourced mobile development are almost never technical.

They are administrative: an app listing on a contractor's personal developer account, a signing certificate that exists on one laptop, a push notification key nobody can find, an analytics property owned by an agency that stopped replying to emails.

Each is trivial to prevent on day one and genuinely painful to fix afterwards. This is the checklist.

## Accounts and ownership

- [ ] Apple Developer Program membership is an **organisation** account under your legal entity, with a D-U-N-S number in your company's name.
- [ ] Google Play developer account is registered to your company, not an individual.
- [ ] At least two people from **your** organisation hold admin or account-holder access to both.
- [ ] The contractor or agency has developer-level access, not account ownership.
- [ ] Domain, DNS, and any associated web properties are owned by your entity.

## Signing and credentials

- [ ] Distribution certificates and provisioning profiles are stored in a shared, access-controlled vault your company owns.
- [ ] The Android upload key and its backup are held by you; Play App Signing is enabled.
- [ ] Push notification keys and service credentials are in your accounts.
- [ ] API keys for third-party SDKs — analytics, crash reporting, maps, payments — are issued from your accounts.
- [ ] No credential exists only on a contractor's machine. Verified by asking, in writing.

## Repository and build

- [ ] Source control is hosted in your organisation from the first commit.
- [ ] CI configuration lives in the repository, and the CI project is under your account.
- [ ] Clone to running on a device takes one documented command.
- [ ] The build does not depend on a manually installed tool on one specific machine.
- [ ] Release builds are produced by CI, not by an individual's laptop.

![Team reviewing mobile implementation details](https://cdn.sanity.io/images/563mnkns/production/4a8e60ec545fd0dddc5f6cb389824cca6e6d2fbc-1600x1068.jpg)

## Contract terms

- [ ] IP assigns to your company on payment: source, designs, build configuration, and — for AI features — prompts and evaluation data.
- [ ] Named key personnel cannot be substituted without your written agreement.
- [ ] A defined exit: notice period, documented handover, knowledge-transfer window with dates.
- [ ] A written change protocol if the engagement is fixed scope.
- [ ] Confidentiality and data handling terms that match your customers' contractual expectations.

## Operational handover artefacts

- [ ] Runbooks for release, rollback, hotfix and credential rotation.
- [ ] Architecture decision records for irreversible choices.
- [ ] Crash reporting and analytics dashboards in your accounts, with alerts routed to your people.
- [ ] Store listing assets — screenshots, descriptions, localisations — in your storage, in editable form.
- [ ] A record of every third-party SDK, its purpose, its licence, and what data it collects.

## AI-specific additions

- [ ] Model provider accounts and API keys belong to your organisation.
- [ ] Prompts, prompt templates and evaluation datasets are versioned in the repository.
- [ ] On-device model artefacts and their build pipeline are reproducible from source.
- [ ] Cost and latency baselines per feature are recorded so regressions are detectable.
- [ ] A written statement of what user data leaves the device and to which providers — you will need it for security reviews and store privacy declarations.

## The rehearsal

One test invalidates or validates the whole list.

**Ask an engineer with no prior access to produce a signed release build and submit it to the store's internal testing track, using only the repository and documentation.**

Every question they must ask the contractor is a gap. Every credential they cannot find is a future emergency. Running this once during the engagement — rather than at the end, when goodwill is lowest — turns each finding into an ordinary task.

Do it in month two, then again before the engagement ends.

---

*The full hiring and engagement playbook — role definition, sourcing, screening, interview formats, cost bands and onboarding — is here: [Hire Mobile App Developers in 2026: A Practical Playbook](https://techcirkle.com/blog/hire-mobile-app-developers).*

## Frequently Asked Questions

### Who should own the Apple and Google developer accounts?

Your legal entity, as organisation accounts, with at least two administrators from your own company. Contractors and agencies should hold developer-level access only. This single decision prevents the most common and most painful outsourcing failure.

### What happens if a contractor holds the signing certificate?

You cannot publish updates without them. On Android, Play App Signing plus a company-held upload key mitigates the risk; on iOS, distribution certificates and provisioning profiles must be stored in a vault your company controls, with more than one person able to access them.

### What contract terms matter most for mobile work?

IP assignment on payment covering source, designs, build configuration and AI assets; named key personnel who cannot be swapped silently; a defined exit with a dated knowledge-transfer window; and a written change protocol for fixed-scope engagements.

### What AI-related assets should be handed over?

Provider accounts and API keys under your organisation, versioned prompts and evaluation datasets in the repository, a reproducible pipeline for any on-device model artefacts, recorded cost and latency baselines, and a written statement of what user data leaves the device and to whom.

### How do I verify the handover is real?

Ask an engineer with no prior access to produce a signed release build and submit it to an internal testing track using only the repository and documentation. Every question they must ask the outgoing team marks a gap worth closing while it is still routine.

### When should the handover rehearsal happen?

Twice: once around month two, when findings are ordinary tasks and goodwill is high, and again before the engagement ends. Leaving it entirely to the end guarantees discovering problems at the point when everyone has least incentive to fix them.
