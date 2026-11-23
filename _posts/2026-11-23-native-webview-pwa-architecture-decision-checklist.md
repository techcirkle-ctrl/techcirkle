---
layout: post
title: "Reference: Native vs Web View vs PWA Architecture Decision Checklist"
date: 2026-11-23 00:00:00 +0000
categories: ["Engineering", "Reference"]
tags: ["architecture", "pwa", "mobile", "checklist", "reference"]
description: "A technical reference checklist for choosing native, web view or PWA: capability audit, offline tiers, distribution, AI, security, cost and launch QA."
image: "https://cdn.sanity.io/images/563mnkns/production/cbdf958dbab7cacc505b39d98c674203cf508d34-2304x1792.jpg?w=1600"
author: "TechCirkle"
---

![Isometric illustration of a web browser window, cloud services and connected devices on a dark blue grid](https://cdn.sanity.io/images/563mnkns/production/cbdf958dbab7cacc505b39d98c674203cf508d34-2304x1792.jpg?w=1600)

A working reference for architecture reviews. Copy it into your ADR template and fill it in before anyone picks a framework. Rationale and scenarios live in the source guide: [native app vs web view vs PWA](https://techcirkle.com/blog/native-app-vs-web-view-vs-pwa).

## 0. Terminology

| Term | Runtime | Distribution | Examples |
| --- | --- | --- | --- |
| Native | Owned by the app | App stores | Swift/SwiftUI, Kotlin/Jetpack Compose |
| Cross-platform (treated as native) | Owned by the app | App stores | React Native, Flutter |
| Web view (hybrid) | System browser engine inside an owned shell | App stores | Capacitor, Cordova on WKWebView / Android WebView |
| PWA | Browser, controlled by user and vendor | URL | Service worker + web app manifest |

## 1. Capability audit (24-month horizon)

Mark each capability `core`, `nice`, or `none`.

```text
[ ] Camera / file upload / clipboard          -> all three OK
[ ] Basic geolocation                         -> all three OK
[ ] Push notifications                        -> PWA: iOS only after home-screen install
[ ] Background location / geofencing          -> native
[ ] Bluetooth Low Energy peripherals          -> native (hybrid via plugin)
[ ] NFC payments                              -> native
[ ] HealthKit / Health Connect                -> native
[ ] CarPlay / Android Auto                    -> native
[ ] Home-screen widgets                       -> native
```

**Rule:** two or more native-only items marked `core` means PWA and web view are effectively ruled out.

## 2. Offline tier

| Tier | Description | Minimum viable option |
| --- | --- | --- |
| T0 | Online only | Any |
| T1 | Shell and cached reading | PWA (service worker) |
| T2 | Short queued writes | PWA (IndexedDB); no Background Sync in Safari |
| T3 | Hours of critical data entry | Native/cross-platform (local DB, OS-scheduled sync) or hybrid + SQLite plugin |

Verify: what happens to unsynced data when the app is killed, the device restarts, or storage is cleared?

## 3. Distribution and acquisition

```text
Primary acquisition channel:   [ store search | search engines | shared links | sales ]
In-app purchase of digital goods: [ yes | no ]
Store presence required:       [ yes | no ]
```

- Store search, ratings or IAP central -> needs store presence.
- Search, links, sales -> PWA removes friction; native needs Universal Links / App Links and web-mirrored content to compete.
- Web view in stores -> must offer real app value, or review may reject it.
- Store rules vary by region and change often.

## 4. AI placement

| Pattern | Examples | Recommendation |
| --- | --- | --- |
| Cloud | Chat, report generation, search | Any front end |
| Instant / offline | Live camera analysis, field voice, on-device redaction | Native/cross-platform |
| Hybrid local + cloud | Small model on device, large in cloud | Native/cross-platform |

Native frameworks: Core ML and Apple on-device model APIs (iOS); LiteRT, ML Kit, Gemini Nano (Android). Web: WebGPU and WebAssembly runtimes, with variable performance and limited NPU access.

## 5. Security and compliance

```text
[ ] Hardware-backed key storage needed
[ ] Biometric auth
[ ] Certificate pinning
[ ] Jailbreak / root detection
[ ] App attestation
[ ] Regulated domain (banking, payments, healthcare)
```

Any of the last two lines checked -> prefer native/cross-platform. Hybrid: audit the JavaScript-to-native bridge and restrict remote content in the shell.

## 6. Team and maintenance

- Existing web team, React codebase -> PWA or web view ships fastest.
- React / React Native team -> cross-platform with shared TypeScript.
- Fully native -> two skill sets, two release trains.
- Maintenance: native/cross-platform need yearly OS/SDK/store-policy work; web view adds plugin upkeep; PWA tracks browser changes, especially Safari.

## 7. Cost envelope (USD, backend excluded)

| Option | Typical build |
| --- | --- |
| PWA | 40k–100k |
| Web view hybrid | 50k–120k |
| Cross-platform | 80k–180k |
| Two native apps | 140k–300k+ |

Annual maintenance: 15–25% of build. Factor in the cost of a future rebuild if capability or offline audits point beyond the cheapest option.

## 8. Migration guardrails (if starting lighter)

```text
[ ] Business rules live in the API, not the client
[ ] Domain types and validation in a shared TypeScript package
[ ] Design tokens in a platform-neutral format
[ ] Storage, push and device APIs behind adapter interfaces
[ ] No core feature depends on a plugin without a native equivalent
```

## 9. Pre-launch QA

```text
[ ] Low-end Android + older iPhone, throttled network, airplane mode
[ ] Kill-mid-save / restart / clear-storage recovery
[ ] Push permission flows on both platforms (incl. iOS home-screen rule)
[ ] Accessibility: dynamic type, screen readers, contrast
[ ] Store guideline review vs feature set (esp. web view + IAP)
[ ] Analytics + crash reporting from day one (incl. service worker errors)
[ ] Update strategy: forced updates (native) / cache invalidation (PWA, web view)
[ ] AI model inventory: where it runs, what data it sees, how it updates
```

## 10. Spike before committing

Prototype the single most demanding screen in the candidate approach. Test on a three-year-old mid-range Android device. Timebox to about two weeks.

Need a second pair of eyes on a completed checklist? Our [mobile app development](https://techcirkle.com/development/mobile-app-development) team runs these reviews. Full context: [the source guide](https://techcirkle.com/blog/native-app-vs-web-view-vs-pwa).

## Frequently Asked Questions

### How do I use this checklist?

Fill it in during an architecture review, before choosing a framework. The capability audit and offline tier usually narrow the options fastest; the remaining sections confirm or refine the choice.

### What is the quickest disqualifier for a PWA?

Two or more native-only capabilities marked as core, or a T3 offline requirement with hours of critical data entry.

### Should cross-platform be scored as native?

Yes. React Native and Flutter own their runtime and reach device features through native modules, so they behave like native apps for these criteria.

### What does a web view app need to pass store review?

Genuine app value beyond a wrapped website, such as native navigation, offline features, notifications or device integrations.

### How long should the validation spike be?

Around two weeks, focused on the riskiest screen, on realistic hardware and networks.
