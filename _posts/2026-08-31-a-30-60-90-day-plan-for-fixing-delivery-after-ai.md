---
layout: post
title: "A 30-60-90 Day Plan for Fixing Delivery After AI"
date: 2026-08-31 00:00:00 +0000
categories: ["DevOps", "Engineering"]
tags: ["devops", "platform-engineering", "cicd", "engineering-management"]
description: "A concrete ninety-day sequence for engineering leaders whose delivery slowed after AI-assisted development increased change volume."
image: "https://cdn.sanity.io/images/563mnkns/production/bbc14dd12c94a565b899b4dd2f2f9b3582ea83ee-1600x598.jpg"
author: "TechCirkle"
---

![Software delivery monitoring](https://cdn.sanity.io/images/563mnkns/production/bbc14dd12c94a565b899b4dd2f2f9b3582ea83ee-1600x598.jpg)

The ordering of this plan is the part that matters. Most delivery improvement programmes fail because they start with architecture, which is expensive, slow to show value, and impossible to justify without a baseline. This sequence starts with measurement and quick mechanical wins so that the structural work is funded by results rather than by faith.

The context: AI-assisted development increased the volume of change flowing into delivery systems without increasing the capacity of anything downstream of writing code. Review, testing, building, releasing and recovering all scale with change volume, and none of them were resized.

## Days 1-30: measure, then take the mechanical wins

**Instrument the baseline.** Before changing anything, capture deployment frequency, lead time from merge to production deploy, change failure rate, and time to restore. Two notes on measurement. Lead time from ticket creation is the wrong number — it is dominated by prioritisation and it flatters you when authoring gets faster. Change failure rate should count rollbacks recorded by the deployment system, not only incidents someone declared, because fast silent rollbacks are otherwise invisible.

**Capture CI economics.** Cost per merge and duration per merge, as trends. Most organisations have never looked at these and are surprised by both.

**Instrument the queue.** Open pull request count, time from PR opened to first review, time from first review to merge, and age of the oldest open PR. Splitting the two time measures is essential — if time-to-first-review dominates, your constraint is human capacity and no pipeline work will address it.

**Then fix the mechanical inefficiencies.** In descending order of typical return: run only the tests a change affects rather than the whole suite; verify your dependency cache is genuinely faster than a cold build, which is not always true; revisit runner sizing that was chosen during a debugging session and never changed; prune matrix builds to combinations customers actually run; schedule non-production environments to shut down outside working hours.

This phase should pay for itself. That matters, because it is what funds the next two.

## Days 31-60: build exactly one paved path

**Pick one service shape.** The most common type of service you actually build, representing a meaningful slice of the portfolio. Not the most interesting one — the most common one.

**Build the path end to end.** Template, self-service provisioning, pipeline, policy checks enforced at merge, observability defaults wired in, and a progressive rollout mechanism with automated rollback on service-level violation.

**Migrate two real services onto it.** Real ones, in production, with real owners who will complain if it is worse than what they had. Two is enough to find what is wrong and few enough to fix it.

**Resist expansion.** The strongest temptation at this stage is to generalise the path to cover every case before any case works. A platform that supports twelve service shapes at launch supports none of them well.

![Engineers collaborating on a change](https://cdn.sanity.io/images/563mnkns/production/330c54432ea871f17b3e82987fd4e830cf2cff82-1600x1066.jpg)

## Days 61-90: propagate and hand over ownership

**Move the next cohort.** With two services proven, the next five are mechanical. Track percentage of services on the path as the adoption metric — it is the honest measure of whether platform work is landing.

**Write the runbooks.** Not after, during. A runbook written by the person who built the thing, verified by someone who did not.

**Name internal owners.** Every system the work touched needs a named person inside your organisation who can operate it. If this is uncomfortable to fill in, that is the finding.

**Run a game day.** Exercise the incident process deliberately before an incident exercises it. Include the rollback path, because that is the mechanism the whole design depends on.

By day ninety you should be able to point at a metric that moved and at an internal engineer who can operate the system without external help. If you can do the first but not the second, the engagement produced a deliverable rather than a capability.

## The part that is not technical

Somewhere in the first sixty days you have to make an explicit decision about review capacity, because that is where the constraint now sits for most teams and no amount of infrastructure resolves it.

The options that work: split large changes automatically, since review effort scales worse than linearly with diff size; define explicitly which categories of change do not require two human reviewers; use model assistance for the mechanical review pass — style, error handling, missing tests — so human attention goes to design and correctness; and set a queue-length threshold that triggers a team conversation, so the constraint is visible rather than ambient.

Generated code arriving faster than humans can review it is a policy decision. Right now it is being made by queue length rather than by anyone in particular.

Full version, including the four business models behind the DevOps label, engagement structures, pricing, and vendor vetting: [DevOps Services Company: How to Choose the Right Partner in the AI Era](https://techcirkle.com/blog/devops-services-company). Architecture background in our [cloud application development guide](https://techcirkle.com/blog/cloud-application-development-guide).

## Frequently Asked Questions

### Why measure before improving anything?

Because a baseline reconstructed after the fact always flatters whoever did the work, and because the measurement itself usually reveals that the assumed bottleneck is not the real one. Two weeks of instrumentation regularly redirects the entire plan.

### What if the mechanical wins are not there?

Then your pipeline is already well maintained, which is good news, and you should move directly to the queue and the paved path. It also means the constraint is almost certainly review capacity.

### Is ninety days realistic for a large organisation?

For one paved path and one cohort of services, yes. For an entire portfolio, no. The plan is deliberately scoped to prove the model rather than to complete the migration.

### Who should own this work?

A named internal engineer with dedicated time, supported by outside help if needed. Work owned entirely by an external partner produces a deliverable that decays after they leave.

### What if leadership wants architecture first?

Show them the CI cost trend and the pull request queue. Both are concrete, both are cheap to fix, and both make the architecture conversation easier because it starts from evidence rather than from advocacy.

### How do we keep the paved path from rotting?

Give it an owner, a roadmap, and adoption metrics — treat it as a product with users. The percentage-of-services-on-path number is the early warning; when it stops rising, something about the path stopped being the fastest route.
