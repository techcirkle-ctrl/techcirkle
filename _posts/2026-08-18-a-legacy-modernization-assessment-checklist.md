---
layout: post
title: "A Legacy Modernization Assessment Checklist"
date: 2026-08-18 00:00:00 +0000
categories: ["Architecture", "Assessment", "Documentation"]
tags: ["checklist", "legacy", "architecture", "assessment"]
description: "A runnable checklist for assessing a legacy system: change-cost diagnostics, path selection per component, seam criteria and cost workstreams."
image: "https://cdn.sanity.io/images/563mnkns/production/95759876ffd43ca6ebe4692e4daad791a5b0ec76-1600x900.jpg"
author: "TechCirkle Editorial Team"
---

![Legacy computing systems shown with modern digital technology](https://cdn.sanity.io/images/563mnkns/production/95759876ffd43ca6ebe4692e4daad791a5b0ec76-1600x900.jpg)

Most modernization proposals begin with a target architecture. This checklist begins with a diagnosis, because the four common causes of "our system is legacy" have four different remedies and only one of them involves writing new code.

Run it before committing budget.

## 1. Change-cost diagnostic

```
[ ] Time from request to production for a SMALL change:  ______
[ ] % of changes causing unrelated breakage:             ______
[ ] Number of people who can safely modify the core:     ______
[ ] Can a new engineer build a working env unaided?      Y / N
[ ] Time for a new engineer to first merged PR:          ______
```

Interpretation — this is the part that determines what you actually fix:

```
slow changes + clean releases      → code structure problem
slow changes + manual release      → PROCESS problem; new code won't help
frequent unrelated breakage        → test coverage problem
only 1-2 people can modify         → knowledge distribution problem
cannot build env from scratch      → environment/config problem
```

A system failing on process, tests or knowledge does not need a rewrite. Rewriting it produces a modern system with the same two-week release cycle, the same missing tests, or the same single point of human failure.

## 2. Cost of the status quo

Do not lead a business case with project cost — it is the only number that looks like a loss. Quantify these instead, with real figures:

```
[ ] Roadmap items delayed >2x or dropped in planning last year:  ______
[ ] Roles open >90 days attributable to this stack:              ______
[ ] Engineers who can safely operate the system:                 ______
    └ of whom retiring / flight-risk within 12 months:           ______
[ ] Engineering days spent on incident diagnosis last year:      ______
[ ] Deferred compliance findings tied to this system:            ______
[ ] Partnerships / integrations declined as infeasible:          ______
```

Two or three of these with credible numbers beat any architectural argument in a budget meeting.

## 3. Per-component path selection

Force an explicit decision for each component. Estates get one blanket decision applied to everything, which is how money gets spent on components that were fine.

```
component: ______________________
  [ ] Retain    — stable, low change rate; document WHY
  [ ] Retire    — check actual usage first; more common than expected
  [ ] Rehost    — infra exit / EOL platform driver only
  [ ] Replatform— managed DB, container, runtime upgrade  ← under-used
  [ ] Refactor  — needs test coverage FIRST
  [ ] Rearchitect — structural change; high cost, high risk
  [ ] Rebuild   — see gate below
```

Rebuild gate — all three must be true:

```
[ ] Original technology has no viable forward path
    (unsupported/unhireable — NOT merely unfashionable)
[ ] System is small enough to fully understand in ~1 week
[ ] Business requirements changed enough that existing
    behaviour is a liability rather than an asset
```

If not all three, incremental replacement is cheaper and safer.

## 4. Seam criteria (before extracting anything)

![Technology transformation and consulting work](https://cdn.sanity.io/images/563mnkns/production/410088051d5bbe9316a8ba315c011e87a2a7c7da-1600x598.jpg)

```
[ ] Boundary already exists in the business domain
    (not invented by engineering for code convenience)
[ ] New component can own its data
    └ OR reads consistently without distributed transactions
[ ] NOT two systems writing the same tables
[ ] Routable at a layer we control (gateway / proxy / flag)
[ ] Extraction will NOT convert frequent in-process calls
    into network calls          ← distributed monolith check
[ ] Rollback achievable in minutes
[ ] Telemetry shows which implementation served a request
```

End-state planning — the step that prevents the seventy-per-cent stall:

```
[ ] Written down: which component migrates LAST
[ ] Estimated: what that final component will cost
[ ] Confirmed: org will fund it once acute pain is relieved
    └ if NO → you are building a permanent hybrid.
              Choose that deliberately, or stop.
```

## 5. Behaviour capture

```
[ ] Characterisation tests generated (AI-assisted OK)
[ ] Every test RUN AGAINST LEGACY first        ← non-negotiable
    └ pass = behaviour confirmed
    └ fail = the test was wrong, fix it
[ ] Surprising passes flagged for senior review
    (asserts something wrong + passes = undocumented
     behaviour something downstream depends on)
[ ] Production traffic replay configured
    [ ] capture window spans a FULL business cycle
        (month-end, quarter-end, annual paths)
    [ ] timestamps / generated IDs normalised before diff
    [ ] sensitive fields scrubbed AT CAPTURE
    [ ] read paths first; write paths shadowed separately
```

## 6. Data workstream (starts before application work)

```
[ ] Profiled ACTUAL data, not just the schema
    [ ] value distributions per column
    [ ] orphaned references
    [ ] impossible date ranges
    [ ] sentinel values standing in for null
    [ ] columns whose meaning changed at a past migration
[ ] Dual-write + continuous comparison planned
[ ] Cutover gate = discrepancy rate ZERO, not "nearly zero"
```

## 7. Cost model — four workstreams, estimated separately

```
[ ] Discovery & documentation      ______
    └ re-estimate if written pre-2024; AI compressed this
      line substantially (historically 40-60% of programme)
[ ] Implementation                 ______
[ ] Data migration & verification  ______
[ ] Cutover & parallel running     ______
    └ 2x infra, 2x on-call, reconciliation effort,
      for N months   ← most commonly missing line
[ ] Discovery allowance (NOT flat contingency)
    unknown findings are certainties, not risks
```

## 8. Sequencing

```
[ ] First delivery is painful + bounded + visible OUTSIDE engineering
    (NOT the worst component — it shows nothing for months
     and invites cancellation at the next budget review)
[ ] Subsequent order follows the dependency graph
[ ] Dependency graph includes UNDOCUMENTED coupling
    └ two apps writing the same table
    └ scheduled jobs on unmanaged hosts
    └ reports reading the database directly
```

## 9. Success criteria — define before starting

```
[ ] Baseline captured NOW (skipped constantly; fatal later)
[ ] Metric tied to the ORIGINAL driver:
      velocity    → lead time to production
      reliability → change failure rate, MTTR
      hiring      → time-to-fill, time-to-first-contribution
      cost        → TCO incl. operational effort
[ ] Qualitative gate: can a new engineer ship a small
    production change in their FIRST WEEK?
```

Full narrative framework behind this checklist: [Legacy Application Modernization](https://techcirkle.com/blog/legacy-application-modernization). We also [run these assessments](https://techcirkle.com/contact-us) for teams wanting an outside read before committing budget.

## Frequently Asked Questions

### What should the assessment produce?

A diagnosis, a per-component path decision, a seam plan with a written end-state, a data profile, and a four-workstream cost model with a captured baseline. A target architecture diagram alone is not an assessment.

### Why estimate four workstreams separately?

Because single-number estimates anchor on implementation, which is visualisable and rarely the largest. Discovery has historically been forty to sixty per cent, and parallel running is the line most often omitted entirely.

### What is the rebuild gate for?

To force all three conditions to be true before a rewrite is approved — no viable forward path for the technology, a system small enough to fully understand, and requirements changed enough that existing behaviour is a liability. Rewrites get chosen for enthusiasm otherwise.

### Why must characterisation tests run against the legacy system first?

An unvalidated test is an assumption with a framework around it. Running it against the original converts belief into evidence, and a test that asserts something apparently wrong yet passes has found undocumented behaviour worth senior review.

### Why profile the data rather than read the schema?

Because the gap between what the schema permits and what the data contains is your real specification — sentinel values, orphaned references, columns whose meaning changed at a past migration. None of that appears in a DDL dump.

### What if the diagnostic points at process rather than code?

Then fix the process first. A system whose changes are slow because of a two-week manual release cycle will still have a two-week release cycle after a rewrite, and you will have spent the budget without moving the metric.
