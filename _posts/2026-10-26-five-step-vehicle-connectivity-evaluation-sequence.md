---
layout: post
title: "A Five-Step Evaluation Sequence for Vehicle Connectivity Platforms"
date: 2026-10-26 00:00:00 +0000
categories: ["IoT", "Engineering Practices", "Architecture"]
tags: ["iot", "evaluation", "architecture", "checklist"]
description: "An ordered evaluation sequence for vehicle IoT connectivity platforms, with the artefact each step must produce before the next begins."
image: "https://cdn.sanity.io/images/563mnkns/production/83f701f2dd487e2c2591600e803f1f3b80c464f3-1600x1066.jpg"
author: "TechCirkle"
---

![An aerial view of trucks parked in a logistics yard](https://cdn.sanity.io/images/563mnkns/production/83f701f2dd487e2c2591600e803f1f3b80c464f3-1600x1066.jpg)

The order is the point. Each step produces an artefact that constrains the next, and running them out of sequence is the mechanism by which fleet programmes end up rearchitecting eighteen months in.

Do not begin step 3 before step 2's artefact exists.

---

## Step 1 — Fix inference placement

**Artefact: a table assigning every workload on your 18-month roadmap to a tier.**

| Tier | Assign when | Examples |
|---|---|---|
| On-vehicle | Must work with no network; intervention latency <200 ms; high input:output compression ratio | Collision warning, distraction detection, harsh-event classification |
| Regional edge | Correlation across a vehicle cohort but not fleet-wide; first-pass data reduction | Route-level traffic inference, depot dwell optimisation |
| Cloud | Requires whole-estate view; training; latency budget in hours | Fleet analytics, retraining, long-horizon maintenance prediction |

Decision rules, applied in order:

1. Must it work without a network? → **on-vehicle**, no exceptions. A safety behaviour that degrades when connectivity drops is not a safety behaviour.
2. Is the input large relative to the output (video, raw IMU, audio)? → **on-vehicle**. The compression ratio is the entire economic argument.
3. Correlates across vehicles within a region? → **regional edge**.
4. Training? → **cloud**, always.
5. Everything else → **cloud**.

**Cover the roadmap, not production.** The common failure is scoping to today's use cases, then adding dashcam coaching in month nine against a contract priced for position data.

---

## Step 2 — Derive the payload envelope

**Artefact: `p_mean` and `p_peak` in bytes per vehicle-month, at three levels.**

| Level | Payload | Driven by |
|---|---|---|
| Minimal | ~5 MB | Position, diagnostics, ignition events |
| Moderate | 50–200 MB | High-frequency CAN bus telemetry |
| Heavy | 2–20 GB | Video, raw sensor streams |

State **peak as well as mean** — overage is assessed on peaks, and fleets with uneven duty cycles have peak-to-mean ratios well above 1.

This step is mechanical once step 1 exists. If it feels like guesswork, step 1 is incomplete.

---

## Step 3 — Model total cost per vehicle-month

**Artefact: a matrix of candidates × three payload levels × countries of operation.**

```
C = S + D(p) + M + R
```

- `S` — per-SIM subscription, ~$0.30–$2/month, effectively fixed. Dominates at low `p`.
- `D(p)` — data transfer. The only usage-dependent term. Usually piecewise: pooled allowance, then materially worse overage.
- `M` — per-device platform / management fee. **Frequently exceeds `S`.** Most commonly missing from first-pass models.
- `R` — roaming and regional premiums. Zero domestically; can exceed `S + D + M` combined internationally.

Checks to apply:

- [ ] Is the data allowance **pooled** across the fleet? Per-SIM allowances punish uneven duty cycles.
- [ ] Is `M` included, and is it per device or per vehicle?
- [ ] Is `R` modelled **per country**, not as a blended global rate?
- [ ] Have you checked permanent-roaming legality? Brazil, Turkey, India and China among others restrict or prohibit it, so a single global SIM is often not legally available.

**Expect rankings to invert** between minimal and heavy. A vendor with a cheap low tier and punitive overage wins decisively at 5 MB and is unaffordable at 5 GB. Vendors demo whichever tier they win.

---

## Step 4 — Pilot on real vehicles in bad conditions

**Artefact: measured results from 20–50 vehicles over 6–8 weeks.**

Route selection is the whole point. Deliberately include your worst: tunnels, underground depots, rural corridors, border crossings. A pilot in good conditions measures nothing.

Measure:

| Metric | Why | Failure threshold |
|---|---|---|
| Data loss rate | Buffer sizing and eviction correctness | Any loss of discrete events |
| Buffer high-water mark | Whether retention survives your worst outage | >80% on a normal week |
| OTA success rate | Whether edge inference is operationally viable | <98% is expensive |
| Time to detect a bricked device | Fleet observability | >24h |
| Allowance burn under packet loss | Retry thrash in the device agent | Any superlinear spike |

**On OTA specifically:** 94% reads acceptable and means ~600 physical service visits on a 10,000-vehicle fleet. Deliberately brick a device and time the detection.

**On the device agent:** ask how it behaves under sustained 20% packet loss *before* the pilot, then verify. A good answer describes local buffering with backpressure, exponential backoff with jitter, and resetting the retry counter on successful transmission rather than on connection establishment.

---

## Step 5 — Verify exit before signing

**Artefact: draft contract language, not an email from a sales engineer.**

- [ ] **SIM portability** — are SIMs locked? What is the documented process and cost to move the fleet?
- [ ] **Raw data export** — complete history, documented format, any time, no volume-scaled fee. Normalised-only export is a lock-in mechanism.
- [ ] **Firmware signing rights** — held by you or under a defined joint arrangement. If only the provider can sign, switching provider means replacing hardware.
- [ ] **Term vs amortisation** — three years on seven-year devices is reasonable; seven is not.

**Structural defence that outranks all four:** keep your own data store as the system of record and treat the platform as transport and management rather than a database. It converts a provider change from a migration project into a configuration change.

---

## The cross-cutting item: normalisation

Not a step, because it applies throughout.

Vehicle data is not a standard. Mixed fleets mean multiple CAN dialects, manufacturer-specific PIDs, model-year variation within a single make, and J1939/OBD-II/proprietary mixtures in heavy vehicles. EVs add battery management data with almost no cross-manufacturer consistency.

The evaluation question is **not** "do you support our vehicles" — every vendor says yes. It is:

> Which specific parameters do you normalise, for each make and model year we operate, and what happens when we add a make you have not seen?

Ask for the list in writing. A vendor who cannot produce one does not have one. Building the layer yourself for a mixed fleet is a standing commitment in person-quarters per year, because every model year changes something.

---

## Post-deployment signal

Connectivity cost should scale **roughly linearly** with vehicle count. Superlinear growth means a high-compression-ratio or cross-vehicle workload is sitting in the cloud, and your invoice history usually identifies the quarter it was introduced.

---

Full discussion, including UNECE R155/R156 security obligations and the build-versus-assemble decision: [How to Choose the Top IoT Connectivity Platform for Vehicles](https://techcirkle.com/blog/top-iot-connectivity-platform-for-vehicles). The orchestration layer above this: [agentic workflow development](https://techcirkle.com/agentic-workflow-development).

## Frequently Asked Questions

### Why must inference placement be decided in step 1 rather than later?

Because payload is derived from it, and payload determines both the cost model and the vendor requirements. Cloud and edge inference differ by three to four orders of magnitude in data volume for the same business outcome, so the two architectures want genuinely different vendors rather than different tiers of one.

### What should the pilot deliberately include?

Your worst operating conditions — tunnels, underground depots, rural corridors and border crossings. A pilot run in good conditions measures nothing useful, because the expensive failure mode is degraded connectivity rather than total outage.

### What OTA success rate is actually acceptable?

Below roughly 98% becomes operationally expensive. At 94% on a 10,000-vehicle fleet, around 600 vehicles require a physical service visit, which is a materially different staffing and cost model than the headline percentage suggests.

### How should roaming costs be modelled?

Country by country, never as a blended global rate, and with permanent-roaming legality checked for each market. Brazil, Turkey, India and China among others restrict or prohibit it, which means locally provisioned profiles rather than a single global SIM.

### What makes raw data export clauses important?

Because normalised-only export is a lock-in mechanism. Secure complete historical export in a documented format, available at any time, without a fee that scales with volume — and keep your own data store as the system of record so a provider change is a configuration change.

### How do I verify a platform genuinely supports a mixed fleet?

Request the written list of parameters normalised per make and model year you operate, and ask what happens when an unfamiliar make is added. Universal claims of support are worthless; the list is the evidence, and vendors without one do not have the capability.
