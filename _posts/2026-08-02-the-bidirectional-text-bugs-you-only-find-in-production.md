---
layout: post
title: "The Bidirectional Text Bugs You Only Find in Production"
date: 2026-08-02 00:00:00 +0000
categories: ["Engineering", "Mobile"]
tags: ["unicode", "i18n", "testing", "mobile"]
description: "Mixed Arabic and Latin content produces rendering bugs that pass every test and are obvious to every user. What causes them and how to catch them."
author: "TechCirkle Editorial Team"
---

There is a category of bug that passes every test you have, survives code review, ships to production, and is instantly obvious to every user in your target market.

Bidirectional text rendering is the purest example. The characters are all present. They are in the correct logical order. Your snapshot test compares strings and passes happily. And the screen shows a phone number with its digits rearranged.

Here is why it happens and how to stop it.

## Logical order versus visual order

Text is stored in *logical* order — the order you would read it aloud. It is displayed in *visual* order — left to right on screen, physically.

For pure LTR or pure RTL content these are trivially related. For mixed content they are not, and the Unicode Bidirectional Algorithm decides the mapping.

The algorithm assigns each character a direction class. Arabic letters are strongly RTL. Latin letters are strongly LTR. And a large set of characters — spaces, most punctuation, and crucially **digits** — are *neutral* or *weak*, taking direction from what surrounds them.

That is the whole problem. Neutral characters at the boundary between an Arabic run and a Latin run can resolve either way, and the resolution depends on context your code did not intend to provide.

## The classic failures

**Phone numbers reordering.** `+966 50 123 4567` inside an Arabic sentence. The digit groups are weak, the separators are neutral, and the whole thing can resolve into a visually reordered sequence that reads as a different number. This is not cosmetic — users copy it, dial it, and it is wrong.

**Trailing punctuation jumping.** An Arabic sentence ending in a Latin brand name followed by a period. The period is neutral and can render at the wrong end of the line, which looks like a typo you cannot find in the source.

**URLs fragmenting.** A URL embedded in Arabic text where slashes and dots are neutral. Segments visually reorder and the link, though functionally intact, appears broken and is untrustworthy to the reader.

**Prices on the wrong side.** Currency symbols and amounts split across the direction boundary, so the amount and its symbol end up separated by intervening Arabic text.

**Concatenation catastrophes.** `arabicLabel + " " + latinValue` is the single most common source. The space is neutral, sits exactly at the boundary, and resolves according to surrounding context — which changes with the data.

## Why your tests do not catch this

Because most tests operate on logical order.

A snapshot test asserting `screen.getByText("+966 50 123 4567")` passes: the string is present in the accessibility tree in logical order. The rendering is wrong; the assertion is not testing rendering.

Even image-based snapshot tests often miss it, because they are usually run with test fixtures containing clean, single-direction content. The bug requires mixed content to manifest, and mixed content is exactly what synthetic test data lacks.

## What actually fixes it

**Use platform formatters.** Number, currency and date formatting through the platform's localized APIs handles direction correctly and eliminates the majority of cases at zero cost. Manual string building for any of these is the root cause more often than any other single practice.

**Isolate embedded runs.** Wrap foreign-direction content in Unicode isolate characters — `U+2066`–`U+2069` — or the platform's bidi-isolation API. Isolation tells the algorithm to treat the run as a self-contained unit, which is exactly what you meant.

**Never concatenate localized fragments with data values.** Use parameterized format strings so the formatter can reason about the whole sentence rather than about pieces glued together at runtime.

**Force direction on known-format fields.** Phone numbers, IBANs, order references, tracking codes, version strings and email addresses have a fixed presentation direction regardless of surrounding text. Mark them explicitly rather than hoping context resolves correctly.

**Ban raw string concatenation in UI code with a lint rule.** This is the enforcement that keeps the other four from decaying, because concatenation is muscle memory.

## Catching it before users do

Three practices, in increasing order of effectiveness:

1. **Test fixtures with realistic mixed content.** Every fixture containing Arabic should also contain a Latin brand name, a phone number, a price and a URL. Synthetic single-direction data hides the entire bug class.
2. **Screenshot tests in both directions** on screens displaying user-supplied or formatted data — visual comparison catches what string assertions cannot.
3. **A native Arabic reader in the release process.** Not a translator reviewing a spreadsheet — someone using the running app on real screens. This is the only check that reliably catches everything, because visual wrongness in Arabic is not detectable by a reviewer who cannot read it.

That last one is not a process nicety. If nobody in your release path reads Arabic, you have no detection mechanism for an entire class of user-visible defect, and your users become your QA.

The full buyer's guide — SAR cost bands, PDPL and residency, local integrations, and how to vet a partner — is here: **[Mobile App Development Company in Saudi Arabia: The 2026 Buyer's Guide](https://techcirkle.com/blog/mobile-app-development-company-saudi-arabia)**. Delivery details on our [mobile app development](https://techcirkle.com/development/mobile-app-development) page.

## Frequently Asked Questions

### What causes bidirectional text bugs?

Neutral and weak characters — spaces, punctuation and digits — take their direction from surrounding context. At the boundary between Arabic and Latin runs they can resolve in ways your code did not intend, reordering the visual output while the logical string stays correct.

### Why do my tests pass when the screen is wrong?

Because most assertions operate on logical order, and the string is correct in logical order. The failure is in visual order after the bidirectional algorithm runs, which only rendering-based checks or a human reader can detect.

### How do I fix a reordered phone number in Arabic text?

Format it through the platform's localized APIs and explicitly mark it as a self-contained run using Unicode isolate characters or the platform's bidi-isolation API, rather than concatenating it into a surrounding Arabic string.

### What are Unicode isolate characters?

Control characters (U+2066–U+2069) that tell the bidirectional algorithm to treat an enclosed run as a self-contained unit, preventing its direction from bleeding into surrounding text or being altered by it.

### Which coding practice causes the most of these bugs?

Concatenating a localized label with a data value using a plain space. The space sits exactly at the direction boundary and resolves according to context that changes with the data. Use parameterized format strings instead, enforced by a lint rule.

### Can automated testing fully cover this?

Not entirely. Realistic mixed-content fixtures and dual-direction screenshot tests catch a great deal, but a native Arabic reader reviewing the running app remains the only reliable final check.
