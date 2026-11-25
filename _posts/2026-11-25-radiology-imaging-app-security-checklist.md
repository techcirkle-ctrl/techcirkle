---
layout: post
title: "A Security Checklist for Radiology and Imaging Apps"
date: 2026-11-25 00:00:00 +0000
categories: ["Security", "Healthcare"]
tags: ["security", "healthcare", "dicom", "privacy", "checklist"]
description: "Security checklist for radiology apps: encryption, device cache expiry, SSO and MFA, audit logs, secure links and de-identifying burned-in text."
image: "https://cdn.sanity.io/images/563mnkns/production/846d16eb03419ca31a7c58c5c652a630a1779bfc-1600x1067.jpg"
author: "TechCirkle"
---

![Hands using a smartphone with colourful app icons floating above the screen, representing a secured mobile imaging app](https://cdn.sanity.io/images/563mnkns/production/846d16eb03419ca31a7c58c5c652a630a1779bfc-1600x1067.jpg)

Imaging systems have a long history of security weakness: DICOM servers exposed to the internet, default credentials, unencrypted transfers between modalities and archives. Mobile and web viewers move that risk further out, onto phones, tablets and home workstations.

This page is a working checklist for teams building radiology and imaging apps. It is the security-focused companion to our broader guide to [radiology app development](https://techcirkle.com/blog/radiology-app). Use it in design reviews and before security questionnaires. It is not legal or compliance advice; confirm obligations for your markets with qualified specialists.

## 1. Transport

- [ ] TLS on every hop: device to API, API to archive, archive to AI services, and internal service-to-service calls.
- [ ] No plaintext classic DICOM across untrusted networks; tunnel or gateway it, and prefer DICOMweb over HTTPS for app-facing access.
- [ ] Certificate pinning considered for mobile clients.
- [ ] No DICOM listener reachable from the public internet.

## 2. Data at rest

- [ ] Archives, object stores and databases encrypted at rest with managed keys.
- [ ] Key rotation and access to keys logged and restricted.
- [ ] Backups encrypted and covered by the same access controls as primary storage.

## 3. On-device caches

Offline and low-bandwidth modes are essential for on-call clinicians, and they are where most mobile risk lives.

- [ ] Cached studies encrypted using the platform keystore.
- [ ] Aggressive cache expiry, measured in hours rather than days unless there is a documented reason.
- [ ] Remote wipe supported and tested.
- [ ] Cache bound to the authenticated session; logout or session expiry purges it.
- [ ] Pixel data never written to shared storage, crash reports, analytics events or logs.
- [ ] Screenshots and app-switcher previews considered and restricted where appropriate.

## 4. Identity and access

- [ ] Single sign-on with the health system's identity provider.
- [ ] Multi-factor authentication enforced.
- [ ] Role-based access control down to study level, not just application level.
- [ ] Break-glass access, where offered, is time-limited, justified and heavily logged.
- [ ] Session timeouts appropriate for shared clinical devices.
- [ ] No shared or default accounts anywhere in the stack.

## 5. Audit logging

- [ ] Every view, export, share, print and download logged with user, study, time and source.
- [ ] Logs are tamper-evident and retained according to policy.
- [ ] AI inference events logged with the model name and version that produced each output.
- [ ] Logs can answer "who accessed this patient's images, and when?" quickly.

Comprehensive audit trails are essential for HIPAA in the US and for equivalent regimes elsewhere.

## 6. Sharing

- [ ] Patient and external-clinician links expire by default.
- [ ] Links can be revoked instantly.
- [ ] Links scoped to the minimum set of studies needed.
- [ ] Access through a link is logged like any other access.

## 7. De-identification for AI and research

Any data that leaves the clinical boundary for model training, evaluation or research needs a de-identification pipeline.

- [ ] DICOM header fields scrubbed according to a documented profile.
- [ ] **Burned-in text detected and removed from pixel data.** Ultrasound, secondary captures and scanned documents often carry names, dates or identifiers inside the image itself. Header scrubbing alone will miss them.
- [ ] Private tags reviewed; unknown private tags removed by default.
- [ ] UIDs remapped consistently so studies remain linkable inside the dataset without exposing originals.
- [ ] Sample-based human review of de-identified output before each dataset release.
- [ ] Dataset versions immutable and traceable to source.

A minimal pipeline sketch:

```
ingest -> header scrub (profile vN) -> private tag policy
       -> burned-in text detection (OCR) -> pixel redaction
       -> UID remap -> QA sampling -> versioned dataset
```

## 8. Third-party and AI vendor risk

- [ ] Each AI or cloud service receives only the data it needs.
- [ ] Business associate agreements, or local equivalents, in place before data flows.
- [ ] An orchestration broker centralises routing so data minimisation is enforced in one place.
- [ ] LLM features such as report drafting run with prompt and output logging, grounding and guardrails; our [LLM integration](https://techcirkle.com/llm-integration) work covers this pattern.

## 9. Application security hygiene

- [ ] Dependency scanning and patching cadence defined.
- [ ] Regular penetration tests, with results available for customer security reviews.
- [ ] Secrets in a vault, never in code or mobile bundles.
- [ ] Threat model maintained for the viewer, the API and the AI pipeline.

## 10. Procurement readiness

Hospitals will send security questionnaires, request penetration test results, review your business associate agreement and ask about data residency.

- [ ] Evidence pack prepared: architecture diagram, data flow, encryption summary, access model, audit capabilities, pen test summary.
- [ ] Data residency options documented per market.
- [ ] Named owner for security reviews with a target turnaround.

Teams that can answer a hospital security review in days rather than months win deals that technically stronger competitors lose. If you want help building an imaging product with these controls designed in, our [custom software development](https://techcirkle.com/development/custom-software-development) team works on exactly this.

## Frequently Asked Questions

### What is burned-in text and why does it matter?

It is text rendered into the image pixels, such as a name or date on an ultrasound frame. Because it is not in the DICOM header, standard scrubbing misses it, so pipelines need OCR-based detection and redaction.

### How long should images stay cached on a mobile device?

As briefly as clinical use allows. Short expiry, encryption, session binding and remote wipe together keep a lost device from becoming a breach.

### Is TLS enough to secure a radiology app?

No. It protects data in transit, but you also need encryption at rest, strong identity, study-level access control, audit logging and safe sharing.

### What should audit logs capture for imaging?

Who viewed, exported or shared which study, when and from where, plus which AI model version produced any result shown to a clinician.

### How should data reach third-party AI services?

Through a broker that sends only the minimum necessary data, under appropriate agreements, with every request and result logged.

### What do hospitals usually ask for in a security review?

Questionnaire answers, penetration test results, the business associate agreement and data residency details. Having an evidence pack ready shortens the process.
