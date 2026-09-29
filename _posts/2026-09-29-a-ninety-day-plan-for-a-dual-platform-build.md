---
layout: post
title: "A Ninety-Day Plan for a Dual-Platform Build"
date: 2026-09-29 00:00:00 +0000
categories: ["Engineering", "Architecture"]
tags: ["engineering", "architecture", "project-planning", "mobile"]
description: "What the first three months of a dual-platform build should produce, week by week, and the single question that tells you if it is on track."
image: "https://cdn.sanity.io/images/563mnkns/production/c8efcf7dd718fa5c8a8ce8b1656ff87c1fdb036a-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

![A product design and engineering team reviewing mobile app layouts and website page designs together](https://cdn.sanity.io/images/563mnkns/production/c8efcf7dd718fa5c8a8ce8b1656ff87c1fdb036a-1600x1066.jpg)

A proposal that opens with a twelve-month Gantt chart is a proposal written by someone who has not finished one of these. Twelve-month plans are fiction by month three; everyone involved knows it and nobody says so.

What a competent team proposes instead is a short, falsifiable first phase. Here is what that looks like, and what each stage should actually produce.

## Weeks 1–2: model the domain, not the screens

No wireframes yet. The output of these two weeks is a typed model of your entities and their state transitions, written down, reviewed by someone on your side who understands the business well enough to say "no, a cancelled booking can still be refunded within 48 hours."

That review is the entire point. Domain modelling forces unresolved product questions into the open, which is uncomfortable in week two and catastrophic in month nine.

**Checkpoint:** can the team describe your product's core state machine without notes? If not, stop. They will not be able to build it later.

## Weeks 3–6: one vertical slice, both surfaces

One real workflow — emphatically not a login screen — implemented end to end on web and mobile, against real persistence and real auth, deployed to a real environment.

This is the only honest test of whether a team can do both surfaces. It is also where the seam problems surface while they are still cheap to fix.

Specific things to look at when it lands:

- **Where the business rule lives.** It should exist in exactly one typed package, imported by both clients, with tests that run without a browser or simulator. If it appears in a web component and again in a mobile screen, that is the finding.
- **The cold-start path.** If opening the app needs four sequential calls before anything renders, the API was designed for a browser and the mobile client will work around it forever.
- **Behaviour on a bad network.** Turn off wifi mid-action. A team that has shipped mobile has thought about this; one that has not produces a spinner that never resolves, or a duplicate write.

**Checkpoint:** can someone uninvolved in building it complete the workflow on both surfaces?

## Weeks 7–12: breadth and the unglamorous half

The temptation here is to add features, because features demo well. The correct use of these six weeks is mostly infrastructure:

- Authentication done properly, including token handling on mobile and a revocable refresh path for a lost device.
- Error states across both surfaces — not just the happy path plus a generic failure screen.
- The observability spine: one event taxonomy, defined in the domain package as typed constants, imported by both clients so names cannot drift.
- CI for both surfaces, including a repeatable mobile build and a release process for both stores that someone other than the original author has executed.

Teams that defer all of this to the end are the teams whose projects slip in month eight, and they slip in a specific way: the features are done and nothing can be safely released.

**Checkpoint:** can a new engineer clone the repository and produce a working build of both surfaces on their first day?

## Day 90: the one question

Can you see real usage data from both surfaces in a single dashboard, queryable together?

If yes, the engagement is on rails. The architecture is genuinely shared, the analytics are genuinely unified, and you can make product decisions on evidence rather than instinct.

If no, no amount of screen count compensates, because you have two products that share a logo.

## Where AI assistance fits in this plan

It compresses weeks 3–6 substantially and barely touches weeks 1–2, and that asymmetry is worth understanding.

Deriving the second surface from a well-specified first one is largely mechanical: generating typed clients from a schema, porting validated forms, producing the test matrix for a state machine that already exists. Assistance is genuinely strong at all of it.

Deciding what the system means is not mechanical and does not compress. If anything the modelling phase deserves more time now, not less, because everything downstream amplifies whatever structure it is pointed at. Aimed at a clean core, assistance produces consistent second-surface code fast. Aimed at scattered rules, it scatters them further at speed, in code that compiles and passes review because the volume exceeds what anyone is actually reading.

The practical safeguard: a review gate that asks where the logic landed rather than whether the code works, enforced mechanically with lint rules rather than reviewer attention.

## What you should hold at day 90

- A domain package with tests, in your repository, that another team could pick up.
- An API contract a third client could be built against without new endpoints.
- Two client codebases with no business rules in them.
- Infrastructure as code and a deployment pipeline someone else has run.
- An event taxonomy with a named owner.

All of it in your accounts from the first commit, not transferred at project close. Ownership from day one is what keeps a partnership honest.

The full guide — including the shared-core architecture and the questions that expose a weak vendor before you sign — is here: [web and mobile app development company](https://techcirkle.com/blog/web-and-mobile-app-development-company).

## Frequently Asked Questions

**Is ninety days enough to see if a partner is working out?**
Comfortably, if the phases are structured this way. It is not enough if the first ninety days are discovery documents and wireframes, which is why the vertical slice is placed where it is.

**What if the team wants longer for discovery?**
Ask what running software the extra time produces. Discovery that produces no code is a way to bill three months before demonstrating competence.

**Should the mobile app be in the app stores by day 90?**
Submitted to a testing track, yes. Publicly released, usually not — but the release process should have been exercised end to end by someone other than whoever built it.

**What if the vertical slice reveals problems?**
That is the plan working. The question is whether the team named the problems before you did.

**How many people does this need?**
Fewer than most proposals suggest. One architect owning the domain package with real veto power, two to four cross-capable engineers with at least one who genuinely understands native behaviour, a designer who has worked on both surfaces, and a quality engineer from week one.

**What is the most common failure at day 90?**
Features complete, infrastructure absent. It looks like success in a demo and becomes a two-month delay the moment anyone tries to release safely.
