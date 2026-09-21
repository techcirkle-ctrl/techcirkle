---
layout: post
title: "Payment App Launch Checklist: Security, PCI Scope and Ops Readiness"
date: 2026-09-21 00:00:00 +0000
categories: ["Security", "Mobile Development"]
tags: ["security", "pci dss", "mobile security", "devops"]
description: "A launch-readiness checklist for payment apps: attestation, certificate pinning, tokenised card entry, PCI DSS 4.0 scope reduction and ops runbooks."
image: "https://cdn.sanity.io/images/563mnkns/production/8cbef389e7114568db5a6fa92e5e8203a41145ed-1600x900.jpg"
author: "TechCirkle Editorial Team"
---

![Customer paying with a smartphone at a contactless POS terminal on a retail counter](https://cdn.sanity.io/images/563mnkns/production/8cbef389e7114568db5a6fa92e5e8203a41145ed-1600x900.jpg)

Launch reviews for payment products tend to fail on the same handful of items: a secret baked into the app binary, a pinning setup with no rotation plan, card data brushing past an application server, or an on-call team with no playbook for a processor outage. None of these are hard to fix before launch. All of them are painful after.

This is the checklist we run before a payment-capable app goes live. It assumes a mobile client talking to your own backend, with card processing, tokenisation and KYC bought from established providers. Tick each line with evidence, not optimism.

The guiding assumption throughout: **the client is compromised.** Every control on the device raises the cost of attack; every limit that matters is enforced on the server.

## 1. Mobile client: device trust

- [ ] **No long-lived secrets in the binary.** API keys and signing material are fetched at runtime after authentication and bound to the device. Assume anything shipped in the app bundle is public.
- [ ] **Platform attestation enabled.** Play Integrity on Android and App Attest on iOS. The backend verifies attestation tokens and rejects or restricts tampered, repackaged or emulated clients.
- [ ] **Attestation failure behaviour defined.** Decide per action: block, require step-up, or allow with lower limits. Test each path.
- [ ] **Runtime protection in place.** Root and jailbreak detection, debugger detection, and screen-capture protection on sensitive views such as card details and one-time codes.
- [ ] **Secure local storage.** Tokens live in Keychain or Keystore-backed storage, never in plain preferences or logs.

## 2. Mobile client: network and authentication

- [ ] **Certificate pinning with a rotation plan.** Pin to a public key or intermediate, ship at least one backup pin, and rehearse a rotation in staging. An untested pin can lock every user out of your app on renewal day.
- [ ] **Biometric step-up for high-risk actions.** Adding a payee, registering a new device, changing contact details or sending an unusual amount should require fresh biometric or equivalent authentication.
- [ ] **Strong Customer Authentication covered.** Where PSD2 or UK equivalents apply, card payments go through 3-D Secure 2 via the processor SDK, with exemptions applied deliberately.
- [ ] **Session binding.** Refresh tokens are bound to the device and revoked on device change, password reset or suspected takeover.

## 3. Card data and PCI DSS 4.0 scope

- [ ] **Tokenised card entry only.** Card fields use the processor's native SDK or hosted components, so the primary account number goes straight to the vault and never transits your servers.
- [ ] **Scope confirmed with your QSA or processor.** With raw PANs kept out, many teams qualify for a short self-assessment questionnaire rather than a full Level 1 assessment. Confirm which one, in writing.
- [ ] **Logs scrubbed.** Mobile crash reporters, API gateways and application logs are checked for accidental card numbers, CVV or full tokens. Add automated detection, not just a one-off grep.
- [ ] **Payment-page scripts controlled.** PCI DSS 4.0 places emphasis on scripts running on payment pages. Any web checkout or webview has an inventory of scripts, integrity checks and change alerts.
- [ ] **Access to the cardholder data environment uses MFA.** Including for engineers and support staff, not just administrators.

![Customer scanning a QR code with a phone to pay at a coffee shop counter](https://cdn.sanity.io/images/563mnkns/production/57256c099c9b8937f3b381837f0145db81b3b20b-1600x1066.jpg)

## 4. Backend controls

- [ ] **Every limit enforced server-side.** Transaction limits, velocity caps and payee rules live in the backend; the app merely displays them.
- [ ] **Idempotency keys on every money-moving endpoint.** Retries return the original result. Keys are forwarded to the processor where supported.
- [ ] **Double-entry, append-only ledger.** No in-place balance updates. Corrections are reversing entries. A daily zero-sum check alerts on drift.
- [ ] **Webhook handling hardened.** Signatures verified, duplicate events deduplicated by provider ID, out-of-order events checked against the payment state machine.
- [ ] **Secrets management and key rotation.** Processor keys, signing keys and database credentials sit in a managed secrets store with a documented rotation schedule.
- [ ] **AI kept advisory.** Any fraud model or LLM can recommend, flag or draft; none can authorise, reverse or move funds on its own. Inputs and outputs are logged for audit.

## 5. Compliance evidence

- [ ] **Licensing position documented.** Either you hold the relevant licence, or a sponsor bank or banking-as-a-service partner does, and your contracts reflect who owns which part of the ledger.
- [ ] **AML and KYC flows live.** Customer due diligence, sanctions screening, transaction monitoring and a suspicious-activity escalation path.
- [ ] **Data residency mapped.** You know where databases, backups, logs and third-party model calls physically run, against GDPR, UK GDPR, CCPA or local rules.
- [ ] **Penetration test completed** on the app and API, with critical and high findings closed and retested.

## 6. Operations runbooks

Launch day is when operations begins. Before go-live, each of these exists as a written, rehearsed runbook:

- [ ] **Processor outage.** How to detect it, how to fail over or degrade gracefully, and what users see.
- [ ] **Duplicate charge reported.** How to confirm it from the ledger, refund safely with idempotency, and communicate with the customer.
- [ ] **Suspected account takeover.** Freeze, revoke sessions, review recent payees and payouts, contact the user.
- [ ] **Reconciliation break.** Daily matching against processor and bank settlement files, with an owner and a deadline for every unexplained item.
- [ ] **Certificate or key compromise.** Emergency rotation steps, including the pinned certificate path.
- [ ] **Monitoring and alerting.** Approval rate, latency and error codes by processor, with alerts on sudden shifts, plus fraud model drift monitoring.

## Using this checklist

Treat each unchecked box as a launch blocker unless someone senior accepts the risk in writing.

This checklist is a companion to our longer guide on [mobile payment platform development](https://techcirkle.com/blog/mobile-payment-platform-development), which covers architecture, rails, compliance, AI and cost ranges. If you want a second pair of eyes on a payment app before release, our [mobile app development](https://techcirkle.com/development/mobile-app-development) team runs these reviews, or you can [contact TechCirkle](https://techcirkle.com/contact-us) directly.

## Frequently Asked Questions

### Is certificate pinning still worth it for payment apps?

Yes, for high-value flows, provided you pin to keys rather than a single leaf certificate, ship backup pins and rehearse rotation. Pinning without a rotation plan creates outage risk that can outweigh the security gain.

### What does app attestation actually protect against?

It lets your backend confirm that requests come from a genuine, unmodified copy of your app on a real device. That makes repackaged clients, emulators and scripted abuse considerably harder, though it does not replace server-side limits.

### How can a new payment app keep its PCI DSS 4.0 scope small?

Use the processor's SDK or hosted fields for card entry so raw card numbers never touch your systems, then confirm your assessment type with your processor or QSA and keep logs free of card data.

### Which runbooks matter most in the first month after launch?

Processor outage, duplicate charges, account takeover and daily reconciliation breaks. These are the incidents most likely to hit early and the ones that erode trust fastest if handled slowly.

### Should fraud models be allowed to block payments automatically?

They can feed an automated decision engine, but the final action should come from deterministic rules with clear reason codes and audit logs. Generative models in particular should never directly authorise or move funds.

### Who should sign off on launch readiness?

Engineering, security and compliance leads together, with every open item fixed or formally risk-accepted.
