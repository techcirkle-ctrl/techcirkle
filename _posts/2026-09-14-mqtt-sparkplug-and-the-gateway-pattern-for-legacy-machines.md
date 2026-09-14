---
layout: post
title: "MQTT, Sparkplug and the Gateway Pattern for Legacy Machines"
date: 2026-09-14 00:00:00 +0000
categories: ["Architecture", "IoT"]
tags: ["mqtt", "iot", "architecture", "manufacturing"]
description: "A practical connectivity design for mixed machine estates, covering namespace structure, gateway responsibilities and the buffering rules that matter."
image: "https://cdn.sanity.io/images/563mnkns/production/4b10323e115d1f1add50d6743e0e07c398396036-1600x1067.jpg"
author: "TechCirkle Editorial Team"
---

![Engineer using a tablet beside industrial equipment](https://cdn.sanity.io/images/563mnkns/production/4b10323e115d1f1add50d6743e0e07c398396036-1600x1067.jpg)

Between a clean architecture diagram and an actual plant sits thirty years of accumulated machinery.

Some of it speaks OPC UA properly. Some speaks Modbus over serial through a converter fitted in a hurry during a shutdown. Some writes a CSV to a file share at end of shift. At least one critical asset has a controller whose vendor no longer exists, and the only person who understands it retires next year.

Connectivity in this environment is routinely a third of the total effort in a first deployment, and it is the line most often missing from the estimate. This post covers a design that keeps it bounded.

## The gateway pattern

Put an edge gateway between the machines and everything else. Every machine connects to the gateway; nothing connects directly to the cloud or to enterprise systems.

The gateway owns five responsibilities, and keeping all five in one governed place is the entire point:

1. **Protocol translation** — OPC UA, Modbus, serial, file drops, vendor SDKs, all normalised to one internal representation.
2. **Buffering** — store-and-forward during uplink loss, with an explicit retention policy and a hard cap.
3. **Security enforcement** — the only path between operational and corporate networks, outbound connections only.
4. **Local inference** — anything with a line-rate deadline, such as visual inspection.
5. **Normalisation** — units, scaling, sign conventions and enumerations resolved here rather than downstream.

That fifth one prevents a specific and recurring class of incident. When unit conversion happens in three different consumers, one of them will eventually disagree, and the disagreement surfaces as a production decision made on a wrong number.

## Namespace design

Standardise on MQTT with a structured namespace for anything new. Sparkplug B is the pragmatic choice because it defines birth and death certificates, state management and a payload format, which removes a set of decisions that are otherwise made inconsistently per integration.

Structure the namespace so that topic hierarchy reflects physical and logical reality:

```
spBv1.0/<group>/<message_type>/<edge_node>/<device>
```

Map group to site, edge node to gateway or cell, device to machine. Resist encoding process semantics into the topic string beyond that — product codes, order numbers and shift identifiers belong in the payload, because they change and topics should not.

Two rules that save significant pain later:

- **Tag names come from a governed dictionary, not from whoever integrated the machine.** Two plants naming the same measurement differently produces datasets that look poolable and are not.
- **Every device publishes a birth certificate listing its tags and types.** Consumers discover capability rather than hard-coding it, and a machine replaced with a newer model does not silently break three downstream services.

![Automated robotic manufacturing](https://cdn.sanity.io/images/563mnkns/production/c25c0c5bf0f2e8a4ee64e1bb17759574e860b409-1600x967.jpg)

## Handling the awkward cases

**Serial and Modbus devices.** Poll them at the gateway and publish changes. Set the poll interval from what the process actually needs, not from what the device can sustain — polling a temperature at 100ms because you can produces volume without information.

**File-drop machines.** Watch the directory, parse, publish, archive the original. Keep the raw file, because parser assumptions will turn out to be wrong at least once and reprocessing needs the source.

**Machines with no digital output.** A stack-light sensor plus an operator terminal for reason entry gets you most of the value of full instrumentation for a fraction of the cost. Downtime duration comes from the light; downtime cause comes from the person. Cause is the part that has value.

**Orphaned controllers.** Treat these as read-only and wrap them. Do not modify anything that cannot be revalidated. Where the only available signal is a relay contact, that is still a usable event.

## Buffering rules

The naive queue-and-replay approach fails in three specific ways.

**Ordering.** Preserve order per aggregate — usually per machine or per work order — rather than globally. A global FIFO is heavier than necessary and still gets the important case wrong if it is partitioned carelessly.

**Duplication.** Retries are guaranteed. Generate a deterministic idempotency key at the gateway from the business event, and deduplicate upstream on that key. Do not assume downstream systems will catch duplicates; most will not, and a duplicated production confirmation becomes phantom inventory.

**Capacity.** Define what happens when the buffer fills, before it fills. Telemetry can be decimated progressively. Production confirmations cannot be dropped, which means telemetry needs a hard cap that guarantees space for transactions.

## Clocks

Edge devices drift and occasionally come up with a clock set to the epoch.

Carry two values on every message: a monotonic sequence number per device, which is what you order by, and the device's wall-clock reading, which is advisory. On reconnect, compare device time against server time and flag implausible skew rather than silently accepting timestamps that place last night's production in 1970.

## The audit that should precede all of this

Before committing to a timeline, walk the floor and record every asset you intend to connect: what it is, what protocol it speaks, what data rate it can sustain, who owns it, whether it can be modified without revalidation, and when its maintenance windows are.

That spreadsheet is a better predictor of your schedule than any vendor estimate, and it is the artefact most often skipped in favour of a proposal that assumes a uniform estate.

Expect surprises. There is always at least one machine that turns out to be more work than the rest combined, and finding it in week two is considerably better than finding it in month five. The broader architecture this feeds into is covered in our [cloud application development guide](https://techcirkle.com/blog/cloud-application-development-guide).

---

## Frequently Asked Questions

**Why put a gateway between machines and the cloud?**
It concentrates protocol translation, buffering, security enforcement, local inference and unit normalisation in one governed place instead of scattering them across point integrations.

**Why Sparkplug rather than plain MQTT?**
It defines birth and death certificates, state management and a payload format, removing decisions that would otherwise be made inconsistently for every integration.

**What belongs in the topic and what belongs in the payload?**
Physical and logical location belongs in the topic. Product codes, order numbers and shift identifiers belong in the payload, because they change frequently and topics should be stable.

**How do you connect a machine with no digital output?**
A stack-light sensor for duration plus an operator terminal for reason entry captures most of the value, and the reason is the part that matters.

**What should the connectivity audit record?**
Every asset's protocol, sustainable data rate, owner, revalidation constraints and maintenance windows. It predicts the schedule better than vendor estimates.

---

*Full version: [Cloud Manufacturing Software in 2026](https://techcirkle.com/blog/cloud-manufacturing-software).*
