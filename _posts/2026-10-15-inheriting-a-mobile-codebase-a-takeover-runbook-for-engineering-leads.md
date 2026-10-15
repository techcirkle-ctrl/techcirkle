---
layout: post
title: "Inheriting a Mobile Codebase: A Takeover Runbook for Engineering Leads"
date: 2026-10-15 00:00:00 +0000
categories: ["Mobile Development", "Engineering Management"]
tags: ["mobile", "ios", "android", "devops"]
description: "A runbook for taking over a mobile app from another vendor: code and dependency audits, reproducible builds, secrets cleanup and a 30/60/90-day plan."
image: "https://cdn.sanity.io/images/563mnkns/production/ed6a33dc4581febc1dc156483d349d66b1425589-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

The handover call lasted forty minutes, the previous vendor sent a zip file and a password spreadsheet, and now the app is yours. Sound familiar? Taking over a mobile codebase is one of the most common ways a maintenance engagement begins, and one of the easiest to get wrong. The instinct is to start fixing bugs on day one. The better move is to spend the first weeks finding out what you actually own. This runbook is the template we follow; copy it into your own repo and adapt it.

![Person holding a smartphone that displays an app update and repair icon](https://cdn.sanity.io/images/563mnkns/production/ed6a33dc4581febc1dc156483d349d66b1425589-1600x1066.jpg)

## Phase 0: Secure ownership before touching code

Before any technical work, confirm that the business, not a vendor or an individual, controls every asset. Record each item in a table with the current owner and the target owner.

| Asset | Check |
| --- | --- |
| Source repositories | In your organization, with full history, not a squashed import |
| Apple Developer and App Store Connect | Account holder is your company; vendor has a revocable role |
| Google Play Console | Same; confirm Play App Signing status and upload key ownership |
| Signing assets | iOS certificates and profiles, Android upload keystore and passwords |
| Cloud consoles and backend | Billing account and admin roles under your company |
| Push, analytics, crash reporting | Projects owned by your accounts, not a vendor's workspace |
| Domains and deep-link files | `apple-app-site-association` and `assetlinks.json` hosting |

If the Android upload key is missing, Play App Signing lets you request an upload key reset, but it takes time. Start that process immediately.

## Phase 1: Build reproducibility

The first technical milestone is simple to state and often hard to reach: **a clean machine can build, sign, and run the app from the repository alone.**

- Clone onto a fresh machine or container. Record every undocumented step you needed.
- Pin toolchains: `.xcode-version`, Gradle wrapper, JDK version, Ruby and Bundler for Fastlane, Node for React Native or Flutter SDK versions for cross-platform apps.
- Commit lockfiles (`Podfile.lock`, `Package.resolved`, `gradle.lockfile` if used, `package-lock.json` or `pnpm-lock.yaml`).
- Move build steps into a script or Fastlane lanes so CI and humans run the same thing.
- Produce a release build and compare its behavior and size with the version live in the stores.

If the live version cannot be reproduced from source, note it in the risk register as critical. You may be maintaining code that is not what users run.

## Phase 2: Secrets and security sweep

Previous vendors often leave credentials in places they should not be.

- Scan full git history, not just the current tree, with a secrets scanner such as gitleaks or trufflehog.
- Look for API keys in `Info.plist`, `strings.xml`, `BuildConfig` fields, and JavaScript bundles.
- Rotate everything found, and everything the previous vendor had access to, regardless of where it was stored.
- Review local storage: tokens in `UserDefaults` or `SharedPreferences` instead of Keychain or Keystore-backed encryption is a common finding.
- Check certificate pinning, TLS configuration, and any outdated cryptography.

![Hand touching a tablet surrounded by virtual technical support and maintenance icons](https://cdn.sanity.io/images/563mnkns/production/17320578898a42bb1b184920492c288ab0943f5f-1600x900.jpg)

## Phase 3: Dependency and toolchain audit

Measure how far behind the project is, then estimate effort per item.

```bash
# Android: list outdated dependencies (with the versions plugin applied)
./gradlew dependencyUpdates -Drevision=release

# iOS: CocoaPods and SwiftPM
pod outdated
swift package show-dependencies --format json > deps.json

# JavaScript-based apps
npm outdated
npm audit --omit=dev
```

Capture results in a spreadsheet with columns for current version, latest version, major versions behind, known vulnerabilities, and estimated upgrade effort. Also record:

- Android `compileSdk`, `targetSdk`, and how close the app is to Google Play's next target API deadline.
- Xcode and Swift versions versus App Store Connect's current minimums.
- Whether privacy manifests exist for the app and for each SDK that needs one.
- Abandoned libraries with no release in a long time, which become migration projects.

## Phase 4: Code and architecture audit

You are looking for change risk, not style issues.

- **Module map.** Sketch the major modules and their dependencies. Note god classes and circular dependencies.
- **Test coverage.** Measure it, but also check whether tests run in CI and pass.
- **Hotspots.** Files with high churn and high complexity (git log frequency combined with size) are where defects cluster.
- **Observability.** Is crash reporting configured with symbol and mapping uploads? Are there performance traces, logs, and release health dashboards?
- **Store compliance.** Outstanding review warnings, data safety form accuracy, and account deletion support.

Write findings as a report with each item rated by severity and effort. That report is the evidence for the decision every takeover eventually faces: maintain as is, remediate, or partially rebuild.

## The 30/60/90-day plan

**Days 1 to 30: stabilize and see.** Complete Phases 0 to 2, get crash reporting and performance monitoring trustworthy, fix active P1 and P2 issues, and publish a baseline of key metrics (crash-free users, ANR rate, startup time, store rating).

**Days 31 to 60: secure and modernize.** Tackle critical findings from Phases 3 and 4: urgent dependency upgrades, security fixes, and any looming store deadline. Stand up CI with automated builds, tests, and staged rollouts. Publish the annual platform calendar.

**Days 61 to 90: improve and plan.** Shift capacity to preventive work: tests around hotspots, performance, and accessibility. Deliver a 12-month roadmap with measurable targets for each metric.

## Further reading

This runbook covers the technical side of a takeover. For scope, SLAs, contract terms, and budgeting, see our complete guide to [mobile app maintenance services for iOS and Android](https://techcirkle.com/blog/mobile-app-maintenance-services). If you would like an independent audit of an inherited app, you can [contact the TechCirkle team](https://techcirkle.com/contact-us).

## Frequently Asked Questions

### How long does a mobile codebase takeover audit take?

For a mid-sized app, one to three weeks is typical. Most of the variance comes from build reproducibility; an app that builds cleanly on day one moves much faster.

### What if the previous vendor will not hand over the Android signing key?

If the app uses Play App Signing, Google holds the app signing key and you can request an upload key reset through the Play Console. Without Play App Signing, recovery is much harder, so check this first.

### Should we rewrite an inherited app instead of maintaining it?

Decide with evidence from the audit. Rewrites make sense when the architecture blocks required changes or the toolchain gap is extreme; otherwise targeted remediation is usually cheaper and less risky.

### Which secrets should be rotated after a vendor handover?

All of them: API keys, backend credentials, push certificates and keys, signing passwords, CI tokens, and third-party SDK keys. Assume anything the previous team could access is exposed.

### What metrics should be baselined in the first month?

Crash-free users and sessions, ANR or hang rate, startup time, app size, store rating, release cadence, and the count of dependencies more than one major version behind.
