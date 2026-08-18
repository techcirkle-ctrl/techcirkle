---
layout: post
title: "An API Gateway Evaluation Checklist"
date: 2026-08-18 00:00:00 +0000
categories: ["Architecture", "Evaluation", "DevOps"]
tags: ["api", "checklist", "architecture", "devops"]
description: "A runnable checklist for evaluating API management tools — consumer taxonomy, tier decision, trial tests, agent traffic simulation and pricing stress."
image: "https://cdn.sanity.io/images/563mnkns/production/96b7be2e5d0e8bf1f1474d6d5d12c86d1fef1837-1600x802.jpg"
author: "TechCirkle Editorial Team"
---

![API integration and performance data on a display](https://cdn.sanity.io/images/563mnkns/production/96b7be2e5d0e8bf1f1474d6d5d12c86d1fef1837-1600x802.jpg)

Feature matrices in this category stopped being informative years ago — the products reached parity on the commodity capabilities and the vendor-supplied comparison table is designed to surface the one column where they differ.

This checklist skips that. Run it over about two weeks.

## 1. Consumer taxonomy (do this first)

Everything else depends on it.

```
consumer type          count   req/day   growth 12mo   notes
─────────────────────────────────────────────────────────────
internal services      ____    ______    ______        
mobile / web clients   ____    ______    ______        
partner integrations   ____    ______    ______        contracts?
public developers      ____    ______    ______        self-serve?
AI agents / MCP        ____    ______    ______        ← fastest growing
batch / ETL            ____    ______    ______        
```

The "public developers, self-serve" row is the one that decides tier. If it is empty and plausibly stays empty for two years, you need a gateway.

## 2. Tier decision

```
[ ] GATEWAY   — traffic control, auth, telemetry for a KNOWN consumer set
                (the substantial majority of organisations)
[ ] PLATFORM  — external devs self-serve credentials, usage-based billing,
                published docs kept in sync, consumer lifecycle/tiering

Trap: buying PLATFORM for an API business you might become.
      Forecast usually wrong; when right, timing off by years.
      Migration is feasible → buy for TODAY.
```

## 3. Capabilities that actually discriminate

Commodity (all products do these — do not spend evaluation time here): basic routing, API key auth, JWT validation, request/response logging, simple per-key rate limits.

Discriminating:

```
[ ] Cost-weighted rate limiting
    └ per-route weights, not just request counts
    └ TEST IT — support varies enormously
[ ] Delegated authorisation
    └ token representing user + agent, not one principal
    └ policy can reference BOTH identities
    └ rate limit / quota / billing attributable to DIFFERENT parties
[ ] Configuration as code, in version control
    └ console-primary tools accumulate undocumented state within a year
[ ] Trace context propagation, unmodified, end to end
    └ gateways that terminate or rewrite trace IDs break tracing
      at exactly the boundary where you need it
[ ] Telemetry export to YOUR existing stack
    └ two dashboards = every incident starts with "which one?"
[ ] Added latency under YOUR load
    └ datasheet numbers measured under conditions unlike yours
```

## 4. Trial tests (not slides)

![AI systems connected to multiple API modules](https://cdn.sanity.io/images/563mnkns/production/7a6b6ea97a06671061ec967e8b03cf1f58dbba8c-1600x883.jpg)

```
AUTH — write your three hardest scenarios, make the vendor
       build them in a trial env with readable config:
       1. ________________________________
       2. ________________________________
       3. ________________________________
       [ ] built and working, not roadmapped
       [ ] configuration is legible to our team

LATENCY
       [ ] p50 / p95 / p99 added latency measured BY US
       [ ] measured at expected PEAK, not average
       [ ] measured with our payload sizes

AGENT TRAFFIC SIMULATION          ← most skipped, most predictive
       [ ] burst: 200 req in 10s, then idle 5 min, repeat
       [ ] retry storm: same call failing, aggressive retry, no backoff
       [ ] fan-out: one logical action → 40 dependent calls
       [ ] endpoint concentration: 90% of traffic on the
           MOST EXPENSIVE route
       [ ] schema/discovery endpoints hit at high rate
       → does weighted limiting hold? do errors stay descriptive?

TRACING
       [ ] trace ID survives gateway unchanged
       [ ] span shows in our APM, correlated with downstream service

FAILURE MODES
       [ ] upstream failure distinguishable from gateway failure
           in logs and metrics  ← conflating these wastes incident hours
       [ ] behaviour when the control plane is unreachable
       [ ] config rollback time
```

## 5. Pricing stress

```
[ ] Model at 1x current volume        ______
[ ] Model at 3x current volume        ______   ← the real number
[ ] Behaviour on burst / spike:       ________________
[ ] Overage terms:                    ________________
[ ] Does agent traffic count as       Y / N
    normal requests?
```

Agent-driven growth is genuinely hard to forecast. Per-request pricing that looks reasonable today can become the dominant line item after one successful year.

## 6. Deployment topology (decide deliberately)

```
[ ] Centralised     — simple, one policy point;
                      single failure point + config queue
[ ] Distributed     — per service/team, central control plane;
                      smaller blast radius, more drift
[ ] Hybrid          — central edge for external + cross-cutting,
                      service-level for fine-grained  ← common, sensible
```

Retrofitting a topology change after a year of accumulated configuration is a substantial project. Choose now.

## 7. Portability guard

```
[ ] Plugin register maintained from day one
    └ what / why-at-edge / owner / removal date
[ ] Review rule: every gateway-logic change answers
    "why is this not in the service?"
[ ] Shim removal tickets created WITH the shim, dated
[ ] Business logic count target: zero
    └ routes + standard policies migrate in weeks
    └ 20 custom plugins = a substantial project
```

## 8. Decision record

```
[ ] Chosen:            ____________________
[ ] Tier:              gateway / platform
[ ] Primary reason:    ____________________
[ ] Rejected + why:    ____________________
[ ] Revisit when:
    [ ] agent traffic exceeds ___% of total
    [ ] public self-serve developers become a real requirement
    [ ] annual spend exceeds ______ (makes terms negotiable)
    [ ] custom plugin count exceeds ___   ← portability alarm
```

Full guide behind this checklist: [API Management Tools in 2026](https://techcirkle.com/blog/api-management-tools). We also [run these evaluations](https://techcirkle.com/contact-us) with teams mid-selection.

## Frequently Asked Questions

### Why start with a consumer taxonomy?

Because it decides the tier, which is the highest-leverage choice in the whole evaluation. If there is no genuine self-serve external developer population now or within two years, you need a gateway and the platform features are cost without benefit.

### What should I not spend evaluation time on?

Commodity capabilities — basic routing, API key auth, JWT validation, logging and simple per-key rate limits. Every product does these adequately, and vendor comparison tables are constructed to highlight the one column where they differ.

### Why simulate agent traffic specifically?

Because it is the traffic pattern most likely to break assumptions: bursty, retry-heavy, fanning out, and concentrating on your most expensive endpoint. A smooth synthetic load test will not reveal whether weighted limiting actually holds.

### What is the trace context test for?

Some gateways terminate or rewrite trace identifiers, which breaks distributed tracing precisely at the boundary you most need to debug across. It is a small detail with disproportionate incident cost and it rarely appears in evaluations.

### Why model pricing at three times volume?

Because agent-mediated growth is hard to forecast and per-request pricing that looks reasonable now can dominate your bill after one good year. Ask specifically how the model behaves on bursts and what overage costs.

### What is the plugin count alarm for?

Portability. Routes and standard policies migrate in weeks; bespoke plugins turn that into a major project. Tracking the count makes the loss of optionality visible while it can still be reversed.
