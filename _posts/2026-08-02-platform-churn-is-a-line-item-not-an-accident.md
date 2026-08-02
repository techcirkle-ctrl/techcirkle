---
layout: post
title: "Platform Churn Is a Line Item, Not an Accident"
date: 2026-08-02 00:00:00 +0000
categories: ["Mobile", "Engineering"]
tags: ["ios", "android", "maintenance", "devops"]
description: "Apple and Google ship breaking changes yearly. What actually breaks, on what cadence, and why 15-22% of build cost per year is the real floor."
author: "TechCirkle Editorial Team"
---

Every mobile budget I review has a build number and no run number. The build number was negotiated carefully over several weeks. The run number is either absent or a placeholder someone wrote as "10% for maintenance" because that sounded conservative.

It is not conservative. The realistic floor is **15–22% of build cost per year**, before a single new feature, and the reason is not bit rot. It is that two platform vendors ship deliberate, published, non-optional changes on an annual cadence, and your app is downstream of both.

An app with no maintenance budget has a scheduled outage. The date is already on Apple's and Google's calendars; nobody has just told you.

## What actually breaks, and when

The recurring categories, roughly in order of how often they bite:

**Target SDK deadlines.** Google Play enforces a minimum target API level for updates, and the bar rises annually. Miss it and you cannot ship an update — including a critical bug fix — until you have done the migration. This is the most common way a "dormant" app becomes an emergency: it was fine until the day you needed to change something.

**Xcode and OS build requirements.** App Store submissions periodically require a minimum Xcode version, which in turn requires a minimum macOS version, which requires CI runner updates. A three-line change can become a two-week toolchain migration if you have skipped a cycle.

**Privacy and permission changes.** Privacy manifests and declared reasons for certain APIs, tracking permission flows, photo and location permission granularity, notification permission timing. These change behaviour for existing users, not just new ones, and they frequently invalidate an onboarding flow you spent real design time on.

**Background execution rules.** Both platforms keep tightening what an app can do when not foregrounded. Background sync, location, and long-running tasks are the usual casualties, and the failure is silent — your app does not crash, it just stops doing the thing your users depend on.

**Push and notification behaviour.** Delivery guarantees, grouping, permission prompt rules and channel requirements shift periodically. Notification-dependent products feel this as an unexplained engagement drop weeks before anyone traces it.

**Dependency chains.** Your three direct dependencies have forty transitive ones. Security advisories arrive on their own schedule. A dependency that stops being maintained is a migration you did not plan, on a timeline you do not control.

## The compounding problem

Skipping a year does not defer the cost. It multiplies it.

Platform migrations are designed assuming you are one version behind, with documented paths and deprecation warnings. Two or three versions behind, the guided paths no longer apply, your dependencies have made their own incompatible jumps, and you are doing archaeology to reconstruct why a workaround exists in code nobody has touched in eighteen months.

The teams that spend the least on maintenance are the ones that spend a little continuously — a scheduled maintenance sprint each quarter, dependency updates in small batches, and beta-cycle testing against the next OS before it ships. The teams that spend the most are the ones that spent nothing for two years and then paid for a rewrite they described as "unavoidable technical debt."

## What a run budget should actually cover

- Quarterly dependency and security updates, batched and tested.
- Testing against iOS and Android betas during the developer preview cycle, not after public release.
- Two planned migration windows a year aligned to the platform release calendar.
- Crash and ANR triage with a defined response target rather than a best-effort promise.
- Store metadata upkeep — screenshots at new device sizes, policy questionnaire updates, and in a Canadian context, French versions of everything.
- Certificate and provisioning profile renewals, with calendar reminders owned by someone specific. Expired certificates are a completely preventable outage that still happens constantly.

## What to ask a development partner

Three questions surface the truth quickly:

1. **Show me your platform-update calendar for the next twelve months.** Firms that maintain apps have one. Firms that build and disappear look surprised by the question.
2. **What's your policy on testing against OS betas?** "We wait for the public release" means your users find the breakage first.
3. **Who owns certificate renewal, and what happens when it expires on a Saturday?** If the answer has no name in it, it has no owner.

And structurally: put the run cost in the same conversation as the build cost, before you sign. A firm that quotes a build in isolation and treats maintenance as a future discussion is optimizing for the number you compare, not the cost you carry.

The full buyer's guide — Canadian cost bands, delivery models, privacy architecture and a 30-day selection process — is here: **[Mobile App Development Company Canada: The 2026 Buyer's Guide](https://techcirkle.com/blog/mobile-app-development-company-canada)**. Our [mobile app development](https://techcirkle.com/development/mobile-app-development) page covers how we structure ongoing delivery.

## Frequently Asked Questions

### How much does mobile app maintenance cost per year?

Budget 15–22% of build cost annually before any new features. That covers platform migrations, dependency and security updates, store metadata upkeep, crash triage and certificate renewals — all of which arrive on schedules you do not control.

### What happens if I skip a year of maintenance?

The cost multiplies rather than defers. Platform migration paths assume you are one version behind; two or three behind, documented paths no longer apply, dependencies have made incompatible jumps, and the work becomes archaeology.

### Why can't I just leave a working app alone?

Because Google Play enforces rising target API levels for updates and App Store submissions periodically require newer toolchains. The app keeps running until the day you need to ship a fix — and then the fix is blocked behind a migration.

### What breaks most often after an OS update?

Background execution, permissions and notification behaviour. These fail silently: the app does not crash, it just stops doing something users relied on, which is usually discovered as an unexplained engagement drop.

### Should we test against OS betas?

Yes. Testing during the developer preview cycle is how you find breakage before your users do. A partner who waits for public release has outsourced discovery to your customers.

### Who should own certificate and provisioning renewals?

A named person with calendar reminders, on your side or your vendor's, agreed in writing. Expired certificates cause entirely preventable outages and remain one of the most common self-inflicted production incidents in mobile.
