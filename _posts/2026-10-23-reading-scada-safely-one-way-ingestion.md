---
layout: post
title: "Reading SCADA Safely: A One-Way Ingestion Pattern"
date: 2026-10-23 00:00:00 +0000
categories: ["Security", "Architecture", "Industrial"]
tags: ["scada", "security", "architecture", "iiot"]
description: "How to get operational technology data into a cloud analytics platform without creating a write path back into a safety-critical network."
image: "https://cdn.sanity.io/images/563mnkns/production/305c5a4bbbd2674a3b51ed924779af526d4832cc-1600x1066.jpg"
author: "TechCirkle"
---

![Engineer inspecting equipment at a water treatment plant](https://cdn.sanity.io/images/563mnkns/production/305c5a4bbbd2674a3b51ed924779af526d4832cc-1600x1066.jpg)

Every industrial site of any size already has SCADA, and that system is correctly treated as safety-critical. If your analytics platform needs its data — and it does, because the best-instrumented assets are already on it — then you are about to meet an OT security team.

This is the point where industrial analytics projects get cancelled rather than merely delayed. Worth designing for it from the start.

## The principle: directionality is not negotiable

Data flows out of the OT network into the analytics platform. Control does not flow back in.

Not "control flows back in with authentication." Not "control flows back in for low-risk setpoints." It does not flow back in as part of this project. If a write path is genuinely required, it is a separate project with its own risk assessment, its own sign-off and its own change control.

Accepting this early is what makes the rest of the conversation possible. Projects that arrive proposing closed-loop control get a long review and often no approval.

## The topology that gets signed

```
┌─────────────── OT zone (Purdue L2/L3) ───────────────┐
│  PLCs / RTUs                                         │
│       │                                              │
│  Historian  ──►  OPC UA server                       │
└───────────────────────┬──────────────────────────────┘
                        │  unidirectional
                        ▼
┌─────────────── DMZ ──────────────────────────────────┐
│  Data diode or one-way gateway                       │
│       │                                              │
│  MQTT broker / staging store                         │
└───────────────────────┬──────────────────────────────┘
                        │  outbound TLS only
                        ▼
┌─────────────── Cloud analytics ──────────────────────┐
│  Ingestion ─► time-series store ─► models ─► alerts  │
│                                        │             │
│                                        ▼             │
│                        Work orders in CMMS/EAM       │
└──────────────────────────────────────────────────────┘
```

Four properties that make this acceptable:

1. **No inbound connections to the OT zone.** The OT side initiates; nothing dials in.
2. **No inbound connections to the DMZ from the internet.** The cloud ingestion pulls, or the DMZ pushes outbound. Never a listening port facing out.
3. **Protocol break at the diode.** The DMZ speaks a different protocol than the OT zone, so an OPC UA exploit does not traverse.
4. **The output lands in the maintenance system, not in SCADA.** Recommendations become work orders a human dispatches.

Point four is the one that resolves the whole argument. It changes "your cloud model will operate my plant" into "your cloud model will put a job in my maintenance queue," which is a completely different risk conversation.

## Reading from the historian, not the PLCs

Go to the historian or an OPC UA server, not directly to controllers.

Controllers are sized for control, not queries. A polling client scanning tags at a rate the integrator did not anticipate is a genuine operational risk, and it is the kind of thing that gets your project removed from site. The historian exists to be read.

```python
# conceptual: subscribe rather than poll
from asyncua import Client

async with Client(url=OPC_UA_HISTORIAN) as client:
    nodes = [client.get_node(nid) for nid in TAG_IDS]
    sub = await client.create_subscription(period=1000, handler=handler)
    await sub.subscribe_data_change(nodes)
```

Subscribe to data changes rather than polling where the server supports it. Lower load, and you get the server's own change detection rather than reimplementing it badly.

Agree the tag list and the sampling rate in writing with the site. "We will read these 340 tags at 1Hz" is a reviewable statement. "We will read the historian" is not, and it will be refused.

## Buffer in the DMZ, assume the WAN fails

Site connectivity drops. If the DMZ broker has no persistence, that is a permanent gap in your dataset, and gaps in training data are expensive in a way that is not obvious until you try to train.

Give the staging layer disk and a retention window sized to the longest realistic outage — a week is a reasonable starting point for a remote site. Backfill on reconnect, and mark backfilled readings so downstream consumers know the order they arrived in.

This is the same late-arrival problem that any field telemetry pipeline has, and it wants the same treatment: store both the measurement timestamp and the receipt timestamp, and make aggregates correct themselves when late data lands.

## Returning results without a write path

So the model has concluded something. How does anything happen?

Through the maintenance system. The output of the analytics platform is a work order, created via the CMMS or EAM API, containing the asset, the location, a priority, the evidence and an access note. A human dispatches it.

This is slower than closed-loop control and it is what gets approved, and in practice the latency rarely matters — a suspected background leak does not need a sub-second response, it needs to reach a crew this week instead of next quarter.

Where an assembled work order is genuinely useful is in removing the twenty minutes of correlation work per alert: pulling asset history, checking for nearby crews, looking up pipe material and age, estimating severity. A constrained agent can assemble all of that and leave the dispatch decision with a person, which is how we build [agentic workflows](https://techcirkle.com/agentic-workflow-development) in operational settings — fixed tool set, bounded action space, approval gate on anything that sends a vehicle.

## A review checklist

Bring this to the security conversation rather than waiting for it:

- Written tag list and sampling rate, agreed with the site.
- Read-only credentials on the historian, scoped to those tags.
- No inbound connections to the OT zone from anywhere.
- Protocol break between OT and DMZ.
- Outbound-only from DMZ to cloud, over TLS, with certificate pinning.
- Buffering in the DMZ sized to the longest expected outage.
- No API, credential or code path in the system capable of writing to SCADA.
- Outputs land in the CMMS; dispatch requires a human.
- Full audit log of every read and every work order created.

The broader architecture — time-series platform, model placement, phasing and cost — is in TechCirkle's [guide to smart water systems](https://techcirkle.com/blog/smart-water-systems), and our [custom software development](https://techcirkle.com/development/custom-software-development) practice covers how these integrations get delivered.

## Frequently Asked Questions

### Why must SCADA integration be unidirectional?
Because SCADA is safety-critical, and any inbound path is a potential route to affecting physical plant. Keeping data flow strictly outbound means an analytics platform compromise cannot become an operational incident, which is the condition most OT security teams require before approving anything.

### Should analytics read from PLCs or from the historian?
From the historian or an OPC UA server. Controllers are sized for control loops rather than queries, and an unanticipated polling load on a PLC is a genuine operational risk. The historian exists to be read and is the correct integration point.

### What is a protocol break and why does it matter?
It means the DMZ speaks a different protocol than the OT zone, so a vulnerability in an industrial protocol implementation cannot traverse from one side to the other. It is one of the properties that makes a one-way architecture defensible rather than merely nominally unidirectional.

### How do analytics results reach operations without a write path?
As work orders created in the maintenance management system, containing the asset, location, priority, evidence and access notes. A human dispatches them. This is slower than closed-loop control and is almost always sufficient, since the decisions involved are not sub-second.

### What happens to data during a site connectivity outage?
The DMZ staging layer should buffer to disk with a retention window sized to the longest realistic outage, then backfill on reconnect. Backfilled readings should be marked, and downstream aggregates must correct themselves when late data arrives.

### Can an AI agent be given control of plant equipment?
Not in this architecture, and we would not recommend it. Let an agent assemble context — asset history, nearby crews, pipe attributes, severity estimate — and draft a work order, with a human approving dispatch. The action space should be bounded and every step audited.
