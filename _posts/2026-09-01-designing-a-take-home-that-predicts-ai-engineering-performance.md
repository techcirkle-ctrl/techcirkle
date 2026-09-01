---
layout: post
title: "Designing a Take-Home That Predicts AI Engineering Performance"
date: 2026-09-01 00:00:00 +0000
categories: ["Engineering", "Hiring"]
tags: ["ai", "hiring", "testing", "engineering"]
description: "Chatbot take-homes teach you nothing. A four-hour evaluation exercise with deliberately mislabelled data reveals who can actually do the job."
image: "https://cdn.sanity.io/images/563mnkns/production/ac641fff307663a9437e4b52b85722badfcf7144-1600x900.jpg"
author: "TechCirkle Editorial Team"
---

![Engineer reviewing evaluation results on screen](https://cdn.sanity.io/images/563mnkns/production/ac641fff307663a9437e4b52b85722badfcf7144-1600x900.jpg)

If your AI take-home asks the candidate to build a chatbot, you are testing the part of the job that stopped being difficult two years ago, and you will not be able to rank your submissions on anything meaningful.

Here is a format that does discriminate, why each part of it is there, and how to grade it.

## The exercise

Provide thirty real examples of a classification or extraction task from your own domain. Ask for three deliverables: a working solution, an evaluation harness, and a short written analysis of where the approach fails. Cap the time at four hours and say so explicitly in the brief.

Three properties of the dataset do all the work:

1. **It is real.** Actual inputs from your product, with the mess intact — inconsistent formatting, truncated fields, the customer who wrote everything in one paragraph.
2. **Several cases are genuinely ambiguous.** Reasonable people would label them differently. There is no correct answer, only a defensible one.
3. **A few are mislabelled.** This is the part that matters most, and you should not mention it.

## Why the mislabelled cases are the whole point

An engineer who treats the provided labels as ground truth will optimise against noise and report a confident accuracy number that means nothing. An engineer with production instincts will notice that a handful of cases look wrong, check them, and say so in the write-up.

That behaviour — distrusting your data before trusting your model — is the single most transferable habit in applied AI work. It is also nearly impossible to fake, because you either have the reflex or you do not.

In practice, roughly a third of candidates catch it. Almost all of those are people who have maintained a real dataset in production.

## Grading

The accuracy number is close to irrelevant. Grade in this order:

**The written analysis (50%).** Did they characterise failure modes rather than just report a score? Did they notice the bad labels? Do they distinguish between cases the approach genuinely cannot handle and cases where the task definition is underspecified? Can they say what they would need to push it further?

**The evaluation harness (30%).** Does it measure something meaningful for the task, or is it accuracy on everything regardless of whether accuracy is the right metric? Are the hard cases represented separately from the easy ones? Could you run it tomorrow against a different approach and get a comparable number?

**The solution (20%).** Does it work, is it readable, does it handle the malformed inputs without falling over. That is genuinely all.

![Panel reviewing a candidate submission](https://cdn.sanity.io/images/563mnkns/production/35b9a3e2d44cc44e52fc58a59fddbd4a5c9cce88-1600x655.jpg)

## Assume AI assistance, and design for it

Every candidate will use AI coding assistants. That is how the job is done now, and screening for their absence selects for the wrong thing entirely.

What this changes is what you are actually measuring. Generating a plausible solution is now cheap for everyone, so the discriminating skill has moved to judgement about the output: whether the harness measures the right thing, whether edge cases are represented, whether they caught the confidently-written helper that silently drops nulls.

A useful addition to the brief: ask candidates to note anything in their submission they are not confident about. Strong engineers produce a specific, honest list. It correlates well with how they behave in code review.

## What the four-hour cap is for

Two reasons, both practical.

It keeps strong senior candidates in your pipeline. They are employed, busy, and will decline a twelve-hour exercise. Long take-homes select for available time rather than ability.

And it forces prioritisation, which is itself a signal. With four hours, a candidate has to decide whether to spend the time on solution quality or on the harness. The ones who invest in the harness and ship a simpler solution are usually the ones you want, and you can only observe that choice if the budget is genuinely tight.

## What to do in the debrief

Spend the follow-up conversation on the analysis, not the code. Useful prompts:

- Which of these cases do you think are labelled wrong, and how confident are you?
- If this went to production tomorrow, what breaks first?
- What would this cost to run per thousand requests, and how would you reduce it?
- Where would you not use a model at all here?

That last one is the maturity question. Experienced engineers usually identify at least one part of the task that a rule, a lookup or a better-designed input form would handle more cheaply and more reliably than any model.

The complete hiring guide — four AI developer archetypes, rate benchmarks by market, screening questions and the in-house versus partner decision — is here: [Hire AI Developer in 2026](https://techcirkle.com/blog/hire-ai-developer). Our own approach to this work is described under [AI development services](https://techcirkle.com/ai-development-services).

## Frequently Asked Questions

### Is a take-home fair to candidates?

Only if it is short, capped, and clearly relevant to the job. Four hours with a stated limit is defensible. Anything longer trades your convenience against their time and will cost you strong senior candidates.

### Should I pay candidates for take-home work?

Increasingly the norm for exercises over a couple of hours, and it improves completion rates noticeably at the senior end. It also obliges you to keep the scope honest.

### What if a candidate does not catch the mislabelled cases?

Not automatically disqualifying, but probe it in the debrief. Ask directly whether they trusted the labels. Some candidates notice and assume the noise was intentional but not worth mentioning — that is a communication issue rather than a judgement one.

### Can I reuse the same exercise across many candidates?

Yes, with a caveat: rotate the dataset periodically, especially if you hire from a small community. The exercise loses its diagnostic value the moment the mislabelled cases become known.

### How does this work for agentic roles?

Adapt it: give a multi-step workflow where one tool intermittently returns malformed responses. You are grading failure design — timeouts, idempotency, compensating actions, human handoff — rather than extraction accuracy.

### What if we have no domain data suitable for this?

Use a public dataset from a related domain and deliberately corrupt a handful of labels yourself. It loses some realism but retains almost all of the diagnostic power, since the core signal is whether the candidate distrusts the data.
