---
layout: post
title: "App Discovery Engineering Checklist With Acceptance Criteria"
date: 2026-11-15 00:00:00 +0000
categories: ["Engineering", "Mobile Development"]
tags: ["appdiscovery", "mobile", "deeplinks", "checklist", "engineering"]
description: "An app discovery engineering checklist in five requirement groups: stability, deep links, declared actions, web layer and analytics, with criteria."
image: "https://cdn.sanity.io/images/563mnkns/production/61b9184290f92dfb82b2d21a63e3ee46ba185fda-1600x1067.jpg"
author: "TechCirkle"
---

This document is a requirements checklist for making a mobile app discoverable. It is written for the engineers who will implement it. Each item can be copied into a backlog as a ticket.

![Hands holding a smartphone with colourful app icons floating above the display](https://cdn.sanity.io/images/563mnkns/production/61b9184290f92dfb82b2d21a63e3ee46ba185fda-1600x1067.jpg)

## Scope

App discovery is the path from a person having a need to that person opening the app for the first time. The systems on that path (store ranking, operating system search, web search, AI assistants) read signals that the build produces. Stability and link architecture cannot be added in an afternoon, so they belong in the requirements from sprint one, alongside security and accessibility.

This checklist covers five requirement groups:

1. Stability
2. Deep links
3. Declared actions
4. Web layer
5. Analytics

Out of scope: listing copy, keyword research, screenshots and paid campaigns. Those are covered in the [full app discovery guide](https://techcirkle.com/blog/app-discovery).

## Group 1: Stability

**Requirement.** The app stays under the stores' bad-behaviour thresholds on the devices its users own.

**Why.** Android vitals defines thresholds of 1.09 percent for user-perceived crash rate and 0.47 percent for user-perceived ANR rate, measured against daily active users. Apps above them can be made less discoverable on Google Play and may carry a warning on the listing. Per-device thresholds also apply.

**Acceptance criteria.**

- Crash reporting is integrated in the first build, not added before launch.
- User-perceived crash rate is below 1.09 percent overall.
- User-perceived ANR rate is below 0.47 percent overall.
- Crash and ANR rates are reported per device model, and no high-traffic model is over threshold.
- Releases use staged rollouts with an automatic halt when stability regresses.
- A performance budget exists for cold start and screen rendering, is documented, and is checked in the release process.
- The test device set includes the low-end devices that appear in the app's own usage data.
- Download size is tracked per release.

## Group 2: Deep links

**Requirement.** Every screen a user might share or search for is reachable by a standard web URL.

**Why.** Without deep links, a shared link, a marketing email and a search result all open the home screen or a web page.

**Acceptance criteria.**

- A deep link map lists every shareable screen and its URL pattern.
- Universal Links are configured on iOS and App Links on Android.
- Each URL opens the matching screen when the app is installed.
- Each URL falls back to the equivalent web page when the app is not installed.
- Deferred deep linking is in place: a user who installs from a link lands on that content at first launch, not on a generic welcome screen.
- New screens cannot ship without an entry in the deep link map.

**Estimate.** For a mid-complexity app, deep linking with deferred links is roughly one to two weeks of engineering.

## Group 3: Declared actions

**Requirement.** The app describes its main actions and entities to the operating system in a structured form.

**Why.** On iOS, the App Intents framework exposes declared actions in Spotlight, Siri, Shortcuts, widgets and the action button. It is also the mechanism through which Apple Intelligence can act inside the app. Android offers comparable hooks through app shortcuts and its newer app functions APIs for Gemini. Assistants pick apps that have declared a capability and skip those that have not.

**Acceptance criteria.**

- The top actions are declared as App Intents on iOS, for example start a workout or reorder the last purchase.
- Equivalent app shortcuts are defined on Android.
- Key entities, such as a project or a saved item, are declared so they can be opened directly.
- User content suited to system search is contributed to on-device indexing on both platforms.
- Each declared action has been run from system search and from voice, and it completes without the user opening the app manually.
- The action catalogue is kept in one place so an in-app assistant can reuse it. This is the same structure used in [LLM integration](https://techcirkle.com/llm-integration) work.

**Estimate.** A useful first set of intents and shortcuts is roughly one to two weeks.

## Group 4: Web layer

**Requirement.** Every piece of public content has an indexable web page that connects to the app.

**Why.** Web search is far larger than store search, and web pages are what AI assistants read. Content that exists only in the binary is invisible to both.

**Acceptance criteria.**

- Public content types (listings, profiles, articles, product pages, templates) are rendered on the server.
- Each page carries a smart banner or install prompt.
- Each page includes structured data that describes the app.
- Each page links to the same content in the app through the deep links from Group 2.
- The web layer and the mobile app share a backend and content model.
- Plain-language pages exist that state what the app does, who it is for, what it costs and how it compares with alternatives.

**Estimate.** Two weeks to two months, depending on content volume. See [web app development](https://techcirkle.com/development/web-app-development).

## Group 5: Analytics

**Requirement.** Installs and first sessions are attributed to their source, and discovery has its own dashboard.

**Why.** Without source-level data the team cannot tell which channel is working.

**Acceptance criteria.**

- Impressions and install conversion are reported by source: search, browse, web referrer and campaign.
- Organic share of installs is tracked over time.
- Deep link open rate and first-session destination are recorded.
- Crash and ANR rates are shown against store thresholds, overall and per device model.
- Day 1, day 7 and day 30 retention are reported by source.
- Keyword rankings are tracked per country for a focused list of terms.
- A monthly manual check records what the main AI assistants answer for the app's category and which sources they cite.

## Definition of done

A build is discovery-ready when all five groups pass. Planned from the start, this work typically adds 8 to 15 percent to the initial budget. Retrofitting it into a live app costs more.

## Frequently Asked Questions

### Which requirement group should a team implement first?

Stability and analytics. Analytics tells you where installs come from, and stability keeps the app under store thresholds.

### What are the Android vitals thresholds this checklist refers to?

A user-perceived crash rate of 1.09 percent and a user-perceived ANR rate of 0.47 percent of daily active users. Apps above them can be made less discoverable on Google Play.

### Do App Intents replace deep links?

No. Deep links map web URLs to screens. App Intents declare actions and entities to the operating system. They solve different problems and a discoverable app needs both.

### How long does the full checklist take to implement?

For a mid-complexity app, deep links take one to two weeks, declared actions another one to two weeks, and the web layer two weeks to two months. Stability tooling is mostly process.

### Can this checklist be applied to an app that is already live?

Yes, but it costs more than building it in. A common order is measurement and stability first, then deep links and declared actions, then the web layer.
