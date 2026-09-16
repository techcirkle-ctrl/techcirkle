---
layout: post
title: "Accessibility Designed In: A Mobile Checklist for iOS and Android"
date: 2026-09-16 00:00:00 +0000
categories: ["Accessibility", "Mobile"]
tags: ["accessibility", "ios", "android", "swiftui", "compose"]
description: "A practical checklist for building accessibility into mobile app design: dynamic type, VoiceOver and TalkBack labels, contrast, targets and motion."
image: "https://cdn.sanity.io/images/563mnkns/production/18398a9338cfe4b675064cbc0920e98d54b28030-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

![Mobile interface wireframes and colour palettes laid out during design planning](https://cdn.sanity.io/images/563mnkns/production/18398a9338cfe4b675064cbc0920e98d54b28030-1600x1066.jpg)

There are two ways accessibility enters a mobile project.

The first is an audit a few weeks before release. Someone turns on the largest text size, finds that half the screens clip, turns on VoiceOver, hears "button, button, image", and files forty tickets. Most get deferred. Some ship.

The second is at the design stage, where accessibility is a set of constraints the layouts are built around. It costs far less and helps everyone, including people reading in sunlight.

This post is the checklist we use for the second approach. Snippets are **illustrative** — simplified to show the intent, not production code.

## 1. Dynamic type at the largest sizes

Both platforms let users scale text far beyond the default. iOS includes accessibility sizes well past the standard range; Android applies a user font scale to `sp` units.

The design implication: **layouts must reflow, not shrink.** Horizontal rows of label plus value become vertical stacks. Truncation is a last resort, never the default.

Design-stage rules:

- Design key screens at the default size **and** at the largest accessibility size before sign-off.
- Never set text inside fixed-height containers.
- Specify which rows switch from horizontal to vertical when text grows.
- Avoid text baked into images.

Illustrative SwiftUI:

```swift
// Illustrative only
struct PriceRow: View {
    @Environment(\.dynamicTypeSize) var size
    var body: some View {
        let layout = size.isAccessibilitySize
            ? AnyLayout(VStackLayout(alignment: .leading))
            : AnyLayout(HStackLayout())
        layout {
            Text("Monthly plan")
            Spacer(minLength: 8)
            Text("Billed monthly").font(.body.weight(.semibold))
        }
    }
}
```

Illustrative Compose:

```kotlin
// Illustrative only: use sp for text so user font scale applies
Text(
    text = "Monthly plan",
    style = MaterialTheme.typography.bodyLarge, // sp-based
    maxLines = Int.MAX_VALUE
)
```

## 2. Screen reader labels, written by designers

VoiceOver and TalkBack read what the code exposes. If the design file does not say what an icon button is called, a developer will guess, or it will be announced as "button".

Design-stage rules:

- Annotate **every** icon-only control with its spoken label.
- Define reading order for complex cards (title first, then metadata, then actions).
- Mark decorative images as decorative so they are skipped.
- Group related elements so a card is one announcement, not six.
- Specify state announcements: selected, expanded, loading, error.

Illustrative SwiftUI:

```swift
// Illustrative only
Button(action: toggleFavourite) {
    Image(systemName: isFavourite ? "heart.fill" : "heart")
}
.accessibilityLabel("Favourite")
.accessibilityValue(isFavourite ? "On" : "Off")
```

Illustrative Compose:

```kotlin
// Illustrative only
IconButton(
    onClick = onToggle,
    modifier = Modifier.semantics {
        contentDescription = "Favourite"
        stateDescription = if (isFavourite) "On" else "Off"
    }
) { Icon(Icons.Default.Favorite, contentDescription = null) }
```

## 3. Contrast as a token property

Contrast failures are nearly always design-system failures: a grey chosen for elegance that fails against a tinted surface.

Design-stage rules:

- Target WCAG 2.2 AA as a baseline: 4.5:1 for body text, 3:1 for large text and meaningful non-text elements such as icons and input borders.
- Check contrast **per token pair** (text token on surface token), in light and dark mode.
- Never rely on colour alone for status; pair it with an icon or text.
- Check disabled and placeholder states deliberately rather than by accident.

![Mobile screen sketches and interface components on a planning desk](https://cdn.sanity.io/images/563mnkns/production/dd6a47f69bc53bbd0bda8c42d7462eb0a854dff0-1600x1068.jpg)

## 4. Touch targets and thumb reach

Small targets hurt anyone with a tremor or on a moving bus.

Design-stage rules:

- iOS guidance recommends at least 44×44 points; Material recommends at least 48×48 dp.
- The **hit area** can exceed the visible icon; specify both in the design.
- Keep enough spacing between adjacent destructive and non-destructive actions.
- Put primary actions within comfortable one-handed reach on large phones.

Illustrative Compose:

```kotlin
// Illustrative only: Material components enforce a minimum
// interactive size; custom clickables should too
Box(
    modifier = Modifier
        .minimumInteractiveComponentSize()
        .clickable(onClickLabel = "Close") { onClose() }
) { Icon(Icons.Default.Close, contentDescription = "Close") }
```

## 5. Reduced motion

Parallax, zooming transitions and bouncing elements can cause genuine discomfort for people with vestibular disorders.

Design-stage rules:

- For every significant animation, specify a reduced-motion alternative (usually a cross-fade or no animation).
- Never convey information through motion alone.
- Avoid auto-playing looping animation on key screens.

Illustrative SwiftUI:

```swift
// Illustrative only
@Environment(\.accessibilityReduceMotion) var reduceMotion

.transition(reduceMotion ? .opacity : .move(edge: .bottom))
```

On Android, read the system animator duration scale or the "remove animations" setting and apply the same fallback.

## 6. Testing on real devices

Simulators miss glare, thumb reach and older hardware.

- Navigate each core journey **eyes closed** with VoiceOver on an iPhone and TalkBack on an Android phone.
- Run the largest text size on a small-screen device, not just a large one.
- Test with a mid-range Android, where performance and accessibility services interact.
- Use Accessibility Inspector (Xcode) and Accessibility Scanner (Android) as a first pass, not a verdict.
- Where possible, include people who use assistive technology daily in usability sessions.

## The checklist

Copy this into your design review template.

- [ ] Key screens designed at default and largest dynamic type sizes
- [ ] No text in fixed-height containers or baked into images
- [ ] Every icon-only control has a spoken label in the spec
- [ ] Reading order and grouping defined for complex components
- [ ] Decorative images marked as decorative
- [ ] State changes (selected, loading, error) have announcements
- [ ] Contrast checked per token pair in light and dark mode
- [ ] Status never conveyed by colour alone
- [ ] Touch targets meet 44pt / 48dp, hit areas specified
- [ ] Reduced-motion alternative for every significant animation
- [ ] Core journeys tested with VoiceOver and TalkBack on real devices
- [ ] Largest text size tested on a small device
- [ ] Automated scanner run, findings triaged

## Why this belongs at the design stage

Every item above is cheap in Figma and expensive in code. A label in the spec is one line; discovered in QA, it is a ticket, a build and a regression test.

This is one of the clearest tests when evaluating a design partner: ask how they test with VoiceOver and TalkBack, and how their layouts handle the largest text sizes. We cover that and other evaluation criteria in our guide to [choosing a mobile app design agency](https://techcirkle.com/blog/mobile-app-design-agency). For teams that want design and native engineering in one backlog, see our [mobile app development](https://techcirkle.com/development/mobile-app-development) work.

---

## Frequently Asked Questions

### Why design for the largest text sizes instead of testing them later?

Supporting them usually changes layout structure, which is far cheaper to adjust in a design file than in built components.

### Who should write accessibility labels, designers or developers?

Designers should specify them in the handoff, because they know the intent of each control. Developers then implement exactly what is specified.

### What contrast level should a mobile app target?

WCAG 2.2 AA is a sensible baseline: 4.5:1 for normal text and 3:1 for large text and important non-text elements.

### Can a touch target be larger than the visible icon?

Yes. The tappable area can extend beyond the drawn icon, and the design should state both sizes so engineers build it correctly.

### Is an automated accessibility scanner enough?

No. Scanners catch missing labels and small targets, but only real screen-reader use on devices reveals confusing order and unclear announcements.

### Does reduced motion mean removing all animation?

Not necessarily. It usually means replacing large movement with subtle fades, and never relying on motion alone to communicate information.

---

*Full version: [Mobile App Design Agency: How to Choose One That Designs for Engineering, Retention and AI](https://techcirkle.com/blog/mobile-app-design-agency).*
