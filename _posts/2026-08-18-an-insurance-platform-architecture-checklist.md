---
layout: post
title: "An Insurance Platform Architecture Checklist"
date: 2026-08-18 00:00:00 +0000
categories: ["Architecture", "Insurance", "Checklists"]
tags: ["insurance", "architecture", "checklist", "compliance"]
description: "A runnable checklist for insurance platform work — build/buy per capability, bitemporal data, integration inventory, AI governance and cost workstreams."
image: "https://cdn.sanity.io/images/563mnkns/production/9831dfbf3d9742ba56c8eaa503218493610a7615-1600x549.jpg"
author: "TechCirkle Editorial Team"
---

![Insurance technology and analytics interface](https://cdn.sanity.io/images/563mnkns/production/9831dfbf3d9742ba56c8eaa503218493610a7615-1600x549.jpg)

Insurance programmes fail at predictable points. This checklist targets those points specifically. Run it before committing budget.

## 1. Build / buy, per capability (not per system)

```
capability              decision   rationale
──────────────────────────────────────────────────────────────
policy administration   [ ] BUY    process; industry-standard,
                                   30 yrs of edge cases
billing & finance       [ ] BUY    externally defined; no
                                   advantage available
regulatory reporting    [ ] BUY    fixed schemas, hard deadlines
underwriting & pricing  [ ] BUILD? ← your view of RISK
                                     build IF specialty book
distribution            [ ] ?      comparison platforms → buy
                                   direct / tied agents → build
claims workflow         [ ] BUY    process
claims intelligence     [ ] BUILD  ← where AI moves numbers
```

Heuristic: **build what encodes your view of risk, buy what encodes the industry's view of process.**

Pricing build gate:

```
[ ] We write a specialty book where our risk view differs
[ ] We have made a pricing change competitors did not,
    within the last 12 months
[ ] We can OPERATE what we build (not just deliver it)
    └ a bespoke engine one person understands is worse
      than a vendor's, which has a support desk
```

All three, or buy the engine.

## 2. Data model — the irreversible one

```
[ ] Bitemporal: valid time AND transaction time, independently
    [ ] valid_from / valid_to    (true in the world)
    [ ] tx_from   / tx_to        (what we believed, when)
[ ] Facts are NEVER updated in place
    └ close tx window, insert new row
    └ enforce at DB level; app discipline erodes
[ ] Backdated endorsement in test fixtures FROM DAY ONE
    └ the scenario that exposes every modelling mistake
[ ] Claims modelled as time series, not rows
    (development tails run years)
[ ] Reinsurance apportionment retained per treaty
[ ] Current-state VIEW exists so ordinary code is not
    littered with temporal predicates
```

**This section is not merely expensive to retrofit — it is frequently impossible.** An overwritten belief exists nowhere; backups restore snapshots, not queryable belief history.

Test: *can the system explain a decision made last year using only what it knew at the time?*

## 3. Integration inventory (enumerate BEFORE estimating)

![Insurance agent reviewing documentation with a client](https://cdn.sanity.io/images/563mnkns/production/c611e1ce2a6c1d8d8cfd60f61d24cade1d5d540e-1600x1066.jpg)

```
counterparty            contract  SLA  format  failure behaviour
────────────────────────────────────────────────────────────────
hazard / peril data     [ ]       [ ]  [ ]     [ ]
credit reference        [ ]       [ ]  [ ]     [ ]
vehicle / property db   [ ]       [ ]  [ ]     [ ]
sanctions / PEP screen  [ ]       [ ]  [ ]     [ ]
payments                [ ]       [ ]  [ ]     [ ]
premium finance         [ ]       [ ]  [ ]     [ ]
reinsurance bordereaux  [ ]       [ ]  [ ]     [ ]  ← always late
broker connections      [ ]       [ ]  [ ]     [ ]  (per broker!)
comparison platforms    [ ]       [ ]  [ ]     [ ]
regulatory returns      [ ]       [ ]  [ ]     [ ]
document generation     [ ]       [ ]  [ ]     [ ]
archival / retention    [ ]       [ ]  [ ]     [ ]
```

```
[ ] Estimated PER COUNTERPARTY, not per protocol
    (two brokers on one standard = two integrations)
[ ] Degradation behaviour agreed WITH AN UNDERWRITER
    for every dependency — decline? load? queue?
[ ] Counterparty-dependent work started in month 1
    (credentials, security reviews, certification slots
     have lead times you do not control)
[ ] One integration built end to end EARLY, incl. failure
    paths + reconciliation, to calibrate the rest
```

Budget ~50% of programme effort here. Plans allocating 20% slip by a year.

## 4. AI governance

```
Document extraction
[ ] Confidence score PER FIELD
[ ] Auto-accept threshold MEASURED against a labelled
    sample of OUR documents — not chosen by intuition,
    not taken from a vendor benchmark
[ ] Thresholds set PER FIELD TYPE
    (sum insured ≠ free-text description in error cost)
[ ] Re-validated quarterly (document mix drifts)
[ ] Source doc + page + region linked to every value,
    permanently
[ ] Model version logged with every extraction
[ ] Review action recorded: auto / confirmed / corrected
    └ corrected values retain what they were corrected FROM
[ ] Sample of AUTO-ACCEPTED items reviewed periodically
    └ only way to detect silent degradation

Underwriting assistance
[ ] Model SURFACES facts with citations; human DECIDES
[ ] Overriding a model score is NOT costly or unusual
    └ if it is, the model is deciding, whatever the
      governance doc says
[ ] Audit trail: inputs, model version, output, human action
[ ] Pricing METHODOLOGY not replaced by opaque model
    └ enrich inputs, don't replace the actuarial core
```

## 5. Core replacement (only if genuinely blocking)

```
[ ] Migration by RENEWAL COHORT, not big bang
    └ new business on new system
    └ existing book migrates as policies renew
    └ defect affects one cohort, not the portfolio
[ ] Accepted: some historical claims may stay read-only
    on the old system indefinitely (normal outcome)
[ ] Vendor estimate understood as CONFIGURATION ONLY
    └ excludes our integrations, data migration,
      process change — usually the majority of cost
```

## 6. Cost workstreams

```
[ ] Discovery / current-state analysis    ______
[ ] Configuration (vendor estimate)       ______
[ ] Integration (~50%)                    ______
[ ] Data migration & verification         ______
[ ] Parallel running                      ______  ← often missing
    2x infra, 2x ops, reconciliation, N months
[ ] Process change & training             ______
```

## 7. Sequencing

```
1. [ ] Intelligence layer over EXISTING systems
       (document extraction + triage) — months, no core change
2. [ ] Data layer with bitemporality
3. [ ] Worst operational workflow (claims intake / submissions)
4. [ ] Pricing tooling, IF the build gate passed
5. [ ] Core replacement, IF genuinely blocking, by cohort
```

Deliberately the reverse of how these get pitched — front-loads measurable value, defers highest-risk work until you have credibility and better operational data.

Full guide behind this checklist: [Insurance Software Solutions](https://techcirkle.com/blog/insurance-software-solutions). We also [run these assessments](https://techcirkle.com/contact-us).

## Frequently Asked Questions

### Why decide build/buy per capability rather than per system?

Because vendors sell core platforms as all-or-nothing while the correct answer differs across the six functional areas. Policy administration is process worth buying; pricing is often where your competitive position lives.

### What makes the data model section irreversible?

An overwritten belief exists nowhere. Backups restore snapshots rather than a queryable history of what was believed when, so a system without bitemporality cannot later answer what it knew on a past date.

### Why enumerate integrations before estimating?

Because the estimate is wrong mainly because the list is incomplete. Estimating per counterparty rather than per protocol, and building one end to end early, are what make the remaining numbers credible.

### How should auto-accept thresholds be set?

Measured against a labelled sample of your own documents, per field type, with tolerance reflecting the cost of an error in that field. Re-validate quarterly, because document mix drifts.

### When is a model effectively making underwriting decisions?

When overriding its output is costly or unusual within the workflow, regardless of what governance documentation states. Regulators assess the practical effect rather than the stated policy.

### Why sequence the core replacement last?

Because it carries the highest risk and longest payback, and the earlier steps generate the operational data and organisational credibility that make it survivable. Programmes delivering nothing until year three are routinely cancelled in year two.
