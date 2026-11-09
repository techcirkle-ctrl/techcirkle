---
layout: post
title: "Field Guide: Shipping Many Branded Apps From One Repo"
date: 2026-11-09 00:00:00 +0000
categories: ["Engineering Notes", "Mobile"]
tags: ["mobile", "checklist", "cicd", "multitenancy", "engineering"]
description: "Engineering notes and a working checklist for shipping white label apps from one repo: config, builds, signing, releases and tenant isolation."
image: "https://cdn.sanity.io/images/563mnkns/production/d1a0594d6820fd487d6ce68521a65731835774cc-1600x1015.jpg"
author: "TechCirkle"
---

These notes are a working checklist for a team shipping several branded mobile apps from one repository. Use them for reviewing an existing platform or planning a new one. Each section ends with the failure it is meant to prevent. The definition of done throughout: onboarding a client is a configuration task, and never a development project.

![A hand holding a frameless smartphone with a blank white screen](https://cdn.sanity.io/images/563mnkns/production/d1a0594d6820fd487d6ce68521a65731835774cc-1600x1015.jpg)

## 0. Confirm you need separate apps at all

Separate store listings are the expensive path, so check for a cheaper one first.

- [ ] Could one app with tenant selection at login meet the need?
- [ ] Would a progressive web app do the job?
- [ ] For business customers, does Apple's custom app distribution apply?
- [ ] Is each client's need for its own listing written down?

*Prevents: building signing and submission machinery nobody required.*

## 1. Repository and tenant layout

```text
/app                 shared application code
/design-system       components, driven only by tokens
/tenants
  /northside
    manifest.json    identity and build inputs
    tokens.json      color, type, spacing
    /assets          icons, splash
    /listing         store text, screenshots, privacy answers
/scripts             generators for targets and flavors
```

- [ ] One directory per tenant, and no tenant logic outside it.
- [ ] Manifest has a schema, validated in CI.
- [ ] No secrets in the repo. Manifests carry references only.
- [ ] Protected branch policy: no long-lived client branches.
- [ ] Lint rule or review rule rejecting tenant-ID conditionals in shared code.

*Prevents: the private fork, which is cheap on the day and expensive for every release after it.*

## 2. Tenant configuration

- [ ] Branding, flags, content, integrations, languages and policies live in a versioned configuration service.
- [ ] Every config change has an author and can be rolled back.
- [ ] Unknown or missing flags fall back to a safe default.
- [ ] Design tokens cover color, type and spacing, so no per-client code is needed for visuals.
- [ ] Extension points exist and are documented: webhooks, plug-in integrations, configurable workflows.

*Prevents: "just this once" code paths.*

## 3. Build targets

Flutter flavors, React Native with Expo application configuration, or Xcode schemes with Android product flavors all work. The choice matters less than this: targets are generated.

- [ ] Bundle ID, application ID, icons and display name derived from the manifest.
- [ ] Deep link domains generated per tenant, with association files published.
- [ ] Push credentials configured per tenant.
- [ ] Building any tenant is one command: `make build TENANT=northside PLATFORM=ios`.
- [ ] A nightly job builds every tenant, not just the one someone happens to be working on.

*Prevents: a tenant that has quietly stopped compiling, discovered on release day.*

## 4. Accounts and signing

Store rules drive this section. Apple's guideline 4.2.6 rejects apps from a commercialized template or app generation service unless the content provider submits them directly, 4.3 is regularly cited against lookalike apps sharing a binary, and Google Play's spam policies are similar. Reinterpretation happens, so re-read the current text at intervals.

- [ ] Each client's app is published under the client's own developer account.
- [ ] Client enrollment is tracked as an onboarding task, including the D-U-N-S number for an Apple organization account.
- [ ] Signing identities, upload keys and store API keys are held in a secrets manager, keyed by tenant.
- [ ] CI jobs can read only the secrets of the tenant they build.
- [ ] Certificate and credential expiry is monitored per tenant.
- [ ] A documented process exists for a client leaving, including handing over credentials.

*Prevents: an expired certificate blocking a hotfix, or one compromised job exposing every client.*

## 5. Store submission

Manual releases across many listings do not scale.

- [ ] Build and submission run through an automated pipeline.
- [ ] Listing text, screenshots and privacy disclosures are versioned per tenant.
- [ ] Each listing has real client-specific content and functionality. Cosmetic reskins are the weakest case at review.
- [ ] Review outcomes are logged per tenant, per release.
- [ ] Someone owns compatibility work for each new iOS and Android version.

*Prevents: a release week consumed by copy and paste, and a review history no one can recall.*

## 6. Release safety

- [ ] Over-the-air updates roll out in stages.
- [ ] Each tenant has a kill switch.
- [ ] A minimum supported version is enforced, with a forced-upgrade path.
- [ ] Release waves start with a low-risk group of tenants.
- [ ] Rollback has been rehearsed, and not merely documented.

*Prevents: one bad bundle reaching every brand at once.*

## 7. Data isolation

- [ ] Isolation model chosen: row-level, schema per tenant, or dedicated database as a paid tier.
- [ ] Tenant identity enforced at the data layer, not remembered in application code.
- [ ] Cross-tenant tests run in CI. Seed two fictional tenants and try to read one from the other's session.

*Prevents: the incident that ends client relationships.*

## 8. If the product has AI features

- [ ] Tenant filter enforced inside the retrieval layer.
- [ ] Prompt cache keys and conversation memory partitioned by tenant.
- [ ] Agent credentials scoped per tenant and per user.
- [ ] System prompts kept apart from retrieved content, which is treated as untrusted.
- [ ] Tokens, embeddings and tool calls metered per tenant.
- [ ] Each tenant has a budget and a defined behavior at the limit.
- [ ] Evaluations run against each tenant's scenarios before a prompt change ships.

*Prevents: leaks through paths ordinary access checks do not reach, and a heavy client eating the margin.*

## Closing note

Most items here are cheap on the first tenant and painful to retrofit by the tenth. AI tools make it easy to skip them by patching individual apps, which is exactly why the checklist belongs in the release process.

For the non-engineering context on cost, contracts and buy-versus-build, see our [full guide to white label apps](https://techcirkle.com/blog/white-label-apps). If you need help building this kind of pipeline, our [mobile app development](https://techcirkle.com/development/mobile-app-development) team does this work.

## Frequently Asked Questions

### How many tenants justify automating store submission?

Automate early. Manual releases stop working after a handful of clients, and the pipeline is far easier to build while there are few listings to migrate.

### Should each brand have its own git branch?

No. Keep one main line and one directory per tenant. Long-lived client branches drift apart and multiply the work of each patch and operating system update.

### What is the minimum for release safety across many apps?

Staged rollouts, a per-tenant kill switch and an enforced minimum supported version. Together they let you stop a bad release and retire old builds without affecting other tenants.
