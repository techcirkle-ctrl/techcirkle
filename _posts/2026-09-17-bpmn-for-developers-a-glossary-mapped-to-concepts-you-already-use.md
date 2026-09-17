---
layout: post
title: "BPMN for Developers: A Glossary Mapped to Concepts You Already Use"
date: 2026-09-17 00:00:00 +0000
categories: ["Engineering", "Architecture"]
tags: ["bpmn", "bpm", "orchestration", "backend", "architecture"]
description: "A developer glossary for BPM in software: BPMN events, gateways, boundary events and compensation mapped to async, try/catch, sagas and feature flags."
image: "https://cdn.sanity.io/images/563mnkns/production/37e5a48d5ddd9ae55e9b883b2fa3e12ee7d7d76d-1600x1066.jpg"
author: "TechCirkle Editorial Team"
---

![Developer screen showing a BPMN diagram beside source code in an editor](https://cdn.sanity.io/images/563mnkns/production/37e5a48d5ddd9ae55e9b883b2fa3e12ee7d7d76d-1600x1066.jpg)

Many engineers see BPMN circles and diamonds and assume it is documentation for non-coders. That is a costly assumption. BPMN 2.0 (also published as ISO/IEC 19510) has execution semantics, and engines such as Camunda and Flowable run it directly. Every element maps to a pattern you have probably hand-rolled at least once, usually with less rigour.

This glossary translates the notation into engineering terms, so you can decide when a process engine saves effort and when plain code is enough. Snippets are illustrative pseudocode, not engine-specific APIs.

## Process Definition and Process Instance

**BPMN:** A process definition is the versioned model. A process instance is one execution of it, such as a single loan application.

**You already know it as:** a class and an object, or a workflow definition and a job run. The difference is persistence. The engine stores instance state durably, so an instance waiting three days survives deploys and restarts.

```text
definition: loan-approval v7
instance:   loan-approval v7 / case 12931 / waiting at "Verify documents"
```

## Start, End and Intermediate Events

**BPMN:** Circles. A start event triggers a process (message, timer, form). An intermediate event means waiting for something.

**You already know it as:** an event handler or consumer entry point, and an `await` on an external signal. The engine turns a blocking wait into a persisted subscription, so no thread sits idle.

```text
on message "PaymentConfirmed" correlate by orderId -> resume instance
```

## Service Task

**BPMN:** A rounded rectangle that calls a system.

**You already know it as:** a function call to an API or a job worker pulling from a queue. In most modern engines, workers poll for jobs of a given type, which looks a lot like a queue consumer. Make handlers idempotent, because the engine will retry.

## User Task

**BPMN:** Work a person performs, surfaced in a task list.

**You already know it as:** a record in a `pending_reviews` table plus a UI and a status column, which you then wire to notifications and permissions. The engine gives you assignment, claiming and completion semantics so you only build the screen.

## Exclusive, Parallel and Event-Based Gateways

**BPMN:** Diamonds controlling flow.

- **Exclusive gateway:** `if / else if / else`. Exactly one path. Watch for overlapping or gapped conditions, the classic bug.
- **Parallel gateway:** `Promise.all` or a fan-out/fan-in. Every branch runs, and the join waits for all.
- **Event-based gateway:** `Promise.race` between external events, such as a customer reply versus a timeout.

```text
race(
  waitFor("CustomerReplied"),
  timer("P2D")          // ISO 8601 duration: two days
)
```

## Boundary Events

**BPMN:** An event attached to the edge of a task that interrupts or supplements it.

**You already know it as:** `try/catch` for error boundaries and a timeout wrapper for timer boundaries. A non-interrupting timer is closer to a scheduled reminder that fires while the task keeps waiting, for example escalating an approval untouched for 48 hours.

![Engineer sketching gateways and boundary events on a whiteboard next to pseudocode](https://cdn.sanity.io/images/563mnkns/production/5f2cf1d82443a6547c3eb667e7be337fb91c07c8-1600x1066.jpg)

## Subprocess and Call Activity

**BPMN:** An embedded subprocess groups steps. A call activity invokes a separately deployed process.

**You already know it as:** a private helper function versus a shared library or service. A reusable KYC check as a call activity is the same instinct as extracting a module.

## Compensation

**BPMN:** Defined undo actions for completed steps when a later step fails.

**You already know it as:** the saga pattern. Reserve stock, charge the card, and if shipping fails, refund and release. BPMN makes the compensation paths visible instead of scattering them across services.

## Business Rule Task and DMN

**BPMN:** A task that evaluates a decision, usually a DMN decision table.

**You already know it as:** config-driven rules or a feature-flag-style lookup, with versioning. Thresholds and routing live outside code, so policy owners can change them without a release, and every instance records which rule version it used.

## Pools, Lanes and Message Flows

**BPMN:** Pools represent participants, lanes represent roles, message flows cross pool boundaries.

**You already know it as:** service ownership boundaries and the contracts between them. Lanes are handy for spotting handoffs, which is where latency hides.

## Orchestration vs Choreography

A BPMN process is orchestration: one component owns the end-to-end flow. Event-driven choreography has each service react independently. Choreography scales well but makes "why did this order take eleven days?" hard to answer. Many teams keep events for integration and add an orchestrator for the few long-running flows that need an owner. Code-first durable execution frameworks like Temporal solve similar problems in general-purpose code, which suits engineering-owned flows better than stakeholder-editable ones.

## Where AI Agents Fit in the Glossary

An LLM call is just a service task with non-deterministic output. Treat it that way: a strict output schema, a boundary timer, an error path, retries with a fallback model, and a gateway (not the model) that decides what happens to the result based on confidence and risk. Log model and prompt versions as process variables for traceability. This pattern is central to the [agentic workflow development](https://techcirkle.com/agentic-workflow-development) we do, and the full reasoning is in our guide to [BPM in software](https://techcirkle.com/blog/bpm-in-software). If the process is part of a product, the surrounding task UIs usually become a [web app development](https://techcirkle.com/development/web-app-development) effort in their own right.

## Frequently Asked Questions

### Is BPMN actually executable or just documentation?

It is executable. BPMN 2.0 defines execution semantics, and engines such as Camunda and Flowable run models directly.

### When should I use a process engine instead of plain code?

When flows are long-running, involve human tasks, need timers and escalations, or require an audit trail. Short synchronous logic is usually fine in code.

### How do BPMN models fit into Git and CI?

Models are XML files that can be versioned, reviewed in pull requests and exercised by automated process tests in your pipeline.

### What is the most common modelling bug?

Exclusive gateways with overlapping or incomplete conditions, followed by parallel splits without a matching join.

### How is Temporal different from a BPMN engine?

Temporal expresses workflows in code rather than diagrams, which suits engineering-owned flows over stakeholder-editable ones.

### How should LLM calls be modelled?

As service tasks with defined schemas, timeouts, error paths and a rule-based gateway that decides how outputs are used.
