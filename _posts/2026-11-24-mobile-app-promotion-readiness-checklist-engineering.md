---
layout: post
title: "Mobile App Promotion Readiness: An Engineering Checklist Before the First Campaign"
date: 2026-11-24 00:00:00 +0000
categories: ["Engineering", "Reference"]
tags: ["mobile", "checklist", "analytics", "ios", "android"]
description: "Checklist for mobile app promotion readiness: events, server revenue, iOS conversion values, deferred deep links, remote config and consent."
image: "https://cdn.sanity.io/images/563mnkns/production/846d16eb03419ca31a7c58c5c652a630a1779bfc-1600x1067.jpg"
author: "TechCirkle"
---

![Hands operating a smartphone with an array of colourful app icons floating above the display](https://cdn.sanity.io/images/563mnkns/production/846d16eb03419ca31a7c58c5c652a630a1779bfc-1600x1067.jpg)

This page is a reference checklist for engineering teams. Run it before the first paid campaign goes live. Every unchecked item tends to raise the cost of each valuable user once spending starts.

Context and strategy — channels, store optimisation, creative, budgeting — are covered in the full guide to [mobile app promotion](https://techcirkle.com/blog/mobile-app-promotion). This page covers only what has to exist in the product.

Treat each section as a gate: one owner per item, and "done" means verified in a production build, not merged.

## 1. Event taxonomy

- [ ] Versioned event schema lives in the repo (e.g. `events/schema.yaml`).
- [ ] Naming convention is fixed (`object_action`, snake_case, past tense).
- [ ] Typed event clients are generated for iOS, Android and backend from the schema.
- [ ] CI fails on events not declared in the schema.
- [ ] Core funnel events exist: `onboarding_step_completed`, `first_value_reached`, `paywall_viewed`, `purchase_completed`, `subscription_renewed`, `invite_sent`.
- [ ] `first_value_reached` is defined in writing and agreed with product.
- [ ] Every event carries `platform`, `app_version`, `install_source` and `consent_state`.

## 2. Server-confirmed revenue

- [ ] App Store Server Notifications (v2) and Google Play Real-time Developer Notifications are consumed by the backend.
- [ ] Notification signatures are verified before processing.
- [ ] Purchase and renewal events are emitted only from the server.
- [ ] Refunds and revocations are emitted as events.
- [ ] Processing is idempotent on transaction ID.
- [ ] A reconciliation report compares store revenue, MMP revenue and ad-platform-reported revenue.

## 3. Event forwarding

- [ ] One forwarding service fans events out to the MMP, ad platform conversion APIs, warehouse and messaging tool.
- [ ] Per-destination allowlists: ad platforms receive only conversion events.
- [ ] Consent is enforced centrally from `consent_state`.
- [ ] Stable `event_id` shared between client and server for deduplication.
- [ ] Retries with backoff, dead-letter queue and a replay tool.

## 4. iOS privacy attribution

- [ ] SKAdNetwork / AdAttributionKit integration verified with the MMP and ad networks in use.
- [ ] Conversion-value schema documented: bit allocation, coarse mapping, later-window meaning.
- [ ] Values only ever increase; updates go through one wrapper.
- [ ] Revenue buckets come from server-confirmed purchases.
- [ ] Schema version is remote-configurable and logged.
- [ ] Decoding table is mirrored in the warehouse.

## 5. Deep links

- [ ] Universal Links (iOS) and App Links (Android) verified on production domains.
- [ ] Deferred deep linking works: a user who taps an ad or link for specific content lands on that content after first install.
- [ ] Link parameters are captured as `install_source` / campaign properties.
- [ ] Links are tested on a fresh device, not one with the app already installed.

## 6. Remote configuration and experiments

- [ ] Onboarding copy, step order and paywall layout are remote-configurable.
- [ ] Feature flags support percentage rollouts and kill switches.
- [ ] Experiment assignment is logged as an event property.

## 7. Onboarding

- [ ] Time to `first_value_reached` is measured and dashboarded.
- [ ] Core feature is usable before account creation where the product allows.
- [ ] Permission prompts are deferred until the feature that needs them.
- [ ] Paywall position is an experiment, not a constant.
- [ ] If an LLM-guided setup is used, structured app logic, not the model, controls what is saved. See [LLM integration](https://techcirkle.com/llm-integration) patterns.

## 8. Consent and privacy

- [ ] ATT prompt (iOS 14.5+) appears at a moment where the benefit is clear, with a pre-prompt explanation.
- [ ] Consent management covers GDPR, UK GDPR and applicable US state privacy laws.
- [ ] Consent changes propagate to the forwarding service within a defined time.
- [ ] Privacy nutrition labels and Play data safety section match actual data flows.

## 9. Store presence

- [ ] Store listing experiments / custom product pages are set up for primary campaigns.
- [ ] In-app rating prompt fires after a success moment, rate-limited.
- [ ] Review monitoring routes recurring complaints into the backlog.

## 10. Measurement readiness

- [ ] Cohort dashboards: activation, D1/D7/D30 retention, revenue by install week and source.
- [ ] Predictive LTV model planned or live, with a defined horizon and retraining schedule.
- [ ] Incrementality method chosen (geo holdouts, lift studies, channel pauses).
- [ ] For larger budgets, MMM evaluated (Google's Meridian, Meta's Robyn are open source).
- [ ] Re-engagement messaging is event-triggered, frequency-capped, and opt-out/uninstall rates are tracked.

## Sign-off

Promotion should not scale until sections 1–5 and 8 are complete. Sections 6, 7, 9 and 10 can follow in the first weeks, but each gap makes paid spend less efficient.

Need help closing the gaps? Our [mobile app development services](https://techcirkle.com/development/mobile-app-development) treat this checklist as launch scope, and the predictive models behind section 10 are built by our [AI development services](https://techcirkle.com/ai-development-services) team.

## Frequently Asked Questions

### Which checklist items are hard blockers for launch spend?

Event taxonomy, server-confirmed revenue, event forwarding, iOS privacy attribution, deep links and consent. Without them, campaign data is unreliable from the first day.

### How do I test deferred deep linking properly?

Use a device that has never had the app installed, tap a campaign link for specific content, install from the store, open the app and confirm it lands on that content.

### Why must revenue events come from the server?

Server notifications from Apple and Google are verified and include refunds, while client callbacks can be lost to crashes, offline states or blocked SDKs.

### Does every app need a predictive LTV model before launch?

No, but it should be planned. Clean events and cohort dashboards come first; the model follows once a few months of cohort data exist.

### Who should own this checklist?

An engineering lead, with sign-off from product and growth. It sits between teams, which is why it often goes unowned.
