---
layout: post
title: "Field Notes: From Idea to Paying Customers in 90 Days"
date: 2026-11-14 00:00:00 +0000
categories: ["Engineering Notes", "AI Products"]
tags: ["ai", "engineering", "checklist", "startups", "saas"]
description: "Engineering field notes for launching an AI service for local businesses in 90 days: a month-by-month checklist and three numbers to know."
image: "https://cdn.sanity.io/images/563mnkns/production/f031a6f3574d59fead005dba1ac065f9203c780b-1600x1244.jpg"
author: "TechCirkle"
---

These notes assume a small team, one idea for an AI service aimed at local businesses, and a quarter to find out whether anyone will pay for it. The sequence puts learning in front of building. Work through the items in order, and treat a month you cannot close as a reason to narrow the scope, not to push on.

![A smartphone showing an analytics dashboard inside glowing neon rings](https://cdn.sanity.io/images/563mnkns/production/f031a6f3574d59fead005dba1ac065f9203c780b-1600x1244.jpg)

## Ground rules

- One vertical. One job to be done. Everything else waits.
- The product must act: book, quote or resolve. Taking a message does not count, because owners already have voicemail.
- Every conversation has a route to a person.
- Each conversation has a compute cost. Meter it from the first live call.
- Fix the failures you see in transcripts before the ones you imagined.

## Month 1: Days 1 to 30, pick and prove the problem

No serious code yet. The output of this month is a decision and five names.

- [ ] Choose one vertical and one job, for example call handling for a single trade
- [ ] Interview fifteen to twenty owners
- [ ] Listen to real call recordings if you can get them
- [ ] Map the two or three systems the trade actually uses for scheduling, jobs and payments
- [ ] Check that those systems expose APIs that can read availability and write appointments
- [ ] Sign up five design partners
- [ ] Decide the likely distribution channel: marketplace, trade association, supplier, franchise head office or agency

**Note:** the channel is an architecture input. A marketplace listing needs a deep integration with the host platform. A white-label arrangement needs theming, partner billing and tenant hierarchies. A franchise deal needs central control and reporting. Decide now and avoid a rebuild.

**Exit check:** five design partners have agreed, and you can describe the job in the owner's own words.

## Month 2: Days 31 to 60, build the narrowest useful version

Scope: one channel, one integration, a knowledge base per design partner, and a human fallback for everything else.

- [ ] Stand up a single channel. Text first is cheaper to get right. Voice adds latency engineering, telephony and far more testing
- [ ] If you choose voice: streaming speech recognition, fast first-token responses and interruption handling
- [ ] Conversation layer with instructions, guardrails and state, and four possible moves: answer, clarify, call a tool, hand over
- [ ] Knowledge store isolated per tenant, with services, prices, hours, policies and service areas
- [ ] Make knowledge correctable by a non-technical owner, and versioned so a wrong answer can be traced
- [ ] One integration that completes the job end to end
- [ ] Human handoff with a clear summary of the conversation so far
- [ ] Go live with supervision
- [ ] Read every transcript

**Note:** tenant isolation is not a later refactor. Put it in from the first partner. Multi-tenancy is basic [SaaS development](https://techcirkle.com/development/saas-development) hygiene and far cheaper now than after partner ten.

**Note:** test on bad lines. An assistant that works in a quiet office and fails on a mobile call from a van loses the customer in a day.

**Exit check:** real customers of your design partners are getting jobs booked or questions resolved without your help on most routine requests, and failures are logged with a cause.

## Month 3: Days 61 to 90, charge and test the channel

- [ ] Usage metering per tenant, by minutes or conversations
- [ ] Billing, with a base subscription and usage tiers. No unlimited plan
- [ ] Model routing: smaller models for routine turns, larger ones only when needed
- [ ] Cache common answers
- [ ] Onboarding flow that does not need you on a call. Aim for minutes of the owner's time
- [ ] AI disclosure at the start of conversations
- [ ] Recording disclosure and consent capture
- [ ] Retention settings and an audit log
- [ ] Start charging the design partners
- [ ] Take their results to one distribution channel and see whether it can produce customers repeatedly

**Note:** consent and disclosure are much harder to retrofit than to design in. Outbound AI calls and marketing texts carry stricter consent requirements than inbound answering, so get legal review before you add them.

**Exit check:** at least some design partners are paying, and you have run one real test of the channel.

![A business person using a smartphone surrounded by analytics and strategy icons](https://cdn.sanity.io/images/563mnkns/production/29c3888beae03adcddfadbc8a4f9719421437a72-1600x1022.jpg)

## Day 90: the three numbers

By the end of the quarter you should know three things. Write them down as numbers, not impressions.

1. **Retention.** Do owners keep the product switched on?
2. **Cost to serve.** What does one conversation cost you across speech recognition, the language model, speech synthesis and telephony?
3. **Channel economics.** Can your chosen channel deliver customers at a price that leaves a margin?

Those three numbers are worth more than any feature list. If one of them is bad, you know what the next quarter is for.

## Deliberately left out

- A second vertical. Three shallow products lose to one deep one.
- A second or third integration, unless a design partner cannot function without it.
- An owner mobile app. Owners will expect one eventually. It is not a 90-day item.
- Fine-tuned or self-hosted models. Those make sense at scale or when data residency rules demand them.
- Automated conversation evaluation at full depth. Start it early in a simple form, since it is easy to skip and expensive to lack.

## What usually takes longer than planned

The action layer. Each integration has its own quirks, limits and failure modes, and multi-step tasks such as checking availability, booking, confirming and creating a follow-up multiply them. Treat that sequence as a workflow with explicit states and retries. That is the core of [agentic workflow development](https://techcirkle.com/agentic-workflow-development), and it is where most of the engineering time goes.

## Reality check on scope

As a planning guide, a focused text-first product for a single vertical with one or two integrations can usually reach paying customers in roughly three months, for a budget in the tens of thousands of dollars. A voice-capable, multi-tenant platform with several integrations, billing and an owner app is a different project, more commonly six to nine months. If your 90-day plan looks like the second description, cut it down.

Background on the opportunities, unit economics, architecture and compliance is in our full guide to [AI services for local businesses](https://techcirkle.com/blog/ai-services-for-local-businesses).

## Frequently Asked Questions

### Is 90 days realistic for reaching paying customers?

It can be for a narrow scope: one vertical, one channel, one integration and a small group of design partners. A voice-capable, multi-tenant platform with several integrations is more commonly a six to nine month project.

### Why sign design partners before writing code?

They supply the real calls, systems and corrections that shape the product. Five committed partners in month one mean the month two build is tested against live use, not assumptions.

### Should the first version be voice or text?

Text first is the lighter route. Real-time voice adds latency engineering, telephony and far more testing. Choose voice first only if the job you picked, such as call handling, cannot be done without it.

### What should be metered from day one?

Usage per tenant, by minutes or conversations, along with the cost of each conversation. Gross margin depends on usage, so pricing decisions made without this data are guesses.

### What are the three numbers to know at day 90?

Whether owners keep the product switched on, what a conversation costs to serve, and whether the chosen distribution channel can deliver customers at a price that leaves a margin.
